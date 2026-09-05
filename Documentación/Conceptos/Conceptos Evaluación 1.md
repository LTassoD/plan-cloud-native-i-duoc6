# Conceptos Clave — Evaluación 1

## API Manager / API Gateway

Punto de entrada central entre aplicaciones (frontend, móvil, terceros) y servicios backend. Todo petición pasa por él.

**Funciones principales:**
- Centralizar seguridad: validar roles, claves de API, tokens OAuth2
- Gestionar ciclo de vida: diseño → publicación → versionamiento → deprecación → retiro
- Desacoplar cliente del backend
- Monitoreo y analítica: tráfico, errores, consumo por versión

## Azure API Management (APIM)

Equivalente de AWS API Gateway. Servicio gestionado con 3 componentes:

| Componente | Función |
|-----------|---------|
| **Gateway** | Recibe peticiones, aplica políticas, enruta al backend |
| **Developer Portal** | Portal interactivo para descubrir, probar y consumir APIs |
| **Policies** | Configuración XML que aplica reglas en el flujo de la petición |

## CORS (Cross-Origin Resource Sharing)

Mecanismo de seguridad HTTP basado en cabeceras.

- **Same-Origin Policy (SOP)**: bloquea peticiones entre dominios distintos
- **CORS relaja** esa restricción de forma controlada

**Cabeceras clave:**
- `Access-Control-Allow-Origin`: qué dominios pueden acceder
- `Access-Control-Allow-Methods`: verbos permitidos (GET, POST, PUT, DELETE, OPTIONS)
- `Access-Control-Allow-Headers`: headers permitidos (`Content-Type`, `Authorization`)

**Tipos de peticiones:**
- **Simple**: envía cabecera `Origin`, espera `Access-Control-Allow-Origin` coincidente
- **Preflight**: peticiones complejas (PUT/DELETE, headers personalizados) → primero envían `OPTIONS` automático

## Versionamiento de APIs

Las APIs cambian, pero no puedes romper a tus clientes. Versionar = señalizar (`/v1`, `/v2`).

- **SemVer** (semver.org): `MAJOR.MINOR.PATCH`
- En el Gateway es el único lugar donde puedes versionar sin romper otros sistemas

**Proceso de deprecación:**
1. Aviso oficial: headers `Deprecation: true` y `Sunset: fecha`
2. Comunicación a integradores
3. Coexistencia de versiones (`/v1` y `/v2` activas)
4. Monitoreo de uso residual
5. Apagado controlado

## Stack — Azure

| Servicio Azure | Función en el semestre |
|---------------|----------------------|
| Azure API Management | API Gateway (rutas, CORS, versionamiento, políticas) |
| Azure AD / AD B2C | Autenticación centralizada (OAuth2, OpenID Connect) |
| Azure App Service | Hosting de microservicios Spring Boot |
| Azure Cosmos DB | Base de datos NoSQL |
| Azure Service Bus | Mensajería (equivalente a RabbitMQ) |
| Azure Event Hubs | Eventos (equivalente a Apache Kafka) |
| Azure Container Instances | Despliegue de contenedores Docker |
| Azure DevOps / GitHub | CI/CD |

## Guías de referencia

- [Azure API Management docs](https://learn.microsoft.com/azure/api-management/)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [IETF — HTTP Deprecation Header (RFC 8594)](https://www.rfc-editor.org/rfc/rfc8594)
