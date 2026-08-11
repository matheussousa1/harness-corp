# AGENTS.md — regras do repositório

> **Fonte única de regras para qualquer agente de IA neste projeto.**
> Claude Code, Codex, Gemini CLI, opencode, Cursor, Copilot e afins leem daqui.
> `CLAUDE.md` e `GEMINI.md` apontam para este arquivo — não duplique regra neles.

Projeto: **harness-corp** — plano de controle de IA corporativa + app desktop.
Contexto de negócio e stack: `README.md`. Não releia se já leu nesta sessão.

## Regra zero

Nenhuma implementação de feature começa sem uma SPEC aprovada em `specs/`.
Se te pedirem código de feature sem SPEC, **pare e escreva a SPEC primeiro**.
Exceções: bugfix pontual, ajuste de docs, tooling de build.

## Fluxo obrigatório

```
spec  SPEC-XXX   → specs/SPEC-XXX-<slug>.md   (o QUE e o PORQUÊ)
plan  SPEC-XXX   → seção Plano dentro da SPEC  (o COMO, com arquivos)
tasks SPEC-XXX   → seção Tasks dentro da SPEC  (fatias verificáveis)
implement SPEC-XXX T-nn
verify    SPEC-XXX
measure   SPEC-XXX
```

As instruções de cada etapa estão em **`prompts/workflows/<etapa>.md`** e os papéis em
**`prompts/roles/<papel>.md`**. Ambos são neutros de ferramenta.

- Se a sua ferramenta tem slash commands, eles já apontam para lá
  (`.claude/commands/`, `.gemini/commands/`, `.opencode/commands/`).
- Se não tem, o usuário vai dizer algo como *"implement SPEC-000 T-01"*. Nesse caso:
  **leia `prompts/workflows/implement.md` e siga por completo**, tratando
  `{ARGS}` como `SPEC-000 T-01`.

Ver `docs/03-processo/multi-agente.md`.

## Ordem de construção

**Backend-first, contract-first.** O contrato (`packages/contracts/openapi.yaml`)
muda antes do servidor; o servidor antes do cliente. O desktop nunca define regra de
negócio — só renderiza o que o Control Plane autoriza. (ADR-0005)

## Invariantes de segurança — violar = build quebrado

1. **Nenhuma API key de provider** pode aparecer em `apps/desktop/**`, em variável de
   ambiente do cliente, ou em resposta de API. Só existe em `apps/control-plane`.
2. Toda rota de inferência resolve **política em cascata** (workspace > pessoal >
   nenhuma) antes de qualquer chamada a provider.
3. Toda inferência chama `budget.reserve()` **antes** do turno e `budget.settle()`
   depois. Sem reserva ⇒ 402.
4. Qualquer erro ao carregar/validar política ⇒ **negar** (fail-closed). Nunca
   `except: pass` seguido de "assume permitido".
5. Credenciais de ferramentas do usuário vão para o keychain do SO, nunca em disco em
   texto claro.

Se um requisito parecer exigir violar 1–5, **não implemente**: registre em
`docs/02-arquitetura/adr/` e pergunte.

Detalhamento: `docs/02-arquitetura/modelo-de-seguranca.md`.

## Modelo de domínio

`docs/02-arquitetura/modelo-de-dominio.md` define as entidades, relações e invariantes
do sistema inteiro. Não invente entidade nem campo: se precisar de algo que não está
lá, é mudança de domínio — atualize o documento na SPEC **antes** de implementar.

## Convenções de código

- **Idioma:** docs, specs e stories em pt-BR. Código, identificadores, mensagens de
  commit e nomes de arquivo de código em **inglês**.
- Python: 3.12+, `ruff` + `mypy --strict`, Pydantic v2 para todo I/O de fronteira.
  Sem `Any` em assinatura pública.
- TypeScript: `strict: true`, sem `any`, sem `@ts-ignore` sem comentário justificando.
- Electron: `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true`.
  Renderer fala com main **só** via IPC tipado no preload allowlist.
- Nós de LangGraph são funções puras sobre o `State`; efeito colateral só em tools.
- Código gerado do contrato vive em `packages/contracts/generated/` e **nunca** é
  editado à mão.
- Sem número mágico: limites, timeouts e preços de modelo vêm de config.

## Verificação

Um comando só:

```bash
just check
```

(lint + typecheck + testes + validação de contrato — ADR-0006). Se você mudou código
e não rodou `just check`, o trabalho não está pronto. `just --list` mostra os alvos.

## Definition of Done

`docs/03-processo/definition-of-ready-done.md`. Resumo: critérios de aceite da SPEC
verificados **com saída de comando colada**, `just check` verde, ADR criado se houve
decisão arquitetural, métricas registradas em `metrics/`.

## O que NÃO fazer

- Não criar abstração para um único caso de uso.
- Não adicionar dependência sem justificar na SPEC.
- Não "melhorar" arquivos fora do escopo da task atual — abra item em `backlog/`.
- Não deixar TODO sem item correspondente em `backlog/`.
- Não duplicar regra deste arquivo em `CLAUDE.md`, `GEMINI.md` ou nos shims de comando.
