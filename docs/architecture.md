# Arquitectura del NOC Bot

## Visión general

El NOC Bot es un workflow de n8n que opera en modo polling: se dispara por schedule, lee alertas de Slack, las enriquece con datos de observabilidad y responde en el hilo usando análisis de Claude AI.

---

## Diagrama de flujo

```
┌─────────────────────┐
│   Schedule Trigger  │  (ejecución periódica)
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Get Channel History│  Slack: obtiene mensajes del canal NOC
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Formatear Mensaje  │  Normaliza los datos del mensaje
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│   Tiene contenido?  │◄── NO → (fin, sin alertas)
└────────┬────────────┘
         │ SÍ
         ▼
┌─────────────────────┐
│   Parsear Alerta    │  Extrae campos estructurados de la alerta
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│   Debe responder?   │◄── NO → (fin, ya fue respondida)
└────────┬────────────┘
         │ SÍ
         │
         ├──────────────────────────────────────┐
         │                                      │
         ▼                                      ▼
┌─────────────────┐                   ┌──────────────────────┐
│ DD: Metrics 30m │                   │ Slack: Correlación 1h│
│ DD: Logs 15min  │                   │ AWS Health (opcional) │
│ DD: Events 1h   │                   └──────────┬───────────┘
└────────┬────────┘                              │
         │                                       │
         └───────────────┬───────────────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │ Consolidar          │  Merge de todos los datos
              │ Enriquecimiento     │
              └────────┬────────────┘
                       │
                       ▼
              ┌─────────────────────┐
              │ Procesar            │  Formatea contexto para la IA
              │ Enriquecimiento     │
              └────────┬────────────┘
                       │
                       ▼
              ┌─────────────────────┐
              │ Claude: Análisis IA │  HTTP → Anthropic API
              └────────┬────────────┘
                       │
                       ▼
              ┌─────────────────────┐
              │ Adjuntar respuesta  │  Prepara el mensaje final
              │ IA                  │
              └────────┬────────────┘
                       │
                       ▼
              ┌─────────────────────┐
              │ Construir Mensaje   │  Formatea bloques de Slack
              │ Final               │
              └────────┬────────────┘
                       │
                       ├───────────────────────┐
                       │                       │
                       ▼                       ▼
              ┌────────────────┐    ┌──────────────────────┐
              │ Responder en   │    │ Registrar en          │
              │ hilo (Slack)   │    │ Google Sheets         │
              └────────────────┘    └──────────────────────┘
```

---

## Nodos del workflow

| Nodo | Tipo | Descripción |
|---|---|---|
| Schedule Trigger | Trigger | Dispara el workflow periódicamente |
| Get Channel History | Slack | Lee mensajes del canal NOC |
| Formatear Mensaje | Set | Normaliza campos del mensaje |
| Tiene contenido | IF | Verifica si hay mensajes para procesar |
| Parsear Alerta | Code | Extrae campos estructurados (servicio, severidad, etc.) |
| Debe responder | IF | Evita responder a alertas ya respondidas |
| DD: Metrics (30min) | HTTP Request | GET métricas de Datadog |
| DD: Logs (15min) | HTTP Request | GET logs recientes de Datadog |
| DD: Events (1h) | HTTP Request | GET eventos de Datadog |
| Slack: Correlación (1h) | Slack | Busca mensajes relacionados en Slack |
| AWS Health (opcional) | HTTP Request | Consulta AWS Health API |
| Consolidar enriquecimiento | Set | Combina todos los datos de contexto |
| Procesar Enriquecimiento | Code | Formatea el prompt para Claude |
| Claude: Análisis IA | HTTP Request | POST a la API de Anthropic |
| Adjuntar respuesta IA | Set | Adjunta el análisis al objeto de alerta |
| Construir Mensaje Final | Code | Construye los bloques de Slack |
| Responder en hilo | Slack | Publica la respuesta en el hilo |
| Registrar en Sheets | Google Sheets | Guarda el registro de la alerta |
| Merge | Merge | Sincroniza ramas paralelas |

---

## Fuentes de datos para enriquecimiento

### Datadog
- **Métricas**: ventana de 30 minutos centrada en el momento de la alerta
- **Logs**: últimos 15 minutos, filtrados por servicio afectado
- **Eventos**: última hora, para detectar deploys o cambios de configuración

### Slack
- Mensajes correlacionados en la última hora en el canal NOC

### AWS Health *(opcional)*
- Estado de servicios de AWS que puedan afectar la infraestructura

---

## Salida

El bot responde en el **hilo** de la alerta original con:
- Diagnóstico del problema
- Métricas y logs relevantes
- Posibles causas raíz
- Recomendación de acción

Adicionalmente, registra cada alerta procesada en **Google Sheets** con:
- Timestamp
- Servicio afectado
- Severidad
- Resumen del análisis de Claude
