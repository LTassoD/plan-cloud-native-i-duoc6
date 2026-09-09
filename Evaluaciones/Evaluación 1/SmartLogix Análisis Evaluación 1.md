# SmartLogix — Análisis Técnico para Evaluación Parcial 1

**Asignatura:** DSY1107 Desarrollo Cloud Native I
**Proyecto referencial:** SmartLogix (github.com/felipeflores-m/SmartLogix)
**Enfoque:** API Gateway, JWT y Claims, CORS, versionado, OAuth2/OIDC

---

## 1. Qué es SmartLogix

Sistema académico de gestión logística (inventario, pedidos, envíos, transportistas) organizado como:
- **Frontend:** React + TypeScript + Vite + Tailwind
- **5 microservicios Spring Boot 4 / Java 21**
- **API Gateway:** entrada única del frontend
- **Infra Docker:** 4 x PostgreSQL + RabbitMQ + pgAdmin

### Arquitectura

```
Frontend React (5173)
      │
      ▼  (nunca llama directo a servicios internos)
API Gateway (8080)  ← entrada única, valida JWT, enruta
      │
      ├──► identity-service  (8084): login, emite JWT, usuarios
      ├──► inventory-service (8081): productos, stock, bodegas
      ├──► order-service     (8082): pedidos, clientes
      └──► shipping-service  (8083): envíos, transportistas

PostgreSQL (4) + RabbitMQ + pgAdmin
```

### Puertos y arranque

```
cd identity-service  && mvnw.cmd spring-boot:run   # :8084
cd api-gateway       && mvnw.cmd spring-boot:run   # :8080
cd inventory-service && mvnw.cmd spring-boot:run   # :8081
cd order-service     && mvnw.cmd spring-boot:run   # :8082
cd shipping-service  && mvnw.cmd spring-boot:run   # :8083
```

Usuarios semilla: `admin@smartlogix.cl / admin123`, `operator@smartlogix.cl / operator123`, `viewer@smartlogix.cl / viewer123`.

---

## 2. Concepto 1 — API Manager / Gateway

### Qué es (para la teoría)
Un API Manager (o API Gateway) es el **punto de entrada central** por donde pasan todas las peticiones del frontend hacia los backend. Resuelve: exposición unificada, seguridad centralizada, enrutamiento, CORS, observabilidad y rendimiento.

### Cómo lo implementa SmartLogix
Spring Cloud Gateway Server WebMVC. Las rutas se definen en `api-gateway/src/main/resources/application.properties`:

```properties
spring.cloud.gateway.server.webmvc.routes[0].id=identity-service
spring.cloud.gateway.server.webmvc.routes[0].uri=http://localhost:8084
spring.cloud.gateway.server.webmvc.routes[0].predicates[0]=Path=/api/auth/**

spring.cloud.gateway.server.webmvc.routes[2].id=inventory-service
spring.cloud.gateway.server.webmvc.routes[2].uri=http://localhost:8081
spring.cloud.gateway.server.webmvc.routes[2].predicates[0]=Path=/api/inventory/**
```

Significado: una petición a `http://localhost:8080/api/inventory/products` es enrutada a `http://localhost:8081/api/inventory/products`.

También expone health agregado en `GET /api/system/health` (consulta `/actuator/health` de los 4 servicios) y las rutas en `/actuator/gateway/routes`.

### Conceptos que demuestra
- Exposición unificada (un solo puerto 8080 para el frontend)
- Seguridad centralizada (validación JWT antes de reenviar)
- Enrutamiento por path
- Health agregado del sistema

---

## 3. Concepto 2 — JWT y Claims

### Qué es (para la teoría)
JWT (JSON Web Token) es un token compacto con 3 partes separadas por puntos: **Header.Payload.Signature**. Los **claims** son los datos dentro del payload (quiénes, cuándo expira, qué rol, etc.).

### Cómo lo implementa SmartLogix — Emisión del token

`identity-service/src/main/java/cl/duoc/smartlogix/identity/infrastructure/security/JwtTokenService.java`

```java
JwtClaimsSet claims = JwtClaimsSet.builder()
        .issuer(jwtProperties.getIssuer())        // claim iss
        .issuedAt(now)                            // claim iat
        .expiresAt(expiresAt)                     // claim exp
        .subject(user.getEmail())                 // claim sub
        .claim("userId", user.getId())            // claim custom
        .claim("role", user.getRole().getName().name())  // claim role
        .build();
JwsHeader header = JwsHeader.with(MacAlgorithm.HS256).type("JWT").build();

return jwtEncoder.encode(JwtEncoderParameters.from(header, claims)).getTokenValue();
```

### Claims reales de un token

