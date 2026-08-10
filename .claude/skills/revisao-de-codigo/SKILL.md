---
name: revisao-de-codigo
description: Use antes de abrir um Pull Request (para revisar seu próprio trabalho antes de submeter) e ao revisar o PR de um colega. Aplica-se a repositórios com arquivos HTML/CSS/JS soltos, sem build ou testes automatizados. Compara o branch atual com a main e reporta apenas o que mudou.
---

# Revisão de código

## Quando usar

- Antes de abrir um Pull Request, para revisar seu próprio código antes de submeter.
- Ao revisar o Pull Request de um colega.

## O que esta skill NÃO faz

- Não reclama de formatação (indentação, espaçamento, ponto e vírgula, aspas simples vs duplas).
- Não aponta um problema sem dizer exatamente onde ele está (arquivo e linha).
- Não sugere refatoração sem explicar qual problema concreto aquela mudança resolve.

## Passo 1 — Descobrir o que mudou

Rode, nessa ordem:

```bash
git fetch origin main
git diff origin/main --stat
```

Isso lista quais arquivos foram alterados. Em seguida:

```bash
git diff origin/main
```

Isso mostra as mudanças linha a linha. **A revisão deve considerar apenas o conteúdo que aparece neste diff.** Código que já existia antes na `main` e não foi tocado não deve ser comentado.

## Passo 2 — Investigar cada arquivo alterado

Para cada arquivo que aparece no diff:

1. Abra o arquivo completo (não só o trecho do diff) para entender o contexto ao redor da mudança.
2. Se o arquivo for HTML com JavaScript embutido, identifique as funções relacionadas à mudança (ex: funções de validação, manipulação de formulário, eventos de botão).
3. Use `grep -n "<termo>" <arquivo>` para localizar rapidamente onde uma função, variável ou padrão específico aparece, e obter o número da linha.

## Passo 3 — Verificar comportamento, não apenas ler

Sempre que for possível confirmar um problema executando algo (e não apenas lendo o código), faça isso antes de reportar. Nesse tipo de projeto (HTML/JS sem testes automatizados), "executar" significa:

- Rodar `grep -n` para confirmar (ou negar) a presença de uma validação, checagem ou padrão específico no arquivo.
- Rodar `git log -p -- <arquivo>` se for preciso entender se um trecho é novo ou pré-existente.
- Quando o ambiente permitir, abrir o arquivo HTML em um navegador e testar manualmente casos de entrada conhecidos (por exemplo, valores que deveriam ser aceitos e valores que deveriam ser rejeitados) para confirmar se o comportamento bate com o que o código sugere.

Se não for possível executar nada que confirme um achado (por exemplo, um comportamento que dependeria de rodar a aplicação com dados reais indisponíveis), isso deve ser marcado explicitamente como "apenas lido" no relatório final, e não apresentado como algo confirmado.

## Passo 4 — Classificar cada achado

Todo achado entra em exatamente uma destas duas categorias, nunca as duas:

**Quebra o programa** — o código produz um resultado logicamente incorreto ou impede o uso da funcionalidade. Nesse tipo de projeto, isso inclui:
- A validação aceita uma entrada que deveria ser rejeitada, ou rejeita uma entrada que deveria ser aceita (erro de lógica).
- Um erro de JavaScript interrompe a execução (ex: função undefined, exceção não tratada) impedindo o botão ou campo de funcionar.
- HTML malformado que impede o elemento de renderizar ou responder a eventos.

**Preferência** — tudo que funciona corretamente, mas poderia ser escrito de outro jeito (nomes de variável, forma de organizar o código, uso de uma função em vez de outra com o mesmo resultado). Itens de preferência só devem ser citados se vierem acompanhados do problema concreto que resolvem (Passo 5) — nunca como sugestão solta.

## Passo 5 — Reportar cada achado

Para cada achado, incluir obrigatoriamente:

1. **Arquivo e linha** exatos (ex: `validador/index.html:42`).
2. **Como reproduzir**: a entrada exata ou a ação exata que expõe o problema (ex: "digitar 111.111.111-11 no campo e clicar em Validar").
3. **Categoria**: "quebra o programa" ou "preferência" — nunca misturado.
4. Se for preferência: qual problema concreto a mudança sugerida resolveria.

## Passo 6 — Relatório final

Ao final da revisão, apresentar duas listas separadas:

- **Verificado executando comando**: cada achado desta lista deve citar qual comando foi rodado para confirmá-lo.
- **Apenas lido**: achados identificados por leitura do código, sem confirmação por execução.

Não misturar as duas listas nem omitir qual delas se aplica a cada achado.
