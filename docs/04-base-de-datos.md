# 04 — Diseño de la base de datos

## Reglas transversales

1. **Dinero en enteros.** Todos los importes son `Int` en **centavos de peso**.
   Nunca `Float`: los decimales flotantes producen diferencias de centavos que
   aparecen meses después en la conciliación de Mercado Pago.
2. **Identificadores.** `cuid2` para claves primarias (no revelan volumen de
   ventas ni permiten enumerar pedidos ajenos). Los pedidos además llevan un
   `order_number` legible (`JB-2026-00042`) para hablar por teléfono.
3. **Borrado lógico.** Productos, clientes y pedidos nunca se borran físicamente:
   `deleted_at`.
4. **Auditoría.** Toda escritura administrativa sobre precio, stock o estado de
   pedido queda registrada en `audit_logs` con usuario, valor anterior y nuevo.
5. **Precios históricos congelados.** `order_items` guarda copia del nombre, SKU
   y precio al momento de la compra. Si mañana cambia el precio, el pedido viejo
   **no cambia**.

> **Qué desapareció respecto de la versión anterior:** todo lo relativo a peso
> variable (`price_per_kg_cents`, `actual_weight_grams`, `order_adjustments` por
> peso), cadena de frío y vencimiento de carne. El pollito se vende por unidad a
> precio cerrado. El modelo se simplifica de forma importante.

---

## Mapa de entidades

```
                    ┌──────────────┐
                    │  categories  │──┐
                    └──────┬───────┘  │
                           │          │
      ┌────────────────────▼──────────▼─────────────┐
      │                 products                    │
      │  product_type: CHICK | SUPPLY | BUNDLE      │
      └──┬──────────┬──────────┬──────────┬─────────┘
         │          │          │          │
  ┌──────▼─────┐ ┌──▼──────┐ ┌─▼───────┐ ┌▼──────────────┐
  │chick_specs │ │ images  │ │ attrs   │ │bundle_items   │
  │ línea, sexo│ └─────────┘ └─────────┘ │kit de arranque│
  │ aptitud    │                         └───────────────┘
  └──────┬─────┘
         │
   ┌─────▼──────────┐        ┌──────────────┐
   │ hatch_batches  │        │ stock_items  │  (sólo insumos)
   │ FECHA DE       │        │              │
   │ NACIMIENTO     │        └──────┬───────┘
   └─────┬──────────┘               │
         │                    ┌─────▼─────────┐
         │                    │stock_movements│
         │                    └───────────────┘
         │
    ┌────┴───────────┬─────────────────┐
    ▼                ▼                 ▼
┌────────────┐ ┌──────────────┐ ┌──────────────┐
│ cart_items │ │ order_items  │ │price_list_   │
└─────┬──────┘ └──────┬───────┘ │   items      │
      │               │         └──────┬───────┘
 ┌────▼────┐   ┌──────▼──────┐    ┌────▼────────┐
 │  carts  │   │   orders    │    │price_lists  │
 └─────────┘   └──┬───┬───┬──┘    └────┬────────┘
                  │   │   │            │
       ┌──────────▼┐ ┌▼───▼──────┐ ┌───▼──────────┐
       │ payments  │ │dispatches │ │customer_     │
       │           │ │ TRANSPORTE│ │  groups      │
       └───────────┘ └─────┬─────┘ └───┬──────────┘
                           │           │
                    ┌──────▼─────┐ ┌───▼───────┐
                    │  carriers  │ │ customers │
                    │+ destinos  │ └───────────┘
                    └────────────┘
```

---

## 1. Identidad y clientes

### `users`
Credenciales y acceso. Separado de `customers`: un usuario admin no es un cliente.

`id`, `email` unique, `password_hash?`, `name`, `phone?`,
`role` (`CUSTOMER`, `ADMIN`, `MANAGER`, `SALES`, `WAREHOUSE`),
`email_verified_at?`, `last_login_at?`, `is_active`, `created_at`, `updated_at`.

`accounts`, `sessions`, `verification_tokens`: tablas estándar de Auth.js.

### `customers`

