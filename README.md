# Bizkaibus en directo — mapa de posición de autobuses

Esta carpeta es un repositorio completo listo para subir a GitHub. Una vez
publicado, tendrás una web propia (gratis, en `https://TU-USUARIO.github.io/TU-REPO/`)
que muestra los autobuses de Bizkaibus en tiempo real sobre un mapa,
sin depender de ningún proxy externo ni de cuentas de terceros.

## Por qué este enfoque

El feed oficial de Bizkaibus (formato SIRI, publicado por el Consorcio de
Transportes de Bizkaia) no envía cabeceras CORS, así que un navegador no
puede leerlo directamente desde una página en otro dominio. La solución más
estable es que un proceso de servidor (no un navegador) descargue el XML
periódicamente y lo publique como archivo estático junto a la propia app:
al estar ambos en el mismo dominio, CORS deja de ser un problema.

Aquí ese "proceso de servidor" es una GitHub Action gratuita que se ejecuta
cada 2 minutos.

## Pasos para desplegarlo

### 1. Crear el repositorio
1. Entra en GitHub (ya tienes cuenta) y pulsa **New repository**.
2. Ponle un nombre, por ejemplo `bizkaibus-mapa`. Puede ser público o privado
   (con privado, GitHub Pages también funciona en el plan gratuito).
3. Crea el repositorio vacío (sin README, sin .gitignore).

### 2. Subir estos archivos
Sube el contenido completo de esta carpeta al repositorio, manteniendo la
estructura de carpetas:

```
tu-repo/
├── .github/
│   └── workflows/
│       └── actualizar-feed.yml
├── bizkaibus-mapa.html
└── feeds/
    └── .gitkeep
```

La forma más sencilla si no usas git desde la terminal: en la página del
repo en GitHub, usa **Add file → Upload files**, arrastra todos los archivos
y carpetas, y confirma el commit. GitHub respeta la estructura de carpetas
al arrastrar.

### 3. Activar GitHub Pages
1. En el repositorio, ve a **Settings → Pages**.
2. En "Build and deployment", en el campo **Source**, elige **GitHub Actions**
   (no "Deploy from a branch").
3. Guarda. No hace falta nada más aquí; el propio workflow se encarga de
   publicar.

### 4. Lanzar la primera ejecución
1. Ve a la pestaña **Actions** del repositorio.
2. Si no se ha lanzado sola al subir los archivos, verás el workflow
   "Actualizar feed Bizkaibus" en la lista. Pulsa sobre él y luego en
   **Run workflow** (botón a la derecha) para forzar la primera ejecución.
3. Espera a que el job termine (ícono verde ✓, tarda menos de un minuto).

### 5. Abrir tu web
Tu mapa estará disponible en:

```
https://TU-USUARIO.github.io/TU-REPO/
```

Sustituye `TU-USUARIO` por tu nombre de usuario de GitHub y `TU-REPO` por el
nombre que le hayas puesto al repositorio. GitHub también te muestra esta URL
exacta en Settings → Pages una vez activado.

## Cómo se mantiene actualizado

El workflow (`.github/workflows/actualizar-feed.yml`) está programado para
ejecutarse automáticamente cada 2 minutos (`cron: "*/2 * * * *"`). Cada vez
que corre:
1. Descarga los tres feeds XML de Bizkaibus (posiciones, retrasos, alertas).
2. Los guarda en la carpeta `feeds/`.
3. Vuelve a publicar todo el sitio en GitHub Pages.

No necesitas hacer nada manualmente después del despliegue inicial.

**Nota sobre la frecuencia real:** GitHub Actions con triggers `schedule` en
el plan gratuito no garantiza que el cron se dispare exactamente cada 2
minutos — en periodos de mucha carga en GitHub puede retrasarse algunos
minutos. Para este caso de uso (posición de autobuses) un margen de uno o
dos minutos no debería notarse demasiado, pero si necesitas algo más
puntual, esa es una limitación a tener en cuenta.

## Si quieres cambiar algo

- **Frecuencia de actualización**: cambia el valor de `cron` en
  `actualizar-feed.yml`. Por ejemplo `*/5 * * * *` para cada 5 minutos
  (recomendable si quieres ir sobre seguro con los límites de minutos
  gratuitos de Actions, aunque para un repo personal no debería ser problema).
- **Frecuencia de refresco en el navegador**: el propio mapa tiene un
  desplegable para elegir cada cuántos segundos vuelve a leer el archivo
  ya publicado (esto no descarga nada de Bizkaia, solo relee tu copia).
- **Volver al feed directo de Bizkaia** (sin pasar por tu copia): dentro de
  `bizkaibus-mapa.html`, busca la constante `URL_FEED` y sustitúyela por la
  URL original que aparece comentada justo encima. Ten en cuenta que esto
  reintroduce el problema de CORS que motivó esta solución.
