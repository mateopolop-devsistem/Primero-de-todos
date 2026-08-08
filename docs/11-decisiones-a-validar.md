# 11 — Decisiones a validar

**Estado: tercera ronda.** La primera corrigió un error de base (se planificaba
sobre el supuesto de que JB vendía pollo faenado; vende **pollito bebé**). La
segunda incorporó los nacimientos de los jueves y la **sobreventa deliberada**.
En esta tercera queda cerrado el criterio de reparto ante faltantes.

**No hay decisiones bloqueantes pendientes.** Lo que queda son datos para cargar
y definiciones que se ajustan sobre la marcha.

---

## A · Lo que ya está respondido

| # | Pregunta | Respuesta | Impacto en el sistema |
|---|---|---|---|
| A1 | ¿Cómo se cobra el pollo? | **No aplica.** JB vende pollito BB, no carne. Precio **por unidad**, cerrado | Se eliminó toda la maquinaria de peso variable y ajuste posterior. El sistema se simplificó de forma importante |
| — | ¿Y la variación de peso? | Es una decisión del **cliente**, no de JB. El que apunta a carnicería cría hasta ~3,2 kg (6 por cajón de 20 kg); el que apunta a parrillada faena en ~2 kg (10 por cajón) | Se convirtió en una **función**: el asesor de compra y el campo `crate_count` |
| A1b | Peso del pollito | **45 g mínimo.** Menos de 45 g es descarte y no se vende | Campo `min_weight_grams`, y argumento de calidad comunicable |
| A1c | Cantidad mínima | **50 unidades.** Se vende de a 50 o de a 100 | `min_order_qty = 50`, `qty_step = 50` |
| A1d | Líneas que se venden | Doble pechuga, ponedor, campero / de chacra, ecológico. **Precios distintos** | 4 categorías + tabla `chick_specs` |
| A3 | Envíos | **Todo el país.** Recorridos propios sólo por Córdoba; el resto por **comisionistas y transportes** | Se reemplazó el módulo de envíos por uno de despacho con transportes, agencias, días de salida y flete a cargo del destinatario |
| A3b | Cadena de frío | **No aplica.** Es un pollito vivo, no carne | Se eliminó. En su lugar aparece la restricción inversa: el pollito necesita **calor** y no puede esperar días a que salga el transporte |
| B1 | Nacimientos | **Jueves.** Reservas desde 21 días antes hasta 2 días antes. **Se vende de más a propósito porque a veces nacen menos** | Módulo de camadas con sobreventa controlada, historial de rendimiento, y protocolos de faltante y excedente |
| B1b | ¿Quién se queda corto ante un faltante? | **Proteger a los pedidos chicos**, con la fecha de reserva como desempate entre los grandes | `shortage_policy = PROTECT_SMALL` por defecto. **Es una opción del panel, no código**: cambiarla después es elegir otra en un desplegable |

---

## B · Lo que quedó abierto

### ~~B1 · Los nacimientos~~ — RESPONDIDA ✅

| Pregunta | Respuesta | Impacto |
|---|---|---|
| ¿Qué día nacen? | **Jueves** | Calendario semanal fijo. El público entiende "nacemos todos los jueves" |
| ¿Con cuánta anticipación reservan? | **Depende: hay clientes de 21 días y clientes de 2 días** | Las reservas quedan **abiertas hasta el día anterior**. Cerrar antes eliminaría toda la venta de último momento |
| ¿Nacen de menos? | **Sí, y por eso JB vende de más a propósito, "tanteando"** | **Cambia el núcleo del módulo.** Ver abajo |

#### Lo que esto corrigió

La versión anterior de esta planificación decía, como regla innegociable:
*"nunca comprometer más pollitos de los que van a nacer"*. **Era incorrecta.** JB
sobrevende deliberadamente porque el nacimiento real es incierto, y ajusta según
cómo venga. Eso no es un error a prevenir: es cómo funciona el negocio.

El modelo pasó a ser de **sobreventa controlada**, como el de una aerolínea:

