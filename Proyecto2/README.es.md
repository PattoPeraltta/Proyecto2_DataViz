# 🎵 Reproductor de Música en Vinilo con SvelteKit

Un reproductor de música interactivo y visualmente atractivo construido con SvelteKit. Navega por una colección de álbumes icónicos, consulta sus detalles y reproduce pistas de muestra, todo presentado como discos de vinilo animados.

## 🚀 Características

- **Pantalla de bienvenida animada**: Da la bienvenida a los usuarios con un efecto de máquina de escribir.
- **Reproductor de discos de vinilo**: Los álbumes se visualizan como vinilos giratorios, con superposiciones para género, época y popularidad.
- **Carrusel de álbumes**: Navega y selecciona entre una colección de álbumes.
- **Información detallada del álbum**: Consulta artista, año, género, ranking, descripción y lista de canciones.
- **Reproducción de audio**: Controles para reproducir, pausar y detener cada álbum.
- **Diseño responsivo**: Funciona perfectamente en escritorio y móvil.
- **Basado en datos**: Los datos de los álbumes se cargan desde un archivo CSV para facilitar actualizaciones.

## 🗂️ Estructura del Proyecto

```
Proyecto2/
├── public/              # Recursos estáticos (imágenes, iconos)
├── src/
│   ├── routes/
│   │   ├── +layout.svelte   # Diseño de la app
│   │   └── +page.svelte     # Interfaz y lógica principal del reproductor
│   └── lib/                 # (Opcional) Componentes/utilidades compartidas
├── static/
│   ├── covers/          # Imágenes de portadas de álbumes
│   ├── data/albums.csv  # Metadatos de álbumes (id, título, artista, ...)
│   ├── music/           # Archivos de audio de cada álbum
│   ├── overlays/        # Superposiciones de la interfaz (época, surcos, ranking)
│   └── vinyls/          # Imágenes base de vinilo por género
├── package.json         # Metadatos y scripts del proyecto
├── svelte.config.js     # Configuración de SvelteKit
└── vite.config.ts       # Configuración de Vite
```

## 📦 Instalación y Desarrollo

1. **Instala las dependencias:**
   ```bash
   npm install
   # o
   pnpm install
   # o
   yarn install
   ```

2. **Ejecuta el servidor de desarrollo:**
   ```bash
   npm run dev
   # o
   npm run dev -- --open  # Abre en el navegador
   ```

3. **Construye para producción:**
   ```bash
   npm run build
   ```

4. **Previsualiza la versión de producción:**
   ```bash
   npm run preview
   ```

## 📝 Datos y Personalización

- **Agregar/Actualizar álbumes:**
  - Edita `static/data/albums.csv` para añadir nuevos álbumes o actualizar los existentes.
  - Coloca las imágenes de portada correspondientes en `static/covers/` y los archivos de audio en `static/music/`.
- **Géneros, épocas y superposiciones:**
  - Agrega o modifica imágenes base de vinilo en `static/vinyls/`.
  - Las superposiciones de época y surcos están en `static/overlays/`.

## 🛠️ Scripts

- `npm run dev` — Inicia el servidor de desarrollo
- `npm run build` — Construye para producción
- `npm run preview` — Previsualiza la versión de producción
- `npm run lint` — Lint del código
- `npm run format` — Formatea el código

## 📚 Tecnologías

- [SvelteKit](https://kit.svelte.dev/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Vite](https://vitejs.dev/)

## 📄 Licencia

MIT (o especifica tu licencia aquí)

---

> Inspirado en la belleza del vinilo y el poder de Svelte.
