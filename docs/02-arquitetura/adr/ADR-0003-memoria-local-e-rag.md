# ADR-0003 — Memória local (`sqlite-vec`) + RAG corporativo server-side

- **Status:** Aceito
- **Data:** 2026-08-10

## Contexto

Duas necessidades diferentes foram confundidas em um só requisito ("o agente precisa
ter memória"):

1. **Memória de trabalho do indivíduo** — o que *este* colaborador vem fazendo.
   Alto volume, baixo valor coletivo, alta sensibilidade pessoal.
2. **Conhecimento da empresa** — políticas, documentação, decisões. Compartilhado,
   precisa de controle de acesso, precisa ser atualizado centralmente.

Tratá-las com a mesma infra leva ou a vazar dado pessoal para o servidor, ou a
espalhar conhecimento corporativo sem ACL.

## Decisão

**Duas memórias, dois lugares.**

| | Memória de trabalho | KB corporativa |
|---|---|---|
| Onde | Máquina do colaborador | Control Plane |
| Store | SQLite + `sqlite-vec` | Postgres + pgvector |
| Escopo | Um usuário | Workspace / coleções |
| Sobe para o servidor? | **Nunca** | É do servidor |
| Controle de acesso | Sistema de arquivos do usuário | Filtro ACL na query (modelo relacional, estilo ReBAC) |
| Quem injeta no contexto | Desktop, como trecho de mensagem | Orquestrador, como resultado de tool |

### `sqlite-vec` com fallback obrigatório

A extensão é binária e nem sempre carrega (plataforma, arquitetura, antivírus).
Padrão obrigatório:

```
try: carrega sqlite-vec  →  busca vetorial em SQL
except: modo brute-force em memória  →  mesma interface, mais lento
```

A interface de busca é a mesma nos dois modos. O app **funciona** nos dois; só muda
a performance. Teste cobre os dois caminhos.

### Filtro ACL vem antes, não depois

```
❌ buscar top-k → filtrar por permissão → devolver o que sobrou
✅ buscar top-k já restrito às coleções que o usuário pode ver
```

Filtrar depois vaza (o k é consumido por chunks proibidos) e cria canal lateral.

## Consequências

**Positivas**
- Dado pessoal de trabalho nunca trafega. Argumento forte de privacidade e de venda.
- Backup/export da memória do usuário = copiar um arquivo.
- Nenhum serviço de vetor extra na infra: reusa Postgres que já existe.

**Negativas / custos aceitos**
- Memória não segue o usuário entre máquinas no V1. Sincronização criptografada
  fica para o backlog.
- Dois caminhos de código de busca (local e servidor) → mitigado por uma interface
  comum `VectorStore` e um conjunto de testes de contrato rodado contra as duas
  implementações.
- Embeddings precisam ser gerados para a memória local → passam pelo Control Plane
  (endpoint de embedding), preservando a invariante "a chave nunca desce".
