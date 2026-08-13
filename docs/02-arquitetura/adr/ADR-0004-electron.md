# ADR-0004 — Electron para o app desktop

- **Status:** Aceito — Electron é o **primeiro** cliente; Tauri vem como **segundo**
  cliente na E9, como prova de arquitetura
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

### Decisão: os dois, em ordem — e o segundo vira prova de arquitetura

A pergunta estava mal colocada. Não é "Electron **ou** Tauri": é **em que ordem**, e
para provar o quê.

O ADR-0002 afirma que o desktop nunca define regra de negócio — que ele só renderiza o
que o Control Plane autoriza. Hoje isso é uma **promessa**. Um segundo cliente,
construído contra o mesmo `openapi.yaml` **sem tocar o servidor**, é a **prova**.
E se o servidor precisar mudar, descobrimos que regra de negócio vazou para o cliente —
achado que nenhum teste unitário pega.

| | Cliente | Quando | Papel |
|---|---|---|---|
| 1º | **Electron** | SPEC-005 (V2) | Cliente de referência. Prova o produto ponta a ponta |
| 2º | **Tauri** | **E9**, depois do contrato estável | Prova que o contrato é client-agnostic |

**Por que Electron primeiro:**
- O Control Plane é a parte cara e arriscada; o desktop existe para prová-lo.
- `sqlite-vec` é trivial em Node e tem atrito em Rust. Resolver o E5 no caminho fácil
  isola o problema: no Tauri sobra o transporte, com o comportamento correto já conhecido.
- Curva de Rust durante o V2 travaria o roadmap inteiro.
- **Um segundo cliente só prova algo se o contrato já estiver estável.** Em paralelo,
  não provaria nada.

**Por que Tauri como segundo e não como descarte:** os argumentos a favor dele seguem
válidos — deny-by-default em tempo de compilação (que é a invariante 5 expressa pelo
framework), RAM e instalador menores, e melhor leitura como material didático em
repositório aberto.

### Critério de aprovação da E9 (binário)

> A SPEC do cliente Tauri é aprovada se, ao terminar, `git log` mostrar **zero commits
> em `apps/control-plane/`**.

Precisou mudar o servidor? Não é falha da task — é falha da arquitetura. Vira ADR
explicando o que vazou e por quê.

### O que precisa existir AGORA para a E9 ser barata

O ADR já dizia que a lógica do cliente fica em `apps/desktop/src/core/` sem importar
`electron`. Comentário não impede ninguém. Vira **gate de CI antes do T-03 escrever a
primeira linha**:

```
nenhum arquivo em apps/desktop/src/core/** pode importar 'electron'
```

Sem esse gate, na hora da E9 tudo estará entrelaçado e o segundo cliente custa 5× mais.
É a diferença entre a ideia funcionar e virar frustração.

### Spike cancelado

O spike de 1 dia existia para escolher entre os dois frameworks. Não há mais escolha a
fazer agora. O risco do `sqlite-vec` em Rust continua real e passa a ser escopo da E9 —
lá ele é conteúdo de estudo, não bloqueio de roadmap.

### Ressalva que sobrevive à decisão

Adotar Tauri na E9 **não** dispensará o gate de hardening: parte dos defaults do próprio
Tauri trabalha contra as garantias que ele promete. O job `security-invariants` continua
obrigatório nos dois clientes — muda o que ele testa (capabilities declaradas em vez de
`webPreferences`), não se ele existe.

## Fontes da revisão

- [Webview Versions — Tauri](https://v2.tauri.app/reference/webview-versions/)
- [Discussão: instabilidade do WebKitGTK — tauri-apps](https://github.com/orgs/tauri-apps/discussions/8524)
- [Linux Graphics Issues — Tauri](https://v2.tauri.app/develop/debug/linux-graphics/)
- [tauri-plugin-rusqlite2 (carrega extensões SQLite)](https://crates.io/crates/tauri-plugin-rusqlite2)
- [tauri-plugin-sql (oficial, via sqlx)](https://crates.io/crates/tauri-plugin-sql)
