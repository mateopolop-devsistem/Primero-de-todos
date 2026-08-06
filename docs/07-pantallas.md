# 07 — Estructura de pantallas

Cada pantalla se describe con: objetivo, estructura de arriba hacia abajo,
comportamiento en mobile y criterio de éxito.

---

## 1. HOME `/`

**Objetivo:** que un visitante nuevo entienda qué vende JB y llegue al catálogo
en un clic. No es una página institucional: es la entrada a la tienda.

```
┌────────────────────────────────────────────────────────────┐
│ BARRA SUPERIOR   Envíos en 24-48h · WhatsApp 11-xxxx-xxxx  │
├────────────────────────────────────────────────────────────┤
│ HEADER  [JB]  Pollos ▾  Insumos ▾  Ofertas  Mayoristas     │
│         [ 🔍 Buscar productos... ]      [👤]  [🛒 3]        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  HERO                                                      │
│  ┌──────────────────────────┬───────────────────────────┐  │
│  │ Pollos frescos,          │                           │  │
│  │ directo del productor    │   [ foto real de planta   │  │
│  │                          │     o producto ]          │  │
│  │ Comprá online y recibí   │                           │  │
│  │ en 24 h.                 │                           │  │
│  │                          │                           │  │
│  │ [ Ver pollos ] [Insumos] │                           │  │
│  └──────────────────────────┴───────────────────────────┘  │
│                                                            │
│  SEÑALES DE CONFIANZA (franja)                             │
│  ❄ Cadena de frío  🚚 24-48h  🔒 Pago seguro  💬 WhatsApp  │
│                                                            │
│  ACCESO A LAS DOS LÍNEAS                                   │
│  ┌────────────────────────┐  ┌────────────────────────┐    │
│  │      🐔  POLLOS        │  │   🌾  INSUMOS          │    │
│  │  Enteros, cortes,      │  │  Alimento, sanidad,    │    │
│  │  pollitos BB           │  │  equipamiento          │    │
│  │  [ Ver catálogo → ]    │  │  [ Ver catálogo → ]    │    │
│  └────────────────────────┘  └────────────────────────┘    │
│                                                            │
│  ★ LOS MÁS PEDIDOS  (carrusel · 8 productos)               │
│  [tarjeta] [tarjeta] [tarjeta] [tarjeta]  →                │
│                                                            │
│  CATEGORÍAS DESTACADAS  (grilla de 6 con foto)             │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  ¿Comprás por cantidad?                              │  │
│  │  Accedé a precios mayoristas para granjas,           │  │
│  │  agropecuarias, veterinarias y revendedores.         │  │
│  │              [ Solicitar cuenta mayorista ]          │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  📅 PRÓXIMAS CAMADAS DE POLLITOS BB                        │
│  ┌────────────┬────────────┬────────────┐                  │
│  │ 12/09      │ 26/09      │ 10/10      │                  │
│  │ 340 disp.  │ 1.000 disp.│ Próximam.  │                  │
│  │ [Reservar] │ [Reservar] │ [Avisarme] │                  │
│  └────────────┴────────────┴────────────┘                  │
│                                                            │
│  CÓMO COMPRAR  (1 Elegí · 2 Pagá · 3 Recibí)               │
│                                                            │
│  QUIÉNES SOMOS  (foto real + 3 líneas + [Conocenos])       │
│                                                            │
│  TESTIMONIOS  (3, con nombre y localidad)                  │
│                                                            │
│  NEWSLETTER  "Enterate de ofertas y camadas"               │
├────────────────────────────────────────────────────────────┤
│ FOOTER  Catálogo · Ayuda · Legales · Contacto              │
│         JB S.A. · CUIT xx-xxxxxxxx-x · Dirección · Horario │
│         Datos Personales (AAIP) · Botón de Arrepentimiento │
└────────────────────────────────────────────────────────────┘
                                        [💬 WhatsApp flotante]
```

**Mobile:** hero apilado con la foto arriba; los dos accesos de línea pasan a
ancho completo; carruseles con scroll horizontal por gestos; barra inferior fija
(Inicio · Catálogo · Buscar · Carrito).

**Criterio de éxito:** ≥ 60% de las sesiones llegan al catálogo. LCP < 2 s.

