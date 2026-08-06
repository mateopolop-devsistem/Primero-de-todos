# 04 — Diseño de la base de datos

## Reglas transversales

1. **Dinero en enteros.** Todos los importes son `Int` en **centavos de peso**
   (`price_cents = 1250000` → $12.500,00). Nunca `Float`. Los decimales flotantes
   producen diferencias de centavos que aparecen meses después en la conciliación
   de Mercado Pago.
2. **Peso en enteros.** Todos los pesos son `Int` en **gramos**. Mismo motivo.
3. **Identificadores.** `cuid2` para claves primarias (no revelan volumen de
   ventas ni permiten enumerar pedidos ajenos). Los pedidos además llevan un
   `order_number` legible y secuencial (`JB-2026-00042`) para hablar por teléfono.
4. **Borrado lógico.** Productos, clientes y pedidos nunca se borran físicamente:
   `deleted_at`. Un pedido borrado destruye el historial contable.
5. **Auditoría.** Toda escritura administrativa que toque precio, stock o estado
   de pedido queda registrada en `audit_logs` con usuario, antes y después.
6. **Precios históricos congelados.** `order_items` guarda copia del nombre, SKU
   y precio del producto al momento de la compra. Si mañana cambia el precio, el
   pedido viejo **no cambia**.

---

## Mapa de entidades

```
                    ┌──────────────┐
                    │  categories  │──┐ (jerarquía propia)
                    └──────┬───────┘  │
                           │          │
      ┌────────────────────▼──────────▼─┐
      │            products             │
      │  sale_unit, is_variable_weight  │
      └──┬─────────────┬────────────┬───┘
         │             │            │
   ┌─────▼─────┐ ┌─────▼──────┐ ┌───▼─────────────┐
   │ variants  │ │  images    │ │ product_attrs   │
   └─────┬─────┘ └────────────┘ └─────────────────┘
         │
    ┌────┴──────────┬──────────────────┬──────────────┐
    ▼               ▼                  ▼              ▼
┌────────┐  ┌───────────────┐  ┌──────────────┐  ┌──────────┐
│ stock  │  │price_list_    │  │  cart_items  │  │hatch_    │
│ _items │  │   items       │  │              │  │ batches  │
└───┬────┘  └───────┬───────┘  └──────┬───────┘  └────┬─────┘
    │               │                 │               │
┌───▼────────┐ ┌────▼────────┐  ┌─────▼─────┐         │
│ stock_     │ │ price_lists │  │   carts   │         │
│ movements  │ └────┬────────┘  └─────┬─────┘         │
└────────────┘      │                 │               │
                ┌───▼──────────┐      │               │
                │customer_     │      │               │
                │  groups      │      │               │
                └───┬──────────┘      │               │
                    │                 │               │
              ┌─────▼─────┐           │               │
              │ customers │───────────┘               │
              └─────┬─────┘                           │
                    │                                 │
        ┌───────────▼──────────────────────────┐      │
        │              orders                  │◄─────┘
        │  total_estimated / total_final       │
        └──┬────────┬──────────┬────────┬──────┘
           │        │          │        │
    ┌──────▼──┐ ┌───▼────┐ ┌───▼────┐ ┌─▼──────────┐
    │ order_  │ │payments│ │shipment│ │order_status│
    │ items   │ │        │ │        │ │ _history   │
    └─────────┘ └────────┘ └────────┘ └────────────┘
```

---

## 1. Identidad y clientes

### `users`
Credenciales y acceso. Separado de `customers` a propósito: un usuario admin no
es un cliente.

| Campo | Tipo | Nota |
|---|---|---|
| id | String PK | |
| email | String unique | |
| password_hash | String? | null si entra por Google |
| name | String | |
| phone | String? | |
| role | Enum | `CUSTOMER`, `ADMIN`, `MANAGER`, `WAREHOUSE`, `SALES` |
| email_verified_at | DateTime? | |
| last_login_at | DateTime? | |
| is_active | Boolean | |
| created_at / updated_at | DateTime | |

