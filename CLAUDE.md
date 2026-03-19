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
2. **Dashboard** (`TabDashboard`) — Vista de multiples SKUs con margen real (ads vs organico)
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