| Campo | Tipo | Nota |
|---|---|---|
| id | String PK | |
| user_id | String? unique FK | null = compró como invitado |
| type | Enum | `RETAIL`, `WHOLESALE` |
| status | Enum | `ACTIVE`, `PENDING_APPROVAL`, `SUSPENDED` |
| group_id | String? FK | determina la lista de precios |
| business_name | String? | razón social |
| tax_id | String? | CUIT/CUIL |
| tax_condition | Enum? | `RESPONSABLE_INSCRIPTO`, `MONOTRIBUTO`, `EXENTO`, `CONSUMIDOR_FINAL` |
| **producer_type** ⚠ | Enum? | `GRANJA`, `PRODUCTOR_CHICO`, `PATIO`, `AGROPECUARIA`, `VETERINARIA`, `REVENDEDOR`, `OTRO` |
| **target_market** ⚠ | Enum? | a qué le vende: `CARNICERIA`, `PARRILLADA`, `HUEVO`, `CONSUMO_PROPIO`, `REVENTA` |
| **typical_volume** ⚠ | Int? | cantidad habitual de pollitos por compra |
| province, city | String? | clave: define transporte disponible |
| email, phone, whatsapp | String | |
| default_address_id | String? | |
| **default_carrier_id** ⚠ | String? | "yo siempre retiro por Expreso X" |
| **default_carrier_destination_id** ⚠ | String? | agencia habitual de retiro |
| credit_limit_cents | Int? | Fase 4 — cuenta corriente |
| notes | Text? | uso interno |
| total_orders, total_spent_cents | Int | desnormalizado para el panel |
| last_order_at | DateTime? | |
| accepts_marketing | Boolean | Ley 25.326 |
| created_at, updated_at, deleted_at | DateTime | |

> **`target_market` y `producer_type` no son decoración.** Son la base del
> asesor de compra, de la segmentación comercial y de la recompra sugerida
> ("hace 50 días compraste 500 parrilleros: ¿reponés?").

> **`default_carrier_id` ahorra el paso más molesto del checkout.** El cliente
> del interior siempre despacha por el mismo transporte. Recordarlo convierte
> una compra de 6 campos en una de 1.

### `customer_groups`
`id`, `name` ("Minorista", "Granja", "Distribuidor"), `price_list_id`,
`discount_percent`, `min_order_qty`, `is_default`, `priority`.

### `addresses`
`id`, `customer_id`, `label`, `recipient_name`, `phone`, `street`, `number`,
`city`, `province`, `postal_code`, `lat/lng?`, `notes`, `is_default`.

---

## 2. Catálogo

### `categories`
Jerarquía por auto-referencia (`parent_id`).

`id`, `parent_id?`, `name`, `slug` unique, `description?`, `image_url?`,
`position`, `is_active`, `show_in_menu`, `seo_title?`, `seo_description?`.

Estructura inicial propuesta:

```
Pollitos BB  ★ (línea principal)
├── Parrillero doble pechuga     → carne, engorde rápido
├── Ponedora                     → huevo
├── Campero / de chacra          → doble propósito, crecimiento lento
└── Ecológico

Insumos avícolas
├── Alimento balanceado          (iniciador, crecimiento, terminador, ponedora)
├── Sanidad                      (vacunas, vitaminas, antibióticos, desinfectantes)
├── Equipamiento                 (comederos, bebederos, criadoras, campanas)
├── Cama y sustrato              (viruta, cáscara)
└── Accesorios

Kits
└── Kit de arranque              (pollitos + alimento + comedero + bebedero + criadora)
```

### `products`

| Campo | Tipo | Nota |
|---|---|---|
| id | String PK | |
| sku | String unique | |
| name | String | |
| slug | String unique | |
| short_description | String? | tarjeta y meta description |
| description | Text? | |
| category_id | String FK | |
| brand? | String | relevante en insumos |
| **product_type** ⚠ | Enum | `CHICK`, `SUPPLY`, `BUNDLE` |
| **sale_unit** ⚠ | Enum | `UNIT`, `BAG`, `BOX` |
| base_price_cents | Int | precio unitario de lista |
| compare_at_price_cents | Int? | precio tachado |
| cost_cents | Int? | interno, para el reporte de margen |
| vat_rate | Enum | `VAT_10_5`, `VAT_21`, `VAT_0` |
| **min_order_qty** ⚠ | Int | pollitos: **50** |
| **qty_step** ⚠ | Int | pollitos: **50** |
| max_order_qty | Int? | tope por pedido |
| **requires_live_transport** ⚠ | Boolean | animal vivo: limita transportes y plazos |
| track_inventory | Boolean | false en pollitos (van por camada) |
| allow_backorder | Boolean | |
| status | Enum | `DRAFT`, `ACTIVE`, `PAUSED`, `ARCHIVED` |
| is_featured, is_new | Boolean | |
| wholesale_only | Boolean | visible sólo para mayoristas aprobados |
| seo_title, seo_description | String? | |
| search_vector | tsvector | índice GIN |
| view_count, sales_count | Int | |
| created_at, updated_at, deleted_at | DateTime | |