`accounts`, `sessions`, `verification_tokens`: tablas estándar de Auth.js.

### `customers`
Perfil comercial. Un usuario tiene como máximo un customer.

| Campo | Tipo | Nota |
|---|---|---|
| id | String PK | |
| user_id | String? unique FK | null = compró como invitado |
| type | Enum | `RETAIL`, `WHOLESALE` |
| status | Enum | `ACTIVE`, `PENDING_APPROVAL`, `SUSPENDED` |
| group_id | String? FK → customer_groups | determina la lista de precios |
| business_name | String? | razón social |
| tax_id | String? | CUIT/CUIL |
| tax_condition | Enum? | `RESPONSABLE_INSCRIPTO`, `MONOTRIBUTO`, `EXENTO`, `CONSUMIDOR_FINAL` |
| business_type | Enum? | `GRANJA`, `AGROPECUARIA`, `VETERINARIA`, `REVENDEDOR`, `EMPRENDEDOR`, `OTRO` |
| email / phone / whatsapp | String | |
| default_address_id | String? | |
| credit_limit_cents | Int? | Fase 4 — cuenta corriente |
| notes | Text? | uso interno |
| total_orders / total_spent_cents | Int | desnormalizado para el panel |
| last_order_at | DateTime? | |
| accepts_marketing | Boolean | Ley 25.326 |
| created_at / updated_at / deleted_at | DateTime | |

> **Clave de negocio:** un mayorista se registra con `status = PENDING_APPROVAL`
> y ve precios minoristas hasta que JB lo aprueba. Evita que cualquiera acceda
> al precio mayorista con sólo crear una cuenta.

### `customer_groups`
`id`, `name` ("Minorista", "Mayorista", "Distribuidor", "Granja grande"),
`price_list_id`, `discount_percent`, `min_order_cents`, `is_default`, `priority`.

### `addresses`
`id`, `customer_id`, `label` ("Depósito", "Casa"), `recipient_name`, `phone`,
`street`, `number`, `apartment`, `city`, `province`, `postal_code`,
`shipping_zone_id?`, `lat/lng?`, `notes` (referencias para el repartidor),
`is_default`.

---

## 2. Catálogo

### `categories`
Jerarquía por auto-referencia (`parent_id`). Dos raíces principales: **Pollos** e
**Insumos avícolas**.

`id`, `parent_id?`, `name`, `slug` unique, `description?`, `image_url?`,
`icon?`, `position`, `is_active`, `show_in_menu`, `seo_title?`,
`seo_description?`, `product_count` (desnormalizado).

Estructura inicial propuesta:

```
Pollos
├── Pollo entero              (fresco / congelado)
├── Cortes                    (pechuga, pata muslo, alas, menudencias)
├── Pollitos BB               (parrillero, ponedora — VENTA POR CAMADA)
├── Pollas / recría
└── Huevos                    (si aplica)

Insumos avícolas
├── Alimento balanceado       (iniciador, crecimiento, terminador, ponedora)
├── Sanidad                   (vacunas, antibióticos, vitaminas, desinfectantes)
├── Equipamiento              (comederos, bebederos, criadoras, campanas)
├── Cama y sustrato
└── Accesorios
```

### `products`
El corazón del modelo. Los campos marcados **⚠** son los que un e-commerce
genérico no tiene y JB sí necesita.

