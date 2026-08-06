# 10 — Plan de desarrollo por fases

## Supuestos del plan

- Equipo: **1 desarrollador full stack dedicado** (si son 2, los tiempos se
  reducen ~35%, no 50%).
- Sprints de **2 semanas** con demo al final de cada uno.
- JB asigna **un referente único** que valida y provee contenido. Es el mayor
  riesgo del cronograma: sin fotos, textos y datos de productos a tiempo, el
  desarrollo se frena aunque el código esté listo.

---

## Vista general

| Fase | Duración | Objetivo | Estado al final |
|---|---|---|---|
| **0 · Definición** | 1–2 sem | Validar decisiones, contenido y diseño | Nada que rehacer después |
| **1 · MVP vendible** | 10–12 sem | Vender online de punta a punta | **En producción, facturando** |
| **2 · Crecimiento** | 6–8 sem | Más conversión, menos trabajo manual | Optimizado |
| **3 · Escala** | 8–10 sem | Facturación, logística, fidelización | Operación integrada |
| **4 · Expansión** | continuo | B2B avanzado, app, nuevos canales | Plataforma madura |

**Salida a producción: entre la semana 11 y la 14 desde el inicio.**

---

## FASE 0 · Definición (1–2 semanas)

**No se escribe código de producto.** Es la fase que más plata ahorra.

### Entregables

| # | Entregable | Responsable |
|---|---|---|
| 0.1 | Decisiones del documento 11 respondidas | **JB** |
| 0.2 | Inventario real de productos (nombre, precio, unidad, peso, stock) | **JB** |
| 0.3 | Fotos de producto y de la planta | **JB** |
| 0.4 | Identidad de marca (logo, colores) o mínimo viable | JB / Diseño |
| 0.5 | Datos fiscales, bancarios, cuenta de Mercado Pago | **JB** |
| 0.6 | Zonas de reparto, costos y días | **JB** |
| 0.7 | Textos legales (términos, privacidad, envíos, devoluciones) | Desarrollo + JB |
| 0.8 | Diseño en Figma: home, catálogo, ficha, carrito, checkout (mobile + desktop) | Diseño |
| 0.9 | Repositorio, entornos, CI/CD, dominio | Desarrollo |

### Criterio de salida
Diseño aprobado por JB **y** las 6 decisiones bloqueantes del documento 11
cerradas por escrito.

> **Sin fotos reales de producto, la fase 1 no arranca.** Es el activo que más
> impacta la conversión y no se puede reemplazar con banco de imágenes.

---

## FASE 1 · MVP vendible (10–12 semanas)

**Objetivo:** que un cliente compre pollos online, pague y reciba, sin que nadie
de JB intervenga manualmente.

### Sprint 1 — Cimientos (sem. 1–2)
- Next.js + TypeScript + Tailwind + design system base
- Prisma + esquema completo + migraciones + seed
- Auth.js (registro, login, recuperación de clave)
- Layout: header, footer, navegación mobile
- Middleware de rutas protegidas, Sentry, CI
- **Demo:** navegar el esqueleto, registrarse, iniciar sesión

### Sprint 2 — Catálogo (sem. 3–4)
- Módulo `catalog`: productos, variantes, categorías, atributos, imágenes
- Listado con filtros, orden y paginación
- Ficha de producto (los cuatro tipos, incluido peso variable)
- Búsqueda con Postgres full-text + autocompletado
- Admin: alta y edición de productos, categorías, imágenes
- **Demo:** JB carga productos reales y se ven en la web

### Sprint 3 — Precios, stock y carrito (sem. 5–6)
- Módulo `pricing`: listas, grupos, escalas por volumen, IVA
- Módulo `inventory`: stock, reservas con bloqueo, movimientos
- Carrito del lado del servidor + drawer + página
- Cálculo de peso estimado
- Admin: gestión de stock y precios
- **Demo:** agregar al carrito con precio correcto y stock reservado

### Sprint 4 — Checkout y pagos (sem. 7–8) ⚠ el más riesgoso
- Módulo `shipping`: zonas, métodos, cálculo, cadena de frío, franjas
- Checkout completo con validaciones
- **Mercado Pago Checkout Pro** + webhook + idempotencia + conciliación
- Transferencia bancaria + subida y validación de comprobante
- Creación del pedido, máquina de estados
- Emails transaccionales (Resend + React Email)
- **Demo:** compra real de punta a punta con dinero real en modo prueba

### Sprint 5 — Panel y peso variable (sem. 9–10)
- Dashboard con las colas de atención
- Pedidos: tablero, tabla, detalle, cambio de estado
- **Pantalla de pesaje + motor de ajuste + notificaciones**
- Clientes y aprobación de mayoristas
- Validación de transferencias
- Cuenta del cliente: pedidos, seguimiento, direcciones
- **Demo:** JB opera un día completo de pedidos desde el panel

### Sprint 6 — Cierre y salida a producción (sem. 11–12)
- Home definitiva con CMS de secciones y banners
- Páginas institucionales y legales, botón de arrepentimiento
- SEO: metadatos, sitemap, datos estructurados de producto, Open Graph
- Rendimiento: imágenes, caché, Core Web Vitals
- Accesibilidad: revisión WCAG AA
- Tests E2E del flujo de compra (Playwright)
- Carga masiva del catálogo completo
- **Capacitación del equipo de JB + manual de uso**
- Revisión de seguridad
- **Salida a producción**

