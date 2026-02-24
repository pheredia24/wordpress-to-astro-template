# Handoff – {{PROJECT_NAME}}

> El **agente** rellena este documento al cerrar la Fase 7 (deploy y documentación), sustituyendo todos los placeholders `{{...}}` por los valores reales del proyecto.

## Enlaces útiles

> El agente debe rellenar cada URL con el valor real (repositorio, Studio en producción, Sanity Manage, Vercel, sitio nuevo, sitio origen).

| Qué | URL |
|-----|-----|
| **Repositorio** | {{REPO_URL}} |
| **Sanity Studio** (editar contenido) | {{STUDIO_URL}} |
| **Sanity Manage** (proyecto, tokens, dataset) | {{SANITY_MANAGE_URL}} |
| **Vercel** (deploys, variables, dominio) | {{VERCEL_PROJECT_URL}} |
| **Sitio web nuevo** | {{PRODUCTION_URL}} |
| **Sitio que hemos duplicado** (referencia) | {{ORIGIN_SITE_URL}} |

---

## Sanity Studio

- **Ubicación:** carpeta `studio/`
- **Dataset:** `production` (configurable con `SANITY_DATASET`)

### Cómo ejecutar el Studio

1. Copiar variables de entorno:
   ```bash
   cp .env.example .env
   ```
   Editar `.env` y definir `SANITY_PROJECT_ID` (y opcionalmente `SANITY_DATASET`; por defecto `production`).

2. Si no tienes proyecto Sanity, crearlo en https://sanity.io/manage o con `sanity init --create-project` (ver **guide.md**) y poner `projectId` en `.env`. Para el seed con escritura, añadir `SANITY_API_TOKEN`.

3. Instalar dependencias del studio y arrancar:
   ```bash
   cd studio && npm install && npm run dev
   ```
   El Studio quedará disponible en la URL que indique el CLI (por ejemplo `http://localhost:3333`).

4. En `studio/sanity.config.ts` se usan `process.env.SANITY_PROJECT_ID` y `process.env.SANITY_DATASET`. En modo `sanity dev` las variables se cargan desde un `.env` en la raíz del repo.

## Seed de contenido

El script `scripts/seed-sanity.mjs` crea o actualiza documentos en Sanity a partir de los JSON de contenido (`content/*.json`). Si el documento ya existe, hace **patch** para no sobrescribir ediciones manuales ni imágenes ya enlazadas.

### Cómo ejecutar el seed

Desde la **raíz del repo**:

```bash
# Con .env configurado
npm run seed-sanity

# O con variables en línea
SANITY_PROJECT_ID=tu_project_id SANITY_DATASET=production SANITY_API_TOKEN=tu_token node scripts/seed-sanity.mjs
```

- **Con token:** crea/actualiza documentos y puede subir imágenes desde `assets/origin` o `assets/optimized` según la lista de medios del proyecto.
- Para omitir subida de imágenes: `node scripts/seed-sanity.mjs --no-images`

### Variables de entorno (seed / Studio)

| Variable | Obligatoria | Uso |
|----------|-------------|-----|
| `SANITY_PROJECT_ID` | Sí | Proyecto en sanity.io |
| `SANITY_DATASET` | No (por defecto `production`) | Dataset a usar |
| `SANITY_API_TOKEN` | Para escribir y subir assets | Token con permisos de escritura (Sanity → API → Tokens) |

## Estructura del desk (no técnicos)

En el Studio, el escritorio se agrupa según la estructura del sitio (Fase 3). Ejemplos: Páginas del sitio, Proyectos/Blog/Servicios, Legal, Configuración. Actualizar esta sección con los grupos reales del proyecto.

## Build Astro (Fase 5)

- **Comandos:** desde la raíz: `npm install`, `npm run dev` (desarrollo), `npm run build` (producción). Salida estática en `dist/`.
- **Contenido:** el sitio puede leer de `content/*.json` o de Sanity (GROQ). Documentar aquí la fuente usada.
- **Formulario de contacto:** el `action` se toma de **`PUBLIC_CONTACT_FORM_ACTION`**. Configurar en Vercel o `.env` la URL de Formspree, Resend o similar antes de producción.
- **Favicon / logo:** documentar en qué carpeta están y cómo generarlos si hay script (ej. `node scripts/generate-favicons.mjs`).

## Deploy y producción (Fase 7)

- **Repositorio GitHub:** {{REPO_URL}} (conectado a Vercel; el deploy se dispara por push a la rama conectada).
- **URL de producción:** {{PRODUCTION_URL}}
- **Proyecto Vercel:** {{VERCEL_PROJECT}} (para `vercel --scope` si aplica).
- **Cómo desplegar:** push a la rama `main` (o la conectada) para deploy automático; o desde la raíz: `vercel --prod` (y `--scope` si hace falta).
- **Dominio propio:** en Vercel → Project Settings → Domains añadir el dominio y configurar DNS (CNAME o registros que indique Vercel).
- **Variables de entorno en Vercel:** `PUBLIC_CONTACT_FORM_ACTION`; si el front usa Sanity, `PUBLIC_SANITY_PROJECT_ID` y `PUBLIC_SANITY_DATASET`.

### Deploy hook Sanity → Vercel (rebuild al publicar)

1. **Crear el deploy hook en Vercel** (solo desde el dashboard): proyecto → **Settings** → **Git** → **Deploy Hooks** → **Create Hook** (nombre ej. "Sanity", rama `main`), copiar la URL.
2. **Configurar en el Studio:** añadir en `.env` (raíz): `SANITY_STUDIO_VERCEL_DEPLOY_HOOK_URL=<url_copiada>`. Si el Studio usa `sanity-plugin-vercel-deploy`, la herramienta "Deploy" permitirá disparar un rebuild desde el Studio.
3. Documentar aquí cuando esté configurado.

## Cómo editar contenido y desplegar

- **Editar en Sanity:** Abrir el Studio en producción: **{{STUDIO_URL}}** (o `cd studio && npm run dev` en local). Editar documentos y publicar. Si el front usa Sanity, un rebuild (deploy hook o push) actualizará el sitio.
- **Editar sin Sanity:** Editar los JSON en `content/`; luego `npm run build` y desplegar (push o `vercel --prod`).
- **Desplegar tras cambios:** Push a la rama conectada o ejecutar `vercel --prod` desde la raíz.

## Redirects

Los redirects (URL antigua → nueva) se generan desde `phase1/urls.json`. Para regenerar: `node scripts/generate-redirects.mjs` (el resultado se escribe en `vercel.json`). Documentar si el script tiene otro nombre en este proyecto.

## Resumen variables de entorno

| Dónde | Variable | Uso |
|-------|----------|-----|
| Raíz / Studio | SANITY_PROJECT_ID, SANITY_DATASET | Sanity Studio y seed |
| Raíz (opcional) | SANITY_API_TOKEN | Seed con escritura y subida de imágenes |
| Vercel / .env (build) | PUBLIC_CONTACT_FORM_ACTION | URL del endpoint del formulario de contacto |
| Vercel (si Sanity en front) | PUBLIC_SANITY_PROJECT_ID, PUBLIC_SANITY_DATASET | Astro leyendo de Sanity |
| Raíz / Studio (opcional) | SANITY_STUDIO_VERCEL_DEPLOY_HOOK_URL | Deploy Hook de Vercel para rebuild desde Sanity al publicar |