| Campo | Tipo | Nota |
|---|---|---|
| id | String PK | |
| sku | String unique | |
| name | String | |
| slug | String unique | |
| short_description | String? | va en la tarjeta y en el meta description |
| description | Text? | rich text |
| category_id | String FK | |
| brand? | String | relevante en insumos |
| **product_type** ⚠ | Enum | `SIMPLE`, `VARIABLE_WEIGHT`, `HATCH_PREORDER`, `BUNDLE` |
| **sale_unit** ⚠ | Enum | `UNIT`, `KG`, `BOX`, `BAG` |
| **is_variable_weight** ⚠ | Boolean | true → el precio final depende del pesaje |
| **price_per_kg_cents** ⚠ | Int? | obligatorio si `sale_unit = KG` |
| **avg_weight_grams** ⚠ | Int? | peso promedio por unidad, para estimar |
| **weight_tolerance_pct** ⚠ | Int | default 10. Rango informado al cliente |
| base_price_cents | Int | precio de lista minorista |
| compare_at_price_cents | Int? | precio tachado para promociones |
| cost_cents | Int? | interno, para el reporte de margen |
| **vat_rate** ⚠ | Enum | `VAT_10_5` (carne aviar), `VAT_21` (insumos), `VAT_0` |
| **requires_cold_chain** ⚠ | Boolean | bloquea envío por correo, exige reparto propio |
| **min_order_qty** ⚠ | Int | ej.: pollitos por múltiplos de 100 |
| **qty_step** ⚠ | Int | incremento permitido |
| max_order_qty | Int? | tope por pedido cuando hay stock escaso |
| **shelf_life_days** ⚠ | Int? | vida útil, para gestión de lotes |
| **storage_type** ⚠ | Enum | `AMBIENT`, `REFRIGERATED`, `FROZEN` |
| track_inventory | Boolean | |
| allow_backorder | Boolean | |
| status | Enum | `DRAFT`, `ACTIVE`, `PAUSED`, `ARCHIVED` |
| is_featured / is_new | Boolean | |
| **wholesale_only** ⚠ | Boolean | producto visible sólo para mayoristas aprobados |
| seo_title / seo_description | String? | |
| search_vector | tsvector | índice GIN de búsqueda |
| view_count / sales_count | Int | para ordenar por popularidad |
| created_at / updated_at / deleted_at | DateTime | |

### `product_variants`
Presentaciones del mismo producto: bolsa de 25 kg vs. 40 kg, caja de 100 pollitos
vs. 500, macho vs. hembra.

`id`, `product_id`, `sku` unique, `name`, `option_values` (JSONB), `price_cents?`
(anula el del producto), `price_per_kg_cents?`, `avg_weight_grams?`,
`barcode?`, `image_id?`, `position`, `is_active`.

### `product_images`
`id`, `product_id`, `variant_id?`, `url`, `public_id` (Cloudinary), `alt`
(obligatorio, accesibilidad + SEO), `width`, `height`, `blur_hash`, `position`,
`is_primary`.

### `product_attributes`
Ficha técnica flexible sin migraciones: `id`, `product_id`, `key`
("Peso promedio", "Edad", "Composición", "Presentación"), `value`, `unit?`,
`group?`, `position`, `is_filterable`.

---

## 3. Precios

### `price_lists`
`id`, `name`, `currency` (ARS), `is_default`, `valid_from?`, `valid_to?`,
`is_active`.

### `price_list_items`
`id`, `price_list_id`, `product_id`, `variant_id?`, `price_cents`,
`price_per_kg_cents?`, `min_qty` (para escalas por volumen), `is_active`.

> **Precio por volumen:** varias filas con distinto `min_qty` sobre el mismo
> producto habilitan "de 1 a 9 unidades $X; de 10 a 49 $Y; 50+ $Z", que es
> exactamente lo que un revendedor espera ver.

### Algoritmo de resolución de precio (única fuente de verdad)