```json
{"iss":"smartlogix-identity-service","sub":"viewer@smartlogix.cl","role":"VIEWER","exp":1778720805,"iat":1778717205,"userId":3}
```

### Configuración de firma (HS256)

`identity-service/.../config/SecurityConfig.java`

```java
@Bean
public JwtEncoder jwtEncoder(JwtProperties jwtProperties) {
    return new NimbusJwtEncoder(new ImmutableSecret<>(secretKey(jwtProperties.getSecret())));
}

@Bean
public JwtDecoder jwtDecoder(JwtProperties jwtProperties) {
    return NimbusJwtDecoder.withSecretKey(secretKey(jwtProperties.getSecret()))
            .macAlgorithm(MacAlgorithm.HS256)
            .build();
}
```

Secreto en `application.properties`:

```properties
smartlogix.jwt.issuer=smartlogix-identity-service
smartlogix.jwt.expiration-seconds=3600
smartlogix.jwt.secret=smartlogix-academic-mvp-secret-key-change-later-minimum-64-characters
```

### Conceptos que demuestra
- Estructura JWT (header/payload/signature)
- Claims estándar: iss, sub, iat, exp
- Claims custom: userId, role
- Firma simétrica HS256 (compartida entre todos los servicios)
- Expiración del token (3600 s)

---

## 4. Concepto 3 — Validación del JWT y autorización por rol

### Qué es (para la teoría)
El JWT es "auto-contenido": la autorización se decide leyendo los claims (sin consultar la BD en cada request). Spring Security necesita un **conversor** que transforme el claim `role` en una autoridad `ROLE_x`.

### Cómo lo implementa SmartLogix

`identity-service/.../config/SecurityConfig.java`

```java
.oauth2ResourceServer(oauth2 -> oauth2.jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthenticationConverter())));
```

```java
@Bean
public Converter<Jwt, ? extends AbstractAuthenticationToken> jwtAuthenticationConverter() {
    return jwt -> {
        String role = jwt.getClaimAsString("role");
        List<SimpleGrantedAuthority> authorities = role == null || role.isBlank()
                ? List.of()
                : List.of(new SimpleGrantedAuthority("ROLE_" + role));

        return new JwtAuthenticationToken(jwt, authorities, jwt.getSubject());
    };
}
```

### Reglas de autorización (SecurityFilterChain)

```java
.authorizeHttpRequests(authorize -> authorize
        .requestMatchers(HttpMethod.POST, "/api/auth/login").permitAll()
        .requestMatchers("/actuator/health", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
        .requestMatchers(HttpMethod.OPTIONS, "/**").permitAll()
        .requestMatchers("/api/auth/me").authenticated()
        .requestMatchers("/api/users", "/api/users/**").hasRole("ADMIN")
        .anyRequest().authenticated()
)
```

El gateway añade reglas por verbo HTTP sobre negocio:

```java
.requestMatchers(HttpMethod.GET, "/api/inventory/**", "/api/orders/**", "/api/shipping/**")
.hasAnyRole("ADMIN", "OPERATOR", "VIEWER")          // GET: todos los roles
.requestMatchers("/api/inventory/**", "/api/orders/**", "/api/shipping/**")
.hasAnyRole("ADMIN", "OPERATOR")                    // mutaciones: solo ADMIN/OPERATOR
```

### Roles
- `ADMIN`: acceso completo
- `OPERATOR`: operación diaria
- `VIEWER`: solo lectura

### Conceptos que demuestra
- Validación del token en resource server
- Conversión claim → autoridad (ROLE_x)
- RBAC por URL y por verbo HTTP
- Doble capa de validación: gateway Y cada microservicio revalida el token

---

## 5. Concepto 4 — CORS

### Qué es (para la teoría)
CORS (Cross-Origin Resource Sharing) relaja la Same-Origin Policy. Cuando el navegador envía requests con headers no simples (como `Authorization`), primero dispara un **preflight OPTIONS**. El servidor debe responder qué orígenes, métodos y headers permite.

### Cómo lo implementa SmartLogix