> **Legales obligatorios en Argentina:** el enlace a *Botón de Arrepentimiento*
> (Res. 424/2020) y a *Defensa del Consumidor* son requisitos legales para venta
> online. Van en el pie de página desde el día uno.

---

## 2. CATÁLOGO `/productos` y `/productos/[categoria]`

**Objetivo:** encontrar el producto rápido, sin sentir que hay que "buscar bien".

```
┌────────────────────────────────────────────────────────────┐
│ Inicio › Pollos › Pollo entero                             │
│                                                            │
│ Pollo entero                              12 productos     │
│ Frescos y congelados, faena diaria.                        │
├──────────────┬─────────────────────────────────────────────┤
│ FILTROS      │  [Ordenar: Relevancia ▾]  [▦ ▤]             │
│              │                                             │
│ Categoría    │  ┌────────┐ ┌────────┐ ┌────────┐ ┌───────┐│
│ ☑ Pollos     │  │        │ │        │ │        │ │       ││
│   ☐ Entero   │  │ tarjeta│ │ tarjeta│ │ tarjeta│ │tarjeta││
│   ☐ Cortes   │  │        │ │        │ │        │ │       ││
│              │  └────────┘ └────────┘ └────────┘ └───────┘│
│ Precio       │                                             │
│ [──●───●──]  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌───────┐│
│              │  └────────┘ └────────┘ └────────┘ └───────┘│
│ Presentación │                                             │
│ ☐ Fresco     │            [ Cargar más ]                   │
│ ☐ Congelado  │                                             │
│              │                                             │
│ Peso aprox.  │                                             │
│ ☐ 1,8-2,2 kg │                                             │
│ ☐ 2,2-2,8 kg │                                             │
│              │                                             │
│ ☑ Sólo con   │                                             │
│   stock      │                                             │
│              │                                             │
│ [ Limpiar ]  │                                             │
└──────────────┴─────────────────────────────────────────────┘
```

- Filtros en la URL (`?precio=1000-5000&stock=1`) → compartible e indexable.
- Orden: Relevancia · Menor precio · Mayor precio · Más vendidos · Novedades.
- Paginación por "Cargar más" con URL actualizada (SEO + mobile).
- **Mobile:** filtros en un panel inferior con contador de filtros activos y
  botón fijo `[ Ver 24 resultados ]`.

**Criterio de éxito:** ≥ 35% de clics desde el listado a una ficha.

---

## 3. FICHA DE PRODUCTO `/producto/[slug]`

**Objetivo:** resolver todas las dudas y llevar al carrito. Es la pantalla que
más ingresos genera y donde más detalle se justifica.

```
┌────────────────────────────────────────────────────────────┐
│ Inicio › Pollos › Pollo entero fresco                      │
├───────────────────────────┬────────────────────────────────┤
│                           │  Pollo entero fresco           │
│  ┌─────────────────────┐  │  SKU: POL-ENT-001              │
│  │                     │  │  ● En stock                    │
│  │   foto principal    │  │                                │
│  │      con zoom       │  │  $4.200 /kg                    │
│  │                     │  │  ≈ $10.080 la unidad (~2,4 kg) │
│  └─────────────────────┘  │                                │
│  [▫][▫][▫][▫]             │  ┌──────────────────────────┐  │
│                           │  │ ⚖ Precio estimado        │  │
│                           │  │ Se cobra el peso real,   │  │
│                           │  │ variación habitual ±10%. │  │
│                           │  │ Te avisamos antes de     │  │
│                           │  │ entregar.   [Cómo funciona]│ │
│                           │  └──────────────────────────┘  │
│                           │                                │
│                           │  Presentación                  │
│                           │  ( Fresco ) ( Congelado )      │
│                           │                                │
│                           │  Cantidad                      │
│                           │  [ − ]  20  [ + ]  unidades    │
│                           │  Total estimado: $201.600      │
│                           │                                │
│                           │  [   Agregar al carrito   ]    │
│                           │  [ Consultar por WhatsApp ]    │
│                           │                                │
│                           │  🚚 Calculá tu envío           │
│                           │  [ CP ] [Calcular]             │
│                           │  → Reparto propio · $3.500     │
│                           │    Entrega mar 09/09           │
│                           │                                │
│                           │  ❄ Requiere cadena de frío     │
│                           │  🔒 Pago protegido             │
├───────────────────────────┴────────────────────────────────┤
│ [ Descripción ] [ Ficha técnica ] [ Envíos ] [ Preguntas ] │
│                                                            │
│  Ficha técnica                                             │
│  Peso promedio ......... 2,4 kg                            │
│  Presentación .......... Entero, sin menudencias           │
│  Conservación .......... 0 a 4 °C                          │
│  Vida útil ............. 5 días refrigerado                │
│                                                            │
├────────────────────────────────────────────────────────────┤
│  QUIENES COMPRARON ESTO TAMBIÉN LLEVARON  (carrusel)       │
│  PRODUCTOS RELACIONADOS                                    │
└────────────────────────────────────────────────────────────┘
```

