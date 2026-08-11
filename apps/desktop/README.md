# desktop

Cliente Electron + React + TypeScript. **Cliente não confiável, por definição.**

## O que este app NÃO faz

- Não guarda chave de provider. Nenhuma. Nem em dev.
- Não abre conexão com host de provider.
- Não decide política. Recebe listas já filtradas do Control Plane.
- Não decide aprovação de ação. Ele **exibe** o pedido; a decisão é gravada no servidor.

Se um requisito parecer exigir qualquer uma dessas quatro coisas, o requisito está
errado — ver `docs/02-arquitetura/adr/ADR-0002-tres-planos.md`.

## Estrutura alvo

```
src/
  main/      processo principal Electron: janela, IPC, keychain, spawn de MCP local
  preload/   ponte com ALLOWLIST explícita de canais. Nada de ipcRenderer cru.
  renderer/  UI React
  core/      lógica do cliente SEM importar 'electron' (mantém portabilidade — ADR-0004)
  memory/    SQLite + sqlite-vec, com fallback brute-force obrigatório
```

## Hardening obrigatório (gate de CI)

```ts
webPreferences: {
  contextIsolation: true,
  nodeIntegration: false,
  sandbox: true,
  webSecurity: true,
}
```

Mais: CSP restritiva, `setWindowOpenHandler` negando janelas novas, `will-navigate`
bloqueando saída da origem do app, auto-update assinado.

## Credenciais

Token de sessão do usuário e credenciais de ferramentas de terceiros vão para o
**keychain do SO** (Keychain / Credential Manager / libsecret). Nunca `localStorage`,
nunca JSON em disco.

Estado: não iniciado. Nasce em `specs/SPEC-000-fundacao-e-gates.md` (esqueleto) e
ganha função em `SPEC-005`.
