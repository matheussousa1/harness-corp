# Pesquisa — Como times de software usam IA hoje (Ago/2026)

> Insumo obrigatório de leitura antes de escrever qualquer SPEC. Este documento
> justifica as escolhas de arquitetura e de processo do projeto.

## 1. O que mudou: de "vibe coding" para spec-driven

O padrão dominante em times de engenharia grandes **não é mais prompt ad-hoc**. É o
loop disciplinado:

```
SPEC  →  PLAN  →  TASKS  →  IMPLEMENT  →  VERIFY
```

A spec vira um **contrato executável**: objetivo, restrições, critérios de aceite
versionados em Git, ao lado do código. Ferramentas de referência do mercado:
GitHub Spec Kit, AWS Kiro, Claude Code Skills/Commands. Todas model-agnostic.

**Por que isso importa aqui:** quando o agente escreve o código, a spec passa a ser
o artefato de maior alavancagem que um humano produz. É exatamente a "Camada 3 —
Processos" do harness.

### Consequência prática adotada neste repo
- Nenhuma task de implementação começa sem uma SPEC aprovada (`specs/SPEC-XXX-*.md`).
- Toda SPEC tem critérios de aceite verificáveis por máquina (teste ou comando).
- Papéis separados: **Coordinator → Implementor(es) → Verifier**. Implementors podem
  rodar em paralelo; o Verifier é quem detecta conflito antes do merge.

## 2. O dado incômodo: IA aumenta velocidade e instabilidade juntas

Achados do DORA (State of DevOps / ROI of AI-Assisted Software Development):

| Sinal | Efeito observado |
|---|---|
| Adoção de IA | ↑ efetividade individual e qualidade percebida do código |
| Entrega | ↑ **instabilidade** de delivery |
| Causa provável | volume: geração de código cresce mais rápido que a capacidade de review/deploy absorver |
| Carga cognitiva | +67,4% de contextos de PR por dev/dia; +13,8% de retrabalho (work restarts) |
| Trabalho parado | +26% de tarefas em progresso sem atividade por 7+ dias |

Tese central do DORA: **IA é amplificador**. Fundamentos fracos → amplifica o caos.
Fundamentos fortes (CI, testes, review, plataforma) → é onde o ROI aparece.

Isso valida diretamente o ponto do vídeo-base: *"a IA multiplica a sua senioridade —
e também a sua confusão"*.

### Consequência prática adotada neste repo
- Gate de qualidade automatizado desde o dia 1 (`.github/workflows/ci.yml`), antes
  de existir feature.
- Métricas DORA **estendidas** com atribuição de IA, custo por entrega e
  durabilidade do código (ver `docs/04-metricas/`). DORA puro não basta quando
  30–70% do código é gerado.

## 3. Frameworks de orquestração: o que virou padrão

| Framework | Posição em 2026 | Uso neste projeto |
|---|---|---|
| **LangGraph** | Padrão de fato para agentes em produção. Grafo explícito, estado persistente, ciclos, branching condicional, human-in-the-loop nativo | **Runtime oficial** do harness (ADR-0001) |
| CrewAI | Colaboração multi-agente por papéis; menos controle de fluxo | Não usado |
| Claude Agent SDK / OpenAI Agents SDK | Ótimos para agente único acoplado ao provider | Usado só como *driver* de provider, atrás da nossa abstração |
| PydanticAI | Tipagem forte, testabilidade | Adotado para os **schemas de I/O** de tools e nós |
| n8n / Make | Automação de negócio, não engenharia de agente | Fora de escopo |

Padrões de produção do LangGraph que viraram consenso:
- Separar **API de request** de **workers de fila** (pub/sub). Nada de agente rodando
  dentro do request HTTP.
- **Checkpointer durável** (Postgres). >60% dos incidentes em produção de agentes
  vêm de gestão de estado.
- Orquestrador como serviço dedicado → planner, executores de tool e memória escalam
  independentemente.
- Container multi-stage, base slim, usuário não-root.
- **OpenTelemetry em cada passo do grafo**: latência de LLM, duração de tool, custo.

## 4. Gateway de IA corporativo: a camada que todo mundo construiu

Times enterprise hoje roteiam para **4+ providers** (Anthropic, OpenAI, Google Vertex,
AWS Bedrock) e dezenas de tiers de modelo. O gateway é onde vivem:

- API unificada (troca de modelo sem tocar na aplicação)
- Roteamento + **failover automático** (provider degradado ou rate-limited)
- **Budget hierárquico**: org → workspace → time → usuário, em tokens ou R$/mês
- Cache semântico
- Log de compliance e visibilidade de custo cross-provider
- Virtual keys por time (a chave real nunca sai do gateway)

Limitação recorrente de soluções prontas (ex.: gateways gerenciados de edge):
**não têm budget hierárquico nem virtual key por time**. É exatamente o gap que o
nosso Control Plane preenche.

**Isto valida ponto a ponto os princípios do harness corporativo do vídeo:**
"a chave nunca desce", "política em cascata", "budget antes do turno".

## 5. Memória e RAG local-first

Para desktop com dado sensível, o padrão é **local-first**:
- SQLite embarcado (já vem com qualquer app Electron) + extensão **`sqlite-vec`**
  para busca vetorial.
