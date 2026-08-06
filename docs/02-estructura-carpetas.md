# 02 — Estructura de carpetas

## Criterio

Organización **por dominio de negocio, no por tipo de archivo**. Una carpeta
`components/` con 200 componentes sueltos es imposible de mantener. Un desarrollador
nuevo tiene que poder abrir `src/modules/orders/` y entender todo lo que hace un
pedido sin saltar por siete carpetas.

---

## Árbol completo

```
jb-plataforma/
├── .github/
│   └── workflows/
│       ├── ci.yml                    # lint + typecheck + tests en cada PR
│       └── e2e.yml                   # Playwright sobre la preview
├── docs/                             # esta documentación
├── prisma/
│   ├── schema.prisma
│   ├── migrations/
│   └── seed/
│       ├── index.ts
│       ├── categorias.ts
│       ├── productos-demo.ts
│       └── zonas-envio.ts
├── public/
│   ├── fonts/
│   ├── icons/
│   └── manifest.json                 # PWA
├── src/
│   ├── app/                          # ── CAPA UI / RUTAS ──────────────
│   │   ├── (tienda)/                 # grupo público, layout con header+footer
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx                        # HOME
│   │   │   ├── pollos/
│   │   │   │   └── page.tsx                    # landing de la línea principal
│   │   │   ├── insumos/
│   │   │   │   └── page.tsx
│   │   │   ├── productos/
│   │   │   │   ├── page.tsx                    # catálogo + filtros
│   │   │   │   └── [...categoria]/page.tsx     # categoría y subcategorías
│   │   │   ├── producto/
│   │   │   │   └── [slug]/
│   │   │   │       ├── page.tsx
│   │   │   │       ├── opengraph-image.tsx
│   │   │   │       └── loading.tsx
│   │   │   ├── buscar/page.tsx
│   │   │   ├── carrito/page.tsx
│   │   │   ├── mayoristas/page.tsx             # captación B2B
│   │   │   ├── nosotros/page.tsx
│   │   │   ├── contacto/page.tsx
│   │   │   └── (legales)/
│   │   │       ├── terminos/page.tsx
│   │   │       ├── privacidad/page.tsx
│   │   │       ├── envios/page.tsx
│   │   │       └── devoluciones/page.tsx
│   │   │
│   │   ├── (checkout)/               # layout SIN header ni menú: cero fugas
│   │   │   ├── layout.tsx
│   │   │   └── checkout/
│   │   │       ├── page.tsx
│   │   │       ├── pago/page.tsx
│   │   │       └── confirmacion/[orderNumber]/page.tsx
│   │   │
│   │   ├── (cuenta)/
│   │   │   ├── layout.tsx
│   │   │   ├── ingresar/page.tsx
│   │   │   ├── registro/page.tsx
│   │   │   ├── recuperar-clave/page.tsx
│   │   │   └── mi-cuenta/
│   │   │       ├── page.tsx                    # resumen
│   │   │       ├── pedidos/
│   │   │       │   ├── page.tsx
│   │   │       │   └── [orderNumber]/page.tsx  # seguimiento + ajuste de peso
│   │   │       ├── direcciones/page.tsx
│   │   │       ├── datos-fiscales/page.tsx
│   │   │       ├── comprobantes/page.tsx
│   │   │       └── favoritos/page.tsx
│   │   │
│   │   ├── admin/                    # ── PANEL ADMINISTRATIVO ─────────
│   │   │   ├── layout.tsx                      # sidebar + guarda por rol
│   │   │   ├── page.tsx                        # dashboard
│   │   │   ├── pedidos/
│   │   │   │   ├── page.tsx                    # tablero + tabla
│   │   │   │   ├── [id]/page.tsx
│   │   │   │   └── preparacion/page.tsx        # pantalla de pesaje
│   │   │   ├── productos/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── nuevo/page.tsx
│   │   │   │   ├── [id]/page.tsx
│   │   │   │   └── importar/page.tsx           # carga masiva CSV
│   │   │   ├── categorias/page.tsx
│   │   │   ├── stock/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── movimientos/page.tsx
│   │   │   │   └── camadas/page.tsx            # pollitos BB por fecha
│   │   │   ├── precios/
│   │   │   │   ├── listas/page.tsx
│   │   │   │   └── actualizacion-masiva/page.tsx
│   │   │   ├── clientes/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── [id]/page.tsx
│   │   │   │   └── solicitudes-mayoristas/page.tsx
│   │   │   ├── envios/
│   │   │   │   ├── zonas/page.tsx
│   │   │   │   └── hoja-de-ruta/page.tsx
│   │   │   ├── pagos/
│   │   │   │   ├── page.tsx
│   │   │   │   └── transferencias/page.tsx     # validar comprobantes
│   │   │   ├── cupones/page.tsx
│   │   │   ├── contenido/
│   │   │   │   ├── home/page.tsx
│   │   │   │   └── banners/page.tsx
│   │   │   ├── reportes/page.tsx
│   │   │   ├── usuarios/page.tsx
│   │   │   └── configuracion/page.tsx
│   │   │
│   │   ├── api/
│   │   │   ├── auth/[...nextauth]/route.ts
│   │   │   ├── webhooks/
│   │   │   │   ├── mercadopago/route.ts
│   │   │   │   └── qstash/route.ts
│   │   │   ├── search/suggest/route.ts         # autocompletado
│   │   │   └── health/route.ts
│   │   │
│   │   ├── sitemap.ts
│   │   ├── robots.ts
│   │   ├── manifest.ts
│   │   ├── global-error.tsx
│   │   └── not-found.tsx
│   │
│   ├── modules/                      # ── LÓGICA DE NEGOCIO ────────────
│   │   ├── catalog/
│   │   │   ├── domain/
│   │   │   │   ├── product.ts                  # entidad + reglas puras
│   │   │   │   ├── category.ts
│   │   │   │   └── types.ts
│   │   │   ├── application/
│   │   │   │   ├── get-product-detail.ts
│   │   │   │   ├── list-products.ts
│   │   │   │   ├── search-products.ts
│   │   │   │   └── upsert-product.ts
│   │   │   ├── infrastructure/
│   │   │   │   ├── product.repository.ts
│   │   │   │   └── search.repository.ts
│   │   │   ├── ui/
│   │   │   │   ├── product-card.tsx
│   │   │   │   ├── product-gallery.tsx
│   │   │   │   ├── product-grid.tsx
│   │   │   │   ├── category-nav.tsx
│   │   │   │   └── filters/
│   │   │   ├── actions.ts                      # Server Actions del módulo
│   │   │   ├── schemas.ts                      # Zod
│   │   │   └── index.ts                        # ÚNICA superficie pública
│   │   │
│   │   ├── pricing/                            # precios, listas, IVA, cupones
│   │   ├── inventory/                          # stock, lotes, reservas, camadas
│   │   ├── cart/
│   │   ├── checkout/
│   │   ├── orders/                             # incluye ajuste por peso real
│   │   ├── payments/                           # mercadopago/ y transfer/
│   │   ├── shipping/
│   │   ├── customers/
│   │   ├── notifications/                      # email/, whatsapp/, templates/
│   │   ├── cms/
│   │   └── reporting/
│   │       └── (misma estructura interna en todos)
│   │
│   ├── shared/                       # ── TRANSVERSAL ──────────────────
│   │   ├── ui/                                 # design system, sin negocio
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── sheet.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── data-table.tsx
│   │   │   ├── price.tsx                       # formato $ ARS unificado
│   │   │   ├── empty-state.tsx
│   │   │   └── skeleton.tsx
│   │   ├── layout/
│   │   │   ├── header.tsx
│   │   │   ├── mobile-nav.tsx
│   │   │   ├── footer.tsx
│   │   │   └── whatsapp-fab.tsx
│   │   ├── lib/
│   │   │   ├── db.ts                           # cliente Prisma singleton
│   │   │   ├── redis.ts
│   │   │   ├── auth.ts
│   │   │   ├── events.ts                       # bus de eventos de dominio
│   │   │   ├── money.ts                        # aritmética en centavos
│   │   │   ├── weight.ts                       # aritmética en gramos
│   │   │   ├── logger.ts
│   │   │   ├── rate-limit.ts
│   │   │   ├── slugify.ts
│   │   │   └── result.ts                       # Result<T, E>
│   │   ├── hooks/
│   │   ├── config/
│   │   │   ├── site.ts                         # datos de JB, contacto, redes
│   │   │   ├── navigation.ts
│   │   │   └── constants.ts
│   │   └── types/
│   │
│   ├── middleware.ts                 # protección de /admin y /mi-cuenta
│   └── styles/
│       ├── globals.css
│       └── tokens.css                # variables del design system
│
├── tests/
│   ├── unit/                         # dominio puro
│   ├── integration/                  # casos de uso con DB de test
│   └── e2e/
│       ├── compra-invitado.spec.ts
│       ├── compra-mayorista.spec.ts
│       └── admin-pedidos.spec.ts
│
├── .env.example
├── docker-compose.yml                # Postgres + Redis local
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── vitest.config.ts
├── playwright.config.ts
└── package.json
```

