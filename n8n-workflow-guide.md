# 📰 Simply News - Guía de Flujo Personalizado con n8n y NewsAPI

## 🎯 Objetivo
Crear un flujo automatizado en n8n que procese las preferencias de cada cliente y genere noticias personalizadas usando NewsAPI.

## 📋 Estructura de Datos Recibidos

### Datos del Cliente (desde survey.html)
```json
{
  "clientId": "sn_1703123456789_abc123def",
  "email": "user@example.com",
  "name": "Juan Pérez",
  "telegram": "@juanperez",
  
  "newsPreferences": {
    "topics": ["technology", "business", "health"],
    "countries": ["us", "es", "fr"],
    "sources": ["cnn", "bbc-news", "reuters"],
    "language": "en",
    "frequency": "daily"
  },
  
  "location": {
    "primaryCountry": "es",
    "timezone": "Europe/Madrid",
    "selectedCountries": ["🇪🇸 Spain", "🇺🇸 United States"]
  },
  
  "metadata": {
    "timestamp": "2024-01-01T12:00:00.000Z",
    "source": "simply-news-survey",
    "version": "2.0",
    "invalidSources": ["some-local-source"],
    "totalSourcesSelected": 5
  },
  
  "workflowConfig": {
    "enablePersonalization": true,
    "createSchedule": true,
    "setupTelegramBot": true,
    "clientType": "survey_subscriber"
  }
}
```

## 🔧 Flujo de n8n - Configuración Paso a Paso

### 1. **Webhook de Entrada** 
```
Nodo: Webhook
URL: /webhook/simply-news-client-setup
Método: POST
```

### 2. **Validación y Procesamiento de Datos**
```javascript
// Nodo: Code (JavaScript)
const clientData = $json;

// Validar datos requeridos
if (!clientData.email || !clientData.newsPreferences) {
  throw new Error('Datos incompletos del cliente');
}

// Procesar preferencias de noticias
const processedData = {
  ...clientData,
  processedAt: new Date().toISOString(),
  status: 'processing'
};

return { json: processedData };
```

### 3. **Guardar Cliente en Base de Datos**
```sql
-- Nodo: PostgreSQL/MySQL
INSERT INTO clients (
  client_id, email, name, telegram, 
  news_preferences, location_data, 
  created_at, status
) VALUES (
  '{{ $json.clientId }}',
  '{{ $json.email }}',
  '{{ $json.name }}',
  '{{ $json.telegram }}',
  '{{ JSON.stringify($json.newsPreferences) }}',
  '{{ JSON.stringify($json.location) }}',
  NOW(),
  'active'
);
```

### 4. **Crear Llamadas Personalizadas a NewsAPI**
```javascript
// Nodo: Code - Generar llamadas NewsAPI
const preferences = $json.newsPreferences;
const clientId = $json.clientId;

const newsApiCalls = [];

// Para cada combinación de país y categoría
preferences.countries.forEach(country => {
  preferences.topics.forEach(topic => {
    
    // Llamada por fuentes específicas
    if (preferences.sources.length > 0) {
      newsApiCalls.push({
        clientId: clientId,
        type: 'sources',
        url: `https://newsapi.org/v2/top-headlines`,
        params: {
          sources: preferences.sources.slice(0, 20).join(','), // Max 20 sources
          language: preferences.language,
          pageSize: 10
        }
      });
    }
    
    // Llamada por país y categoría
    newsApiCalls.push({
      clientId: clientId,
      type: 'country_category',
      url: `https://newsapi.org/v2/top-headlines`,
      params: {
        country: country,
        category: topic,
        language: preferences.language,
        pageSize: 5
      }
    });
    
    // Llamada de búsqueda personalizada
    newsApiCalls.push({
      clientId: clientId,
      type: 'search',
      url: `https://newsapi.org/v2/everything`,
      params: {
        q: topic,
        language: preferences.language,
        sortBy: 'publishedAt',
        pageSize: 5,
        from: new Date(Date.now() - 24*60*60*1000).toISOString() // Últimas 24h
      }
    });
  });
});