**Mobile:** galería a ancho completo con puntos indicadores; información apilada;
**barra inferior fija con precio + `[ Agregar ]`** que aparece al pasar el bloque
de compra.

**Variantes de la ficha según el tipo de producto:**

| Tipo | Diferencia |
|---|---|
| Peso variable (pollo) | Precio por kg + estimado + aviso de peso |
| Simple (comedero) | Precio unitario, sin aviso de peso |
| Bolsa (balanceado) | Precio por bolsa + precio por kg como referencia + escalas por volumen |
| **Camada (pollitos BB)** | Reemplaza el bloque de compra por el **selector de camada** |

### Bloque de compra de camadas

```
┌──────────────────────────────────────────────────┐
│  Elegí la fecha de nacimiento                    │
│                                                  │
│  ○ 12/09/2026 · 340 disponibles                  │
│    Reservás hasta el 05/09 · Retiro 12 al 14/09  │
│  ● 26/09/2026 · 1.000 disponibles                │
│    Reservás hasta el 19/09 · Retiro 26 al 28/09  │
│  ○ 10/10/2026 · Próximamente   [ Avisarme ]      │
│                                                  │
│  Cantidad  [ − ] 500 [ + ]   (de a 100)          │
│  Total: $XXX.XXX                                 │
│                                                  │
│  [        Reservar camada        ]               │
│  Se paga al reservar. Retiro en planta.          │
└──────────────────────────────────────────────────┘
```

---

## 4. BÚSQUEDA `/buscar?q=`

- Autocompletado desde 2 caracteres: productos (miniatura + precio), categorías,
  búsquedas recientes.
- Resultados con los mismos filtros del catálogo.
- **Sin resultados:** *"No encontramos «X»"* + sugerencias (corrección de tipeo),
  productos de la categoría más cercana, y CTA a WhatsApp. Toda búsqueda vacía
  se registra en `search_queries`.

---

## 5. CARRITO `/carrito` (+ drawer)

```
┌────────────────────────────────────────────────────────────┐
│ Tu carrito (3 productos)                                   │
├──────────────────────────────────┬─────────────────────────┤
│ ┌──────────────────────────────┐ │  RESUMEN                │
│ │ [img] Pollo entero fresco    │ │                         │
│ │       $4.200/kg · ~2,4 kg    │ │  Subtotal    $214.100   │
│ │       [−] 20 [+]   ~$201.600 │ │  Envío         $3.500   │
│ │       🗑 Quitar               │ │  ─────────────────────  │
│ └──────────────────────────────┘ │  Total est.  $217.600   │
│ ┌──────────────────────────────┐ │                         │
│ │ [img] Balanceado iniciador   │ │  ⚖ El total puede       │
│ │       Bolsa 25 kg   $12.500  │ │    variar según el      │
│ │       [−] 1 [+]      $12.500 │ │    peso real.           │
│ └──────────────────────────────┘ │                         │
│                                  │  [ Finalizar compra ]   │
│ ┌──────────────────────────────┐ │  [ Seguir comprando ]   │
│ │ 🚚 Te faltan $32.400 para    │ │                         │
│ │    el envío gratis           │ │  Cupón [______] [Aplicar]│
│ └──────────────────────────────┘ │                         │
│                                  │  🔒 Compra protegida    │
│ Entrega en [CP: 1704] ▾          │  Aceptamos: [MP] [🏦]   │
│ Reparto propio · mar 09/09       │                         │
└──────────────────────────────────┴─────────────────────────┘
```

