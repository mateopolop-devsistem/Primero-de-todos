# 09 — Panel administrativo

## 1. Principio

El panel no es "la web pero para adentro". Es una **herramienta de trabajo diario**
que va a usar personal no técnico, muchas veces apurado, a veces desde un celular
en el depósito.

Tres reglas:

1. **La operación del día se ve al entrar.** Sin buscar, sin filtrar.
2. **Las acciones frecuentes están a un clic.** Marcar preparado, validar una
   transferencia, cargar un peso.
3. **Nada destructivo sin confirmación**, y todo queda auditado.

---

## 2. Roles y permisos

| Rol | Puede | No puede |
|---|---|---|
| **ADMIN** | Todo, incluida configuración y usuarios | — |
| **MANAGER** | Pedidos, productos, precios, stock, clientes, reportes | Configuración, usuarios, borrar datos |
| **SALES** | Ver y crear pedidos, ver y crear clientes, aprobar mayoristas | Cambiar precios, ver costos ni márgenes |
| **WAREHOUSE** | Ver pedidos a preparar, cargar pesos, ajustar stock, marcar despachos | Ver precios, ver clientes, ver reportes |
| **VIEWER** | Sólo lectura de reportes | Cualquier escritura |

El rol `WAREHOUSE` es intencionalmente restringido: quien prepara pedidos no
necesita ver márgenes ni datos de clientes.

**Fase 2:** segundo factor obligatorio para `ADMIN` y `MANAGER`.

---

## 3. Dashboard `/admin`

Lo primero que ve quien entra a trabajar.

```
┌────────────────────────────────────────────────────────────┐
│  Hoy · martes 9 de septiembre                              │
│                                                            │
│  ⚠ REQUIEREN TU ATENCIÓN                                   │
│  ┌──────────────────┬──────────────────┬─────────────────┐ │
│  │ 3 transferencias │ 7 pedidos a      │ 2 solicitudes   │ │
│  │ por validar      │ preparar         │ mayoristas      │ │
│  │        [ Ver → ] │        [ Ver → ] │      [ Ver → ]  │ │
│  └──────────────────┴──────────────────┴─────────────────┘ │
│  ┌──────────────────┬──────────────────┬─────────────────┐ │
│  │ 5 productos con  │ 4 pedidos con    │ 1 lote vence    │ │
│  │ stock bajo       │ peso sin cargar  │ en 2 días       │ │
│  └──────────────────┴──────────────────┴─────────────────┘ │
│                                                            │
│  VENTAS                              [Hoy][7d][30d][Año]   │
│  ┌────────────────────────────────────────────────────┐    │
│  │  $1.245.300      42 pedidos     Ticket $29.650     │    │
│  │  ▲ 18% vs. período anterior                        │    │
│  │  ▁▂▃▅▆▇█▆▅▃▂  (gráfico)                            │    │
│  └────────────────────────────────────────────────────┘    │
│                                                            │
│  ENTREGAS DE HOY (12)          MÁS VENDIDOS (7d)           │
│  · Zona Norte     5  [Hoja de ruta]   1. Pollo entero  340 │
│  · Zona Centro    4                   2. Balanceado     85 │
│  · Retiro planta  3                   3. Pollito BB   2000 │
│                                                            │
│  BÚSQUEDAS SIN RESULTADO (7d)   ← oportunidades de venta   │
│  "pollo campero" 12 · "vacuna gumboro" 8 · "jaula" 5       │
└────────────────────────────────────────────────────────────┘
```

El bloque de búsquedas sin resultado es intencional: cada término repetido es
demanda real que JB hoy no está capturando.

---

## 4. Pedidos `/admin/pedidos`

El módulo más usado. Dos vistas del mismo dato:

### Vista tablero (operación diaria)

```
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│ POR      │ CONFIR-  │ EN PREPA-│ LISTOS   │ EN       │
│ VALIDAR  │ MADOS    │ RACIÓN   │          │ CAMINO   │
│   (3)    │   (7)    │   (4)    │   (5)    │   (2)    │
├──────────┼──────────┼──────────┼──────────┼──────────┤
│ #00045   │ #00042   │ #00039   │ #00037   │ #00035   │
│ Rosa G.  │ Miguel A.│ Carla P. │ Juan M.  │ Ana T.   │
│ $217.600 │ $890.000 │ $145.200 │ $67.400  │ $32.100  │
│ 🏦 transf│ 💳 MP    │ ⚖ pesar  │ 🚚 Norte │ 🚚 Centro│
│ [Validar]│ [Preparar│ [Cargar  │ [Despach]│ [Entreg.]│
│          │        ] │  peso]   │          │          │
└──────────┴──────────┴──────────┴──────────┴──────────┘
```

Arrastrar entre columnas cambia el estado, con confirmación en las transiciones
irreversibles.

### Vista tabla (búsqueda y análisis)
Filtros por estado, fecha, cliente, medio de pago, zona y monto. Acciones masivas
(marcar preparados, imprimir remitos). Exportación a CSV/Excel.

