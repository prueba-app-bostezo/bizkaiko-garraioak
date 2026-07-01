# Bizkaiko Garraioak — Transporte de Bizkaia en directo

Portal web que reúne en un único sitio información sobre las redes de
transporte público de Bizkaia (posición de vehículos en tiempo real cuando
está disponible, líneas, paradas, horarios y buscador de itinerarios).

🔗 Sitio en producción: **https://bizkaidev.github.io/bizkaiko-garraioak/**

> ⚠️ Este proyecto **no está afiliado a ninguna institución** (Consorcio de
> Transportes de Bizkaia, Diputación, Euskotren, etc.). Toda la información
> se obtiene de fuentes públicas oficiales y se reprocesa aquí para
> ofrecer una vista unificada. Más detalle en la página [FAQ](faq.html).

## Qué ofrece cada página

| Página | Red | Estado | Qué hace |
|---|---|---|---|
| `index.html` | — | — | Portal de entrada con una tarjeta por red y su estado actual |
| `bizkaibus-mapa.html` | Bizkaibus (interurbano) | 🟢 En directo | Mapa con la posición de los autobuses actualizada periódicamente, filtro por línea, panel de itinerario (paradas de ida/vuelta) y un buscador de trayecto por origen/destino/fecha que calcula rutas directas y con trasbordo a partir de los horarios precalculados |
| `metro-mapa.html` | Metro Bilbao (L1/L2) | 🟢 En directo | Mapa con la posición de los trenes en tiempo real, filtro por línea |
| `bilbobus-mapa.html` | Bilbobus (urbano) | 🟡 Construido, no enlazado aún | Mismo tipo de mapa en directo que Bizkaibus; ya funciona pero `index.html` todavía lo marca como "próximamente" y no lo enlaza desde el portal |
| `euskotren-info.html` | Euskotren | 🔵 Solo información | Euskotren no publica la posición de sus trenes en ninguna fuente abierta, así que esta página muestra el trazado estático de las líneas de Bizkaia (E1, E3, E4, L2, L3 y el Tranvía) más un buscador de horarios aproximados por municipio y fecha |
| `euskotren-horarios.html` | Euskotren | ⚪ Sin enlazar | Buscador de horarios de Euskotren independiente (versión previa a la integrada en `euskotren-info.html`); no está enlazado desde ninguna otra página actualmente |
| `faq.html` | — | — | Preguntas frecuentes, disponible en castellano y euskera |
| Renfe Cercanías | — | ⚪ Próximamente | Sin funcionalidad todavía; solo aparece como tarjeta "próximamente" en el portal |

## Cómo funciona (arquitectura actual)

Es un sitio estático (sin build ni framework): HTML, CSS y JavaScript planos,
con [Leaflet](https://leafletjs.com/) y teselas de OpenStreetMap para los
mapas.

- **Datos en directo (Bizkaibus, Metro Bilbao, Bilbobus):** cada mapa hace
  `fetch` desde el propio navegador, a intervalos configurables (10–60 s),
  contra un *worker* de Cloudflare (`https://bizkaibus.cuentasmurf7.workers.dev`,
  con el parámetro `?feed=metro` o `?feed=bilbobus` según la red) que actúa
  de proxy de los feeds oficiales SIRI Vehicle Monitoring y añade las
  cabeceras CORS que el feed original no envía. El sitio **ya no lee un
  fichero estático propio** para pintar los vehículos en el mapa.
- **Datos de referencia (líneas, paradas, horarios, itinerarios):** están
  precalculados y versionados en el repo, en `feeds/*.json` y en
  `horarios_verano.json`. Los usan el buscador de trayectos de Bizkaibus y
  el buscador de horarios de Euskotren; no se descargan en tiempo real.
- **`.github/workflows/sync-bizkaibus.yml`:** Action que se ejecuta cada
  5 minutos (`cron: "*/5 * * * *"`), en cada `push` a `main` y también de
  forma manual. Descarga los tres feeds SIRI oficiales de Bizkaibus
  (posiciones, retrasos, alertas) en XML dentro de `feeds/`, y publica todo
  el sitio en GitHub Pages con `actions/upload-pages-artifact` +
  `actions/deploy-pages`. Estos XML quedan archivados junto al resto de
  ficheros, pero **no son la fuente que leen los mapas en directo** (que
  usan el worker de Cloudflare); sirven de copia/histórico bruto del feed.
- **Sin backend propio ni base de datos:** todo el procesado de rutas,
  trasbordos y horarios ocurre en el navegador con los JSON estáticos.

## Estructura del repositorio

```
bizkaiko-garraioak/
├── .github/workflows/sync-bizkaibus.yml   # cron cada 5 min: archiva feeds SIRI + publica Pages
├── index.html                             # portal principal
├── bizkaibus-mapa.html                    # mapa en directo + buscador de trayecto
├── metro-mapa.html                        # mapa en directo Metro Bilbao
├── bilbobus-mapa.html                     # mapa en directo Bilbobus (sin enlazar en el portal)
├── euskotren-info.html                    # trazado estático + buscador de horarios
├── euskotren-horarios.html                # buscador de horarios independiente (legacy)
├── faq.html                               # FAQ en castellano y euskera
├── horarios_verano.json                   # horarios usados por euskotren-horarios.html
├── feeds/
│   ├── horarios_semanales.json            # horarios de Bizkaibus por línea/día
│   ├── itinerarios_lineas.json            # trazado e itinerario de cada línea de Bizkaibus
│   ├── indice_paradas.json                # índice de paradas
│   ├── lista_paradas.json                 # listado de paradas
│   ├── parada_municipio.json              # relación parada → municipio
│   ├── stops_data.json                    # datos de paradas
│   ├── bizkaibus-vehicle-positions.xml    # archivado por el workflow (histórico)
│   ├── bizkaibus-trip-updates.xml         # archivado por el workflow (histórico)
│   └── bizkaibus-service-alerts.xml       # archivado por el workflow (histórico)
└── img/fondo-portada.jpg
```

## Desarrollo local

No hay dependencias ni paso de build: basta con servir la carpeta como
archivos estáticos, por ejemplo:

```bash
python3 -m http.server 8000
# abrir http://localhost:8000
```

Ten en cuenta que los mapas en directo necesitan poder llegar al worker de
Cloudflare (`bizkaibus.cuentasmurf7.workers.dev`) desde tu navegador para
mostrar vehículos.

## Publicación (GitHub Pages)

El despliegue lo gestiona por completo el workflow
`.github/workflows/sync-bizkaibus.yml`: en **Settings → Pages**, el *Source*
debe estar configurado como **GitHub Actions** (no "Deploy from a branch").
Cada ejecución programada, cada `push` a `main` o cada disparo manual desde
la pestaña **Actions** vuelve a publicar el sitio completo.

## Aviso legal

Según se indica en la propia web ([FAQ](faq.html)): el proyecto no tiene
ánimo de lucro ni monetización, no sustituye a las fuentes oficiales y toda
la información se obtiene de canales públicos de las instituciones
vizcaínas. No se declara ninguna licencia de código en el repositorio.
