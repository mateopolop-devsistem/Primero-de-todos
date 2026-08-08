# 07 — Estructura de pantallas

Cada pantalla se describe con: objetivo, estructura de arriba hacia abajo,
comportamiento en mobile y criterio de éxito.

---

## 1. HOME `/`

**Objetivo:** que un productor entienda en 3 segundos que acá compra pollito BB
con fecha, y que llegue al catálogo o al asesor en un clic.

```
┌────────────────────────────────────────────────────────────┐
│ BARRA SUPERIOR  Despachamos a todo el país · WhatsApp      │
├────────────────────────────────────────────────────────────┤
│ HEADER  [JB]  Pollitos ▾  Insumos ▾  Kits  Nacimientos     │
│         Guías   Mayoristas    [🔍]   [👤]  [🛒 2]           │
├────────────────────────────────────────────────────────────┤
│  HERO                                                      │
│  ┌──────────────────────────┬───────────────────────────┐  │
│  │ Pollito BB de calidad,   │                           │  │
│  │ con fecha asegurada      │  [ foto real de planta    │  │
│  │                          │    de incubación ]        │  │
│  │ Parrillero, ponedora,    │                           │  │
│  │ campero y ecológico.     │                           │  │
│  │ Desde 50 unidades.       │                           │  │
│  │ Despacho a todo el país. │                           │  │
│  │                          │                           │  │
│  │ [Ver nacimientos]        │                           │  │
│  │ [Ayudame a elegir]       │                           │  │
│  └──────────────────────────┴───────────────────────────┘  │
│                                                            │
│  SEÑALES DE CONFIANZA                                      │
│  🐣 Vacunados  📅 Fecha asegurada  🚚 Todo el país  💬 WSP  │
│                                                            │
│  📅 PRÓXIMOS NACIMIENTOS · todos los jueves  ★ estrella    │
│  ┌────────────┬────────────┬────────────┬────────────┐     │
│  │ jue 12/09  │ jue 12/09  │ jue 26/09  │ jue 10/10  │     │
│  │ Parrillero │ Ponedora   │ Parrillero │ Campero    │     │
│  │ 340 disp.  │ 800 disp.  │ 1.000 disp.│ Próxim.    │     │
│  │ cierra mié │ cierra mié │            │            │     │
│  │ [Reservar] │ [Reservar] │ [Reservar] │ [Avisarme] │     │
│  └────────────┴────────────┴────────────┴────────────┘     │
│                        [ Ver calendario completo → ]       │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  ¿No sabés cuál elegir?                              │  │
│  │  Decinos a quién le vas a vender y te decimos qué    │  │
│  │  pollito te conviene.        [ Ayudame a elegir ]    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  NUESTRAS LÍNEAS  (4 tarjetas con foto)                    │
│  ┌────────────┬────────────┬────────────┬────────────┐     │
│  │ PARRILLERO │  PONEDORA  │  CAMPERO   │ ECOLÓGICO  │     │
│  │ doble      │            │ de chacra  │            │     │
│  │ pechuga    │  Huevo     │            │            │     │
│  │ 45-55 días │ XXX huevos │ 90-120 días│            │     │
│  │ ~3,2 kg    │   por año  │            │            │     │
│  └────────────┴────────────┴────────────┴────────────┘     │
│                                                            │
│  🚚 ¿MANDAMOS A TU CIUDAD?                                 │
│  [ Provincia ▾ ] [ Ciudad ▾ ]      [ Consultar ]           │
│                                                            │
│  📦 KITS DE ARRANQUE  "Todo lo que necesitás para empezar" │
│                                                            │
│  INSUMOS  (carrusel: alimento, sanidad, equipamiento)      │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  ¿Comprás de a 500 o más?                            │  │
│  │  Precios por volumen para granjas, agropecuarias,    │  │
│  │  veterinarias y revendedores.                        │  │
│  │              [ Solicitar cuenta mayorista ]          │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  📖 GUÍAS DE CRIANZA  (3 destacadas — motor de SEO)        │
│                                                            │
│  CÓMO COMPRAR  (1 Elegí y reservá · 2 Pagá · 3 Recibí)     │
│  QUIÉNES SOMOS  ·  TESTIMONIOS  ·  NEWSLETTER              │
├────────────────────────────────────────────────────────────┤
│ FOOTER  Catálogo · Guías · Envíos · Legales · Contacto     │
│         JB · CUIT xx-xxxxxxxx-x · Dirección · Horario      │
│         Datos Personales (AAIP) · Botón de Arrepentimiento │
└────────────────────────────────────────────────────────────┘
                                        [💬 WhatsApp flotante]
```

