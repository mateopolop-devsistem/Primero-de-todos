# 01 — Arquitectura del sistema

## 1. Principio rector

> **Monolito modular, no microservicios.**

JB es una empresa, no una big tech. Un equipo chico (1 a 3 desarrolladores) con
microservicios pasa el 70% del tiempo resolviendo infraestructura en lugar de
funcionalidad. La arquitectura propuesta es un **monolito modular desplegado como
una sola aplicación**, pero con **límites internos estrictos** entre dominios.

La ventaja: si en 3 años un módulo (por ejemplo, logística) necesita separarse,
ya está aislado y se extrae sin reescribir. Se paga el costo de la disciplina,
no el de la infraestructura distribuida.

---

## 2. Vista general

```
┌───────────────────────────────────────────────────────────────────────┐
│                             CLIENTES                                  │
│   Mobile web (70-80% del tráfico)   ·   Desktop   ·   PWA instalable  │
└───────────────────────────────┬───────────────────────────────────────┘
                                │ HTTPS
┌───────────────────────────────▼───────────────────────────────────────┐
│                    NEXT.JS (App Router) — Vercel                      │
│                                                                       │
│  ┌─────────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │  Tienda pública     │  │  Cuenta cliente  │  │  Panel admin     │  │
│  │  (RSC, cacheada,    │  │  (privada,       │  │  (privado,       │  │
│  │   SEO, ISR)         │  │   dinámica)      │  │   sin SEO)       │  │
│  └──────────┬──────────┘  └────────┬─────────┘  └────────┬─────────┘  │
│             └───────────────┬──────┴─────────────────────┘            │
│                             ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │  CAPA DE APLICACIÓN — Server Actions + Route Handlers            │  │
│  │  Validación (Zod) · Autorización · Orquestación de casos de uso  │  │
│  └───────────────────────────────┬─────────────────────────────────┘  │
│                                  ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │  MÓDULOS DE DOMINIO                                              │  │
│  │  catalog · pricing · cart · checkout · orders · inventory        │  │
│  │  payments · shipping · customers · notifications · reporting     │  │
│  └───────────────────────────────┬─────────────────────────────────┘  │
│                                  ▼                                    │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │  INFRAESTRUCTURA — repositorios, clientes externos, storage      │  │
│  └───────────────────────────────┬─────────────────────────────────┘  │
└──────────────────────────────────┼────────────────────────────────────┘
                                   │
     ┌──────────────┬──────────────┼──────────────┬──────────────────┐
     ▼              ▼              ▼              ▼                  ▼
┌──────────┐  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌────────────────┐
│PostgreSQL│  │  Redis   │  │Cloudinary │  │  Resend  │  │  Mercado Pago  │
│ (Neon)   │  │(Upstash) │  │ imágenes  │  │  email   │  │  pagos + IPN   │
│ datos +  │  │ caché,   │  │  + CDN    │  │          │  │                │
│ búsqueda │  │ rate lim,│  │           │  │  WhatsApp│  │  ARCA/AFIP     │
│          │  │ colas    │  │           │  │  Cloud   │  │  facturación   │
└──────────┘  └──────────┘  └───────────┘  └──────────┘  └────────────────┘
```

---

## 3. Las tres aplicaciones dentro de una

Aunque se despliega una sola app, hay **tres superficies con requisitos opuestos**.
Confundirlas es el error más común y el más caro.

### 3.1 Tienda pública (`/`, `/productos`, `/producto/[slug]`)

- **Prioridad: velocidad y SEO.** Es la que capta clientes nuevos.
- Renderizado en el servidor (RSC) con **ISR**: el catálogo se regenera cada N
  minutos o bajo demanda al editar un producto (revalidación por tag).
- El HTML llega ya armado. Sin spinners, sin "cargando productos".
- Los precios personalizados (mayorista) se resuelven en una capa dinámica chica
  sobre la página cacheada, para no romper el caché por cliente.
- **Objetivo medible:** LCP < 2.0 s en 4G, Lighthouse ≥ 90 en mobile.

### 3.2 Cuenta del cliente (`/mi-cuenta/*`)

- **Prioridad: exactitud.** Nunca cacheada, siempre datos frescos.
- Historial, seguimiento de pedidos, repetir compra, direcciones, comprobantes.

### 3.3 Panel administrativo (`/admin/*`)

- **Prioridad: densidad de información y velocidad de operación.**
- Sin SEO, sin ISR, sin optimización de first-load. Se puede usar rendering del
  lado del cliente donde convenga (tablas con filtros complejos).
