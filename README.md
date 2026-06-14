# Aula 10 - Consumo de APIs: Solução Completa

## 📋 Descrição

Esta é a solução completa do exercício prático da Aula 10 sobre consumo de APIs. O exercício original continha 4 esqueletos de código que precisavam ser completados com implementações robustas para consumo de APIs REST.

## 🎯 Objetivos

- Implementar fetch com verificação de status HTTP
- Criar timeout com AbortController para requisições demoradas
- Desenvolver retry com backoff exponencial para falhas transitórias
- Comparar requisições sequenciais vs paralelas com Promise.all

## 📁 Arquivos

- `Consumo_APIs.html` - Página interativa com as quatro implementações e seus testes

## 🔧 Implementações

### Exercício 1: Fetch com verificação de status
```javascript
async function fetchWithCheck(url) {
  const response = await fetch(url);

  // Verifica se response.ok é false
  if (!response.ok) {
    throw new Error(`Erro HTTP ${response.status}: ${url}`);
  }

  return response.json();
}
```

### Exercício 2: Timeout com AbortController
```javascript
async function fetchWithTimeout(url, ms = 5000) {
  const controller = new AbortController();
  const timerId = setTimeout(() => controller.abort(), ms);

  try {
    const response = await fetch(url, { signal: controller.signal });
    clearTimeout(timerId);
    return response;
  } catch (err) {
    if (err.name === 'AbortError') {
      throw new Error(`Timeout: sem resposta após ${ms}ms`);
    }
    throw err;
  }
}
```

### Exercício 3: Retry com Backoff Exponencial
```javascript
async function fetchWithRetry(url, options = {}, retries = 3, delay = 500) {
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      const res = await fetch(url, options);

      // Verifica se é um erro retryable e não é a última tentativa
      if (RETRYABLE_STATUS.includes(res.status) && attempt < retries) {
        const wait = delay * Math.pow(2, attempt - 1);
        console.log(`⚠️ Tentativa ${attempt} falhou (${res.status}). Aguardando ${wait}ms...`);
        await new Promise(r => setTimeout(r, wait));
        continue;
      }

      return res;
    } catch (err) {
      if (attempt === retries) throw err;
      const wait = delay * Math.pow(2, attempt - 1);
      console.log(`🔄 Erro de rede. Tentativa ${attempt}. Aguardando ${wait}ms...`);
      await new Promise(r => setTimeout(r, wait));
    }
  }
}
```

### Exercício 4: Requisições Paralelas
```javascript
// Versão sequencial (lenta)
async function buscarSequencial(nomes) {
  const resultados = [];
  for (const nome of nomes) {
    const res = await fetch(BASE + nome);
    const data = await res.json();
    resultados.push(data.name + ' #' + data.id);
  }
  return resultados;
}

// Versão paralela (rápida)
async function buscarParalela(nomes) {
  const promises = nomes.map(nome =>
    fetch(BASE + nome).then(r => r.json())
  );
  const resultados = await Promise.all(promises);
  return resultados.map(d => d.name + ' #' + d.id);
}
```

## 🚀 Como usar

1. Abra o arquivo `Consumo_APIs.html` no navegador.
2. Use os botões de teste de cada exercício ou clique em "Executar Todas as Soluções".
3. Mantenha uma conexão com a internet ativa para acessar a PokéAPI.

## 📊 Conceitos Abordados

- **Tratamento de erros HTTP**: Verificação de status codes
- **Timeout de requisições**: Uso de AbortController para cancelar requisições demoradas
- **Retry inteligente**: Backoff exponencial para evitar sobrecarga do servidor
- **Concorrência**: Comparação entre execução sequencial vs paralela

## 🔗 API Utilizada

- **PokéAPI**: https://pokeapi.co/api/v2/
- **Características**: HTTPS, CORS habilitado, sem necessidade de API Key
- **Endpoint principal**: `/pokemon/{nome}`

## 💡 Melhores Práticas

1. Sempre verifique o status da resposta antes de processar
2. Implemente timeout para evitar requisições travadas
3. Use retry com backoff exponencial para falhas temporárias
4. Aproveite requisições paralelas quando possível para melhorar performance
5. Forneça mensagens de erro claras e informativas

## 📈 Resultados Esperados

- **Exercício 1**: Busca bem-sucedida de dados do Pokémon com tratamento de erros
- **Exercício 2**: Demonstração de timeout funcionando com AbortController
- **Exercício 3**: Retry funcionando com simulação de falhas (503)
- **Exercício 4**: Promise.all sendo ~3-4x mais rápido que requisições sequenciais