**Mobile:** hero apilado con foto arriba; el calendario de nacimientos con scroll
horizontal, **por encima de todo lo demás**; barra inferior fija.

**Criterio de éxito:** ≥ 60% de las sesiones llegan al calendario, al catálogo o
al asesor. LCP < 2 s.

> **Legales obligatorios en Argentina:** enlace a *Botón de Arrepentimiento*
> (Res. 424/2020) y a Defensa del Consumidor, en el pie de todas las pantallas.

---

## 2. ASESOR DE COMPRA `/ayudame-a-elegir`

Tres preguntas, una por pantalla, con barra de progreso. Sin registro.

```
┌──────────────────────────────────────────┐
│  ●○○                          Paso 1 de 3│
│                                          │
│  ¿Para qué querés los pollitos?          │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │ 🥩 Vender a carnicería              │  │
│  │    Pollo grande, ~3,2 kg            │  │
│  │    Entran 6 en un cajón de 20 kg    │  │
│  └────────────────────────────────────┘  │
│  ┌────────────────────────────────────┐  │
│  │ 🔥 Vender a parrilladas             │  │
│  │    Pollo más chico, ~2 kg           │  │
│  │    Entran 10 en un cajón de 20 kg   │  │
│  └────────────────────────────────────┘  │
│  ┌────────────────────────────────────┐  │
│  │ 🥚 Producir huevos                  │  │
│  └────────────────────────────────────┘  │
│  ┌────────────────────────────────────┐  │
│  │ 🏡 Consumo propio / patio           │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

**Resultado:** línea recomendada con el porqué explicado en criollo, fechas
disponibles, kit sugerido si es primerizo, y acceso directo a reservar.

> Esta pantalla es el mayor diferencial del sitio frente a cualquier competidor
> y frente a una tienda enlatada. Traduce el lenguaje del cliente
> ("le vendo a parrilladas") al lenguaje del producto.

---

## 3. CALENDARIO DE NACIMIENTOS `/nacimientos`

```
┌────────────────────────────────────────────────────────────┐
│ Calendario de nacimientos                                  │
│ Reservá tu fecha. El cupo es limitado.                     │
│                                                            │
│ Filtros: [Todas las líneas ▾] [Todos los meses ▾]          │
│                                                            │
│  Nacemos todos los jueves.                                 │
│                                                            │
│  SEPTIEMBRE 2026                                           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ jue 12/09   Parrillero doble pechuga                 │  │
│  │             340 disponibles           ▓▓▓▓▓▓▒▒▒▒     │  │
│  │             ⏰ Reservás hasta el miércoles 18:00      │  │
│  │             Despacho: 12 al 14/09      [ Reservar ]  │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ jue 12/09   Ponedora (hembra)                        │  │
│  │             800 disponibles           ▓▓▒▒▒▒▒▒▒▒     │  │
│  │             ⏰ Reservás hasta el miércoles 18:00      │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ jue 19/09   Parrillero doble pechuga    [ Reservar ] │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ jue 26/09   Parrillero doble pechuga                 │  │
│  │             1.000 disponibles          [ Reservar ]  │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ jue 10/10   Campero de chacra                        │  │
│  │             Próximamente              [ Avisarme ]   │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

La barra de cupo con color (verde → amarillo → rojo) genera urgencia real, no
inventada: el cupo es efectivamente finito.

**El cierre es el día anterior, no una semana antes.** Hay clientes que reservan
con 21 días de anticipación y otros que reservan dos días antes: la web tiene que
servir a los dos. Cerrar temprano perdería toda la venta de último momento, que
es una parte real del negocio.

**Sí se distingue, en cambio, la reserva temprana como algo mejor**, porque le
sirve a JB para decidir cuánto producir:

```
┌──────────────────────────────────────────────────┐
│ 🗓  Reservá con anticipación                     │
│    Si reservás antes del 19/09, tu pedido queda  │
│    priorizado. Producimos según lo reservado.    │
└──────────────────────────────────────────────────┘
```

Eso no es una promesa de marketing: se cumple en la política de faltante, donde
la fecha de reserva es el criterio de desempate.

