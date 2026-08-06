# 03 — Stack tecnológico

## Criterios de elección

1. **Un solo lenguaje.** TypeScript en front y back. Un equipo chico no puede
   mantener dos ecosistemas.
2. **Aburrido y probado antes que nuevo y brillante.** Esta plataforma tiene que
   funcionar 5 años, no ganar un premio.
3. **Contratable.** Si mañana JB necesita otro desarrollador, tiene que
   conseguirlo. Next.js + Postgres es el stack más contratable de la región.
4. **Sin lock-in en lo crítico.** Postgres y Docker permiten migrar de Vercel/Neon
   a un VPS en un fin de semana si el costo cambia.

---

## Stack recomendado

### Núcleo

| Capa | Elección | Por qué |
|---|---|---|
| Framework | **Next.js 15 (App Router)** | SSR/ISR para SEO —determinante para captar clientes nuevos—, Server Actions, un solo despliegue |
| Lenguaje | **TypeScript** (`strict`) | Errores de tipo en un e-commerce = errores de plata |
| Estilos | **Tailwind CSS 4** | Design system con *tokens*, sin CSS muerto acumulado |
| Componentes | **shadcn/ui + Radix UI** | Código propio, no dependencia. Accesibilidad resuelta (teclado, foco, ARIA) |
| Base de datos | **PostgreSQL 16** | Transacciones reales, JSONB para atributos flexibles, full-text search nativo |
| ORM | **Prisma 6** | Migraciones seguras, tipos generados, curva de aprendizaje baja |
| Autenticación | **Auth.js v5** | Sin costo por usuario, credenciales + Google, sesiones en base |
| Validación | **Zod** | Un esquema define validación + tipo. Frontera obligatoria en todo Server Action |
| Formularios | **React Hook Form** | Rendimiento en formularios largos (checkout, alta de producto) |

### Servicios

| Necesidad | Elección | Costo aprox. inicial |
|---|---|---|
| Hosting | **Vercel** (Pro) | USD 20/mes |
| Base de datos | **Neon** (Postgres serverless) | USD 0–19/mes |
| Caché y colas | **Upstash Redis + QStash** | USD 0–10/mes |
| Imágenes | **Cloudinary** | USD 0 hasta 25 créditos |
| Email transaccional | **Resend + React Email** | USD 0–20/mes |
| Pagos | **Mercado Pago** | Comisión por venta |
| Errores | **Sentry** | USD 0–26/mes |
| Analítica de producto | **PostHog** o **Umami** | USD 0 |
| Anti-spam | **Cloudflare Turnstile** | USD 0 |
| Dominio + DNS | **Cloudflare** | ~USD 15/año |

**Costo de infraestructura estimado en Fase 1: USD 40–70/mes.** No incluye
comisiones de Mercado Pago (~4,4% + IVA para acreditación inmediata; menor a 10
o 30 días) ni dominio.

### Desarrollo

| Herramienta | Uso |
|---|---|
| **pnpm** | Gestor de paquetes (más rápido, menos disco) |
| **Vitest** | Tests unitarios y de integración |
| **Playwright** | Tests end-to-end del flujo de compra |
| **Biome** o ESLint + Prettier | Formato y linting |
| **Docker Compose** | Postgres + Redis en local |
| **GitHub Actions** | CI: typecheck, lint, tests, migraciones |

---

## Decisiones evaluadas y descartadas

### Framework

| Opción | Por qué no |
|---|---|
| **WordPress + WooCommerce** | Solución rápida y barata al inicio, pero el peso variable, las listas mayoristas y las reservas de camadas exigirían 6+ plugins de terceros y código a medida sobre ellos. Rendimiento mobile pobre, mantenimiento y seguridad permanentes. **Descartado por el modelo de negocio, no por prejuicio técnico.** |
| **Tiendanube / Shopify** | Excelentes si JB vendiera productos de peso fijo. No modelan precio por kilo con ajuste posterior ni preventa por fecha de nacimiento sin apps caras y limitadas. Comisión sobre ventas. Reevaluable si se decide simplificar el modelo comercial |
| **Astro** | Superior en sitios de contenido, más fricción en la parte interactiva y en el panel admin |
| **Laravel + Vue** | Stack sólido, pero implica dos lenguajes y menos disponibilidad de desarrolladores en el segmento de precio de JB |
| **Medusa.js / Vendure** | E-commerce headless serio, pero exige adaptar su modelo de precios para peso variable **y** además construir todo el frontend. Más complejidad total, no menos |

