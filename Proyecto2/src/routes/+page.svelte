<script>
  import { onMount, tick } from 'svelte';
  import { fade, fly, scale } from 'svelte/transition';
  import { spring, tweened } from 'svelte/motion';
  import { cubicOut, linear } from 'svelte/easing';

  let records = [];
  
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
    let currentRecordId = null;
    let isPlaying = false;
    let volume = 75;
    $: if (audioElement) audioElement.volume = volume / 100;
    let activeIndex = 0;
    let tonearmRotation = spring(0);
    let recordRotation = 0;
    let animationFrame;
    let selectedRecord;
    let audioElement;
    const visibleCount = 4;
  
    // Derived state
    $: selectedRecord = records.find(record => record.id === currentRecordId);
    
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
    $: {
      if (currentRecordId !== null) {
        tonearmRotation.set(25);
      } else {
        tonearmRotation.set(0);
      }
    }
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
      if (plays <  50) return 1;
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
  <audio bind:this={audioElement} on:ended={() => isPlaying = false}></audio>
  <main class="min-h-screen bg-gradient-to-b from-amber-950 to-stone-950 p-4 md:p-8">
    <div class="mx-auto max-w-6xl">
      <h1 class="mb-8 text-center font-serif text-5xl font-bold tracking-tight text-amber-100">Vinyl Vibes</h1>
      
      <div class="grid gap-12">
      <!-- >>> Leyendas / Referencias >>> -->
      <div class="mb-8 bg-amber-800/20 rounded-xl p-6">
        <section class="space-y-8">
          <!-- 1) Colores ⇢ Géneros -->
          <div>
            <h3 class="text-amber-200 font-semibold mb-2">Colores / Géneros</h3>
            <div class="flex gap-8">
              <div class="flex items-center gap-2">
                <img src="/vinyls/Rock.png"      alt="Rock"      class="w-24 h-24"/>
                <span class="text-amber-200">Rock</span>
              </div>
              <div class="flex items-center gap-2">
                <img src="/vinyls/Pop.png"       alt="Pop"       class="w-24 h-24"/>
                <span class="text-amber-200">Pop</span>
              </div>
              <div class="flex items-center gap-2">
                <img src="/vinyls/Latino.png"    alt="Latino"    class="w-24 h-24"/>
                <span class="text-amber-200">Latino</span>
              </div>
              <div class="flex items-center gap-2">
                <img src="/vinyls/Electronica.png" alt="Electrónica" class="w-24 h-24"/>
                <span class="text-amber-200">Electrónica</span>
              </div>
              <div class="flex items-center gap-2">
                <img src="/vinyls/Jazz.png"      alt="Jazz"      class="w-24 h-24"/>
                <span class="text-amber-200">Jazz</span>
              </div>
              <div class="flex items-center gap-2">
                <img src="/vinyls/HipHop.png"    alt="Hip-hop/Rap" class="w-24 h-24"/>
                <span class="text-amber-200">Hip-hop/Rap</span>
              </div>
            </div>
          </div>

            <!-- Desgaste ⇢ Épocas -->
            <div>
              <h3 class="text-amber-200 font-semibold mb-2">Desgaste / Épocas</h3>
              <div class="flex gap-8 items-center">
                <!-- 2000–2025 -->
                <div class="flex flex-col items-center gap-1">
                  <div class="relative w-24 h-24">
                    <img src="/vinyls/Base.png" alt="2000–2025" class="absolute inset-0 w-full h-full object-cover rounded-full"/>
                  </div>
                  <span class="text-amber-200 text-xs">2000 – 2025</span>
                </div>
                <!-- 1950–2000 -->
                <div class="flex flex-col items-center gap-1">
                  <div class="relative w-24 h-24">
                    <img src="/vinyls/Base.png"        alt="1950–2000" class="absolute inset-0 w-full h-full object-cover rounded-full"/>
                    <img src="/overlays/1950-2000.png"   alt="Desgaste medio" class="absolute inset-0 w-full h-full object-cover rounded-full"/>
                  </div>
                  <span class="text-amber-200 text-xs">1950 – 2000</span>
                </div>
                <!-- 1900–1950 -->
                <div class="flex flex-col items-center gap-1">
                  <div class="relative w-24 h-24">
                    <img src="/vinyls/Base.png"        alt="1900–1950" class="absolute inset-0 w-full h-full object-cover rounded-full"/>
                    <img src="/overlays/1900-1950.png"   alt="Desgaste fuerte" class="absolute inset-0 w-full h-full object-cover rounded-full"/>
                  </div>
                  <span class="text-amber-200 text-xs">1900 – 1950</span>
                </div>
              </div>
            
            <!-- Surcos ⇢ Millones de reproducciones -->
            <div>
              <h3 class="text-amber-200 font-semibold mb-2 mt-10">Surcos / Millones de reproducciones</h3>
              <div class="flex gap-8 items-center">
                {#each [1,2,3,4] as grooves}
                  <div class="flex flex-col items-center gap-1">
                    <div class="relative w-24 h-24">
                      <img src="/vinyls/Base.png" class="absolute inset-0 w-full h-full object-cover rounded-full" alt="{grooves} surco(s)"/>
                      <img src={`/overlays/grooves-${grooves}.png`} class="absolute inset-0 w-full h-full object-cover rounded-full" alt="{grooves} surco(s) overlay"/>
                    </div>
                    <span class="text-amber-200 text-xs">
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
            <h3 class="text-amber-200 font-semibold mb-2">Pegatina / Ranking</h3>
            <div class="flex items-center gap-4">
              <div class="relative w-24 h-24">
                <img src="/vinyls/Base.png" alt="Vinilo base" class="absolute inset-0 w-full h-full object-cover rounded-full" />
                <img src="/overlays/star.png" alt="Pegatina ranking" class="absolute inset-0 w-full h-full object-cover"/>
                <span class="absolute inset-0 flex items-center justify-center text-xs font-bold text-black top-12 left-11.5">#</span>
              </div>
              <p class="text-amber-200">
                El número dentro de la estrella indica el ranking del álbum la semana de lanzamiento.
              </p>
            </div>
          </div>
        </section>
      </div>
      <!-- <<< Leyendas / Referencias <<< -->


        <!-- Main Player Section -->
        <div class="grid grid-cols-1 gap-8 rounded-xl bg-gradient-to-br from-amber-900/80 to-amber-950/80 p-6 shadow-2xl backdrop-blur-sm lg:grid-cols-5">
          <!-- Record Player -->
          <div class="col-span-1 lg:col-span-3">
            <div class="relative rounded-lg bg-gradient-to-br from-amber-800 to-amber-950 p-6 shadow-xl">
              <div class="flex items-center justify-between">
                <div class="flex items-center gap-2">
                  <div class="h-3 w-3 rounded-full bg-amber-500 shadow-md"></div>
                  <div class="h-3 w-3 rounded-full shadow-md {isPlaying ? 'bg-green-500' : 'bg-gray-400'}"></div>
                </div>
              </div>
  
              <!-- Turntable -->
              <div class="relative mt-4 aspect-square overflow-hidden rounded-full bg-neutral-900 shadow-inner">
  
                <!-- Record -->
                {#if selectedRecord}
                  <div
                    in:fly={{ y: -300, duration: 600 }}
                    out:fly={{ y: -300, duration: 300 }}
                    class="absolute inset-0 flex items-center justify-center"
                    style={`transform: rotate(${recordRotation}deg); transform-origin: center center;`}
                  >
                    <!--  Contenedor del vinilo: tamaño fijo + recorte circular  -->
                    <div class="relative w-124 aspect-square rounded-full bg-neutral-900 shadow-inner">
                      <!-- 1) Base de color según género -->
                      <img
                        src={vinylBase(selectedRecord)}
                        alt="Vinilo base"
                        class="absolute inset-0 object-cover"
                      />

                      <!-- 2) Overlay de desgaste -->
                      <img
                        src={eraOverlay(selectedRecord)}
                        alt="Desgaste"
                        class="absolute inset-0 object-cover"
                      />

                      <!-- 3) Overlay de surcos -->
                      <img
                        src={groovesOverlay(selectedRecord)}
                        alt="Surcos"
                        class="absolute inset-0 object-cover"
                      />

                      <img src={starSticker()} alt="Ranking star" class="absolute inset-0 object-cover" />
                      <span class="absolute inset-0 flex items-center justify-center text-2xl font-bold text-black top-62 left-59">
                        {selectedRecord.ranking > 100 ? '+100' : `#${selectedRecord.ranking}`}
                      </span>

                    </div>
                  </div>
                {/if}

  
                <!-- Center spindle -->
                <div class="absolute left-1/2 top-1/2 z-10 h-4 w-4 -translate-x-1/2 -translate-y-1/2 rounded-full bg-gray-300 shadow-md"></div>
              </div>
  
              <!-- Tonearm -->
              <div class="absolute right-12 top-12 origin-bottom-right">
                <div
                  class="relative h-40 w-8"
                  style="transform: rotate({$tonearmRotation}deg);"
                >
                  <div class="absolute bottom-0 h-20 w-2 rounded-t-full bg-gradient-to-b from-gray-300 to-gray-400"></div>
                  <div
                    class="absolute bottom-0 right-0 h-4 w-8 rounded-r-full bg-gradient-to-r from-gray-400 to-gray-500"
                    style="transform: translateY(50%);"
                  ></div>
                  <div class="absolute bottom-0 left-0 h-8 w-8 rounded-full bg-gradient-to-br from-gray-300 to-gray-500 shadow-md"></div>
                </div>
              </div>
  
              <!-- Controls -->
              <div class="mt-6 flex items-center justify-between">
                <div class="flex gap-4">
                  <button
                    on:click={handlePlayPause}
                    disabled={currentRecordId === null}
                    class="flex h-12 w-12 items-center justify-center rounded-full shadow-md transition-all {
                      currentRecordId === null
                        ? 'bg-gray-600 cursor-not-allowed opacity-50'
                        : 'bg-amber-600 hover:bg-amber-500 active:scale-95'
                    }"
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
                    on:click={handleStop}
                    disabled={currentRecordId === null}
                    class="flex h-12 w-12 items-center justify-center rounded-full shadow-md transition-all {
                      currentRecordId === null
                        ? 'bg-gray-600 cursor-not-allowed opacity-50'
                        : 'bg-amber-600 hover:bg-amber-500 active:scale-95'
                    }"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                    </svg>
                  </button>
                </div>
  
                <div class="flex w-1/3 items-center gap-3">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-amber-200" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z" />
                  </svg>
                  <input
                    type="range"
                    min="0"
                    max="100"
                    bind:value={volume}
                    class="h-2 w-full appearance-none rounded-full bg-amber-800"
                    style="background-image: linear-gradient(to right, #f59e0b {volume}%, #78350f {volume}%);"
                  />
                </div>
              </div>
            </div>
          </div>

          <!-- Record Info Panel -->
          <div class="col-span-1 lg:col-span-2">
            {#if selectedRecord}
              <div class="h-[630px] overflow-y-auto rounded-lg bg-amber-950/50 p-6">
                <div class="flex flex-col">
                  <div in:fade={{ duration: 500 }}>
                    <h2 class="font-serif text-3xl font-bold text-amber-100">{selectedRecord.title}</h2>
                    <div class="mt-1 flex items-center">
                      <svg xmlns="http://www.w3.org/2000/svg" class="mr-2 h-4 w-4 text-amber-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19V6l12-3v13M9 19c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zm12-3c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zM9 10l12-3" />
                      </svg>
                      <p class="text-lg text-amber-200">{selectedRecord.artist}</p>
                    </div>
  
                    <div class="mt-4 grid grid-cols-2 gap-2">
                      <div class="flex items-center">
                        <svg xmlns="http://www.w3.org/2000/svg" class="mr-2 h-4 w-4 text-amber-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
                        </svg>
                        <p class="text-sm text-amber-300">{selectedRecord.year}</p>
                      </div>
                      <div class="flex items-center">
                        <svg xmlns="http://www.w3.org/2000/svg" class="mr-2 h-4 w-4 text-amber-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-7 7a2 2 0 01-2.828 0l-7-7A1.994 1.994 0 013 12V7a4 4 0 014-4z" />
                        </svg>
                        <p class="text-sm text-amber-300">{selectedRecord.genre}</p>
                      </div>
                    </div>
  
                    <div class="mt-4">
                      <p class="text-sm text-amber-300/80">{selectedRecord.description}</p>
                    </div>
                  </div>
  
                  <div class="mt-6 flex-1 overflow-hidden">
                    <div class="flex items-center">
                      <svg xmlns="http://www.w3.org/2000/svg" class="mr-2 h-4 w-4 text-amber-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19V6l12-3v13M9 19c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zm12-3c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zM9 10l12-3" />
                      </svg>
                      <h3 class="font-medium text-amber-200">Tracks</h3>
                    </div>
  
                    <ul class="mt-2 space-y-1 overflow-y-auto pr-2">
                      {#each selectedRecord.tracks as track, index}
                        <li 
                          in:fly={{ x: -20, delay: index * 100, duration: 300 }}
                          class="flex items-center rounded-md p-2 {isPlaying && index === 0 ? 'bg-amber-900/30' : ''}"
                        >
                          <div class="mr-3 flex h-6 w-6 items-center justify-center rounded-full bg-amber-900/50 text-xs text-amber-300">
                            {index + 1}
                          </div>
                          <span class="text-sm text-amber-200">{track}</span>
                          {#if isPlaying && index === 0}
                            <div class="ml-auto flex space-x-1">
                              {#each Array(3) as _, i}
                                <div 
                                  class="equalizer-bar bg-amber-400"
                                  style="width: 2px; height: {2 + Math.sin((Date.now() / (800 + i * 200)) % (2 * Math.PI)) * 3}px"
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
              <div class="flex h-full items-center justify-center rounded-lg bg-amber-950/50 p-6 text-center">
                <div>
                  <h3 class="font-serif text-xl font-bold text-amber-200">No se selecciono ningun album</h3>
                  <p class="mt-2 text-amber-300/80">Seleccióna un album de la colección para empezar a escuchar</p>
                </div>
              </div>
            {/if}
          </div>
        </div>
  
        <!-- Record Showcase -->
        <div>
          <h2 class="mb-4 font-serif text-3xl font-bold text-amber-200">Tu colección</h2>
          <div class="relative rounded-xl bg-amber-950/30 p-6">
            <div class="absolute left-4 top-1/2 z-10 -translate-y-1/2">
              <button
                on:click={handlePrevious}
                class="flex h-10 w-10 items-center justify-center rounded-full shadow-lg transition-all
                      {activeIndex === 0
                        ? 'bg-gray-600 cursor-not-allowed opacity-50'
                        : 'bg-amber-800 text-white hover:bg-amber-700'}"
                disabled={activeIndex === 0}
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
                </svg>
              </button>
            </div>
  
            <div class="absolute right-4 top-1/2 z-10 -translate-y-1/2">
              <button
                on:click={handleNext}
                class="flex h-10 w-10 items-center justify-center rounded-full shadow-lg transition-all
                      {activeIndex >= records.length - visibleCount
                        ? 'bg-gray-600 cursor-not-allowed opacity-50'
                        : 'bg-amber-800 text-white hover:bg-amber-700'}"
                disabled={activeIndex >= records.length - visibleCount}
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
                      on:click={() => {
                        activeIndex = index;
                        handleRecordSelect(record.id);
                      }}
                      class="group relative aspect-square w-full cursor-pointer overflow-hidden rounded-lg shadow-xl transition-all duration-300 {
                        record.id === currentRecordId ? 'ring-4 ring-amber-400' : ''
                      }"
                      style="background-color: {record.coverColor}; border: none; padding: 0; text-align: left;"
                    >
                      <!-- Album Cover -->
                      <!-- portada redonda de 96×96px centrada -->
                      <img
                      src= {coverFile(record)}
                      alt={`Portada de ${coverFile(record)}`}
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
                      <p class="text-sm font-medium text-amber-200">{record.title}</p>
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
  
  <style>
    /* Custom styles for range input */
    input[type="range"] {
      -webkit-appearance: none;
      appearance: none;
      height: 8px;
      border-radius: 4px;
      outline: none;
    }
  
    input[type="range"]::-webkit-slider-thumb {
      -webkit-appearance: none;
      appearance: none;
      width: 16px;
      height: 16px;
      border-radius: 50%;
      background: #f59e0b;
      cursor: pointer;
      border: 2px solid #78350f;
    }
  
    input[type="range"]::-moz-range-thumb {
      width: 16px;
      height: 16px;
      border-radius: 50%;
      background: #f59e0b;
      cursor: pointer;
      border: 2px solid #78350f;
    }
  
    /* Animation for equalizer bars */
    .equalizer-bar {
      animation: equalize 1.2s ease-in-out infinite;
      animation-delay: calc(var(--i, 0) * 0.2s);
    }
  
    @keyframes equalize {
      0% { height: 2px; }
      50% { height: 8px; }
      100% { height: 2px; }
    }
  </style>