**"Avisarme"** alimenta `hatch_waitlist`: demanda concreta con nombre y cantidad,
y es la primera lista a la que se le ofrece un excedente.

---

## 4. CATÁLOGO `/productos` y `/productos/[categoria]`

```
┌────────────────────────────────────────────────────────────┐
│ Inicio › Pollitos BB › Parrillero                          │
│ Parrillero doble pechuga                    4 productos    │
├──────────────┬─────────────────────────────────────────────┤
│ FILTROS      │  [Ordenar: Relevancia ▾]                    │
│              │                                             │
│ Aptitud      │  ┌────────┐ ┌────────┐ ┌────────┐ ┌───────┐│
│ ☐ Carne      │  │ tarjeta│ │ tarjeta│ │ tarjeta│ │tarjeta││
│ ☐ Huevo      │  └────────┘ └────────┘ └────────┘ └───────┘│
│ ☐ Doble prop.│                                             │
│              │            [ Cargar más ]                   │
│ Sexo         │                                             │
│ ☐ Macho      │                                             │
│ ☐ Hembra     │                                             │
│ ☐ Mixto      │                                             │
│              │                                             │
│ Nacimiento   │                                             │
│ ☐ Este mes   │                                             │
│ ☐ Próximo mes│                                             │
│              │                                             │
│ ☑ Con cupo   │                                             │
│   disponible │                                             │
│ [ Limpiar ]  │                                             │
└──────────────┴─────────────────────────────────────────────┘
```

**El filtro por fecha de nacimiento es tan usado como el de producto.** Muchos
clientes llegan con la fecha decidida, no con la línea.

Filtros en la URL → compartible e indexable. **Mobile:** panel inferior con
contador de filtros y botón fijo `[ Ver 12 resultados ]`.

---

## 5. FICHA DE PRODUCTO `/producto/[slug]`

```
┌────────────────────────────────────────────────────────────┐
│ Inicio › Pollitos BB › Parrillero doble pechuga            │
├───────────────────────────┬────────────────────────────────┤
│                           │  Parrillero doble pechuga      │
│  ┌─────────────────────┐  │  Línea Cobb 500 · Mixto        │
│  │   foto real del     │  │                                │
│  │   pollito BB        │  │  $XXX por unidad               │
│  │   (no banco de      │  │  Desde 50 unidades             │
│  │    imágenes)        │  │                                │
│  └─────────────────────┘  │  PRECIO POR CANTIDAD           │
│  [▫][▫][▫][▫]             │  50-99 .......... $XXX         │
│                           │  100-499 ........ $XXX  −8%    │
│                           │  500-999 ........ $XXX  −15%   │
│                           │  1000+ .......... $XXX  −20%   │
│                           │                                │
│                           │  📅 ELEGÍ LA FECHA             │
│                           │  Nacemos todos los jueves      │
│                           │  ● jue 12/09 · 340 disponibles │
│                           │    ⏰ reservás hasta el mié 18h │
│                           │  ○ jue 19/09 · disponible      │
│                           │  ○ jue 26/09 · 1.000 disponib. │
│                           │  ○ jue 10/10 · próximamente    │
│                           │                                │
│                           │  Cantidad                      │
│                           │  [ − ]  500  [ + ]             │
│                           │  Se vende de a 50              │
│                           │  + 15 sin cargo por mortandad  │
│                           │  Total: $XXX.XXX               │
│                           │                                │
│                           │  [     Reservar ahora     ]    │
│                           │  [ Consultar por WhatsApp ]    │
│                           │                                │
│                           │  🚚 ¿Mandamos a tu ciudad?     │
│                           │  [Provincia ▾][Ciudad ▾][Ver]  │
│                           │  → Expreso del Norte           │
│                           │    sale mar y vie · 18 h       │
│                           │    Agencia Belgrano 450        │
│                           │    Flete ~$XX.XXX, se paga     │
│                           │    al retirar                  │
│                           │                                │
│                           │  🐣 Vacunado: Marek, Gumboro   │
├───────────────────────────┴────────────────────────────────┤
│ [ Descripción ] [ Ficha técnica ] [ Crianza ] [ Envíos ]   │
│                                                            │
│  FICHA TÉCNICA                                             │
│  Línea genética ........ Cobb 500                          │
│  Aptitud ............... Carne                             │
│  Sexo .................. Mixto                             │
│  Peso al nacer ......... 45 g mínimo                       │
│  Listo para faena ...... 45 a 55 días                      │
│  Peso final esperado ... 3,2 kg                            │
│  Rinde ................. 6 pollos por cajón de 20 kg       │
│  Conversión ............ X kg de alimento por kg de pollo  │
│  Vacunas ............... Marek, Gumboro, Newcastle         │
│                                                            │
│  ¿PARA QUIÉN ES ESTE POLLITO?                              │
│  Ideal si vendés a carnicería o pollo de chacra.           │
│  Si tu cliente son parrilladas, mirá la línea [X].         │
├────────────────────────────────────────────────────────────┤
│  LO QUE VAS A NECESITAR  (kit + insumos relacionados)      │
│  GUÍAS RELACIONADAS                                        │
└────────────────────────────────────────────────────────────┘
```

