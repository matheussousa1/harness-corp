> **{ARGS}** = os argumentos passados na invocação (ex.: `SPEC-000 T-01`).
> Fonte única deste fluxo. Os arquivos de comando de cada ferramenta só apontam para cá.

Verifique **{ARGS}** com o papel `verifier` (prompts/roles/verifier.md).

O verifier deve ser um agente **diferente** de quem implementou.

Ele precisa:
1. Executar o comando de verificação de **cada** critério de aceite e colar a saída real.
2. Rodar `just check` e reportar o exit code.
3. Comparar `git diff --stat` com os arquivos previstos no plano.
4. Checar as invariantes de segurança tocadas pela SPEC.
5. Emitir um dos três vereditos.

Se **REPROVADO**: liste as falhas com arquivo:linha e devolva a task para a fila.
Se esta já for a 2ª reprovação da mesma task, marque-a como `blocked` e me chame —
o problema provavelmente está na SPEC ou no fatiamento, não no código.
