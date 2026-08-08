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
| **1 · MVP vendible** | 9–11 sem | Vender online de punta a punta | **En producción, facturando** |
| **2 · Crecimiento** | 6–8 sem | Más conversión, menos trabajo manual | Optimizado |
| **3 · Escala** | 8–10 sem | Facturación, logística, fidelización | Operación integrada |
| **4 · Expansión** | continuo | B2B avanzado, app, nuevos canales | Plataforma madura |

**Salida a producción: entre la semana 10 y la 13 desde el inicio.**

---

## FASE 0 · Definición (1–2 semanas)

**No se escribe código de producto.** Es la fase que más plata ahorra.

### Entregables

| # | Entregable | Responsable |
|---|---|---|
| 0.1 | Decisiones del documento 11 respondidas | **JB** |
| 0.2 | Líneas que se venden, con precio y escalas por cantidad | **JB** |
| 0.2b | **Calendario de nacimientos de los próximos 3 meses** | **JB** |
| 0.3 | Fotos reales de pollitos, planta de incubación y encajonado | **JB** |
| 0.4 | Identidad de marca (logo, colores) o mínimo viable | JB / Diseño |
| 0.5 | Datos fiscales, bancarios, cuenta de Mercado Pago | **JB** |
| 0.6 | **Transportes y comisionistas: ciudades, agencias, días de salida** | **JB** |
| 0.6b | Zonas y días de reparto propio en Córdoba | **JB** |
| 0.7 | Textos legales + **política de mortandad** + plan sanitario | Desarrollo + JB |
| 0.8 | Diseño en Figma: home, catálogo, ficha, carrito, checkout (mobile + desktop) | Diseño |
| 0.9 | Repositorio, entornos, CI/CD, dominio | Desarrollo |

### Criterio de salida
Diseño aprobado por JB **y** las preguntas abiertas del documento 11 cerradas
por escrito — sobre todo **B1 (cómo funcionan los nacimientos)**, que define si
se construye plataforma propia o conviene evaluar una tienda enlatada.

> **Sin fotos reales de pollitos, la fase 1 no arranca.** Es el activo que más
> impacta la conversión y no se puede reemplazar con banco de imágenes. Quien le
> compra a distancia a alguien que no conoce necesita ver el producto real.

---

## FASE 1 · MVP vendible (9–11 semanas)

**Objetivo:** que un productor reserve pollitos online, pague, y reciba el aviso
de despacho con su número de guía, sin que nadie de JB intervenga manualmente.

> **Se acortó ~1 semana** respecto de la versión anterior del plan: al vender por
> unidad a precio cerrado desaparece toda la maquinaria de peso variable. Ese
> tiempo se reinvierte en camadas y transportes, que son los módulos que
> realmente diferencian a JB.

### Sprint 1 — Cimientos (sem. 1–2)
- Next.js + TypeScript + Tailwind + design system base
- Prisma + esquema completo + migraciones + seed
- Auth.js (registro, login, recuperación de clave)
- Layout: header, footer, navegación mobile
- Middleware de rutas protegidas, Sentry, CI
- **Prueba de concepto de Mercado Pago** (adelantada a propósito: es el riesgo
  técnico principal y conviene descubrirlo en la semana 2, no en la 7)
- **Demo:** navegar el esqueleto, registrarse, iniciar sesión

### Sprint 2 — Catálogo y camadas (sem. 3–4) ★
- Módulo `catalog`: productos, `chick_specs`, categorías, atributos, imágenes
- **Módulo `hatchery`: camadas, cupo, cierre, reservas transaccionales**
- **Calendario de nacimientos** (público) + selector de camada en la ficha
- Listado con filtros (incluido filtro por fecha) y búsqueda full-text
- Admin: alta de productos y **alta de camadas**
- **Demo:** JB carga sus líneas reales y sus próximas camadas, y se ven en la web

### Sprint 3 — Precios, carrito y asesor (sem. 5–6)
- Módulo `pricing`: listas, grupos, **escalas por cantidad**, IVA
- Validación de mínimos y múltiplos (50) + cálculo de yapa por mortandad
- Carrito del lado del servidor + drawer + aviso de próximo escalón de precio
- Módulo `inventory` para insumos + kits de arranque
- **Asesor de compra** (3 preguntas → línea recomendada)
- **Demo:** armar un pedido de 500 con el precio por volumen correcto y el cupo
  reservado

### Sprint 4 — Transportes, checkout y pagos (sem. 7–8) ⚠ el más riesgoso
- **Módulo `dispatch`: transportes, agencias de destino, días de salida**
- **Cruce fecha de nacimiento × día de salida** (bloqueo de combinaciones
  inviables)
- Consulta "¿mandás a mi ciudad?" en ficha y página de envíos
- Checkout completo, con el aviso de flete a cargo del destinatario
- **Mercado Pago Checkout Pro** + webhook + idempotencia + conciliación
- Transferencia bancaria + subida y validación de comprobante
- Creación del pedido, máquina de estados, emails transaccionales
- **Demo:** compra de punta a punta desde otra provincia, en modo prueba

