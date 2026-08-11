# Índice da documentação

Ordem de leitura para quem (ou o que) chega agora:

| # | Documento | Leia se… |
|---|---|---|
| 1 | [`00-pesquisa/2026-08-estado-da-arte.md`](00-pesquisa/2026-08-estado-da-arte.md) | quer entender **por que** as decisões são essas |
| 2 | [`01-produto/visao-e-escopo.md`](01-produto/visao-e-escopo.md) | precisa saber o que está dentro e fora do V1 |
| 3 | [`01-produto/glossario.md`](01-produto/glossario.md) | vai escrever qualquer coisa (use os termos daqui) |
| 4 | [`01-produto/personas.md`](01-produto/personas.md) | vai decidir prioridade |
| 5 | [`02-arquitetura/visao-geral.md`](02-arquitetura/visao-geral.md) | vai mexer em código |
| 6 | [`02-arquitetura/modelo-de-seguranca.md`](02-arquitetura/modelo-de-seguranca.md) | **sempre** — são as 5 invariantes |
| 7 | [`02-arquitetura/adr/`](02-arquitetura/adr/) | quer saber por que não foi feito do outro jeito |
| 8 | [`03-processo/metodologia.md`](03-processo/metodologia.md) | vai criar uma SPEC ou implementar |
| 9 | [`03-processo/definition-of-ready-done.md`](03-processo/definition-of-ready-done.md) | vai aprovar ou verificar algo |
| 10 | [`03-processo/pipeline.md`](03-processo/pipeline.md) | vai orquestrar agentes |
| 11 | [`03-processo/agent-experience.md`](03-processo/agent-experience.md) | vai adicionar peça ao harness |
| 12 | [`04-metricas/framework.md`](04-metricas/framework.md) | vai fechar uma entrega |
| 13 | [`05-runbooks/`](05-runbooks/) | algo quebrou |

## ADRs

| ADR | Decisão |
|---|---|
| [0001](02-arquitetura/adr/ADR-0001-langgraph-como-runtime.md) | LangGraph como runtime de orquestração |
| [0002](02-arquitetura/adr/ADR-0002-tres-planos.md) | Arquitetura em três planos (Desktop / Control Plane / Providers) |
| [0003](02-arquitetura/adr/ADR-0003-memoria-local-e-rag.md) | Memória local `sqlite-vec` + RAG corporativo server-side |
| [0004](02-arquitetura/adr/ADR-0004-electron.md) | Electron para o app desktop |
| [0005](02-arquitetura/adr/ADR-0005-backend-first-contract-first.md) | Backend-first e contract-first |

## Onde as coisas ficam

- **O que construir:** `specs/`, `backlog/`
- **Como construir:** `docs/03-processo/`, `.claude/`
- **Quanto custou:** `metrics/`
