# Panel Recomendaciones (App de escritorio)

App de escritorio (Mac + Windows) que abre el **Panel de Configuración** para generar
el JSON de recomendación de productos. El panel en sí es un único HTML autocontenido
(`renderer/index.html`), empaquetado con **Electron**.

## Estructura

```
.
├── renderer/index.html      # El panel (la UI). Editar aquí para cambiar el panel.
├── main.js                  # Proceso principal de Electron (ventana, menú).
├── preload.js               # Vacío a propósito (seguridad).
├── package.json             # Config de la app + electron-builder.
├── build/                   # Iconos: icon.icns (Mac), icon.ico (Windows).
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
