---
name: revisao-validador-cpf
description: Revisa o diff de um branch contra a main em repositórios do trabalho de validador de CPF (validador/index.html, HTML de arquivo único com JS inline). Use sempre antes de abrir um Pull Request e sempre que for revisar o PR de um colega, mesmo que a pessoa só diga "olha meu código", "revisa esse validador", "esse CPF tá certo?" ou cole um index.html com validação de CPF. Também use quando alguém pedir para conferir se uma validação aceita CPF inválido ou rejeita CPF válido.
---

# Revisão de validador de CPF

Revisa **apenas o que mudou** em relação à `main`, separa defeito de gosto pessoal, e sempre diz onde está e como reproduzir.

O código revisado costuma ser gerado por agente e nunca foi executado por ninguém. Por isso a regra central desta skill é: **quando dá para rodar, rode**. Ler o código e achar que está certo não é o mesmo que ver o resultado.

---

## Passo 1 — Descobrir o que mudou

**Modo terminal** (Claude Code, com git disponível). Rode nesta ordem:

```bash
git fetch origin
git diff --stat origin/main...HEAD
git diff origin/main...HEAD
```

Se o alvo for o branch de um colega e você estiver na `main`:

```bash
git fetch origin
git diff --stat origin/main...origin/<branch-do-colega>
git diff origin/main...origin/<branch-do-colega>
```

O `...` (três pontos) compara com o ponto onde os branches se separaram. Com dois pontos você acaba listando commits da `main` que o colega não escreveu, e a revisão vira ruído.

**Modo web** (diff ou arquivo colado na conversa). Não invente o que não foi colado. Se veio o `index.html` inteiro sem diff, pergunte se o arquivo é novo. Se for novo, o arquivo inteiro é o diff. Se não for, peça a saída de `git diff origin/main...HEAD`.

**Regra de escopo:** só entra no relatório o que aparece no diff. Se você notar um problema em linha não modificada, ou não cite, ou cite numa nota separada no fim deixando claro que está fora do diff.

---

## Passo 2 — Executar antes de opinar

Extraia a função de validação do HTML e rode contra casos conhecidos. Se você tem terminal com `node`, faça isso sempre; leva menos de um minuto e transforma achado suspeito em achado provado.

Salve a função do colega num arquivo temporário e rode este arnês, que compara a implementação dele com uma implementação de referência:

```javascript
// referencia.js — implementação correta, usada só como gabarito
function cpfValido(entrada) {
  const d = String(entrada).replace(/\D/g, '');
  if (d.length !== 11) return false;
  if (/^(\d)\1{10}$/.test(d)) return false;
  const dv = (base, pesoInicial) => {
    let soma = 0;
    for (let i = 0; i < base.length; i++) soma += Number(base[i]) * (pesoInicial - i);
    const r = (soma * 10) % 11;
    return r === 10 ? 0 : r;
  };
  return dv(d.slice(0, 9), 10) === Number(d[9]) && dv(d.slice(0, 10), 11) === Number(d[10]);
}

// Gera CPFs válidos para o teste, em vez de depender de uma lista fixa
function gerarValido() {
  const base = Array.from({ length: 9 }, () => Math.floor(Math.random() * 10)).join('');
  const dv = (b, p) => {
    let s = 0;
    for (let i = 0; i < b.length; i++) s += Number(b[i]) * (p - i);
    const r = (s * 10) % 11;
    return r === 10 ? 0 : r;
  };
  const d1 = dv(base, 10);
  const d2 = dv(base + d1, 11);
  return base + d1 + d2;
}

const casos = [
  ['529.982.247-25', true,  'válido com máscara'],
  ['52998224725',    true,  'válido sem máscara'],
  ['111.444.777-35', true,  'válido, dígito verificador 3 e 5'],
  ['529.982.247-26', false, 'último dígito trocado'],
  ['111.111.111-11', false, 'todos os dígitos iguais'],
  ['000.000.000-00', false, 'todos zeros'],
  ['123.456.789-09', true,  'válido, sequência crescente'],
  ['1234567890',     false, 'dez dígitos'],
  ['123456789012',   false, 'doze dígitos'],
  ['',               false, 'campo vazio'],
  ['abcdefghijk',    false, 'só letras'],
  ['529.982.247-2a', false, 'letra no lugar do dígito'],
];

// adiciona 200 CPFs válidos gerados, incluindo os casos de resto 10
for (let i = 0; i < 200; i++) casos.push([gerarValido(), true, 'gerado']);

let falhas = 0;
for (const [entrada, esperado, rotulo] of casos) {
  const obtido = validarCPF(entrada); // <- troque pelo nome da função do colega
  if (Boolean(obtido) !== esperado) {
    falhas++;
    console.log(`FALHOU  ${rotulo.padEnd(28)} entrada="${entrada}" esperado=${esperado} obtido=${obtido}`);
  }
}
console.log(falhas === 0 ? 'todos os casos passaram' : `${falhas} caso(s) falharam`);
```

