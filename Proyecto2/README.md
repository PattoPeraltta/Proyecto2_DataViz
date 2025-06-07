# 🎵 SvelteKit Vinyl Music Player

An interactive, visually-rich music player built with SvelteKit. Browse a collection of iconic albums, view their details, and play sample tracks—all presented as animated vinyl records.

## 🚀 Features

- **Animated Splash Screen**: Welcomes users with a typewriter effect.
- **Vinyl Record Player**: Albums are visualized as spinning vinyls, with overlays for genre, era, and popularity.
- **Album Carousel**: Browse and select from a collection of albums.
- **Detailed Album Info**: View artist, year, genre, ranking, description, and tracklist.
- **Audio Playback**: Play, pause, and stop controls for each album.
- **Responsive Design**: Works seamlessly on desktop and mobile.
- **Data-driven**: Album data loaded from a CSV file for easy updates.

## 🗂️ Project Structure

```
Proyecto2/
├── public/              # Static assets (images, icons)
├── src/
│   ├── routes/
│   │   ├── +layout.svelte   # App layout
│   │   └── +page.svelte     # Main music player UI & logic
│   └── lib/                 # (Optional) Shared components/utilities
├── static/
│   ├── covers/          # Album cover images
│   ├── data/albums.csv  # Album metadata (id, title, artist, ...)
│   ├── music/           # Audio files for each album
│   ├── overlays/        # UI overlays (era, grooves, ranking)
│   └── vinyls/          # Vinyl base images by genre
├── package.json         # Project metadata & scripts
├── svelte.config.js     # SvelteKit config
└── vite.config.ts       # Vite config
```

## 📦 Setup & Development

1. **Install dependencies:**
   ```bash
   npm install
   # or
   pnpm install
   # or
   yarn install
   ```

2. **Run the development server:**
   ```bash
   npm run dev
   # or
   npm run dev -- --open  # Opens in browser
   ```

3. **Build for production:**
   ```bash
   npm run build
   ```

4. **Preview the production build:**
   ```bash
   npm run preview
   ```

## 📝 Data & Customization

- **Add/Update Albums:**
  - Edit `static/data/albums.csv` to add new albums or update existing ones.
  - Place corresponding cover images in `static/covers/` and audio files in `static/music/`.
- **Genres, Eras, and Overlays:**
  - Add or modify vinyl base images in `static/vinyls/`.
  - Overlays for era and grooves are in `static/overlays/`.

## 🛠️ Scripts

- `npm run dev` — Start development server
- `npm run build` — Build for production
- `npm run preview` — Preview production build
- `npm run lint` — Lint code
- `npm run format` — Format code

## 📚 Tech Stack

- [SvelteKit](https://kit.svelte.dev/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Vite](https://vitejs.dev/)

## 📄 License

MIT (or specify your license here)

---

> Inspired by the beauty of vinyl and the power of Svelte.
