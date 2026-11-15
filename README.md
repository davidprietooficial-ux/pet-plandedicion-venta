# pet-plandedicion-venta

Página de venta del programa **Plan de Edición Total (PET)**: landing de una sola página que se comparte al cerrar el webinar semanal, con el checkout de Hotmart embebido y una oferta (bono de IA incluido) que se abre y se cierra sola cada semana.

## Producción

| Dato | Valor |
|---|---|
| URL en producción | https://pet-plandedicion-venta.vercel.app (es la URL canónica en `index.html`) |
| Hosting actual | Vercel. Comprobado el 2026-11-14: el HTML que sirve coincide con `index.html` de `main` |
| Migración a Hostinger | Preparada en el código (último commit: se quitó `vercel.json` y se añadió `public/.htaccess`) |
| Subdominio en Hostinger | `venta-envivo.planediciontotal.com`. Comprobado el 2026-11-14: sirve esta misma landing con las cabeceras de `public/.htaccess`, pero **una versión distinta de `main`** (ver "Problemas conocidos", punto 1). Que sea el destino oficial: **[por confirmar]** |
| Repositorio | github.com/davidprietooficial-ux/pet-plandedicion-venta (única rama: `main`) |

El `name` de `package.json` es `plan-de-edicion-total` (nombre antiguo); no afecta al build.

## Stack

- **Vite 6** + **TypeScript 5** (modo estricto, sin framework): HTML estático y módulos TS de mejora progresiva.
- **Tailwind CSS 4** vía `@tailwindcss/vite`, más `tailwind-animations`.
- **Prettier** con `prettier-plugin-tailwindcss`.
- Sin dependencias en tiempo de ejecución: todo el JS es propio.
- Pago: **Hotmart**, checkout embebido en un `<iframe>` (`checkoutMode=10`).

## Estructura

```
index.html                  Toda la landing (hero, checkout, módulos, testimonios, FAQ, cierre, pie) + JSON-LD
vite.config.ts              Build: sin sourcemaps, assets con hash en assets/, un solo bundle
src/
  main.ts                   Punto de entrada; orden de arranque: estilos → consentimiento → UI → tracking
  datos/oferta.ts           Ventana semanal de la oferta, zona horaria y enlaces/precios de referencia
  lib/oferta.ts             Calcula si la oferta está activa, cuenta regresiva, cambia el iframe de Hotmart
  lib/checkout.ts           Ajusta el alto del iframe de Hotmart con sus mensajes postMessage
  lib/consentimiento.ts     Banner de cookies y cola de scripts de terceros (Consent Mode v2)
  lib/tracking.ts           IDs de píxeles/analítica (vacíos) y eventos de conversión
  lib/testimonios.ts        Reproductor nativo de los videos de testimonios
  lib/contador-vivo.ts      Píldora "Ya entraron N" del hero
  lib/banner.ts             Mide el alto de la barra de oferta fija (--banner-alto)
  lib/revelar.ts            Animaciones de aparición al hacer scroll y contadores
  estilos/tokens.css        Tokens de diseño, fuentes y breakpoints (marcado como "archivo generado")
  estilos/sitio.css         Estilos del sitio (incluye el alto inicial del checkout)
public/                     Se copia tal cual a dist/
  .htaccess                 HTTPS, cabeceras de seguridad, caché y bloqueos (Apache/LiteSpeed)
  robots.txt, favicon.svg, og.jpg
  aviso-legal/, politica-de-cookies/, politica-de-privacidad/, terminos-y-condiciones/   Páginas legales (index.html estático)
  hero/                     Video de fondo del hero, su poster y banner
  testimonios/              4 videos .mp4 + sus posters .webp
  modulos/, bonos/          Imágenes del producto y de los bonos
  fuentes/                  Special Gothic Expanded One e Inter Tight (woff2, autoalojadas)
```

`public/` pesa unos 31 MB, casi todo por los videos de testimonios.

## Correr en local

Requiere Node.js (versión mínima **[por confirmar]**; no hay campo `engines` ni `.nvmrc`).

