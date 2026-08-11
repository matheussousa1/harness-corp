# Validação manual — SPEC-XXX / T-nn: <título da task>

- **O que esta task entregou:** <2 linhas, sem jargão>
- **Tempo estimado do teste:** <n> min
- **Precisa de aprovação humana:** sim

---

## 1. Pré-requisitos

O que precisa estar instalado ou rodando antes. Se já estava, diga isso.

| Item | Como verificar | Se faltar |
|---|---|---|
| `just` | `just --version` | `winget install --id Casey.Just --source winget` |

## 2. Passo a passo

> Um comando por passo. Sempre mostre a **saída esperada**. Sem "deve funcionar":
> o leitor precisa poder comparar o que viu com o que era para ver.

### Passo 1 — <o que este passo prova>

```bash
<comando exato>
```

**Esperado:**

```
<saída literal, ou o trecho que importa>
```

**Se der diferente:** <causa mais provável e o que fazer>

### Passo 2 — …

## 3. Prova pela negativa

> A parte mais importante para estudo. Um teste que só passa não prova nada —
> pode estar passando por acidente. Aqui você **quebra de propósito** e confirma
> que o sistema reclama.

### Quebra 1 — <o que estamos provando que é detectado>

```bash
<comando que introduz a falha>
```

Rode de novo o passo N. **Esperado:** falha, com esta mensagem:

```
<mensagem de erro esperada>
```

**Desfazer:**

```bash
<comando que reverte a quebra>
```

## 4. Checklist de aprovação

> Entregue **em branco**. Quem executa marca. Checklist pré-marcada pelo autor é
> carimbo, não verificação — é exatamente o que este documento existe para evitar.

- [ ] Todos os passos da seção 2 deram a saída esperada
- [ ] Todas as quebras da seção 3 foram detectadas
- [ ] O repositório voltou ao estado limpo (`git status` sem alterações não previstas)

## 5. Para estudar

> 3 a 5 itens. **Por que** foi feito assim, não o que foi feito. Cada item liga a um
> documento do repositório ou a uma fonte externa.

| Conceito | Por que importa aqui | Onde ler mais |
|---|---|---|
| <ex.: fail-closed> | <1 linha> | `docs/02-arquitetura/modelo-de-seguranca.md` |

## 6. Como desfazer tudo

> **Proibido nesta seção:** `git clean -fdx`, `git checkout -- .`, `git reset --hard`
> e qualquer comando que apague por varredura. Enquanto a task não está commitada, os
> arquivos da entrega são *untracked* — varredura apaga a própria task, e comando na
> raiz atinge trabalho de outras tasks.
>
> Liste os arquivos **explicitamente**, um a um.

```bash
<rm -f / rm -rf de cada arquivo e pasta que ESTA task criou>
```

Como conferir antes de rodar: `git status --short` deve listar exatamente esses
arquivos. Listou outra coisa? Pare — tem trabalho de outra task no meio.