- `expected_hatch` — lo que se espera que nazca (base de venta, **no un tope**)
- `oversell_pct` — cuánto se permite vender por encima
- `sellable` — el límite real del sistema
- `actual_hatched` → `hatch_rate` — lo que efectivamente pasó

Y trajo tres cosas que antes no estaban:

1. **Historial de rendimiento.** Cada camada guarda su `hatch_rate`. Con 10 o 15
   camadas, el sistema calcula promedio, desvío y peor caso **por línea**, y
   sugiere cuánto se puede sobrevender con seguridad. *"Ir tanteando" son datos
   que ya existen y hoy se tiran.* La sugerencia nunca se aplica sola: JB decide.
2. **Protocolo de faltante con simulación.** Antes de confirmar, JB ve
   exactamente a quién le toca el recorte y cuánto, y puede corregirlo a mano.
   El aviso a los afectados sale automático.
3. **Protocolo de excedente.** Si nacen de más, el sistema arma la lista de a
   quién ofrecérselos, en orden, y manda la oferta en un clic.

#### El criterio de reparto — DECIDIDO ✅

**Política por defecto: proteger a los pedidos chicos**, con la fecha de reserva
como desempate entre los grandes.

El razonamiento: a una granja que pidió 1.000 le entregás 960 y pierde el 4% de
una producción que igual va a hacer. A un productor de patio que pidió 50 y
recibe 0, lo perdés como cliente para siempre. Servir completos a los chicos
cuesta poco y evita el daño grande. Y el desempate por fecha de reserva empuja a
reservar temprano, que es justo lo que le sirve a JB para decidir cuántos huevos
cargar.

**Tres cosas que hacen esta decisión barata de revertir:**

1. **Es un campo, no código.** `shortage_policy` se elige desde el panel, camada
   por camada, entre cuatro opciones ya construidas: proteger chicos, orden de
   reserva, prorrateo, o manual. Cambiar el criterio general es cambiar el valor
   por defecto en Configuración.
2. **Nunca reparte sola.** El sistema simula, muestra a quién le toca cuánto, y
   recién ahí JB confirma. Se puede corregir a mano en cualquier momento.
3. **Queda registrado.** `batch_allocations` guarda qué política se aplicó y qué
   recibió cada pedido, así que a los 6 meses se puede ver si el criterio funcionó
   o si conviene otro — con datos, no con impresiones.

Dicho de otro modo: no hace falta acertar hoy. Hace falta que sea fácil corregir,
y lo es.

#### Lo que todavía queda de B1 (menor, no bloquea)

- ¿Nacen **todos** los jueves, o algunos jueves según la línea?
- ¿Cuánto solés sobrevender hoy, más o menos? Sirve para arrancar con un número
  razonable hasta que el historial tenga datos propios.

### B2 · Sexado

- ¿Vendés **macho, hembra o mixto**? ¿Depende de la línea?
- En ponedoras, ¿vendés sólo hembras? (es lo habitual)
- ¿El precio cambia según el sexo?

Hoy está modelado como `sex: MALE | FEMALE | MIXED` en `chick_specs`, y puede
funcionar como variante de producto si el precio difiere.

### B3 · Vacunación y mortandad

- ¿Contra qué viene vacunado el pollito? (Marek, Gumboro, Newcastle, bronquitis)
- ¿Es un diferencial tuyo frente a la competencia? Si lo es, va destacado.
- ¿**Agregás pollitos de más** para cubrir la mortandad del viaje? ¿Qué
  porcentaje?
- ¿Cómo manejás hoy un reclamo de "me llegaron muertos"? ¿Reponés, descontás, no
  cubrís?
- ¿Hay un plazo para reclamar?

Esto tiene que estar escrito y visible **antes** de comprar. Es el miedo número
uno del cliente que te compra a distancia sin conocerte.

### B4 · Los insumos

En el pedido original mencionaste "insumos y productos avícolas" como segunda
línea. Con el negocio ya aclarado:

- ¿Qué insumos vendés concretamente? (alimento balanceado, vacunas, comederos,
  bebederos, criadoras, viruta…)