### `chick_specs` ⚠ — la tabla propia del negocio de JB

Uno a uno con `products` cuando `product_type = CHICK`. Es lo que convierte el
catálogo en información útil para un productor.

| Campo | Tipo | Ejemplo |
|---|---|---|
| product_id | String PK FK | |
| **genetic_line** | String | "Cobb 500", "Ross 308", "Hy-Line Brown", "Campero INTA" |
| **purpose** | Enum | `MEAT_HEAVY`, `MEAT_LIGHT`, `EGG`, `DUAL`, `ORGANIC` |
| **sex** | Enum | `MALE`, `FEMALE`, `MIXED` |
| **min_weight_grams** | Int | **45** — por debajo es descarte y no se vende |
| **grow_out_days_min / max** | Int | parrillero 45–55; campero 90–120 |
| **target_weight_grams** | Int | peso vivo esperado a la faena, ej. 3.200 |
| **crate_count** ⚠ | Int? | pollos por cajón de 20 kg que resulta: **6** o **10** |
| **target_market** ⚠ | Enum[] | `CARNICERIA`, `PARRILLADA`, `HUEVO`, `CONSUMO_PROPIO` |
| **feed_conversion** | Decimal? | kg de alimento por kg de pollo |
| **vaccinations** | String[] | "Marek", "Gumboro", "Newcastle", "Bronquitis" |
| **mortality_bonus_pct** | Decimal | pollitos de más que se agregan por mortandad de viaje |
| **eggs_per_year** | Int? | sólo ponedoras |
| notes | Text? | |

> **`crate_count` es el campo que traduce el negocio del cliente.**
> Cajón de 20 kg: nº 6 son 6 pollos de ~3,2 kg (carnicería, pollo de chacra);
> nº 10 son 10 pollos de ~2 kg (parrillada, donde se vende por kilo y conviene
> más cantidad en menos peso). El productor no elige una "línea genética":
> elige a quién le va a vender. Este campo permite que la web hable su idioma.

> **`mortality_bonus_pct`** documenta los pollitos de yapa que se agregan por
> mortandad en el viaje. Que esté en el sistema evita discusiones posteriores y
> permite mostrarlo como garantía: *"Enviamos un 3% adicional sin cargo"*.

### `product_variants`
Presentaciones del mismo producto: bolsa de 25 o 40 kg; en pollitos, el sexado
cuando se vende como variante del mismo producto (macho / hembra / mixto).

`id`, `product_id`, `sku` unique, `name`, `option_values` (JSONB),
`price_cents?`, `barcode?`, `image_id?`, `position`, `is_active`.

### `product_images`
`id`, `product_id`, `variant_id?`, `url`, `public_id`, `alt` (obligatorio),
`width`, `height`, `blur_hash`, `position`, `is_primary`.

### `product_attributes`
Ficha técnica flexible sin migraciones: `id`, `product_id`, `key`, `value`,
`unit?`, `group?`, `position`, `is_filterable`.

### `bundle_items` ⚠ — kits
`id`, `bundle_product_id`, `item_product_id`, `quantity`, `is_optional`,
`price_override_cents?`.

> **El kit de arranque es la mejor oportunidad comercial del catálogo.** Quien
> compra 50 pollitos por primera vez necesita además criadora, comedero,
> bebedero, alimento iniciador y viruta — y no sabe cuál ni cuánto. Venderle el
> paquete armado sube el ticket, baja su tasa de fracaso y lo convierte en
> cliente recurrente. Un cliente al que se le mueren los pollitos no vuelve.

