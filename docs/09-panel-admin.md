# 09 — Panel administrativo

## 1. Principio

El panel no es "la web pero para adentro". Es una **herramienta de trabajo
diario** que va a usar personal no técnico, muchas veces apurado, a veces desde
un celular en la planta.

Tres reglas:

1. **La operación del día se ve al entrar.** Sin buscar, sin filtrar.
2. **Las acciones frecuentes están a un clic.** Validar una transferencia,
   cargar un número de guía, cerrar una camada.
3. **Nada destructivo sin confirmación**, y todo queda auditado.

**El eje del panel es la camada, no el pedido.** JB no despacha pedidos sueltos:
despacha una camada que nace un día y se reparte entre muchos clientes. El panel
tiene que reflejar esa realidad.

---

## 2. Roles y permisos

| Rol | Puede | No puede |
|---|---|---|
| **ADMIN** | Todo, incluida configuración y usuarios | — |
| **MANAGER** | Camadas, pedidos, productos, precios, clientes, despachos, reportes | Configuración, usuarios, borrar datos |
| **SALES** | Ver y crear pedidos, ver y crear clientes, aprobar mayoristas | Cambiar precios, ver costos ni márgenes |
| **WAREHOUSE** | Ver camadas y despachos del día, cargar cantidades y guías, ajustar stock de insumos | Ver precios, clientes ni reportes |
| **VIEWER** | Sólo lectura de reportes | Cualquier escritura |

**Fase 2:** segundo factor obligatorio para `ADMIN` y `MANAGER`.

---

## 3. Dashboard `/admin`

```
┌────────────────────────────────────────────────────────────┐
│  Hoy · jueves 12 de septiembre                             │
│                                                            │
│  🐣 HOY NACE                                               │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Parrillero doble pechuga · comprometidos 1.000       │  │
│  │ 14 pedidos · 9 despachos por transporte              │  │
│  │ Nacidos reales: [_______]        [ Cerrar camada ]   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                            │
│  ⚠ REQUIEREN TU ATENCIÓN                                   │
│  ┌──────────────────┬──────────────────┬─────────────────┐ │
│  │ 3 transferencias │ 9 despachos sin  │ 2 solicitudes   │ │
│  │ por validar      │ número de guía   │ mayoristas      │ │
│  │        [ Ver → ] │        [ Ver → ] │      [ Ver → ]  │ │
│  └──────────────────┴──────────────────┴─────────────────┘ │
│  ┌──────────────────┬──────────────────┬─────────────────┐ │
│  │ Camada 26/09 al  │ 4 pedidos "a     │ 5 insumos con   │ │
│  │ 92% de cupo      │ coordinar envío" │ stock bajo      │ │
│  └──────────────────┴──────────────────┴─────────────────┘ │
│                                                            │
│  PRÓXIMAS CAMADAS                                          │
│  26/09 Parrillero  920/1000 ▓▓▓▓▓▓▓▓▓▒  cierra en 7 días   │
│  26/09 Ponedora    310/800  ▓▓▓▒▒▒▒▒▒▒                     │
│  10/10 Campero     PLANIFICADA — sin publicar              │
│                                                            │
│  VENTAS                              [Hoy][7d][30d][Año]   │
│  $X.XXX.XXX   ·  XX pedidos  ·  Ticket $XX.XXX   ▲ 18%     │
│                                                            │
│  BÚSQUEDAS SIN RESULTADO (7d)   ← demanda no cubierta      │
│  "pollo pekin" 14 · "pata muslo" 9 · "codorniz" 6          │
└────────────────────────────────────────────────────────────┘
```

Dos bloques valen especialmente:

- **"Camada al 92% de cupo"** avisa cuándo abrir otra o subir el precio.
- **"Búsquedas sin resultado"** es demanda real que JB hoy no captura. Si
  aparece "pato" catorce veces en un mes, es una decisión comercial fundada.

---

## 4. Camadas `/admin/camadas` ★ módulo central

### Calendario

```
┌────────────────────────────────────────────────────────────┐
│ Camadas                        [ + Nueva camada ]          │
│ [ Calendario ] [ Lista ]              Septiembre 2026 ◀ ▶  │
│                                                            │
│  lun   mar   mié   jue   vie   sáb   dom                   │
│                     12                                     │
│                    🐣🐣                                     │
│                  Parrill.                                  │
│                  Ponedora                                  │
│                                                            │
│   23    24    25    26    27                               │
│                     🐣                                      │
│                  Parrill.                                  │
│                  920/1000                                  │
└────────────────────────────────────────────────────────────┘
```

