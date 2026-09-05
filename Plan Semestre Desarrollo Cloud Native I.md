# Plan del Semestre — Desarrollo Cloud Native I

**Escuela de Informática y Telecomunicaciones · Duoc UC · Presencial**
**Sigla: DSY1107 · Total: 90 horas (referencial) · Caso práctico en Azure (API Management)**

> Plan tentativo de 16 semanas. Basado en los materiales disponibles en la carpeta (actividades 1.1.x del bloque 1).
> Plataforma: **Azure for Students** ($100 USD crédito) · Tenant: **duoc.cl**

---

## 1. Stack tecnológico

| Servicio Azure | Función |
|---------------|---------|
| Azure API Management | API Gateway (rutas, CORS, versionamiento, políticas) |
| Azure AD / AD B2C | Autenticación centralizada (OAuth2, OpenID Connect) |
| Azure App Service | Hosting de microservicios Spring Boot |
| Azure Cosmos DB | Base de datos NoSQL |
| Azure Service Bus | Mensajería (equivalente a RabbitMQ) |
| Azure Event Hubs | Eventos Kafka-compatible (equivalente a Apache Kafka) |
| Azure Container Instances | Despliegue de contenedores Docker |
| Azure DevOps / GitHub | CI/CD |

---

## 2. Estructura de evaluaciones y ponderación

⚠️ **Pendiente de confirmar en AVA.** Se espera el esquema estándar Duoc: parciales (60%) + Evaluación Final Transversal (40%), con un caso práctico cloud. Anota aquí las ponderaciones reales:

| Evaluación | Tipo | Peso | Entrega |
|---|---|---|---|
| Ev Parcial 1 · ____ | ____ | __% | ____ |
| Ev Parcial 2 · ____ | ____ | __% | ____ |
| Ev Parcial 3 · ____ | ____ | __% | ____ |
| **Evaluación Final Transversal** | ____ | **40%** | ____ |

**Regla de cálculo (referencial):** Nota final = (Parciales × 60%) + (Final Transversal × 40%)

---

## 3. Ruta de aprendizaje y cronograma

### Bloque 1 — EA1: API Management y exponer APIs seguras (semanas 1–4 · materiales disponibles)

| Semana | Actividad | Descripción / Entregable |
|---|---|---|
| 1 | Act 1.1.1 · Conociendo un API Manager | Qué es un API Manager, API Gateway, tipos de API (REST, WebSocket), casos de uso, portal para desarrolladores |
| 2 | Act 1.1.2 · Tutorial: Creando Nuestro Primer API Manager | **Práctica Azure:** crear recurso Azure API Management, ruta GET /datos, integración HTTP con mindicador.cl/api. Capturas de pantalla |
| 3 | Act 1.1.3 · Versionando APIs | Versionamiento semántico, versiones v1/v2 en Azure API Management, proceso de deprecación con headers |
| 4 | Act 1.1.4 · Configurando CORS en el API Gateway | CORS policy XML en Azure API Management, Simple vs Preflight, despliegue y pruebas |

**📌 Ev Parcial 1** → probablemente cierra este bloque (confirmar en AVA).

### Bloques 2 y 3 — Pendientes de materiales (semanas 5–15)

> ⚠️ Los materiales de las actividades 1.2.x en adelante aún no están en la carpeta. Temas probables según el enfoque de la asignatura (confirmar con el docente/AVA):
> - **Microservicios y contenedores** (Docker, Azure App Service, despliegue)
> - **Seguridad e identidad** (Azure AD B2C, OAuth2, OpenID Connect)
> - **Mensajería async** (Azure Service Bus, Azure Event Hubs)
> - **Despliegue y observabilidad** en Azure

### Cierre — Semana 16

| Semana | Actividad | Horas |
|---|---|---|
| 16 | **Evaluación Final Transversal (40%)** | ____ |

---

## 4. Checklist de seguimiento (marca tu avance)

- [ ] Act 1.1.1 · Conociendo un API Manager — fecha: ____
- [ ] Act 1.1.2 · Tutorial: Creando mi Primer API Manager — fecha: ____
- [ ] Act 1.1.3 · Versionando APIs — fecha: ____
- [ ] Act 1.1.4 · Configurando CORS en el API Gateway — fecha: ____
- [ ] **Ev Parcial 1** — fecha: ____
- [ ] Bloque 2 · actividades por confirmar — fecha: ____
- [ ] **Ev Parcial 2** — fecha: ____
- [ ] Bloque 3 · actividades por confirmar — fecha: ____
- [ ] **Ev Parcial 3** — fecha: ____
- [ ] **Evaluación Final Transversal (40%)** — fecha: ____

---

## 5. Referencias

1. [Azure API Management docs](https://learn.microsoft.com/azure/api-management/)
2. [Semantic Versioning 2.0.0](https://semver.org/)
3. [IETF — HTTP Deprecation Header (RFC 8594)](https://www.rfc-editor.org/rfc/rfc8594)
4. [Azure for Students](https://azure.microsoft.com/es-es/free-students/)
