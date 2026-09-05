# Arquitectura General — Semestre

## Diagrama de alto nivel

```
┌─────────────────────────────────────────────────────────┐
│                      CLIENTES                           │
│           (Web, Móvil, Terceros, Postman)               │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTPS
                       ▼
┌─────────────────────────────────────────────────────────┐
│              AZURE API MANAGEMENT (Gateway)             │
│                                                         │
│  • CORS Policy (Access-Control-Allow-Origin)            │
│  • Rate Limiting                                        │
│  • Autenticación (Azure AD B2C / OAuth2 / JWT)         │
│  • Rutas: /v1/datos, /v2/datos, /v1/usuarios, etc.    │
│  • Versionamiento (v1 → v2 con deprecación controlada) │
│  • Logging & Analytics                                  │
└───────────┬──────────────────────┬──────────────────────┘
            │                      │
            ▼                      ▼
┌───────────────────┐  ┌──────────────────────────────────┐
│  API Externa      │  │    BACKEND (Azure)                │
│  mindicador.cl    │  │                                  │
│  /api             │  │  ┌────────────────────────────┐  │
│                   │  │  │ Azure App Service           │  │
│  (indicadores     │  │  │ Spring Boot Microservicios  │  │
│   económicos)     │  │  └───────────┬────────────────┘  │
│                   │  │              │                    │
│                   │  │              ▼                    │
│                   │  │  ┌────────────────────────────┐  │
│                   │  │  │ Azure Cosmos DB             │  │
│                   │  │  │ (Base de datos)             │  │
│                   │  │  └────────────────────────────┘  │
│                   │  │                                  │
│                   │  │  ┌────────────────────────────┐  │
│                   │  │  │ Azure Service Bus           │  │
│                   │  │  │ (Mensajería async)          │  │
│                   │  │  └────────────────────────────┘  │
│                   │  │                                  │
│                   │  │  ┌────────────────────────────┐  │
│                   │  │  │ Azure Event Hubs            │  │
│                   │  │  │ (Eventos Kafka-compatible)  │  │
│                   │  │  └────────────────────────────┘  │
│                   │  └──────────────────────────────────┘
└───────────────────┘
```

## Evaluación 1 — Alcance

Solo se implementa la capa de **API Management** + **CORS** + **Versionamiento**:

```
┌──────────────┐      ┌──────────────────────┐      ┌────────────────┐
│   Postman    │─────▶│  Azure API Management │─────▶│ mindicador.cl  │
│  (pruebas)   │      │  HTTP API + CORS      │      │    /api        │
└──────────────┘      │  + Versiones (v1/v2)  │      └────────────────┘
                      └──────────────────────┘
```

## Evaluaciones posteriores

| Evaluación | Servicios Azure | Contenido |
|-----------|----------------|-----------|
| **Eval 2** | Azure AD B2C + API Management policies | OAuth2, OIDC, JWT, autenticación centralizada |
| **Eval 3** | Azure App Service + Cosmos DB + Service Bus + Event Hubs | Microservicios Spring Boot, FullStack, mensajería async |
| **Proyecto final** | Todos integrados | Sistema completo Cloud Native |

## Crédito Azure

- **Azure for Students**: $100 USD de crédito
- **API Management Consumption**: ~$0.04 / 10,000 llamadas
- **App Service Basic B1**: ~$13/mes
- **Service Bus Basic**: 1M ops gratis
- **Event Hubs Basic**: 1M eventos/día gratis
- **Estimated total semestre**: ~$50-115 (dentro del crédito)
