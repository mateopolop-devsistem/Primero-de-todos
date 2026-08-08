# 08 — Flujo completo de compra

## 1. Vista general

```
DESCUBRIMIENTO → ASESORAMIENTO → ELECCIÓN DE FECHA → CARRITO
   → CHECKOUT (producto + logística) → PAGO
   → ESPERA (SCHEDULED) → NACIMIENTO → ENCAJONADO → DESPACHO
   → RETIRO / ENTREGA → CRIANZA (posventa)
```

Dos diferencias estructurales con un e-commerce común:

1. **Se compra una fecha futura, no un producto en stock.** El pedido vive
   semanas en estado `SCHEDULED`.
2. **JB casi nunca es quien transporta.** Despacha a una agencia y el cliente
   retira en destino, muchas veces pagando el flete ahí.

---

## 2. Del catálogo al carrito

### El asesor de compra (entrada alternativa al catálogo)

```
¿Para qué querés los pollitos?
  ○ Vender a carnicería / pollo de chacra
  ○ Vender a parrilladas
  ○ Producir huevos
  ○ Consumo propio / patio
        ↓
¿Cuántos pensás criar?     [ 50 · 100 · 500 · 1000+ ]
        ↓
¿Es tu primera vez?        [ Sí / No ]
        ↓
RECOMENDACIÓN
  · línea sugerida, con el motivo explicado en criollo
  · fechas de nacimiento disponibles
  · si es primera vez → kit de arranque
  · si es volumen alto → invitación a cuenta mayorista
```

La lógica sale de `chick_specs`: `target_market` y `crate_count` conectan la
pregunta comercial ("¿a quién le vendés?") con el producto correcto.

### Agregar un producto

```
Cliente toca [Reservar] o [Agregar]
   │
   ├─ ¿Hay carrito?  No → crear carrito + cookie httpOnly (30 días)
   │
   ├─ Validaciones del servidor (nunca del navegador):
   │     · producto ACTIVE y no borrado
   │     · cantidad >= min_order_qty (50) y múltiplo de qty_step (50)
   │     · si wholesale_only → mayorista aprobado
   │     · si es pollito → camada OPEN, no vencida, con cupo suficiente
   │     · si es insumo  → stock disponible
   │
   ├─ Resolver PRECIO en el servidor (pricing)
   │     lista del grupo → escala por cantidad → descuento → promo
   │
   ├─ RESERVAR (transacción con SELECT ... FOR UPDATE)
   │     pollitos → hatch_batches.reserved += cantidad
   │     insumos  → stock_items.reserved  += cantidad
   │     carts.reserved_until = ahora + 30 min
   │
   ├─ Recalcular totales
   └─ Abrir el drawer con la fecha comprometida bien visible
```

> **Por qué la reserva es innegociable acá:** el cupo de una camada es físico y
> finito. Comprometer 1.200 pollitos de una camada de 1.000 no se arregla con
> una disculpa: deja a un productor con el galpón vacío y una pérdida que no
> recupera. Es el peor error posible del sistema.

### Validación de mezcla de fechas
Si el carrito tiene pollitos de dos camadas distintas, se avisa de forma
explícita: *"Tu pedido tiene dos fechas de nacimiento (12/09 y 26/09). Se
despachan por separado."* Y se permite separarlo en dos pedidos.

---

## 3. Checkout

```
[Finalizar compra]
   │
   ├─ Revalidar TODO (precios, cupo de camada, vigencia)
   │     algo cambió → volver al carrito con el detalle
   │
   ├─ BLOQUE 1 · CONTACTO
   │     email + teléfono/WhatsApp · sin registro obligatorio
   │     si pide factura A → CUIT + razón social + condición IVA
   │
   ├─ BLOQUE 2 · CÓMO LO RECIBÍS          ← el bloque crítico
   │     ○ Retiro en planta
   │     ○ Reparto propio (Córdoba)
   │     ○ Despacho por transporte (resto del país)
   │
   ├─ BLOQUE 3 · PAGO
   │     ○ Mercado Pago     ○ Transferencia bancaria
   │
   └─ [Confirmar pedido]
         ├─ Validación final del servidor (Zod + reglas de negocio)
         ├─ RECALCULAR el total desde cero
         │     el importe que llegó del navegador se IGNORA
         ├─ Crear ORDER (PENDING_PAYMENT) + order_items
         │     congelando nombre, SKU, precio y fecha de nacimiento
         ├─ Convertir reserva de carrito → reserva de pedido
         └─ Derivar según el medio de pago
```