### Detalle de camada

```
┌────────────────────────────────────────────────────────────┐
│ Camada · Parrillero doble pechuga · 12/09/2026             │
│ Estado: [ ABIERTA ▾ ]                                      │
├────────────────────────────────────────────────────────────┤
│ Cupo total        1.000                                    │
│ Reservado           660  ▓▓▓▓▓▓▒▒▒▒  66%                   │
│ Disponible          340                                    │
│ Cierre de reservas  05/09/2026                             │
│ Ventana de despacho 12 al 14/09                            │
│ Nacidos reales      [_______]                              │
│                                                            │
│ PEDIDOS DE ESTA CAMADA (14)                                │
│ ┌────────────────────────────────────────────────────────┐ │
│ │ #00042 Miguel A.   500  Retiro planta      PAGADO      │ │
│ │ #00045 Carla P.    100  Expreso Norte      PAGADO      │ │
│ │ #00047 Ramón G.     50  Reparto Córdoba    A VALIDAR   │ │
│ └────────────────────────────────────────────────────────┘ │
│                                                            │
│ [ Cerrar reservas ]  [ Registrar nacimiento ]              │
│ [ Generar despachos ]  [ Avisar a los clientes ]           │
└────────────────────────────────────────────────────────────┘
```

### Registrar nacimiento — la acción crítica

```
┌──────────────────────────────────────────────┐
│  Camada 12/09 · Parrillero                   │
│  Comprometidos: 1.000                        │
│                                              │
│  ¿Cuántos nacieron?  [   960   ]             │
│                                              │
│  ⚠ FALTAN 40 POLLITOS                        │
│                                              │
│  ¿Cómo se resuelve?                          │
│  ○ Prorratear entre todos los pedidos        │
│  ● Completar desde otra camada               │
│  ○ Afectar a los pedidos más nuevos          │
│  ○ Resolver pedido por pedido                │
│                                              │
│  ☑ Avisar automáticamente a los afectados    │
│                                              │
│  [        Confirmar nacimiento        ]      │
└──────────────────────────────────────────────┘
```

> **Esta pantalla evita el peor momento operativo del negocio.** Si nacen menos
> de los comprometidos, alguien se va a quedar corto. Que el sistema lo detecte
> el día del nacimiento y avise automáticamente —en lugar de que el cliente se
> entere cuando va a retirar— es la diferencia entre un problema gestionado y un
> cliente perdido.

---

## 5. Despachos `/admin/despachos` ★

La operación del día de nacimiento, **agrupada por transporte**, que es como se
trabaja físicamente.

```
┌────────────────────────────────────────────────────────────┐
│ Despachos del 12/09                    [ Imprimir todo ]   │
│                                                            │
│ 🚚 EXPRESO DEL NORTE · corte 18:00 · 4 pedidos             │
│ ┌────────────────────────────────────────────────────────┐ │
│ │ #00045 Carla P.    La Banda    100+3   Guía [_______]  │ │
│ │ #00051 Juan M.     Frías        50+2   Guía [_______]  │ │
│ │ #00053 Ana T.      Termas      200+6   Guía [_______]  │ │
│ │ #00058 Luis R.     La Banda     50+2   Guía [_______]  │ │
│ │                              [ Marcar todos despachados ]│ │
│ └────────────────────────────────────────────────────────┘ │
│                                                            │
│ 🚚 TRANSPORTE CUYO · corte 16:00 · 2 pedidos               │
│ 🏠 REPARTO PROPIO — Zona sur · 3 pedidos  [ Hoja de ruta ] │
│ 🏢 RETIRO EN PLANTA · 5 pedidos                            │
│                                                            │
│ ⚠ 4 pedidos con envío a coordinar         [ Resolver → ]   │
└────────────────────────────────────────────────────────────┘
```

Al cargar el número de guía se dispara automáticamente el WhatsApp y el email al
cliente. **Es la acción que más consultas ahorra**: "¿ya salió?" desaparece.

### Hoja de ruta (reparto propio en Córdoba)
Pedidos del día agrupados por recorrido, ordenables, con vista imprimible y
vista mobile para el repartidor: dirección, contacto, cantidad, referencias y
forma de pago. Marcado de entrega con foto o firma.

---

## 6. Pedidos `/admin/pedidos`

### Vista tablero

