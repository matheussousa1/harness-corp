# SPEC-XXX — <título curto>

- **Status:** Rascunho | Em revisão | Aprovada | Em execução | Entregue | Cancelada
- **Épico:** E<n> — <nome>
- **Autor:** <spec-writer / humano>
- **Data:** AAAA-MM-DD
- **Teto de custo:** R$ <valor>  ·  **Teto de turnos:** <n>

> Preencha na ordem. Não pule para o Plano antes de fechar os critérios de aceite.

---

## 1. Problema

<Qual dor, de quem, e o que acontece hoje sem isso. Nenhuma menção a solução.>

## 2. Escopo

**Dentro**
- …

**Fora (explícito)**
- …

**Não-objetivo**
- …

## 3. Requisitos funcionais

| ID | Requisito |
|---|---|
| RF-01 | O sistema deve… |
| RF-02 | |

## 4. Requisitos não-funcionais

| ID | Requisito | Número |
|---|---|---|
| RNF-01 | Latência p95 | < … ms |
| RNF-02 | | |

## 5. Invariantes de segurança tocadas

| Invariante | Como esta SPEC a respeita |
|---|---|
| 1. A chave nunca desce | |
| 2. Política em cascata | |
| 3. Budget antes do turno | |
| 4. Aprovação por conversa | |
| 5. Fail-closed | |

<Se nenhuma é tocada, escreva "nenhuma" e justifique em uma linha.>

## 6. Contrato

Mudanças em `packages/contracts/openapi.yaml` / JSON Schemas:

```yaml
# trecho novo ou alterado
```

Quebra compatibilidade? ( ) sim  ( ) não — se sim, plano de versionamento:

## 7. Critérios de aceite

> Regra: cada item precisa ser provável por comando ou teste. Sem exceção.

| ID | Critério | Como verificar |
|---|---|---|
| CA-01 | … | `pytest tests/... -k ...` |
| CA-02 | … | `curl … \| jq …` retorna … |

## 8. Riscos

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|

---

# Plano

## Abordagem escolhida

<3–8 linhas.>

## Alternativas descartadas

| Alternativa | Por que não |
|---|---|

## Arquivos

| Arquivo | Ação | O que muda |
|---|---|---|
| `apps/control-plane/...` | criar | |

## Mudanças de schema/dados

## Plano de teste

- Unit: …
- Integração: …
- Falha/borda: …

## Rollback

<Como desfazer se der errado em produção.>

---

# Tasks

| ID | Task | Depende de | Paralelizável | Verificação |
|---|---|---|---|---|
| T-01 | | — | não | `just check && pytest -k …` |
| T-02 | | T-01 | sim | |

---

# Execução (preenchido durante)

| Task | Status | Implementor | Verify | Notas |
|---|---|---|---|---|

---

# Retro (preenchido no fim)

- **Custo real:** R$ …  ·  **Tempo:** …  ·  **Turnos:** …
- O que ficou ambíguo na SPEC:
- Ajuste de rules/modelo para a próxima:
