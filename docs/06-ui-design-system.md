# 06 — Interfaz y design system (UI)

## 1. Dirección de diseño

La combinación pedida se traduce así:

| Inspiración | Qué se toma | Qué **no** se toma |
|---|---|---|
| **Apple** | Espacio en blanco generoso, tipografía como estructura, foto de producto protagonista, una sola acción clara por pantalla | Minimalismo extremo que esconde información. Acá el precio, la fecha y el cupo se ven siempre |
| **Mercado Libre** | Compra sin fricción, precio enorme y legible, botón de acción imposible de no ver, logística informada antes de comprar | Densidad visual saturada, banners superpuestos, colores compitiendo entre sí |
| **Amazon** | Jerarquía de categorías clara, filtros potentes, ficha con ficha técnica completa, "comprar de nuevo" | Interfaz sobrecargada, exceso de recomendaciones |

**Síntesis en una frase:**
> **La calma de Apple aplicada a la eficiencia de Mercado Libre, con la
> organización de catálogo de Amazon.**

Traducción práctica: fondo limpio, mucho aire, fotos grandes, **un solo color de
acento fuerte reservado para las acciones de compra**, y densidad informativa
sólo donde el usuario la necesita (listado y ficha técnica).

---

## 2. Color

Paleta construida sobre el producto real: el pollito recién nacido y el campo dan
un ámbar cálido y un verde natural. El resto es neutro para que las fotos manden.

### Marca

| Token | Valor | Uso |
|---|---|---|
| `--jb-primary` | `#D97706` (ámbar 600) | Acción principal, precios destacados, enlaces activos |
| `--jb-primary-hover` | `#B45309` | Estado hover |
| `--jb-primary-soft` | `#FEF3C7` | Fondos de realce, badges |
| `--jb-secondary` | `#15803D` (verde 700) | Frescura, confianza, "en stock", checks |
| `--jb-secondary-soft` | `#DCFCE7` | Fondo de mensajes de éxito |
| `--jb-accent` | `#0F172A` | Texto principal, encabezados |

> **Regla del acento:** el ámbar se usa **sólo** para acciones de compra y precio.
> Si aparece en decoración, deja de llamar la atención donde importa.
> En una pantalla debe haber **un solo botón ámbar**.

### Neutros (base cálida, no gris frío)

```
--neutral-0:   #FFFFFF   fondo principal
--neutral-50:  #FAFAF9   fondo de secciones
--neutral-100: #F5F5F4   fondo de tarjetas, skeletons
--neutral-200: #E7E5E4   bordes
--neutral-400: #A8A29E   texto deshabilitado
--neutral-500: #78716C   texto secundario
--neutral-700: #44403C   texto de cuerpo
--neutral-900: #1C1917   títulos
```

### Semánticos

```
--success: #16A34A    pedido confirmado, en stock
--warning: #EA580C    stock bajo, acción requerida
--danger:  #DC2626    error, sin stock, cancelado
--info:    #0284C7    informativo, en preparación
```

### Estados de pedido (color + ícono + texto, nunca sólo color)

| Estado | Color | Ícono |
|---|---|---|
| Pendiente de pago | `neutral-500` | reloj |
| En revisión | `warning` | lupa |
| Reserva confirmada | `info` | check en círculo |
| Programado (espera nacimiento) | `info` | calendario |
| Nacido | `secondary` | pollito |
| Listo / encajonado | `secondary` | check doble |
| Despachado | `info` | camión |
| Retirado / entregado | `success` | check relleno |
| Cancelado | `danger` | cruz |

---

## 3. Tipografía

- **Interfaz:** `Inter` (variable, autohospedada). Excelente legibilidad en
  números — clave en una web donde todo son precios, cantidades y fechas.
- **Títulos:** `Inter Display` o `Instrument Sans` para los `h1`/`h2` de la home.
- **Números tabulares** (`font-variant-numeric: tabular-nums`) obligatorio en
  precios, cantidades y tablas: evita que las cifras "bailen" al actualizarse.

### Escala