- Fallback: se a extensão não carregar na plataforma, cai para cálculo de
  similaridade em memória (brute-force). Padrão try/catch obrigatório.
- Portabilidade de arquivo único → backup e export triviais.
- Dado sensível (jurídico, médico, financeiro, **e conversas internas de empresa**)
  não sai da máquina.

Separação que adotamos:
- **Memória semântica de trabalho** → local, SQLite + `sqlite-vec`, por usuário.
- **Base de conhecimento corporativa (RAG)** → servidor, com controle de acesso por
  relacionamento (estilo ReBAC/Zanzibar): o resultado da busca é filtrado pela
  permissão do usuário *antes* de entrar no contexto do modelo.

## 6. Agent Experience (AX) = Developer Experience

Consenso do mercado e do vídeo-base: **código bom para humano é código bom para
agente**. O que muda é que agora a legibilidade tem ROI mensurável (menos tokens,
menos retrabalho).

Práticas adotadas:
- Estrutura de pastas previsível e rasa; nomes descritivos.
- `CLAUDE.md` na raiz + `CLAUDE.md` por app: regras que o agente lê antes de agir.
- Contratos antes de código (OpenAPI/JSON Schema) — o agente não precisa adivinhar.
- Comandos de verificação de um passo só (`just check`) — agente consegue se
  auto-verificar sem inventar o comando.

## 7. Fase 4 — métricas e otimização

O ciclo de feedback que fecha o harness:

1. Medir por execução: **custo (R$), tokens in/out/cache, wall time, nº de turnos,
   taxa de sucesso do gate**.
2. Perguntar: *o custo valeu a entrega?*
3. Agir: ajustar rules → trocar modelo por um mais barato que entregue igual →
   ajustar o processo.
4. Manter **multiprovider** para que "trocar modelo" seja uma linha de config,
   não um refactor. Isso é a defesa contra dependência de fornecedor.

## 8. Decisões que esta pesquisa fecha

| # | Decisão | ADR |
|---|---|---|
| 1 | LangGraph como runtime de orquestração | ADR-0001 |
| 2 | Arquitetura em 3 planos (Desktop / Control Plane / Providers) | ADR-0002 |
| 3 | Memória local `sqlite-vec` + RAG corporativo server-side | ADR-0003 |
| 4 | Electron (não Tauri) para o app desktop | ADR-0004 |
| 5 | Backend-first, contract-first | ADR-0005 |

## Fontes

- [From Spec Kits to Agentic Teams: 7 AI Trends Changing Software Engineering in 2026 — EPAM](https://www.epam.com/insights/ai/blogs/ai-trends-in-software-development)
- [Spec-Driven Development: A Spec-First Approach to AI-Native Engineering — Microsoft for Developers](https://developer.microsoft.com/blog/spec-driven-development-ai-native-engineering/)
- [Spec-Driven Development in 2026: What It Is, the Tooling, and How Teams Actually Use It — DEV](https://dev.to/krlz/spec-driven-development-in-2026-what-it-is-the-tooling-and-how-teams-actually-use-it-2fk2)
- [Spec-Driven Development & AI Agents Explained — Augment Code](https://www.augmentcode.com/guides/spec-driven-development-ai-agents-explained)
- [New DORA Report Claims Strong Engineering Foundations Drive AI Return on Investment — InfoQ](https://www.infoq.com/news/2026/05/dora-roi-ai-assisted-dev-report/)
- [DORA Metrics Are Not Enough in 2026: What Elite Engineering Teams Track Instead — Oobeya](https://oobeya.io/blog/dora-metrics-not-enough-2026)
- [DORA metrics: the complete guide to measuring DevOps performance in the AI era — DX](https://getdx.com/blog/dora-metrics/)
- [Building LangGraph: Designing an Agent Runtime from first principles — LangChain](https://www.langchain.com/blog/building-langgraph)
- [Production Deployment Architecture and Implementation Strategies for LangGraph](https://medium.com/@manjunath.kvmc/production-deployment-architecture-and-implementation-strategies-for-langgraph-9569a60ea79c)
- [LangGraph Agents in Production: Architecture, Costs & Real-World Outcomes — AlphaBOLD](https://www.alphabold.com/langgraph-agents-in-production/)
- [LLM Gateway Architecture: 2026 Engineering Reference — Digital Applied](https://www.digitalapplied.com/blog/llm-gateway-architecture-2026-engineering-reference)
- [Top 5 Enterprise AI Gateways for Multi-Model Routing in 2026 — Maxim](https://www.getmaxim.ai/articles/top-5-enterprise-ai-gateways-for-multi-model-routing-in-2026/)
- [Road to sqlite-vec: Exploring SQLite as a RAG vector database — Midswirl](https://www.midswirl.com/blog/road-to-sqlite-vec-exploring-sqlite-as-a-rag-vector-database)
- [Local-First RAG: Vector Search in SQLite — SitePoint](https://www.sitepoint.com/local-first-rag-vector-search-in-sqlite-with-hamming-distance/)
- [ReRAG — RAG seguro com controle de acesso ReBAC + SQLite Vector](https://github.com/ory/rerag-rebac)
