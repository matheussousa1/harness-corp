# ADR-0006 — `just` como command runner

- **Status:** Aceito
- **Data:** 2026-08-10
- **Responde à pergunta:** `make.ps1` vs `just`

## Contexto

`docs/03-processo/agent-experience.md` exige **um comando único de verificação**, que
funcione igual para humano e para agente. A máquina de desenvolvimento é Windows; o
alvo de CI e deploy é Linux.

`make` não vem no Windows (Git Bash não inclui). As opções eram: manter `Makefile` +
um `make.ps1` espelho, ou adotar um runner cross-platform.

## Decisão

**`just`**, com um `justfile` na raiz. Alvos: `setup`, `check`, `lint`, `types`,
`test`, `contract`.

Documentação, agentes e CI referenciam `just check`. `make` some do projeto.

## Justificativa

| Critério | `Makefile` + `make.ps1` | `just` |
|---|---|---|
| Uma definição só | **não** — dois arquivos que divergem em silêncio | sim |
| Windows sem instalar nada extra | `make.ps1` sim, `make` não | precisa instalar `just` (1 binário) |
| Sintaxe | tabs obrigatórios, espaço quebra sem avisar | tab ou espaço, tanto faz |
| Comportamento em erro | continua por padrão, precisa de flag | **para no primeiro erro** — que é o que se quer num gate |
| Passar argumentos para a receita | penoso | nativo |
| Instalação | — | `winget install --id Casey.Just`, `scoop`, `cargo`, `brew`, `apt` |
| CI | nativo | `extractions/setup-just` |

O que decidiu: **duplicação silenciosa**. Manter `Makefile` e `make.ps1` em sincronia
é trabalho manual que ninguém faz — e o dia em que divergirem, o agente vai rodar um
gate diferente do que o CI roda. Um gate que mente é pior que gate nenhum.

`make` continuaria certo se tivéssemos dependências reais entre arquivos de build.
Não temos: são atalhos para comandos. É exatamente o caso de uso do `just`.

## Consequências

**Positivas**
- Um arquivo, mesmo comportamento nos três SOs.
- `just check` para no primeiro erro — semântica correta para um gate.
- `just --list` documenta os alvos sozinho (bom para AX: o agente descobre o que existe).

**Negativas / custos aceitos**
- Dependência a instalar na máquina e no CI. Mitigação: `just` está no `winget`,
  `scoop`, `brew` e `apt`; o CI usa `extractions/setup-just`.
- `make` é mais conhecido. Mitigação: a receita `setup` e o README dizem como instalar.

## Instalação (Windows)

```bash
winget install --id Casey.Just --source winget
```

## Regra

Nenhum comando de verificação vive só na cabeça de alguém ou no YAML do CI.
Se o CI roda, o `justfile` roda igual. Divergência entre os dois é bug.

## Fontes

- [Just vs Make vs Task: picking the right command runner — Botmonster](https://botmonster.com/self-hosting/just-vs-make-vs-task-modern-command-runner/)
- [Make vs Just vs Mise vs go-task in 2026 — Mehdi Hadeli](https://mehdihadeli.com/blog/task-runners-comparison-2026)
- [Just: a command runner — LWN.net](https://lwn.net/Articles/1047715/)
