# ADR-0004 — Electron para o app desktop

- **Status:** Aceito
- **Data:** 2026-08-10

## Contexto

O produto precisa rodar na máquina do colaborador para: acessar keychain do SO,
manter memória local em SQLite, falar com servidores MCP locais e integrar com
ferramentas do desktop. Alternativas: Electron, Tauri, app web (PWA), nativo por SO.

## Decisão

**Electron + React + TypeScript.**

## Justificativa

| Critério | Electron | Tauri | Web/PWA | Nativo |
|---|---|---|---|---|
| Acesso a keychain do SO | sim | sim | **não** | sim |
| SQLite + extensão `sqlite-vec` | direto, ecossistema Node maduro | via Rust, `sqlite-vec` menos trilhado | não | sim |
| Spawn de servidores MCP locais (processos) | trivial | possível | **não** | sim |
| Velocidade de desenvolvimento (1 dev) | alta | média (Rust) | alta | baixa (3×) |
| Tamanho do binário | ~120 MB | ~10 MB | — | pequeno |
| Consumo de RAM | alto | baixo | — | baixo |
| Ecossistema/exemplos para app de IA | grande | crescendo | — | pequeno |

Os requisitos que **eliminam** Web/PWA são keychain e spawn de processo MCP local.
Entre Electron e Tauri, o desempate é ecossistema e velocidade de entrega com um
desenvolvedor só; o custo (binário grande, RAM) é aceitável para software corporativo
de desktop.

## Consequências

**Positivas**
- Reuso de todo o ecossistema Node para MCP e SQLite.
- Uma base de código para Windows, macOS e Linux.

**Negativas / custos aceitos**
- ~120 MB de instalador e consumo de RAM de um Chromium.
- **Superfície de segurança maior** → mitigação obrigatória, sem exceção:
  - `contextIsolation: true`, `nodeIntegration: false`, `sandbox: true`, `webSecurity: true`
  - preload expõe allowlist explícita de canais IPC; nada de `ipcRenderer` cru no renderer
  - CSP restritiva; `setWindowOpenHandler` nega janelas novas; `will-navigate` bloqueia
    navegação para fora da origem do app
  - auto-update assinado
- Atualização de versão do Electron vira tarefa recorrente de segurança (item fixo no
  backlog trimestral).

## Reversibilidade

A lógica do cliente fica em `apps/desktop/src/core/` sem importar `electron`.
A camada Electron (`main/`, `preload/`) é fina. Migrar para Tauri depois = reescrever
essa camada fina, não o app.