```
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│ POR      │ RESERVA  │ NACIDOS  │ LISTOS   │ DESPACH. │
│ VALIDAR  │ CONFIRM. │          │          │          │
│   (3)    │  (28)    │   (14)   │   (9)    │   (5)    │
├──────────┼──────────┼──────────┼──────────┼──────────┤
│ #00047   │ #00042   │ #00045   │ #00051   │ #00038   │
│ Ramón G. │ Miguel A.│ Carla P. │ Juan M.  │ Ana T.   │
│ 📅 12/09 │ 📅 26/09 │ 📅 12/09 │ 📅 12/09 │ 📅 29/08 │
│ 🏦 transf│ 💳 MP    │ 🚚 Norte │ 🚚 Norte │ Guía     │
│ [Validar]│          │[Preparar]│ [Guía]   │ 887766   │
└──────────┴──────────┴──────────┴──────────┴──────────┘
```

### Vista tabla
Filtros por estado, **fecha de nacimiento**, cliente, transporte, provincia,
medio de pago y monto. Acciones masivas. Exportación a CSV/Excel.

### Detalle de pedido

```
┌────────────────────────────────────────────────────────────┐
│ Pedido JB-2026-00045          [Imprimir] [Remito] [⋯]      │
│ Estado: NACIDOS ▾                                          │
├────────────────────────────┬───────────────────────────────┤
│ PRODUCTOS                  │ CLIENTE                       │
│ Parrillero doble pechuga   │ Carla Pérez                   │
│ 📅 Camada 12/09            │ Agropecuaria del Norte        │
│ 100 unidades + 3 sin cargo │ CUIT 27-xxxxxxxx-4            │
│ $XXX.XXX                   │ Resp. Inscripto · MAYORISTA   │
│                            │ La Banda, Sgo. del Estero     │
│ Total       $XXX.XXX       │ 📞 · 💬 WhatsApp              │
│                            │ 8 pedidos · $X.XM histórico   │
├────────────────────────────┤ Compra cada ~45 días          │
│ DESPACHO                   │                    [Ver ficha]│
│ Expreso del Norte          ├───────────────────────────────┤
│ Agencia Belgrano 450       │ PAGO                          │
│ Sale mar y vie · 18:00     │ Transferencia · APROBADO      │
│ Flete: a cargo del cliente │ Comprobante [ver] · 09/09     │
│                            │ Validó: Sofía                 │
│ N° de guía [__________]    │                               │
│ [ Marcar despachado ]      │                               │
├────────────────────────────┴───────────────────────────────┤
│ LÍNEA DE TIEMPO                                            │
│ 08/09 14:30  Reserva creada                    (cliente)   │
│ 09/09 10:12  Transferencia aprobada            (Sofía)     │
│ 12/09 07:40  Camada nacida                     (sistema)   │
├────────────────────────────────────────────────────────────┤
│ NOTAS INTERNAS (no visibles para el cliente)               │
└────────────────────────────────────────────────────────────┘
```

### Creación manual de pedidos
Un pedido que llega por WhatsApp o teléfono se carga desde el panel con el mismo
motor de precios y cupos (`source = WHATSAPP | PHONE`). **Centraliza toda la
operación en un solo sistema**, que es medio objetivo del proyecto: aunque el
cliente siga llamando, la venta queda registrada, el cupo se descuenta y las
métricas son reales.

---

## 7. Productos `/admin/productos`

Listado con edición rápida de precio y estado en línea. Alta y edición por
pestañas:

| Pestaña | Campos |
|---|---|
| **General** | Nombre, slug, categoría, descripción, estado |
| **Precios** | Precio base, IVA, **escalas por cantidad** (50/100/500/1000), precios por lista |
| **Datos del pollito** ⚠ | Línea genética, aptitud, sexo, peso mínimo, días a faena, peso final esperado, **pollos por cajón**, mercado objetivo, conversión, vacunas, % de yapa por mortandad |
| **Compra** | Mínimo (50), múltiplo (50), máximo |
| **Camadas** ⚠ | Fechas programadas, cupos, cierres, ventanas de despacho |
| **Inventario** | Sólo insumos: stock, umbral de alerta |
| **Kit** | Productos incluidos y cantidades |
| **Imágenes** | Carga múltiple, reordenar, texto alternativo |
| **Ficha técnica** | Pares clave/valor, marcar filtrables |
| **SEO** | Título, descripción, vista previa de Google |

**Importación masiva** por CSV con plantilla, vista previa y validación fila por
fila. Indispensable para cargar el catálogo de insumos.