---

## 4. Envíos: el punto crítico

Un e-commerce común asume dos cosas que acá son falsas: que el vendedor controla
el envío, y que el total que se paga es el total que cuesta. JB despacha a todo
el país por transportes de terceros, y **el flete lo suele pagar el cliente al
retirar**.

### 4.1 Las tres modalidades

| Modalidad | Cómo funciona | Flete |
|---|---|---|
| **Retiro en planta** | El cliente viene en la ventana de retiro | Sin costo |
| **Reparto propio** | Recorridos por Córdoba, días fijos | Tarifa de JB, cobrada en el pedido |
| **Transporte / comisionista** | JB despacha a la agencia; el cliente retira en destino | **Habitualmente a cargo del destinatario** |

### 4.2 Cómo se resuelve el flete a pagar en destino

Es la decisión de diseño más delicada del checkout. La regla:

> **El total del pedido incluye sólo la mercadería. El flete se muestra como
> estimado, claramente separado, con quién lo paga y cuándo.**

```
┌──────────────────────────────────────────────────────┐
│ Despacho por transporte                              │
│                                                      │
│ Destino: La Banda, Santiago del Estero               │
│ Transporte: Expreso del Norte                        │
│ Sale: martes y viernes · Corte 18:00                 │
│ Retirás en: Agencia Belgrano 450                     │
│                                                      │
│ ─────────────────────────────────────────────────    │
│ Mercadería (lo que pagás ahora)        $XXX.XXX      │
│                                                      │
│ ⓘ Flete estimado $XX.XXX                             │
│   Lo pagás al transporte cuando retirás.             │
│   No está incluido en este total.                    │
│   El monto lo define el transporte.                  │
└──────────────────────────────────────────────────────┘
```

Tres reglas para que esto no genere reclamos:

1. **La palabra "estimado" siempre acompañada de quién define el monto.** JB no
   fija el flete y no puede garantizarlo.
2. **Se repite en tres lugares:** checkout, confirmación y email. La queja
   "nadie me dijo que el flete se pagaba aparte" sólo se evita repitiéndolo.
3. **Si JB tiene tarifa acordada con un transporte**, se marca
   `freight_payment = PREPAID`, se cobra en el pedido y desaparece toda la
   ambigüedad. Es preferible donde sea posible.

### 4.3 El cruce fecha de nacimiento × día de salida ⚠

La restricción central de la logística, y la que ningún e-commerce estándar
modela:

```
El pollito recién nacido tiene reserva de saco vitelino por ~72 h.
No puede esperar a que salga el transporte.

REGLA:
  camada.hatch_date + margen_operativo
      ⋂  carrier_destination.departure_days
      → fechas de despacho válidas

Ejemplo:
  Camada nace jueves 12/09
  Expreso del Norte a La Banda sale martes y viernes
  → Única opción viable: viernes 13/09
  → Si el cliente elige un transporte que sale el martes siguiente,
    el sistema lo BLOQUEA con explicación:
    "Ese transporte sale recién el 17/09, 5 días después del
     nacimiento. No podemos despachar pollitos con esa demora.
     Opciones: Expreso X (sale el 13) o la camada del 26/09."
```

Bloquear una venta imposible es más valioso que aceptarla: la alternativa es un
cliente que recibe una caja de pollitos muertos.

### 4.4 Selección de transporte en el checkout

```
Provincia [ Santiago del Estero ▾ ]   Ciudad [ La Banda ▾ ]
              ↓
Transportes disponibles a La Banda:

  ● Expreso del Norte      sale mar y vie · llega en 18 h
    Agencia Belgrano 450 · flete estimado $XX.XXX (pagás al retirar)
    ✓ Compatible con la camada del 12/09

  ○ Transporte Sur         sale lun · llega en 24 h
    ⚠ No compatible: sale 5 días después del nacimiento

  ○ No está mi ciudad / prefiero coordinar
    → deja el pedido en "logística a coordinar" y avisa al panel
```

La última opción es importante: el mapa de transportes nunca va a estar completo
desde el día uno. Se construye con el uso, y cada pedido "a coordinar" alimenta
la tabla `carrier_destinations`.

