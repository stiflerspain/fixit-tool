# FIXIT SPA - Centro de control · Importacion & ML

## Que es
Herramienta web standalone para FIXIT SPA que calcula costos de importacion desde China (Alibaba/Aliexpress) y precios de venta en Mercado Libre Chile.

## Stack
- **HTML standalone** (`index.html`) — todo en un solo archivo
- React 18 via CDN (unpkg)
- Babel standalone para compilar JSX en el navegador
- Sin build system, sin Node, sin bundler
- Desplegado en GitHub Pages: `https://stiflerspain.github.io/fixit-tool/`

## Estructura de la app (dentro de index.html)
El archivo contiene CSS + JSX inline con los siguientes modulos:

### Pestanas principales
1. **Calculadora** (`TabPricing`) — Calcula costo en bodega y precio sugerido ML para un producto
2. **Dashboard** (`TabDashboard`) — Datos reales de ML: KPIs globales, cards por producto con margen real, selector de cotizaciones, filtro pausadas, orden por ventas
3. **Comparador** (`TabComparador`) — Aliexpress vs Alibaba lado a lado
4. **Escalamiento** (`TabEscalamiento`) — Simula cuando conviene saltar de Aliexpress a Alibaba
5. **Publicidad** (`TabPublicidad`) — Analisis ROAS/ACOS y simulador de escenarios

### Componentes compartidos
- `ParamsPanel` — Parametros globales (USD/CLP, arancel, IVA, comisiones ML, margen objetivo)
- `UsdClpField` — Obtiene tipo de cambio automatico desde Frankfurter API
- `Field` / `Toggle` — Inputs reutilizables
- `useDriveStorage` — Hook para persistencia en Google Drive

### Integracion Google Drive
- Usa Google Drive API con OAuth 2.0 para guardar/cargar cotizaciones
- Scope restringido: `drive.file` (solo archivos creados por la app)
- Carpeta compartida entre socios para datos sincronizados
- Credenciales OAuth configuradas en Google Cloud Console

## Logica de negocio clave
- **CIF** = precio unitario + flete por unidad (en CLP)
- **Arancel** = % sobre CIF (default 6%, desactivable por TLC)
- **IVA importacion** = % sobre (CIF + arancel), puede ser recuperable como credito fiscal
- **Honorarios DHL** = fuera del CIF, siempre son costo
- **Costo en bodega** = CIF + arancel + IVA (si no es recuperable) + DHL
- **Precio sugerido ML** = costo en bodega / (1 - comision - full - ads - margen)

## Convencion de monedas
- Costos de proveedor: USD
- Costos internos y precios ML: CLP
- Formato numerico: es-CL (punto para miles, coma para decimales)

## Para modificar
- Todo el codigo esta en `index.html` dentro de un tag `<script type="text/babel">`
- Los estilos estan en el objeto `S` (shared styles) y en CSS dentro de `<style>`
- Los parametros default estan en `DEFAULT_PARAMS`
- Para agregar una pestana: agregar entrada en `TABS[]` y renderizar en `App()`

## Integracion MercadoLibre API
- **MeliService** — Modulo de autenticacion OAuth 2.0 con PKCE flow + client_secret (ofuscado con XOR). Ubicado despues de DriveService.
- **useMercadoLibre hook** — Maneja estado de conexion ML, fetch de items/ordenes/envios/visitas por rango de fechas.
- **MlMappingService** — Guarda/carga `ml-mappings.json` en Drive para asociar cotizaciones con productos ML.
- **TabDashboard reemplazado** — Dashboard con datos reales de ML: selector de fechas (hoy/7d/30d/custom), 6 KPIs globales, cards por producto con metricas reales, selector de cotizaciones multi-select con costo ponderado. Ordenado por ventas desc. Filtro para ocultar pausadas.
- **App ML Client ID**: `8682313366878154`
- **Client Secret**: ofuscado con XOR (key: FIXIT) — no en texto plano
- **Endpoints usados**: `/users/me`, `/users/{id}/items/search`, `/items?ids=`, `/orders/search`, `/shipments/{id}/costs`, `/items/visits`
- **CORS**: ML API no acepta header Authorization desde browser (preflight bloqueado). Se usa `?access_token=` como query param.
- **Envio**: solo se cuenta `senders[0].cost` (costo real del vendedor). Envio gratis = $0.

## Changelog

### 2026-03-25
- **feat: integracion MercadoLibre API** — Nuevo modulo `MeliService` con OAuth PKCE + client_secret ofuscado, `useMercadoLibre` hook, dashboard con datos reales. Selector de fechas (hoy, 7 dias, 30 dias, personalizado). 6 KPIs globales + cards por producto con margen real. Selector de cotizaciones multi-select para asignar costo de importacion con calculo ponderado. Mappings guardados en Drive como `ml-mappings.json`. Indicador ML en header.
- **fix: CORS ML API** — ML bloquea preflight con header Authorization. Cambiado a `?access_token=` como query param.
- **fix: client_secret requerido** — ML requiere client_secret incluso con PKCE. Agregado ofuscado con XOR (key FIXIT).
- **fix: envio vendedor** — Solo se cuenta `senders[0].cost` (costo real del vendedor, $0 en envio gratis). Antes restaba costos del comprador incorrectamente.
- **feat: ordenar por ventas y filtro pausadas** — Productos ordenados de mayor a menor ventas. Checkbox "Ocultar pausadas" en controles del dashboard.
- **fix: iconos botones** — Reemplazados HTML entities (`&#8635;`, `&orarr;`) por Unicode escapes (`\u21BB`) en strings JS para que se rendericen correctamente en JSX.

### 2026-03-23
- **feat: restaurar parametros al cargar cotizacion** — `loadCotizacion()` ahora restaura `parametrosSnapshot` (USD/CLP, arancel, IVA, comisiones ML, margen) para saber exactamente como fue evaluada. `handleNew()` resetea a `DEFAULT_PARAMS`. Notificacion visual al restaurar.

### 2026-03-22 — d852dfa
- **fix: carpetas duplicadas en Drive** — `ensureFolderStructure()` se ejecutaba concurrentemente (restore + OAuth callback). Agregado mutex (`_ensurePromise`) y early return si IDs ya estan cacheados.
- **feat: responsive movil** — Media queries para <768px y <480px. Grids colapsan a 1 columna, tabs con scroll horizontal, header apilado, panel cotizaciones full-width, padding reducido.

### 2026-03-22 — f379a64
- **feat: credenciales Google Drive** — Configurado CLIENT_ID y API_KEY reales del proyecto FIXIT Tool en Google Cloud Console (cuenta fixitspacl@gmail.com).
- **feat: modo publicidad fijo** — Toggle entre % del precio y costo fijo CLP por unidad para ads en parametros globales (`mlAdsMode`, `mlAdsFijo`).
- **feat: desglose precio manual** — Copiada la distribucion del precio de venta sugerido en la seccion simulador de precio manual (sin precio minimo).
