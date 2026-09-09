# SmartLogix — Paso a Paso: Levantar el Proyecto + Cómo se Incorpora la Seguridad

**Asignatura:** DSY1107 Desarrollo Cloud Native I
**Objetivo:** Entender el orden de arranque y cómo cada configuración de seguridad se activa conforme se levanta cada servicio.

---

## Panorama: el orden importa

La seguridad NO aparece "de golpe". Se va construyendo en capas mientras arranca el proyecto:

```
PASO 0   Docker (PostgreSQL + RabbitMQ + pgAdmin)   → infraestructura, sin seguridad
PASO 1   identity-service  (8084)                    → define CI quien puede autenticarse
PASO 2   inventory / order / shipping (8081/2/3)     → resource servers: validan el JWT
PASO 3   api-gateway (8080)                          → consolida: enruta + valida + CORS
PASO 4   frontend (5173)                             → SPA con login y Bearer token
```

El Gateway y los microservicios asumen que ya existe un "emisor" de tokens. Por eso **identity-service va primero** (y aunque en el .bat se lanza antes, cada uno es independiente).

---

## Prerrequisitos

- Java 21
- Maven Wrapper incluido (mvnw.cmd en cada servicio) — no requiere Maven global
- Node.js 22 (para frontend manual)
- Docker Desktop (para Postgres/RabbitMQ/pgAdmin)
- El repositorio clonado: `git clone https://github.com/felipeflores-m/SmartLogix.git`

---

## PASO 0 — Levantar la infraestructura (Docker)

Solo infraestructura: 4 bases PostgreSQL (una por servicio), RabbitMQ y pgAdmin. Aquí **no hay seguridad de la aplicación**; solo credenciales de base local.

```bash
# en la raíz del proyecto
docker compose up -d
```

Servicios que quedan arriba:

| Servicio | Puerto | BD/credenciales |
|---|---|---|
| postgres-identity | 5436 | smartlogix / smartlogix123, db: smartlogix_identity_db |
| postgres-inventory | 5433 | smartlogix / smartlogix123, db: smartlogix_inventory_db |
| postgres-orders | 5434 | smartlogix / smartlogix123, db: smartlogix_orders_db |
| postgres-shipping | 5435 | smartlogix / smartlogix123, db: smartlogix_shipping_db |
| rabbitmq | 5672 y 15672 (UI) | smartlogix / smartlogix123 |
| pgadmin | 5050 | admin@smartlogix.cl / admin123 |

> Los 5 microservicios Spring Boot se ejecutan manualmente (no containerizados). Docker solo aporta BD y mensajería. Esto queda claro en `docs/DOCKER.md`.

---

## PASO 1 — identity-service (puerto 8084): el "emisor" de identidad

Este es el paso donde **se define la seguridad de autenticación**.

### 1a. Arranque

```bash
cd identity-service
mvnw.cmd spring-boot:run
```

### 1b. Qué se genera al arrancar

1. **Se carga `application.properties`**:
   ```properties
   server.port=8084
   smartlogix.jwt.issuer=smartlogix-identity-service
   smartlogix.jwt.expiration-seconds=3600
   smartlogix.jwt.secret=smartlogix-academic-mvp-secret-key-change-later-minimum-64-characters
   ```
   → Acá se definen los **parámetros del token** (issuer, expiración, secreto).

2. **Spring Security construye los beans** (`SecurityConfig.java`):
   - `PasswordEncoder` (BCrypt) — para validar contraseñas
   - `JwtEncoder` — la "máquina" que firma tokens (HS256 con el secreto)
   - `JwtDecoder` — la "máquina" que verifica tokens
   - `jwtAuthenticationConverter` — traduce `role` (claim) → `ROLE_x` (autoridad)

3. **La cadena de filtros (`SecurityFilterChain`)** se activa: define qué rutas son públicas y cuáles exigen token/rol (login público, /users solo ADMIN, resto autenticado).

4. **`DataSeeder` corre su CommandLineRunner** (`DataSeeder.java:23-35`):
   - Crea los roles ADMIN, OPERATOR, VIEWER
   - Crea los usuarios seed con contraseña BCrypt (`passwordEncoder.encode`)
   - Usuarios: `admin@smartlogix.cl/admin123`, `operator@smartlogix.cl/operator123`, `viewer@smartlogix.cl/viewer123`

### 1c. Seguridad incorporada en este paso

- ✅ Es el único servicio capaz de **emitir** tokens (login → JWT)
- ✅ Define **roles** y **usuarios seed**
- ✅ Protege `/api/users` con `hasRole("ADMIN")`
- ❌ Todavía NO hay gateway ni otros servicios validando — eso viene después

---

## PASO 2 — inventory / order / shipping (8081, 8082, 8083): los Resource Servers

Cada servicio de negocio **valida el JWT de forma independiente** (no confía ciegamente en nadie). Al arrancar hacen lo mismo que identity pero sin emitir tokens.

### 2a. Arranque (3 terminales separadas)

```bash
cd inventory-service && mvnw.cmd spring-boot:run   # 8081
cd order-service     && mvnw.cmd spring-boot:run   # 8082
cd shipping-service  && mvnw.cmd spring-boot:run   # 8083
```

### 2b. Qué se carga en cada uno

Cada uno tiene su `SecurityConfig.java` (idéntico en los 3), su `application.properties` y su propio `DataSeeder` (con datos de negocio: productos, clientes, transportistas).