---

## 3. Nacimientos (el corazón del catálogo)

### `hatch_batches` ⚠
El pollito no es stock de depósito: **nace un día y sale ese día**.

| Campo | Tipo | Nota |
|---|---|---|
| id | String PK | |
| product_id | String FK | la línea que nace |
| **hatch_date** | Date | fecha de nacimiento |
| **capacity** | Int | cupo total de la camada |
| **reserved** | Int | ya vendido |
| **available** | Int generado | `capacity - reserved` |
| **status** | Enum | `PLANNED`, `OPEN`, `CLOSED`, `HATCHED`, `DISPATCHED`, `CANCELLED` |
| **order_deadline** | DateTime | límite para reservar |
| **dispatch_from / dispatch_to** | Date | ventana de despacho y retiro |
| price_cents? | Int | precio propio de la camada |
| **actual_hatched** | Int? | cuántos nacieron realmente |
| notes | Text? | |

**Regla dura:** `reserved` sólo se modifica dentro de una transacción con
`SELECT ... FOR UPDATE`. Comprometer más pollitos que los que van a nacer es el
peor error posible del sistema: no se resuelve con una disculpa, deja a un
productor sin producción.

### `hatch_waitlist`
Para camadas agotadas o todavía no publicadas: `id`, `product_id`,
`hatch_date?`, `customer_id?`, `email`, `phone`, `quantity`, `notified_at?`.

Es un activo comercial: demanda concreta con nombre y cantidad.

---

## 4. Precios

### `price_lists`
`id`, `name`, `currency` (ARS), `is_default`, `valid_from?`, `valid_to?`,
`is_active`.

### `price_list_items`
`id`, `price_list_id`, `product_id`, `variant_id?`, `price_cents`,
**`min_qty`**, `is_active`.

**Escalas por cantidad** — así se modela el precio real del rubro:

| Producto | min_qty | Precio unitario |
|---|---|---|
| Parrillero doble pechuga | 50 | $X |
| Parrillero doble pechuga | 100 | $X − 8% |
| Parrillero doble pechuga | 500 | $X − 15% |
| Parrillero doble pechuga | 1000 | $X − 20% |

Se toma la fila con el `min_qty` más alto que no supere la cantidad pedida.

### Algoritmo de resolución de precio (única fuente de verdad)

```
precioUnitario(producto, variante, cantidad, cliente):
  1. lista = cliente.grupo.lista_de_precios ?? lista_por_defecto
  2. item  = price_list_items del producto/variante
             con min_qty <= cantidad, tomando el min_qty más alto
  3. precio = item?.price_cents
              ?? camada.price_cents
              ?? variante.price_cents
              ?? producto.base_price_cents
  4. aplicar descuento del grupo
  5. aplicar promoción vigente (la mejor; no acumulables por defecto)
  6. aplicar cupón si corresponde
  7. RESP. INSCRIPTO → mostrar neto + IVA discriminado
     resto          → precio final con IVA incluido
```

Vive **una sola vez**, en `modules/pricing`.

### `coupons`
`id`, `code` unique, `type` (`PERCENT`, `FIXED`, `FREE_SHIPPING`), `value`,
`min_order_cents?`, `max_discount_cents?`, `usage_limit?`, `usage_count`,
`per_customer_limit?`, `applies_to` JSONB, `customer_group_ids?`, `starts_at`,
`ends_at`, `is_active`.

---

## 5. Inventario (sólo insumos)

Los pollitos no usan estas tablas: su disponibilidad la da `hatch_batches`.

### `warehouses`
`id`, `name`, `address`, `is_active`, `allows_pickup`.

### `stock_items`
`id`, `product_id`, `variant_id?`, `warehouse_id`, `quantity`, `reserved`,
`available` (generado), `low_stock_threshold`, `updated_at`.
Índice único `(product_id, variant_id, warehouse_id)`.

### `stock_movements`
Libro mayor, sólo inserciones: `id`, `stock_item_id`, `type` (`PURCHASE`,
`SALE`, `RETURN`, `ADJUSTMENT`, `LOSS`, `TRANSFER`, `RESERVATION`, `RELEASE`),
`quantity` (con signo), `quantity_before`, `quantity_after`, `reference_type`,
`reference_id`, `reason?`, `user_id?`, `created_at`.

