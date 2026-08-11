# SPEC-001 — Gateway de inferência multiprovider com streaming

- **Status:** Em revisão
- **Épico:** E1 — Gateway de inferência
- **Data:** 2026-08-10
- **Teto de custo:** R$ 40 · **Teto de turnos:** 25
- **Depende de:** SPEC-000

---

## 1. Problema

Hoje cada colaborador fala direto com o provider usando a chave que tiver. A empresa
não sabe quanto gasta, não controla qual modelo é usado, não tem trilha de auditoria,
e a chave corporativa (quando existe) está espalhada em N máquinas.

Além disso, mudar de provider hoje exigiria mexer no cliente de cada colaborador —
lock-in prático.

Esta SPEC entrega a peça que resolve os três: um endpoint único de inferência,
server-side, com streaming, que abstrai o provider.

## 2. Escopo

**Dentro**
- `POST /v1/chat` com streaming SSE
- Drivers de provider: Anthropic e OpenAI (dois, para provar a abstração)
- Registro de modelos com metadados (provider, tier, preços in/out/cache, limites)
- Failover: provider indisponível/rate-limited → próximo da cadeia
- Normalização de resposta e de erro entre providers
- Contabilidade de tokens e custo por requisição, gravada no ledger
- Injeção da chave server-side a partir do vault

**Fora (explícito)**
- Resolução de política (SPEC-002)
- Reserva de budget (SPEC-003) — aqui só **medimos** o custo, ainda não bloqueamos
- Tool calling / MCP (SPEC-007)
- RAG (SPEC-006)
- Grafo LangGraph completo — esta SPEC é o `router/`, chamado por um nó trivial

**Não-objetivo**
- Não vamos suportar todo parâmetro de todo provider. Superfície comum + escape
  hatch por `provider_params` explicitamente opt-in.

## 3. Requisitos funcionais

| ID | Requisito |
|---|---|
| RF-01 | `POST /v1/chat` aceita `{workspace_id, model_id, messages[], stream}` e responde SSE quando `stream=true` |
| RF-02 | O `model_id` é lógico (ex.: `fast-general`), resolvido para (provider, modelo concreto) pelo registro |
| RF-03 | A API key do provider é lida do vault **dentro** do `router/` e nunca aparece em resposta, log ou trace |
| RF-04 | Falha do provider primário (5xx, 429, timeout) aciona o próximo da cadeia de failover do `model_id` |
| RF-05 | Failover é registrado em auditoria com provider de origem, destino e motivo |
| RF-06 | Erros de provider são normalizados para um envelope único `{code, message, retryable, provider}` |
| RF-07 | Ao fim de cada requisição, tokens (in/out/cache) e custo em BRL são gravados no ledger |
| RF-08 | Adicionar um provider novo exige apenas um driver + entrada de config; nenhum outro módulo muda |
| RF-09 | Todo evento SSE carrega `request_id` correlacionável com o trace OTel |
| RF-10 | Cliente desconectar no meio do stream cancela a chamada ao provider e liquida o custo parcial |

## 4. Requisitos não-funcionais

| ID | Requisito | Número |
|---|---|---|
| RNF-01 | Overhead do gateway sobre a chamada crua (p50) | < 150 ms |
| RNF-02 | Tempo até o primeiro token (p95), modelo rápido | < 1,2 s |
| RNF-03 | Precisão do custo calculado vs. fatura do provider | ±3% |
| RNF-04 | Chamadas simultâneas por worker | ≥ 50 |
| RNF-05 | Cobertura de teste em `router/` | ≥ 90% |

## 5. Invariantes de segurança tocadas

| Invariante | Como esta SPEC a respeita |
|---|---|
| 1. A chave nunca desce | RF-03; gate de CI da SPEC-000 continua ativo; redaction obrigatória no logger |
| 2. Política em cascata | Não implementada aqui, **mas** o handler já chama `policy.resolve()` que nesta SPEC é um stub que **nega tudo exceto workspaces de teste** — fail-closed desde o começo, não depois |
| 3. Budget antes do turno | Stub `budget.reserve()` que sempre aprova, **mas já no lugar certo do fluxo**, e o custo real já é gravado (RF-07). SPEC-003 só troca o stub |
| 4. Aprovação por conversa | Não tocada (sem tools ainda) |
| 5. Fail-closed | O stub de política nega por padrão. Se o registro de modelos não carregar, a rota retorna 503, não "usa o default" |

