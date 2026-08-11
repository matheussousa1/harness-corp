# Roadmap

Incrementos, não datas. Cada marco é uma **fatia vertical** que funciona ponta a ponta.

## V0 — "O tubo existe" (E0 + E1)

**Prova:** dá para conversar com um LLM passando pelo Control Plane, sem chave na
máquina, com custo medido e failover funcionando.

| SPEC | Entrega |
|---|---|
| SPEC-000 | Monorepo, `just check`, CI com 5 gates de segurança |
| SPEC-001 | `POST /v1/chat` streaming, 2 providers, failover, ledger de custo |

**Critério de saída:** `curl` no `/v1/chat` responde em streaming; teste de CI prova
que nenhuma chave chega ao cliente; custo do turno aparece no ledger.

---

## V1 — "O controle existe" (E2 + E3)

**Prova:** o admin decide o que cada workspace pode usar e até quanto; o sistema nega
corretamente.

| SPEC | Entrega |
|---|---|
| SPEC-002 | Policy engine: cascata, fail-closed, middleware em toda rota de inferência |
| SPEC-003 | Budget hierárquico: `reserve()`/`settle()`, contadores atômicos, alerta 80% |
| SPEC-004 | Painel admin: workspaces, modelos, tools, budgets, políticas |

**Critério de saída:** os stubs da SPEC-001 morrem; US-020 a US-024 passam.

---

## V2 — "O colaborador usa" (E4 + E5)

**Prova:** existe produto. Alguém consegue trabalhar com isso.

| SPEC | Entrega |
|---|---|
| SPEC-005 | Desktop Electron: auth, chat streaming, seletor de modelos permitidos, hardening |
| SPEC-006 | Memória local SQLite + `sqlite-vec` com fallback |

**Critério de saída:** instalar o app numa máquina limpa, autenticar, conversar,
e a memória local recuperar contexto de conversa anterior.

---

## V3 — "O agente sabe e age" (E6 + E7)

| SPEC | Entrega |
|---|---|
| SPEC-007 | KB corporativa com filtro ACL na consulta |
| SPEC-008 | Tools via MCP + aprovação por conversa com `interrupt()` do LangGraph |

**Critério de saída:** o agente responde citando documento interno que o usuário pode
ver, e pede confirmação antes de uma ação de escrita — sobrevivendo a restart do worker.

---

## V4 — "Dá para operar" (E8)

| SPEC | Entrega |
|---|---|
| SPEC-009 | OTel em cada nó, Langfuse, dashboards de custo, relatórios do admin |

**Critério de saída:** dado um `request_id`, ver o traço completo com custo por passo;
relatório mensal com erro < 3% vs. fatura.

---

## Ordem de trabalho (o que fazer agora)

1. ✅ Fundação de processo e arquitetura — **feito**
2. ⏭️ Revisar e aprovar `SPEC-001` (está em revisão)
3. ⏭️ Executar `SPEC-000` (fundação técnica precede tudo)
4. ⏭️ Executar `SPEC-001`
5. ⏭️ Medir a primeira entrega em `metrics/` e rodar a primeira retro do harness

## Regra de WIP

**Uma SPEC em execução por vez.** Tasks dentro dela podem ser paralelas (máx. 3
Implementors). Duas SPECs abertas simultaneamente = o Coordinator vira gargalo e a
qualidade do review cai — que é exatamente o risco que o DORA aponta na adoção de IA.