return newsApiCalls.map(call => ({ json: call }));
```

### 5. **Ejecutar Llamadas a NewsAPI**
```javascript
// Nodo: HTTP Request (para cada llamada)
// Configuración dinámica:
{
  "method": "GET",
  "url": "{{ $json.url }}",
  "headers": {
    "X-API-Key": "TU_NEWSAPI_KEY_AQUI"
  },
  "qs": "{{ $json.params }}"
}
```

### 6. **Procesar y Filtrar Noticias**
```javascript
// Nodo: Code - Procesar respuestas NewsAPI
const apiResponse = $json;
const clientId = $('Code').all()[0].json.clientId;

if (!apiResponse.articles || apiResponse.articles.length === 0) {
  return { json: { clientId, articles: [], status: 'no_articles' } };
}

// Filtrar y procesar artículos
const processedArticles = apiResponse.articles
  .filter(article => article.title && article.url)
  .map(article => ({
    title: article.title,
    description: article.description,
    url: article.url,
    source: article.source.name,
    publishedAt: article.publishedAt,
    urlToImage: article.urlToImage,
    relevanceScore: calculateRelevance(article, preferences) // Función personalizada
  }))
  .sort((a, b) => b.relevanceScore - a.relevanceScore)
  .slice(0, 10); // Top 10 más relevantes

return { 
  json: { 
    clientId, 
    articles: processedArticles, 
    processedAt: new Date().toISOString() 
  } 
};

function calculateRelevance(article, preferences) {
  let score = 0;
  
  // Bonus por fuente preferida
  if (preferences.sources.some(source => 
    article.source.name.toLowerCase().includes(source.toLowerCase())
  )) {
    score += 10;
  }
  
  // Bonus por temas en título
  preferences.topics.forEach(topic => {
    if (article.title.toLowerCase().includes(topic.toLowerCase())) {
      score += 5;
    }
  });
  
  // Bonus por recencia
  const hoursAgo = (Date.now() - new Date(article.publishedAt)) / (1000 * 60 * 60);
  if (hoursAgo < 6) score += 3;
  else if (hoursAgo < 12) score += 2;
  else if (hoursAgo < 24) score += 1;
  
  return score;
}
```

### 7. **Generar Newsletter Personalizado**
```javascript
// Nodo: Code - Generar HTML Newsletter
const clientData = $('Webhook').first().json;
const articles = $json.articles;

const newsletter = generatePersonalizedNewsletter(clientData, articles);

return { json: { 
  clientId: clientData.clientId,
  email: clientData.email,
  newsletter: newsletter,
  articleCount: articles.length
}};

function generatePersonalizedNewsletter(client, articles) {
  return `
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Simply News - ${client.name}</title>
  <style>
    body { font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; }
    .header { background: #667eea; color: white; padding: 20px; text-align: center; }
    .article { border-bottom: 1px solid #eee; padding: 15px 0; }
    .article img { max-width: 100%; height: auto; }
    .source { color: #666; font-size: 12px; }
    .footer { background: #f8f9fa; padding: 20px; text-align: center; margin-top: 30px; }
  </style>
</head>
<body>
  <div class="header">
    <h1>📰 Simply News</h1>
    <p>Personalized for ${client.name}</p>
    <p>${new Date().toLocaleDateString()}</p>
  </div>
  
  <div class="content">
    <h2>Your ${articles.length} Top News Stories</h2>
    
    ${articles.map(article => `
      <div class="article">
        ${article.urlToImage ? `<img src="${article.urlToImage}" alt="${article.title}">` : ''}
        <h3><a href="${article.url}" target="_blank">${article.title}</a></h3>
        <p>${article.description || ''}</p>
        <div class="source">
          <strong>${article.source}</strong> • ${new Date(article.publishedAt).toLocaleString()}
          ${article.relevanceScore > 5 ? ' • ⭐ High Relevance' : ''}
        </div>
      </div>
    `).join('')}
  </div>
  
  <div class="footer">
    <p>Delivered by Simply News | <a href="#unsubscribe">Unsubscribe</a></p>
    <p>Frequency: ${client.newsPreferences.frequency} | 
       Topics: ${client.newsPreferences.topics.join(', ')}</p>
  </div>
</body>
</html>`;
}
```

### 8. **Envío por Email**
```javascript
// Nodo: Send Email
{
  "to": "{{ $json.email }}",
  "subject": "📰 Simply News - Your Personalized News Update",
  "html": "{{ $json.newsletter }}",
  "from": "news@simplynews.com"
}
```

### 9. **Envío por Telegram (Opcional)**
```javascript
// Nodo: Telegram
// Si el cliente tiene Telegram configurado
if ($('Webhook').first().json.telegram) {
  // Generar versión reducida para Telegram
  const articles = $json.articles.slice(0, 5); // Solo top 5
  const telegramMessage = articles.map(article => 
    `📰 *${article.title}*\n${article.description}\n🔗 [Read more](${article.url})\n📊 Source: ${article.source}\n`
  ).join('\n---\n');
  
  return {
    json: {
      chat_id: $('Webhook').first().json.telegram,
      text: `🌟 *Simply News Update*\n\n${telegramMessage}`,
      parse_mode: 'Markdown'
    }
  };
}
```

### 10. **Programar Entregas Automáticas**
```javascript
// Nodo: Schedule Trigger
// Configurar según la frecuencia del cliente:
const frequency = $('Webhook').first().json.newsPreferences.frequency;

