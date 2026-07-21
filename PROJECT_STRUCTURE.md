# Clipdock Marketplace 3.0 — Estructura del Proyecto

**Repositorio:** `github.com/depsoniac/Clipdock-Marketplace-3.0`  
**Tipo:** Repositorio público de datos. No es una app ejecutable.  
**Distribuido mediante:** GitHub Pages / GitHub raw (rama `main`).

---

## Propósito

Este repositorio es la fuente de verdad del catálogo de plugins de ClipDock.  
ClipDock lo consume vía URLs raw de GitHub. Store Manager lo edita localmente y hace push.

---

## Archivos en la raíz

| Archivo | Descripción | ¿Público? |
|---|---|---|
| `catalog.json` | Configuración UI de la tienda: filtros, secciones, hero, botones. Editado con Store Manager o a mano. | ✅ URL pública |
| `catalog-resolved.json` | Catálogo completo con versiones, URLs de descarga, sha256, tamaños. **Generado automáticamente por GitHub Actions. No editar a mano.** | ✅ URL pública (URL principal que lee ClipDock) |
| `notifications.json` | Notificaciones activas que ClipDock muestra en su centro de notificaciones. Editado con Store Manager. | ✅ URL pública |
| `README.md` | Descripción del repositorio. | — |
| `.gitignore` | Excluye node_modules, caché, temporales. | — |

---

## Carpetas

### `plugins/`
Cada subcarpeta es un plugin. Convención de nombres: `id-del-plugin` en minúsculas con guiones.

```
plugins/
  index.json              ← índice de carpetas (autogenerado por Actions, NO editar)
  _template/              ← plantilla para crear plugins nuevos
    plugin.json
  audify/
    plugin.json
    assets/
      logo.png / logo.svg
      banner.png / banner.svg
      screenshots/
  ... (un directorio por plugin)
```

**Archivo `plugin.json`** — el único archivo obligatorio por plugin.  
Campos clave que consume ClipDock:

| Campo | Obligatorio | Descripción |
|---|---|---|
| `id` | ✅ | Identificador estable. **Nunca cambiar una vez publicado.** |
| `slug` | ✅ | Igual al id. Usado como nombre de carpeta de instalación. |
| `name` | ✅ | Nombre visible. |
| `version` | ✅ | Versión del manifiesto (semver). |
| `type` | ✅ | Tipo: `after-effects-script`, `adobe-cep`, `clipdock-window`, `clipdock-symbiont`. |
| `installMode` | ✅ | Modo de instalación: `download-ae-script`, `download-adobe-cep`, `clipdock-window`, `clipdock-symbiont`. |
| `enabled` | opcional | `false` oculta el plugin del catálogo. |
| `repository.owner` | recomendado | Owner de GitHub del repo del plugin. |
| `repository.repo` | recomendado | Nombre del repo del plugin en GitHub. |
| `release.assetName` | recomendado | Nombre exacto del archivo ZIP en los releases del repo. |
| `images.logo` | opcional | Ruta relativa del logo dentro de `assets/`. |
| `images.banner` | opcional | Ruta relativa del banner. |
| `images.screenshots` | opcional | Array de rutas de capturas. |
| `description` | opcional | Descripción corta. |
| `links.github` | opcional | URL del repo. |
| `links.release` | opcional | URL de releases. |

### `assets/`
Assets globales del marketplace (no pertenecen a un plugin específico).

| Archivo | Descripción |
|---|---|
| `notifications/release-v2-hero.svg` | SVG hero para notificación de release 2.0. Referenciado localmente en `ClipDock/assets/notifications/` (copia exacta). La notificación activa apunta a la imagen del repo de `clipdock-postits`, no a este archivo. |

### `scripts/`
| Archivo | Descripción |
|---|---|
| `build-catalog.mjs` | Robot que genera `catalog-resolved.json` e `index.json`. Lo ejecuta GitHub Actions. Puede correr localmente: `node scripts/build-catalog.mjs`. |

### `docs/`
| Archivo | Descripción |
|---|---|
| `plugin-repo-notify.yml` | Workflow de ejemplo para que repositorios de plugins disparen el rebuild del catálogo al publicar un release. |

### `.github/workflows/`
| Archivo | Descripción |
|---|---|
| `build-catalog.yml` | GitHub Action que regenera el catálogo. Se dispara en push a `main` (cuando cambia un `plugin.json`, `catalog.json` o el script), cada 6 horas, manualmente, y por `repository_dispatch` de tipo `plugin-released`. |

---

## Flujo de publicación

```
1. Store Manager edita plugin.json / catalog.json / notifications.json localmente
2. Store Manager hace commit + push a GitHub (rama main)
3. GitHub Action detecta el cambio y corre scripts/build-catalog.mjs
4. El script consulta la API de GitHub para resolver versiones y URLs de releases
5. Genera catalog-resolved.json y plugins/index.json
6. Hace commit automático con "[skip ci]" en el mensaje
7. ClipDock lee catalog-resolved.json en el próximo fetch
```

---

## Rutas públicas — NUNCA cambiar sin verificar consumidores

| URL | Consumido por |
|---|---|
| `.../main/catalog-resolved.json` | ClipDock (URL principal) |
| `.../main/catalog.json` | ClipDock (fallback), Store Manager |
| `.../main/notifications.json` | ClipDock (renderer/app.js) |
| `.../main/plugins/index.json` | ClipDock (discovery folder-index) |
| `.../main/plugins/<folder>/plugin.json` | ClipDock (expansión de catálogo) |
| `.../main/plugins/<folder>/assets/<file>` | ClipDock (logos, banners, capturas) |

---

## Conexiones con los otros proyectos

- **Store Manager** → edita este repositorio localmente y publica via `git push`.
- **ClipDock** → consume URLs raw de este repositorio. No modifica nada aquí.