| Token | Tamaño | Peso | Uso |
|---|---|---|---|
| `display` | 40 / 56 px | 700 | Título del hero |
| `h1` | 30 / 36 px | 700 | Título de página |
| `h2` | 24 / 30 px | 600 | Sección |
| `h3` | 20 / 24 px | 600 | Nombre de producto en ficha |
| `body-lg` | 18 px | 400 | Texto destacado |
| `body` | 16 px | 400 | Base — nunca menos en texto principal |
| `body-sm` | 14 px | 400 | Secundario |
| `caption` | 12 px | 500 | Etiquetas, badges |
| `price-lg` | 32 px | 700 | Precio en ficha de producto |
| `price` | 20 px | 700 | Precio en tarjeta |

Interlineado: 1,5 en cuerpo; 1,2 en títulos. Ancho máximo de párrafo: 65 caracteres.

---

## 4. Espaciado, radios y sombras

**Escala de 4 px:** `4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96`.
Nada de valores arbitrarios: el ritmo visual consistente es lo que produce la
sensación de "prolijo".

**Radios:** `sm 6px` (badges) · `md 10px` (botones, inputs) · `lg 14px` (tarjetas)
· `xl 20px` (modales) · `full` (avatares, chips).

**Sombras:** sutiles y de un solo nivel visible por capa. Nada de sombras duras.

```
sm:  0 1px 2px   rgba(0,0,0,.05)
md:  0 4px 12px  rgba(0,0,0,.07)
lg:  0 12px 32px rgba(0,0,0,.10)
```

**Bordes antes que sombras** para delimitar tarjetas en listados: menos ruido
visual con muchos elementos en pantalla.

---

## 5. Grilla y breakpoints

| Breakpoint | Ancho | Columnas de producto | Contenedor |
|---|---|---|---|
| `xs` | 375 px | 2 | 100% – 32 px |
| `sm` | 640 px | 2 | 100% – 48 px |
| `md` | 768 px | 3 | 720 px |
| `lg` | 1024 px | 4 | 960 px |
| `xl` | 1280 px | 4 | 1200 px |
| `2xl` | 1536 px | 5 | 1360 px |

**Dos columnas en mobile, no una.** Permite comparar productos de un vistazo y
reduce el scroll a la mitad — es lo que espera quien viene de Mercado Libre.

---

## 6. Componentes clave

### 6.1 Tarjeta de producto

```
┌──────────────────────────────┐
│  ┌────────────────────────┐  │
│  │                        │  │ ← imagen 1:1, lazy, blur
│  │      [foto real]       │  │
│  │                  ♡     │  │ ← favorito (Fase 2)
│  │  [OFERTA]              │  │ ← badge esquina sup. izq.
│  └────────────────────────┘  │
│  Parrillero doble pechuga    │ ← 2 líneas máx.
│  Cobb 500 · Mixto            │ ← línea y sexo
│                              │
│  $XXX por unidad             │ ← precio: lo más grande
│  Desde 50 · baja a partir    │ ← escala por cantidad
│  de 100                      │
│  📅 Nace el 12/09            │ ← FECHA: tan importante
│  ▓▓▓▓▓▓▒▒▒▒ 340 disponibles  │ ← barra de cupo
│                              │
│  [    Reservar    ]          │ ← ámbar, ancho completo
└──────────────────────────────┘
```

Variantes: `default`, `compact` (carruseles), `list` (resultados de búsqueda con
más datos), `batch` (tarjeta de camada en el calendario), `skeleton`.

**Detalle crítico:** la tarjeta lleva **dos datos de primer nivel, no uno**:
el precio y la fecha de nacimiento. En este negocio el cliente descarta un
producto por la fecha tanto como por el precio, así que ocultar la fecha en la
ficha obliga a un clic que muchos no dan.

**La barra de cupo** usa color (verde → amarillo → rojo) y es urgencia real, no
inventada: el cupo de una camada es físicamente finito.

### 6.2 Botones

| Variante | Uso | Aspecto |
|---|---|---|
| `primary` | Comprar, agregar, confirmar | Fondo ámbar, texto blanco, 48 px de alto |
| `secondary` | Acción alternativa | Borde neutro, fondo blanco |
| `ghost` | Terciaria | Sólo texto |
| `danger` | Cancelar pedido, eliminar | Fondo rojo |
| `whatsapp` | Contacto | Verde WhatsApp, ícono |