```
precioFinal(producto, variante, cantidad, cliente):
  1. lista = cliente.grupo.lista_de_precios ?? lista_por_defecto
  2. item = price_list_items donde coincida producto/variante
            y min_qty <= cantidad, tomando el min_qty más alto
  3. precio = item?.price_cents ?? variante.price_cents ?? producto.base_price_cents
  4. si producto.is_variable_weight:
        precio = precio_por_kg * peso_estimado_gramos / 1000
  5. aplicar descuento del grupo (customer_groups.discount_percent)
  6. aplicar promoción vigente (la mejor, no acumulables por defecto)
  7. aplicar cupón si corresponde
  8. si cliente es RESPONSABLE_INSCRIPTO → mostrar neto + IVA discriminado
     si no → mostrar precio final con IVA incluido
```

Este algoritmo vive **una sola vez**, en `modules/pricing`.

### `coupons`
`id`, `code` unique, `type` (`PERCENT`, `FIXED`, `FREE_SHIPPING`), `value`,
`min_order_cents?`, `max_discount_cents?`, `usage_limit?`, `usage_count`,
`per_customer_limit?`, `applies_to` (JSONB: categorías/productos),
`customer_group_ids?`, `starts_at`, `ends_at`, `is_active`.

---

## 4. Inventario

### `warehouses`
`id`, `name` ("Planta", "Depósito centro"), `address`, `is_active`,
`allows_pickup`.

### `stock_items`
| Campo | Nota |
|---|---|
| id, product_id, variant_id?, warehouse_id | |
| `quantity` | físico total |
| `reserved` | comprometido por carritos y pedidos sin despachar |
| `available` | columna generada: `quantity - reserved` |
| `low_stock_threshold` | dispara alerta en el panel |
| `updated_at` | |

Índice único `(product_id, variant_id, warehouse_id)`.

> **Regla anti-sobreventa:** toda modificación de `reserved` ocurre dentro de una
> transacción con `SELECT ... FOR UPDATE` sobre la fila. Es la única forma de
> evitar vender el mismo lote dos veces cuando entran dos pedidos simultáneos.

### `stock_movements`
Libro mayor de stock, sólo inserciones. Nunca se edita ni se borra.

`id`, `stock_item_id`, `type` (`PURCHASE`, `PRODUCTION`, `SALE`, `RETURN`,
`ADJUSTMENT`, `LOSS`, `TRANSFER`, `RESERVATION`, `RELEASE`), `quantity` (con
signo), `quantity_before`, `quantity_after`, `reference_type`, `reference_id`,
`batch_id?`, `reason?`, `user_id?`, `created_at`.

`LOSS` (mortandad, merma, producto vencido) es especialmente relevante en este
rubro y alimenta el reporte de pérdidas.

### `batches` — lotes
Trazabilidad de producto fresco: `id`, `product_id`, `batch_code`,
`production_date`, `expiry_date`, `quantity_initial`, `quantity_remaining`,
`supplier?`, `notes?`.

Permite responder "¿a qué clientes les vendimos el lote del 12/03?" — requisito
sanitario real en alimentos.

### `hatch_batches` — camadas de pollitos BB ⚠
La venta de pollitos no es venta de stock: es **preventa contra una fecha de
nacimiento**.

| Campo | Nota |
|---|---|
| id, product_id | ej.: "Pollito BB parrillero" |
| `hatch_date` | fecha de nacimiento programada |
| `capacity` | cantidad total de la camada |
| `reserved` | ya vendido |
| `available` | generada |
| `status` | `PLANNED`, `OPEN`, `CLOSED`, `HATCHED`, `DELIVERED`, `CANCELLED` |
| `order_deadline` | fecha límite de reserva |
| `pickup_from`, `pickup_to` | ventana de retiro/entrega |
| `price_cents?` | precio propio de la camada |
| `notes` | |

El cliente elige la camada como si eligiera una variante, y el pedido queda
`SCHEDULED` hasta la fecha. Modelar esto desde el día uno evita reescribir el
checkout entero en la Fase 3.

---

## 5. Carrito

### `carts`
`id`, `customer_id?`, `session_token` (cookie httpOnly para invitados),
`price_list_id`, `currency`, `subtotal_cents`, `discount_cents`,
`shipping_cents`, `total_cents`, `coupon_id?`, `expires_at`, `reserved_until?`,
`created_at`, `updated_at`.