**Vacío:** ilustración + *"Todavía no agregaste productos"* + accesos a Pollos e
Insumos + los más vendidos.

**Regla:** el envío se calcula **acá**, no en el checkout.

---

## 6. CHECKOUT `/checkout`

**Objetivo:** cero distracciones. Sin menú, sin footer, sin enlaces de salida.
Todo en **una página con tres bloques**, no en pasos separados (menos abandono).

```
┌────────────────────────────────────────────────────────────┐
│  [JB]                                    🔒 Compra segura  │
├──────────────────────────────────┬─────────────────────────┤
│  1  CONTACTO                     │  TU PEDIDO              │
│  ┌────────────────────────────┐  │  ┌───────────────────┐  │
│  │ Email *                    │  │  │ [img] Pollo x20   │  │
│  │ Teléfono / WhatsApp *      │  │  │       ~$201.600   │  │
│  │ ☐ Quiero factura A         │  │  │ [img] Balanc. x1  │  │
│  └────────────────────────────┘  │  │        $12.500    │  │
│  ¿Ya tenés cuenta? Ingresá       │  └───────────────────┘  │
│                                  │                         │
│  2  ENTREGA                      │  Subtotal   $214.100    │
│  ┌────────────────────────────┐  │  Envío        $3.500    │
│  │ ● Envío a domicilio        │  │  ────────────────────   │
│  │ ○ Retiro en planta (grat.) │  │  Total est. $217.600    │
│  │                            │  │                         │
│  │ Provincia / Localidad / CP │  │  ⚖ Se ajusta según      │
│  │ Calle / N° / Piso          │  │    peso real            │
│  │ Referencias                │  │                         │
│  │                            │  │  [ Confirmar pedido ]   │
│  │ Día de entrega:            │  │                         │
│  │ ( mar 09 ) ( jue 11 )      │  │  Al confirmar aceptás   │
│  │ Franja: ( 8-13 ) ( 13-18 ) │  │  los Términos.          │
│  └────────────────────────────┘  │                         │
│                                  │                         │
│  3  PAGO                         │                         │
│  ┌────────────────────────────┐  │                         │
│  │ ● Mercado Pago             │  │                         │
│  │   Tarjeta, dinero en cuenta│  │                         │
│  │ ○ Transferencia bancaria   │  │                         │
│  │   5% off · CBU al confirmar│  │                         │
│  └────────────────────────────┘  │                         │
└──────────────────────────────────┴─────────────────────────┘
```

**Mobile:** el resumen colapsa arriba (`Ver detalle ▾`) y el botón de confirmar
queda fijo abajo con el total siempre visible.

**Reglas duras:**
- Sin registro obligatorio.
- Autocompletado de localidad por código postal.
- Validación en tiempo real, con mensajes concretos.
- El total **nunca** cambia entre el carrito y el checkout.
- Si el stock cambió mientras el cliente completaba datos, se avisa **antes** de
  cobrar, indicando exactamente qué producto y qué opciones tiene.

---

## 7. CONFIRMACIÓN `/checkout/confirmacion/[orderNumber]`

```
        ✅  ¡Listo, Rosa! Recibimos tu pedido

              Pedido N° JB-2026-00042

  ┌──────────────────────────────────────────────┐
  │ ¿QUÉ PASA AHORA?                             │
  │ ✓ 1. Recibimos tu pedido                     │
  │   2. Lo preparamos y pesamos                 │
  │   3. Te avisamos el total final              │
  │   4. Te lo entregamos el mar 09/09, 8-13 h   │
  └──────────────────────────────────────────────┘

  Te enviamos el detalle a rosa@mail.com
  y te vamos avisando por WhatsApp.

  [ Seguir mi pedido ]   [ Volver a la tienda ]

  ┌──────────────────────────────────────────────┐
  │ Creá tu cuenta con un clic para seguir tus   │
  │ pedidos y repetirlos más rápido.             │
  │              [ Crear cuenta ]                │
  └──────────────────────────────────────────────┘
```

