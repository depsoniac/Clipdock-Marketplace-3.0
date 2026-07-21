# Clipdock Marketplace 3.0 — Guía de Desarrollo

**Repositorio:** `github.com/depsoniac/Clipdock-Marketplace-3.0`  
**Rama principal:** `main`

---

## Requisitos

- Node.js LTS (solo para correr el script localmente)
- Git configurado con acceso a push en el repo
- Opcional: variable de entorno `GITHUB_TOKEN` para evitar el límite de 60 req/h de la API de GitHub

---

## Instalar dependencias del script

```bash
cd "C:\Users\DepsoniacPC\Documents\GitHub\Clipdock-Marketplace-3.0"
npm install
```

> El `package.json` no existe en este repo — el script `build-catalog.mjs` usa solo módulos de Node.js core (`fs`, `path`, `crypto`) y `fetch` nativo (Node 18+). No requiere `npm install`.

---

## Regenerar el catálogo localmente

```bat
cd "C:\Users\DepsoniacPC\Documents\GitHub\Clipdock-Marketplace-3.0"
node scripts\build-catalog.mjs
```

Con token de GitHub (recomendado para no toparse con el límite de API):

```bat
set GITHUB_TOKEN=ghp_xxxxx
node scripts\build-catalog.mjs
```

El script genera o actualiza:
- `catalog-resolved.json` — catálogo completo con versiones y URLs de descarga
- `plugins/index.json` — lista de carpetas de plugins activos

---

## Agregar un plugin nuevo

1. Crear la carpeta `plugins/<id-del-plugin>/`
2. Copiar `plugins/_template/plugin.json` y editarlo
3. Agregar assets en `plugins/<id-del-plugin>/assets/` (logo, banner, screenshots)
4. Verificar que `plugin.json` tenga `id`, `slug`, `name`, `version`, `type`, `installMode`
5. Publicar con Store Manager o con git directamente

```bat
cd "C:\Users\DepsoniacPC\Documents\GitHub\Clipdock-Marketplace-3.0"
git add plugins\<id-del-plugin>
git commit -m "feat: agregar plugin <id-del-plugin>"
git push origin main
```

GitHub Actions regenera el catálogo automáticamente en ~1 minuto.

---

## Editar un plugin existente

Editar `plugins/<folder>/plugin.json` directamente o con Store Manager.

**Campos que NUNCA se deben cambiar una vez publicados:**
- `id` — es el identificador con el que ClipDock registra la instalación
- `slug` — determina el nombre de la carpeta de instalación
- `installDirName` — si está definido, igual de estable que `slug`

---

## Publicar notificaciones

Editar `notifications.json`. Ver la guía interna en el campo `_systemGuide` dentro del propio archivo.

Reglas clave:
- Solo los elementos en el array `notifications` son avisos activos.
- Cada aviso necesita un `id` estable y un `revision` único al editar.
- No poner ejemplos ni avisos viejos en `notifications` o volverán a mostrarse.

---

## Publicar cambios manualmente (sin Store Manager)

```bat
cd "C:\Users\DepsoniacPC\Documents\GitHub\Clipdock-Marketplace-3.0"
git add -A
git commit -m "Actualizar tienda"
git push origin main
```

---

## Sincronizar cambios del robot (GitHub Actions)

El robot hace commits automáticos de `catalog-resolved.json` y `plugins/index.json`.
Antes de editar, hacer pull:

```bat
git pull origin main
```

---

## Verificar que el catálogo es válido

```bat
node -e "JSON.parse(require('fs').readFileSync('catalog-resolved.json','utf8')); console.log('OK')"
node -e "JSON.parse(require('fs').readFileSync('catalog.json','utf8')); console.log('OK')"
node -e "JSON.parse(require('fs').readFileSync('notifications.json','utf8')); console.log('OK')"
```

---

## Configurar un repo de plugin para notificar al Marketplace al publicar un release

Copiar `docs/plugin-repo-notify.yml` al repo del plugin como `.github/workflows/notify-marketplace.yml`.  
Esto lanza un `repository_dispatch` al Marketplace cuando se publica un release, lo que dispara la regeneración del catálogo inmediatamente.