**Carrito del lado del servidor**, no en `localStorage`. Motivos: el mayorista
arma el pedido en la computadora y lo cierra desde el celular; se puede recuperar
un carrito abandonado por email; y el precio no se puede manipular desde el
navegador.

### `cart_items`
`id`, `cart_id`, `product_id`, `variant_id?`, `hatch_batch_id?`, `quantity`,
`unit_price_cents`, `estimated_weight_grams?`, `subtotal_cents`, `added_at`.

---

## 6. Pedidos

### `orders`

| Campo | Nota |
|---|---|
| id | |
| `order_number` | `JB-2026-00042`, secuencial y legible |
| customer_id?, email, phone | |
| `status` | ver máquina de estados abajo |
| `payment_status` | `PENDING`, `IN_REVIEW`, `PAID`, `PARTIALLY_REFUNDED`, `REFUNDED`, `FAILED` |
| `fulfillment_status` | `UNFULFILLED`, `PREPARING`, `READY`, `IN_TRANSIT`, `DELIVERED` |
| **subtotal_estimated_cents** ⚠ | lo que se cobra al confirmar |
| **subtotal_final_cents** ⚠ | tras el pesaje real |
| discount_cents, shipping_cents, tax_cents | |
| **total_estimated_cents / total_final_cents** ⚠ | |
| **weight_adjustment_cents** ⚠ | diferencia con signo |
| **adjustment_status** ⚠ | `NOT_APPLICABLE`, `PENDING`, `CHARGED`, `REFUNDED`, `CREDITED` |
| currency, price_list_id | |
| shipping_method_id?, shipping_address (JSONB congelado) | |
| billing_address (JSONB), tax_id?, tax_condition? | |
| `delivery_date?`, `delivery_slot?` | |
| `scheduled_for?` | pedidos de camada |
| customer_notes?, internal_notes? | |
| `source` | `WEB`, `WHATSAPP`, `PHONE`, `ADMIN` — el panel también carga pedidos manuales |
| ip_address?, user_agent? | antifraude |
| confirmed_at?, paid_at?, prepared_at?, delivered_at?, cancelled_at? | |
| cancellation_reason? | |
| created_at / updated_at | |

### `order_items`

`id`, `order_id`, `product_id?`, `variant_id?`, `hatch_batch_id?`,
`product_name` (copia), `product_sku` (copia), `variant_name?`,
`quantity`, `unit_price_cents`, `price_per_kg_cents?`,
**`estimated_weight_grams?`**, **`actual_weight_grams?`**,
`subtotal_estimated_cents`, `subtotal_final_cents?`, `vat_rate`,
`vat_amount_cents`, `batch_id?`, `fulfilled_quantity`, `notes?`.

### Máquina de estados del pedido

```
                        ┌──────────────────┐
                        │  PENDING_PAYMENT │◄── se crea la orden
                        └────────┬─────────┘
              transferencia ┌────┴────┐ Mercado Pago
                            ▼         ▼
                  ┌──────────────┐  ┌──────────────┐
                  │ PAYMENT_IN_  │  │  CONFIRMED   │
                  │   REVIEW     │  │   (pagado)   │
                  └──────┬───────┘  └──────┬───────┘
                 aprobado│                 │
                         └────────┬────────┘
                                  ▼
                          ┌───────────────┐
                          │   PREPARING   │  ← se arma el pedido
                          └───────┬───────┘
                                  ▼
                     ┌─────────────────────────┐
                     │  WEIGHED  (peso real)   │ ⚠ sólo peso variable
                     │  → genera ajuste ±      │
                     └───────────┬─────────────┘
                                 ▼
                          ┌──────────────┐
                          │    READY     │  retiro o despacho
                          └──────┬───────┘
                                 ▼
                          ┌──────────────┐
                          │  IN_TRANSIT  │
                          └──────┬───────┘
                                 ▼
                          ┌──────────────┐
                          │  DELIVERED   │
                          └──────────────┘

Estados terminales alternativos: CANCELLED · REFUNDED · FAILED
Estado paralelo para camadas: SCHEDULED (espera fecha de nacimiento)
```

