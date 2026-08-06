# 08 — Flujo completo de compra

## 1. Vista general

```
DESCUBRIMIENTO → EXPLORACIÓN → CARRITO → CHECKOUT → PAGO
      → PREPARACIÓN → PESAJE Y AJUSTE → ENTREGA → POSVENTA
```

Los pasos 1 a 5 los hace el cliente. Los pasos 6 a 8 los hace JB. El paso 7 es el
que distingue esta plataforma de un e-commerce genérico.

---

## 2. Del catálogo al carrito

### Agregar un producto

```
Cliente toca [Agregar al carrito]
   │
   ├─ ¿Hay carrito?  No → crear carrito + cookie httpOnly (30 días)
   │
   ├─ Validaciones del servidor (nunca del navegador):
   │     · producto ACTIVE y no borrado
   │     · cantidad ≥ min_order_qty y múltiplo de qty_step
   │     · si wholesale_only → cliente mayorista aprobado
   │     · stock disponible ≥ cantidad  (o allow_backorder)
   │
   ├─ Resolver PRECIO en el servidor (pricing)
   │     lista del grupo → escala por volumen → descuento → promo
   │
   ├─ Si is_variable_weight:
   │     peso_estimado = avg_weight_grams × cantidad
   │     subtotal = precio_por_kg × peso_estimado / 1000
   │
   ├─ RESERVAR stock (transacción con bloqueo de fila)
   │     stock_items.reserved += cantidad
   │     carts.reserved_until = ahora + 20 min
   │
   ├─ Recalcular totales del carrito
   └─ Revalidar y abrir el drawer
```

**Por qué reservar desde el carrito:** en productos frescos con stock limitado
(y sobre todo en camadas), no reservar produce el peor error posible: confirmar
una venta que no se puede cumplir. La reserva vence a los 20 minutos y una tarea
programada la libera.

### Recálculo permanente
Cada vez que se abre el carrito se revalidan precios y stock. Si algo cambió, se
informa de forma explícita: *"El precio del pollo entero se actualizó a
$4.350/kg"* o *"Sólo quedan 12 unidades disponibles"*.

---

## 3. Checkout

```
[Finalizar compra]
   │
   ├─ Revalidar TODO el carrito (precios, stock, vigencia)
   │     algo cambió → volver al carrito con el detalle
   │
   ├─ BLOQUE 1 · CONTACTO
   │     email + teléfono · sin registro obligatorio
   │     si pide factura A → CUIT + razón social + condición IVA
   │
   ├─ BLOQUE 2 · ENTREGA
   │     ├─ Envío a domicilio
   │     │    CP → zona → métodos disponibles
   │     │    ⚠ si algún ítem requires_cold_chain:
   │     │        sólo métodos con supports_cold_chain
   │     │    elegir día y franja (delivery_slots con cupo)
   │     └─ Retiro en planta
   │          elegir día y horario, sin costo
   │
   ├─ BLOQUE 3 · PAGO
   │     ○ Mercado Pago     ○ Transferencia bancaria
   │
   └─ [Confirmar pedido]
         │
         ├─ Validación final del servidor (Zod + reglas de negocio)
         ├─ RECALCULAR el total completo desde cero
         │     el importe que llegó del navegador se IGNORA
         ├─ Crear ORDER (status PENDING_PAYMENT) + order_items
         │     congelando nombre, SKU y precio de cada ítem
         ├─ Convertir reserva de carrito → reserva de pedido
         ├─ Vaciar el carrito
         └─ Derivar según el medio de pago
```

---

## 4. Pago

### 4.1 Mercado Pago (Checkout Pro, Fase 1)

```
Se crea la PREFERENCIA en el servidor
  · items, importes, external_reference = order.id
  · back_urls: éxito / pendiente / error
  · notification_url = /api/webhooks/mercadopago
  · expiración: 30 min (alineada con la reserva de stock)
        │
        ▼
Redirección a Mercado Pago  →  el cliente paga
        │
        ├──────────────── camino A (confiable) ────────────────┐
        │  WEBHOOK  POST /api/webhooks/mercadopago             │
        │  1. Verificar firma HMAC          → si falla, 401    │
        │  2. ¿event_id ya procesado?       → si sí, 200 y fin │
        │  3. Consultar la API de MP el pago real              │
        │     (nunca se confía en el payload del webhook)      │
        │  4. Según el estado:                                 │
        │       approved  → confirmarPedido()                  │
        │       rejected  → liberar stock, avisar al cliente   │
        │       pending   → dejar PENDING, notificar           │
        │       refunded  → registrar reembolso                │
        │  5. Registrar en payment_events y responder 200      │
        └──────────────────────────────────────────────────────┘
        │
        └── camino B (sólo visual) ────────────────────────────┐
           Vuelve a /checkout/confirmacion/[orderNumber]        │
           NO se marca pagado desde acá: la redirección es      │
           manipulable. Si el webhook aún no llegó, se muestra  │
           "Estamos confirmando tu pago" y se consulta cada 3 s │
           ───────────────────────────────────────────────────-─┘
```