```bash
npm install
npm run dev            # servidor de Vite en http://localhost:5173 (abre el navegador)
npm run build          # tsc --noEmit (chequeo de tipos) + vite build → dist/
npm run preview        # sirve dist/ en http://localhost:4173
npm run formato        # prettier --write .
npm run formato:check  # prettier --check .
```

Para ver la página como queda **después** del corte de la oferta sin esperar al martes: añadir `?vista=vencida` a la URL.

## Despliegue

### Vercel (producción actual)

- Sirve `pet-plandedicion-venta.vercel.app`. Nombre del proyecto en Vercel: **[por confirmar]**.
- **[por confirmar]** que esté conectado a GitHub y despliegue solo con cada push a `main`.
- Ya no hay `vercel.json`: Vercel usa la detección automática de Vite (`npm run build`, salida `dist/`) **[por confirmar en el panel]**.
- Ojo: en Vercel **no se aplican** las cabeceras de `public/.htaccess` (ver "Problemas conocidos", punto 2). Las que tenía el antiguo `vercel.json` siguen en el historial: `git show 254cc42:vercel.json`.

### Hostinger (destino preparado en el código)

En Hostinger ningún subdominio está conectado a Git: se sube a mano (Administrador de archivos de hPanel, FTP, o la API/MCP de Hostinger).

1. `npm run build` en local.
2. Comprobar que existe `dist/.htaccess` (Vite lo copia desde `public/`): `test -f dist/.htaccess`.
3. Subir **el contenido** de `dist/` (no la carpeta en sí, ni el repo) a la carpeta raíz del subdominio. Subdominio de destino: **[por confirmar]** (`venta-envivo` es el candidato). Ojo: subir `main` a `venta-envivo` **reemplaza la versión distinta que hay hoy ahí** (precio 37/47 USD); confirmar antes cuál es la correcta.
4. Si el dominio final deja de ser `vercel.app`, cambiar antes del build las 4 apariciones de `https://pet-plandedicion-venta.vercel.app` en `index.html` (canonical, `og:url`, `og:image`, JSON-LD).
5. Verificar después de subir:
   - `curl -sI https://<dominio>/` muestra `Content-Security-Policy` y `Strict-Transport-Security`.
   - `curl -s -o /dev/null -w "%{http_code}\n" https://<dominio>/.env` devuelve 403 o 404 (en `venta-envivo` da 403 hoy).
   - El checkout de Hotmart carga y se puede pagar.

## Oferta semanal, precios y enlaces (lo que más se edita)

La oferta **no tiene fecha fija**: `src/lib/oferta.ts` recalcula en cada segundo la ventana de la semana en curso, en hora de Bogotá (UTC−5, sin horario de verano). Hoy va del **lunes 6:00 pm al martes 10:00 pm**.

| Qué cambiar | Dónde |
|---|---|
| Día y hora de inicio y de fin de la oferta | `src/datos/oferta.ts` (`DIA_INICIO_OFERTA`, `HORA_INICIO_OFERTA`, `DIA_FIN_OFERTA`, `HORA_FIN_OFERTA`) |
| Enlace del checkout con oferta (con el bono de IA) | `index.html`, iframe `data-checkout-frame`: atributos `data-src-activa` **y** `src` (deben ser iguales) |
| Enlace del checkout regular (después del corte) | `index.html`, mismo iframe: `data-src-vencida` |
| Precio visible (47 USD por cada 6 meses de acceso, con renovación automática) y precio del bono de IA (27 USD) | `index.html`: tarjeta de precio, FAQ visible **y** FAQ del JSON-LD del `<head>`. Las constantes `PRECIO_USD` y `PRECIO_BONO_IA_USD` de `src/datos/oferta.ts` no se usan en ninguna parte |
| Texto "24 horas" de la oferta | `index.html`: `meta description`, `og:description`, la FAQ (visible y JSON-LD) y el valor inicial `24:00:00` de los dos `data-cuenta-regresiva` |
| Textos, módulos, FAQ, testimonios | `index.html` (las FAQ están dos veces: en `<details>` y en el JSON-LD) |
| Qué se muestra solo durante o después de la oferta | Atributos `data-oferta-estado="activa"` / `"vencida"` en `index.html` |
| Videos e imágenes | `public/hero/`, `public/testimonios/`, `public/modulos/`, `public/bonos/` (mismo nombre de archivo = sin tocar HTML) |
| Textos legales y datos del titular | `public/*/index.html` |
| Alto inicial del checkout | `src/lib/checkout.ts` (`TECHO_INICIAL_*_PX`) **y** `src/estilos/sitio.css` (`--checkout-alto-movil`, `--checkout-alto-tablet`): los dos pares deben coincidir |

