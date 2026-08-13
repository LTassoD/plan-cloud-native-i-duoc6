# Plan del Semestre — Desarrollo Cloud Native I

**Escuela de Informática y Telecomunicaciones · Duoc UC · Presencial**
**Sigla: [confirmar en AVA] · Total: 90 horas (referencial) · Caso práctico en AWS Academy (API Manager / API Gateway)**

> Plan tentativo de 16 semanas. Basado en los materiales disponibles en la carpeta (actividades 1.1.x del bloque 1).
> **La sigla, el programa completo y la ponderación no estaban en la carpeta**: confirma en AVA antes de fijar fechas.

---

## 1. Estructura de evaluaciones y ponderación

⚠️ **Pendiente de confirmar en AVA.** Se espera el esquema estándar Duoc: parciales (60%) + Evaluación Final Transversal (40%), con un caso práctico cloud. Anota aquí las ponderaciones reales:

| Evaluación | Tipo | Peso | Entrega |
|---|---|---|---|
| Ev Parcial 1 · ____ | ____ | __% | ____ |
| Ev Parcial 2 · ____ | ____ | __% | ____ |
| Ev Parcial 3 · ____ | ____ | __% | ____ |
| **Evaluación Final Transversal** | ____ | **40%** | ____ |

**Regla de cálculo (referencial):** Nota final = (Parciales × 60%) + (Final Transversal × 40%)

---

## 2. Ruta de aprendizaje y cronograma

### Bloque 1 — EA1: API Management y exponer APIs seguras (semanas 1–4 · materiales disponibles)

| Semana | Actividad | Descripción / Entregable |
|---|---|---|
| 1 | Act 1.1.1 · Conociendo un API Manager | Qué es un API Manager (punto de entrada central gestionado), API Gateway, tipos de API (REST, WebSocket), casos de uso, portal para desarrolladores. Conceptos de la guía |
| 2 | Act 1.1.2 · Tutorial: Creando Nuestro Primer API Manager | **Práctica AWS Academy:** ingresar al tenant de AWS, crear un servicio de API Gateway para securitizar APIs. Capturas de pantalla del proceso |
| 3 | Act 1.1.3 · Versionando APIs | Por qué versionar, versionamiento semántico, versionamiento en API Gateway, proceso de deprecación de una API, buenas prácticas |
| 4 | Act 1.1.4 · Configurando CORS en el API Gateway | Qué es CORS (Cross-Origin Resource Sharing) y por qué se configura en el API Gateway. **Práctica en AWS** |

**📌 Ev Parcial 1** → probablemente cierra este bloque (confirmar en AVA).

### Bloques 2 y 3 — Pendientes de materiales (semanas 5–15)

> ⚠️ Los materiales de las actividades 1.2.x en adelante aún no están en la carpeta. Temas probables según el enfoque de la asignatura (confirmar con el docente/AVA):
> - **Microservicios y contenedores** (arquitecturas nativas cloud, Docker, orquestación)
> - **Seguridad e identidad** (Identity as a Service, control de accesos, JWT/OAuth)
> - **Colas y streaming de datos**
> - **Despliegue y observabilidad** en la nube

### Cierre — Semana 16

| Semana | Actividad | Horas |
|---|---|---|
| 16 | **Evaluación Final Transversal (40%)** | ____ |

---

## 3. Detalle de entregables ya disponibles (guías en la carpeta)

### Act 1.1.1 — Conociendo un API Manager (guía)
- **12 páginas.** API Manager como capa gestionada que elimina la administración de infraestructura: punto de entrada central, puente seguro entre cliente y servicios backend, REST y WebSocket, portal para desarrolladores.
- **Tip:** estudia los desafíos que resuelve (infraestructura manual, carga operativa, retrasos en desarrollo, vulnerabilidades).

### Act 1.1.2 — Tutorial: Creando Nuestro Primer API Manager (práctica AWS)
- **13 páginas · práctica en AWS Academy.** Crear un API Manager para securitizar APIs usando el servicio de API Gateway. Debes registrar cada paso con capturas.

### Act 1.1.3 — Versionando APIs (guía)
- **15 páginas.** Versionamiento semántico, versionamiento en API Gateway, proceso de deprecación de una API y buenas prácticas. Razones: no romper clientes existentes, agregar funcionalidades sin afectar consumidores.

### Act 1.1.4 — Configurando CORS en el API Gateway (guía 2025)
- **6 páginas.** CORS (Cross-Origin Resource Sharing): mecanismo de seguridad de navegadores, cuándo se requiere y cómo configurarlo en el API Gateway de AWS.

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

## 5. Referencias sugeridas

1. AWS Academy (portal con el tenant de prácticas).
2. Documentación de AWS API Gateway.
3. Documentación de CORS (MDN Web Docs) y versionamiento semántico (semver.org).