**Mobile:** galería a ancho completo; información apilada; **barra inferior fija
con precio + fecha elegida + `[ Reservar ]`**.

**Detalles que importan:**
- La **escala por cantidad visible en la ficha** empuja el ticket hacia arriba
  sola: el cliente ve que pasando a 100 baja el precio unitario.
- Los **pollitos de yapa por mortandad** se muestran como beneficio explícito,
  no como letra chica.
- El bloque *"¿para quién es este pollito?"* cruza-vende hacia la línea correcta
  y evita la venta equivocada, que termina en un cliente insatisfecho.

### Variantes de la ficha

| Tipo | Diferencia |
|---|---|
| Pollito BB | Selector de camada + escala por cantidad + múltiplos de 50 |
| Insumo | Stock simple, precio unitario, sin fecha |
| Kit de arranque | Detalle de lo que incluye, con opción de quitar ítems |

---

## 6. BÚSQUEDA `/buscar?q=`

Autocompletado desde 2 caracteres: productos con miniatura y precio, categorías,
guías de crianza, y búsquedas recientes.

**Sin resultados:** sugerencias con corrección de tipeo, la categoría más
cercana, acceso al asesor y CTA a WhatsApp. Toda búsqueda vacía se registra en
`search_queries`.

---

## 7. CONSULTA DE ENVÍOS `/envios`

Pantalla propia, porque es la objeción número uno del cliente del interior.

```
┌────────────────────────────────────────────────────────────┐
│ Despachamos a todo el país                                 │
│                                                            │
│ ¿A dónde te lo mandamos?                                   │
│ [ Provincia ▾ ]  [ Ciudad ▾ ]        [ Consultar ]         │
│                                                            │
│ ┌────────────────────────────────────────────────────────┐ │
│ │ ✓ Despachamos a La Banda, Santiago del Estero          │ │
│ │                                                        │ │
│ │ Expreso del Norte                                      │ │
│ │ Sale: martes y viernes · Corte 18:00                   │ │
│ │ Llega en: ~18 h                                        │ │
│ │ Retirás en: Agencia Belgrano 450                       │ │
│ │ Flete estimado: $XX.XXX — lo pagás al retirar          │ │
│ └────────────────────────────────────────────────────────┘ │
│                                                            │
│ CÓMO FUNCIONA                                              │
│ 1. Reservás y pagás la mercadería acá                      │
│ 2. El día del nacimiento despachamos al transporte         │
│ 3. Te mandamos el número de guía por WhatsApp              │
│ 4. Retirás en la agencia y pagás el flete ahí              │
│                                                            │
│ ⚠ IMPORTANTE                                               │
│ El pollito viaja vivo. Retiralo apenas llegue.             │
│ Tenés que tener la criadora encendida antes de buscarlo.   │
│                                                            │
│ POLÍTICA DE MORTANDAD                                      │
│ Enviamos un X% adicional sin cargo. Si igual tenés         │
│ pérdidas, reportalo dentro de las X horas con fotos.       │
└────────────────────────────────────────────────────────────┘
```

---

## 8. CARRITO `/carrito` (+ drawer)