---

## 5. Pago

### 5.1 Mercado Pago (Checkout Pro, Fase 1)

```
Se crea la PREFERENCIA en el servidor
  · items, importes, external_reference = order.id
  · back_urls: éxito / pendiente / error
  · notification_url = /api/webhooks/mercadopago
  · expiración alineada con la reserva de cupo
        ▼
Redirección a Mercado Pago → el cliente paga
        │
        ├──────────── camino A (el confiable) ────────────────┐
        │  WEBHOOK  POST /api/webhooks/mercadopago            │
        │  1. Verificar firma HMAC        → si falla, 401     │
        │  2. ¿event_id ya procesado?     → si sí, 200 y fin  │
        │  3. Consultar la API de MP el pago real             │
        │     (nunca se confía en el payload del webhook)     │
        │  4. approved → confirmarPedido()                    │
        │     rejected → liberar cupo, avisar                 │
        │     pending  → mantener reserva, notificar          │
        │  5. Registrar en payment_events y responder 200     │
        └─────────────────────────────────────────────────────┘
        │
        └── camino B (sólo visual) ───────────────────────────┐
           Vuelve a /checkout/confirmacion/[orderNumber].      │
           NO se marca pagado desde acá: la redirección es     │
           manipulable. Si el webhook no llegó, se muestra     │
           "Estamos confirmando tu pago" y se consulta.        │
           ──────────────────────────────────────────────────-─┘
```

**Red de seguridad:** una tarea diaria consulta a Mercado Pago todos los pedidos
`PENDING_PAYMENT` de las últimas 48 h. Los webhooks a veces no llegan.

### 5.2 Transferencia bancaria

Es el medio dominante en B2B y el que evita la comisión en tickets altos.

```
Confirma el pedido
   ↓
Pantalla + email con: CBU · alias · titular · CUIT · monto EXACTO
   ↓
PENDING_PAYMENT · cupo reservado 48 h
   ↓
El cliente sube el comprobante
   ↓
PAYMENT_IN_REVIEW · aviso al panel
   ↓
JB valida: [Aprobar] / [Rechazar + motivo]
   ↓
Aprobado → confirmarPedido()
   ↓
Sin comprobante: recordatorio a las 24 h,
liberación automática del cupo a las 48 h
```

> **Ajuste por proximidad de la camada:** si faltan menos de 48 h para el
> `order_deadline`, el plazo de pago se acorta automáticamente. No tiene sentido
> retener cupo de una camada que cierra mañana.

### 5.3 `confirmarPedido()` — única función de confirmación

```
TRANSACCIÓN
  1. order.status → CONFIRMED → SCHEDULED, payment_status → PAID
  2. Convertir la reserva en compromiso firme
       pollitos: hatch_batches.reserved queda confirmado
       insumos:  stock_items.quantity -= cantidad
                 insertar stock_movements (type = SALE)
  3. Registrar en order_status_history
  4. Incrementar métricas
COMMIT
       ↓
EVENTOS (asíncronos, fuera de la transacción)
  → email de confirmación con fecha de nacimiento y datos de despacho
  → WhatsApp (Fase 2)
  → aviso interno
  → guía de crianza si es cliente primerizo
  → factura electrónica (Fase 3)
```

**Idempotente por diseño:** si el webhook llega dos veces, la segunda no hace
nada.

---

## 6. La espera: de la compra al nacimiento

Este período —que puede ser de 2 a 5 semanas— es donde un e-commerce común no
tiene nada que decir y donde JB puede diferenciarse.

```
SCHEDULED
   │
   ├─ Inmediato: confirmación con fecha comprometida
   ├─ Si es primerizo: guía "Cómo preparar la llegada del pollito"
   ├─ 7 días antes: "Preparate — temperatura, comederos, viruta"
   ├─ 48 h antes: recordatorio con transporte, agencia y horario
   │
   ▼
HATCHED (nació la camada)
   ├─ Se carga `actual_hatched`
   ├─ Si nacieron menos de los comprometidos → protocolo de faltante
   │     · se avisa proactivamente, no cuando el cliente reclama
   │     · opciones: completar con la camada siguiente, entrega parcial
   │       con reintegro proporcional, o reintegro total
   ▼
READY (contado y encajonado, con los pollitos de yapa por mortandad)
   ▼
DISPATCHED / IN_TRANSIT / PICKUP_READY
   ├─ Transporte → se carga el número de guía → WhatsApp + email al instante
   ├─ Reparto propio → hoja de ruta del día
   └─ Retiro → aviso "ya podés retirar"
   ▼
DELIVERED
```