### Detalle de pedido

```
┌────────────────────────────────────────────────────────────┐
│ Pedido JB-2026-00042            [Imprimir] [Factura] [⋯]  │
│ Estado: EN PREPARACIÓN ▾                                   │
├────────────────────────────┬───────────────────────────────┤
│ PRODUCTOS                  │ CLIENTE                       │
│ ┌────────────────────────┐ │ Miguel Álvarez                │
│ │ Pollo entero x20       │ │ Granja San Miguel             │
│ │ est. 48,00 kg          │ │ CUIT 20-xxxxxxxx-5            │
│ │ real [______] kg  ⚖    │ │ Resp. Inscripto · MAYORISTA   │
│ │ $201.600               │ │ 📞 11-xxxx-xxxx  💬 WhatsApp   │
│ ├────────────────────────┤ │ 12 pedidos · $4.2M histórico  │
│ │ Balanceado 25 kg x1    │ │                    [Ver ficha]│
│ │ $12.500                │ ├───────────────────────────────┤
│ └────────────────────────┘ │ ENTREGA                       │
│                            │ Reparto propio · Zona Norte   │
│ Subtotal      $214.100     │ Ruta 12, km 4                 │
│ Envío           $3.500     │ mar 09/09 · 8 a 13 h          │
│ ─────────────────────────  │ "Portón verde, tocar bocina"  │
│ Total est.    $217.600     ├───────────────────────────────┤
│ Total final   $———         │ PAGO                          │
│                            │ Mercado Pago · APROBADO       │
│ [ Cargar pesos reales ]    │ ID 1234567 · 08/09 14:32      │
│                            │ [Ver en MP] [Reembolsar]      │
├────────────────────────────┴───────────────────────────────┤
│ LÍNEA DE TIEMPO                                            │
│ 08/09 14:30  Pedido creado                    (cliente)    │
│ 08/09 14:32  Pago aprobado                    (sistema)    │
│ 09/09 08:15  En preparación                   (Sofía)      │
├────────────────────────────────────────────────────────────┤
│ NOTAS INTERNAS (no visibles para el cliente)               │
└────────────────────────────────────────────────────────────┘
```

### Pantalla de pesaje `/admin/pedidos/preparacion`

Optimizada para **usarse en el depósito, con una mano, en un celular o tablet**.

```
┌──────────────────────────────────┐
│  Pedido JB-2026-00042            │
│  Miguel Álvarez                  │
│                                  │
│  Pollo entero  ·  20 unidades    │
│  Estimado: 48,00 kg              │
│                                  │
│  Peso real                       │
│  ┌────────────────────────────┐  │
│  │        47,60          kg   │  │ ← teclado numérico grande
│  └────────────────────────────┘  │
│                                  │
│  Estimado  48,00 kg   $201.600   │
│  Real      47,60 kg   $199.920   │
│  Diferencia          −$1.680     │
│  ✓ A favor del cliente           │
│  → Reembolso automático          │
│                                  │
│  Lote entregado [ L-2609-A  ▾ ]  │
│                                  │
│  [   Confirmar y continuar   ]   │
└──────────────────────────────────┘
```

Al confirmar: se recalcula el pedido, se genera el `order_adjustment`, se dispara
la resolución (reembolso, crédito o solicitud de pago) y se notifica al cliente.
Todo en una acción.

### Creación manual de pedidos
Un pedido que llega por WhatsApp o teléfono se carga desde el panel con el mismo
motor de precios y stock (`source = WHATSAPP | PHONE`). **Centraliza toda la
operación en un solo sistema**, que es medio objetivo del proyecto.

---

## 5. Productos `/admin/productos`

### Listado
Tabla con miniatura, nombre, SKU, categoría, precio, stock, estado. Filtros y
búsqueda. Edición rápida de precio y stock **en línea**, sin abrir la ficha.
Acciones masivas: activar, pausar, cambiar categoría, ajustar precios por %.

### Alta y edición (por pestañas)

| Pestaña | Campos |
|---|---|
| **General** | Nombre, slug, categoría, marca, descripción corta y larga, estado |
| **Precios** | Tipo de venta (unidad/kg/bolsa), precio base o por kg, precio comparativo, costo, IVA, precios por lista |
| **Peso variable** ⚠ | Activar, peso promedio, tolerancia %, unidad de venta |
| **Inventario** | SKU, seguimiento, stock por depósito, umbral de alerta, mínimo y paso de compra, vida útil, tipo de conservación |
| **Variantes** | Presentaciones con precio y stock propios |
| **Imágenes** | Carga múltiple con arrastrar, reordenar, texto alternativo, principal |
| **Ficha técnica** | Pares clave/valor, marcar cuáles son filtrables |
| **Camadas** ⚠ | Sólo si es `HATCH_PREORDER`: fechas, capacidad, límites, ventanas |
| **SEO** | Título, descripción, vista previa del resultado de Google |

