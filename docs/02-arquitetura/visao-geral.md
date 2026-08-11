# Arquitetura — Visão Geral

## Regra estruturante

**Três planos, uma direção de confiança.** O Desktop nunca fala com Provider.
Toda decisão (política, budget, modelo, tool) é do Control Plane.

```mermaid
flowchart LR
  subgraph MAQ["Máquina do colaborador"]
    UI["Electron Renderer<br/>chat, aprovações"]
    MAIN["Electron Main<br/>IPC, keychain"]
    LOCAL[("SQLite + sqlite-vec<br/>memória local")]
    UI <-->|IPC tipado| MAIN
    MAIN <--> LOCAL
  end

  subgraph CP["Control Plane (servidor da empresa)"]
    API["FastAPI<br/>/v1/chat, /v1/policies"]
    POL["Policy Engine<br/>cascata + fail-closed"]
    BUD["Budget Service<br/>reserve / settle"]
    ORQ["Orquestrador LangGraph<br/>workers"]
    RAG["KB corporativa<br/>pgvector + filtro ACL"]
    ROUT["Router multiprovider<br/>failover"]
    VAULT[("Vault de chaves")]
    PG[("Postgres<br/>checkpointer + auditoria")]
    RDS[("Redis<br/>fila + contadores")]
  end

  subgraph PROV["Providers"]
    ANT["Anthropic"]
    OAI["OpenAI"]
    GOO["Google"]
    GLM["GLM / outros"]
  end

  MAIN -->|HTTPS + token do usuário| API
  API --> POL --> BUD --> ORQ
  ORQ --> RAG
  ORQ --> ROUT
  ROUT --> VAULT
  ROUT --> ANT & OAI & GOO & GLM
  ORQ <--> PG
  API <--> RDS
  ORQ <--> RDS

  ADMIN["Painel Admin"] --> API
```

## Componentes

### 1. Desktop (`apps/desktop`)
- Electron: `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true`.
- Renderer não tem acesso a `fs`, `net` nem a segredo. Só IPC via preload allowlist.
- Guarda: token de sessão do usuário e credenciais de ferramentas **no keychain do SO**.
- Memória local: SQLite + `sqlite-vec`, com fallback brute-force se a extensão não
  carregar na plataforma.
- Recebe do Control Plane a lista de modelos e tools **permitidos**. Nada mais aparece na UI.

### 2. Control Plane (`apps/control-plane`)

| Módulo | Responsabilidade |
|---|---|
| `api/` | FastAPI. Autentica, valida entrada, enfileira, faz streaming SSE de volta. **Nunca chama LLM direto.** |
| `policy/` | Resolve política em cascata. Única fonte de verdade sobre "pode ou não pode". Fail-closed. |
| `budget/` | `reserve()` antes do turno, `settle()` depois. Contadores atômicos no Redis, ledger no Postgres. |
| `orchestrator/` | Grafos LangGraph. Roda em **worker**, fora do request HTTP. |
| `router/` | Escolhe provider/modelo, aplica failover, normaliza a resposta. Única camada que toca a chave. |
| `knowledge/` | Ingest + busca na KB corporativa. Filtra por ACL **antes** de devolver ao grafo. |
| `audit/` | Log append-only de decisão de política, uso de tool, tokens e custo. |

### 3. Providers
Atrás de uma interface única. Adicionar provider = implementar um driver + linha de
config de preço. Nenhum outro módulo muda.

## Fluxo de um turno

```mermaid
sequenceDiagram
  participant D as Desktop
  participant A as API
  participant P as Policy
  participant B as Budget
  participant O as Orquestrador (LangGraph)
  participant R as Router
  participant V as Provider

  D->>A: POST /v1/chat {workspace, model, mensagem}
  A->>P: resolve(user, workspace)
  alt política ausente/corrompida
    P-->>A: DENY (fail-closed)
    A-->>D: 403 policy_unavailable
  end
  P-->>A: allow {modelos, tools, modo_aprovacao}
  A->>B: reserve(estimativa)
  alt sem saldo
    B-->>A: DENY
    A-->>D: 402 budget_exceeded
  end
  A->>O: enfileira turno (Redis)
  O->>O: carrega checkpoint (Postgres)
  O->>R: chamada de modelo
  R->>V: request com a chave (server-side)
  V-->>R: stream
  R-->>O: tokens
  O-->>A: eventos SSE
  A-->>D: stream da resposta
  Note over O: tool call → se modo=confirmação,<br/>interrupt() do LangGraph e espera humano
  O->>B: settle(uso real)
  O->>A: fim do turno + métricas
```

## Por que LangGraph e não um loop `while`

| Necessidade do produto | Recurso do LangGraph |
|---|---|
| Aprovação humana no meio da execução | `interrupt()` + retomada por checkpoint |
| Turno sobrevive a restart do worker | Checkpointer Postgres |
| Fluxo auditável ("por que o agente fez isso?") | Grafo explícito, cada nó rastreado |
| Subagentes com escopo próprio | Subgrafos |
| Limite de passos, sem loop infinito | Guardas de aresta + `recursion_limit` |

## Decisões de deploy

- API e workers são **processos separados**; escalam independentemente.
- Container multi-stage, base slim, usuário não-root.
- Estado do agente **sempre** no Postgres. Nada de estado em memória de processo —
  é a origem de >60% dos incidentes em agentes de produção.
- OpenTelemetry em cada nó do grafo e cada chamada de tool.

## Onde este projeto pode falhar (e o que vigia isso)

| Falha | Vigia |
|---|---|
| Chave vazar para o cliente | Teste de CI que varre `apps/desktop/**` e o bundle |
| Rota nova esquecer política | Teste que enumera as rotas e exige o middleware |
| Estado perdido em restart | Teste de integração: mata o worker no meio do turno e retoma |
| Custo estourar em silêncio | Alerta em 80% do budget + corte em 100% |
