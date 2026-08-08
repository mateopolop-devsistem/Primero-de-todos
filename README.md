# Plataforma Web JB

Planificación técnica y de producto para la plataforma de e-commerce de **JB** —
producción y venta de **pollito bebé (BB)** y de insumos avícolas.

> **Estado del proyecto: PLANIFICACIÓN.** Este repositorio todavía no contiene
> código de aplicación. El objetivo de esta etapa es validar decisiones antes de
> escribir la primera línea.

---

## Qué vende JB (y qué no)

Esta definición es la base de todo el resto del documento, y corrige un
malentendido de la primera versión de esta planificación.

```
   JB  ───►  GRANJA / PRODUCTOR  ───►  FAENA  ───►  CARNICERÍA / PARRILLADA
   │              │
   │              └─ engorda 45 a 90 días según la línea
   │
   └─ vende POLLITO BEBÉ de un día (45 g o más) + insumos para criarlo
```

**JB está arriba en la cadena.** Vende el animal vivo de un día de vida, no carne.
Consecuencias directas sobre el diseño del sistema:

| Lo que NO aplica | Lo que SÍ aplica |
|---|---|
| Precio por kilo | Precio **por unidad**, con escalas por cantidad |
| Peso variable y ajuste posterior | Precio cerrado y conocido al comprar |
| Cadena de frío | **Cadena de calor**: el pollito viaja y se cría a 32-35 °C |
| Cortes, pollo entero, congelado | Líneas genéticas: parrillero, ponedora, campero, ecológico |
| Reparto refrigerado local | **Despacho a todo el país** por transporte y comisionistas |
| Vencimiento y lotes de carne | Fecha de nacimiento, sexado, plan sanitario |

---

## Resumen ejecutivo

JB no necesita "una página web": necesita un **canal de venta online** que
reemplace el flujo actual de *WhatsApp → pedir catálogo → pasar precios →
coordinar pago → coordinar transporte*. Ese flujo hoy tiene tres costos ocultos:

1. **No escala.** Cada venta consume tiempo humano. Duplicar ventas implica
   duplicar personas atendiendo el teléfono.
2. **Pierde clientes fuera de horario**, y sobre todo pierde a los que están
   lejos: hoy el cliente de otra provincia tiene que confiar en un desconocido
   por WhatsApp para mandarle plata y esperar un encargo.
3. **No deja datos.** No hay historial, ni recompra, ni forma de saber que un
   cliente que compraba 500 parrilleros por mes dejó de comprar.

### Las tres decisiones que ordenan la arquitectura

**1 · El producto se vende por unidad, en múltiplos.**
Mínimo 50 pollitos, de a 50 o de a 100. Sin peso variable. Esto **simplifica
enormemente** el sistema respecto de la versión anterior de este plan: el precio
es cerrado, el checkout es estándar y desaparece toda la maquinaria de ajuste
posterior. Lo que sí hace falta es un motor de **escalas por cantidad**
(50 / 100 / 500 / 1.000+) y de **múltiplos obligatorios** de compra.

**2 · El envío es a todo el país, por transporte de terceros.**
Este es ahora el punto logístico crítico, y no se parece en nada a un e-commerce
común. Hay tres modalidades y la tercera es la dominante:

| Modalidad | Cómo funciona |
|---|---|
| Retiro en planta | El cliente viene |
| Reparto propio | Recorridos por Córdoba, en días fijos |
| **Transporte / comisionista** | JB despacha a la agencia o terminal; el cliente retira en destino. **El flete lo suele pagar el cliente al retirar** |

Que el flete se pague en destino y no se conozca al momento de comprar rompe el
supuesto básico de cualquier plataforma de e-commerce (que el total es total).
Está resuelto en [`docs/08-flujo-de-compra.md`](docs/08-flujo-de-compra.md#4-envíos-el-punto-crítico).

**3 · El pollito es un ser vivo con fecha.**
No es stock de depósito: nace un día determinado y tiene que salir ese día. La
venta es **reserva contra fecha de nacimiento**. Esto define el catálogo, el
checkout y la operación diaria.

### La oportunidad que aparece del negocio

Un cliente nuevo no sabe qué pollito comprar. Lo que sabe es **a quién le va a
vender**. La web puede traducir eso:

> *¿A quién le vendés?*
> **Carnicería / pollo de chacra** → pollo grande, ~3,2 kg, cajón nº 6 → línea X
> **Parrillada** → pollo chico, ~2 kg, cajón nº 10 → línea Y
> **Huevos** → ponedora
> **Consumo propio / patio** → campero

Ningún competidor hace esto. Es la diferencia entre un catálogo y un asesor, y
es lo que justifica una plataforma propia en lugar de una tienda enlatada.

---

## Índice de la documentación

| # | Documento | Qué responde |
|---|---|---|
| 01 | [Arquitectura del sistema](docs/01-arquitectura.md) | Cómo se estructura el software, qué módulos hay |
| 02 | [Estructura de carpetas](docs/02-estructura-carpetas.md) | Dónde va cada archivo y por qué |
| 03 | [Stack tecnológico](docs/03-stack-tecnologico.md) | Qué tecnologías, alternativas y costos |
| 04 | [Base de datos](docs/04-base-de-datos.md) | Modelo de datos completo |
| 05 | [Experiencia de usuario (UX)](docs/05-ux.md) | Perfiles reales, recorridos, principios |
| 06 | [Interfaz y design system (UI)](docs/06-ui-design-system.md) | Color, tipografía, componentes |
| 07 | [Estructura de pantallas](docs/07-pantallas.md) | Pantalla por pantalla |
| 08 | [Flujo completo de compra](docs/08-flujo-de-compra.md) | Del catálogo al despacho, incluidos pagos y transporte |
| 09 | [Panel administrativo](docs/09-panel-admin.md) | Módulos, roles, operatoria diaria |
| 10 | [Plan de desarrollo por fases](docs/10-plan-de-desarrollo.md) | Cronograma y criterios de aceptación |
| 11 | [Decisiones a validar](docs/11-decisiones-a-validar.md) | **Empezar por acá.** Lo que falta definir |

---

## Próximo paso

Quedan 4 preguntas abiertas en
[`docs/11-decisiones-a-validar.md`](docs/11-decisiones-a-validar.md).
La principal: **cómo funcionan los nacimientos** (fechas fijas, frecuencia,
anticipación de reserva). De eso depende si el catálogo muestra stock o muestra
un calendario.