const schedules = {
  'hourly': '0 * * * *',
  'daily': '0 8 * * *',      // 8 AM diario
  'weekly': '0 8 * * 1',     // Lunes 8 AM
  'monthly': '0 8 1 * *'     // Primer día del mes 8 AM
};

return { json: { cron: schedules[frequency] || schedules['daily'] } };
```

## 🗄️ Base de Datos Recomendada

```sql
-- Tabla de clientes
CREATE TABLE clients (
  id SERIAL PRIMARY KEY,
  client_id VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(255) NOT NULL,
  name VARCHAR(255),
  telegram VARCHAR(100),
  news_preferences JSONB,
  location_data JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  last_sent TIMESTAMP,
  status VARCHAR(20) DEFAULT 'active',
  unsubscribe_token VARCHAR(100)
);

-- Tabla de entregas
CREATE TABLE deliveries (
  id SERIAL PRIMARY KEY,
  client_id VARCHAR(50) REFERENCES clients(client_id),
  delivered_at TIMESTAMP DEFAULT NOW(),
  article_count INTEGER,
  delivery_method VARCHAR(20), -- 'email', 'telegram'
  status VARCHAR(20) DEFAULT 'sent' -- 'sent', 'failed', 'bounced'
);

-- Índices para optimización
CREATE INDEX idx_clients_email ON clients(email);
CREATE INDEX idx_clients_status ON clients(status);
CREATE INDEX idx_deliveries_client_date ON deliveries(client_id, delivered_at);
```

## 🔄 Automatización Completa

### Workflow Principal:
1. **Trigger diario** → Obtener clientes activos
2. **Para cada cliente** → Ejecutar flujo personalizado
3. **Generar noticias** → Llamadas específicas a NewsAPI
4. **Procesar contenido** → Filtrar y personalizar
5. **Enviar newsletter** → Email + Telegram
6. **Registrar entrega** → Log en base de datos

### Configuración de Webhooks en n8n:
```
https://tu-n8n.com/webhook/simply-news-client-setup    // Setup inicial
https://tu-n8n.com/webhook/simply-news-daily           // Trigger diario
https://tu-n8n.com/webhook/simply-news-unsubscribe     // Cancelación
```

## 🎛️ Variables de Entorno Necesarias

```env
NEWSAPI_KEY=tu_clave_newsapi_aqui
EMAIL_HOST=smtp.gmail.com
EMAIL_USER=news@simplynews.com
EMAIL_PASS=tu_contraseña_email
TELEGRAM_BOT_TOKEN=tu_token_bot_telegram
DATABASE_URL=postgresql://user:pass@localhost/simplynews
WEBHOOK_SECRET=tu_secreto_webhook
```

## 🔧 Configuración de NewsAPI

### Límites de NewsAPI:
- **Developer Plan**: 1,000 requests/month
- **Business Plan**: 250,000 requests/month
- **Enterprise**: Sin límites

### Optimización de Llamadas:
1. **Cachear resultados** durante 1-2 horas
2. **Combinar múltiples requests** en uno solo cuando sea posible
3. **Usar parámetros eficientes**: `pageSize`, `sources`, `from`
4. **Implementar fallbacks** para cuando se alcancen límites

¡Con esta configuración tendrás un sistema completamente personalizado donde cada cliente recibe noticias específicas según sus preferencias! 🚀 