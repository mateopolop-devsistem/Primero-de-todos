# 11 — Decisiones a validar

Este documento es el que necesita tu respuesta. Está dividido en **decisiones
bloqueantes** (sin ellas hay riesgo real de rehacer trabajo) y **decisiones
importantes** (se pueden asumir por defecto y ajustar después).

---

## A · Decisiones bloqueantes

### A1 · ¿Cómo se cobra el pollo? ⚠ la más importante

El pollo se vende por kilo, pero el peso real se conoce recién al preparar el
pedido. Hay que elegir cómo resolverlo:

| Opción | Cómo lo ve el cliente | Implicancia |
|---|---|---|
| **1. Precio cerrado por unidad** | "Pollo entero $10.000" | Lo más simple. JB absorbe la variación de peso |
| **2. Estimado + ajuste** ✅ recomendada | "$4.200/kg · ~2,4 kg ≈ $10.080. Se cobra el peso real" | Refleja el negocio real. Requiere pantalla de pesaje y notificaciones |
| **3. Rangos de peso** | "Pollo 2,0–2,4 kg · $9.240" | Precio cerrado sin perder margen, pero hay que clasificar por peso antes de despachar |

**Necesito saber:**
- ¿Cuánto varía en la práctica el peso de un pollo? (mínimo y máximo reales)
- ¿Hoy cómo se lo cobran a un cliente que compra 20 pollos?
- ¿Se puede pesar y clasificar antes de publicar, o el peso se sabe recién al armar el pedido?

---

### A2 · ¿Se venden pollitos BB por camada?

Todo el módulo de camadas (calendario, reservas, pedidos programados,
recordatorios) depende de esto. Es **la funcionalidad más diferencial de la
plataforma** para el público B2B, y también la más costosa de agregar después.

**Necesito saber:**
- ¿JB vende pollitos BB (bebé) o sólo pollo para consumo?
- Si vende: ¿con qué anticipación se reservan? ¿Cada cuánto hay camadas?
- ¿Cantidad mínima y múltiplos? (¿de a 100? ¿de a 50?)
- ¿Se entregan en planta o se reparten?

Si la respuesta es "no vendemos pollitos", el alcance de la Fase 1 baja
aproximadamente 1,5 sprints.

---

### A3 · ¿Cómo funcionan realmente los envíos?

Los pollos frescos requieren cadena de frío. Eso limita las opciones y hay que
codificarlo desde el inicio.

**Necesito saber:**
- ¿Reparto propio? ¿Con qué vehículos y en qué radio?
- ¿Qué días se reparte y a qué zonas?
- ¿Costo del envío: fijo, por zona, por monto, por distancia?
- ¿Hay envío gratis a partir de cierto monto?
- ¿Existe retiro en planta? ¿En qué horarios?
- ¿Los insumos secos (alimento, comederos) pueden ir por correo a otras provincias?
- ¿Hay pedido mínimo? ¿Distinto para minorista y mayorista?

---

### A4 · ¿Cómo se maneja el precio mayorista?

**Necesito saber:**
- ¿El precio mayorista es público o requiere cuenta aprobada?
  (recomendado: **cuenta aprobada** — protege el precio y genera una base de datos
  de clientes B2B)
- ¿Cuántos niveles de precio hay? (minorista / mayorista / distribuidor / otro)
- ¿El descuento es por porcentaje general o precio específico por producto?
- ¿Hay descuentos por cantidad dentro del mismo nivel? (ej.: 10+ unidades)
- ¿Hay productos que sólo se venden a mayoristas?
- ¿Cuál es el mínimo de compra mayorista?

---

### A5 · ¿Qué medios de pago y con qué condiciones?

**Necesito saber:**
- ¿JB ya tiene cuenta de Mercado Pago? ¿A nombre de la empresa (CUIT)?
- ¿Se ofrecen cuotas? ¿Con interés o sin interés?
- ¿Se hace descuento por transferencia? ¿Qué porcentaje? (sugerido: 5%)
- ¿Se acepta efectivo contra entrega? (agrega complejidad al reparto)
- ¿Los mayoristas pagan al contado o hay cuenta corriente? (la cuenta corriente
  está prevista para Fase 4, pero conviene saberlo desde ahora)

---

### A6 · ¿Se factura electrónicamente desde el día uno?

La facturación electrónica ARCA está planificada para la Fase 3, pero si es
obligatoria desde el inicio, sube a la Fase 1 y suma ~2 semanas.

**Necesito saber:**
- ¿Hoy cómo se factura? ¿Con qué sistema?
- ¿Se puede seguir facturando por fuera de la web en los primeros meses?
- ¿Qué proporción de las ventas es factura A (a responsables inscriptos)?
- ¿Los pollos llevan IVA 10,5% y los insumos 21%, se confirma?

