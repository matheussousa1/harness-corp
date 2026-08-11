# SPEC-000 — Fundação do repositório e gates de segurança

- **Status:** Aprovada
- **Épico:** E0 — Fundação
- **Data:** 2026-08-10
- **Teto de custo:** R$ 15 · **Teto de turnos:** 20

---

## 1. Problema

O projeto ainda não tem código. A pesquisa (`docs/00-pesquisa/`) mostra que adoção de
IA aumenta velocidade **e instabilidade** porque a geração cresce mais rápido que a
capacidade de review. Se as primeiras features entrarem antes dos gates, o gate vira
dívida e nunca é feito.

Além disso, três das cinco invariantes de segurança do produto só são confiáveis se
forem **verificadas por máquina** — comentário em `CLAUDE.md` não impede ninguém de
colocar uma chave no cliente.

## 2. Escopo

**Dentro**
- Estrutura de monorepo com `apps/control-plane` (Python) e `apps/desktop` (TS)
- `make check` como comando único de verificação
- CI com 5 gates: lint, types, test, contract, security-invariants
- Esqueleto de `packages/contracts/openapi.yaml` (só `/health`)
- Script de coleta de métricas (`metrics/collect.py`)

**Fora**
- Qualquer feature de produto
- Deploy / infra de produção
- UI além de uma janela em branco que sobe

**Não-objetivo**
- Não é para ficar bonito. É para o gate existir antes da primeira linha de feature.

## 3. Requisitos funcionais

| ID | Requisito |
|---|---|
| RF-01 | `make check` roda lint, types, test e contract nos dois apps e retorna exit code agregado |
| RF-02 | CI executa `make check` mais o job `security-invariants` em todo push e PR |
| RF-03 | Gate falha o build se um padrão de chave de provider aparecer em `apps/desktop/**` ou no bundle gerado |
| RF-04 | Gate falha o build se um host de provider for referenciado a partir de `apps/desktop/**` |
| RF-05 | Gate falha o build se houver `except`/`catch` que engula erro de política sem re-lançar |
| RF-06 | `apps/control-plane` sobe com `GET /health` respondendo `{"status":"ok"}` |
| RF-07 | `apps/desktop` builda com `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true` |
| RF-08 | Tipos TS e modelos Pydantic são **gerados** do `openapi.yaml`; CI falha se houver diff não commitado |

## 4. Requisitos não-funcionais

| ID | Requisito | Número |
|---|---|---|
| RNF-01 | Tempo de `make check` local | < 90 s |
| RNF-02 | Tempo do CI completo | < 6 min |
| RNF-03 | Cobertura mínima nos módulos `policy/` e `budget/` (quando existirem) | 90% |

## 5. Invariantes de segurança tocadas

| Invariante | Como esta SPEC a respeita |
|---|---|
| 1. A chave nunca desce | RF-03, RF-04 tornam a violação impossível de mergear |
| 2. Política em cascata | Ainda não há política; o gate de rotas nasce em SPEC-002 |
| 3. Budget antes do turno | idem |
| 4. Aprovação por conversa | não tocada |
| 5. Fail-closed | RF-05 proíbe o padrão fail-open no código desde o commit 1 |

## 6. Contrato

```yaml
openapi: 3.1.0
info: { title: harness-corp control plane, version: 0.1.0 }
paths:
  /health:
    get:
      responses:
        "200":
          content:
            application/json:
              schema:
                type: object
                required: [status]
                properties: { status: { type: string, enum: [ok] } }
```

Quebra compatibilidade? Não (primeira versão).

## 7. Critérios de aceite

| ID | Critério | Como verificar |
|---|---|---|
| CA-01 | `make check` passa em repositório limpo | `make check; echo $?` → `0` |
| CA-02 | Chave plantada no desktop quebra o build | plantar `sk-ant-test123` em `apps/desktop/src/x.ts` → job `security-invariants` falha |
| CA-03 | Host de provider no desktop quebra o build | plantar `https://api.anthropic.com` em `apps/desktop/src/x.ts` → job falha |
| CA-04 | Padrão fail-open quebra o build | plantar `except PolicyError: policy = Policy.default()` → job falha |
| CA-05 | `/health` responde | `curl -s localhost:8000/health \| jq -e '.status=="ok"'` |
| CA-06 | Contrato dessincronizado quebra o build | editar `openapi.yaml` sem regenerar → CI falha no job `contract` |
| CA-07 | Config insegura do Electron quebra o build | trocar para `nodeIntegration: true` → teste `test_electron_hardening` falha |