- Segregado por middleware y por rol. Aunque comparte el deploy, **no comparte
  ni el layout, ni los componentes, ni las reglas de caché** con la tienda.

---

## 4. Módulos de dominio

Cada módulo posee sus tablas, sus reglas y su API interna. **Ningún módulo lee
tablas de otro módulo directamente**: se comunica a través de la interfaz pública
del módulo (`modules/<x>/index.ts`).

| Módulo | Responsabilidad | No es responsable de |
|---|---|---|
| **catalog** | Productos, líneas genéticas (`chick_specs`), categorías, atributos, imágenes, búsqueda | Precios finales, disponibilidad |
| **pricing** | Listas de precios, escalas por cantidad, grupos de cliente, promociones, cupones, IVA | Mostrar precios en pantalla |
| **hatchery** ⚠ | **Camadas: fechas de nacimiento, cupo, reservas, cierre, nacimiento real, lista de espera** | Decidir si se puede vender (eso lo pregunta checkout) |
| **inventory** | Stock de insumos por depósito, lotes, movimientos | Cupo de camadas (eso es `hatchery`) |
| **advisor** ⚠ | Asesor de compra: traduce mercado objetivo → línea recomendada | Vender |
| **cart** | Carrito persistente, ítems, recálculo, expiración de reservas | Cobrar |
| **checkout** | Validación del pedido, logística, creación de la orden | Procesar el pago |
| **orders** | Ciclo de vida del pedido, estados, faltantes, reclamos, historial | Cobrar, despachar |
| **payments** | Mercado Pago, transferencias, comprobantes, conciliación, reembolsos | Cambiar el estado del pedido (lo notifica) |
| **dispatch** ⚠ | **Transportes, agencias de destino, días de salida, guías, recorridos propios, retiro** | Cobrar el flete (muchas veces no lo cobra JB) |
| **customers** | Cuentas, perfiles fiscales, direcciones, grupos, aprobación mayorista | Autenticación (delega en auth) |
| **notifications** | Email, WhatsApp, notificaciones internas, plantillas | Decidir cuándo notificar (lo dispara cada módulo por evento) |
| **reporting** | Métricas de venta, ocupación de camadas, embudo, mortandad por transporte | Escribir en tablas operativas |
| **cms** | Home editable, banners, páginas, FAQ, **guías de crianza** | Catálogo |

> **`hatchery` y `dispatch` son los dos módulos que no existen en un e-commerce
> genérico** y los que justifican una plataforma propia. El resto es estándar.

### Comunicación entre módulos: eventos de dominio

Los módulos se acoplan por eventos, no por llamadas directas, cuando la acción es
un efecto secundario:

```
order.paid            → hatchery.confirmarReserva
                      → notifications.confirmarConFecha
                      → dispatch.planificarDespacho
                      → invoicing.emitirComprobante   (Fase 3)

batch.hatched         → orders.marcarNacidos
                      → dispatch.generarDespachosDelDia
                      → notifications.avisarNacimiento

batch.short_hatch  ⚠  → orders.aplicarProtocoloDeFaltante
                      → payments.reintegrarProporcional
                      → notifications.avisarFaltante

dispatch.shipped      → notifications.enviarNumeroDeGuia

customer.approved_wholesale → notifications.darBienvenidaMayorista
```

`batch.short_hatch` (nacieron menos de los comprometidos) es el evento que más
importa que esté bien resuelto: dispara el aviso proactivo al cliente antes de
que se entere por su cuenta.

Implementación en Fase 1: un despachador de eventos en proceso, síncrono para lo
crítico y encolado en **Upstash QStash** para lo que puede fallar sin romper la
compra (emails, WhatsApp, facturación). Si el email de confirmación falla, **la
compra no se cae**.

---

## 5. Capas y regla de dependencia

```
   ui  →  application  →  domain  ←  infrastructure
```

- **domain**: entidades, reglas de negocio puras, tipos. Sin imports de Next, de
  la base de datos ni de librerías externas. Es testeable con Vitest en milisegundos.
- **application**: casos de uso (`agregarAlCarrito`, `confirmarPedido`,
  `ajustarPesoReal`). Orquesta domain + repositorios. Acá vive la transacción.
- **infrastructure**: implementaciones concretas (Prisma, Mercado Pago SDK,
  Cloudinary, Resend).
- **ui**: componentes React y rutas. **Nunca** contiene lógica de negocio. Si un
  componente calcula un precio, hay un error de diseño.

