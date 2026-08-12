# ADR-0004 — Electron para o app desktop

- **Status:** Aceito **provisoriamente** — revisão agendada para o início da SPEC-005
- **Data:** 2026-08-10 · **Revisado:** 2026-08-12

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

O toolchain de frontend (TypeScript, Vite, React, vitest) entregue em SPEC-000 T-01 é
**idêntico** nos dois caminhos. Trocar para Tauri não descarta nada do que existe hoje.

---

## Revisão de 2026-08-12 — Tauri revisitado

Motivo: benchmarks de mercado reforçando Tauri (instalador ~8MB vs ~165MB, RAM ociosa
~45MB vs ~180MB, cold start ~1,4s vs ~3,2s).

### O que esses números mudam: nada

São o mesmo custo que este ADR já aceitou explicitamente, medido com mais precisão.
Peso e RAM **não** foram os critérios de decisão. Foram: keychain (empate),
`sqlite-vec`, spawn de MCP local, e velocidade de entrega com um dev.

Um ADR se reverte quando uma premissa cai, não quando um custo já aceito é remedido.
Ressalva sobre as fontes: são blogs de conteúdo, e cold start varia por ordens de
grandeza conforme app e hardware. Direção correta, precisão não auditável.

### Premissa reverificada: `sqlite-vec` — confirmada

O plugin oficial `tauri-plugin-sql` usa `sqlx` e não carrega extensão de forma direta.
As saídas são buildar binários cross-platform da extensão e emitir `LOAD_EXTENSION`
manualmente, ou adotar `tauri-plugin-rusqlite2`, de comunidade. Viável, não trivial.

### Custo que faltava neste ADR: fragmentação de WebView

Tauri usa o WebView do SO: **WebView2 (Chromium) no Windows, WKWebView no macOS,
WebKitGTK no Linux** — três superfícies de bug distintas. Há discussão aberta no
repositório do Tauri relatando WebKitGTK instável e piorando a cada release
(animação CSS deixando o app borrado, elementos HTML com comportamento divergente),
com contribuidores desaconselhando Tauri para quem precisa de Linux hoje.

Os 165MB do Electron compram exatamente isto: um motor, um comportamento.
Nenhum benchmark de RAM mede esse custo.

**Consequência importante:** se o V1 não tiver Linux como alvo — provável, dado que a
frota corporativa típica é Windows — essa fraqueza do Tauri some do cálculo, e a
decisão fica muito mais aberta.

### O que este ADR subestimou

- **RAM em máquina corporativa.** 135MB de diferença numa máquina com 8GB rodando
  Teams, Chrome e Excel não é cosmético. Foi tratado como se fosse.
- **Valor de portfólio.** Tauri lê melhor que Electron em 2026, e o projeto é vitrine.

### Decisão: adiar com critério, não manter por inércia

Não se troca de framework por benchmark de terceiro — seria contradizer a tese de
medição deste projeto. Também não se mantém a escolha sem reexaminar.

**Spike obrigatório de 1 dia (timebox rígido) no início da SPEC-005**, antes de
escrever qualquer código de desktop. Provar em Tauri v2:

| # | Prova | Passou se |
|---|---|---|
| 1 | Abrir SQLite com `sqlite-vec` carregado e rodar uma busca vetorial | consulta retorna resultado, nos 3 SOs alvo |
| 2 | Fallback brute-force quando a extensão não carrega | mesma interface, resultado correto |
| 3 | Spawnar um servidor MCP local via sidecar e conversar com ele | handshake e uma chamada de tool |
| 4 | Ler e gravar credencial no keychain do SO | round-trip |

**Migra para Tauri** se as quatro provas passarem dentro do timebox.
**Mantém Electron** se alguma travar — e aí o ADR se confirma pelo motivo certo,
com evidência própria em vez de premissa herdada.

Registrar o resultado em `metrics/` e emendar este ADR com o veredito.

## Fontes da revisão

- [Webview Versions — Tauri](https://v2.tauri.app/reference/webview-versions/)
- [Discussão: instabilidade do WebKitGTK — tauri-apps](https://github.com/orgs/tauri-apps/discussions/8524)
- [Linux Graphics Issues — Tauri](https://v2.tauri.app/develop/debug/linux-graphics/)
- [tauri-plugin-rusqlite2 (carrega extensões SQLite)](https://crates.io/crates/tauri-plugin-rusqlite2)
- [tauri-plugin-sql (oficial, via sqlx)](https://crates.io/crates/tauri-plugin-sql)