```
┌──────────────────────────────────┬─────────────────────────┐
│ ┌──────────────────────────────┐ │  RESUMEN                │
│ │ [img] Parrillero doble pech. │ │                         │
│ │ 📅 Nace el 12/09             │ │  Mercadería  $XXX.XXX   │
│ │ 500 unidades + 15 sin cargo  │ │  ────────────────────   │
│ │ [−] 500 [+]      $XXX.XXX    │ │  Total       $XXX.XXX   │
│ └──────────────────────────────┘ │                         │
│ ┌──────────────────────────────┐ │  ⓘ El flete se paga     │
│ │ [img] Alimento iniciador     │ │    al retirar y no      │
│ │ Bolsa 25 kg      $XX.XXX     │ │    está incluido        │
│ │ [−] 4 [+]        $XX.XXX     │ │                         │
│ └──────────────────────────────┘ │  [ Finalizar compra ]   │
│                                  │  [ Seguir comprando ]   │
│ ┌──────────────────────────────┐ │                         │
│ │ 💡 Sumá 100 más y el precio  │ │  Cupón [____] [Aplicar] │
│ │    unitario baja 8%          │ │                         │
│ └──────────────────────────────┘ │  🔒 Compra protegida    │
└──────────────────────────────────┴─────────────────────────┘
```

El aviso de **próximo escalón de precio** es la función que más sube el ticket
promedio en venta por volumen. Cuesta poco y se paga sola.

**Vacío:** accesos a las líneas + calendario de nacimientos + asesor.

---

## 9. CHECKOUT `/checkout`

Sin menú, sin footer, sin salidas. Una página, tres bloques.

```
┌──────────────────────────────────┬─────────────────────────┐
│  1  CONTACTO                     │  TU PEDIDO              │
│  Email * · Teléfono/WhatsApp *   │  Parrillero x500        │
│  ☐ Quiero factura A              │  📅 Nace el 12/09       │
│                                  │  Alimento x4            │
│  2  CÓMO LO RECIBÍS              │                         │
│  ○ Retiro en planta (sin cargo)  │  Mercadería $XXX.XXX    │
│  ○ Reparto propio (Córdoba)      │  ──────────────────     │
│  ● Despacho por transporte       │  Total     $XXX.XXX     │
│                                  │                         │
│    Provincia [ Sgo. del Estero ▾]│  ⓘ Flete estimado       │
│    Ciudad    [ La Banda ▾ ]      │    $XX.XXX, lo pagás    │
│                                  │    al transporte cuando │
│    ● Expreso del Norte           │    retirás. No está     │
│      sale mar y vie · 18 h       │    incluido acá.        │
│      Agencia Belgrano 450        │                         │
│      ✓ compatible con el 12/09   │  [ Confirmar pedido ]   │
│                                  │                         │
│    ○ Transporte Sur              │  Al confirmar aceptás   │
│      ⚠ sale 5 días después del   │  los Términos y la      │
│        nacimiento — no disponible│  política de mortandad. │
│                                  │                         │
│    ○ No está mi ciudad           │                         │
│                                  │                         │
│  3  PAGO                         │                         │
│  ● Mercado Pago                  │                         │
│  ○ Transferencia (5% off)        │                         │
└──────────────────────────────────┴─────────────────────────┘
```

**Reglas duras:**
- Sin registro obligatorio.
- El total **nunca** cambia entre carrito y checkout.
- El flete a cargo del destinatario se aclara en checkout, confirmación y email.
- Un transporte incompatible con la fecha de nacimiento se **bloquea con la
  explicación a la vista**, no se oculta.
- Si el cupo cambió mientras completaba datos, se avisa antes de cobrar.

---

## 10. CONFIRMACIÓN `/checkout/confirmacion/[orderNumber]`

```
        ✅  ¡Listo, Miguel! Reservamos tus pollitos

              Pedido N° JB-2026-00042

  ┌──────────────────────────────────────────────┐
  │ 📅 Nacen el jueves 12/09                     │
  │ 🚚 Despachamos por Expreso del Norte         │
  │    Retirás en Agencia Belgrano 450, La Banda │
  │ ⓘ El flete lo pagás al retirar               │
  └──────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────┐
  │ ¿QUÉ PASA AHORA?                             │
  │ ✓ 1. Reserva confirmada                      │
  │   2. Te avisamos 7 días antes para que       │
  │      prepares la criadora                    │
  │   3. El 12/09 nacen y despachamos            │
  │   4. Te mandamos el número de guía           │
  └──────────────────────────────────────────────┘

  📖 Mientras tanto: [ Cómo recibir tu pollito ]

  [ Seguir mi pedido ]   [ Volver a la tienda ]
```