### Sprint 5 — Panel operativo (sem. 9–10) ★
- Dashboard con las colas de atención
- **Camadas: detalle, cierre, registrar nacimiento, protocolo de faltante**
- **Despachos agrupados por transporte + carga de número de guía**
- Hoja de ruta para el reparto propio de Córdoba
- Pedidos: tablero, tabla, detalle, carga manual
- Clientes, aprobación de mayoristas, validación de transferencias
- Cuenta del cliente: pedidos, seguimiento, datos de despacho
- **Demo:** JB opera un día de nacimiento completo desde el panel

### Sprint 6 — Cierre y salida a producción (sem. 11)
- Home definitiva con CMS de secciones y banners
- **Guías de crianza** (5 iniciales) — motor de SEO
- Páginas legales, política de mortandad, botón de arrepentimiento
- SEO: metadatos, sitemap, datos estructurados, Open Graph
- Rendimiento, accesibilidad WCAG AA, tests E2E (Playwright)
- Carga del catálogo completo + primeros 10 destinos de transporte
- **Capacitación del equipo de JB + manual de uso**
- Revisión de seguridad
- **Salida a producción**

### Criterios de aceptación de la Fase 1

- [ ] Un productor reserva una camada con Mercado Pago sin intervención humana
- [ ] Un cliente compra por transferencia y JB la valida en el panel
- [ ] **El cupo de una camada nunca se sobrevende** (probado con reservas
      concurrentes). Criterio innegociable
- [ ] **El sistema bloquea un transporte que sale días después del nacimiento**,
      con la explicación a la vista
- [ ] JB registra un nacimiento incompleto y los clientes afectados reciben el
      aviso automáticamente
- [ ] Al cargar un número de guía, el cliente lo recibe por email
- [ ] Un cliente del interior consulta su ciudad y obtiene transporte, agencia y
      días de salida
- [ ] Un mayorista se registra, es aprobado y ve precios por volumen
- [ ] Lighthouse mobile ≥ 90 en home, catálogo y ficha
- [ ] La web funciona bien en un Android de gama media con señal regular
- [ ] JB carga una camada nueva sin ayuda técnica
- [ ] Backup automático probado con una restauración real
- [ ] Emails transaccionales llegan a bandeja de entrada, no a spam

---

## FASE 2 · Crecimiento (6–8 semanas)

**Objetivo:** vender más con el mismo tráfico y reducir el trabajo manual.

| Sprint | Contenido |
|---|---|
| 7 | **Checkout Bricks** (pago sin salir del sitio), recuperación de carrito abandonado, cupones y promociones |
| 8 | **WhatsApp Business API**: confirmación, recordatorio de despacho, número de guía y seguimiento de crianza automatizados |
| 9 | "Repetir pedido" en un clic, **recompra automática por ciclo productivo**, listas recurrentes B2B, favoritos |
| 10 | Google Merchant Center, Meta Pixel + API de Conversiones, PWA instalable con notificaciones push, blog SEO |

**Impacto esperado:** +25–40% de conversión respecto del cierre de la Fase 1.

---

## FASE 3 · Escala (8–10 semanas)

**Objetivo:** integrar la operación completa y sostener el crecimiento.

| Sprint | Contenido |
|---|---|
| 11 | **Facturación electrónica ARCA** (TusFacturas/Facturante): factura A y B automática, notas de crédito por faltantes y reclamos |
| 12 | Logística: mapa completo de transportes, integración con Andreani/OCA **sólo para insumos secos**, optimización de rutas de reparto propio |
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
| **Los nacimientos no funcionan como se asumió** | Media | Alto | Es la pregunta B1 del documento 11. Bloquea el sprint 2 y hay que cerrarla en Fase 0 |
| **El mapa de transportes no se puede armar a tiempo** | Media | Alto | No hace falta que esté completo: se arranca con los 10 destinos más frecuentes y la opción "no está mi ciudad", y se completa con el uso |
| **Los clientes del interior no confían en pagar por adelantado** | Media | Alto | Política de mortandad visible, fotos reales, testimonios de productores de otras provincias, y transferencia como alternativa a la tarjeta |
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
| **Ocupación de camadas** | Panel | ≥ 85% del cupo vendido |
| **Ventas fuera de Córdoba** | Panel | ≥ 35% |
| Tasa de conversión | PostHog | 1,5–2,5% |
| Ticket promedio | Panel | ≥ ticket de WhatsApp |
| Clientes nuevos captados por la web | Panel | ≥ 30/mes |
| Recompra dentro de 1,5 ciclos productivos | Panel | ≥ 35% |
| Tiempo de gestión por pedido | Medición interna | −60% |
| Core Web Vitals | Vercel Analytics | Todos en verde |
| Tasa de error | Sentry | < 0,1% de sesiones |

**La métrica que define el éxito del proyecto no es el tráfico: es el porcentaje
de pedidos que entran sin que nadie de JB conteste un mensaje.**