> Decisão de processo: stubs **fail-closed** e no lugar certo do fluxo. Isso evita a
> armadilha de "depois a gente pluga a política" — o ponto de plug já existe e já nega.

## 6. Contrato

```yaml
paths:
  /v1/chat:
    post:
      operationId: createChatCompletion
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: '#/components/schemas/ChatRequest' }
      responses:
        "200":
          description: SSE stream (text/event-stream) ou JSON se stream=false
        "403": { $ref: '#/components/responses/PolicyDenied' }
        "402": { $ref: '#/components/responses/BudgetExceeded' }
        "503": { $ref: '#/components/responses/UpstreamUnavailable' }

components:
  schemas:
    ChatRequest:
      type: object
      required: [workspace_id, model_id, messages]
      properties:
        workspace_id: { type: string }
        model_id:     { type: string, description: "id lógico, não do provider" }
        messages:
          type: array
          items: { $ref: '#/components/schemas/Message' }
        stream:       { type: boolean, default: true }
        max_tokens:   { type: integer, minimum: 1 }
        provider_params:
          type: object
          description: "escape hatch opt-in; não é repassado a menos que o modelo permita"
    ErrorEnvelope:
      type: object
      required: [code, message, retryable]
      properties:
        code:      { type: string, enum: [policy_denied, budget_exceeded, upstream_error, rate_limited, invalid_request, internal] }
        message:   { type: string }
        retryable: { type: boolean }
        provider:  { type: string, nullable: true }
        request_id:{ type: string }
```

Eventos SSE: `meta` (request_id, provider, modelo resolvido) → `delta`* →
`usage` (tokens + custo) → `done` | `error`.

Quebra compatibilidade? Não (endpoint novo).

## 7. Critérios de aceite

| ID | Critério | Como verificar |
|---|---|---|
| CA-01 | Stream funciona ponta a ponta com Anthropic | `pytest tests/e2e/test_chat_stream.py -k anthropic` (cassete gravado) |
| CA-02 | Stream funciona ponta a ponta com OpenAI, mesmo formato de evento | idem `-k openai`; asserção compara o **schema dos eventos** entre os dois |
| CA-03 | Nenhuma chave em resposta, log ou trace | `pytest -k test_no_secret_leak` — fuzz + varredura dos logs capturados |
| CA-04 | Failover aciona ao 429 do primário | `pytest -k test_failover_on_rate_limit`; auditoria contém origem/destino/motivo |
| CA-05 | Custo calculado bate com o esperado | `pytest -k test_cost_accounting` com usage fixo e tabela de preço; erro < 0,5% |
| CA-06 | Novo provider = 1 arquivo | teste adiciona `FakeProvider` só com o driver e o registro; suíte passa sem tocar `api/` |
| CA-07 | Workspace sem política é negado | `curl -d '{"workspace_id":"desconhecido",...}'` → 403 `policy_denied` |
| CA-08 | Registro de modelos ausente ⇒ 503, não default | teste sobe app com registro vazio → `/v1/chat` retorna 503 |
| CA-09 | Overhead p50 < 150 ms | `pytest tests/perf/test_overhead.py` contra provider fake com latência fixa |
| CA-10 | Desconexão do cliente cancela upstream | `pytest -k test_client_disconnect_cancels`; provider fake registra `cancelled` |

## 8. Riscos

| Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|
| Diferenças semânticas entre providers vazam para o contrato | alta | alto | Testes de conformidade rodam a **mesma** suíte contra todos os drivers |
| Tabela de preços desatualizar silenciosamente | alta | médio | Preço em config versionada + teste que compara com a fatura mensal; alerta se divergir >3% |
| Cancelamento não propagar e vazar custo | média | médio | CA-10; timeout duro por requisição |
| SSE atrás de proxy bufferizar | média | alto | `X-Accel-Buffering: no`, teste de integração com proxy real |

---

# Plano

## Abordagem escolhida

`router/` como pacote isolado, sem dependência de FastAPI, exposto por uma interface:

