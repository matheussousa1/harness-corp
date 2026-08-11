# Modelo de Segurança

Cinco invariantes. Cada uma tem dono no código e teste que a prova.

---

## 1. A chave nunca desce

**Regra:** a API key do provider existe apenas dentro do Control Plane, injetada
server-side pelo `router/`. Nunca em resposta de API, log, variável de ambiente do
cliente, ou bundle do Electron.

**Implementação**
- Chaves em vault (env do servidor em dev; secret manager em prod), lidas só por `router/`.
- O desktop autentica com **token de sessão do usuário**, não com chave de provider.
- Log estruturado com redaction: qualquer valor que case `sk-`, `sk-ant-`, `AIza` é mascarado.

**Teste que prova**
- `test_no_provider_key_in_client_bundle`: varre o bundle buildado do desktop.
- `test_api_response_never_contains_secret`: fuzz nas rotas, procura padrões de chave.

---

## 2. Política em cascata

**Regra:** precedência estrita — **Workspace > default pessoal > nenhuma**.
"Nenhuma" resolve para **negar**, não para "liberado".

```
resolve(user, workspace):
    p = policy_of(workspace)          # 1º
    if p is None: p = personal_default(user)   # 2º
    if p is None: return DENY_ALL              # 3º
    return p
```

**O que a política controla**
- Modelos habilitados (por provider e tier)
- Tools / servidores MCP habilitados
- Limites: max tokens por turno, max turnos por conversa, max custo por conversa
- Modo de aprovação: `auto` | `confirm` | `deny`
- Acesso à KB corporativa (quais coleções)

**Regra de ouro:** o usuário **não pode** ampliar o que o workspace concedeu. Um
default pessoal só se aplica onde não há política de workspace — nunca sobrepõe.

**Teste que prova**
- Teste de propriedade: para qualquer par (política pessoal, política de workspace),
  o efetivo é sempre ⊆ workspace quando workspace existe.

---

## 3. Budget antes do turno

**Regra:** nenhuma inferência começa sem reserva aprovada.

```
reserve(scope, estimativa)  →  ok | 402 budget_exceeded
   ... turno executa ...
settle(reserva_id, uso_real)
```

- Escopos hierárquicos: `org > workspace > time > usuário`. **Todos** são checados;
  o mais restritivo vence.
- Estimativa = tokens de entrada contados + teto de saída do modelo × preço do tier.
- Contadores atômicos no Redis (Lua script para checar-e-debitar sem race).
- Ledger imutável no Postgres para conciliação com a fatura do provider.
- Reserva não liquidada expira (TTL) e é devolvida — worker morto não congela budget.
- Alerta em 80%, corte em 100%.

**Teste que prova**
- Concorrência: N turnos simultâneos com saldo para N−1 ⇒ exatamente um 402.

---

## 4. Aprovação por conversa

**Regra:** o modo de aprovação vem da política e é aplicado no orquestrador, não na UI.

| Modo | Comportamento na chamada de tool |
|---|---|
| `auto` | Executa e registra em auditoria |
| `confirm` | `interrupt()` no grafo; estado persiste; espera decisão humana |
| `deny` | Tool não é sequer exposta ao modelo |

Ferramentas com efeito destrutivo ou externo (enviar e-mail, escrever em sistema
terceiro, apagar dado) são **sempre** `confirm` no mínimo, independente da política —
a política pode endurecer, nunca afrouxar.

A UI mostra o pedido de confirmação, mas **a decisão é gravada no servidor**. Cliente
comprometido não consegue auto-aprovar.

---

## 5. Fail-closed

**Regra:** ausência de informação = ausência de permissão.

| Situação | Comportamento |
|---|---|
| Cache de política corrompido ou ausente | Bloqueia o despacho. Trata como sem permissão. |
| Painel admin (host) offline | Degrada para **apenas** o que já estava liberado e válido no cache assinado; nenhum recurso novo. |
| Erro ao desserializar política | Bloqueia. Não usa default permissivo. |
| Timeout do Policy Engine | Bloqueia. |

**Proibido no código:**

```python
try:
    policy = load_policy()
except Exception:
    policy = Policy.default()   # ❌ isso é fail-open. Nunca.
```

O cache local de política é **assinado** e tem TTL. Expirado ou assinatura inválida ⇒
bloqueia.

---

## Superfícies adicionais

### Desktop (Electron)
- `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true`,
  `webSecurity: true`. Sem `remote`.
- Preload expõe uma allowlist explícita de canais IPC. Nada de `ipcRenderer` cru.
- CSP restritiva; navegação externa bloqueada (`will-navigate` / `setWindowOpenHandler`).
- Credenciais de ferramentas de terceiros (ex.: token do ClickUp) → **keychain do SO**
  (Keychain / Credential Manager / libsecret). Nunca em `localStorage` ou JSON em disco.

### RAG corporativo
- Filtro de permissão aplicado na **query**, não depois. Chunk que o usuário não pode
  ver nunca entra no contexto do modelo.
- Modelo estilo ReBAC: a permissão é derivada da relação usuário↔recurso.

### Auditoria
Append-only, com: quem, workspace, política efetiva aplicada, modelo, tools chamadas,
decisões de aprovação, tokens, custo, resultado. É o que responde "por que o agente
fez isso?" seis meses depois.

---

## Ameaças consideradas

| Ameaça | Defesa |
|---|---|
| Colaborador extrai a chave da empresa | Invariante 1 |
| Colaborador habilita modelo/tool não aprovado | Invariante 2, lista vem do servidor |
| Um usuário consome o budget de todos | Invariante 3, escopo hierárquico |
| Prompt injection faz o agente chamar tool destrutiva | Invariante 4 (`confirm` forçado) + allowlist de tools |
| Servidor de política cai e tudo vira permitido | Invariante 5 |
| Vazamento de dado de um setor para outro via RAG | Filtro ACL na query |
| Cliente adulterado auto-aprova ações | Decisão de aprovação é server-side |
