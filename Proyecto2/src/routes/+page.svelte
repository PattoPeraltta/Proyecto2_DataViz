<script>
  import { run } from 'svelte/legacy';

  import { onMount, tick } from 'svelte';
  import { fade, fly, scale } from 'svelte/transition';
  import { spring, tweened } from 'svelte/motion';
  import { cubicOut, linear } from 'svelte/easing';

  const fullText = "Welcome to My Music Player"; // ← change this to whatever message you like
  let displayedText = $state("");
  let typingIndex = 0;

  onMount(() => {
    // Type one character every 100ms
    const interval = setInterval(() => {
      if (typingIndex < fullText.length) {
        displayedText += fullText[typingIndex];
        typingIndex += 1;
      } else {
        clearInterval(interval);
        // Optionally: after a short pause, keep splash in place until user scrolls
      }
    }, 100);
  });

  let records = $state([]);
  let audioElement = $state();
  
  onMount(async () => {
    const res = await fetch('/data/albums.csv');
    const text = await res.text();
    const parsed = parseCSV(text);
    records = parsed.map(entry => ({
      id: +entry.id,
      title: entry.title,
      artist: entry.artist,
      year: +entry.year,
      genre: entry.genre,
      playsMillions: +entry.playsMillions,
      ranking: +entry.ranking,
      description: entry.description,
      tracks: entry.tracks.split(';').map(t => t.trim())
    }));
  });

  function parseCSV(text) {
    const lines = text.trim().split('\n');
    const headers = lines[0].split(',');
    return lines.slice(1).map(line => {
      const values = line.split(/,(?=(?:(?:[^"]*"){2})*[^"]*$)/).map(val => val.replace(/^"|"$/g, ''));
      const obj = {};
      headers.forEach((h, i) => obj[h] = values[i]);
      return obj;
    });
  }

  // State variables
  let currentRecordId = $state(null);
  let isPlaying = $state(false);
  let volume = $state(75);
  let activeIndex = $state(0);
  let tonearmRotation = spring(0);
  let recordRotation = $state(0);
  let animationFrame;
  let selectedRecord = $derived(records.find(record => record.id === currentRecordId));

  const visibleCount = 4;
  
  // Derived state
  
    
  // Handle record selection
  function handleRecordSelect(id) {
    if (currentRecordId === id) return;
    isPlaying = false;
    currentRecordId = id;

    const record = records.find(r => r.id === id);
    const fileName = toCamelCase(record.title);
    if (audioElement) {
      audioElement.src = `/music/${fileName}.mp3`;
      audioElement.load();
    }

    setTimeout(() => {
      isPlaying = true;
      if (audioElement) audioElement.play();
    }, 800);
  }
  
  // Handle play/pause
  function handlePlayPause() {
    if (currentRecordId === null || !audioElement) return;
    isPlaying = !isPlaying;
    isPlaying ? audioElement.play() : audioElement.pause();
  }
  
  // Handle stop
  function handleStop() {
    isPlaying = false;
    if (audioElement) {
      audioElement.pause();
      audioElement.currentTime = 0;
    }
    setTimeout(() => {
      currentRecordId = null;
    }, 300);
  }
  
  // Carousel navigation
  function handlePrevious() {
    activeIndex = Math.max(activeIndex - 1, 0);
  }

  function handleNext() {
    activeIndex = Math.min(activeIndex + 1, records.length - visibleCount);
  }

  // Animation for record rotation
  function animateRecord() {
    if (isPlaying) {
      recordRotation += 1;
      if (recordRotation >= 360) recordRotation = 0;
    }
    animationFrame = requestAnimationFrame(animateRecord);
  }
  
  // Lifecycle
  onMount(() => {
    animationFrame = requestAnimationFrame(animateRecord);
    return () => cancelAnimationFrame(animationFrame);
  });
  
  // Update tonearm position when record changes
  run(() => {
    if (currentRecordId !== null) {
      tonearmRotation.set(25);
    } else {
      tonearmRotation.set(0);
    }
  });

  const normalize = str =>
    str
      .toLowerCase()
      .replace(/\s+/g, "")
      .replace(/[^\w]/g, "");

  function getEra(year) {
    if (year < 1950) return "1900-1950";
    if (year < 2000) return "1950-2000";
    return "2000-2025";
  }

  function getGrooves(plays) {
    if (plays < 50) return 1;
    if (plays < 100) return 2;
    if (plays < 500) return 3;
    return 4;
  }

  const coverSrc     = record => `/covers/${normalize(record.title)}_cover.png`;
  const vinylBase    = record => `/vinyls/${record.genre}.png`;
  const eraOverlay   = record => `/overlays/${getEra(record.year)}.png`;
  const groovesOverlay = record => `/overlays/grooves-${getGrooves(record.playsMillions)}.png`;
  const starSticker  = ()     => `/overlays/star.png`;

  function toCamelCase(str) {
    return str
      .split(' ')
      .map((word, i) =>
        i === 0
          ? word.charAt(0).toLowerCase() + word.slice(1)
          : word.charAt(0).toUpperCase() + word.slice(1)
      )
      .join('');
  }

  // ruta al cover en static/covers
  const coverFile = record => `/covers/${toCamelCase(record.title)}.png`;
</script>

<audio bind:this={audioElement} onended={() => isPlaying = false}></audio>
<div class="splash">
  <span class="typewriter">{displayedText}</span>
</div>
<div class="main-content">
  <main class="min-h-screen bg-zinc-900 p-4 md:p-8">
    <div class="mx-auto max-w-6xl">      
      <div class="grid gap-12">
        <!-- >>> Leyendas / Referencias >>> -->
        <div class="mb-8 rounded-xl border-2 border-amber-700/50 bg-zinc-800/80 p-8 shadow-lg">
          <section class="space-y-10">
            <!-- 1) Colores ⇢ Géneros -->
            <div>
              <h3 class="mb-6 text-xl font-bold text-amber-300">Colores / Géneros</h3>
              <div class="grid grid-cols-2 gap-6 md:grid-cols-3 lg:grid-cols-6">
                <div class="flex flex-col items-center gap-3">
                  <div class="relative h-24 w-24 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                    <img src="/vinyls/Rock.png" alt="Rock" class="h-full w-full object-cover"/>
                  </div>
                  <span class="text-center font-medium text-amber-300">Rock</span>
                </div>
                <div class="flex flex-col items-center gap-3">
                  <div class="relative h-24 w-24 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                    <img src="/vinyls/Pop.png" alt="Pop" class="h-full w-full object-cover"/>
                  </div>
                  <span class="text-center font-medium text-amber-300">Pop</span>
                </div>
                <div class="flex flex-col items-center gap-3">
                  <div class="relative h-24 w-24 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                    <img src="/vinyls/Latino.png" alt="Latino" class="h-full w-full object-cover"/>
                  </div>
                  <span class="text-center font-medium text-amber-300">Latino</span>
                </div>
                <div class="flex flex-col items-center gap-3">
                  <div class="relative h-24 w-24 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                    <img src="/vinyls/Electronica.png" alt="Electrónica" class="h-full w-full object-cover"/>
                  </div>
                  <span class="text-center font-medium text-amber-300">Electrónica</span>
                </div>
                <div class="flex flex-col items-center gap-3">
                  <div class="relative h-24 w-24 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                    <img src="/vinyls/Jazz.png" alt="Jazz" class="h-full w-full object-cover"/>
                  </div>
                  <span class="text-center font-medium text-amber-300">Jazz</span>
                </div>
                <div class="flex flex-col items-center gap-3">
                  <div class="relative h-24 w-24 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                    <img src="/vinyls/HipHop.png" alt="Hip-hop/Rap" class="h-full w-full object-cover"/>
                  </div>
                  <span class="text-center font-medium text-amber-300">Hip-hop/Rap</span>
                </div>
              </div>
            </div>

            <!-- Desgaste ⇢ Épocas -->
            <div>
              <h3 class="mb-6 mt-8 text-xl font-bold text-amber-300 ">Desgaste / Épocas</h3>
              <div class="grid grid-cols-1 gap-6 md:grid-cols-3">
                <!-- 2000–2025 -->
                <div class="flex flex-col items-center gap-3">
                  <div class="relative h-28 w-28 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                    <img src="/vinyls/Base.png" alt="2000–2025" class="absolute inset-0 w-full h-full object-cover"/>
                  </div>
                  <span class="text-center font-medium text-amber-300">2000 – 2025</span>
                </div>
                <!-- 1950–2000 -->
                <div class="flex flex-col items-center gap-3">
                  <div class="relative h-28 w-28 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                    <img src="/vinyls/Base.png" alt="1950–2000" class="absolute inset-0 w-full h-full object-cover"/>
                    <img src="/overlays/1950-2000.png" alt="Desgaste medio" class="absolute inset-0 w-full h-full object-cover"/>
                  </div>
                  <span class="text-center font-medium text-amber-300">1950 – 2000</span>
                </div>
                <!-- 1900–1950 -->
                <div class="flex flex-col items-center gap-3">
                  <div class="relative h-28 w-28 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                    <img src="/vinyls/Base.png" alt="1900–1950" class="absolute inset-0 w-full h-full object-cover"/>
                    <img src="/overlays/1900-1950.png" alt="Desgaste fuerte" class="absolute inset-0 w-full h-full object-cover"/>
                  </div>
                  <span class="text-center font-medium text-amber-300">1900 – 1950</span>
                </div>
              </div>
            
            <!-- Surcos ⇢ Millones de reproducciones -->
            <div>
              <h3 class="mb-6 mt-8 text-xl font-bold text-amber-300">Surcos / Millones de reproducciones</h3>
              <div class="grid grid-cols-1 gap-6 md:grid-cols-4">
                {#each [1,2,3,4] as grooves}
                  <div class="flex flex-col items-center gap-3">
                    <div class="relative h-28 w-28 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                      <img src="/vinyls/Base.png" class="absolute inset-0 w-full h-full object-cover" alt="{grooves} surco(s)"/>
                      <img src={`/overlays/grooves-${grooves}.png`} class="absolute inset-0 w-full h-full object-cover" alt="{grooves} surco(s) overlay"/>
                    </div>
                    <span class="text-center font-medium text-amber-300">
                      {grooves === 1
                        ? "< 50 M"
                        : grooves === 2
                        ? "< 100 M"
                        : grooves === 3
                        ? "< 500 M"
                        : "> 500 M"}
                    </span>
                  </div>
                {/each}
              </div>
            </div>
          </div>

          <!-- 4) Pegatina ⇢ Ranking -->
          <div>
            <h3 class="mb-6 mt-8 text-xl font-bold text-amber-300">Pegatina / Ranking</h3>
            <div class="flex flex-col items-start gap-4 md:flex-row md:items-center">
              <div class="relative h-28 w-28 overflow-hidden rounded-full ring-2 ring-amber-700/30">
                <img src="/vinyls/Base.png" alt="Vinilo base" class="absolute inset-0 w-full h-full object-cover" />
                <img src="/overlays/star.png" alt="Pegatina ranking" class="absolute inset-0 w-full h-full object-cover"/>
              </div>
              <p class="text-amber-300">
                El número dentro de la estrella indica el ranking del álbum la semana de lanzamiento.
              </p>
            </div>
          </div>
        </section>
      </div>
      <!-- <<< Leyendas / Referencias <<< -->


        <!-- Main Player Section -->
        <div class="grid grid-cols-1 gap-8 rounded-xl bg-gradient-to-br from-zinc-800 to-zinc-900 p-6 shadow-2xl lg:grid-cols-5">
          <!-- Record Player -->
          <div class="col-span-1 lg:col-span-3">
            <div class="relative rounded-lg bg-gradient-to-br from-zinc-700 to-zinc-900 p-6 shadow-xl">
              <div class="flex items-center justify-between">
                <div class="flex items-center gap-2">
                  <div class="h-3 w-3 rounded-full bg-amber-500 shadow-md"></div>
                  <div class="h-3 w-3 rounded-full shadow-md {isPlaying ? 'bg-emerald-500' : 'bg-zinc-400'}"></div>
                </div>
              </div>
  
              <!-- Turntable -->
              <div class="relative mt-4 aspect-square overflow-hidden rounded-full bg-zinc-900 shadow-inner">
  
                <!-- Record -->
                {#if selectedRecord}
                  <div
                    in:fly={{ y: -300, duration: 600 }}
                    out:fly={{ y: -300, duration: 300 }}
                    class="absolute inset-0 flex items-center justify-center"
                    style={`transform: rotate(${recordRotation}deg); transform-origin: center center;`}
                  >
                    <!--  Contenedor del vinilo: tamaño fijo + recorte circular  -->
                    <div class="relative w-124 aspect-square rounded-full bg-zinc-900 shadow-inner">
                      <!-- 1) Base de color según género -->
                      <img
                        src={vinylBase(selectedRecord) || "/placeholder.svg"}
                        alt="Vinilo base"
                        class="absolute inset-0 object-cover"
                      />

                      <!-- 2) Overlay de desgaste -->
                      <img
                        src={eraOverlay(selectedRecord) || "/placeholder.svg"}
                        alt="Desgaste"
                        class="absolute inset-0 object-cover"
                      />

                      <!-- 3) Overlay de surcos -->
                      <img
                        src={groovesOverlay(selectedRecord) || "/placeholder.svg"}
                        alt="Surcos"
                        class="absolute inset-0 object-cover"
                      />

                      <img src={starSticker() || "/placeholder.svg"} alt="Ranking star" class="absolute inset-0 object-cover" />
                      <span class="absolute inset-0 flex items-center justify-center text-2xl font-bold text-black top-62 left-59">
                        {selectedRecord.ranking > 100 ? '+100' : `#${selectedRecord.ranking}`}
                      </span>

                    </div>
                  </div>
                {/if}

  
                <!-- Center spindle -->
                <div class="absolute left-1/2 top-1/2 z-10 h-4 w-4 -translate-x-1/2 -translate-y-1/2 rounded-full bg-zinc-300 shadow-md"></div>
              </div>
  
              <!-- Tonearm -->
              <div class="absolute right-12 top-12 origin-bottom-right">
                <div
                  class="relative h-40 w-8"
                  style="transform: rotate({$tonearmRotation}deg);"
                >
                  <div class="absolute bottom-0 h-20 w-2 rounded-t-full bg-gradient-to-b from-zinc-300 to-zinc-400"></div>
                  <div
                    class="absolute bottom-0 right-0 h-4 w-8 rounded-r-full bg-gradient-to-r from-zinc-400 to-zinc-500"
                    style="transform: translateY(50%);"
                  ></div>
                  <div class="absolute bottom-0 left-0 h-8 w-8 rounded-full bg-gradient-to-br from-zinc-300 to-zinc-500 shadow-md"></div>
                </div>
              </div>
  
              <!-- Controls -->
              <div class="mt-6 flex items-center justify-between">
                <div class="flex gap-4">
                  <button
                    onclick={handlePlayPause}
                    disabled={currentRecordId === null}
                    class="flex h-12 w-12 items-center justify-center rounded-full shadow-md transition-all {
                      currentRecordId === null
                        ? 'bg-zinc-600 cursor-not-allowed opacity-50'
                        : 'bg-amber-700 hover:bg-amber-600 active:scale-95'
                    }"
                    aria-label="Play/Pause"
                  >
                    {#if isPlaying}
                      <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 9v6m4-6v6" />
                      </svg>
                    {:else}
                      <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14.752 11.168l-3.197-2.132A1 1 0 0010 9.87v4.263a1 1 0 001.555.832l3.197-2.132a1 1 0 000-1.664z" />
                      </svg>
                    {/if}
                  </button>
                  <button
                    onclick={handleStop}
                    disabled={currentRecordId === null}
                    class="flex h-12 w-12 items-center justify-center rounded-full shadow-md transition-all {
                      currentRecordId === null
                        ? 'bg-zinc-600 cursor-not-allowed opacity-50'
                        : 'bg-amber-700 hover:bg-amber-600 active:scale-95'
                    }"
                    aria-label="Stop"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                    </svg>
                  </button>
                </div>
  
                <div class="flex w-1/3 items-center gap-3">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-amber-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z" />
                  </svg>
                  <input
                    type="range"
                    min="0"
                    max="100"
                    bind:value={volume}
                    class="volume-slider h-2 w-full appearance-none rounded-full bg-zinc-700"
                    style="background-image: linear-gradient(to right, #b45309 {volume}%, #3f3f46 {volume}%);"
                  />
                </div>
              </div>
            </div>
          </div>

          <!-- Record Info Panel -->
          <div class="col-span-1 lg:col-span-2">
            {#if selectedRecord}
              <div class="h-[630px] overflow-y-auto rounded-lg bg-zinc-800/50 p-6 shadow-inner">
                <div class="flex flex-col">
                  <div in:fade={{ duration: 500 }}>
                    <h2 class="font-serif text-3xl font-bold text-amber-200">{selectedRecord.title}</h2>
                    <div class="mt-1 flex items-center">
                      <svg xmlns="http://www.w3.org/2000/svg" class="mr-2 h-4 w-4 text-amber-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19V6l12-3v13M9 19c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zm12-3c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zM9 10l12-3" />
                      </svg>
                      <p class="text-lg font-medium text-amber-300">{selectedRecord.artist}</p>
                    </div>
  
                    <div class="mt-4 grid grid-cols-2 gap-4">
                      <div class="flex items-center rounded-md bg-zinc-700/30 px-3 py-2">
                        <span class="text-sm text-amber-400">Año:</span>
                        <span class="ml-2 text-sm font-medium text-amber-300">{selectedRecord.year}</span>
                      </div>
                      <div class="flex items-center rounded-md bg-zinc-700/30 px-3 py-2">
                        <span class="text-sm text-amber-400">Género:</span>
                        <span class="ml-2 text-sm font-medium text-amber-300">{selectedRecord.genre}</span>
                      </div>
                    </div>
  
                    <div class="mt-4 rounded-md bg-zinc-700/20 p-4">
                      <p class="text-sm leading-relaxed text-amber-300/90">{selectedRecord.description}</p>
                    </div>
                  </div>
  
                  <div class="mt-6 flex-1 overflow-hidden">
                    <div class="flex items-center">
                      <svg xmlns="http://www.w3.org/2000/svg" class="mr-2 h-4 w-4 text-amber-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19V6l12-3v13M9 19c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zm12-3c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zM9 10l12-3" />
                      </svg>
                      <h3 class="font-serif text-lg font-medium text-amber-300">Tracks</h3>
                    </div>
  
                    <ul class="mt-4 space-y-2 overflow-y-auto pr-2">
                      {#each selectedRecord.tracks as track, index}
                        <li 
                          in:fly={{ x: -20, delay: index * 100, duration: 300 }}
                          class="flex items-center rounded-md p-3 transition-colors {isPlaying && index === 0 ? 'bg-amber-900/20' : 'hover:bg-zinc-700/20'}"
                        >
                          <div class="mr-3 flex h-7 w-7 items-center justify-center rounded-full bg-amber-800/40 text-xs font-medium text-amber-300">
                            {index + 1}
                          </div>
                          <span class="text-sm font-medium text-amber-200">{track}</span>
                          {#if isPlaying && index === 0}
                            <div class="ml-auto flex space-x-1">
                              {#each Array(3) as _, i}
                                <div 
                                  class="equalizer-bar bg-amber-500"
                                  style="width: 2px; height: {8 + Math.sin((Date.now() / (800 + i * 200)) % (2 * Math.PI)) * 4}px; animation-delay: {i * 0.2}s;"
                                ></div>
                              {/each}
                            </div>
                          {/if}
                        </li>
                      {/each}
                    </ul>
                  </div>
                </div>
              </div>
            {:else}
              <div class="flex h-full items-center justify-center rounded-lg bg-zinc-800/50 p-6 text-center">
                <div>
                  <h3 class="text-xl font-bold text-amber-300">No se seleccionó ningún álbum</h3>
                  <p class="mt-2 text-amber-400/80">Selecciona un álbum de la colección para empezar a escuchar</p>
                </div>
              </div>
            {/if}
          </div>
        </div>
  
        <!-- Record Showcase -->
        <div>
          <div class="relative rounded-xl border-2 border-amber-700/50 bg-zinc-800/30 p-6 shadow-lg">
            <div class="absolute left-4 top-1/2 z-10 -translate-y-1/2">
              <button
                onclick={handlePrevious}
                class="flex h-10 w-10 items-center justify-center rounded-full shadow-lg transition-all
                      {activeIndex === 0
                        ? 'bg-zinc-600 cursor-not-allowed opacity-50'
                        : 'bg-amber-700 text-white hover:bg-amber-600'}"
                disabled={activeIndex === 0}
                aria-label="Previous"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
                </svg>
              </button>
            </div>
  
            <div class="absolute right-4 top-1/2 z-10 -translate-y-1/2">
              <button
                onclick={handleNext}
                class="flex h-10 w-10 items-center justify-center rounded-full shadow-lg transition-all
                      {activeIndex >= records.length - visibleCount
                        ? 'bg-zinc-600 cursor-not-allowed opacity-50'
                        : 'bg-amber-700 text-white hover:bg-amber-600'}"
                disabled={activeIndex >= records.length - visibleCount}
                aria-label="Next"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
                </svg>
              </button>
            </div>
  
            <div class="mx-16 overflow-hidden">
              <div
                class="flex transition-transform duration-500 ease-out"
                style="transform: translateX(-{activeIndex * 25}%);"
              >
                {#each records as record, index}
                  <div class="w-1/4 flex-shrink-0 px-4 py-6">
                    <button
                      onclick={() => {
                        activeIndex = index;
                        handleRecordSelect(record.id);
                      }}
                      class="group relative aspect-square w-full cursor-pointer overflow-hidden rounded-lg shadow-xl transition-all duration-300 {
                        record.id === currentRecordId ? 'ring-4 ring-amber-500' : ''
                      }"
                      style="background-color: {record.coverColor}; border: none; padding: 0; text-align: left;"
                      aria-label={`Play ${record.title}`}
                    >
                      <!-- Album Cover -->
                      <!-- portada redonda de 96×96px centrada -->
                      <img
                        src={coverFile(record) || "/placeholder.svg"}
                        alt={`Portada de ${record.title}`}
                        class="absolute inset-0 m-auto h-48 w-48 object-cover "
                      />
  

                      <!-- Hover Effect -->
                      <div class="absolute inset-0 flex items-center justify-center bg-black/60 opacity-0 transition-opacity group-hover:opacity-100">
                        <div class="rounded-full bg-white/20 p-3">
                          {#if record.id === currentRecordId}
                            <span class="text-sm font-medium text-white">Now Playing</span>
                          {:else}
                            <span class="text-sm font-medium text-white">Play Album</span>
                          {/if}
                        </div>
                      </div>
                    </button>
                    <div class="mt-3 text-center">
                      <p class="text-sm font-medium text-amber-300">{record.title}</p>
                    </div>
                  </div>
                {/each}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</div>

<style>
  /* Custom styles for range input */
  .volume-slider {
    -webkit-appearance: none;
    appearance: none;
    height: 8px;
    border-radius: 4px;
    outline: none;
  }
  
  .volume-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #b45309; /* amber-700 */
    cursor: pointer;
    border: 2px solid #78350f; /* amber-900 */
  }
  
  .volume-slider::-moz-range-thumb {
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #b45309; /* amber-700 */
    cursor: pointer;
    border: 2px solid #78350f; /* amber-900 */
  }
  
  /* Animation for equalizer bars */
  .equalizer-bar {
    animation: equalize 1.2s ease-in-out infinite;
  }
  
  @keyframes equalize {
    0% { height: 2px; }
    50% { height: 12px; }
    100% { height: 2px; }
  }

  /* 1) Make sure the browser's default white never shows */
  :global(body) {
    margin: 0;
    padding: 0;
    background-color: #18181b; /* zinc-900 */
    font-family: "Courier New", monospace;
  }

  /* 2) Keep splash fixed at 100vh until user scrolls past it */
  .splash {
    position: relative;
    width: 100%;
    height: 100vh;
    background: #18181b; /* zinc-900 */
    display: flex;
    justify-content: center;
    align-items: center;
    overflow: hidden;
  }

  .typewriter {
    color: #fcd34d; /* amber-300 */
    font-family: "Courier New", monospace;
    font-size: clamp(2rem, 5vw, 5rem);
    white-space: nowrap;
    border-right: 2px solid rgba(255,255,255,0.75);
    animation: blink-cursor 0.7s steps(1) infinite;
  }

  @keyframes blink-cursor {
    0%, 100% { border-color: transparent; }
    50% { border-color: white; }
  }

  /* 3) Keep the main content exactly 100vh below, so scrolling reveals it */
  .main-content {
    margin-top: 0;
  }
</style>
