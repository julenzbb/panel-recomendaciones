# Guía del proyecto — Panel Recomendaciones

Panel de configuración (un único HTML autocontenido) para generar el JSON de
recomendación de productos. Se distribuye como **web** (GitHub Pages) y como **app
de escritorio** (Electron).

- Fuente única de la UI: `renderer/index.html`. **Todo el panel se edita aquí.**
- Web pública: https://julenzbb.github.io/panel-recomendaciones/
- Deploy web automático: `.github/workflows/deploy-web.yml` (push a `main`).
- Build apps + Release: `.github/workflows/build.yml` (al crear tag `vX.Y.Z`).

## REGLA OBLIGATORIA: analítica y error tracking en cada cambio

Este proyecto usa **PostHog** (instancia UE) para analítica de uso **y** error
tracking. Al añadir o modificar funcionalidad, **siempre** hay que instrumentar el
comportamiento nuevo. No se considera completo un cambio de UI sin su tracking.

### Cómo añadir un evento

Usar el helper global `track(nombre, props)` ya definido en `renderer/index.html`.
Es seguro: no rompe nada si PostHog no está cargado (app de escritorio, sin clave,
o bloqueado por adblock).

```js
track("nombre_evento", { propiedad: valor, plataforma: state.platform });
```

Para errores, usar `reportError(err, { contexto: "..." })` en los `catch`.

### Convenciones de nombrado

- Nombres de evento y propiedades en **español, snake_case** (coherencia con los
  existentes): `json_generado`, `modulo_anadido`, `caso_uso_seleccionado`.
- Verbo en pasado para acciones completadas: `_anadido`, `_eliminado`, `_generado`.
- Propiedades **planas** (PostHog filtra/agrupa mejor): nada de objetos anidados.
  Para listas, pasar arrays de strings (p.ej. `tipos_modulo: ["crossSelling"]`).
- Incluir siempre `plataforma: state.platform` cuando tenga sentido.

### Qué instrumentar al añadir features

- **Nuevo botón/acción principal** → un evento al ejecutarse.
- **Nuevo flujo de generación/exportación** → enriquecer o reutilizar
  `summarizeConfig(json)` para mandar el desglose (tipos de módulo, fuentes, validación).
- **Nuevo punto de fallo** (parseo, red, validación) → `reportError()` en el catch.
- **Nuevos campos de configuración relevantes** → reflejarlos en `summarizeConfig()`.

### Eventos actuales (mantener esta lista al día)

| Evento | Cuándo | Props clave |
|---|---|---|
| `app_iniciada` | Al cargar | `entorno` (web/app_escritorio), `plataforma_inicial` |
| `plataforma_cambiada` | Cambio app/web | `plataforma` |
| `vista_web_cambiada` | Cambio mobile/desktop | `vista` |
| `modal_caso_uso_abierto` | Abrir modal casos de uso | `plataforma` |
| `caso_uso_seleccionado` | Elegir un caso de uso | `caso`, `grupo`, `plataforma` |
| `modulo_anadido` | Añadir módulo | `metodo` (vacio/caso_uso), `total_modulos` |
| `modulo_eliminado` | Eliminar módulo | `tipo`, `total_modulos` |
| `config_importada` | Importar JSON OK | `plataforma` |
| `config_importada_error` | Importar JSON inválido | `mensaje` |
| `config_reseteada` | Botón borrar todo | — |
| `json_generado` | Copiar o descargar JSON | `metodo`, + `summarizeConfig()`, + tiempo al primer JSON |

`summarizeConfig(json)` aporta: `num_modulos`, `num_fuentes`, `tipos_modulo`,
`tipo_modulo_principal`, `fuentes_usadas`, `modulos_con_layout`,
`modulos_con_analytics`, `validacion_errores`, `validacion_avisos`, `validacion_ok`.

### Configuración PostHog

- Clave (`phc_...`) y host UE están en el `<head>` de `renderer/index.html`.
- La `phc_` es **pública por diseño** (va en el frontend); es correcto que esté en el repo.
- La analítica **solo se activa en web** (`http/https`); en la app de escritorio
  (`file:`) queda inerte a propósito, para no meter ruido.

## Despliegue

- Web: cualquier push a `main` que toque `renderer/` se publica solo en Pages.
- Apps: `git tag vX.Y.Z && git push origin vX.Y.Z` dispara build + Release.