## 8. Riscos

| Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|
| Gate de regex com falso positivo (ex.: string em teste) | média | baixo | allowlist explícita por caminho, revisada |
| `make` indisponível no Windows | alta | médio | usar `just` ou script `make.ps1` equivalente; decidir no plano |
| CI lento desanima o uso | média | médio | cache de dependências; jobs em paralelo |

---

# Plano

## Abordagem escolhida

Monorepo simples, sem ferramenta de monorepo (nx/turbo) — overhead não se paga com
dois apps. `Makefile` na raiz orquestra os dois toolchains; no Windows, `make.ps1`
com os mesmos alvos. Gates de segurança em script Python único
(`scripts/security_invariants.py`) para ficar testável, não em `grep` no YAML do CI.

## Alternativas descartadas

| Alternativa | Por que não |
|---|---|
| Dois repositórios separados | Contrato compartilhado ficaria dessincronizado |
| Gates só como pre-commit hook | Contornável com `--no-verify`; precisa ser no CI |
| `grep` inline no workflow | Não testável, não roda local, quebra em cross-platform |

## Arquivos

| Arquivo | Ação | O que muda |
|---|---|---|
| `Makefile`, `make.ps1` | criar | alvos `check`, `lint`, `types`, `test`, `contract` |
| `scripts/security_invariants.py` | criar | os 3 gates de invariante + testes próprios |
| `packages/contracts/openapi.yaml` | criar | `/health` |
| `packages/contracts/generate.py` | criar | gera modelos Pydantic + tipos TS |
| `apps/control-plane/` | criar | FastAPI mínimo, `pyproject.toml`, ruff, mypy |
| `apps/desktop/` | criar | Electron + Vite + React, `tsconfig` strict |
| `apps/desktop/src/main/window.ts` | criar | hardening obrigatório |
| `.github/workflows/ci.yml` | existe | preencher os 5 jobs |
| `metrics/collect.py` | criar | agrega custo/tokens/tempo em `metrics/SPEC-XXX.json` |

## Plano de teste

- Unit: `scripts/security_invariants.py` tem suíte com casos positivos e negativos
  (arquivo que deve passar / arquivo que deve falhar).
- Integração: subir a API e bater em `/health`.
- Falha/borda: cada CA-02..CA-04 é um teste que **planta** a violação em fixture e
  espera exit code ≠ 0.

## Rollback

Nada em produção. Reverter o commit.

---

# Tasks

| ID | Task | Depende de | Paralelizável | Verificação |
|---|---|---|---|---|
| T-01 | Esqueleto do monorepo + `Makefile`/`make.ps1` | — | não | `make check` roda (mesmo sem nada a checar) |
| T-02 | `apps/control-plane` FastAPI + `/health` + ruff/mypy | T-01 | sim | `pytest tests/test_health.py` |
| T-03 | `apps/desktop` Electron+Vite+React com hardening | T-01 | sim | `test_electron_hardening` |
| T-04 | `openapi.yaml` + geração de tipos nos dois lados | T-01 | sim | `make contract` sem diff |
| T-05 | `scripts/security_invariants.py` + suíte | T-01 | sim | suíte verde; CA-02..CA-04 |
| T-06 | `.github/workflows/ci.yml` com os 5 jobs | T-02..T-05 | não | CI verde no PR |
| T-07 | `metrics/collect.py` | T-01 | sim | gera JSON válido do schema |

---

# Execução

| Task | Status | Implementor | Verify | Notas |
|---|---|---|---|---|
| T-01 | pendente | | | |
| T-02 | pendente | | | |
| T-03 | pendente | | | |
| T-04 | pendente | | | |
| T-05 | pendente | | | |
| T-06 | pendente | | | |
| T-07 | pendente | | | |

---

# Retro

- **Custo real:** — · **Tempo:** — · **Turnos:** —