### `batches`
Trazabilidad de insumos con vencimiento (vacunas, alimento): `id`, `product_id`,
`batch_code`, `expiry_date`, `quantity_initial`, `quantity_remaining`,
`supplier?`.

---

## 6. Carrito

### `carts`
`id`, `customer_id?`, `session_token` (cookie httpOnly), `price_list_id`,
`subtotal_cents`, `discount_cents`, `shipping_cents`, `total_cents`,
`coupon_id?`, `expires_at`, `reserved_until?`, `created_at`, `updated_at`.

Carrito **del lado del servidor**, no en `localStorage`: el cliente arma el
pedido en la computadora y lo cierra desde el celular, se puede recuperar por
email, y el precio no se manipula desde el navegador.

### `cart_items`
`id`, `cart_id`, `product_id`, `variant_id?`, **`hatch_batch_id?`**, `quantity`,
`unit_price_cents`, `subtotal_cents`, `added_at`.

---

## 7. Pedidos

### `orders`

| Campo | Nota |
|---|---|
| id, `order_number` | `JB-2026-00042` |
| customer_id?, email, phone | |
| `status` | ver máquina de estados |
| `payment_status` | `PENDING`, `IN_REVIEW`, `PAID`, `REFUNDED`, `FAILED` |
| `fulfillment_status` | `UNFULFILLED`, `SCHEDULED`, `READY`, `DISPATCHED`, `DELIVERED` |
| subtotal_cents, discount_cents, tax_cents, total_cents | |
| **shipping_cents** | 0 cuando el flete es a cargo del destinatario |
| **freight_payment** ⚠ | `PREPAID`, `COLLECT`, `NOT_APPLICABLE` |
| currency, price_list_id | |
| shipping_method_id?, shipping_address (JSONB congelado) | |
| billing_address (JSONB), tax_id?, tax_condition? | |
| **hatch_date?** ⚠ | fecha comprometida de nacimiento |
| **dispatch_date?** ⚠ | fecha de despacho |
| customer_notes?, internal_notes? | |
| `source` | `WEB`, `WHATSAPP`, `PHONE`, `ADMIN` |
| ip_address?, user_agent? | |
| confirmed_at?, paid_at?, dispatched_at?, delivered_at?, cancelled_at? | |
| cancellation_reason? | |
| created_at, updated_at | |

### `order_items`
`id`, `order_id`, `product_id?`, `variant_id?`, **`hatch_batch_id?`**,
`product_name` (copia), `product_sku` (copia), `variant_name?`, `quantity`,
**`bonus_quantity`** (pollitos de yapa por mortandad), `unit_price_cents`,
`subtotal_cents`, `vat_rate`, `vat_amount_cents`, `notes?`.

### Máquina de estados

```
                  ┌──────────────────┐
                  │ PENDING_PAYMENT  │◄── se crea la orden
                  └────────┬─────────┘
        transferencia ┌────┴────┐ Mercado Pago
                      ▼         ▼
            ┌──────────────┐  ┌──────────────┐
            │ PAYMENT_IN_  │  │  CONFIRMED   │
            │   REVIEW     │  │              │
            └──────┬───────┘  └──────┬───────┘
                   └────────┬────────┘
                            ▼
                    ┌───────────────┐
                    │   SCHEDULED   │  espera la fecha de nacimiento
                    └───────┬───────┘
                            ▼
                    ┌───────────────┐
                    │    HATCHED    │  nació la camada
                    └───────┬───────┘
                            ▼
                    ┌───────────────┐
                    │     READY     │  contado, encajonado, listo
                    └───────┬───────┘
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
      ┌────────────┐ ┌────────────┐ ┌──────────┐
      │ DISPATCHED │ │ IN_TRANSIT │ │  PICKUP  │
      │ transporte │ │reparto prop│ │  READY   │
      └──────┬─────┘ └─────┬──────┘ └────┬─────┘
             └─────────────┼─────────────┘
                           ▼
                   ┌──────────────┐
                   │  DELIVERED   │
                   └──────────────┘

Terminales alternativos: CANCELLED · REFUNDED · FAILED
```

`SCHEDULED` es el estado normal, no la excepción: casi todo pedido de pollitos
espera una fecha.