**Red de seguridad:** una tarea diaria consulta a Mercado Pago todos los pedidos
`PENDING_PAYMENT` de las últimas 48 h. Los webhooks a veces no llegan; la
conciliación evita perder ventas ya cobradas.

### 4.2 Transferencia bancaria

```
Confirma el pedido
   ↓
Pantalla + email con: CBU · alias · titular · CUIT · monto EXACTO
   ↓
Pedido en PENDING_PAYMENT · stock reservado 48 h (no 20 min)
   ↓
El cliente sube el comprobante (desde la confirmación o su cuenta)
   ↓
Estado → PAYMENT_IN_REVIEW · aviso interno al panel
   ↓
JB valida en el panel: [Aprobar] / [Rechazar + motivo]
   ↓
Aprobado → confirmarPedido()   Rechazado → email con el motivo
   ↓
Si a las 48 h no hay comprobante: recordatorio a las 24 h,
cancelación automática y liberación de stock a las 48 h
```

**Por qué es importante:** en tickets B2B altos, la comisión de Mercado Pago es
significativa. El descuento por transferencia (sugerido: 5%) beneficia a ambas
partes y es la forma de pago que este público ya usa.

### 4.3 `confirmarPedido()` — única función de confirmación

```
TRANSACCIÓN
  1. order.status → CONFIRMED, payment_status → PAID, paid_at
  2. Convertir reserva en descuento real de stock
       stock_items.quantity -= cantidad
       stock_items.reserved -= cantidad
       insertar stock_movements (type = SALE)
  3. Si hay camada → hatch_batches.reserved += cantidad
  4. Registrar en order_status_history
  5. Incrementar métricas (sales_count, totales del cliente)
COMMIT
       ↓
EVENTOS (asíncronos, fuera de la transacción)
  → email de confirmación
  → WhatsApp (Fase 2)
  → aviso interno
  → creación del envío
  → factura electrónica (Fase 3)
```

**Idempotente por diseño:** si el webhook llega dos veces, la segunda no hace
nada. Es la protección contra el doble descuento de stock.

---

## 5. El problema del peso variable

### El conflicto

| Realidad de JB | Restricción técnica |
|---|---|
| El pollo se vende por kilo | Mercado Pago necesita un importe fijo al cobrar |
| El peso real se conoce al preparar | El cobro ocurre antes de preparar |
| Un pollo puede pesar 2,2 o 2,6 kg | El cliente no acepta que le cobren distinto sin aviso |

### Las cuatro opciones evaluadas

| Opción | Cómo funciona | Veredicto |
|---|---|---|
| **A · Precio por unidad fijo** | Se vende "pollo entero $10.000", sin importar el peso | Simplísimo, pero JB pierde margen en los pesados o pierde clientes en los livianos. Sólo sirve si el proveedor entrega rangos muy parejos |
| **B · Estimado + ajuste** ✅ | Se cobra el estimado, se pesa, se ajusta la diferencia | **Recomendada.** Refleja la realidad del negocio y es transparente |
| **C · Preautorización y captura** | Se retiene el monto y se captura el real | Técnicamente elegante, pero el soporte de captura parcial en Mercado Pago es limitado y complica el flujo. No recomendada en Fase 1 |
| **D · Rangos de peso como variantes** | "Pollo 2,0–2,4 kg" a precio cerrado | Buen intermedio si el pesaje individual es viable antes de publicar. Alto costo operativo |

### Opción B en detalle — recomendada

```
COMPRA
  Cliente pide 20 pollos
  Peso estimado: 20 × 2.400 g = 48.000 g = 48 kg
  Total estimado: 48 × $4.200 = $201.600
  → SE COBRA $201.600 (+ envío)
  Se muestra en todo momento: "estimado, ±10%"

PREPARACIÓN (panel · pantalla de pesaje)
  El operario carga el peso real: 47.600 g
  El sistema calcula: 47,6 × $4.200 = $199.920
  Diferencia: −$1.680  (a favor del cliente)
      ↓
REGLA DE RESOLUCIÓN AUTOMÁTICA
  ┌────────────────────────────────────────────────────────┐
  │ Diferencia a FAVOR del cliente (pesó menos)            │
  │   · ≤ $2.000  → crédito para la próxima compra         │
  │                 (o reembolso si el cliente lo pide)    │
  │   · > $2.000  → reembolso parcial automático por MP    │
  │                                                        │
  │ Diferencia a favor de JB (pesó más)                    │
  │   · dentro de la tolerancia (±10%) y ≤ $2.000          │
  │             → se absorbe, NO se cobra                  │
  │   · > $2.000 → se pide autorización al cliente con     │
  │                link de pago de la diferencia.          │
  │                Sin respuesta en 24 h → se entrega el   │
  │                peso equivalente a lo pagado            │
  └────────────────────────────────────────────────────────┘
      ↓
NOTIFICACIÓN (siempre, aunque no haya diferencia)
  Email + WhatsApp con estimado, real, diferencia y resolución
      ↓
ENTREGA con el detalle impreso o digital
```