Producto en Hotmart: el código de producto público que aparece en los enlaces es `D100998112T` (oferta `off=1oaqxr4m`, regular `off=r4q32fqa`).

## Tracking e integraciones

| Integración | Estado en el código |
|---|---|
| Hotmart | Checkout embebido (`pay.hotmart.com`, `checkoutMode=10`). El alto del iframe se ajusta con el mensaje `elements_document_height_update` que manda Hotmart |
| Meta Pixel, GA4, GTM, Google Ads, TikTok Pixel | **Desactivados**: todos los IDs están vacíos en `IDS` de `src/lib/tracking.ts`. Hoy la página no envía analítica |
| Consentimiento de cookies | `src/lib/consentimiento.ts`: ningún tercero se carga antes de "Aceptar todo"; la decisión se guarda en `localStorage` (clave `consentimiento`). Subir `VERSION` vuelve a preguntar a todos |
| Eventos de conversión | `clic_whatsapp` y `clic_telefono` (directos), y `conversion` con `tipo: 'clic_comprar'` al pulsar un `data-cta-compra`. Se envían a `gtag` y a `fbq` (`trackCustom`) solo si están cargados |
| WhatsApp | Solo en las páginas legales (enlace `wa.me` de contacto). La landing no tiene enlaces `wa.me` ni `tel:` |
| Microsoft Clarity, formularios | No hay |

Para activar un píxel: poner su ID en `IDS` **y** ampliar la CSP de `public/.htaccess` (ver abajo); si no, el navegador lo bloquea.

## Seguridad

Todo está en `public/.htaccess` (solo funciona en Apache/LiteSpeed, es decir, en Hostinger):

- Redirección a HTTPS y quita el `www`.
- HSTS de un año con `includeSubDomains; preload`: no activar en un dominio sin certificado funcionando, porque no se puede deshacer durante ese año.
- CSP estricta: `script-src 'self'`, `connect-src 'self'`, `frame-src https://*.hotmart.com`, `frame-ancestors 'none'`, y Trusted Types (`trusted-types scripts-analitica`, política definida en `src/lib/tracking.ts`, que solo acepta scripts de googletagmanager.com, connect.facebook.net y analytics.tiktok.com).
- `Permissions-Policy` con `payment` permitido solo para `pay.hotmart.com`; `X-Frame-Options: DENY`, `nosniff` y `Referrer-Policy`.
- Caché de un año para assets con hash y sin caché para HTML (para que el cambio de precio u oferta se vea al momento).
- Bloquea archivos ocultos, `.env`, `.map`, `.json`, `.md`, etc.; desactiva el listado de carpetas; bloquea métodos HTTP no usados; limita el cuerpo de las peticiones a 2 MB.
- Build sin sourcemaps (`vite.config.ts`) y `.gitignore` excluye `.env*`, `*.pem`, `*.key` y `*.map`.

## Problemas conocidos

