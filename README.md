# NOC Bot

Un bot de respuesta automática para el equipo de operaciones (NOC) construido en n8n. Monitorea un canal de Slack en busca de alertas, las enriquece con datos de Datadog y genera análisis usando Claude AI, respondiendo directamente en el hilo de la alerta.

---

## ¿Qué hace?

El bot ejecuta el siguiente flujo de forma programada:

1. **Lectura de Slack** — Lee el historial del canal NOC buscando mensajes no respondidos.
2. **Parseo de alertas** — Identifica y estructura las alertas activas.
3. **Enriquecimiento con Datadog** — Consulta en paralelo:
   - Métricas de los últimos 30 minutos
   - Logs de los últimos 15 minutos
   - Eventos de la última hora
4. **Correlación en Slack** — Busca mensajes relacionados en la última hora.
5. **AWS Health** *(opcional)* — Consulta el estado de servicios AWS.
6. **Análisis con Claude AI** — Envía todo el contexto a Claude para generar un diagnóstico y recomendación.
7. **Respuesta en hilo** — Publica el análisis en el hilo de la alerta original en Slack.
8. **Registro en Google Sheets** — Guarda cada alerta procesada para auditoría y métricas.

---

## Arquitectura

Ver [`docs/architecture.md`](docs/architecture.md) para el diagrama detallado del flujo.

---

## Instalación

### Requisitos previos

- [n8n](https://n8n.io/) (self-hosted o cloud)
- Cuenta de Datadog con acceso a la API
- Bot de Slack con permisos de lectura/escritura en el canal NOC
- Google Sheets API habilitada
- API key de Anthropic (Claude)

### Pasos

1. Clona este repositorio:
   ```bash
   git clone https://github.com/juansandoval-arch/noc-bot.git
   cd noc-bot
   ```

2. Copia el archivo de variables de entorno y complétalo:
   ```bash
   cp .env.example .env
   # Edita .env con tus credenciales reales
   ```

3. Importa el workflow en n8n:
   - Ve a **Workflows → Import from file**
   - Selecciona `workflows/NOC_bot.json`

4. Configura las credenciales en n8n:
   - **Slack**: OAuth2 o Bot Token con scopes `channels:history`, `chat:write`
   - **Datadog**: API Key + Application Key
   - **Google Sheets**: Service Account o OAuth2
   - **Anthropic (Claude)**: API Key via HTTP Header Auth

5. Activa el workflow.

---

## Variables de entorno

| Variable | Descripción |
|---|---|
| `ANTHROPIC_API_KEY` | API key de Anthropic para Claude |
| `DD_API_KEY` | API key de Datadog |
| `DD_APP_KEY` | Application key de Datadog |
| `DD_SITE` | Sitio de Datadog (ej. `datadoghq.com`) |
| `SLACK_CHANNEL_ID` | ID del canal Slack que monitorea el bot |
| `GOOGLE_SHEET_ID` | ID de la hoja de cálculo para el registro de alertas |

---

## Estructura del repositorio

```
noc-bot/
├── workflows/
│   └── NOC_bot.json      # Workflow de n8n (importar directamente)
├── docs/
│   └── architecture.md   # Diagrama y descripción de la arquitectura
├── .env.example          # Plantilla de variables de entorno
├── .gitignore
└── README.md
```