**El aviso proactivo de faltante es una decisión de negocio, no técnica.** Un
productor al que le avisan con 5 días de anticipación reacomoda su plan. Uno que
se entera el día del retiro pierde el ciclo y no vuelve a comprar.

---

## 7. Posventa: mortandad y crianza

### Reclamo por mortandad

```
El cliente retira y encuentra animales muertos
   ↓
Desde su cuenta: [Reportar problema] en el pedido
   ↓
Formulario: cantidad + fotos + fecha y hora de retiro
   ↓
Se crea order_adjustment (type = MORTALITY_CLAIM)
   ↓
JB evalúa en el panel dentro de las 24 h
   ↓
Resolución: reposición en la próxima camada · crédito ·
            reintegro parcial · rechazo fundado
   ↓
Notificación con el motivo de la decisión
```

**Requisitos que hacen que esto funcione:** política escrita y visible **antes**
de comprar (plazo para reclamar, qué evidencia hace falta, qué cubre y qué no),
y el `mortality_bonus_pct` declarado desde el principio — *"enviamos un 3%
adicional sin cargo para cubrir la mortandad de viaje"*.

### Acompañamiento de la crianza

Secuencia automática por email y WhatsApp según los días transcurridos:
recepción, primera semana (temperatura), cambio de alimento, plan sanitario, y
—según `grow_out_days`— el aviso de recompra en el momento exacto del ciclo.

Esto no es marketing: es lo que baja la mortandad del cliente, y un cliente cuya
crianza sale bien vuelve a comprar.

---

## 8. Cancelaciones

| Situación | Regla |
|---|---|
| Cliente cancela antes del `order_deadline` | Automático. Reintegro total, cupo liberado |
| Cliente cancela después del cierre de camada | Requiere aprobación. El cupo ya está comprometido con la incubación |
| Cliente cancela después del nacimiento | Sin reintegro salvo excepción comercial. El animal ya existe |
| JB cancela por camada fallida | Reintegro total automático + prioridad en la camada siguiente |
| Nacieron menos de los comprometidos | Entrega parcial con reintegro proporcional, o pase a la camada siguiente, a elección del cliente |
| Botón de arrepentimiento (10 días) | Obligatorio por ley. Aplica antes del despacho; los términos deben declarar la excepción de animales vivos ya despachados |

---

## 9. Puntos de falla y cómo se manejan

| Falla | Manejo |
|---|---|
| El webhook de MP no llega | Conciliación diaria + consulta desde la confirmación |
| El cliente cierra el navegador al pagar | El pedido existe; el webhook lo confirma igual |
| Dos clientes toman el último cupo | Reserva con bloqueo transaccional |
| La camada nace incompleta | Protocolo de faltante con aviso proactivo |
| El transporte no sale ese día | Aviso + reprogramación + opción de reintegro |
| El cliente no retira de la agencia | Aviso a las 24 h y a las 48 h; el animal vivo no espera |
| El email de confirmación falla | Encolado con reintentos; nunca bloquea la compra |
| Caída de Mercado Pago | Se sigue ofreciendo transferencia; aviso visible |

---

## 10. Métricas del embudo

| Etapa | Métrica | Objetivo Fase 1 |
|---|---|---|
| Visita → catálogo o asesor | % que avanza | > 60% |
| Asesor iniciado → completado | % que termina las 3 preguntas | > 70% |
| Catálogo → ficha | % de clic | > 35% |
| **Consulta de transporte → carrito** | conversión del cliente del interior | > 25% |
| Ficha → carrito | tasa de agregado | > 12% |
| Carrito → checkout | % que inicia | > 45% |
| Checkout → pago | % que confirma | > 70% |
| **Global** | **tasa de conversión** | **1,5–2,5%** |
| Posventa | recompra dentro de 1,5 ciclos productivos | > 35% |

La consulta de transporte se instrumenta como evento propio: es el momento de
verdad del cliente del interior, y su tasa de abandono dice si el problema es el
precio del flete o la falta de cobertura.
