# Guía definitiva: WordPress → Astro (proceso repetible)

Proceso para migrar sitios WordPress a una web custom con Astro + Sanity + Vercel. Pensado para que **agentes de IA trabajen de forma autónoma** de punta a punta: no se salta ningún paso; cada fase tiene una salida clara que alimenta la siguiente.

**Alcance:** Las webs pueden ser de cualquier tipo (portfolio, corporativo, blog, servicios, etc.). La estructura de URLs y los tipos de contenido (listados, detalle, legal, etc.) se definen en la Fase 1 según cada sitio; no se asume ninguna ruta concreta (p. ej. no es obligatorio tener `/proyectos`).

**Prioridad:** Lanzar a producción cuanto antes. No hay fase de QA formal; se prioriza el deploy.

**Replicar primero, mejorar después:** La prioridad es una **réplica fiel** del sitio origen. No se debe cambiar estructura visual, layout por página, colores, tipografía ni comportamiento del header/footer hasta tener esa réplica cerrada. Las mejoras técnicas (rendimiento, accesibilidad, SEO) se aplican **sin alterar** la apariencia ni la estructura; cualquier mejora de UX o navegación se deja para después del entregable.

**Replicar** = misma estructura de URLs y navegación; mismo contenido; **misma estructura de cada página** (secciones, orden, tipo de listado/grid, componentes); **mismos colores y tipografías** (extraídos del origen); **mismo header y footer** (todos los enlaces, incluidos redes sociales; comportamiento como sticky/fixed si el origen lo tiene). **Mejorar** = solo mejoras técnicas sin rediseño: rendimiento (imágenes optimizadas, lazy load, formatos modernos), HTML semántico, accesibilidad básica, SEO técnico, código mantenible. No inventar paletas, fuentes ni layouts; no rediseñar listados ni páginas de detalle.

**Para el agente:** Cada fase incluye criterios de cierre y entregables obligatorios. La prioridad es **replicar fielmente** el sitio origen (colores, tipografía, estructura de cada página, header y footer con todos los enlaces y comportamiento); no se considera una fase completada si falta alguno de los artefactos o criterios listados en la guía (auditoría con body/imágenes/forms **y layout por tipo y header/footer**, lista de medios completa, **screenshots de referencia del origen (1.6) y verificación visual (6.0)**, **colores y tipografía extraídos del origen**, imágenes optimizadas, ortografía documentada, Sanity o excepción documentada, **Studio desplegado en producción con URL en handoff**, favicon set completo, **header/footer replicando enlaces y comportamiento**, **páginas replicando estructura del origen**, formularios con consentimiento y destino real, SEO con og/twitter/JSON-LD, deploy staging y producción, handoff con edición y variables, deploy hook Sanity → Vercel documentado). Si un paso no aplica al sitio concreto (ej. no hay galerías), documentar la excepción en el repo en lugar de omitir sin constancia.

---

## Preparación (antes de Fase 1)

El humano hace los logins en los CLIs **al iniciar el proyecto** (en la misma máquina donde correrá el agente). Así, si el agente comprueba que todo está bien, el humano puede irse y el proceso continúa solo.

| Qué | Cómo (humano) |
|-----|----------------|
| **Vercel CLI** | `vercel login` (abre navegador). |
| **Sanity CLI** | `sanity login` (abre navegador). |
| **Git / GitHub** | `gh auth login` (o credenciales git ya configuradas), si el repo se sube a GitHub. |
| **Formulario de contacto (opcional)** | Si se quiere destino real desde el primer deploy: configurar Formspree, Resend u otro, obtener la URL del endpoint del formulario, y dejarla en `.env` como `CONTACT_FORM_ACTION=<url>`. Si no se deja, el agente documentará en handoff que debe configurarse. |

**Para el agente — obligatorio antes de Fase 1:** (1) **Comprobar logins:** ejecutar `vercel whoami`; `sanity debug` (o comando que muestre usuario en Sanity); `gh auth status` (o comprobar que `git push` funcionaría). Si algún comando falla o pide login, informar al humano y **no continuar** hasta que esté resuelto. (2) **Repo GitHub y conexión Vercel:** Si no hay repositorio git inicializado: `git init`, añadir `.gitignore` (incluir `.env`, `node_modules`, `dist`, `studio/dist`), hacer commit inicial. Si no hay remoto de GitHub: crear el repo con `gh repo create <nombre-repo> --private --source=. --remote=origin --push` (o `--public` si aplica); si el repo ya existe en GitHub, añadir `git remote add origin <url>` y hacer push. Conectar el proyecto al repo en Vercel para que los pushes disparen deploy: `vercel git connect <url-del-repo>` (ej. `https://github.com/<org>/<repo>`); si pide confirmación, aceptar. Si el directorio no está vinculado a un proyecto Vercel (no existe `.vercel/project.json`), ejecutar antes `vercel link` y elegir o crear el proyecto. Documentar en el handoff la URL del repositorio y que el deploy se dispara por push a la rama conectada. (3) **Sanity:** Si en el repo no existen `SANITY_PROJECT_ID` y `SANITY_API_TOKEN` en `.env` (o equivalente), el agente **debe crear** el proyecto y el token en Fase 3 usando el CLI (tras `sanity login`): `sanity init -y --create-project "<nombre>" --dataset production --bare` para obtener project ID, luego `sanity tokens add "Seed Token" --role=editor --yes --json` en el contexto del proyecto, y escribir las variables en `.env`. No pedir al humano que cree el proyecto manualmente salvo que el CLI falle. Si **todos los logins responden OK**, continuar con la Fase 1 y todas las fases posteriormente, sin parar.

---

## Resumen por fases

| Fase | Objetivo | Salida |
|------|----------|--------|
| **0. Preparación** | Entorno listo para deploy y CMS | Humano: logins hechos. Agente: comprobar con vercel whoami, sanity debug, gh auth status; si OK, seguir; si no, avisar y no continuar. |
| **1. Descubrimiento** | Inventario completo de la web actual | Sitemap, estructura, auditoría (contenido + layout por tipo + header/footer completos), lista de medios, **screenshots de referencia** por tipo de página y viewport |
| **2. Marca y contenido** | Branding y contenido listos para CMS | Colores y tipografía **extraídos del origen**, logos, imágenes optimizadas, contenido en JSON/CMS |
| **3. Plataforma y CMS** | Stack elegido y contenido en Sanity | Sanity con schemas, contenido seedeado, paneles definidos |
| **4. Diseño de interfaz** | Header, footer, layouts **del origen** | Guía de estilos con valores del origen; header (comportamiento + enlaces) y footer (todos los enlaces); layouts por tipo replicando estructura |
| **5. Build** | Páginas, formularios, redirects, SEO | Sitio funcional en staging |
| **6. Responsive y pulido** | Verificación visual vs referencia, móvil, tacto, rendimiento, accesibilidad | Sitio fiel al origen y listo para producción |
| **7. Deploy y entrega** | Producción y handoff | Deploy prod cuanto antes, documentación |

