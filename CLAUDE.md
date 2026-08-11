# Regras do repositório (leia antes de agir)

Projeto: **harness-corp** — plano de controle de IA corporativa + app desktop.
Contexto de negócio e stack: `README.md`. Não repita a leitura se já leu nesta sessão.

## Regra zero

Nenhuma implementação de feature começa sem uma SPEC aprovada em `specs/`.
Se te pedirem código de feature sem SPEC, **pare e escreva a SPEC primeiro**
(`/spec`). Exceções: bugfix pontual, ajuste de docs, tooling de build.

## Fluxo obrigatório

```
/spec <feature>   → specs/SPEC-XXX-<slug>.md      (o QUE e o PORQUÊ)
/plan SPEC-XXX    → seção Plano dentro da SPEC     (o COMO, com arquivos)
/tasks SPEC-XXX   → seção Tasks dentro da SPEC     (fatias verificáveis)
/implement SPEC-XXX T-nn
/verify SPEC-XXX  → roda gates + confere critérios de aceite
```

## Ordem de construção

**Backend-first, contract-first.** O contrato (`packages/contracts/openapi.yaml`)
muda antes do servidor; o servidor antes do cliente. O desktop nunca define regra
de negócio — ele só renderiza o que o Control Plane autoriza.

## Invariantes de segurança — violar = build quebrado

1. **Nenhuma API key de provider** pode aparecer em `apps/desktop/**`, em variável
   de ambiente do cliente, ou em resposta de API. Só existe em `apps/control-plane`.
2. Toda rota de inferência resolve **política em cascata** (workspace > pessoal >
   nenhuma) antes de qualquer chamada a provider.
3. Toda inferência chama `budget.reserve()` **antes** do turno e `budget.settle()`
   depois. Sem reserva ⇒ 402.
4. Qualquer erro ao carregar/validar política ⇒ **negar** (fail-closed). Nunca
   `except: pass` seguido de "assume permitido".
5. Credenciais de ferramentas do usuário vão para o keychain do SO, nunca em disco
   em texto claro.

Se um requisito parecer exigir violar 1–5, **não implemente**: registre em
`docs/02-arquitetura/adr/` e pergunte.

## Convenções de código

- **Idioma:** docs, specs e stories em pt-BR. Código, identificadores, mensagens de
  commit, nomes de arquivo de código em **inglês**.
- Python: 3.12+, `ruff` + `mypy --strict`, Pydantic v2 para todo I/O de fronteira.
  Sem `Any` em assinatura pública.
- TypeScript: `strict: true`, sem `any`, sem `@ts-ignore` sem comentário justificando.
- Electron: `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true`.
  Renderer fala com main **só** via IPC tipado no preload allowlist.
- Nós de LangGraph são funções puras sobre o `State`; efeito colateral só em tools.
- Sem número mágico: limites, timeouts e preços de modelo vêm de config.

## Verificação

Um comando só:

```bash
make check
```

(equivale a lint + typecheck + testes + validação de contrato). Se você mudou
código e não rodou `make check`, o trabalho não está pronto.

## Definition of Done

Ver `docs/03-processo/definition-of-ready-done.md`. Resumo: critérios de aceite da
SPEC verificados, `make check` verde, ADR criado se houve decisão arquitetural,
métricas da execução registradas em `metrics/`.

## O que NÃO fazer

- Não criar abstração para um único caso de uso.
- Não adicionar dependência sem justificar na SPEC (custo de manutenção é real).
- Não "melhorar" arquivos fora do escopo da task atual — abra um item no backlog.
- Não deixar TODO sem item correspondente em `backlog/`.