### Importación masiva
Carga por CSV/Excel con plantilla descargable, vista previa antes de aplicar,
validación fila por fila e informe de errores. **Indispensable para la carga
inicial del catálogo**: nadie va a cargar 300 insumos de a uno.

---

## 6. Stock `/admin/stock`

- **Vista general:** producto, físico, reservado, disponible, alerta.
- **Ajuste rápido:** cantidad + motivo obligatorio (compra, producción, merma,
  mortandad, vencimiento, corrección) → genera movimiento auditado.
- **Movimientos:** historial completo filtrable, exportable.
- **Lotes:** alta con fecha de producción y vencimiento, seguimiento de saldo,
  alerta de vencimiento próximo, y **trazabilidad inversa** (a qué clientes se les
  entregó un lote determinado).
- **Camadas:** calendario de fechas de nacimiento, capacidad vs. reservado,
  apertura y cierre, y listado de pedidos asociados a cada camada.

---

## 7. Precios `/admin/precios`

- **Listas:** minorista, mayorista, distribuidor. Alta y edición.
- **Edición masiva:** *"Aumentar 12% todos los productos de la categoría
  Alimento en la lista Mayorista"*, con **vista previa antes de aplicar** y
  posibilidad de revertir.
- **Escalas por volumen:** definición de tramos por cantidad.
- **Historial de cambios:** quién cambió qué precio y cuándo — imprescindible en
  un contexto de precios que se actualizan seguido.

---

## 8. Clientes `/admin/clientes`

- Listado con tipo, grupo, cantidad de pedidos, total gastado, último pedido.
- Ficha: datos, fiscales, direcciones, historial completo, notas internas,
  cambio de grupo, contacto directo por WhatsApp.
- **Solicitudes mayoristas:** cola de aprobación con los datos del formulario,
  botones `[Aprobar y asignar grupo]` / `[Rechazar]`, y email automático.
- Segmentos (Fase 2): inactivos a 60 días, top 20 por facturación, etc.

---

## 9. Pagos `/admin/pagos`

- **Transferencias por validar:** comprobante ampliable, monto esperado vs.
  informado, datos del pedido, `[Aprobar]` / `[Rechazar + motivo]`. La cola con
  más impacto operativo del panel.
- **Pagos de Mercado Pago:** listado, estado, enlace a MP, reembolso total o
  parcial.
- **Conciliación:** pedidos pagados sin webhook, y pagos sin pedido asociado.

---

## 10. Envíos `/admin/envios`

- **Zonas y métodos:** alta de zonas por CP o radio, costos, envío gratis desde,
  días de reparto, hora de corte, soporte de cadena de frío.
- **Cupos por franja:** capacidad por día y horario, para no comprometer más
  entregas de las que se pueden hacer.
- **Hoja de ruta:** pedidos del día agrupados por zona, ordenables, con vista
  imprimible y vista mobile para el repartidor (dirección, contacto, ítems,
  referencia, forma de pago). Marcado de entrega con foto o firma.

---

## 11. Contenido `/admin/contenido`

- **Home:** activar, desactivar y reordenar secciones; editar textos e imágenes
  del hero; elegir productos destacados.
- **Banners:** por ubicación, con fechas de vigencia y versión mobile/desktop.
- **Páginas y FAQ:** editor de texto enriquecido.

JB tiene que poder cambiar una promoción sin llamar al desarrollador. Es lo que
determina que la web siga viva a los seis meses.

---

## 12. Reportes `/admin/reportes`

| Reporte | Para qué |
|---|---|
| Ventas por período | Evolución, comparación con período anterior |
| Ventas por producto y categoría | Qué mover y qué discontinuar |
| **Margen por producto** | Precio − costo. Sólo `ADMIN`/`MANAGER` |
| Clientes nuevos vs. recurrentes | Salud del negocio a mediano plazo |
| Top clientes | Base para atención dedicada |
| Embudo de conversión | Dónde se pierden las ventas |
| **Búsquedas sin resultado** | Demanda no cubierta |
| Carritos abandonados | Monto perdido y recuperado |
| Stock: rotación y mermas | Eficiencia operativa |
| Cumplimiento de entregas | % entregado en la fecha prometida |
| Ajustes por peso | Desvío promedio → permite calibrar `avg_weight_grams` |

Todos exportables a CSV/Excel.

---

## 13. Configuración `/admin/configuracion`

Datos de la empresa · CBU, alias y titular · WhatsApp y horarios · mínimos de
compra · **tolerancia de peso y umbral de ajuste** · plazos de reserva de stock ·
textos legales · integraciones (claves de MP, email) · usuarios y roles ·
registro de auditoría.

---

## 14. Uso en mobile

El panel completo es responsive, pero **tres pantallas se diseñan primero para
mobile** porque se usan fuera del escritorio:

1. **Pesaje** — en el depósito.
2. **Hoja de ruta** — en el reparto.
3. **Validación de transferencias** — desde cualquier lado, es urgente.

El resto (alta de productos, reportes) se optimiza para escritorio.