---

## B · Decisiones importantes (con valor por defecto)

Si no hay respuesta, se avanza con la opción marcada y se ajusta después.

| # | Decisión | Por defecto |
|---|---|---|
| B1 | Umbral de ajuste por peso que se absorbe sin cobrar | **$2.000** |
| B2 | Tolerancia de peso informada al cliente | **±10%** |
| B3 | Diferencia a favor del cliente | Reembolso si supera el umbral; crédito si no |
| B4 | Tiempo de reserva de stock en el carrito | **20 minutos** |
| B5 | Plazo para pagar por transferencia | **48 horas** |
| B6 | Compra sin registro | **Sí, habilitada** |
| B7 | Precios mostrados a minoristas | **Con IVA incluido** |
| B8 | Precios mostrados a mayoristas | **Neto + IVA discriminado** |
| B9 | Idioma y moneda | Español (Argentina), ARS |
| B10 | Modo oscuro | Fase posterior |
| B11 | Reseñas de productos | Fase 3 |
| B12 | Blog | Fase 2 |

---

## C · Confirmación del stack técnico

La propuesta es **Next.js + PostgreSQL + Prisma + Vercel** (documento 03).

Sólo hace falta una decisión tuya si:

- **Ya existe un equipo o proveedor con otro stack.** Si JB ya trabaja con
  desarrolladores de PHP/Laravel o WordPress, conviene evaluarlo: el mejor stack
  es el que el equipo puede mantener.
- **Hay preferencia por una plataforma cerrada** (Tiendanube, Shopify). Es
  legítimo y más barato al inicio, pero **no soporta bien el peso variable ni las
  camadas**. Si la respuesta a A1 es "opción 1: precio cerrado por unidad" y a A2
  es "no vendemos pollitos", entonces Tiendanube pasa a ser una alternativa
  seriamente considerable y honestamente más económica.

**Esa dependencia es real y conviene decidirla temprano:** el modelo comercial
determina si hace falta una plataforma a medida o no.

---

## D · Información que necesito de JB para arrancar

| # | Qué | Formato | Bloquea |
|---|---|---|---|
| D1 | Listado completo de productos con precio, unidad de venta y peso | Excel/CSV | Sprint 2 |
| D2 | Fotos de productos (mínimo 1 por producto, ideal 3) | JPG/PNG alta resolución | Sprint 2 |
| D3 | Fotos de la planta, el equipo y el reparto | JPG/PNG | Sprint 6 |
| D4 | Logo en vectorial (SVG/AI) y colores institucionales | Archivo | Sprint 1 |
| D5 | Razón social, CUIT, domicilio fiscal, condición IVA | Texto | Sprint 6 |
| D6 | CBU, alias y titular de la cuenta | Texto | Sprint 4 |
| D7 | Credenciales de Mercado Pago (prueba y producción) | Acceso | Sprint 4 |
| D8 | Zonas de reparto, días y costos | Documento | Sprint 4 |
| D9 | Historia de la empresa, 2 o 3 párrafos | Texto | Sprint 6 |
| D10 | Dominio (¿ya se tiene? ¿cuál?) | Acceso al DNS | Sprint 1 |
| D11 | Teléfono y WhatsApp de atención + horarios | Texto | Sprint 1 |
| D12 | Redes sociales de JB | Enlaces | Sprint 6 |
| D13 | 3 a 5 clientes dispuestos a dar testimonio | Contactos | Sprint 6 |

---

## E · Preguntas de negocio que ayudan a priorizar

Estas no bloquean el desarrollo, pero cambian dónde se pone el esfuerzo:

1. ¿Cuántos pedidos por mes maneja JB hoy? ¿Y cuál es el ticket promedio?
2. ¿Qué proporción es minorista y qué proporción mayorista?
3. ¿Cuánto tiempo se dedica hoy a atender pedidos por WhatsApp?
4. ¿Cuál es el radio geográfico actual y hasta dónde se quiere llegar?
5. ¿Cuáles son los 10 productos que más se venden?
6. ¿Hay estacionalidad? (¿fiestas, verano, ciclos de producción?)
7. ¿Quiénes son los competidores y venden online?
8. ¿Hay un objetivo concreto de ventas para el primer año de la web?
9. ¿Quién de JB va a administrar la plataforma día a día? ¿Qué tan cómodo se
   siente con la tecnología?
10. ¿Hay presupuesto asignado para publicidad digital al lanzar? Sin tráfico, la
    mejor web del mundo no vende.

---

## Cómo responder

No hace falta un documento formal. Alcanza con contestar en línea, aunque sea
"no sé todavía" — eso también es información útil.

**Prioridad: A1, A2 y A3.** Con esas tres respuestas ya se puede cerrar el
alcance de la Fase 1 y empezar el diseño en Figma.