1. **`venta-envivo` no sirve lo que hay en `main`.** Comprobado el 2026-11-14: la página publicada en Hostinger dice "37 USD dentro de la ventana de la oferta, 47 USD después" y "hasta el lunes 7:00pm", mientras que `main` dice 47 USD y "24 horas". Ese texto no está en ningún commit del repo, así que de dónde salió está **[por confirmar]**. Antes de volver a subir a ese subdominio, decidir cuál de las dos versiones vale y llevarla a `main`.
2. **En Vercel no hay cabeceras de seguridad.** Se quitó `vercel.json` y `.htaccess` no aplica en Vercel. Comprobado el 2026-11-14 con `curl -sI`: Vercel no envía CSP, `X-Frame-Options`, `Permissions-Policy` ni `nosniff`; solo su HSTS por defecto.
3. **Activar analítica exige tocar la CSP.** `script-src 'self'` y `connect-src 'self'` bloquean GA4, GTM, Meta y TikTok aunque se pongan los IDs. Además, la carga de Meta y TikTok no usa el snippet oficial (que crea `fbq`/`ttq` antes del script): **[por confirmar]** que funcione, probar antes de dar por buena la medición.
4. **24 h en el texto y 28 h en el código.** El copy dice "24 horas" y el contador arranca en `24:00:00`, pero la ventana real (lunes 6 pm → martes 10 pm) dura 28 h.
5. **Marcadores de enlace sin usar.** `ENLACE_OFERTA` y `ENLACE_REGULAR` en `src/datos/oferta.ts` siguen con `PENDIENTE-...`. Hoy ningún elemento tiene `data-cta-compra`, así que no afectan; si alguien añade un botón con ese atributo, apuntará a un enlace roto. Los enlaces reales están en el iframe de `index.html`.
6. **Canonical y Open Graph fijados a `vercel.app`.** También en la copia de `venta-envivo` (comprobado). Hay que cambiarlos si el dominio definitivo es otro.
7. **`robots.txt` con restos de plantilla.** `Sitemap: https://EJEMPLO.com/sitemap.xml` (no hay sitemap) y `Disallow: /formulario.php` (no existe).
8. **Sin JS se muestra siempre la oferta activa.** Es una decisión documentada en `src/lib/oferta.ts`.
9. **La píldora "Ya entraron N" no viene de datos reales.** Se calcula en el navegador: empieza en 1 y sube de uno en uno cada 15-25 s hasta 70 (`src/lib/contador-vivo.ts`).
10. **Las 4 páginas legales están en borrador.** Cada una abre con un comentario "BORRADOR… NO es revisión legal" y dice que se copió del sitio hermano de la masterclass (`pet-editagratis-form`). En `aviso-legal` faltan los datos reales del titular (persona o empresa, NIT/cédula, ciudad).
11. **No se puede reabrir el banner de cookies.** `reabrirPreferencias()` existe en `src/lib/consentimiento.ts`, pero ningún enlace del pie la llama.
12. **Referencias a herramientas que no están en el repo.** Los comentarios citan `npm run gate`, `blindar.md` y un "compilador de tokens" que generaría `src/estilos/tokens.css` (el archivo dice "ARCHIVO GENERADO — NO SE EDITA A MANO"). Nada de eso existe aquí: en la práctica `tokens.css` se edita a mano **[por confirmar]**.
13. **Sin tests.** La única verificación automática es `tsc --noEmit` dentro de `npm run build`.

## Repos relacionados

Todos en github.com/davidprietooficial-ux, con la convención `pet-<sitio>`:

| Repo | Sitio |
|---|---|
| pet-form-webinar | form-webinar.planediciontotal.com (Hostinger) |
| pet-clase | clase.planediciontotal.com (Hostinger) |
| **pet-plandedicion-venta** | pet-plandedicion-venta.vercel.app (Vercel), este repo |
| pet-capcutclub | Landing de CapCut Club |
| pet-capcutclub-preview | Boceto de revisión en GitHub Pages |
| pet-workshopexclusivo | workshopexclusivo.planediciontotal.com (Hostinger, privado) |

Sitio de referencia mencionado en el código (no está en la lista de repos): `pet-editagratis-form.vercel.app`, de donde se copiaron el pie, el formato del bloque del checkout y las páginas legales. Su repo: **[por confirmar]**.

Subdominios existentes en Hostinger bajo `planediciontotal.com`: `clase`, `form-webinar`, `venta-envivo`, `capcutclub` (y `workshopexclusivo`, en creación).