### Base de datos

| Opción | Por qué no |
|---|---|
| MySQL | Menos capacidades para búsqueda de texto y JSON |
| MongoDB | Un e-commerce es relacional por naturaleza (pedido → ítems → stock). Las transacciones ACID no son opcionales acá |
| Supabase | Muy buena alternativa a Neon. Se descarta sólo porque su fuerte (auth y realtime integrados) no se aprovecha con Auth.js. **Es el plan B natural** |

### ORM

Prisma vs. **Drizzle**: Drizzle es más liviano y genera SQL más predecible.
Prisma gana por herramientas de migración maduras, Prisma Studio (útil para que
personal no técnico de JB mire datos) y documentación abundante. Si el equipo
final tiene experiencia con Drizzle, **el cambio no afecta la arquitectura**:
sólo la carpeta `infrastructure` de cada módulo.

### Autenticación

Auth.js vs. **Clerk**: Clerk ahorra ~1 semana de desarrollo y trae 2FA y gestión
de usuarios lista. Se descarta por costo recurrente por usuario activo — con
miles de clientes B2C esporádicos, el precio crece contra un componente que se
escribe una vez. Si el time-to-market pesa más que el costo, Clerk es defendible.

### Búsqueda

Fase 1: **PostgreSQL full-text** con `tsvector`, `pg_trgm` (tolerancia a errores
de tipeo: "pollito" ≈ "poyito") y diccionario en español. Suficiente y sobrado
hasta ~5.000 productos.

Fase 3, si el catálogo crece o hace falta búsqueda facetada compleja:
**Meilisearch** o **Typesense** (autohospedado o gestionado, ~USD 30/mes).

---

## Integraciones específicas de Argentina

| Integración | Fase | Nota |
|---|---|---|
| **Mercado Pago Checkout Pro** | 1 | Redirección. Máxima confianza del comprador, mínima superficie de riesgo (JB nunca toca datos de tarjeta) |
| **Mercado Pago Checkout Bricks** | 2 | Pago embebido en el sitio. Mejora conversión ~5–10%, pero exige el flujo Pro ya estable |
| **Transferencia bancaria** | 1 | CBU/alias + subida de comprobante + validación manual en el panel. Clave para B2B: evita la comisión de MP en tickets altos |
| **WhatsApp Business Cloud API** | 2 | Notificación de estado del pedido por el canal que el cliente ya usa. Alta tasa de apertura |
| **ARCA (ex AFIP) — facturación electrónica** | 3 | Vía **TusFacturas** o **Facturante**. IVA 10,5% en carne aviar vs. 21% en insumos: el modelo de datos ya lo contempla |
| **Andreani / Correo Argentino / OCA** | 3 | Sólo si se vende fuera del radio de reparto propio. **Los pollos frescos requieren cadena de frío: reparto propio o retiro.** Los insumos secos sí pueden ir por correo |
| **Google Merchant Center** | 2 | Feed de productos para Google Shopping. Fuente de tráfico de compra directa |
| **Meta Pixel + API de Conversiones** | 2 | Remarketing en Instagram/Facebook, donde está el público avícola |

---

## Variables de entorno (referencia)

```bash
# Base
DATABASE_URL=
DIRECT_URL=                      # migraciones sin pooling
NEXT_PUBLIC_SITE_URL=

# Auth
AUTH_SECRET=
AUTH_GOOGLE_ID=
AUTH_GOOGLE_SECRET=

# Mercado Pago
MP_ACCESS_TOKEN=
MP_PUBLIC_KEY=
MP_WEBHOOK_SECRET=

# Infra
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=
QSTASH_TOKEN=
CLOUDINARY_URL=
RESEND_API_KEY=

# Observabilidad
SENTRY_DSN=
NEXT_PUBLIC_POSTHOG_KEY=

# Negocio
JB_WHATSAPP_NUMBER=
JB_CUIT=
JB_BANK_CBU=
JB_BANK_ALIAS=
```

Ningún secreto se versiona. `.env.example` documenta las claves sin valores.

---

## Plan de salida (anti lock-in)

Si Vercel o Neon dejan de convenir económicamente:

1. Next.js corre en cualquier Node con `output: 'standalone'` → Dockerfile listo.
2. Postgres es Postgres → `pg_dump` y restaurar en un VPS.
3. Redis y colas → Redis autohospedado.
4. Cloudinary → S3 compatible + servidor de imágenes.

**Migración estimada: 2 a 4 días.** Ese es el precio del lock-in, y es aceptable.
