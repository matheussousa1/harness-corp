# metrics/

Saída da **Fase 4**. Um arquivo `SPEC-XXX.json` por entrega, no schema descrito em
`docs/04-metricas/framework.md`.

Estes arquivos são versionados de propósito: a série histórica é o que permite
responder "o harness está melhorando ou só ficando mais caro?".

## Como preencher

```
/measure SPEC-XXX
```

## O que olhar a cada 5 SPECs

1. `cost_brl` por task está caindo? Se não, por quê.
2. `cache_read / input` está subindo? (contexto estável = mais barato)
3. `turns` alto → **ambiguidade de SPEC**, não modelo fraco. Corrija o processo antes
   de subir de modelo.
4. `verify_rejections` alto → critérios de aceite fracos ou tasks grandes demais.
5. `files_read` alto com `files_changed` baixo → dívida de Agent Experience.

## Regra

Uma variável por ciclo de otimização. Trocar modelo e rules ao mesmo tempo não ensina
nada — e a série histórica fica inútil.