### `order_status_history`
`id`, `order_id`, `from_status`, `to_status`, `notes?`, `user_id?`,
`is_customer_visible`, `created_at`.

Alimenta la línea de tiempo que ve el cliente en el seguimiento del pedido.

### `order_adjustments` ⚠
`id`, `order_id`, `type` (`WEIGHT_DIFFERENCE`, `MISSING_ITEM`, `SUBSTITUTION`,
`MANUAL_DISCOUNT`, `SHIPPING_CORRECTION`), `amount_cents` (con signo),
`description`, `resolution` (`CHARGE_NOW`, `REFUND`, `STORE_CREDIT`,
`NEXT_ORDER`, `WAIVED`), `status`, `user_id`, `created_at`.

---

## 7. Pagos

### `payments`
`id`, `order_id`, `method` (`MERCADOPAGO`, `BANK_TRANSFER`, `CASH_ON_DELIVERY`,
`STORE_CREDIT`, `ACCOUNT`), `status`, `amount_cents`, `currency`,
`external_id?` (id de pago de MP), `external_status?`, `external_payload` (JSONB
crudo), `installments?`, `card_last_four?`, `payer_email?`,
`transfer_proof_url?`, `transfer_reference?`, `verified_by?`, `verified_at?`,
`rejection_reason?`, `paid_at?`, `created_at`.

### `payment_events`
Bitácora de webhooks para idempotencia: `id`, `payment_id?`, `provider`,
`event_id` unique, `event_type`, `payload` JSONB, `processed_at?`, `error?`.

> El `event_id` único es lo que impide que un webhook reintentado por Mercado
> Pago descuente stock dos veces.

### `refunds`
`id`, `payment_id`, `amount_cents`, `reason`, `external_id?`, `status`,
`user_id`, `created_at`.

---

## 8. Envíos

### `shipping_zones`
`id`, `name` ("Ciudad", "Zona norte 0–15 km", "Interior"), `type`
(`POSTAL_CODES`, `RADIUS`, `PROVINCE`), `postal_codes` (String[]),
`center_lat/lng?`, `radius_km?`, `is_active`.

### `shipping_methods`
`id`, `zone_id`, `name` ("Reparto propio", "Retiro en planta", "Correo"),
`type` (`DELIVERY`, `PICKUP`, `CARRIER`), `price_cents`,
`free_over_cents?`, `min_order_cents?`, `estimated_days_min/max`,
**`supports_cold_chain`** ⚠, `delivery_days` (String[]: `MON`,`WED`,`FRI`),
`cutoff_time` ("14:00" — después de esa hora pasa al día siguiente),
`is_active`, `position`.

> **Restricción de negocio codificada:** si el carrito contiene algún producto
> con `requires_cold_chain = true`, sólo se ofrecen métodos con
> `supports_cold_chain = true`. Esto evita el peor escenario posible: pollo fresco
> despachado por correo.

### `delivery_slots`
`id`, `shipping_method_id`, `date`, `start_time`, `end_time`, `capacity`,
`booked`, `is_active`.

### `shipments`
`id`, `order_id`, `method_id`, `status`, `tracking_number?`, `carrier?`,
`driver_name?`, `vehicle?`, `route_id?`, `scheduled_date`, `dispatched_at?`,
`delivered_at?`, `delivery_proof_url?` (foto o firma), `recipient_name?`,
`notes?`.

---

## 9. Contenido y soporte

