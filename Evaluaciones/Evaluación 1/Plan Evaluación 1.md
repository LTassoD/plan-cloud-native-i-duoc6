# Evaluación Parcial 1 — DSY1107 Desarrollo Cloud Native I

**Estudiante:** Luis Tasso
**Proyecto:** Eco-Mantenimiento (Sistema de gestión de mantenimiento de flota)
**Modalidad de trabajo:** Individual

---

## 1. Enunciado de la evaluación

La evaluación parcial 1 consiste en:

1. **Explicar los conceptos** de las guías del bloque 1 (API Manager, CORS, versionamiento, OAuth2/OIDC, JWT y Claims, IDaaS/CIAM).
2. **Demostrar en un proyecto propio** la configuración de los tokens (OAuth2/OIDC + JWT) usando Identity as a Service.

**Entregables:**
- Informe escrito
- Presentación oral con demostración

---

## 2. Stack tecnológico de la solución

| Capa | Tecnología | Rol |
|------|-----------|-----|
| **Frontend** | React + TypeScript + Vite | Interfaz de usuario |
| **Backend** | Spring Boot 3.2.5 (Java 17) | API REST del dominio |
| **Base de datos** | H2 (en memoria) | Datos de ejemplo |
| **Identity Provider (IDaaS)** | Azure Active Directory B2C (ecomantenimiento.onmicrosoft.com) | Autenticación OAuth2/OIDC, emisión de tokens |
| **API Gateway** | Azure API Management | Exposición y protección de la API |
| **Validación JWT** | Spring Security + JWT / Azure AD B2C | Validación de tokens en el backend |

---

## 3. Arquitectura de la solución

```
┌────────────────────┐
│      USUARIO       │
│  (navegador)       │
└─────────┬──────────┘
          │ 1. Inicia sesión
          ▼
┌────────────────────┐          ┌──────────────────────────────┐
│ Frontend React     │ ───────▶ │ Azure AD B2C (IDaaS)          │
│ (Eco-Mantenimiento)│          │ • Authorization Code + PKCE  │
└─────────┬──────────┘          │ • Emite access_token +      │
          │                     │   id_token (JWT)             │
          │ 2. Token obtenido   └──────────────────────────────┘
          │
          ▼
┌────────────────────┐
│ Azure API Management│ ── 3. Valida el JWT ──▶ 4. Enruta
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│ Backend Spring Boot │
│ (local) + H2        │
│ • /api/trucks       │
│ • /api/maintenance  │
└────────────────────┘
```

---

## 4. Plan de trabajo

### Fase 1: Preparación de Azure (Cloud)
- [x] Crear tenant Azure AD B2C (`ecomantenimiento.onmicrosoft.com`)
- [x] Crear user flow (signup-signin) — `B2C_1_signup-signin`
- [x] Registrar aplicación en Azure AD B2C (eco-mantenimiento-app)
- [x] Configurar scopes y endpoints
- [x] Crear Azure API Management (`eco-apim`, tier Consumption)
- [x] Crear API **eco-api** en APIM (service-url: `https://mindicador.cl/api`, path `eco`)
- [x] Crear operación GET `get-indicadores`
- [x] Crear version set `eco-versions` (esquema Segment)
- [ ] **Portal**: asignar la API a la versión `v1` (y crear `v2`) dentro del version set
- [ ] **Portal**: aplicar política CORS a nivel de API (permitir `http://localhost:5173`)
- [ ] Probar gateway URL (`https://eco-apim.azure-api.net/eco/...`)

### Fase 2: Backend (Spring Boot)
- [x] Agregar dependencia spring-security (OAuth2 resource server)
- [x] Configurar validación de JWT (issuer, audience, JWKS)
- [x] Proteger los endpoints `/api/**`
- [x] Verificar tokens con Spring Security
- [ ] Compilar y levantar backend con Eclipse (Gradle local)

### Fase 3: Frontend (React)
- [x] Integrar librería MSAL (`@azure/msal-browser@5`, `@azure/msal-react@5`)
- [x] Reemplazar el AuthContext demo por el flujo OAuth2/OIDC con MSAL
- [x] Adjuntar `Authorization: Bearer <token>` a las peticiones (interceptor axios)
- [x] Configurar rutas protegidas (`ProtectedRoute` + `MsalProvider`)

### Fase 4: Integración y demostración
- [ ] Probar flujo completo (login → token → petición autenticada)
- [ ] Capturar evidencia (Postman, jwt.io, pantallas en vivo)
- [ ] Versionar API (v1/v2)

### Fase 5: Documentación
- [ ] Redactar informe escrito
- [ ] Preparar presentación oral
- [ ] Enlistar conceptos explicados

---

## 5. Conceptos que debo explicar en la evaluación

### Bloque 1.1 — API Management
- Qué es un API Manager / API Gateway (punto de entrada central)
- Problemas que resuelve (integración, seguridad, carga operativa, rendimiento)
- Tipos de APIS (REST, HTTP, WebSocket)
- Versionamiento de APIs (SemVer, coexistencia, deprecación)
- CORS (Same-Origin Policy, Simple vs Preflight)

### Bloque 1.2 — Seguridad e Identidad
- Identity as a Service (IDaaS) y CIAM
- OAuth2 vs OpenID Connect (autenticación vs autorización)
- Los 4 actores de OAuth2 (Resource Owner, Client, Authorization Server, Resource Server)
- Flujo Authorization Code + PKCE
- Tokens: Access Token, ID Token, Refresh Token
- JWT (estructura Header.Payload.Signature)
- Claims (iss, sub, aud, exp, iat)
- Scopes (openid, email, profile)
- Endpoints (/authorize, /token, /userinfo, /jwks)
- Validación de JWT en el API Gateway

---

## 6. Entidades del dominio Eco-Mantenimiento (para la demo)

| Entidad | Controller | Descripción |
|---------|-----------|-------------|
| Truck | TruckController | Camiones de la flota |
| Driver | DriverController | Conductores |
| MaintenanceService | MaintenanceServiceController | Servicios de mantenimiento |
| Checklist | ChecklistController | Check-list de inspección |
| MaintenanceSchedule | MaintenanceScheduleController | Programación de mantenimiento |

La demo usará estos endpoints protegidos por token: `/api/trucks`, `/api/maintenance-services`, `/api/checklists`, etc.

---

## 7. Referencias (guías)

| Guía | Concepto |
|------|----------|
| 1.1.1 | Qué es un API Manager |
| 1.1.2 | Crear un API Manager (Azure APIM) |
| 1.1.3 | Versionar APIs |
| 1.1.4 | CORS en el API Gateway |
| 1.2.1 | OAuth2 y OIDC |
| 1.2.2 | IDaaS y CIAM |
| 1.2.3 | Configurar un Tenant |
| 1.2.4 | Configurar apps en un IDaaS |
| 1.2.7 | JWT y Claims |
| 1.2.8 | Decodificar tokens JWT |
