# ADR-0002 — Arquitetura em três planos

- **Status:** Aceito
- **Data:** 2026-08-10

## Contexto

O app roda na máquina do colaborador e precisa falar com LLMs. O caminho óbvio
(desktop → provider, com chave local) é o que todo produto de chat faz e é
inaceitável aqui: expõe a chave da empresa, torna política inaplicável e deixa o
custo invisível.

## Decisão

Três planos, com direção de confiança única:

```
Desktop (não confiável)  →  Control Plane (confiável)  →  Providers
```

- O Desktop **nunca** abre conexão com provider. Nem em desenvolvimento. Nem atrás
  de flag.
- Toda decisão de política, budget, modelo e tool é do Control Plane.
- O Desktop recebe apenas listas já filtradas ("estes modelos você pode usar").

## Alternativas rejeitadas

| Alternativa | Por que não |
|---|---|
| Desktop chama provider com chave do usuário | Chave na máquina; sem controle corporativo; é o problema que o produto resolve |
| Desktop chama provider com chave da empresa embutida | Pior: chave corporativa em N máquinas, extraível do bundle |
| Proxy local no Electron main | O segredo continua na máquina; só muda de processo |
| Gateway gerenciado de terceiro (edge) | Não oferece budget hierárquico nem virtual key por time — exatamente os requisitos do produto |

## Consequências

**Positivas**
- As cinco invariantes de segurança ficam aplicáveis em **um** lugar.
- Trocar provider é operação de servidor: zero deploy de cliente.
- Custo e auditoria são completos por construção — nada escapa.

**Negativas / custos aceitos**
- O Control Plane vira ponto único de falha → precisa de HA e de degradação
  fail-closed bem definida (ver modelo de segurança).
- Latência extra de um hop. Orçamento: **< 150 ms p50** de overhead sobre a chamada
  crua. Medido no CI de performance.
- Modo offline do desktop é limitado a memória local e histórico; sem inferência.

## Regra de verificação

Teste de CI que falha o build se o bundle do desktop contiver qualquer host de
provider (`api.anthropic.com`, `api.openai.com`, `generativelanguage.googleapis.com`, …)
ou padrão de chave.