---

## Reglas de la estructura

1. **`modules/*/index.ts` es la única puerta de entrada.** Importar
   `modules/orders/infrastructure/...` desde otro módulo está prohibido y se
   bloquea con una regla de ESLint (`import/no-restricted-paths`).

2. **`shared/ui` no conoce el negocio.** Un `<Button>` no sabe qué es un pollo.
   Si un componente menciona "producto" o "pedido", va en el módulo correspondiente.

3. **`app/` no tiene lógica.** Las páginas arman datos y componen UI. Si un
   archivo dentro de `app/` supera ~150 líneas, hay lógica mal ubicada.

4. **El dinero se maneja en centavos enteros y el peso en gramos enteros.** Nunca
   `float`. `money.ts` y `weight.ts` son obligatorios; sumar precios con `+` sobre
   decimales genera diferencias de centavos que después aparecen en la conciliación.

5. **Nombres de rutas en español, código en inglés.** La URL es parte del producto
   y del SEO (`/producto/pollo-entero-fresco`); el código sigue la convención
   universal del ecosistema. Es la combinación que menos fricción genera.

---

## Alias de importación

```jsonc
{
  "paths": {
    "@/*":         ["./src/*"],
    "@modules/*":  ["./src/modules/*"],
    "@shared/*":   ["./src/shared/*"],
    "@ui/*":       ["./src/shared/ui/*"]
  }
}
```

---

## Convenciones de nombres

| Elemento | Convención | Ejemplo |
|---|---|---|
| Archivos y carpetas | `kebab-case` | `product-card.tsx` |
| Componentes React | `PascalCase` | `ProductCard` |
| Funciones y variables | `camelCase` | `calcularPrecioFinal` |
| Tipos e interfaces | `PascalCase` | `OrderStatus` |
| Constantes globales | `SCREAMING_SNAKE` | `MAX_CART_ITEMS` |
| Tablas de la base | `snake_case` plural | `order_items` |
| Server Actions | verbo + sustantivo | `addToCartAction` |
