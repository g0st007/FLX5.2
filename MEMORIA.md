# MEMORIA DEL PROYECTO FLUX

Este archivo reemplaza a las capturas de "memoria" de Claude. Si otra persona (o otra IA) continúa el
proyecto, debe leerlo completo antes de tocar nada y **actualizarlo en cada entrega**
(sección "Estado actual" y "Historial"). Se usa igual que la memoria: es lo que se sabe del dueño, de lo que
quiere y de lo que ya pasó.

## 1. Qué es Flux y qué quiere el dueño
- App Android (Kotlin + Jetpack Compose): descargador de video/audio con yt-dlp + FFmpeg, navegador propio con
  bloqueo de anuncios y detector de videos, reproductor propio (Media3), bóveda con código y gestor de archivos.
- Es un proyecto profesional. Quiere perfeccionarlo hasta ser **la mejor app del mercado**, tomando lo mejor de
  apps parecidas pero todo en uno, mejorando lo que a las otras se les critica y cubriendo lo que les falta.
- Cree que los **detalles y el profesionalismo** hacen que un usuario elija una app sobre otra.
- La tipografía la elige Claude. El mensaje de la pantalla principal es lo primero que ve el usuario: marketing
  que atrape (p. ej. cantidad de sitios compatibles), no texto insípido. Texto principal blanco en modo oscuro
  y negro en claro. Las capturas de otras apps son solo inspiración: no copiar textos ni colores (no van con la
  paleta de Flux); la tipografía sí puede inspirar.
- Ofrece adjuntar imágenes/videos de inspiración y enlaces de apps de referencia para funciones.
- Referencias que usa: **Aloha** (navegador, gestor de archivos/descargas, reproductor propio),
  **Video DownloadHelper** (detección casi instantánea y genérica), y otro descargador que lista TODAS las
  calidades con peso (el estándar a igualar).
- Escribe en español.

## 2. Reglas de entrega (preferencias del dueño)
1. En el zip de cada entrega incluir `referencias/` con imágenes de cómo debería verse cada sección con los
   cambios nuevos, para no tener que compilar por cada función agregada.
2. En cambios pequeños, `referencias/` lleva **solo** la imagen de la nueva adición (no todas las secciones).
3. Incluir este `MEMORIA.md` actualizado en la raíz del zip.
4. Los cambios siempre parten de la **última versión que compiló y abrió** (ver Historial).
5. No hay forma de compilar dentro del chat: revisar el código con mucho cuidado (imports, llaves, nombres)
   antes de entregar. Un error de compilación le cuesta una vuelta entera al dueño.

## 3. Estructura del código (`app/src/main/java/app/flux/downloader/`)
- `engine/` descargas: `Engine` (yt-dlp/FFmpeg), `MediaProbe` (lee HLS/DASH/directos: calidades, duración,
  peso), `NativeDownloader`.
- `browser/` `BrowserController` (WebView, política de seguridad, detección con `SNIFF_JS`, redirecciones,
  pop-ups, menú de enlaces), `BrowserScreen` (UI del navegador y selector de descarga), `DetectedMedia`
  (modelo + `cleanDetected`: une calidades del mismo video y filtra basura).
- `adblock/` `FilterManager`/`FilterEngine` (listas EasyList, uBlock, etc.).
- `ui/` pantallas y reproductor; `vault/` carpeta privada; `settings/` ajustes.

## 4. Decisiones de diseño ya tomadas
- Selector de descarga: cada video con **todas** sus calidades y **peso** a la vista; "Reproduciendo ahora"
  arriba; en galerías se pueden elegir varios y hay "Todos" (sin duplicados del mismo video) y también
  descargar solo el que suena. Las calidades de un mismo video se ven para elegir; "Todos" no repite.
- Al darle Descargar en la pestaña Descargas, el enlace se limpia solo y desaparece la tarjeta de análisis.
- Progreso de descarga configurable en Ajustes; por defecto "MB descargados y peso total" (no escribir "MB/GB").
- El navegador viene por defecto en modo escritorio. Las descargas de Reddit pasan por RedGifs (si no, error
  por depender del inicio de sesión).
- Pantalla completa con el reproductor propio de Flux (con funciones que el del sitio no trae); la barra
  "Video detectado" tiene botón de pantalla completa.
- Pestaña "Archivos" (carpeta de Flux, carpeta privada…) tipo gestor de Aloha.

