mkdir -p .claude/skills/revisao-de-codigo# Revisão — codigo/franca

## Ferramenta e modelo

Usei o Claude (modelo Claude Sonnet 5, via chat com acesso a terminal) com uma skill própria para revisão de código, `revisao-validador-cpf`, que segue um roteiro fixo: descobrir o diff contra a `main`, extrair a lógica de validação e rodá-la contra casos de teste em vez de só ler o código, e separar o que quebra o programa do que é preferência de estilo.

## Uma pergunta que ela me fez e que eu não tinha pensado

Antes de rodar a revisão, ela perguntou se a verificação deveria **rodar os comandos automaticamente** (build, testes) ou **só listar os comandos** pra eu rodar manualmente. Eu não tinha pensado nessa diferença — meu instinto seria só ler o código e confiar na leitura. Escolher "rodar automaticamente" foi o que permitiu pegar o achado abaixo: ele só apareceu porque a lógica foi de fato executada contra 213 casos de teste (13 manuais + 200 CPFs válidos gerados por uma implementação de referência independente), não porque alguém leu o código e achou que estava certo.

## Achado

Não houve bug que quebra o programa — os 213 casos passaram. Mas apareceu um achado de manutenção: o repositório já tem um validador de CPF funcionando em `teste.html` (raiz do projeto), com a mesma lógica de checksum implementada de um jeito diferente do `validador/index.html` novo. Ter dois validadores separados significa que uma correção futura pode ser aplicada em um e esquecida no outro, sem ninguém perceber — é o tipo de coisa que passa batido numa leitura rápida porque cada arquivo, isolado, parece correto.