Si el pago fue por **transferencia**: CBU, alias, titular, CUIT, monto exacto,
botones de copiar, plazo, y subida del comprobante.

---

## 11. MI CUENTA `/mi-cuenta`

| Pantalla | Contenido |
|---|---|
| Resumen | Próxima entrega con cuenta regresiva, último pedido, "Repetir compra" |
| Pedidos | Lista con estado, fecha de nacimiento, total |
| Detalle | Línea de tiempo, datos de despacho y guía, `[Repetir]`, `[Reportar problema]` |
| Mis fechas | Calendario personal de nacimientos reservados |
| Direcciones y transporte habitual | Incluye transporte y agencia predeterminados |
| Datos fiscales | CUIT, condición IVA, razón social |
| Comprobantes | Descarga de facturas (Fase 3) |

**Detalle de pedido despachado:**

```
Pedido JB-2026-00042 · Despachado

●────●────●────●────●────○
Res. Pago Nació Encaj. Desp. Retirado

┌──────────────────────────────────────────────────┐
│ 🚚 DATOS DEL DESPACHO                            │
│ Expreso del Norte · Guía N° 887766               │
│ Salió: 12/09 18:30                               │
│ Retirás en: Agencia Belgrano 450, La Banda       │
│ Llegada estimada: 13/09 por la mañana            │
│ Flete: se paga al retirar                        │
│                                                  │
│ ⚠ Retiralo apenas llegue. Viaja vivo.            │
│                                                  │
│ [ Llamar a la agencia ]  [ Reportar problema ]   │
└──────────────────────────────────────────────────┘

Enviados: 500 + 15 sin cargo por mortandad = 515
```

---

## 12. GUÍAS DE CRIANZA `/guias`

Motor de tráfico orgánico y de fidelización. Quien busca *"a cuántos grados se
cría un pollito bebé"* es un cliente potencial que todavía no sabe que JB existe.

Contenidos iniciales: cómo recibir el pollito · temperatura semana por semana ·
alimentación por etapa · plan sanitario · errores frecuentes del primerizo ·
cuántos pollitos entran por m² · cuándo está listo para faena.

Cada guía cierra con los productos que menciona.

---

## 13. MAYORISTAS `/mayoristas`

Landing de captación B2B: beneficios (precios por volumen, prioridad de cupo en
camadas, atención dedicada, factura A), a quién va dirigido, cómo funciona en 3
pasos, formulario (razón social, CUIT, condición IVA, rubro, provincia, volumen
estimado), *"te respondemos en menos de 24 h hábiles"*, y testimonios.

---

## 14. Pantallas de soporte

`/nosotros` · `/contacto` · `/ayuda` (FAQ) · `/envios` · `/politica-mortandad` ·
`/terminos` · `/privacidad` · `/arrepentimiento` · `404` · `500`.

---

## 15. Emails y WhatsApp transaccionales

| Mensaje | Disparador | Contenido esencial |
|---|---|---|
| Reserva recibida | Orden creada | N°, fecha de nacimiento, total, próximos pasos |
| Datos para transferir | Pago por transferencia | CBU, alias, monto, plazo |
| Pago confirmado | Pago aprobado | Confirmación + fecha comprometida |
| Comprobante rechazado | Validación fallida | Motivo + cómo resolverlo |
| **Preparate** | 7 días antes | Criadora, temperatura, comederos, viruta |
| **Recordatorio de despacho** | 48 h antes | Transporte, agencia, horario |
| **Despachado** | Carga de la guía | N° de guía, agencia, llegada estimada |
| **Faltante de camada** | Nacimiento incompleto | Aviso proactivo, cuánto se pudo cubrir y las 3 opciones |
| **Oferta de excedente** | Nacimiento por encima de lo vendido | Cantidad, precio y hasta cuándo. Primero a los recortados |
| Retirado / entregado | Entrega | Agradecimiento + guía de primera semana |
| Seguimiento de crianza | Días 3, 10, 30 | Consejos + insumos de la etapa |
| **Recompra sugerida** | Según `grow_out_days` | "Ya estás faenando: ¿reponés?" |
| Carrito abandonado | 4 h y 24 h | Productos + fecha que estaba mirando |
| Camada nueva publicada | Alta de camada | A la lista de espera y a recurrentes |
| Cuenta mayorista aprobada | Aprobación | Bienvenida + cómo ver precios |