```python
class ProviderDriver(Protocol):
    async def stream(self, req: NormalizedRequest) -> AsyncIterator[StreamEvent]: ...
    def estimate_tokens(self, req: NormalizedRequest) -> int: ...
```

Registro de modelos em YAML versionado (`config/models.yaml`): id lógico → cadeia
ordenada de (provider, modelo, preços, limites). Failover percorre a cadeia.

A rota FastAPI é fina: valida → `policy.resolve()` (stub fail-closed) →
`budget.reserve()` (stub) → `router.stream()` → repassa SSE → `budget.settle()` +
ledger. O grafo LangGraph completo entra na SPEC-005; aqui um nó trivial já usa o
`router` para provar a interface.

## Alternativas descartadas

| Alternativa | Por que não |
|---|---|
| Usar um gateway pronto (LiteLLM/Bifrost) como núcleo | Nosso diferencial é budget hierárquico + política em cascata, que eles não fazem; e vira dependência no caminho crítico de segurança |
| Formato de evento igual ao da OpenAI ("padrão de fato") | Amarra nossa API à evolução de um provider — o oposto do objetivo |
| Implementar 4 providers de uma vez | 2 já provam a abstração; os outros viram tasks baratas depois |

## Arquivos

| Arquivo | Ação | O que muda |
|---|---|---|
| `apps/control-plane/src/router/__init__.py` | criar | fachada `Router.stream()` |
| `apps/control-plane/src/router/types.py` | criar | `NormalizedRequest`, `StreamEvent`, `Usage` (Pydantic) |
| `apps/control-plane/src/router/registry.py` | criar | carga e validação de `config/models.yaml` |
| `apps/control-plane/src/router/failover.py` | criar | política de retry/cadeia |
| `apps/control-plane/src/router/pricing.py` | criar | cálculo de custo BRL |
| `apps/control-plane/src/router/drivers/anthropic.py` | criar | driver |
| `apps/control-plane/src/router/drivers/openai.py` | criar | driver |
| `apps/control-plane/src/api/routes/chat.py` | criar | rota SSE |
| `apps/control-plane/src/policy/stub.py` | criar | resolve() que nega por padrão |
| `apps/control-plane/src/budget/stub.py` | criar | reserve() aprova, settle() grava ledger |
| `apps/control-plane/src/audit/ledger.py` | criar | append-only |
| `config/models.yaml` | criar | registro |
| `packages/contracts/openapi.yaml` | alterar | `/v1/chat` + schemas |

## Plano de teste

- **Conformidade:** uma suíte parametrizada roda contra todos os drivers registrados.
  Driver novo entra na parametrização automaticamente.
- **Cassetes** gravados de chamadas reais (um por provider) para não gastar em CI.
- **Provider fake** com latência e falhas programáveis para failover, perf e cancelamento.
- **Segurança:** teste que captura todo log/trace da suíte e falha se casar padrão de chave.

## Rollback

Endpoint novo, sem consumidor em produção. Reverter commit.

---

# Tasks

| ID | Task | Depende de | Paralelizável | Verificação |
|---|---|---|---|---|
| T-01 | `router/types.py` + `registry.py` + `config/models.yaml` | — | não | `pytest -k registry` |
| T-02 | `pricing.py` + tabela de preços | T-01 | sim | CA-05 |
| T-03 | Driver Anthropic + cassete | T-01 | sim | CA-01 |
| T-04 | Driver OpenAI + cassete | T-01 | sim | CA-02 |
| T-05 | `failover.py` + provider fake | T-01 | sim | CA-04 |
| T-06 | Rota SSE `/v1/chat` + contrato | T-01 | não | CA-01, CA-02 |
| T-07 | Stubs fail-closed de policy/budget + ledger | T-01 | sim | CA-07, CA-08 |
| T-08 | Teste de vazamento de segredo | T-03, T-04 | não | CA-03 |
| T-09 | Teste de overhead e de cancelamento | T-06 | não | CA-09, CA-10 |
| T-10 | Suíte de conformidade parametrizada | T-03, T-04 | não | CA-06 |

---

# Execução

| Task | Status | Implementor | Verify | Notas |
|---|---|---|---|---|
| T-01 | pendente | | | |

---

# Retro

- **Custo real:** — · **Tempo:** — · **Turnos:** —