Rode com `node teste.js`. Se a função depender do DOM (`document.getElementById` dentro dela), stub o mínimo ou copie só o miolo do cálculo — e registre no relatório que você isolou a função, porque isso muda o que o teste prova.

**Se não der para executar** (modo web, ou função grudada no DOM sem separação possível), não invente resultado. Faça a conferência lendo e diga no relatório que foi leitura.

---

## Passo 3 — Onde esses validadores costumam falhar

Confira cada item. Não é para listar os que passaram, é para saber onde olhar.

1. **Segundo dígito verificador calculado sobre a base errada.** O primeiro DV usa os 9 primeiros dígitos com pesos 10 a 2. O segundo usa os 10 primeiros, já incluindo o DV1, com pesos 11 a 2. Trocar isso faz a função aceitar CPF inválido.
2. **Resto 10.** `(soma * 10) % 11` pode dar 10, e nesse caso o dígito é 0. Quem esqueceu esse caso rejeita CPFs válidos de verdade. É o defeito mais fácil de passar batido na leitura e o mais fácil de provar rodando.
3. **Dígitos repetidos.** `111.111.111-11` passa no cálculo dos dois dígitos. Sem o bloqueio explícito, a função aprova.
4. **Comprimento.** Verificar depois de limpar a máscara, não antes. E rejeitar 10 e 12 dígitos, não só string vazia.
5. **Máscara e caracteres não numéricos.** Se a limpeza usa `replace(/\D/g,'')`, letras somem silenciosamente e `'123.456.789-0a'` vira 10 dígitos. Veja se cai no teste de comprimento ou escapa.
6. **Comparação de tipo.** `d[9] == dv` com `==` funciona por coincidência; `===` entre string e número não. Só vira achado se mudar o resultado — teste antes de citar.
7. **Retorno para campo vazio.** Alguns retornam `undefined` em vez de `false`, e a tela mostra "CPF inválido" por acidente, não por decisão.
8. **Saída na tela.** `innerHTML` com o valor digitado pelo usuário é injeção de HTML. Teste digitando `<img src=x onerror=alert(1)>` e veja se executa.
9. **Botão sem `type="button"` dentro de `<form>`.** A página recarrega e o resultado some antes de ser lido.

---

## Passo 4 — Classificar

Duas listas, e elas não se misturam. Um achado só entra na primeira se você conseguir descrever a entrada que produz o resultado errado.

- **Quebra o programa:** a função dá resposta errada, a página quebra, ou o usuário vê algo que não é verdade.
- **Preferência:** melhora legibilidade, nomes, estrutura. Não muda nenhum resultado.

Se um achado da segunda lista não vier acompanhado do problema concreto que resolve, corte o achado. "Poderia extrair numa função" não é achado. "Esse bloco está duplicado nas linhas 34 e 51 e uma cópia já divergiu da outra" é.

**Fora de escopo, nunca cite:** indentação, aspas simples vs duplas, ponto e vírgula, quebra de linha, ordem dos atributos, tamanho de linha.

---

## Passo 5 — Escrever o relatório

Salve em `docs/estudo/revisao-<branch>.md` e use exatamente este formato:

```markdown
# Revisão — <branch> (comparado com main)

Arquivos no diff: <lista>

## Quebra o programa

### 1. <título curto do problema>
**Onde:** validador/index.html:<linha>
**Reproduzir:** <ação exata: qual valor digitar, qual botão clicar, o que aparece>
**Esperado:** <o que deveria acontecer>
**Obtido:** <o que acontece>

### 2. ...

## Preferência

### 1. <título curto>
**Onde:** validador/index.html:<linha>
**Problema que resolve:** <consequência concreta de deixar como está>

## O que foi verificado

**Executando comando:**
- <comando rodado> — <resultado>

**Apenas lido:**
- <o que foi conferido só por leitura, e por quê>
```

A seção final não é enfeite. Ela é o que separa achado provado de suspeita, e é o que permite a quem lê saber em que confiar. Se você não rodou nada, escreva "nenhum comando executado" e liste tudo em "apenas lido" — isso é honesto e útil. Inventar que rodou é o pior resultado possível desta skill.

Se nenhum achado entrar na primeira lista, diga isso com todas as letras em vez de promover um achado de preferência para preencher espaço.

---

## Ao comentar no PR do colega

Escreva com suas palavras, um achado por comentário, ancorado na linha. Cole o passo de reprodução, não o relatório inteiro. Comece pelos achados da primeira lista; se os de preferência não couberem, deixe de fora.
