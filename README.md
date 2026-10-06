# bitchord-ia-addon

Addon de BitChord que busca FLAC en el Internet Archive (archive.org) — sin
cuenta, sin API key. Implementa el contrato oficial de 3 rutas:
https://bitchord.kushagrasingh.in/docs

- `GET /manifest.json`
- `GET /search?q=...`
- `GET /stream/:id`

## Cómo funciona

1. `/search` consulta `archive.org/advancedsearch.php` pidiendo items de
   audio que declaren formato FLAC.
2. Para cada item encontrado, pide su metadata (`archive.org/metadata/{id}`)
   y extrae los archivos `.flac` reales.
3. `/stream/:id` arma la URL directa de descarga
   (`archive.org/download/{id}/{archivo}`) — el audio no pasa por este
   servidor, solo la metadata.

Cachea búsquedas 5 min y metadata 30 min en memoria para no saturar
archive.org.

## Correrlo

```bash
npm install
npm start
```

Por defecto escucha en el puerto 8787. Prueba:

```
http://localhost:8787/manifest.json
http://localhost:8787/search?q=miles%20davis
```

**BitChord exige HTTPS**, así que `localhost` no sirve directo desde el
celular. Dos formas rápidas de tener una URL pública con HTTPS:

### Opción A — Desplegar en Render (gratis, recomendado)

1. Sube esta carpeta a un repo de GitHub.
2. En [render.com](https://render.com) → New → Web Service → conecta el repo.
3. Build command: `npm install` · Start command: `npm start`.
4. Render te da una URL `https://tu-app.onrender.com` con HTTPS automático.

### Opción B — Correrlo local + túnel

Si prefieres correrlo en tu propio servidor/PC/Termux:

```bash
npm start
# en otra terminal:
cloudflared tunnel --url http://localhost:8787
```

`cloudflared` te da una URL pública HTTPS que apunta a tu máquina.

## Agregarlo en BitChord

Settings → Sources → **Add an addon** → pega:

```
https://tu-app.onrender.com/manifest.json
```

(o solo la raíz `https://tu-app.onrender.com`, BitChord acepta ambas).

## Notas

- El catálogo de archive.org es enorme pero desigual: hay discos completos
  en FLAC real y también muchos items sin FLAC (esos quedan filtrados).
- Si una búsqueda no encuentra nada, BitChord sigue con la siguiente
  fuente habilitada (JioSaavn/YouTube Music) automáticamente.
- `allowDownloads: 1` — las descargas también vienen de este addon por
  defecto. Cámbialo a `0` en `server.js` si no quieres eso.
