# Agent Experience (AX)

> "Código bom para humano é código bom para agente." A diferença em 2026 é que a
> legibilidade agora tem custo mensurável: contexto ruim = mais tokens, mais turnos,
> mais retrabalho, mais R$.

## Princípio da subtração

Antes de adicionar qualquer skill, MCP, plugin ou hook ao harness:

1. **Zere.** Observe o agente puro resolvendo a tarefa.
2. Identifique **onde exatamente** ele falhou ou gastou demais.
3. Adicione **uma** peça que ataca aquela falha.
4. Meça de novo.

Nada entra no harness "porque parece útil". Toda peça de `.claude/` precisa de uma
frase respondendo *que falha observada ela corrige*.

Sintoma de harness inchado: o agente lê 40 mil tokens de instrução para escrever uma
função de 10 linhas.

## Regras de AX deste repositório

### 1. Estrutura previsível e rasa
Um agente que precisa de 6 buscas para achar onde mexer já queimou orçamento.
Máximo 3 níveis de pasta dentro de `apps/*/src/`. Nome de pasta = responsabilidade.

### 2. Contrato antes de código
`packages/contracts/openapi.yaml` e os JSON Schemas são a fonte da verdade.
O agente lê o contrato em vez de inferir o formato do payload lendo 5 handlers.

### 3. Um comando de verificação

```bash
just check
```

Não `npm run lint && cd .. && pytest -k ... && ...`. Um comando, saída clara,
exit code confiável. É o que permite o agente se auto-verificar sem inventar.

### 4. Erro que diz o que fazer

```
❌ ValueError: invalid policy
✅ PolicyError: workspace 'acme' sem política e sem default pessoal.
   Fail-closed aplicado (ADR-0002 §5). Defina em /admin/workspaces/acme/policy.
```

Mensagem de erro é interface para o agente. Um erro bom evita uma rodada inteira
de investigação.

### 5. Regras perto do código
`CLAUDE.md` na raiz (regras globais) + `CLAUDE.md` por app (regras locais).
O agente não deve precisar carregar a raiz inteira para mexer no desktop.

### 6. Nomes que não exigem contexto
`resolve_effective_policy()` > `getPolicy2()`.
`BudgetReservation` > `BRec`.
Abreviação economiza 3 tokens e custa 300 de confusão.

### 7. Estado explícito
Nada de estado global implícito. O `State` do grafo é uma classe Pydantic declarada.
O agente consegue ler a estrutura em um lugar só.

### 8. Testes como documentação executável
O nome do teste descreve o comportamento:
`test_personal_default_never_widens_workspace_policy()`.
Um agente aprende as regras do sistema lendo a suíte.

## O que medir de AX

| Sinal | O que indica | Alvo |
|---|---|---|
| Tokens de contexto por task | Repositório difícil de navegar | ↓ ao longo do tempo |
| Turnos até o gate ficar verde | Ambiguidade na SPEC ou erro ruim | ≤ 3 |
| Nº de arquivos lidos por task | Estrutura confusa | ≤ 12 |
| Taxa de "agente mexeu fora do escopo" | Fronteira de módulo mal definida | ~0 |

Registrado em `metrics/`. Piorou por 2 SPECs seguidas ⇒ é dívida de AX, vira item
no backlog.

## Anti-padrões de AX

| Anti-padrão | Efeito |
|---|---|
| `utils.py` com 900 linhas de coisas não relacionadas | Agente lê tudo, aproveita 2% |
| Config espalhada em 6 lugares | Agente muda no lugar errado |
| Comentário que repete o código | Tokens sem informação |
| Mesmo conceito com 3 nomes (`policy`, `rules`, `perms`) | Busca falha; ver `glossario.md` |
| Instrução em `CLAUDE.md` que ninguém verifica | O agente aprende a ignorar o arquivo |
