# ADR-0001 — LangGraph como runtime de orquestração

- **Status:** Aceito
- **Data:** 2026-08-10
- **Contexto de decisão:** Camada 2/3 do harness

## Contexto

Precisamos executar turnos de agente que: chamam múltiplas tools, podem parar no meio
para pedir aprovação humana, sobrevivem a restart de processo, e precisam ser
auditáveis passo a passo. Alternativas avaliadas: loop `while` próprio, CrewAI,
Claude Agent SDK / OpenAI Agents SDK, LangGraph.

## Decisão

**LangGraph** como runtime de orquestração no Control Plane, com checkpointer Postgres.

Os SDKs de provider (Claude Agent SDK etc.) ficam **abaixo** da nossa abstração, como
drivers dentro do `router/` — nunca como orquestrador.

## Justificativa

| Requisito | LangGraph | Loop próprio | CrewAI | SDK de provider |
|---|---|---|---|---|
| Human-in-the-loop com pausa durável | `interrupt()` nativo | construir | fraco | limitado |
| Estado sobrevive a restart | checkpointer | construir | não | não |
| Fluxo explícito e auditável | grafo | implícito no código | papéis, fluxo opaco | opaco |
| Subagentes com escopo | subgrafos | construir | sim | parcial |
| Independência de provider | sim | sim | sim | **não** (é o ponto do lock-in) |
| Maturidade em produção 2026 | padrão de fato | — | menor | acoplado |

O fator decisivo é o par **durabilidade de estado + human-in-the-loop**, que são
requisitos de segurança do produto (aprovação por conversa), não conveniências.

## Consequências

**Positivas**
- Aprovação humana e retomada saem de graça do runtime.
- Cada nó é ponto natural de telemetria (OTel) e de contabilidade de custo.
- Trocar de modelo não toca o grafo.

**Negativas / custos aceitos**
- Control Plane fica em Python (LangGraph maduro é Python-first). Stack poliglota
  com o desktop em TypeScript.
- Curva de aprendizado de estado/reducers.
- Dependência de um framework de terceiro no coração do sistema → mitigação:
  nós são funções puras sobre um `State` nosso; a lógica de negócio não importa
  nada do LangGraph.

## Restrições impostas ao código

- Nó de grafo = função pura sobre `State`. Efeito colateral só em tool.
- `recursion_limit` obrigatório em todo grafo (sem loop infinito).
- Checkpointer **sempre** Postgres, inclusive em dev. Nada de `MemorySaver` fora de teste.
- Orquestrador roda em worker, nunca dentro do request HTTP.
