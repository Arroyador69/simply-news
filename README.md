# Simply News - AI-Powered News & Personalized Delivery

Una aplicación web moderna que combina noticias personalizadas con un chatbot inteligente y un sistema de entrega automatizada de noticias.

## 🚀 Características

- **Diseño Moderno**: Interfaz elegante con gradientes y efectos visuales
- **Responsive**: Se adapta perfectamente a todos los dispositivos
- **Chatbot Integrado**: Funcionalidad de chat inteligente
- **Noticias Personalizadas**: Sistema de suscripción personalizada por país, idioma y temas
- **Entrega Automatizada**: Integración con n8n para envío automático de noticias
- **Cobertura Global**: Soporte para más de 50 países con fuentes locales
- **Multi-idioma**: Fuentes de noticias en múltiples idiomas según el país
- **UX Optimizada**: Experiencia de usuario fluida e intuitiva

## 📋 Formulario de Suscripción Personalizada

El sistema incluye un formulario avanzado (`survey.html`) que permite a los usuarios:

- **Información Personal**: Nombre, email y usuario de Telegram
- **Selección de País**: Más de 50 países con banderas (soporte completo de NewsAPI)
- **Idioma Dinámico**: Selección automática de idiomas disponibles por país
- **Fuentes de Noticias**: Selección específica de fuentes locales por país/idioma
- **Temas de Interés**: 
  - Business & Finance
  - Entertainment
  - General News
  - Health & Medical
  - Science & Technology
  - Sports
  - Technology
  - Cybersecurity
  - Geopolitics
  - Artificial Intelligence
- **Frecuencia**: Personalización de horarios de entrega
- **Zona Horaria**: Configuración automática según ubicación

## 🔧 Integración con n8n

El sistema está diseñado para integrarse con workflows de n8n para:

- **Procesamiento Automático**: Los datos del formulario se envían vía webhook
- **Entrega Programada**: Noticias enviadas según preferencias de horario
- **Filtrado Inteligente**: Contenido filtrado por temas y fuentes seleccionadas
- **Multi-canal**: Envío por email y Telegram
- **Personalización**: Contenido adaptado a intereses específicos

Ver `n8n-workflow-guide.md` para instrucciones detalladas de configuración.

## 🛠️ Tecnologías

- HTML5
- CSS3 (con animaciones y efectos modernos)
- JavaScript (manejo dinámico de formularios)
- NewsAPI (fuentes de noticias globales)
- n8n (automatización de workflows)
- Diseño responsive

## 📱 Estructura del Proyecto

```
simply-news/
├── index.html              # Página principal con chatbot
├── survey.html             # Formulario de suscripción personalizada
├── ejemplo-datos-cliente.json  # Ejemplo de datos enviados por el formulario
├── n8n-workflow-guide.md   # Guía de configuración de n8n
└── README.md               # Este archivo
```

## 🚀 Cómo usar

1. **Página Principal**: Abre `index.html` para la experiencia de chatbot
2. **Suscripción**: Visita `survey.html` para configurar noticias personalizadas
3. **Configuración n8n**: Sigue la guía en `n8n-workflow-guide.md`
4. **Webhook**: Configura el endpoint para recibir datos del formulario

## 🌍 Países y Fuentes Soportadas

El sistema soporta fuentes de noticias específicas para más de 50 países, incluyendo:
- Estados Unidos, Reino Unido, Canadá
- España, Francia, Alemania, Italia
- Brasil, Argentina, México
- Japón, Corea del Sur, India
- Australia, Sudáfrica
- Y muchos más...

Cada país incluye fuentes locales en idiomas nativos cuando están disponibles.

## 🌐 Demo Online

Visita la versión online en: [GitHub Pages](https://arroyador69.github.io/simply-news)

## 📄 Licencia

Este proyecto está bajo la Licencia MIT.

---

Desarrollado con ❤️ para una experiencia personalizada de noticias globales. 