# Panel Recomendaciones

Panel de configuración para generar el JSON de recomendación de productos.
El panel es un único HTML autocontenido (`renderer/index.html`).

Se puede usar de dos formas:

## 1. Versión web (recomendada — sin instalar nada)

**https://julenzbb.github.io/panel-recomendaciones/**

Solo abrir el enlace en el navegador. Es la forma más sencilla para equipos
de negocio: cero instalación, siempre actualizada, sin avisos de seguridad.
Se actualiza automáticamente en cada cambio que se sube a `main`.

> El panel funciona 100% en local en el navegador (no envía datos a ningún
> servidor); solo genera el JSON que luego copias o descargas.

## 2. App de escritorio (Mac + Windows)

Empaquetada con **Electron**. Útil para trabajar sin conexión o tener un icono
en el dock/escritorio. Descargas en la sección **Releases** del repo.

> Nota macOS: la app no está firmada con certificado de Apple, así que la primera
> vez hay que abrirla con **clic derecho → Abrir** (o Ajustes del Sistema →
> Privacidad y Seguridad → "Abrir de todos modos"). Por eso se recomienda la web.

## Estructura

```
.
├── renderer/index.html      # El panel (la UI). Editar aquí para cambiar el panel.
├── main.js                  # Proceso principal de Electron (ventana, menú).
├── preload.js               # Vacío a propósito (seguridad).
├── package.json             # Config de la app + electron-builder.
├── build/                   # Iconos: icon.icns (Mac), icon.ico (Windows).
└── .github/workflows/
    ├── build.yml            # CI: compila apps Mac+Win y publica Release.
    └── deploy-web.yml       # CI: publica la web en GitHub Pages.
└── .github/workflows/build.yml  # CI: compila Mac+Win y publica Release.
```

> El panel completo vive en `renderer/index.html`. Para cambiar la UI o la lógica
> del panel, edita ese archivo. Puedes abrirlo directamente en el navegador
> (doble clic) para probar sin Electron.


## Requisitos

- **Node.js 18+** (recomendado 20). Descárgalo en https://nodejs.org
  (en este equipo no estaba instalado; instálalo antes de probar en local).

## Probar en local (Mac)

```bash
npm install
npm start
```

## Compilar instaladores en local

- **Mac (.dmg / .zip):**
  ```bash
  npm run dist:mac
  ```
- **Windows (.exe):** desde Mac NO es fiable. Usa GitHub Actions (abajo)
  o una máquina Windows con `npm run dist:win`.

Los resultados quedan en `dist/`.

## Compilar Mac + Windows automáticamente (recomendado)

El workflow `.github/workflows/build.yml` compila **ambas** plataformas en la nube.

1. Sube el proyecto a un repositorio de GitHub.
2. Crea y sube un tag de versión:
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```
3. GitHub Actions compilará Mac (.dmg/.zip) y Windows (.exe) y los publicará
   en una **Release** del repo, lista para descargar por el equipo.

También puedes lanzarlo a mano en la pestaña **Actions → Build desktop apps → Run workflow**
(en ese caso los binarios quedan como *artifacts* del run, sin crear Release).

## Iconos

Coloca tus iconos en `build/`:
- `build/icon.icns` (Mac)
- `build/icon.ico` (Windows, 256x256)

Si no los pones, Electron usará el icono por defecto (la app igual compila).

## Firma de código (opcional)

El workflow desactiva la firma (`CSC_IDENTITY_AUTO_DISCOVERY: false`) para que compile
sin certificados. Sin firma:
- **Mac:** al abrir por primera vez, botón derecho → *Abrir* (Gatekeeper).
- **Windows:** SmartScreen puede avisar; *Más información → Ejecutar de todos modos*.

Para distribución masiva sin avisos, habría que añadir certificados de firma
(Apple Developer ID y un cert de Windows) como secrets del repo.