### Criterios de aceptación de la Fase 1

- [ ] Un cliente compra con Mercado Pago sin intervención humana
- [ ] Un cliente compra por transferencia y JB la valida en el panel
- [ ] El peso variable se cobra estimado, se ajusta y se notifica
- [ ] Un mayorista se registra, es aprobado y ve precios mayoristas
- [ ] El stock nunca se sobrevende (probado con pedidos concurrentes)
- [ ] Lighthouse mobile ≥ 90 en home, catálogo y ficha
- [ ] La web funciona bien en iPhone SE y en un Android de gama media
- [ ] JB carga un producto nuevo sin ayuda técnica
- [ ] Backup automático probado con una restauración real
- [ ] Emails transaccionales llegan a bandeja de entrada, no a spam

---

## FASE 2 · Crecimiento (6–8 semanas)

**Objetivo:** vender más con el mismo tráfico y reducir el trabajo manual.

| Sprint | Contenido |
|---|---|
| 7 | **Checkout Bricks** (pago sin salir del sitio), recuperación de carrito abandonado, cupones y promociones |
| 8 | **WhatsApp Business API**: confirmación, ajuste de peso, despacho y entrega automatizados |
| 9 | Favoritos, "repetir pedido" en un clic, listas de compra recurrente para B2B, recomendaciones |
| 10 | Google Merchant Center, Meta Pixel + API de Conversiones, PWA instalable con notificaciones push, blog SEO |

**Impacto esperado:** +25–40% de conversión respecto del cierre de la Fase 1.

---

## FASE 3 · Escala (8–10 semanas)

**Objetivo:** integrar la operación completa y sostener el crecimiento.

| Sprint | Contenido |
|---|---|
| 11 | **Facturación electrónica ARCA** (TusFacturas/Facturante): factura A y B automática, notas de crédito por ajuste de peso |
| 12 | Logística: integración con Andreani/OCA para insumos secos, seguimiento, optimización de rutas de reparto propio |
| 13 | Reseñas de productos, preguntas y respuestas, sistema de reclamos y devoluciones con logística inversa |
| 14 | Multi-depósito con asignación automática, búsqueda avanzada (Meilisearch), reportes ampliados |

---

## FASE 4 · Expansión (continuo)

- **Cuenta corriente B2B** con límite de crédito y vencimientos
- **Portal de distribuidores** con precios y condiciones propias
- **Suscripciones**: entrega recurrente semanal o mensual
- **Programa de fidelización** por volumen
- App nativa (sólo si la PWA muestra un techo real)
- Integración con ERP o sistema contable
- Marketplace: venta de productos de terceros

---

## Riesgos y mitigación

| Riesgo | Prob. | Impacto | Mitigación |
|---|---|---|---|
| **Contenido de JB que no llega a tiempo** | Alta | Alto | Fase 0 con entregables y fechas firmadas. Sesión de fotos agendada antes del sprint 2 |
| **Cambios de alcance a mitad de camino** | Alta | Alto | Alcance de Fase 1 congelado por escrito. Todo lo nuevo va a Fase 2 |
| **Integración con Mercado Pago más compleja de lo previsto** | Media | Alto | Prueba de concepto en el sprint 1, no en el 4. Es el riesgo técnico principal |
| **El modelo de peso variable no convence al cliente final** | Media | Alto | Validar el mensaje con 5 clientes reales en Fase 0, antes de construirlo |
| **Baja adopción: los clientes siguen usando WhatsApp** | Media | Alto | Plan de lanzamiento: descuento exclusivo web, respuesta de WhatsApp con link a la web, capacitación a clientes clave |
| **El equipo de JB no usa el panel** | Media | Alto | Capacitación en el sprint 6 + manual + acompañamiento las primeras 4 semanas |
| **Sobreventa de stock** | Baja | Alto | Reservas transaccionales + test de concurrencia obligatorio |
| **Costos de infraestructura mayores a lo previsto** | Baja | Medio | Alertas de consumo + plan de salida documentado |

---

## Después del lanzamiento

**Semanas 1 a 4:** soporte intensivo diario, corrección de errores en menos de
24 h, acompañamiento al equipo, revisión semanal de métricas.

**Mensual, en régimen:** actualizaciones de seguridad, monitoreo de rendimiento,
informe de métricas, backlog de mejoras priorizado con JB.

**Presupuesto de mantenimiento sugerido:** 10–15 h/mes.

---

## Qué medir desde el día uno

| Métrica | Herramienta | Objetivo mes 6 |
|---|---|---|
| Pedidos web / pedidos totales | Panel | ≥ 40% |
| Tasa de conversión | PostHog | 1,5–2,5% |
| Ticket promedio | Panel | ≥ ticket de WhatsApp |
| Clientes nuevos captados por la web | Panel | ≥ 30/mes |
| Recompra a 90 días | Panel | ≥ 30% |
| Tiempo de gestión por pedido | Medición interna | −60% |
| Core Web Vitals | Vercel Analytics | Todos en verde |
| Tasa de error | Sentry | < 0,1% de sesiones |

**La métrica que define el éxito del proyecto no es el tráfico: es el porcentaje
de pedidos que entran sin que nadie de JB conteste un mensaje.**