- ¿Son propios o los revendés?
- ¿Los despachás por los mismos transportes o van aparte?
- ¿Te interesa el **kit de arranque** (pollitos + criadora + comedero + bebedero
  + alimento iniciador + viruta) para el cliente primerizo de 50 pollitos?

El kit es, en mi opinión, la mejor oportunidad comercial del catálogo: sube el
ticket, baja la mortandad del cliente y hace que vuelva. Un cliente al que se le
mueren los pollitos no compra nunca más.

### B5 · Precio mayorista

- ¿El precio por volumen es **público** o requiere cuenta aprobada?
  (recomendado: cuenta aprobada — protege el precio y te arma la base de clientes)
- ¿Cuántos niveles hay? (minorista / granja / distribuidor)
- ¿Los tramos son por cantidad (50 / 100 / 500 / 1000) o es un descuento general?
- ¿Hay líneas que sólo vendés a mayoristas?

### B6 · Pagos

- ¿Ya tenés cuenta de Mercado Pago a nombre de la empresa (con CUIT)?
- ¿Ofrecés cuotas? ¿Con o sin interés?
- ¿Hacés descuento por transferencia? ¿Qué porcentaje? (sugerido: 5%)
- ¿Los clientes grandes pagan al contado o hay cuenta corriente?
  (la cuenta corriente está prevista para Fase 4)

### B7 · Facturación

- ¿Hoy cómo facturás? ¿Con qué sistema?
- ¿Se puede seguir facturando por fuera de la web los primeros meses?
  (así la facturación electrónica queda en Fase 3 y no atrasa el lanzamiento)
- ¿Qué proporción de tus ventas es factura A?
- ¿El pollito BB lleva IVA 10,5% o 21%?

---

## C · Decisiones con valor por defecto

Si no hay respuesta, se avanza con esto y se ajusta después.

| # | Decisión | Por defecto |
|---|---|---|
| C0 | **Política de faltante** | **Proteger pedidos chicos** (decidido, editable en el panel) |
| C1 | Tiempo de reserva de cupo en el carrito | 30 minutos |
| C2 | Plazo para pagar por transferencia | 48 h (se acorta si la camada cierra antes) |
| C3 | Compra sin registro | Sí, habilitada |
| C4 | Precios a minoristas | Con IVA incluido |
| C5 | Precios a mayoristas | Neto + IVA discriminado |
| C6 | Flete por transporte | A cargo del destinatario, mostrado como estimado |
| C7 | Idioma y moneda | Español (Argentina), ARS |
| C8 | Modo oscuro | Fase posterior |
| C9 | Reseñas de productos | Fase 3 |
| C10 | Guías de crianza | **Fase 1** — son el motor de SEO del sitio |

---

## D · El stack — la duda quedó cerrada

En las rondas anteriores dejé abierta la posibilidad de que **Tiendanube fuera la
opción recomendada**: si el precio era cerrado por unidad y no había reserva por
fecha, una tienda enlatada era más barata y más rápida, y correspondía decirlo.

**La respuesta a B1 cierra esa duda, y no a favor de Tiendanube.** Lo que
describiste no es un catálogo con stock: es un sistema de producción con
sobreventa controlada.

| Lo que necesitás | Tiendanube / Shopify |
|---|---|
| Vender contra una fecha de nacimiento con cupo | Se simula con "productos por fecha" y se rompe al gestionar el cupo |
| **Vender de más a propósito, con un margen configurable** | No existe. El stock es un tope duro |
| **Repartir el faltante entre pedidos, con simulación previa** | No existe, en ninguna forma |
| **Aprender el rendimiento histórico y sugerir cuánto sobrevender** | No existe |
| Colocar un excedente urgente por orden de prioridad | No existe |
| Cruzar fecha de nacimiento con días de salida del transporte | No existe |
| Flete a cargo del destinatario | Choca con el modelo de checkout |

Las tres del medio son el corazón de tu operación de los jueves, y ninguna
plataforma enlatada las tiene ni las va a tener: son específicas de vender
animales vivos que nacen en cantidad incierta.

