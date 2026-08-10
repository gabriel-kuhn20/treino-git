# treino-git


// Exemplo simples: valida se um número é primo
function ehPrimo(numero) {
  if (numero < 2) return false;
  for (let i = 2; i <= Math.sqrt(numero); i++) {
    if (numero % i === 0) return false;
  }
  return true;
}

// Testando
for (let n = 1; n <= 20; n++) {
  if (ehPrimo(n)) {
    console.log(`${n} é primo`);
  }
}