### `order_status_history`
`id`, `order_id`, `from_status`, `to_status`, `notes?`, `user_id?`,
`is_customer_visible`, `created_at`.

### `order_adjustments`
`id`, `order_id`, `type` (`MISSING_ITEM`, `SUBSTITUTION`, `MORTALITY_CLAIM`,
`MANUAL_DISCOUNT`, `FREIGHT_CORRECTION`), `amount_cents` (con signo),
`description`, `resolution` (`REFUND`, `STORE_CREDIT`, `NEXT_ORDER`,
`REPLACEMENT`, `WAIVED`), `status`, `user_id`, `created_at`.

> `MORTALITY_CLAIM` reemplaza al viejo ajuste por peso. Es el reclamo real de
> este negocio: *"me llegaron 12 muertos"*. Tener el caso modelado — con foto,
> cantidad y resolución — convierte el peor momento de la relación en una
> gestión ordenada.

---

## 8. Pagos

### `payments`
`id`, `order_id`, `method` (`MERCADOPAGO`, `BANK_TRANSFER`, `CASH`,
`STORE_CREDIT`, `ACCOUNT`), `status`, `amount_cents`, `external_id?`,
`external_status?`, `external_payload` JSONB, `installments?`, `payer_email?`,
`transfer_proof_url?`, `transfer_reference?`, `verified_by?`, `verified_at?`,
`rejection_reason?`, `paid_at?`, `created_at`.

### `payment_events`
Bitácora de webhooks para idempotencia: `id`, `payment_id?`, `provider`,
`event_id` **unique**, `event_type`, `payload` JSONB, `processed_at?`, `error?`.

El `event_id` único es lo que impide que un webhook reintentado por Mercado Pago
confirme dos veces la misma reserva.

### `refunds`
`id`, `payment_id`, `amount_cents`, `reason`, `external_id?`, `status`,
`user_id`, `created_at`.

---

## 9. Despacho y transportes ⚠

El módulo que reemplaza al de "envíos" de un e-commerce común. JB despacha a
**todo el país** y en la mayoría de los casos **no es quien transporta**.

### `carriers` — transportes y comisionistas
`id`, `name` ("Expreso San Luis", "Comisionista Pérez"), `type`
(`FREIGHT_COMPANY`, `BUS_PARCEL`, `COMMISSIONER`, `OWN`), `contact_name`,
`phone`, `whatsapp?`, `website?`, **`accepts_live_animals`**,
`coverage_notes`, `is_active`, `position`.

### `carrier_destinations` — agencias y terminales
`id`, `carrier_id`, `province`, `city`, `agency_name`, `address`, `phone?`,
**`departure_days`** (String[]: `MON`,`WED`,`FRI`), **`cutoff_time`**,
`transit_hours?`, `estimated_freight_cents?`, `is_active`.

> **Esta tabla es la que hace posible el checkout del interior.** El cliente de
> Santiago del Estero elige su ciudad y el sistema le muestra qué transportes
> salen, qué días y a qué agencia va a tener que ir a retirar. Hoy eso son diez
> mensajes de WhatsApp.

> **`departure_days` cruzado con `hatch_date` es la restricción central:** un
> pollito no puede esperar tres días a que salga el transporte. Si la camada nace
> un jueves y el transporte a esa ciudad sale martes y viernes, el sistema debe
> ofrecer el viernes y descartar el martes.

### `shipping_methods`
`id`, `name`, `type` (`PICKUP`, `OWN_ROUTE`, `CARRIER`), `carrier_id?`,
`price_cents?`, **`freight_payment`** (`PREPAID`, `COLLECT`),
`free_over_cents?`, `min_order_cents?`, `is_active`, `position`.

### `own_routes` — recorridos propios (Córdoba)
`id`, `name` ("Zona sur", "Ruta 9 norte"), `weekday`, `towns` (String[]),
`capacity`, `notes`, `is_active`.

### `dispatches`
`id`, `order_id`, `method_id`, `carrier_id?`, `carrier_destination_id?`,
`status`, **`tracking_code?`** (número de guía o encomienda), `crates_count`,
`chicks_count`, `dispatched_at?`, `driver_name?`, `delivered_at?`,
`delivery_proof_url?`, `recipient_name?`, `notes?`.