---

## Dependencias entre fases (alto nivel)

```
0. Preparación (humano: login Vercel, Sanity, GitHub) ──────────┐
    ↓                                                           │
1. Descubrimiento ──────────────────────────────────────────────┐
    ↓                                                           │
2. Marca y contenido ←── (si estructura no clara, refinar 1.3)  │
    ↓                                                           │
3. Plataforma y CMS ←── (si contenido no encaja, ajustar 3.3)   │
    ↓                                                           │
4. Diseño de interfaz                                            │
    ↓                                                           │
5. Build                                                         │
    ↓                                                           │
6. Responsive y pulido                                           │
    ↓                                                           │
7. Deploy y entrega (prioridad: lanzar a prod cuanto antes)
```

---

## Fase 1: Descubrimiento e inventario

**Objetivo:** Conocer al 100% la web actual: URLs, tipos de página, contenido y medios.

| # | Tarea | Depende de | Salida / Notas |
|---|--------|------------|----------------|
| 1.1 | **Extraer todas las URLs** (sitemap.xml, crawl o ambos) | — | Lista de URLs (ej. `urls.txt` o JSON). Input: URL de la web origen. |
| 1.2 | **Clasificar URLs** por tipo: home, listados, detalle (por tipo de contenido: proyectos, servicios, entradas, etc.), legal, blog si aplica | 1.1 | Mapa URL → tipo de página |
| 1.3 | **Documentar estructura de la web**: navegación, jerarquía, qué páginas comparten plantilla (depende del sitio: puede haber /proyectos, /servicios, /productos, etc.) | 1.2 | Doc o esquema: “nav, árbol, templates” |
| 1.4 | **Auditoría de contenido por URL**: textos, bloques, imágenes in-page, formularios, embeds; **por tipo de página, estructura de layout** (secciones en orden); **global: header** (comportamiento al scroll: sticky/fixed/none, todos los enlaces); **footer** (todos los enlaces: legal, contacto, redes sociales — uno por uno, ej. LinkedIn, Twitter, etc.) | 1.2 | Inventario (spreadsheet o JSON) por URL + doc de layout por tipo y header/footer |
| 1.5 | **Extraer referencias a todos los medios**: imágenes, PDFs, etc. (URL origen + uso previsto), a partir de la auditoría (1.4) o del crawl | 1.4 (o 1.1 si se hace en el mismo crawl) | Lista de assets con contexto |
| 1.6 | **Capturar screenshots de referencia del sitio origen**: al menos una captura por **tipo de página** (home, un listado, un detalle, una legal) y por **viewport** (desktop ej. 1280px, móvil ej. 375px). Guardar en `phase1/reference-screenshots/` con nombres claros (ej. `home-desktop.png`, `listado-mobile.png`). Son la referencia visual para comprobar fidelidad más adelante | 1.2, 1.4 | Carpeta con screenshots origen (por tipo y viewport) |

**Regla para el agente:** No pasar a Fase 2 hasta tener: (1) todas las URLs con tipo asignado, (2) cada URL con al menos título y tipo de contenido, (3) lista de medios sin huecos obvios, (4) **screenshots de referencia** (1.6) por tipo de página y viewport, (5) **estructura de layout por tipo de página** y **especificación global de header (comportamiento + enlaces) y footer (todos los enlaces)**. Si la estructura no está clara, inspeccionar más URLs por patrón antes de seguir.

**Formato obligatorio de la auditoría (1.4):** El inventario por URL debe incluir, para cada URL relevante: **title**; **tipo de página**; **body** (texto principal o resumen de bloques); **imágenes in-page** (lista de URLs o referencias con contexto: hero, galería, card, etc.); **formularios** (campos, action si existe); **embeds** si los hay (vídeo, mapa, etc.). Además, para **cada tipo de página** (home, listado de proyectos, listado por categoría, detalle de proyecto, legal, etc.): **estructura de layout** — secciones o bloques en orden (ej. hero, título, grid de cards, sidebar, CTA), tipo de presentación (grid, lista, filtros, etc.). A nivel **global**: **header** — comportamiento al hacer scroll (sticky, fixed, estático); lista completa de enlaces del menú. **Footer** — lista completa de enlaces: legales, contacto, y **cada red social** con su URL (LinkedIn, Twitter, Instagram, etc.); no omitir ninguno. No basta con título y tipo: sin body e imágenes por URL la Fase 2 y el build quedarán incompletos. **Regla estricta:** Para cada URL, el agente debe obtener el HTML (fetch/curl o crawl que devuelva HTML) y parsear con herramienta adecuada (cheerio, jsdom, regex sobre el HTML, etc.) para extraer todos los `img` (atributo `src`, resuelto a URL absoluta) y cada `form` (atributo `action`, campos por `name`/`id`). Rellenar en el artefacto de auditoría los arrays `images` (con `src` y `context` según el contenedor: hero, galería, card, logo) y `forms`. No dar por válida una auditoría en la que `images` esté vacío para URLs que, según el sitemap o la inspección, contienen imágenes.

**Formato obligatorio de la lista de medios (1.5):** Una lista (JSON, CSV o texto) donde cada medio tenga: **URL origen** (absoluta), **uso previsto** (ej. “logo header”, “galería proyecto X”, “hero home”), y opcionalmente **página/URL donde aparece**. Construir esta lista **a partir de la auditoría:** por cada entrada en `audit[*].images`, añadir una fila en la lista de medios con URL origen, uso previsto (contexto) y página. Incluir también cualquier imagen que aparezca en el sitemap por URL si no está ya en la auditoría. Si el sitemap indica N imágenes para una URL, la lista de medios debe reflejarlas (extraer del HTML si hace falta). No cerrar la Fase 1 con huecos obvios: todas las URLs con imágenes en el origen deben estar representadas en la lista de medios.

**Salida de la fase:** `urls.json`, documento/esquema de estructura (nav, árbol, templates), auditoría por URL (title, body, images, forms), **estructura de layout por tipo de página** y **especificación de header (comportamiento al scroll + enlaces) y footer (todos los enlaces, incluidas redes sociales)**, lista de medios con URLs, **screenshots de referencia** del origen por tipo de página y viewport en `phase1/reference-screenshots/`.

---

## Fase 2: Marca y contenido

**Objetivo:** Tener branding definido (aunque sea mínimo) y contenido listo para migrar. Parte de esto puede ir en paralelo con Fase 1 una vez hay estructura.