---

## 8. Precios `/admin/precios`

- **Listas:** minorista, granja, distribuidor.
- **Escalas por cantidad:** definición visual de tramos por producto y lista.
- **Actualización masiva:** *"Aumentar 12% todos los pollitos en la lista
  Minorista"*, con **vista previa antes de aplicar** y posibilidad de revertir.
- **Historial de cambios:** quién cambió qué precio y cuándo. Imprescindible con
  precios que se actualizan seguido.

---

## 9. Clientes `/admin/clientes`

- Listado con tipo, grupo, provincia, pedidos, total gastado, último pedido y
  **frecuencia de compra**.
- Ficha: datos, fiscales, transporte habitual, historial, notas internas, cambio
  de grupo, WhatsApp directo.
- **Solicitudes mayoristas:** cola de aprobación con `[Aprobar y asignar grupo]`
  / `[Rechazar]` y email automático.
- **Clientes a reponer** ⚠: quienes según su ciclo productivo ya deberían estar
  comprando de nuevo y no lo hicieron. Es una lista de llamadas priorizada, y
  probablemente la pantalla con mejor retorno comercial del panel.

---

## 10. Pagos `/admin/pagos`

- **Transferencias por validar:** comprobante ampliable, monto esperado vs.
  informado, datos del pedido, `[Aprobar]` / `[Rechazar + motivo]`.
- **Mercado Pago:** listado, estado, enlace a MP, reembolso total o parcial.
- **Conciliación:** pedidos pagados sin webhook y pagos sin pedido asociado.

---

## 11. Transportes `/admin/transportes` ⚠

- **Transportes y comisionistas:** alta, contacto, si aceptan animales vivos,
  cobertura.
- **Destinos:** por transporte, la ciudad, la agencia, la dirección, los días de
  salida, la hora de corte, las horas de viaje y el flete estimado.
- **Recorridos propios:** zonas de Córdoba y días.

> Esta tabla se construye con el uso. Cada pedido que entra como "no está mi
> ciudad" aparece acá como sugerencia para dar de alta. En seis meses, el mapa de
> transportes de JB es un activo que ningún competidor tiene documentado.

---

## 12. Contenido `/admin/contenido`

Home (secciones activables y reordenables), banners con vigencia, páginas, FAQ y
**guías de crianza** con editor enriquecido.

JB tiene que poder publicar una guía o cambiar una promoción sin llamar al
desarrollador. Es lo que determina que la web siga viva a los seis meses.

---

## 13. Reportes `/admin/reportes`

| Reporte | Para qué |
|---|---|
| Ventas por período | Evolución y comparación |
| Ventas por línea | Qué se vende y qué no |
| **Ocupación de camadas** | % de cupo vendido por fecha → ajustar producción |
| **Margen por producto** | Sólo `ADMIN`/`MANAGER` |
| **Ventas por provincia** | Dónde crecer y qué transporte reforzar |
| Clientes nuevos vs. recurrentes | Salud del negocio |
| **Recompra por ciclo** | Cuántos vuelven dentro de 1,5 ciclos productivos |
| Embudo de conversión | Dónde se pierden las ventas |
| **Búsquedas sin resultado** | Demanda no cubierta |
| **Uso del asesor** | Qué mercado declara la gente y qué termina comprando |
| Carritos abandonados | Monto perdido y recuperado |
| **Nacimiento real vs. comprometido** | Precisión de la planificación |
| **Reclamos por mortandad** | Por transporte y destino → detecta al transporte problemático |

Todos exportables a CSV/Excel.

> El cruce **mortandad × transporte** es el reporte que más plata puede ahorrar:
> si un transporte concentra los reclamos, el problema no es el pollito.

---

## 14. Configuración `/admin/configuracion`

Datos de la empresa · CBU, alias y titular · WhatsApp y horarios · mínimos y
múltiplos por defecto · **% de yapa por mortandad** · plazos de reserva y de
pago · política de mortandad · textos legales · integraciones (Mercado Pago,
email) · usuarios y roles · registro de auditoría.

---

## 15. Uso en mobile

El panel completo es responsive, pero **tres pantallas se diseñan primero para
mobile** porque se usan fuera del escritorio:

1. **Despachos** — en la planta, el día del nacimiento, cargando guías.
2. **Hoja de ruta** — en el reparto.
3. **Validación de transferencias** — desde cualquier lado, es urgente.

El resto (alta de productos, reportes) se optimiza para escritorio.