**Por qué se absorben las diferencias chicas a favor de JB:** cobrar $800 extra
genera un reclamo, una llamada y desconfianza que cuestan más que $800. Absorberlas
y comunicarlo ("si pesa un poco más, no te cobramos de más") es además un
argumento comercial fuerte. El umbral es configurable desde el panel.

**Decisión pendiente:** el umbral de $2.000 y la tolerancia del 10% son
propuestas. Ver documento 11.

---

## 6. Preparación y entrega

```
CONFIRMED
   ↓  JB toma el pedido
PREPARING
   ↓  se arma físicamente
   ├─ productos de peso variable → pantalla de pesaje → WEIGHED
   ├─ faltante de stock → sustituir (con aviso) o cancelar el ítem
   └─ registro del lote entregado (trazabilidad)
   ↓
READY
   ├─ Retiro en planta → aviso al cliente "ya podés retirar"
   └─ Reparto propio   → se asigna a la hoja de ruta del día
   ↓
IN_TRANSIT   (aviso con franja horaria)
   ↓
DELIVERED    (comprobante: foto o firma) + email de agradecimiento
```

### Hoja de ruta (panel)
Los pedidos `READY` con entrega en la fecha se agrupan por zona, se ordenan y
generan una hoja imprimible o vista mobile para el repartidor, con dirección,
teléfono, ítems, total y forma de pago.

---

## 7. Flujo de camadas (pollitos BB)

```
JB publica la camada: fecha de nacimiento, capacidad, límite, ventana de retiro
   ↓
Cliente reserva N pollitos  →  hatch_batches.reserved += N
   ↓
PAGO (mismo flujo)  →  pedido en estado SCHEDULED
   ↓
Al llegar el order_deadline → camada CLOSED, se cierra la reserva
   ↓
48 h antes del retiro → recordatorio automático
   ↓
Nacimiento → HATCHED → pedidos pasan a READY
   ↓
Retiro en la ventana definida → DELIVERED

Si la camada se cancela o falla:
   → reembolso automático + reasignación ofrecida a la camada siguiente
```

Este flujo es el que más fideliza al cliente B2B: quien reserva su camada con
JB planifica su producción con JB.

---

## 8. Cancelaciones y devoluciones

| Situación | Regla |
|---|---|
| Cliente cancela antes de preparar | Automático desde su cuenta. Reembolso total, stock liberado |
| Cliente cancela ya preparado | Requiere aprobación de JB. Producto fresco: puede no reembolsarse |
| JB cancela por falta de stock | Reembolso total automático + aviso + compensación sugerida |
| Producto en mal estado | Reclamo con foto desde la cuenta → evaluación → reembolso o reposición |
| Botón de arrepentimiento (10 días) | Obligatorio por ley. Con la salvedad legal de productos perecederos, que deben estar declarados en los términos |

---

## 9. Puntos de falla y cómo se manejan

| Falla | Manejo |
|---|---|
| El webhook de MP no llega | Conciliación diaria + consulta desde la pantalla de confirmación |
| El cliente cierra el navegador al pagar | El pedido existe; el webhook lo confirma igual y avisa por email |
| Dos clientes compran la última unidad | La reserva con bloqueo transaccional impide la sobreventa |
| El precio cambió durante la sesión | Revalidación en el checkout con aviso explícito |
| El email de confirmación falla | Encolado con reintentos; nunca bloquea la compra |
| Caída de Mercado Pago | Se sigue ofreciendo transferencia; aviso visible |
| Pedido pagado sin stock real | Aviso proactivo con opciones: esperar, sustituir o reembolsar |

---

## 10. Métricas del embudo

| Etapa | Métrica | Objetivo Fase 1 |
|---|---|---|
| Visita → catálogo | % que ve productos | > 60% |
| Catálogo → ficha | % de clic en producto | > 35% |
| Ficha → carrito | Tasa de agregado | > 12% |
| Carrito → checkout | % que inicia el checkout | > 45% |
| Checkout → pago | % que confirma | > 70% |
| **Global** | **Tasa de conversión** | **1,5–2,5%** |
| Posventa | Recompra a 90 días | > 30% |

Cada etapa se instrumenta con eventos desde el día uno. Sin medición no hay
optimización posible.