| # | Tarea | Depende de | Salida / Notas |
|---|--------|------------|----------------|
| 2.1 | **Extraer branding desde la web origen**: logo(s), **colores** (valores exactos: hex, RGB o equivalentes del CSS/computed styles del sitio), **tipografías** (familias y pesos usados en títulos, cuerpo, etc.). Inspeccionar CSS, estilos computados o fuentes del origen; **no sustituir por paletas o fuentes genéricas**; documentar los valores extraídos. Si algún valor no es extraíble, documentar la excepción y definir un mínimo solo para ese caso | 1.3, 1.4 | Notas de marca o guía de estilos borrador con colores y fuentes del origen |
| 2.2 | **Descargar y guardar logos** (identificar en header/footer o 1.5, guardar en SVG/PNG) | 2.1, 1.5 | Assets en repo (ej. `public/`) |
| 2.3 | **Extraer imágenes**: descargar, redimensionar si hace falta | 1.5 | Carpeta de imágenes (origen para optimización) |
| 2.4 | **Optimizar imágenes con TinyJPG** (o similar): comprimir para reducir peso; luego formato moderno (WebP/AVIF) donde aplique | 2.3 | Imágenes optimizadas para CMS o CDN |
| 2.5 | **Extraer textos y estructura** a datos (scraping o export): por página y por tipo de contenido (proyectos, entradas, servicios, etc.) | 1.4, 1.3 | JSON/CSV por tipo de contenido |
| 2.6 | **Revisión ortográfica y de estilo** en los textos (antes o después de seed; si es antes, menos conflicto) | 2.5 | Contenido corregido en fuentes de verdad |

**Orden para el agente:** 2.1 y 2.2 en cuanto tengas 1.3 y 1.5. 2.3 en cuanto tengas 1.5. 2.4 después de 2.3. 2.5 cuando tengas 1.4 y 1.3 (transformar auditoría en JSON/CSV por tipo de contenido). 2.6 al final sobre los textos de 2.5.

**Regla:** No pasar a Fase 3 sin: branding **extraído del origen** (colores y tipografías documentados con valores del sitio, no inventados), logos en repo, imágenes optimizadas, contenido en JSON/CSV por tipo, ortografía revisada.

**Imágenes (2.3 y 2.4) — obligatorio:** Descargar **todas** las imágenes referenciadas en la lista de medios (1.5), no solo el logo. Guardar en una carpeta de origen (ej. `content/images/` o `assets/origin/`). Después, optimizar **todas** con sharp/squoosh (o equivalente): comprimir y generar versiones en **formatos modernos**: **WebP y AVIF** (ambos cuando la herramienta lo permita; como mínimo WebP). Si el sitio no tiene galerías ni imágenes in-page, solo logo, basta con el logo optimizado; si tiene galerías o imágenes por página, todas deben estar descargadas y optimizadas. La lista de medios (1.5) es la fuente de verdad: toda URL en ella debe tener su archivo descargado y sus versiones optimizadas generadas.

