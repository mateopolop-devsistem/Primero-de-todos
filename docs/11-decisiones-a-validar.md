# 11 — Decisiones a validar

**Estado: segunda ronda.** La primera ronda corrigió un error de base — se había
planificado sobre el supuesto de que JB vendía pollo faenado. No es así: JB vende
**pollito bebé**, la materia prima. Toda la documentación fue reescrita.

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

---

## B · Lo que quedó abierto

### B1 · Los nacimientos ⚠ la pregunta principal

Te cortaste justo cuando ibas a contestar esto. Es lo que define si el catálogo
muestra un **calendario** o simplemente **stock**, y hoy está diseñado como
calendario porque es lo que corresponde a un pollito de un día.

- ¿JB **incuba** o compra el pollito a una incubadora y lo revende?
- ¿Los nacimientos son en **días fijos**? (típicamente un día fijo por semana)
- ¿Cada cuánto hay nacimientos de cada línea? ¿Todas las semanas? ¿Alternadas?
- ¿Con cuánta anticipación te reserva un cliente? ¿Días, semanas?
- ¿Se puede vender "para la semana que viene" o hay que reservar con más tiempo?
- ¿Cuántos pollitos salen por camada, aproximadamente?
- ¿Alguna vez nacen menos de los comprometidos? ¿Cómo lo resolvés hoy?

**Si la respuesta es "tengo pollitos casi siempre disponibles"**, el módulo de
camadas se simplifica mucho y el catálogo pasa a mostrar stock común. Es menos
trabajo, así que conviene saberlo antes de empezar.

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

## D · El stack, revisado con honestidad

En la primera ronda dije que si el pollo se vendía a precio cerrado por unidad y
no había camadas, **Tiendanube pasaba a ser una alternativa seriamente
considerable y más barata**. Ahora se cumple la primera condición: el precio es
cerrado por unidad. Corresponde revisarlo.

**Qué se puede hacer razonablemente bien en Tiendanube o Shopify:** catálogo,
carrito, Mercado Pago, mínimos de compra, precios mayoristas (con app), y
despacho a coordinar.

**Qué no, y es donde se juega el proyecto:**

| Necesidad | Por qué no encaja en una tienda enlatada |
|---|---|
| **Reserva contra fecha de nacimiento con cupo** | No existe el concepto. Se simula con "productos" por fecha, y se rompe al gestionar cupo, cierre y faltantes |
| **Cruce fecha de nacimiento × día de salida del transporte** | Imposible sin desarrollo propio. Es lo que evita despachar animales que se mueren en el camino |
| **Mapa de transportes con agencias y días** | No hay nada parecido. Es tu diferencial logístico |
| **Flete a cargo del destinatario** | Choca con el modelo de checkout de cualquier plataforma |
| **Asesor de compra por mercado objetivo** | Se puede hacer con una app externa, pobre y desconectada del catálogo |
| **Panel por camada, no por pedido** | Es tu operación real y no se puede reproducir |
| **Recompra por ciclo productivo** | Requiere `grow_out_days` por línea |

**Mi recomendación, sin adornos:** si la respuesta a **B1** es *"tengo pollitos
casi siempre disponibles, sin fechas"*, entonces Tiendanube es una opción
legítima para empezar, más barata y más rápida, y te conviene evaluarla en serio
antes de invertir en desarrollo. Si la respuesta es *"se vende por fecha de
nacimiento y hay que reservar"*, entonces la plataforma propia se justifica
sola: el calendario de camadas y el cruce con los transportes **son el producto**,
no un adorno.

Por eso B1 es la pregunta principal.

---

## E · Información que necesito para arrancar

| # | Qué | Bloquea |
|---|---|---|
| E1 | Listado de líneas con precio, mínimo y escalas por cantidad | Sprint 2 |
| E2 | **Fotos reales** de pollitos, planta y encajonado | Sprint 2 |
| E3 | Logo en vectorial y colores | Sprint 1 |
| E4 | Calendario de nacimientos de los próximos 3 meses | Sprint 3 |
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

**La prioritaria es B1 (los nacimientos).** De esa respuesta depende si seguimos
con plataforma propia o si te conviene evaluar algo más simple y barato — y eso
es mejor saberlo ahora que dentro de tres meses.
