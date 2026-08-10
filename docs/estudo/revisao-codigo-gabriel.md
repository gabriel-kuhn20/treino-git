# Revisão — codigo/gabriel (comparado com main)

Arquivos no diff: `validador/index.html` (arquivo novo, 104 linhas)

## Quebra o programa

### 1. CPF com todos os dígitos iguais é aceito como válido
**Onde:** validador/index.html:58-96 (falta um bloqueio explícito antes do cálculo dos dígitos verificadores)
**Reproduzir:** Abrir a página, digitar `111.111.111-11` (ou `000.000.000-00`, ou qualquer sequência com os 11 dígitos repetidos) no campo e clicar em "Validar".
**Esperado:** "CPF inválido" — CPFs com todos os dígitos iguais nunca são documentos reais e devem ser rejeitados antes mesmo de calcular os dígitos verificadores.
**Obtido:** "CPF válido" (texto verde). Isso não é um caso isolado: para qualquer dígito `d` repetido 11 vezes, a soma dos dois dígitos verificadores dá matematicamente igual a `d`, então **todas as 10 sequências repetidas** (`000...0` até `999...9`) passam pelo cálculo e são aprovadas. Confirmado rodando a função extraída do arquivo contra um harness de teste (ver seção "O que foi verificado").

## Preferência

### 1. Ramo `resto == 11` nunca é alcançado
**Onde:** validador/index.html:72 e validador/index.html:88
**Problema que resolve:** `resto` vem de `(soma * 10) % 11`, cujo resultado só pode estar entre 0 e 10 — nunca 11. Quem ler o código pode gastar tempo tentando descobrir em que situação o CPF produziria resto 11, quando isso nunca acontece. Manter só a checagem `resto == 10` deixa claro que esse é o único caso especial real.

## O que foi verificado

**Executando comando:**
- Extraí a função `validar()` de `validador/index.html` (branch `codigo/gabriel`) para um arquivo Node, trocando a leitura/escrita do DOM por um retorno booleano, mantendo a lógica de validação intacta. Rodei com `node teste.js` contra 13 casos manuais + 200 CPFs válidos gerados aleatoriamente (implementação de referência com todos os dígitos-verificadores calculados corretamente, incluindo o caso de resto 10).
  - Resultado: 3 falhas, todas do mesmo tipo — `111.111.111-11`, `000.000.000-00` e `222.222.222-22` retornaram `true` (deveriam ser `false`). Os 200 CPFs válidos gerados e os demais 10 casos manuais (comprimento errado, dígito trocado, letras, campo vazio) passaram corretamente, confirmando que o cálculo dos dígitos verificadores em si (bases, pesos e tratamento do resto 10) está correto — o único problema é a ausência do bloqueio de dígitos repetidos.

**Apenas lido:**
- Conferi que o HTML não usa `<form>`, então o `<button onclick="validar()">` não recarrega a página — não é um risco aqui.
- Conferi que `resultado.innerHTML` só recebe strings fixas (`"CPF inválido"` / `"CPF válido"`), nunca o valor digitado pelo usuário — não há injeção de HTML via esse campo.
- Conferi que a limpeza `cpf.replace(/[^\d]+/g, '')` remove letras antes da checagem de comprimento, então entradas como `529.982.247-2a` caem corretamente no caso de comprimento inválido (confirmado também via execução, caso "letra no lugar do dígito").