**Ortografía (2.6) — obligatorio:** Revisar ortografía y estilo en los textos extraídos (content/*.json o CSV). Dejar constancia: un comentario en el repo, una línea en un doc (ej. “Ortografía revisada en content/*.json, fecha”) o un checklist en HANDOFF. No pasar a Fase 3 sin esta revisión documentada.

**Salida:** Branding + logos, imágenes optimizadas (TinyJPG o sharp/squoosh + formatos modernos), `content/*.json` (o CSV) por tipo de contenido, textos corregidos.

---

## Fase 3: Plataforma y CMS

**Objetivo:** Stack fijado, Sanity con schemas y contenido migrado, paneles útiles para técnicos y no técnicos.

| # | Tarea | Depende de | Salida / Notas |
|---|--------|------------|----------------|
| 3.1 | **Fijar stack**: framework (Astro), CMS (Sanity), hosting (Vercel) | — | Decisión documentada (fija para este proceso): crear al inicio de Fase 3 un documento, p. ej. `docs/stack.md` o sección en README, que indique explícitamente "Stack: Astro, Sanity, Vercel". |
| 3.2 | **Crear proyecto Sanity** (proyecto + dataset), instalar Sanity Studio, configurar env (projectId, dataset) para uso desde Astro | 3.1 | Repo con `/studio-*` y variables de entorno listas. Si no existen `SANITY_PROJECT_ID` y `SANITY_API_TOKEN` en `.env`, el agente debe crearlos con el CLI: `sanity init -y --create-project "<nombre>" --dataset production --bare` (capturar project ID de la salida), luego en el contexto del proyecto `sanity tokens add "Seed Token" --role=editor --yes --json` y escribir projectId, dataset y token en `.env`. |
| 3.3 | **Diseñar e implementar schemas** según estructura (singletons, páginas, tipos de contenido que tenga la web, config) | 1.3, 2.5 | Schemas en código |
| 3.4 | **Panel de administración técnico**: Structure Tool por defecto, grupos y campos pensados para dev | 3.3 | Studio usable para desarrollo |
| 3.5 | **Scripts de seed**: crear/actualizar documentos desde JSON (patch si existe para no pisar imágenes/ediciones); ejecutar seed usando `SANITY_API_TOKEN` si el script lo requiere | 2.5, 3.3 | Scripts `seed-*`, documentación de uso; seed ejecutado sin errores; comprobar en Studio que los documentos tienen referencias a imágenes/assets enlazadas |
| 3.6 | **Subir imágenes al CMS** (o CDN): enlazar assets a documentos (hero, galerías, cards, etc.) | 2.4, 3.5 | Documentos con referencias a assets |
| 3.7 | **Revisión ortográfica en Sanity** (si no se hizo en 2.6) | 3.5, 3.6 | Contenido final en CMS |
| 3.8 | **Panel de administración no técnico**: Custom Desk según la web (ej. “Páginas del sitio”, tipos de contenido como “Proyectos”/“Blog”/“Servicios”, “Configuración”) | 3.4, 3.3 | Studio amigable para cliente |
| 3.9 | **Desplegar Sanity Studio en producción** | 3.5, 3.6, 3.8 | El agente debe desplegar el Studio sin intervención humana. En `sanity.cli.ts` (o config equivalente) configurar **studioHost** con un nombre único para evitar colisiones: p. ej. slug identificativo del sitio + id aleatorio corto (4–6 caracteres alfanuméricos, ej. `miempresa-a3f9x`). Ejecutar desde la carpeta del Studio `sanity deploy -y`; la URL resultante será `https://<studioHost>.sanity.studio/`. Documentar esa URL en el handoff (7.3). No dar por cerrada la Fase 3 sin Studio desplegado y URL anotada. |

**Regla para el agente:** Si al seedar falla o el contenido no encaja en los schemas, ajustar schemas (3.3) y re-ejecutar seed (3.5) e imágenes (3.6). No pasar a Fase 4 sin: schemas implementados, seed sin errores, documentos con imágenes enlazadas, Custom desk configurado, **y Studio desplegado en producción (3.9) con URL documentada**.

**No omitir la Fase 3:** Esta guía asume stack Astro + **Sanity** + Vercel. El contenido debe vivir en Sanity (schemas, seed, Studio, Custom desk). Si en un caso excepcional se decide no usar Sanity (p. ej. sitio solo estático), crear **`docs/EXCEPCION_SANITY.md`** explicando el motivo y cómo se sirve el contenido (solo JSON local, Markdown, etc.); no dar por cerrada la Fase 3 sin los entregables de Sanity o sin ese documento de excepción.

**Salida:** Sanity con contenido completo (texto + referencias a assets), paneles técnico y no técnico listos, **y Studio desplegado en `https://<studioHost>.sanity.studio/` con URL documentada en handoff**.

---

## Fase 4: Diseño de interfaz

**Objetivo:** Guía de estilos y decisiones de layout **extraídas del sitio origen** (no inventadas), para que el build replique fielmente la apariencia y la estructura.

| # | Tarea | Depende de | Salida / Notas |
|---|--------|------------|----------------|
| 4.1 | **Guía de estilos definitiva**: **usar colores y tipografías extraídos en 2.1** (valores del origen); espaciado y componentes (botones, cards) según el origen; breakpoints. No definir paletas ni fuentes nuevas; la guía debe reflejar el sitio origen | 2.1, 1.3 | Doc/HTML o design tokens en CSS con valores del origen |
| 4.2 | **Header**: estructura (logo, nav, CTA), **comportamiento al scroll** (sticky, fixed o estático, según 1.4), comportamiento desktop y móvil (menú hamburguesa, etc.); **todos los enlaces** del menú. Replicar lo documentado en la auditoría (1.4) | 4.1, 2.2, 1.3 | Especificación o maqueta |
| 4.3 | **Footer**: estructura (enlaces, contacto, legal, **cada red social con su URL** — LinkedIn, Twitter, etc., sin omitir ninguna), desktop y móvil. Replicar todos los enlaces capturados en 1.4 | 4.1, 1.3 | Especificación o maqueta |
| 4.4 | **Favicon y app icons** (generar desde logo) | 2.2, 4.1 | Incluir al menos **favicon.ico** (o equivalente reconocible por navegadores, p. ej. PNG 32×32 como favicon) y, si aplica, favicon.svg, apple-touch-icon; documentar en guía de estilos o HANDOFF dónde están y cómo se referencian |
| 4.5 | **Layouts por tipo de página**: home, listados, detalle, legal, y los que apliquen (blog, etc.) | 1.3, 4.1 | Definición de secciones y componentes por tipo |

**Regla para el agente:** Todo debe **derivar del sitio origen**: guía de estilos con colores y tipografías extraídos (2.1), header y footer según la especificación de 1.4 (todos los enlaces, comportamiento del header). No inventar estilos ni omitir enlaces. No pasar a Fase 5 sin: guía de estilos (doc o tokens con valores del origen), header y footer especificados (desktop, móvil, comportamiento al scroll, lista completa de enlaces del footer incluidas redes sociales), favicon/iconos, layouts definidos por tipo de página.

**Guía de estilos (4.1) — contenido mínimo y origen:** Los colores y la tipografía deben ser **los extraídos del sitio origen** en 2.1 (valores hex/familias documentados); no sustituir por paletas o fuentes genéricas. El doc o los tokens deben incluir: tipografía (familias y tamaños del origen); colores (primario, secundario, fondo, texto — valores del origen); espaciado (márgenes/paddings según el origen o escala coherente); **componentes** (botones, cards según el origen); **breakpoints**. Si se usan design tokens en CSS, usar variables que reflejen esos valores (ej. `--color-primary`, `--font-*`).

**Header y footer (4.2 y 4.3):** Debe existir un documento (o sección en la guía de estilos) que describa: estructura del header (logo, navegación, CTA si hay); **comportamiento al hacer scroll** (sticky, fixed o estático — según lo capturado en 1.4); comportamiento en desktop y en móvil (menú hamburguesa, desplegables); **lista completa de enlaces del footer**: legales, contacto, y **cada red social con su URL** (LinkedIn, Twitter, Instagram, etc.). Replicar lo documentado en la auditoría; no omitir enlaces. Comportamiento del footer en móvil. Puede ser texto + esquema o maqueta; no basta con “implementado en código” sin especificación reutilizable.

**Favicon y app icons (4.4) — obligatorio:** Generar desde el logo y entregar en el repo: **favicon.ico** (o equivalente), **favicon.svg** (recomendado), **apple-touch-icon** (tamaño según estándar, ej. 180×180). Incluir en el layout base; no usar solo una imagen PNG genérica como único icono.

**Layouts por tipo (4.5):** Documentar para cada tipo de página del sitio (home, listados, detalle de contenido, legal, blog si aplica) la **estructura y orden del origen**: qué secciones tiene, en qué orden, qué componentes (grid, lista, filtros, sidebar, etc.). Esta definición es el **contrato de replicación**: el build debe reflejar esta estructura; no rediseñar ni simplificar listados o páginas de detalle respecto al origen.

**Salida:** Guía de estilos, especificación de header/footer e iconos, layouts por tipo.

---

## Fase 5: Build

**Objetivo:** Sitio Astro funcional en staging con todas las páginas, formularios, redirects y SEO básico.

| # | Tarea | Depende de | Salida / Notas |
|---|--------|------------|----------------|
| 5.1 | **Proyecto Astro** + integración Sanity cuando existan `SANITY_PROJECT_ID` y `SANITY_API_TOKEN` en `.env` (env, queries GROQ); si no hay Sanity en env, servir contenido desde `content/*.json` o fuentes estáticas y documentar en handoff | 3.1, 3.2 | Repo Astro con integración CMS o con doc que indique "contenido desde JSON estático" |
| 5.2 | **Layout base**: `BaseLayout`, CSS global, meta base | 4.1 | Layout y estilos globales |
| 5.3 | **Header y footer** (desktop y móvil) | 4.2, 4.3, 5.2 | Componentes Header/Footer |
| 5.4 | **Páginas por tipo**: home, listados, detalle (ej. `[slug].astro`), legal, y los que apliquen (blog, etc.) | 3.3, 4.5, 5.2 | Rutas y componentes de página |
| 5.5 | **Componentes de imagen**: responsive, lightbox si aplica | 4.1, 3.6 | Componentes reutilizables |
| 5.6 | **Formularios**: implementación y conexión (email, CRM, BD, etc.) + consentimiento y avisos legales; si no se especifica destino, usar envío por email (ej. Resend, Formspree) o guardar en Sanity | 1.4 | Formularios funcionando |
| 5.7 | **Redirects**: generar **todos** los redirects desde la lista de URLs del sitio origen (1.1) hacia las rutas equivalentes en Astro (5.4); cada URL antigua debe tener entrada en vercel.json (o equivalente: meta refresh si output static); no dar por cerrada la tarea con redirects incompletos o solo parciales | 1.1, 1.2, 5.4 | Lista completa de redirects aplicada (URL origen → URL nueva) |
| 5.8 | **Configuración SEO**: meta tags (title, description), Open Graph, Twitter cards, structured data (JSON-LD si aplica), sitemap generado, robots.txt | 1.2, 3.3 | SEO listo para producción |
| 5.9 | **Despliegue en staging** (Vercel preview o similar) | 5.1–5.8 | URL de staging |

**Rutas equivalentes al origen (5.4):** Cada URL pública del sitio origen (según 1.1 y 1.2) debe tener una ruta equivalente en el sitio Astro. Si el origen tiene listados bajo rutas propias (ej. /categoria-a/, /categoria-b/ además de /proyectos/), crear esas rutas salvo que se documente explícitamente la decisión de unificarlas; no omitir rutas sin dejar constancia.

**Replicar estructura de cada página (5.4):** El layout y los componentes de cada tipo de página (home, listados, detalle, legal, etc.) deben **replicar la estructura y el orden del sitio origen** definidos en 4.5: mismas secciones, mismo orden, mismo tipo de presentación (grid, lista, filtros, sidebar, etc.). No rediseñar listados ni páginas de detalle para "mejorar" la UX; la prioridad es la réplica fiel. Las mejoras de estructura o navegación se dejan para después del entregable.

**Componente de imagen (5.5):** Implementar al menos un componente de imagen reutilizable: responsive (srcset o picture), lazy load por defecto para imágenes below-the-fold. **Si el sitio origen tiene galerías o fotos ampliables** (según auditoría 1.4), incluir **lightbox** (o equivalente) es obligatorio en el componente o en la página de detalle; no dar por cerrada 5.5 sin lightbox cuando la auditoría indique galerías.

**Formularios (5.6) — obligatorio:** (1) **Destino configurado:** el `action` del formulario debe apuntar a un endpoint real. Si existe la variable de entorno `CONTACT_FORM_ACTION` (o equivalente acordado), usarla como action; si no, configurar Formspree, Resend, Sanity o API propia y documentar en el handoff (7.3) qué variable debe rellenar el cliente. No dejar placeholders tipo YOUR_FORM_ID sin sustituir o documentar. (2) **Consentimiento y avisos:** en formularios que recojan datos personales, incluir checkbox de aceptación de la política de privacidad (o equivalente) y enlace al aviso legal/privacidad; si el sitio no tiene página legal, crearla o enlazar a una URL externa indicada.

**SEO (5.8) — obligatorio:** En el layout base o por página: **meta title y description**; **Open Graph** (og:title, og:description, og:image, og:url como mínimo); **Twitter cards** (twitter:card, twitter:title, twitter:description, twitter:image como mínimo); **sitemap** generado (integración tipo @astrojs/sitemap); **robots.txt**. **JSON-LD estructurado:** en el layout base incluir al menos **Organization** y **WebSite**; en páginas de detalle (proyecto, artículo, servicio, etc.) incluir el tipo que aplique (Article, Product, o schema equivalente). No cerrar 5.8 sin JSON-LD en layout y en páginas de contenido cuando el sitio tenga ese tipo de páginas.

**Staging (5.9):** Desplegar en Vercel (o equivalente) y obtener una URL de preview/staging antes de dar por cerrada la Fase 5. No pasar a Fase 6 sin URL de staging verificable.

**Regla para el agente:** No pasar a Fase 6 sin: todas las rutas respondiendo, **layout y estructura de cada tipo de página replicando el origen** (4.5, 5.4), **colores y tipografía del origen** aplicados (4.1), **header y footer** con todos los enlaces y comportamiento (sticky etc.) según 1.4/4.2/4.3, formularios con destino configurado y consentimiento/avisos, redirects aplicados, SEO config completo (meta, og, twitter, sitemap, robots, JSON-LD si aplica), componente de imagen con lazy load (y lightbox si aplica), deploy staging con URL. Si es la primera vez, crear proyecto en Vercel, conectar repo y configurar env (Sanity, etc.) antes de 5.9.

**Checklist de replicación (antes de dar por cerrada Fase 5):** (1) Colores y tipografía coinciden con el origen (extraídos, no genéricos). (2) Estructura de cada tipo de página (secciones, orden, tipo de listado/grid) coincide con el origen. (3) Header: mismos enlaces y comportamiento al scroll (sticky/fixed/etc.). (4) Footer: todos los enlaces del origen, incluida cada red social (LinkedIn, Twitter, etc.). No considerar mejoras de estructura o navegación hasta tener esta réplica cerrada.

**Salida:** Sitio desplegado en staging (URL de preview), listo para responsive y después producción.

---

## Fase 6: Responsive y pulido

**Objetivo:** Verificar fidelidad visual al origen, ajustar móvil, tacto, rendimiento y accesibilidad.

| # | Tarea | Depende de | Salida / Notas |
|---|--------|------------|----------------|
| 6.0 | **Verificación visual frente a referencia**: Comparar el sitio nuevo (staging) con los screenshots de referencia del origen (1.6) **por cada tipo de página** (home, listado, detalle, legal) y por viewport (desktop, móvil). Comprobar: colores y tipografía equivalentes al origen; misma estructura de secciones y orden; header y footer con mismos enlaces y comportamiento. Si hay diferencias evidentes, **corregir antes de seguir** (ajustar estilos, layout o contenido). Opcional: guardar screenshots del sitio nuevo en la misma convención (ej. `phase6/new-screenshots/`) o documentar en handoff que se ha realizado la comparación. No dar por cerrada la Fase 6 sin esta verificación hecha | 5.9, 1.6 | Comparación documentada; discrepancias corregidas |
| 6.1 | **Ajustes móvil por página**: reducir paddings, ocultar o simplificar secciones en móvil | 5.4, 4.5 | CSS/componentes responsive |
| 6.2 | **Grids de imágenes en móvil** (columnas, tamaño tap, galerías) | 5.5, 6.1 | Comportamiento móvil definido |
| 6.3 | **Evitar dependencia de hover en móvil**: no solo hover para información crítica; tap targets adecuados | 5.3, 5.4 | Interacciones táctiles correctas |
| 6.4 | **Rendimiento**: lazy load, formatos de imagen (ya optimizadas con TinyJPG en 2.4), critical CSS | 5.5, 2.4 | Métricas aceptables |
| 6.5 | **Accesibilidad**: contraste, foco, labels, estructura de encabezados | 4.1, 5.4 | Revisión a11y |

**Regla para el agente:** No dar por cerrada la Fase 6 sin (1) **haber realizado la verificación visual (6.0)** comparando con los screenshots de referencia y corregido diferencias, y (2) haber **aplicado en código** cada ítem del checklist siguiente (no basta con documentar; los cambios deben verse en el sitio). Revisar al menos un breakpoint móvil; asegurar que no haya información crítica solo en hover; tap targets y grids de imágenes correctos. Luego pasar a Fase 7.

**Checklist Fase 6 (no omitir):** (0) **6.0** Verificación visual: comparar cada tipo de página del sitio nuevo con los screenshots de referencia (phase1/reference-screenshots/); colores, tipografía, estructura y header/footer deben coincidir con el origen; corregir discrepancias. (1) **6.1** Ajustes móvil por página: en al menos un breakpoint móvil, revisar paddings y si alguna sección debe ocultarse o simplificarse; aplicar en CSS o componentes. (2) **6.2** Si hay galerías o grids de imágenes: definir columnas, tamaño de tap (mín. ~44px) y comportamiento en móvil. (3) **6.3** Comprobar que la información crítica no dependa solo de hover (tooltips, textos en hover); tap targets suficientes. (4) **6.4** Imágenes: usar lazy load (loading="lazy" o equivalente) en las que no sean above-the-fold; servir formatos modernos (WebP/AVIF) según 2.4; considerar critical CSS para above-the-fold si el rendimiento lo requiere. (5) **6.5** Revisión a11y mínima: contraste de texto/fondo, foco visible en interactivos, labels en formularios, estructura de encabezados (un solo h1 por página, jerarquía h1→h2→h3 coherente).

**Salida:** Sitio listo para producción desde el punto de vista UX y técnico.

---

## Fase 7: Deploy y entrega

**Objetivo:** Lanzar a producción cuanto antes y dejar documentado el handoff. No hay QA formal; se prioriza el deploy.

| # | Tarea | Depende de | Salida / Notas |
|---|--------|------------|----------------|
| 7.1 | **Deploy a producción** (Vercel) | 5.9, 6.x | URL de producción |
| 7.2 | **DNS y dominio** (si cambia o se apunta a nueva plataforma) | 7.1 | Dominio apuntando al nuevo sitio |
| 7.3 | **Documentación y handoff**: cómo editar en Sanity, cómo desplegar, variables de entorno necesarias | 3.8 | Doc en repo (ej. README o docs/HANDOFF.md) |
| 7.4 | **Deploy hook Sanity → Vercel** (rebuild al publicar en Sanity) | 7.1 | Hook creado y configurado; documentado en handoff |

**Regla para el agente:** 7.1 requiere build de producción exitoso (tras 6.x). Si el dominio es nuevo o cambia, 7.2 (DNS) puede ser manual; documentar en 7.3 qué debe configurarse.

**Deploy y documentación (7.1–7.3):** (1) Ejecutar deploy a producción (Vercel u otro) y verificar que la URL de producción responde. (2) Si el dominio o DNS cambian, documentar en el handoff los pasos necesarios (registro, CNAME, etc.). (3) El agente debe **rellenar el HANDOFF** (p. ej. `docs/HANDOFF.md`), incluyendo la tabla **Enlaces útiles** con las URLs reales: repositorio (remote de git), Sanity Studio (URL del deploy 3.9), Sanity Manage (proyecto en sanity.io), Vercel (URL del proyecto en el dashboard), sitio web nuevo (producción), sitio origen que se ha duplicado. (4) El handoff debe incluir de forma explícita: **cómo editar contenido** (Sanity: **URL del Studio en producción** — `https://<studioHost>.sanity.studio/` documentada en 3.9 —, acceso, tipos de documento; o si no hay Sanity: rutas de los archivos JSON/CSV/Markdown y **cómo desplegar tras editar** — p. ej. "tras editar `content/paginas.json`, ejecutar build y deploy"); **cómo desplegar** (comando de build, plataforma, rama o trigger); **variables de entorno** necesarias (Sanity projectId/dataset/token si aplica; **acción del formulario de contacto** — p. ej. `CONTACT_FORM_ACTION` o ID de Formspree/Resend — si aplica; dominio). No dar la Fase 7 por cerrada sin URL de producción y documento de handoff que cubra edición de contenido, despliegue y variables (incluidas las del formulario cuando exista).

**Deploy hook Sanity → Vercel (7.4):** Para que cada publicación en Sanity dispare un rebuild del sitio en Vercel: (1) **Crear el deploy hook en Vercel** — no existe API/CLI pública; hay que hacerlo en el **dashboard**: proyecto Vercel → **Settings** → **Git** → **Deploy Hooks** → **Create Hook** (nombre ej. "Sanity", rama a desplegar ej. `main`), copiar la URL generada. (2) **Configurar en el Studio:** añadir en `.env` (raíz): `SANITY_STUDIO_VERCEL_DEPLOY_HOOK_URL=<url_copiada>`. Si el Studio incluye el plugin `sanity-plugin-vercel-deploy`, la herramienta "Deploy" usará esa URL para disparar rebuilds; opcionalmente el usuario puede añadir un token de Vercel como Studio Secret para ver el estado del deployment. (3) Documentar en el handoff que el deploy hook está configurado y que, al publicar en Sanity, puede dispararse un rebuild desde la herramienta Deploy del Studio (o mediante un webhook/script que haga POST a esa URL). No dar por cerrada 7.4 sin haber documentado en handoff los pasos para crear/configurar el hook y la variable `SANITY_STUDIO_VERCEL_DEPLOY_HOOK_URL`.

---

## Orden lineal sugerido (checklist)

Para usar como checklist en un nuevo sitio, este es un orden posible que respeta dependencias. **Al final de cada fase (tras sus tareas), hacer `git commit` con mensaje descriptivo de la fase. Al final del proceso completo, hacer `git push`.**

0. **Preparación:** Humano hace logins (Vercel, Sanity, GitHub) al iniciar. Agente, antes de Fase 1: ejecutar `vercel whoami`, `sanity debug`, `gh auth status`; si algo falla, avisar y no continuar; si todo OK, proseguir (el humano puede irse).
1. **URLs:** extraer con sitemap/crawl; input = URL de la web origen (1.1)
2. **Estructura:** clasificar URLs, documentar nav y templates (1.2–1.3)
3. **Auditoría:** contenido y medios por URL (1.4–1.5); **screenshots de referencia** del origen por tipo de página y viewport (1.6)
4. **Branding:** definir marca, descargar logos (2.1–2.2)
5. **Imágenes:** extraer y descargar (2.3)
6. **Optimizar imágenes con TinyJPG** (2.4)
7. **Contenido:** extraer textos a JSON/CSV (2.5)
8. **Ortografía:** revisar textos en fuentes (2.6)
9. **Stack:** Astro, Sanity, Vercel (3.1)
10. **Sanity:** proyecto, Studio, schemas (3.2–3.4)
11. **Seed:** contenido e imágenes en Sanity (3.5–3.6)
12. **Ortografía en CMS** si no en 2.6 (3.7)
13. **Custom desk** para no técnicos (3.8)
14. **Deploy Studio en producción:** configurar `studioHost` único (slug + id aleatorio) en sanity.cli.ts, ejecutar `sanity deploy -y` desde la carpeta del Studio, documentar URL en handoff (3.9)
14. **Guía de estilos** definitiva (4.1)
15. **Header y footer** (especificación/maqueta) (4.2–4.3)
16. **Favicon y app icons** (4.4)
17. **Layouts por tipo de página** (4.5)
18. **Build Astro:** layout, header, footer, páginas, imágenes (5.1–5.5)
19. **Formularios** (5.6)
20. **Redirects** (5.7)
21. **Configuración SEO** (5.8)
22. **Deploy staging** (5.9)
23. **Verificación visual** frente a screenshots de referencia; corregir discrepancias (6.0)
24. **Responsive y pulido** (6.1–6.5)
25. **Deploy producción, handoff y deploy hook Sanity → Vercel** (7.1–7.4)
26. **git push** al remoto (tras los commits hechos al cierre de cada fase)

---

## Para agentes: ejecutar esta guía de punta a punta

Para que un agente pueda replicar una web WordPress → Astro y mejorarla siguiendo esta guía, conviene tener claro lo siguiente.

### Input que el agente necesita

**Para empezar:** (1) El humano hace los logins (Vercel, Sanity, GitHub) al iniciar el proyecto. (2) El agente recibe la **URL de la web origen** y, **antes de la Fase 1**, comprueba que los logins están bien (`vercel whoami`, `sanity debug`, `gh auth status`); si falla algo, avisa y no continúa; si todo OK, sigue con la Fase 1 y el humano puede irse. No hay kickoff ni brief: el objetivo es extraer todas las URLs, estudiar el contenido en cada una y recrear la web en Astro (misma estructura y contenido, mejoras técnicas).

| Fase | Input obligatorio | Input opcional |
|------|-------------------|----------------|
| 0 | Humano: logins hechos. Agente: comprobar al inicio; si OK, proseguir. | — |
| 1 | URL de la web origen (para sitemap/crawl) | — |
| 2 | Salidas de Fase 1 (URLs, estructura, auditoría, medios) | Logos/assets si no se pueden extraer de la web |
| 3–7 | Salidas de fases anteriores | Variables de entorno (Sanity, Vercel); dominio final |

### Pistas específicas de WordPress

- **URLs:** Probar `https://<origen>/sitemap.xml`, `https://<origen>/wp-sitemap.xml` o `https://<origen>/sitemap_index.xml`. Si no hay sitemap, crawlear desde la home (seguir enlaces internos del mismo dominio).
- **Contenido:** El cuerpo suele estar en `<article>`, `.post`, `.entry-content`, `.content` o similar. Título en `h1` o `.entry-title`. WordPress suele poner imágenes en `wp-content/uploads`; extraer `src` de `<img>` y enlaces a adjuntos.
- **Metadatos:** Revisar `<meta name="description">`, `og:title`, `og:description`, `og:image`; a veces hay más datos en JSON-LD o en la propia página.
- **Tipos de página:** Inferir por URL: `/page-slug/` o raíz = páginas estáticas; `/category/`, `/tag/`, `/blog/` = listados; path con un segmento que no es página conocida = a menudo detalle (entrada o custom post type). La lista de URLs (1.1) + inspección de unas pocas por patrón permite clasificar (1.2).

### Inferir schemas de Sanity desde la auditoría

- **Un documento tipo por cada “tipo de página” con contenido editable:** home = singleton `homepage`, contacto = singleton `contacto`, etc.
- **Un documento tipo por cada listado/detalle de contenido:** si hay “proyectos” → tipo `project` con slug, título, cuerpo, imágenes, campos que hayas visto en la auditoría; si hay blog → tipo `blogPost` con title, slug, body, mainImage, publishedAt.
- **Singleton de configuración:** `siteSettings` (nombre sitio, redes, footer, etc.).
- Campos = lo que extrajiste en 1.4/2.5 (título, descripción, imágenes, bloques de texto, fechas, categorías). Si un campo es lista de ítems (galería, proyectos destacados), usar array de referencias o array de objetos.

### Optimización de imágenes sin depender de TinyJPG web

TinyJPG es manual en la web. Para automatizar: usar **sharp** (Node) o **squoosh** (CLI) para comprimir y convertir a WebP/AVIF; o un script que descargue, redimensione y comprima. Interpretar “TinyJPG” como “comprimir imágenes para reducir peso” y usar la herramienta que permita automatización (sharp, squoosh, etc.).

### Artefactos que el agente debe producir (por fase)

| Fase | Artefactos |
|------|------------|
| 1 | `urls.json`; `structure.json` o doc (nav, árbol, templates); `audit.json` por cada URL con title, tipo, body, imágenes in-page, forms, embeds; **estructura de layout por tipo de página**; **header** (comportamiento al scroll + enlaces) y **footer** (todos los enlaces, incluidas redes sociales); lista de medios (URL origen + uso previsto); **screenshots de referencia** del origen en `phase1/reference-screenshots/` (por tipo de página y viewport) |
| 2 | Logos en repo; **colores y tipografías extraídos del origen** (documentados); todas las imágenes descargadas y optimizadas (WebP/AVIF donde aplique); `content/*.json` por tipo; revisión ortográfica documentada |
| 3 | Repo con `studio-*/` (schemas, seed scripts); contenido en Sanity; Custom desk; **Studio desplegado en producción** (URL `https://<studioHost>.sanity.studio/` documentada en handoff); o doc de excepción si no se usa Sanity |
| 4 | Guía de estilos **con colores y tipografía del origen** (tokens/componentes/breakpoints); especificación header (comportamiento al scroll, todos los enlaces) y footer (todos los enlaces, **cada red social**); favicon.ico, favicon.svg, apple-touch-icon; **layouts por tipo replicando estructura y orden del origen** |
| 5 | Astro + Sanity (env, GROQ); todas las rutas equivalentes al origen; **páginas replicando estructura y layout del origen** (4.5); componente de imagen (lazy load, lightbox si aplica); formularios con destino real y consentimiento/avisos; redirects; SEO completo (meta, og, twitter, sitemap, robots, JSON-LD si aplica); URL de staging |
| 6 | **Verificación visual** (6.0): comparación sitio nuevo vs screenshots de referencia; discrepancias corregidas; ajustes móvil por página; grids de imágenes; revisión hover/tap; lazy load y formatos imagen; revisión a11y (contraste, foco, labels, encabezados) |
| 7 | URL de producción; handoff con edición de contenido, despliegue y variables de entorno; deploy hook Sanity → Vercel documentado en handoff |

### Criterios de “fase completada” (para no avanzar a ciegas)

- **Fase 1:** Todas las URLs clasificadas; audit con body, imágenes y forms por URL; **screenshots de referencia** del origen por tipo de página y viewport (1.6); **estructura de layout por tipo de página**; **header (comportamiento al scroll + enlaces) y footer (todos los enlaces, incluidas redes)**; lista de medios completa (URL + uso).
- **Fase 2:** Contenido en JSON/CSV; **colores y tipografías extraídos del origen** documentados; todas las imágenes descargadas y optimizadas (o solo logo si el sitio no tiene más); ortografía revisada y documentada.
- **Fase 3:** Sanity con schemas, seed, documentos con imágenes, Custom desk, **Studio desplegado (URL en handoff)**; o excepción documentada.
- **Fase 4:** Guía con **colores y tipografía del origen** (tokens/componentes/breakpoints); especificación header (comportamiento al scroll, todos los enlaces) y footer (**todos los enlaces, cada red social**); favicon.ico, favicon.svg, apple-touch-icon; **layouts por tipo replicando estructura y orden del origen**.
- **Fase 5:** Todas las rutas del origen; **layout y estructura de cada tipo de página replicando el origen**; colores y tipografía del origen aplicados; header y footer con todos los enlaces y comportamiento; componente de imagen (lazy load; lightbox si hay galerías); formularios con destino y consentimiento/avisos; og, twitter, sitemap, robots, JSON-LD si aplica; URL de staging.
- **Fase 6:** **Verificación visual (6.0)** hecha: comparación con screenshots de referencia por tipo de página y viewport; discrepancias de colores, tipografía, estructura o header/footer corregidas; móvil revisado; grids y tap targets; sin información crítica solo en hover; revisión a11y (contraste, foco, labels, h1–h6).
- **Fase 7:** URL de producción; handoff con edición de contenido, despliegue y variables de entorno; deploy hook Sanity → Vercel configurado y documentado.

### Control de versiones (git)

- **Al final de cada fase (1 a 7):** Hacer commit de los entregables de esa fase antes de pasar a la siguiente. Ejemplo: `git add . && git commit -m "Fase N: <descripción breve>"` (ej. "Fase 1: descubrimiento e inventario", "Fase 2: marca y contenido"). No dar por cerrada una fase sin haber hecho commit.
- **Al final del proceso completo:** Una vez cerrada la Fase 7 (deploy, handoff y deploy hook documentados), hacer **git push** al remoto (`git push` o `git push origin main` según la rama). Así todo el trabajo queda subido en un único push final tras los commits locales por fase.

### Qué puede fallar y qué hacer

- **La apariencia del nuevo sitio difiere del origen:** Suele deberse a colores o tipografía no extraídos correctamente (usar valores del origen, no genéricos), estructura de secciones simplificada o reordenada, o header/footer incompletos. La guía mitiga esto con: **screenshots de referencia (1.6)** del origen por tipo de página y viewport, y **verificación visual obligatoria (6.0)** antes de dar por cerrada la Fase 6: comparar el sitio nuevo con esas capturas y corregir discrepancias antes de seguir.
- **No hay sitemap:** Crawlear desde la home; limitar profundidad y mismo dominio.
- **Páginas con mucho JS:** Usar browser/crawl que renderice (Playwright/Puppeteer) para obtener HTML final.
- **Contenido que no encaja en el schema:** Ajustar schema y re-ejecutar seed (patch para no perder imágenes).
- **Imágenes rotas o bloqueadas:** Guardar URL origen en el contenido y marcar para revisión manual; no bloquear el pipeline.
- **Colores o tipografía no extraíbles:** Inspeccionar CSS computado o fuentes del sitio; si un valor concreto no se puede obtener, documentar la excepción y usar el más cercano del origen; no sustituir por paleta o fuente genérica sin constancia.

Con esto, un agente puede ejecutar la guía de punta a punta de forma autónoma: cada fase tiene criterios de “no avanzar sin X” y artefactos concretos; no se salta ningún paso.

---

## Qué hace cada rol (humano vs agente)

- **Agente (autónomo):** Ejecuta todas las fases con prioridad **replicar primero**: crawl/sitemap, clasificación, auditoría (con layout por tipo y header/footer completos), extracción de branding **del origen** (colores y tipografías, no genéricos), optimización de imágenes, schemas, seed, **deploy del Studio en producción** (studioHost único + `sanity deploy -y`), guía de estilos y layouts **extraídos del origen**, build Astro **replicando estructura y apariencia del origen**, formularios, redirects, SEO, responsive, deploy del sitio, documentación.
- **Humano (opcional):** Criterio en diseño, aprobaciones o overrides puntuales; DNS/dominio si no está automatizado.

---

## Notas y ejemplos

- **Estructura de URLs:** Depende de cada web. Los detalles pueden vivir en la raíz (`/proyecto-slug`, `/servicio-slug`) o bajo una sección (`/proyectos/proyecto-slug`, `/blog/entrada-slug`). Puede haber listados en `/servicios`, `/productos`, `/blog`, etc. Definir en 1.3 y reflejarlo en redirects (5.7).
- **Seed:** Usar `patch` cuando el documento exista para no pisar imágenes ni ediciones manuales en Sanity.
- **Redirects en static:** Con `output: 'static'`, usar meta refresh o configuración de hosting (Vercel redirects) en lugar de `Astro.redirect`.
- **Sanity:** Variables de entorno: `PUBLIC_SANITY_*` para build; secrets en `.env`/`.env.local`. Node 20.x en Vercel si 24.x da problemas.
- **Custom desk:** Agrupar por “Páginas del sitio”, los tipos de contenido que tenga la web (Proyectos, Blog, Servicios, etc.) y “Configuración”; ordenar listas (ej. por `_updatedAt` o `publishedAt`) mejora la UX para el cliente.
- **TinyJPG / optimización de imágenes:** Objetivo = comprimir para reducir peso. TinyJPG web es manual; para agentes usar sharp, squoosh CLI o similar (ver sección “Para agentes”).
