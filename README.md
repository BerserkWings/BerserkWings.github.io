# 🖥️ BerserkWings.github.io
 
Blog personal de writeups de ciberseguridad (**HackTheBox**, **TheHackerLabs**) y notas de aprendizaje, construido con **Hugo** — tema propio (cyberpunk/terminal), sin usar ningún theme de terceros.
 
**🔗 Sitio en vivo:** [berserkwings.github.io](https://berserkwings.github.io/)
 
---
 
## 📑 Índice
 
- [¿Cómo está construido?](#-cómo-está-construido)
- [Estructura del repositorio](#-estructura-del-repositorio)
- [Front matter de un post](#-front-matter-de-un-post)
- [Crear un post nuevo](#-crear-un-post-nuevo)
- [Desarrollo local](#-desarrollo-local)
- [Desplegar en GitHub Pages](#-desplegar-en-github-pages)
- [Buy Me a Coffee](#-buy-me-a-coffee)
- [Cómo funciona el buscador](#-cómo-funciona-el-buscador-y-por-qué-es-seguro)
---
 
## 🛠️ ¿Cómo está construido?
 
El sitio migró de **Jekyll** a **Hugo**. Algunas piezas destacadas:
 
- **ASCII art generado en el navegador** — el avatar se convierte a caracteres ASCII con JavaScript y Canvas, no es una imagen estática.
- **Fondo animado estilo Matrix** — canvas decorativo, con límite de FPS, pausa automática si la pestaña no está visible, y respeto a `prefers-reduced-motion`.
- **Sistema de embargo para máquinas activas** — un writeup puede publicarse (y aparecer en su lugar cronológico correcto, según su fecha real) sin revelar el contenido mientras la máquina de HTB/THL siga activa en la plataforma. Ver [`unlock_date`](#-front-matter-de-un-post) más abajo.
- **Buscador propio** — índice generado por Hugo en cada build (`search-index.json`), sin depender de ningún servicio externo, y que respeta el mismo embargo (nunca expone el contenido real de un post embargado).
- **Feed RSS propio** — personalizado para respetar el embargo también (Hugo trae uno por defecto que no lo hace).
- **Página 404 propia**, con la misma estética de terminal.
- **Recompilación diaria automática** (vía GitHub Actions) para que los writeups embargados se desbloqueen solos en la fecha indicada, sin necesidad de hacer push ese día.
---
 
## 📁 Estructura del repositorio
 
```text
hugo-site/
├── archetypes/default.md            # plantilla para "hugo new"
├── content/
│   ├── posts/                       # tus writeups (.md)
│   └── pages/                       # acerca-de-mi, buscador, todos
├── layouts/
│   ├── _default/                    # baseof, single (post), list, search, about, allposts
│   ├── categories/terms.html        # listado de categorías (tarjetas con conteo)
│   ├── tags/terms.html              # nube de etiquetas por frecuencia
│   ├── pages/single.html            # layout simple para páginas estáticas
│   ├── partials/                    # head, header, footer, sidebar, TOC, paginación, etc.
│   └── index.searchindex.json       # genera el índice del buscador
├── assets/
│   ├── css/main.css                 # design tokens + todos los estilos
│   └── js/main.js                   # buscador, carruseles, ASCII art, frame buster, etc.
├── static/assets/images/            # tus imágenes (cópialas aquí)
├── scripts/migrate_posts.py         # migrador Jekyll -> Hugo
└── .github/workflows/hugo.yml       # build + deploy a GitHub Pages
```
 
---
 
## 📝 Front matter de un post
 
Campos relevantes que usa el tema, además de los estándar de Hugo (`title`, `date`, `categories`, `tags`):
 
| Campo | Descripción |
|---|---|
| `excerpt` | Resumen corto usado en las tarjetas y el buscador. |
| `header.teaser` | Ruta a la imagen de portada del post. |
| `unlock_date` *(opcional)* | Si la máquina sigue activa en la plataforma, esta fecha activa la pantalla de "candado" en el post, en las tarjetas, en el buscador y en el RSS. |
 
Ver `archetypes/default.md` para el detalle completo.
 
### Crear un post nuevo
 
```bash
hugo new posts/htb-writeup-nombremaquina.md
```
 
---
 
## 💻 Desarrollo local
 
Requiere **Hugo Extended** (versión fijada en `.github/workflows/hugo.yml`).
 
```bash
hugo server -D
```
 
---
 
## 🚀 Desplegar en GitHub Pages
 
1. En **Settings → Pages → Build and deployment → Source**, elige **"GitHub Actions"** (los repos nuevos no lo traen activado por defecto).
2. Sube el proyecto (`git push`) a la rama `master`.
3. El workflow `.github/workflows/hugo.yml` compila con la versión de Hugo fijada (con checksum verificado) y publica a GitHub Pages.
> Actualiza `baseURL` en `hugo.toml` si alguna vez cambias de dominio o pruebas en un repo con otro nombre — Hugo genera rutas distintas según eso.
 
---
 
## ☕ Buy Me a Coffee
 
En cuanto tengas tu cuenta, en `hugo.toml`:
 
```toml
buymeacoffee_username = "tu-usuario"
```
 
El botón aparece solo al final de cada post y en el sidebar de perfil. Mientras el campo esté vacío, no se muestra nada.
 
---
 
## 🔍 Cómo funciona el buscador (y por qué es seguro)
 
Hugo genera `search-index.json` en cada build con título, excerpt, categorías y tags de cada post — **nunca el contenido completo**, y **nunca los posts embargados** (`unlock_date` en el futuro quedan fuera del índice hasta que se desbloquean). El JavaScript del buscador carga ese JSON estático y filtra en el navegador, sin llamadas a ningún servicio externo ni backend propio: no hay superficie de ataque adicional más allá de servir un archivo estático.
 
---
 
## ⚖️ Licencia
 
- **Código:** MIT — ver [`LICENSE`](./LICENSE).
- **Contenido escrito (writeups y posts):** todos los derechos reservados.
- Ver [`DISCLAIMER.md`](./DISCLAIMER.md) para el deslinde de responsabilidad sobre el contenido de hacking ético.
