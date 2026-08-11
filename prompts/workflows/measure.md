> **{ARGS}** = os argumentos passados na invocação (ex.: `SPEC-000 T-01`).
> Fonte única deste fluxo. Os arquivos de comando de cada ferramenta só apontam para cá.

Feche a medição da SPEC {ARGS}.

1. Colete os números da execução (custo em BRL, tokens in/output/cache, tempo de
   parede, turnos, reprovações do verifier, arquivos lidos/alterados, violações de
   escopo, modelo por estágio, minutos humanos).
2. Grave em `metrics/{ARGS}.json` no formato de
   `docs/04-metricas/framework.md`.
3. Calcule as derivadas: `custo_por_task`, `razao_retrabalho`.
4. Preencha a seção **Retro** da SPEC com:
   - o que ficou ambíguo na SPEC (causa raiz dos turnos extras)
   - **um** ajuste de rules ou de modelo para a próxima SPEC — só um, para conseguir
     atribuir o efeito
5. Compare com as 3 SPECs anteriores. Se `turns` ou `cost_brl` subiu, diga a hipótese.

Regra: não proponha trocar de modelo se `turns` está alto por ambiguidade de SPEC.
Corrija o processo primeiro; modelo depois.