## 5. Estado actual: v0.5.2 (parte de v0.5.0 → 0.5.1 → 0.5.2)
### Hecho en 0.5.2 (sin compilar aún)
- Un anuncio ya no se une al video como si fuera una calidad: `sameVideo` descarta la unión si las duraciones
  conocidas difieren más de 2 s; la portada igual solo une si ambas duraciones se conocen (o ninguna). Causa
  del bug: el anuncio y el video comparten `<video>` y portada, y se unían por la portada.
- "Reproduciendo · 240p" siendo 1080p: ahora se usa la resolución real del `<video>` (`playingHeight`,
  evento `resize` incluido) en la barra "Video detectado".
- Calidades sin tener que reproducirlas: el script de detección también lee las respuestas JSON/JS de fetch y
  XHR (p. ej. la lista `mediaDefinitions` que muchas páginas piden aparte) y saca de ahí las URL de medios, con
  la calidad más cercana a cada URL (`qHint`). Antes solo se leía el HTML inicial.
- Tope de detecciones subido de 40 a 100 (la basura ya no ahoga a lo real).

### Hecho en 0.5.1 (sin compilar aún)
### Hecho en 0.5.1 (sin compilar aún)
- Detección sin basura (`cleanDetected` en `DetectedMedia.kt`):
  - si la página tiene un video largo (≥ 2 min) se descartan los clips < 45 s (anuncios previos, previews,
    miniaturas animadas) y los directos < 8 MB sin duración;
  - las listas de streaming sin duración pasan a ser calidades del video principal;
  - si lo que "sonaba" era un anuncio, "Reproduciendo ahora" pasa al contenido real;
  - el video sin título toma el título de la página; el contador del botón Descargar usa la cuenta limpia.
- `BrowserController`: un medio que las listas de filtros marcan como anuncio no se lista como video; el script
  de detección (`junkEl`) ignora `<video>` diminutos, con `loop`+`muted` corto o dentro de contenedores de
  anuncio/preview.
- `MediaProbe`: si la primera lista de medios HLS no se lee, prueba otras variantes (antes quedaba sin
  duración ni peso y salía como "Video N").
- Las filas sin título muestran calidad y duración en vez de "Video N".

### Pendiente / por verificar con el dueño
- Verificar en Pornhub que salgan 1080/720/480/240 reales sin reproducirlas (si el extractor del sitio devuelve
  menos calidades que el otro descargador, mirar `Engine.parseInfo`: solo cuenta formatos con alto > 0).
- Verificar en dispositivo la detección en Pornhub, MissAV y sitios genéricos tras 0.5.1 (el dueño manda videos).
- Miniaturas que no cargan en la propia página del navegador aunque se desactiven opciones de seguridad.
- Redirecciones y pop-ups: en 0.5.0 mejoraron (menú al mantener pulsado un enlace funciona); vigilar que no
  vuelvan a colarse con el bloqueo activado y que volver atrás funcione tras una redirección bloqueada.
- Bloqueador de anuncios: a veces deja pasar anuncios invasivos.
- Parpadeo inicial al cambiar de cualquier pestaña al navegador.
- Algunas páginas solo cargan en modo escritorio (y otras al revés): no depender solo de eso.
- Error al descargar de sitios genéricos: "ERROR: The extracted extension ('php') is unusual…".
- Falla "[generic] Unable to extract flashvars" al analizar en Descargas.

## 6. Historial de versiones
- v0.1.2: primera que compiló y abrió (los cambios siguientes partieron de ella).
- v0.2.0: no compilaba (referencia a `mediaRegex`); la corregida es FLX3-main.
- v0.4.1: reproductor propio, selector de galerías, progreso configurable.
- v0.5.0: todas las calidades unidas por video, menú al mantener pulsado un enlace, pestaña en segundo plano.
  Regresión: mucha basura detectada (anuncios y previews como videos) y "Reproduciendo ahora" mostraba el
  anuncio previo.
- v0.5.1: filtro de basura en la detección (ver sección 5).
- v0.5.2: anuncio ya no cuenta como calidad, resolución real en "Reproduciendo", calidades desde respuestas JSON.

## 7. Sobre las pruebas del dueño
- Manda videos de pantalla de sitios genéricos (incluye NSFW, solo como prueba) para comparar Flux con el otro
  descargador y con Aloha. Se usan únicamente para pulir la detección y descarga.
