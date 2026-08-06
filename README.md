# Plataforma Web JB

Planificación técnica y de producto para la plataforma de e-commerce de **JB** —
producción, distribución y comercialización de pollos + venta de insumos avícolas.

> **Estado del proyecto: PLANIFICACIÓN.** Este repositorio todavía no contiene
> código de aplicación. El objetivo de esta etapa es validar decisiones antes de
> escribir la primera línea.

---

## Resumen ejecutivo

JB no necesita "una página web": necesita un **canal de venta online** que reemplace
el flujo actual de *WhatsApp → pedir catálogo → pasar precios → coordinar pago*.
Ese flujo hoy tiene tres costos ocultos:

1. **No escala.** Cada venta consume tiempo humano. Duplicar ventas implica
   duplicar personas atendiendo el teléfono.
2. **Pierde clientes fuera de horario.** El 100% de la demanda nocturna y de fin
   de semana se cae o se demora.
3. **No deja datos.** No hay historial de compra, ni recompra automática, ni
   segmentación, ni forma de saber qué producto se busca y no se encuentra.

La plataforma ataca esos tres puntos. Todo lo demás (diseño, animaciones, blog)
es secundario frente a eso.

### La decisión más importante del proyecto

**El pollo se vende por peso y el peso real se conoce recién al preparar el pedido.**
Un e-commerce tradicional asume precio fijo por unidad. Acá no. Esto condiciona:
el modelo de datos, el checkout, el cobro con Mercado Pago, la facturación y el
panel administrativo.

La solución propuesta es el modelo de **peso estimado + ajuste por peso real**,
documentado en detalle en [`docs/08-flujo-de-compra.md`](docs/08-flujo-de-compra.md#5-el-problema-del-peso-variable).

### La segunda decisión más importante

JB vende a **dos públicos distintos con la misma plataforma**:

| | B2C / minorista | B2B / mayorista |
|---|---|---|
| Quién | Consumidor final, microemprendedor | Granjas, agropecuarias, veterinarias, revendedores |
| Ticket | Bajo, esporádico | Alto, recurrente |
| Precio | Lista pública, IVA incluido | Lista mayorista, precio neto + IVA |
| Registro | Compra como invitado | Cuenta aprobada, CUIT, condición IVA |
| Pago | Mercado Pago | Transferencia, eventualmente cuenta corriente |
| Qué valora | Confianza, simplicidad | Velocidad, recompra, precio, disponibilidad |

Una sola tienda con **listas de precios por grupo de cliente** resuelve ambos sin
duplicar catálogo ni mantener dos sitios. Es la decisión de arquitectura que más
trabajo futuro evita.

---

## Índice de la documentación

| # | Documento | Qué responde |
|---|---|---|
| 01 | [Arquitectura del sistema](docs/01-arquitectura.md) | Cómo se estructura el software, qué módulos hay, cómo se comunican |
| 02 | [Estructura de carpetas](docs/02-estructura-carpetas.md) | Dónde va cada archivo y por qué |
| 03 | [Stack tecnológico](docs/03-stack-tecnologico.md) | Qué tecnologías, con alternativas evaluadas y costos |
| 04 | [Base de datos](docs/04-base-de-datos.md) | Modelo de datos completo, tabla por tabla |
| 05 | [Experiencia de usuario (UX)](docs/05-ux.md) | Perfiles, recorridos, principios, fricciones a eliminar |
| 06 | [Interfaz y design system (UI)](docs/06-ui-design-system.md) | Color, tipografía, espaciado, componentes |
| 07 | [Estructura de pantallas](docs/07-pantallas.md) | Sección por sección, pantalla por pantalla |
| 08 | [Flujo completo de compra](docs/08-flujo-de-compra.md) | Del catálogo a la entrega, incluidos pagos y peso variable |
| 09 | [Panel administrativo](docs/09-panel-admin.md) | Módulos, roles, operatoria diaria |
| 10 | [Plan de desarrollo por fases](docs/10-plan-de-desarrollo.md) | Cronograma, sprints, criterios de aceptación |
| 11 | [Decisiones a validar](docs/11-decisiones-a-validar.md) | **Empezar por acá.** Lo que necesito que definas |

---

## Cómo leer esto

- Si tenés **15 minutos**: leé este README y el documento 11.
- Si tenés **1 hora**: sumá los documentos 05, 07 y 08 (producto y experiencia).
- Si sos **técnico**: 01, 03, 04 y 10.

## Próximo paso

Responder el cuestionario de [`docs/11-decisiones-a-validar.md`](docs/11-decisiones-a-validar.md).
Hay 6 decisiones bloqueantes ahí: sin ellas, cualquier código que escribamos tiene
riesgo alto de rehacerse.
