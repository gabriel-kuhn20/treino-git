# Revisão — codigo/lucca (comparado com main)

Arquivos no diff: `Validador/index.html` (arquivo novo, 240 linhas)

## Quebra o programa

Nenhum achado. O cálculo dos dois dígitos verificadores, o bloqueio de CPFs com
dígitos repetidos e a limpeza da máscara antes da checagem de comprimento
funcionaram corretamente em todos os casos testados.

## Preferência

### 1. Cálculo do 1º e do 2º dígito verificador duplicado em dois blocos quase idênticos
**Onde:** Validador/index.html:218-224 e Validador/index.html:227-233
**Problema que resolve:** os dois blocos fazem a mesma coisa (somar dígito × peso
e tratar resto 10), mudando só o peso inicial (10 no primeiro, 11 no segundo) e o
range do loop (9 dígitos no primeiro, 10 no segundo). Se alguém precisar ajustar
essa lógica no futuro — por exemplo, corrigir um erro de peso — é fácil mudar um
bloco e esquecer o outro, já que são visualmente quase iguais mas com três
literais diferentes (peso, range, índice do dígito verificado) que precisam ficar
sincronizados. Extrair uma função `calcularDV(digitos, pesoInicial)` elimina esse
risco.

## O que foi verificado

**Executando comando:**
- Extraí a função `validarCPF()` de `Validador/index.html` (branch `codigo/lucca`)
  para um arquivo Node, sem alterar a lógica de cálculo, e reproduzi também o
  fluxo de `validar()` (limpeza da máscara + checagem de comprimento antes de
  chamar `validarCPF()`). Rodei com `node teste.js` contra 13 casos manuais + 200
  CPFs válidos gerados aleatoriamente. Resultado: **todos os casos passaram**.
- Rodei uma busca dirigida (20.000 tentativas) até encontrar um CPF cujo primeiro
  dígito verificador cai exatamente no caso de borda `resto === 10` (dígito vira
  0) — caso mais fácil de passar batido numa leitura. Exemplo encontrado:
  `25967984500`. `validarCPF()` retornou `true` corretamente.

**Apenas lido:**
- Conferi que não há `<form>` envolvendo o botão `#btnValidar`, então o clique
  não recarrega a página — não é um risco aqui.
- Conferi que `resultadoTexto.textContent` e `resultadoIcon.textContent` só
  recebem strings fixas do próprio código (`'CPF válido.'`, `'CPF inválido.'`,
  `'✓'`, `'✕'`), nunca o valor digitado pelo usuário — não há injeção de HTML
  via esses campos (usa `textContent`, não `innerHTML`).
- Conferi que a comparação `resto !== digitos[9]` é numérica dos dois lados
  (`digitos` vem de `.map(Number)`), então não há problema de coerção de tipo.
