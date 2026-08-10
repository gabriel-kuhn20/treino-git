# Estudo — Skill de revisão de código

**Ferramenta e modelo:** Claude (claude.ai), skill criada com o `skill-creator`
da Anthropic e executada em modo agente com acesso a terminal/git para rodar o
diff e o harness de teste em Node.

**Pergunta que ela me fez e que eu não tinha pensado:**
> [PREENCHER — qual pergunta o skill-creator te fez ao montar a skill que você
> não tinha considerado antes? ex.: se a revisão deveria separar "quebra o
> programa" de "preferência", ou como provar um achado em vez de só descrever.]

**Achado que peguei:**
Na revisão do `codigo/lucca`, a skill não encontrou nenhum problema que quebra o
programa — o cálculo dos dígitos verificadores, o bloqueio de CPFs com dígitos
repetidos e a limpeza da máscara funcionaram corretamente em todos os testes
(13 casos manuais + 200 CPFs gerados + um caso forçado de borda com resto 10).
O único achado foi de preferência: o cálculo do 1º e do 2º dígito verificador
está duplicado em dois blocos quase idênticos (linhas 218-224 e 227-233), que
diferem só no peso inicial e no range do loop — um risco de inconsistência caso
alguém precise ajustar essa lógica no futuro sem atualizar os dois blocos.
Relatório completo em `docs/estudo/revisao-codigo-lucca.md`.