`api-gateway/src/main/java/cl/duoc/smartlogix/gateway/config/CorsConfig.java`

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:3000", "http://localhost:5173")
                .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
                .allowedHeaders("Authorization", "Content-Type", "X-Correlation-Id")
                .exposedHeaders("X-Correlation-Id")
                .maxAge(3600);
    }
}
```

Además, cada microservicio hace `.cors(Customizer.withDefaults())` para el caso de llamada directa.

### Conceptos que demuestra
- Same-Origin Policy y por qué el frontend en :5173 no puede llamar al backend :8080 sin CORS
- Preflight OPTIONS (por eso se permite el método OPTIONS)
- Headers permitidos (Authorization para el Bearer token)

---

## 6. Concepto Faltante — Versionado de APIs

SmartLogix **no implementa versionado** de APIs (no hay `/api/v1/`). Las únicas menciones de "version" son metadatos de OpenAPI (`OpenApiConfig.java` con version "1.0.0").

**Punto a mencionar en la defensa (honestidad técnica):** para versionar se usaría prefijo `/api/v1/...` en los controllers o un esquema header/query. En el proyecto propio (Eco-Mantenimiento/Azure APIM) el versionado SÍ está implementado (v1/v2 con Segment scheme).

---

## 7. OAuth2/OIDC en SmartLogix — el matiz importante

SmartLogix usa `spring-boot-starter-oauth2-resource-server`, pero **NO implementa el protocolo OAuth2/OIDC completo**:

| Característica OAuth2/OIDC | ¿Presente? |
|---|---|
| Authorization Server externo (Azure AD B2C, Keycloak, Auth0) | ❌ No |
| Flujo Authorization Code + PKCE | ❌ No |
| client_id / client_secret | ❌ No |
| Scopes (openid, email) | ❌ No |
| Refresh tokens | ❌ No |
| JWKS (claves públicas) | ❌ No (secreto simétrico compartido) |
| Validación de issuer | ❌ No (issuer es un claim, no se valida) |
| Emisor y validador JWT propio (HS256) | ✅ Sí |

**Conclusión técnica:** SmartLogix demuestra el **patrón de JWT Bearer + Resource Server** y el **API Gateway**, pero la autenticación es local (formulario propio), no "Identity as a Service".

---

## 8. Comparación con Eco-Mantenimiento (proyecto propio)

| Concepto | SmartLogix | Eco-Mantenimiento |
|---|---|---|
| Emisión de tokens | Interna HS256 (local) | Azure AD B2C (IDaaS, RSA/JWKS) |
| OAuth2/OIDC | Solo resource server JWT | Authorization Code + PKCE (MSAL) |
| API Gateway | Spring Cloud Gateway | Azure API Management |
| CORS | CorsConfig.java | Política CORS en APIM |
| Versionado | ❌ | ✅ v1/v2 |
| Roles | ADMIN/OPERATOR/VIEWER | Perfil de usuario B2C |
| Validación JWT | Gateway + microservicios (secreto) | Backend Spring (JWKS) |

---

## 9. Flujo completo para la demo

```
1. Frontend: POST http://localhost:8080/api/auth/login  (email + password)
2. Gateway:  recibe, NO valida (permitAll), enruta a identity-service :8084
3. Identity: valida BCrypt, genera JWT (iss/exp/sub/role), responde {accessToken}
4. Frontend: guarda token y lo envía como "Authorization: Bearer <jwt>"
5. GET /api/inventory/products → gateway valida JWT + rol → enruta a :8081 → 200
6. Sin token → 401; con token VIEWER y método mutante → 403
```

Frontend (`frontend/src/lib/security/httpClient.ts`):

```ts
const token = auth ? authTokenProvider.getToken() : null;
if (token) {
    requestHeaders.set("Authorization", `Bearer ${token}`);
}
```

---

## 10. Checklist para la defensa (basado en docs/evidencia-defensa.md)

- [ ] Login exitoso con usuario admin (obtener JWT)
- [ ] Mostrar JWT decodificado (jwt.io o Swagger): header/payload/signature
- [ ] Explicar claims (iss, sub, exp, iat, role, userId)
- [ ] Mostrar `/actuator/gateway/routes` (enrutamiento)
- [ ] Stock antes/después de un pedido (evento RabbitMQ ORDER_OUT)
- [ ] Shipment auto-creado, IN_TRANSIT → DELIVERED
- [ ] Llamada sin token → 401; con token → 200
- [ ] Acceso denegado por rol → 403
- [ ] Swagger de cada servicio (`/v3/api-docs`)
- [ ] Tablas en pgAdmin
- [ ] Explicar diferencias: JWT propio (HS256) vs IDaaS (Azure AD B2C)

---

## Referencias dentro del repositorio

- `identity-service/.../config/SecurityConfig.java` — firma/validación JWT + autorización
- `identity-service/.../infrastructure/security/JwtTokenService.java` — emisión del token
- `identity-service/.../presentation/controller/AuthController.java` — login y /me
- `api-gateway/src/main/resources/application.properties` — rutas del gateway
- `api-gateway/.../config/SecurityConfig.java` — validación JWT en el gateway
- `api-gateway/.../config/CorsConfig.java` — CORS
- `frontend/src/lib/security/httpClient.ts` — Bearer token en peticiones
- `docs/` — ARCHITECTURE.md, BACKEND.md, API.md, evidencia-defensa.md