**Conclusión: la plataforma propia se justifica.** No porque sea más linda, sino
porque el jueves a la mañana, cuando faltan 60 pollitos y hay 14 pedidos, ninguna
otra herramienta te va a decir a quién recortar ni avisarle sola.

Dicho eso, sigue valiendo la advertencia general: si en algún momento el
presupuesto no da para desarrollo propio, es mejor una tienda enlatada funcionando
que un proyecto a medida a medio hacer.

## E · Información que necesito para arrancar

| # | Qué | Bloquea |
|---|---|---|
| E1 | Listado de líneas con precio, mínimo y escalas por cantidad | Sprint 2 |
| E2 | **Fotos reales** de pollitos, planta y encajonado | Sprint 2 |
| E3 | Logo en vectorial y colores | Sprint 1 |
| E4 | Calendario de nacimientos de los próximos 3 meses | Sprint 3 |
| E4b | **Historial de las últimas camadas: esperado vs. nacido real.** Aunque sea anotado a mano o de memoria — con 6 u 8 datos el sistema ya sugiere algo útil desde el día uno en vez de esperar 4 meses | Sprint 5 |
| E5 | Lista de transportes y comisionistas con los que trabajás, con ciudades y días | Sprint 4 |
| E6 | Zonas y días de reparto propio en Córdoba | Sprint 4 |
| E7 | Razón social, CUIT, domicilio fiscal, condición IVA | Sprint 6 |
| E8 | CBU, alias y titular | Sprint 4 |
| E9 | Credenciales de Mercado Pago (prueba y producción) | Sprint 4 |
| E10 | Política de mortandad, escrita | Sprint 4 |
| E11 | Plan sanitario (contra qué vacunás) | Sprint 2 |
| E12 | Historia de la empresa, 2 o 3 párrafos | Sprint 6 |
| E13 | Dominio (¿cuál? ¿ya lo tenés?) | Sprint 1 |
| E14 | WhatsApp de atención y horarios | Sprint 1 |
| E15 | 3 a 5 productores dispuestos a dar testimonio | Sprint 6 |

> **E5 es el que más trabajo te va a dar y el que más valor tiene.** Es tu mapa
> logístico, y hoy está en la cabeza de quien atiende el WhatsApp. Pasarlo a una
> tabla es lo que permite que la web responda "sí, mandamos a tu ciudad" sin que
> intervenga nadie. No hace falta que esté completo para arrancar: se puede
> empezar con los 10 destinos más frecuentes y crecer con el uso.

---

## F · Preguntas de negocio que ayudan a priorizar

No bloquean, pero cambian dónde se pone el esfuerzo:

1. ¿Cuántos pollitos vendés por mes, aproximadamente?
2. ¿Qué proporción va a granjas grandes y qué proporción a productores chicos?
3. ¿Qué proporción es Córdoba y qué proporción el resto del país?
4. ¿Cuánto tiempo por día se dedica a atender pedidos por WhatsApp?
5. ¿Cuál de las cuatro líneas se vende más?
6. ¿Hay estacionalidad? ¿Se cría más en alguna época del año?
7. ¿Tus competidores venden online?
8. ¿Quién de JB va a administrar la plataforma día a día?
9. ¿Hay presupuesto para publicidad digital al lanzar? Sin tráfico, la mejor web
   del mundo no vende.

---

## Cómo responder

Como venías: contestando de corrido, sin formato. Alcanza con eso.

**Ya no hay decisiones bloqueantes.** La de fondo quedó cerrada —va plataforma
propia— y el criterio de reparto está definido y es editable.

Lo que queda es de arranque, no de diseño. Las dos más útiles cuando puedas:

1. **Los insumos** (B4): si entran o no en la Fase 1
2. **Mercado Pago** (B6): si ya tenés cuenta a nombre de la empresa

Y el paquete de datos de la sección E, que es lo que efectivamente destraba el
comienzo: fotos, líneas con precios, y los primeros transportes con sus destinos.