- **`home_sections`**: `id`, `type` (`HERO`, `CATEGORY_GRID`, `PRODUCT_CARROUSEL`,
  `BANNER`, `TESTIMONIALS`, `TRUST_BADGES`, `CTA_WHOLESALE`), `title?`,
  `config` JSONB, `position`, `is_active`, `starts_at?`, `ends_at?`.
  → JB cambia la home sin tocar código.
- **`banners`**: `id`, `placement`, `image_desktop`, `image_mobile`, `link`,
  `alt`, `position`, `starts_at`, `ends_at`, `is_active`.
- **`pages`**: páginas institucionales editables.
- **`faqs`**: `id`, `category`, `question`, `answer`, `position`.
- **`leads`**: `id`, `email`, `phone?`, `source`, `interest`, `created_at`.
- **`search_queries`** ⚠: `id`, `query`, `results_count`, `customer_id?`,
  `clicked_product_id?`, `created_at`.
  → **Reporte más valioso del sistema:** qué buscan los clientes y no encuentran.
  Cada búsqueda con 0 resultados es una venta perdida identificable.
- **`product_reviews`** (Fase 3): con `is_verified_purchase`.
- **`audit_logs`**: `id`, `user_id`, `action`, `entity_type`, `entity_id`,
  `changes` JSONB, `ip`, `created_at`.
- **`settings`**: clave-valor con `group`, para configuración editable (CBU,
  WhatsApp, mínimos de compra, textos legales).

---

## 10. Índices críticos

```sql
-- Catálogo
CREATE INDEX ON products (status, category_id) WHERE deleted_at IS NULL;
CREATE INDEX ON products (slug) WHERE deleted_at IS NULL;
CREATE INDEX ON products USING GIN (search_vector);
CREATE INDEX ON products USING GIN (name gin_trgm_ops);  -- errores de tipeo

-- Operación diaria del panel
CREATE INDEX ON orders (status, created_at DESC);
CREATE INDEX ON orders (customer_id, created_at DESC);
CREATE INDEX ON orders (order_number);
CREATE INDEX ON orders (delivery_date) WHERE status IN ('READY','IN_TRANSIT');

-- Stock y carritos
CREATE UNIQUE INDEX ON stock_items (product_id, variant_id, warehouse_id);
CREATE INDEX ON carts (expires_at) WHERE customer_id IS NULL;

-- Idempotencia de webhooks
CREATE UNIQUE INDEX ON payment_events (provider, event_id);
```

---

## 11. Tareas programadas

| Tarea | Frecuencia | Qué hace |
|---|---|---|
| Liberar reservas vencidas | cada 5 min | Devuelve stock de carritos y pedidos impagos |
| Limpiar carritos de invitado | diaria | Borra los de más de 30 días |
| Recuperación de carrito abandonado | cada hora | Email a las 4 h y a las 24 h |
| Alerta de stock bajo | diaria 8:00 | Aviso interno |
| Alerta de vencimiento de lote | diaria | Productos próximos a vencer |
| Recordatorio de camada | diaria | Avisa al cliente 48 h antes del retiro |
| Cierre de reservas de camada | diaria | Pasa a `CLOSED` al llegar el `order_deadline` |
| Conciliación con Mercado Pago | diaria | Detecta pagos sin webhook recibido |
| Recalcular métricas | cada hora | `product_count`, `sales_count`, totales de cliente |

---

## 12. Qué se deja fuera a propósito en Fase 1

| Postergado | Fase |
|---|---|
| Multi-moneda | No previsto |
| Devoluciones con logística inversa | 3 |
| Cuenta corriente y límite de crédito | 4 |
| Lista de deseos / favoritos | 2 |
| Reseñas de productos | 3 |
| Programa de fidelización | 4 |
| Multi-depósito con ruteo automático | 3 |

Las columnas ya están previstas en el modelo (`credit_limit_cents`,
`warehouse_id`) para que activarlas no requiera migraciones dolorosas.
