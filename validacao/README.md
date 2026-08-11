# validacao/

Um tutorial de teste manual por task, escrito **pelo Implementor**, para o humano
aprovar a entrega com as próprias mãos.

Arquivo: `validacao/SPEC-XXX-T-nn.md`, a partir de `validacao/_template.md`.

## Por que isto existe

Três motivos, em ordem de peso:

1. **Aprendizado.** Este projeto é base de estudo. Ver o gate de segurança bloquear
   uma chave plantada ensina mais do que ler que ele existe.
2. **Aprovação informada.** Aprovar sem executar é carimbo. O tutorial torna a
   aprovação uma verificação real, feita por quem responde pelo produto.
3. **Tutorial que não funciona denuncia código que não funciona.** O Verifier executa
   o tutorial **verbatim**. Se os passos não reproduzem o resultado prometido, é
   REPROVADO — mesmo que a suíte automatizada esteja verde. Já pegou entrega que só
   funcionava na máquina de quem escreveu.

## Regras

- **Toda task gera um arquivo.** Se a task não produz nada observável manualmente
  (ex.: refactor interno), o arquivo documenta o comando automatizado que a cobre e
  o que exatamente ele prova. Não pule — arquivo ausente é REPROVADO.
- **Comando exato + saída esperada.** "Deve funcionar" não é saída esperada.
- **Seção 3 (prova pela negativa) é obrigatória** quando a task entrega um gate,
  uma validação ou uma regra de segurança. É onde se prova que o mecanismo está
  ligado, e não apenas presente.
- **Seção 5 explica o porquê, não o quê.** O "o quê" já está no código.
- Português, direto, sem pressupor que o leitor lembra da SPEC.

## Relação com os outros artefatos

| Artefato | Pergunta que responde | Público |
|---|---|---|
| SPEC — critérios de aceite | O que precisa ser verdade? | agente + humano |
| Suíte de testes | Continua verdade a cada commit? | máquina |
| `validacao/` | Como **eu** confirmo com as mãos, e o que aprendo? | humano |

Os três são complementares. Nenhum substitui o outro.