Si el pago fue por **transferencia**, esta pantalla muestra CBU, alias, titular,
CUIT, monto exacto, botones de copiar, y la subida del comprobante.

---

## 8. MI CUENTA `/mi-cuenta`

| Pantalla | Contenido |
|---|---|
| Resumen | Último pedido con su estado, accesos rápidos, "Repetir compra" |
| Pedidos | Lista con estado, fecha, total, y filtros |
| Detalle de pedido | Línea de tiempo, ítems con peso estimado vs. real, ajuste, comprobantes, seguimiento, `[Repetir pedido]`, `[Ayuda]` |
| Direcciones | Alta, edición, predeterminada |
| Datos fiscales | CUIT, condición IVA, razón social |
| Comprobantes | Descarga de facturas (Fase 3) |
| Favoritos | Fase 2 |

**Pantalla clave — detalle con ajuste de peso:**

```
Pedido JB-2026-00042 · Entregado

●────●────●────●────● 
Recib. Pago Prep. Pesado Entreg.

┌──────────────────────────────────────────────────┐
│ AJUSTE POR PESO REAL                             │
│ Pollo entero x20                                 │
│   Estimado: 48,00 kg  →  $201.600                │
│   Real:     47,60 kg  →  $199.920                │
│   Diferencia a tu favor:      −$1.680            │
│                                                  │
│ Total cobrado:   $217.600                        │
│ Total final:     $215.920                        │
│ Te devolvemos $1.680 por Mercado Pago (3-5 días) │
└──────────────────────────────────────────────────┘
```

Esta transparencia es lo que convierte el peso variable de problema en argumento
de confianza.

---

## 9. MAYORISTAS `/mayoristas`

Landing de captación B2B, no un formulario administrativo.

1. Encabezado: *"Precios mayoristas para tu negocio"*.
2. Beneficios: precios por volumen, camadas reservadas, atención dedicada,
   facturación A, entregas programadas.
3. A quién va dirigido: granjas, agropecuarias, veterinarias, revendedores.
4. Cómo funciona: 3 pasos.
5. Formulario: razón social, CUIT, condición IVA, rubro, localidad, volumen
   estimado, contacto.
6. *"Te respondemos en menos de 24 h hábiles"*.
7. Testimonios de clientes mayoristas.

---

## 10. Pantallas de soporte

| Ruta | Contenido |
|---|---|
| `/nosotros` | Historia, planta, equipo, fotos reales, valores |
| `/contacto` | Formulario, WhatsApp, teléfono, mapa, horarios |
| `/ayuda` | FAQ por categorías, buscador de preguntas |
| `/envios` | Zonas, costos, plazos, cadena de frío |
| `/devoluciones` | Política, plazos, procedimiento |
| `/terminos`, `/privacidad` | Legales (Ley 25.326, Defensa del Consumidor) |
| `/arrepentimiento` | Botón de arrepentimiento — obligatorio |
| `404` | Buscador + categorías principales + productos destacados |
| `500` | Mensaje claro + WhatsApp de contacto |

---

## 11. Emails transaccionales

Se diseñan con la misma identidad, en HTML responsive y con versión de texto plano.

| Email | Disparador | Contenido esencial |
|---|---|---|
| Pedido recibido | Orden creada | N°, detalle, total estimado, próximos pasos |
| Datos para transferir | Pago por transferencia | CBU, alias, monto, plazo, subida de comprobante |
| Pago confirmado | Pago aprobado | Confirmación + fecha de entrega |
| Comprobante rechazado | Validación fallida | Motivo claro + cómo resolverlo |
| **Ajuste por peso** | Pesaje cargado | Estimado vs. real, diferencia, cómo se resuelve |
| Pedido en camino | Despacho | Franja horaria, contacto del repartidor |
| Pedido entregado | Entrega | Agradecimiento + invitación a repetir |
| Carrito abandonado | 4 h y 24 h | Productos + acceso directo al carrito |
| Recordatorio de camada | 48 h antes | Fecha, cantidad, lugar y horario de retiro |
| Cuenta mayorista aprobada | Aprobación | Bienvenida + cómo ver precios mayoristas |