Todos con estado `loading` obligatorio (spinner + deshabilitado). Un usuario que
no ve respuesta toca dos veces y genera pedidos duplicados.

### 6.3 Selector de cantidad
`[ − ]  500  [ + ]` con entrada manual, respetando `min_order_qty` (50) y
`qty_step` (50). Debajo, dos leyendas:

```
Se vende de a 50
+ 15 sin cargo por mortandad
💡 Sumá 100 más y el precio unitario baja 8%
```

El aviso del **próximo escalón de precio** es el componente que más sube el
ticket promedio en venta por volumen. Cuesta poco y se paga solo.

### 6.4 Selector de camada
Componente propio, presente en ficha, carrito y checkout:

```
📅 Elegí la fecha de nacimiento
   ● 12/09 · 340 disponibles · reservás hasta el 05/09
   ○ 26/09 · 1.000 disponibles
   ○ 10/10 · próximamente          [ Avisarme ]
```

Estados: con cupo, cupo bajo (< 15%), agotada (con lista de espera), cerrada,
planificada.

### 6.4b Aviso de flete
Presente en ficha, carrito, checkout, confirmación y email:

```
ⓘ  Flete estimado $XX.XXX
    Lo pagás al transporte cuando retirás.
    No está incluido en este total.        [ Cómo funciona ]
```

Aparecer en los cinco lugares es deliberado: la queja *"nadie me dijo que el
flete se pagaba aparte"* sólo se evita repitiendo el mensaje.

### 6.5 Cajón de carrito (drawer)
Se abre al agregar un producto. Muestra ítems con su fecha de nacimiento,
subtotal, el aviso de próximo escalón de precio, y dos acciones:
*Seguir comprando* / *Finalizar compra*.
**No redirige a otra página al agregar**: interrumpir la navegación baja el
tamaño del pedido.

### 6.6 Barra de búsqueda
Con autocompletado a partir de 2 caracteres: productos con miniatura y precio,
categorías sugeridas y búsquedas recientes. En mobile se abre a pantalla completa.

### 6.7 Señales de confianza
Franja de 4 elementos con ícono + texto corto, reutilizable en home, ficha y
checkout: *Vacunados de origen · Fecha de nacimiento asegurada · Despacho a todo
el país · Pago seguro con Mercado Pago*.

---

## 7. Movimiento

Sutil y funcional. Nada decorativo.

| Interacción | Duración | Curva |
|---|---|---|
| Hover / focus | 150 ms | `ease-out` |
| Apertura de drawer / modal | 250 ms | `cubic-bezier(.32,.72,0,1)` |
| Entrada de página | 200 ms | fade + 8 px de desplazamiento |
| Skeleton | 1.500 ms | pulso |
| Contador del carrito | 300 ms | escala 1 → 1.2 → 1 |

Se respeta `prefers-reduced-motion` en todos los casos.

---

## 8. Modo oscuro

**Fuera de alcance en Fase 1.** Los tokens ya se definen como variables CSS para
que activarlo después sea trabajo de horas y no de días. El fondo claro con foto real
convierte mejor en catálogos de producto; no es prioridad.

---

## 9. Voz y tono

- **Directo y en argentino neutro.** "Agregar al carrito", no "Añadir a la cesta".
- **Vos**, no *usted* ni *tú*: "Elegí tu forma de pago", "Ingresá tu código postal".
- **Sin jerga técnica.** "Te avisamos por WhatsApp", no "notificación asíncrona".
- **Sin promesas vacías.** "Entregamos en 24–48 h en la ciudad", no "envío rápido".
- **Los errores dicen qué hacer:** "No pudimos procesar el pago. Probá con otra
  tarjeta o elegí transferencia bancaria."

---

## 10. Identidad de marca

Si JB ya tiene logo e identidad, la paleta se ajusta a ella y este documento se
adapta (es la vía recomendada: la coherencia con la marca existente vale más que
cualquier propuesta nueva).

Si no la tiene, se incluye en Fase 1 un trabajo mínimo de identidad:
logotipo en versiones horizontal, vertical e isotipo; favicon; paleta;
tipografía; y una guía de uso de una página. Sin eso, la web se ve improvisada
por más buena que sea la interfaz.