---

## 10. Contenido y soporte

- **`home_sections`**: `id`, `type` (`HERO`, `HATCH_CALENDAR`, `ADVISOR`,
  `CATEGORY_GRID`, `PRODUCT_CARROUSEL`, `BANNER`, `TRUST_BADGES`,
  `CTA_WHOLESALE`), `title?`, `config` JSONB, `position`, `is_active`,
  `starts_at?`, `ends_at?`.
- **`banners`**, **`pages`**, **`faqs`**: contenido editable por JB sin código.
- **`guides`** ⚠: contenido de crianza (cómo recibir el pollito, temperatura por
  semana, plan sanitario, alimentación). **Es la principal fuente de tráfico
  orgánico:** quien busca "cuántos grados necesita un pollito bebé" es un cliente
  potencial que todavía no sabe que existe JB.
- **`leads`**: `id`, `email`, `phone?`, `source`, `interest`, `created_at`.
- **`search_queries`** ⚠: `id`, `query`, `results_count`, `customer_id?`,
  `clicked_product_id?`, `created_at`. Cada búsqueda con 0 resultados es una
  venta perdida identificable.
- **`product_reviews`** (Fase 3), **`audit_logs`**, **`settings`**.

---

## 11. Índices críticos

```sql
-- Catálogo
CREATE INDEX ON products (status, category_id) WHERE deleted_at IS NULL;
CREATE INDEX ON products (slug) WHERE deleted_at IS NULL;
CREATE INDEX ON products USING GIN (search_vector);
CREATE INDEX ON products USING GIN (name gin_trgm_ops);   -- errores de tipeo

-- Nacimientos: la consulta más frecuente del sitio
CREATE INDEX ON hatch_batches (product_id, hatch_date)
  WHERE status IN ('OPEN','PLANNED');
CREATE INDEX ON hatch_batches (hatch_date, status);

-- Operación diaria
CREATE INDEX ON orders (status, hatch_date);
CREATE INDEX ON orders (dispatch_date) WHERE status = 'READY';
CREATE INDEX ON orders (customer_id, created_at DESC);
CREATE INDEX ON orders (order_number);

-- Transportes
CREATE INDEX ON carrier_destinations (province, city) WHERE is_active;

-- Idempotencia de webhooks
CREATE UNIQUE INDEX ON payment_events (provider, event_id);
```

---

## 12. Tareas programadas

| Tarea | Frecuencia | Qué hace |
|---|---|---|
| Liberar reservas vencidas | cada 5 min | Devuelve cupo de camadas y stock de insumos |
| Cerrar camadas | diaria | `OPEN` → `CLOSED` al pasar el `order_deadline` |
| **Recordatorio de despacho** | diaria | Avisa al cliente 48 h antes: fecha, transporte y agencia |
| **Aviso de camada próxima** | semanal | A la lista de espera y a clientes recurrentes |
| **Recompra sugerida** | diaria | Según `grow_out_days`: quien compró parrilleros hace 50 días ya faenó |
| Alerta de stock bajo (insumos) | diaria | Aviso interno |
| Vencimiento de lotes | diaria | Vacunas y alimento próximos a vencer |
| Carrito abandonado | cada hora | Email a las 4 h y a las 24 h |
| Conciliación con Mercado Pago | diaria | Detecta pagos sin webhook recibido |
| Recalcular métricas | cada hora | `sales_count`, totales por cliente |

> **La recompra sugerida es la tarea con mayor retorno del sistema.** El ciclo
> productivo es predecible: quien compró 500 parrilleros hace 50 días está
> faenando ahora y necesita reponer. Un mensaje automático en ese momento exacto
> vale más que cualquier campaña.

---

## 13. Qué se deja fuera a propósito en Fase 1

| Postergado | Fase |
|---|---|
| Cuenta corriente y límite de crédito | 4 |
| Reseñas de productos | 3 |
| Devoluciones con logística inversa | 3 |
| Multi-depósito | 3 |
| Programa de fidelización | 4 |
| Multi-moneda | No previsto |

Las columnas ya están previstas (`credit_limit_cents`, `warehouse_id`) para que
activarlas no requiera migraciones dolorosas.