**Regla dura:** la lógica de precios existe **una sola vez**, en `pricing`. Si el
precio se calcula distinto en la ficha de producto que en el carrito, aparecen
reclamos de clientes. Esto no es purismo: es la fuente número uno de bugs caros
en e-commerce.

---

## 6. Mutaciones: Server Actions

Toda mutación pasa por un Server Action con el mismo esqueleto:

```
1. Autenticar   → ¿quién es?
2. Autorizar    → ¿puede hacer esto?
3. Validar      → Zod sobre la entrada, sin excepción
4. Ejecutar     → caso de uso en application
5. Revalidar    → revalidateTag / revalidatePath
6. Responder    → { ok: true, data } | { ok: false, error }
```

Nunca se lanza una excepción cruda a la UI. El cliente recibe siempre un resultado
tipado, y los errores inesperados van a Sentry.

**Excepción:** los *webhooks* (Mercado Pago, y a futuro proveedores de envío) son
Route Handlers en `app/api/webhooks/*`, con verificación de firma, idempotencia
por `event_id` y respuesta 200 inmediata.

---

## 7. Seguridad

| Riesgo | Mitigación |
|---|---|
| Manipulación de precios desde el cliente | El precio **jamás** viaja del navegador al servidor. Se recalcula íntegro en el servidor al confirmar el pedido |
| Webhook de pago falsificado | Verificación de firma HMAC + consulta de confirmación a la API de MP antes de marcar pagado |
| **Sobreventa de cupo de camada** | Reserva con bloqueo transaccional (`SELECT ... FOR UPDATE`) + TTL de 30 min. Es el riesgo operativo más grave: comprometer más pollitos de los que van a nacer deja a un productor sin producción |
| Acceso al panel admin | Middleware por rol + segundo factor para roles con permisos financieros (Fase 2) |
| Fuerza bruta en login | Rate limiting por IP y por email en Upstash |
| Datos fiscales de clientes | Cifrado en reposo del proveedor + acceso restringido por rol + registro de auditoría |
| Subida de comprobantes | Validación de tipo MIME real, límite de tamaño, almacenamiento fuera del webroot |
| Spam en formularios | Cloudflare Turnstile |

Cumplimiento: **Ley 25.326 de Protección de Datos Personales** (Argentina).
Requiere política de privacidad, base de datos registrada ante la AAIP y
mecanismo de baja/rectificación. Está contemplado en la Fase 1.

---

## 8. Rendimiento

- **Imágenes**: el 80% del peso de un e-commerce. Cloudinary con `f_auto,q_auto`,
  AVIF/WebP, `next/image` con `sizes` correcto y placeholder borroso.
- **Caché por capas**: CDN (estático) → ISR (catálogo) → Redis (precios y stock,
  TTL corto) → Postgres.
- **Invalidación por tags**: al editar un producto, se invalida
  `producto:{id}`, `categoria:{id}` y `home` — no toda la web.
- **Presupuesto de JS**: ≤ 180 KB comprimidos en la ruta de producto. Cada
  dependencia nueva del lado del cliente se justifica.

---

## 9. Entornos

| Entorno | Uso | Base de datos | Mercado Pago |
|---|---|---|---|
| `local` | Desarrollo | Docker Postgres | Credenciales de prueba |
| `preview` | Una URL por Pull Request | Rama de Neon (copia) | Credenciales de prueba |
| `staging` | Validación del cliente antes de publicar | Neon staging con datos reales anonimizados | Credenciales de prueba |
| `production` | Público | Neon production | Credenciales productivas |

Backups: automáticos diarios de Neon con retención de 30 días + *point-in-time
recovery* de 7 días. **Se prueba una restauración antes de salir a producción.**
Un backup que nunca se restauró no es un backup.

---

## 10. Qué NO está en la arquitectura (y por qué)

| Descartado | Motivo |
|---|---|
| Microservicios | Complejidad injustificada para el volumen y el equipo |
| App móvil nativa | La PWA cubre el caso de uso al 90% del costo. Reevaluar en Fase 4 |
| GraphQL | Server Actions + RSC ya eliminan el problema del *over-fetching* |
| Headless CMS externo | El CMS propio para home y banners es más simple y sin costo mensual |
| Elasticsearch en Fase 1 | Postgres full-text alcanza sobrado hasta ~5.000 SKU |
| Kubernetes | Vercel resuelve el escalado sin equipo de infraestructura |
