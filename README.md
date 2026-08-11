# harness-corp

Harness corporativo de IA: um **plano de controle** que centraliza modelos,
ferramentas, políticas e budget de IA de uma empresa, mais um **app desktop** onde o
colaborador conversa com o agente.

Não é "mais um wrapper de chat". O produto é a **camada de harness**: regras,
memória, skills, subagentes e processos que fazem qualquer modelo virar um
colaborador confiável dentro de uma empresa — e que permite trocar o modelo sem
reescrever nada.

> Projeto de estudo + portfólio + base para modernizar operação real com agentes.

---

## As 4 camadas

| Camada | O que é | Onde vive |
|---|---|---|
| **1 — Cérebro** | O LLM puro (Claude, GPT, Gemini, GLM…) | Providers externos |
| **2 — Harness proprietário** | System prompt, rules, memória, skills, subagentes, tools/MCP | `apps/control-plane`, `apps/desktop` |
| **3 — Processos** | Workflows, filas, pipelines spec-driven (nada de loop infinito) | `pipeline/`, `specs/` |
| **4 — Métricas** | Custo, tempo, tokens, qualidade → otimização e troca de modelo | `metrics/`, `docs/04-metricas/` |

## Princípios de segurança (não-negociáveis)

1. **A chave nunca desce.** API key de provider só existe no Control Plane.
2. **Política em cascata.** Workspace > default pessoal > nenhuma política.
3. **Budget antes do turno.** Consumo é verificado *antes* de liberar a inferência.
4. **Aprovação por conversa.** Admin define o que é auto-aprovado e o que exige confirmação.
5. **Fail-closed.** Política ausente, corrompida ou host offline ⇒ bloqueia.

Detalhamento: [`docs/02-arquitetura/modelo-de-seguranca.md`](docs/02-arquitetura/modelo-de-seguranca.md)

## Stack

| Peça | Escolha |
|---|---|
| Orquestração de agentes | **LangGraph** (Python) |
| Tipagem de tools/nós | Pydantic / PydanticAI |
| API do Control Plane | FastAPI + SSE |
| Estado do agente | Postgres (checkpointer LangGraph) |
| Fila / workers | Redis + worker pool |
| RAG corporativo | Postgres + pgvector, filtrado por permissão |
| Desktop | Electron + React + TypeScript |
| Memória local | SQLite + `sqlite-vec` (fallback brute-force) |
| Tools externas | MCP (Model Context Protocol) |
| Observabilidade | OpenTelemetry + Langfuse (self-host) |

## Mapa do repositório

```
docs/
  00-pesquisa/     pesquisa de mercado que fundamenta as decisões
  01-produto/      visão, escopo, personas, glossário
  02-arquitetura/  visão geral, modelo de segurança, ADRs
  03-processo/     metodologia spec-driven, DoR/DoD, pipeline, agent experience
  04-metricas/     framework de métricas da Fase 4
  05-runbooks/     operação
specs/             SPEC-XXX (contrato executável por feature) + templates
backlog/           épicos e user stories
pipeline/          definição dos grafos LangGraph do harness de desenvolvimento
apps/
  control-plane/   backend (FastAPI + LangGraph)
  desktop/         cliente Electron
packages/contracts/ OpenAPI + JSON Schemas compartilhados
metrics/           saída bruta das execuções (custo/tokens/tempo)
.claude/           harness DESTE repositório: agentes, comandos, skills, hooks
```

## Como o trabalho anda aqui

```
SPEC  →  PLAN  →  TASKS  →  IMPLEMENT  →  VERIFY  →  MEASURE
```

Nenhuma linha de código de feature entra sem SPEC aprovada.
Leia [`docs/03-processo/metodologia.md`](docs/03-processo/metodologia.md).

## Estado atual

**Fase 0 — Fundação.** Pesquisa, arquitetura, processo e backlog definidos.
Código de aplicação ainda não iniciado. Próximo passo: `SPEC-001` (Gateway de
Inferência) — ver [`backlog/roadmap.md`](backlog/roadmap.md).