- **El mismo secreto HS256 compartido** (`smartlogix.jwt.secret`) en su `application.properties` → pueden verificar cualquier token que firme identity-service.
- `JwtDecoder` HS256 → valida firma del token en cada request.
- Reglas por rol (p. ej. inventory):
  ```java
  .requestMatchers(HttpMethod.GET, "/api/inventory/**")
  .hasAnyRole("ADMIN", "OPERATOR", "VIEWER")
  .requestMatchers("/api/inventory/**")
  .hasAnyRole("ADMIN", "OPERATOR")
  ```

### 2c. Seguridad incorporada en este paso

- ✅ Cada microservicio **valida el JWT** (autenticación) por su cuenta
- ✅ Aplica **autorización por rol** sobre sus endpoints
- ✅ Expone Swagger/health públicos
- ❌ Aún no hay punto único de entrada (el frontend podría llamarlos directo)

---

## PASO 3 — api-gateway (8080): la consolidación de seguridad

El gateway es donde **se junta todo**: enruta, valida y aplica CORS. Es el último Spring Boot en arrancar porque necesita que los servicios destino existan para reenviar.

### 3a. Arranque

```bash
cd api-gateway
mvnw.cmd spring-boot:run   # 8080
```

### 3b. Qué se carga

1. **Las rutas** (`application.properties:12-30`): mapea cada `/api/*` a su puerto interno.
2. **`CorsConfig.java`**: define orígenes, métodos y headers permitidos.
3. **`SecurityConfig.java` del gateway**: valida JWT y aplica RBAC **antes** de reenviar:
   - `/api/auth/login` público (para que el frontend pueda obtener el token)
   - GET de negocio → cualquier rol
   - Mutaciones → solo ADMIN/OPERATOR
   - Requiere el claim `role` obligatorio (lanza `JwtException` si falta)
4. **Filtros extra**: `CorrelationIdFilter` agrega `X-Correlation-Id` a cada request (trazabilidad).
5. **Health agregado** `/api/system/health` consulta los 4 servicios.

### 3c. Seguridad incorporada en este paso

- ✅ **Un solo punto de entrada** (concepto API Manager)
- ✅ Segunda capa de validación JWT (además de la de cada servicio)
- ✅ **CORS centralizado** (el navegador del frontend puede llamar)
- ✅ RBAC consolidado por verbo HTTP
- ✅ Trazabilidad

---

## PASO 4 — frontend React (5173): consume a través del gateway con el Bearer token

```bash
cd frontend
npm install
npm run dev            # 5173 (producción: docker compose up --build lo sirve en :5173)
```

Seguridad en el frontend:
- **Login con formulario** → `POST /api/auth/login` (a través del gateway en :8080)
- Guarda el token en `sessionStorage` (`authTokenProvider.ts`, clave `smartlogix.accessToken`)
- **HTTP client** (`httpClient.ts:97-99`) agrega automáticamente:
  ```ts
  requestHeaders.set("Authorization", `Bearer ${token}`);
  ```
- Maneja 401 → limpia sesión y redirige a /login
- **`ProtectedRoute`** protege rutas; **permisos por rol** (`roleAccess.ts`, 34 permisos) ocultan UI según rol

---

## Resumen: la línea de tiempo de la seguridad

| Orden | Servicio | Qué aporta a la seguridad | Concepto de la evaluación |
|---|---|---|---|
| 0 | Docker | (ninguno, solo datos) | Infraestructura |
| 1 | identity-service | Emite JWT, define roles/usuarios | JWT, Claims, RBAC |
| 2 | inventory/order/shipping | Validan JWT en cada request | Resource Server, autorización |
| 3 | api-gateway | Entrada única, valida, enruta, CORS | API Manager, CORS, validación JWT |
| 4 | frontend | Envía Bearer, protege rutas/UI | Cliente SPA, tokens |

---

## Demostración de la progresión (para la defensa)

Puedes mostrar cómo la seguridad NO existía y luego va apareciendo:

1. **Con solo Docker**: nada de APIs → nada que proteger.
2. **Con solo identity**: puedes hacer login y obtener JWT, pero el resto del negocio no existe.
3. **Con los 3 servicios (sin gateway)**: podrías llamar a `localhost:8081/api/inventory/products` **directo** con tu token... y CORS lo bloquea desde el navegador (regresa a Same-Origin). Se confirma la necesidad del gateway.
4. **Con el gateway**: todo pasa por `localhost:8080`; CORS funciona; sin token → 401; con token y rol equivocado → 403.
5. **Con frontend**: login visual, token en memoria, UI adaptada por rol.

---

## Notas de operación

- Detener: `CTRL+C` en cada ventana CMD del microservicio.
- Parar infraestructura: `docker compose down` (mantiene datos) o `docker compose down -v` (borra volúmenes).
- Script automático: `start-smartlogix.bat` abre las 5 ventanas (valida carpetas y lanza en orden identity → gateway → inventory → order → shipping).
- Ver rutas del gateway: `GET http://localhost:8080/actuator/gateway/routes` (público).
- Health agregado: `GET http://localhost:8080/api/system/health` (público).

---

## Referencia de archivos clave

- `start-smartlogix.bat` — lanzador automático (identifica el orden y los puertos)
- `docker-compose.yml` — infraestructura (Postgres x4, RabbitMQ, pgAdmin, frontend)
- `identity-service/.../config/SecurityConfig.java` — encoder/decoder JWT + autorización
- `identity-service/.../security/JwtTokenService.java` — emisión del token (claims)
- `identity-service/.../config/DataSeeder.java` — usuarios/roles semilla
- `api-gateway/.../config/CorsConfig.java` — CORS
- `api-gateway/src/main/resources/application.properties` — rutas de enrutamiento
- `api-gateway/.../config/SecurityConfig.java` — validación JWT en el gateway
- `frontend/src/lib/security/httpClient.ts` — Bearer token en peticiones