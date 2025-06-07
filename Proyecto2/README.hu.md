# 🎵 SvelteKit Vinyl Zenelejátszó

Egy interaktív, vizuálisan gazdag zenelejátszó SvelteKit-tel. Böngéssz ikonikus albumok gyűjteményében, nézd meg a részleteiket, és hallgass bele a mintaszámokba – mindezt animált bakelitlemezek formájában.

## 🚀 Funkciók

- **Animált Nyitóképernyő**: Gépelő effektussal üdvözli a felhasználót.
- **Bakelit Lemezlejátszó**: Az albumok forgó bakelitlemezként jelennek meg, műfaj, korszak és népszerűség átfedésekkel.
- **Album Körhinta**: Albumok böngészése és kiválasztása.
- **Részletes Albuminformációk**: Előadó, év, műfaj, rangsor, leírás és számlista megtekintése.
- **Hang lejátszás**: Lejátszás, szüneteltetés és leállítás minden albumhoz.
- **Reszponzív dizájn**: Asztali és mobil eszközökön is tökéletesen működik.
- **Adatvezérelt**: Az albumadatok CSV fájlból töltődnek be a könnyű frissítés érdekében.

## 🗂️ Projektstruktúra

```
Proyecto2/
├── public/              # Statikus elemek (képek, ikonok)
├── src/
│   ├── routes/
│   │   ├── +layout.svelte   # Alkalmazás elrendezése
│   │   └── +page.svelte     # Fő zenelejátszó UI & logika
│   └── lib/                 # (Opcionális) Megosztott komponensek/segédeszközök
├── static/
│   ├── covers/          # Album borítóképek
│   ├── data/albums.csv  # Album metaadatok (id, cím, előadó, ...)
│   ├── music/           # Hangfájlok minden albumhoz
│   ├── overlays/        # UI átfedések (korszak, barázdák, rangsor)
│   └── vinyls/          # Bakelit alapképek műfaj szerint
├── package.json         # Projekt metaadatok & scriptek
├── svelte.config.js     # SvelteKit konfiguráció
└── vite.config.ts       # Vite konfiguráció
```

## 📦 Telepítés & Fejlesztés

1. **Függőségek telepítése:**
   ```bash
   npm install
   # vagy
   pnpm install
   # vagy
   yarn install
   ```

2. **Fejlesztői szerver indítása:**
   ```bash
   npm run dev
   # vagy
   npm run dev -- --open  # Megnyitás böngészőben
   ```

3. **Build készítése éles környezethez:**
   ```bash
   npm run build
   ```

4. **Éles build előnézete:**
   ```bash
   npm run preview
   ```

## 📝 Adatok & Testreszabás

- **Albumok hozzáadása/frissítése:**
  - Szerkeszd a `static/data/albums.csv` fájlt új albumok hozzáadásához vagy meglévők frissítéséhez.
  - Helyezd el a megfelelő borítóképeket a `static/covers/` mappában és a hangfájlokat a `static/music/` mappában.
- **Műfajok, korszakok és átfedések:**
  - Adj hozzá vagy módosíts bakelit alapképeket a `static/vinyls/` mappában.
  - Korszak és barázda átfedések a `static/overlays/` mappában találhatók.

## 🛠️ Parancsok

- `npm run dev` — Fejlesztői szerver indítása
- `npm run build` — Build készítése éles környezethez
- `npm run preview` — Éles build előnézete
- `npm run lint` — Kód ellenőrzése
- `npm run format` — Kód formázása

## 📚 Technológiai Stack

- [SvelteKit](https://kit.svelte.dev/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Vite](https://vitejs.dev/)

## 📄 Licenc

MIT (vagy add meg a saját licencet itt)

---

> A bakelit szépsége és a Svelte ereje inspirálta.
