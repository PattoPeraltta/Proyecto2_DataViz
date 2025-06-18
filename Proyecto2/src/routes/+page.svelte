<script>
  import { onMount, tick } from 'svelte';
  import { fade, fly, scale } from 'svelte/transition';
  import { spring, tweened } from 'svelte/motion';
  import { cubicOut, linear } from 'svelte/easing';

  // Typewriter effect
  let fullText = "The Story of Music: Encoded in Vinyl";
  let displayedText = $state("");
  let typingIndex = 0;

  // Data and player state
  let records = $state([]);
  let audioElement = $state();
  let currentRecordId = $state(null);
  let isPlaying = $state(false);
  let volume = $state(75);
  let activeIndex = $state(0);
  let tonearmRotation = spring(0);
  let recordRotation = $state(0);
  let animationFrame;

  // Scroll sections
  let heroSection = $state();
  let industrySection = $state();
  let visualizationSection = $state();
  let playerSection = $state();
  let dashboardSection = $state();

  const visibleCount = 4;
  const selectedRecord = () => records.find(record => record.id === currentRecordId);

  onMount(() => {
    // Typewriter animation
    const interval = setInterval(() => {
      if (typingIndex < fullText.length) {
        displayedText += fullText[typingIndex];
        typingIndex += 1;
      } else {
        clearInterval(interval);
      }
    }, 100);

    // Load data
    loadRecords();
    
    // Start record animation
    animationFrame = requestAnimationFrame(animateRecord);
    
    return () => {
      clearInterval(interval);
      cancelAnimationFrame(animationFrame);
    };
  });

  async function loadRecords() {
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
  }

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

  function animateRecord() {
    if (isPlaying) {
      recordRotation += 1;
      if (recordRotation >= 360) recordRotation = 0;
    }
    animationFrame = requestAnimationFrame(animateRecord);
  }

  // Player functions
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

  function handlePlayPause() {
    if (currentRecordId === null || !audioElement) return;
    isPlaying = !isPlaying;
    isPlaying ? audioElement.play() : audioElement.pause();
  }

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

  function handlePrevious() {
    activeIndex = Math.max(activeIndex - 1, 0);
  }

  function handleNext() {
    activeIndex = Math.min(activeIndex + 1, records.length - visibleCount);
  }

  // Utility functions
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

  function normalize(str) {
    return str
      .toLowerCase()
      .replace(/\s+/g, "")
      .replace(/[^\w]/g, "");
  }

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

  const coverSrc = record => `/covers/${normalize(record.title)}_cover.png`;
  const vinylBase = record => `/vinyls/${record.genre}.png`;
  const eraOverlay = record => `/overlays/${getEra(record.year)}.png`;
  const groovesOverlay = record => `/overlays/grooves-${getGrooves(record.playsMillions)}.png`;
  const starSticker = () => `/overlays/star.png`;
  const coverFile = record => `/covers/${toCamelCase(record.title)}.png`;

  // Update tonearm position
  function updateTonearm() {
    if (currentRecordId !== null) {
      tonearmRotation.set(25);
    } else {
      tonearmRotation.set(0);
    }
  }
</script>

<svelte:window />
<audio bind:this={audioElement} onended={() => isPlaying = false}></audio>

<!-- Hero Section -->
<section bind:this={heroSection} class="hero-section">
  <div class="hero-content">
    <h1 class="typewriter">{displayedText}</h1>
    <p class="hero-subtitle" style="opacity: {displayedText.length === fullText.length ? 1 : 0}">
      A data visualization journey through decades of musical history
    </p>
  </div>
  <div class="scroll-indicator" style="opacity: {displayedText.length === fullText.length ? 1 : 0}">
    <div class="scroll-arrow"></div>
    <span>Scroll to explore</span>
  </div>
</section>

<!-- Industry Overview Section -->
<section bind:this={industrySection} class="industry-section">
  <div class="container">
    <div class="industry-content">
      <h2 class="section-title">The Evolution of Music</h2>
      
      <div class="industry-stats">
        <div class="stat-card">
          <div class="stat-number">2.7B</div>
          <div class="stat-label">Global music streams daily</div>
        </div>
        <div class="stat-card">
          <div class="stat-number">100M+</div>
          <div class="stat-label">Songs available digitally</div>
        </div>
        <div class="stat-card">
          <div class="stat-number">$26.2B</div>
          <div class="stat-label">Global music industry revenue (2023)</div>
        </div>
      </div>

      <div class="industry-narrative">
        <p>
          From the crackling warmth of vinyl records to the crystal clarity of digital streaming, 
          the music industry has undergone a remarkable transformation. Each era has left its mark, 
          not just in sound quality, but in how we discover, consume, and connect with music.
        </p>
        <p>
          The vinyl record, once thought obsolete, has experienced a renaissance. In 2023, vinyl 
          sales reached their highest point since 1987, proving that physical media still holds 
          a special place in our hearts and collections.
        </p>
      </div>

      <div class="timeline">
        <div class="timeline-item">
          <div class="timeline-year">1950s</div>
          <div class="timeline-content">Birth of Rock & Roll</div>
        </div>
        <div class="timeline-item">
          <div class="timeline-year">1980s</div>
          <div class="timeline-content">Digital Revolution Begins</div>
        </div>
        <div class="timeline-item">
          <div class="timeline-year">2000s</div>
          <div class="timeline-content">Streaming Era Emerges</div>
        </div>
        <div class="timeline-item">
          <div class="timeline-year">2020s</div>
          <div class="timeline-content">Vinyl Renaissance</div>
        </div>
      </div>
    </div>
  </div>
</section>

<!-- Visualization Explanation Section -->
<section bind:this={visualizationSection} class="visualization-section">
  <div class="container">
    <div class="viz-content">
      <h2 class="section-title">Decoding Musical Data</h2>
      
      <p class="viz-intro">
        Every vinyl record tells a story through visual elements. Our visualization system 
        encodes multiple data dimensions into the familiar form of a record, creating an 
        intuitive way to explore musical history and popularity.
      </p>

      <!-- Legend Section -->
      <div class="legend-grid">
        <!-- Colors / Genres -->
        <div class="legend-section">
          <h3 class="legend-title">Colors Represent Genres</h3>
          <div class="genre-grid">
            {#each ['Rock', 'Pop', 'Latino', 'Electronica', 'Jazz', 'HipHop'] as genre}
              <div class="genre-item">
                <div class="vinyl-preview">
                  <img src="/vinyls/{genre}.png" alt="{genre}" />
                </div>
                <span>{genre === 'HipHop' ? 'Hip-Hop' : genre === 'Electronica' ? 'Electronic' : genre}</span>
              </div>
            {/each}
          </div>
        </div>

        <!-- Era Wear -->
        <div class="legend-section">
          <h3 class="legend-title">Wear Patterns Show Musical Eras</h3>
          <div class="era-grid">
            <div class="era-item">
              <div class="vinyl-preview">
                <img src="/vinyls/Base.png" alt="2000-2025" />
              </div>
              <span>2000 – 2025<br/>Digital Age</span>
            </div>
            <div class="era-item">
              <div class="vinyl-preview">
                <img src="/vinyls/Base.png" alt="1950-2000" />
                <img src="/overlays/1950-2000.png" alt="Medium wear" />
              </div>
              <span>1950 – 2000<br/>Golden Era</span>
            </div>
            <div class="era-item">
              <div class="vinyl-preview">
                <img src="/vinyls/Base.png" alt="1900-1950" />
                <img src="/overlays/1900-1950.png" alt="Heavy wear" />
              </div>
              <span>1900 – 1950<br/>Early Recordings</span>
            </div>
          </div>
        </div>

        <!-- Grooves / Popularity -->
        <div class="legend-section">
          <h3 class="legend-title">Groove Density Indicates Popularity</h3>
          <div class="grooves-grid">
            {#each [1, 2, 3, 4] as grooves}
              <div class="groove-item">
                <div class="vinyl-preview">
                  <img src="/vinyls/Base.png" alt="Base" />
                  <img src="/overlays/grooves-{grooves}.png" alt="{grooves} grooves" />
                </div>
                <span>
                  {grooves === 1 ? '< 50M' : grooves === 2 ? '< 100M' : grooves === 3 ? '< 500M' : '> 500M'}
                  <br/>streams
                </span>
              </div>
            {/each}
          </div>
        </div>

        <!-- Rankings -->
        <div class="legend-section">
          <h3 class="legend-title">Star Rankings Show Chart Performance</h3>
          <div class="ranking-explanation">
            <div class="vinyl-preview large">
              <img src="/vinyls/Base.png" alt="Base" />
              <img src="/overlays/star.png" alt="Star ranking" />
              <span class="ranking-number">#1</span>
            </div>
            <p>The number within the star indicates the album's peak chart position during its release week.</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</section>

<!-- Interactive Player Section -->
<section bind:this={playerSection} class="player-section">
  <div class="container">
    <div class="player-content">
      <h2 class="section-title">Interactive Music Experience</h2>
      
      <!-- Main Player -->
      <div class="player-grid">
        <!-- Turntable -->
        <div class="turntable-container">
          <div class="turntable">
            <div class="turntable-controls">
              <div class="power-light"></div>
              <div class="status-light {isPlaying ? 'playing' : ''}"></div>
            </div>

            <div class="record-area">
              {#if selectedRecord()}
                <div
                  class="vinyl-record"
                  in:fly={{ y: -300, duration: 600 }}
                  out:fly={{ y: -300, duration: 300 }}
                  style="transform: rotate({recordRotation}deg)"
                >
                  <img src={vinylBase(selectedRecord()) || "/placeholder.svg"} alt="Vinyl base" class="vinyl-layer" />
                  <img src={eraOverlay(selectedRecord()) || "/placeholder.svg"} alt="Era overlay" class="vinyl-layer" />
                  <img src={groovesOverlay(selectedRecord()) || "/placeholder.svg"} alt="Grooves overlay" class="vinyl-layer" />
                  <img src={starSticker() || "/placeholder.svg"} alt="Ranking star" class="vinyl-layer" />
                  <div class="numerito"> 
                  <span class="ranking-display">
                    {selectedRecord().ranking > 100 ? '+100' : `#${selectedRecord().ranking}`}
                  </span>
                  </div>
                </div>
              {/if}
              <div class="spindle"></div>
            </div>

            <div class="tonearm" style="transform: rotate({$tonearmRotation}deg)">
              <div class="tonearm-body"></div>
              <div class="tonearm-head"></div>
              <div class="tonearm-base"></div>
            </div>

            <div class="player-controls">
              <button
                onclick={handlePlayPause}
                disabled={currentRecordId === null}
                class="control-btn play-btn"
                class:disabled={currentRecordId === null}
                aria-label="Play/Pause"
              >
                {#if isPlaying}
                  <svg viewBox="0 0 24 24" fill="currentColor">
                    <path d="M6 4h4v16H6V4zm8 0h4v16h-4V4z"/>
                  </svg>
                {:else}
                  <svg viewBox="0 0 24 24" fill="currentColor">
                    <path d="M8 5v14l11-7z"/>
                  </svg>
                {/if}
              </button>
              
              <button
                onclick={handleStop}
                disabled={currentRecordId === null}
                class="control-btn stop-btn"
                class:disabled={currentRecordId === null}
                aria-label="Stop"
              >
                <svg viewBox="0 0 24 24" fill="currentColor">
                  <path d="M6 6h12v12H6z"/>
                </svg>
              </button>

              <div class="volume-control">
                <svg viewBox="0 0 24 24" fill="currentColor" class="volume-icon">
                  <path d="M3 9v6h4l5 5V4L7 9H3zm13.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02zM14 3.23v2.06c2.89.86 5 3.54 5 6.71s-2.11 5.85-5 6.71v2.06c4.01-.91 7-4.49 7-8.77s-2.99-7.86-7-8.77z"/>
                </svg>
                <input
                  type="range"
                  min="0"
                  max="100"
                  bind:value={volume}
                  class="volume-slider"
                  aria-label="Volume Control"
                />
              </div>
            </div>
          </div>
        </div>

        <!-- Record Info -->
        <div class="record-info">
          {#if selectedRecord()}
            <div class="record-details" in:fade={{ duration: 500 }}>
              <h3 class="record-title">{selectedRecord().title}</h3>
              <p class="record-artist">{selectedRecord().artist}</p>
              
              <div class="record-meta">
                <div class="meta-item">
                  <span class="meta-label">Year:</span>
                  <span class="meta-value">{selectedRecord().year}</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">Genre:</span>
                  <span class="meta-value">{selectedRecord().genre}</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">Streams:</span>
                  <span class="meta-value">{selectedRecord().playsMillions}M</span>
                </div>
                <div class="meta-item">
                  <span class="meta-label">Peak Chart:</span>
                  <span class="meta-value">#{selectedRecord().ranking}</span>
                </div>
              </div>

              <div class="record-description">
                <p>{selectedRecord().description}</p>
              </div>

              <div class="tracklist">
                <h4>Tracklist</h4>
                <ul>
                  {#each selectedRecord().tracks as track, index}
                    <li class="track-item" class:playing={isPlaying && index === 0}>
                      <span class="track-number">{index + 1}</span>
                      <span class="track-name">{track}</span>
                      {#if isPlaying && index === 0}
                        <div class="equalizer">
                          {#each Array(3) as _, i}
                            <div class="eq-bar" style="animation-delay: {i * 0.2}s"></div>
                          {/each}
                        </div>
                      {/if}
                    </li>
                  {/each}
                </ul>
              </div>
            </div>
          {:else}
            <div class="no-selection">
              <h3>Select an Album</h3>
              <p>Choose a record from the collection below to start exploring</p>
            </div>
          {/if}
        </div>
      </div>

      <!-- Record Collection -->
      <div class="record-collection">
        <h3>Album Collection</h3>
        <div class="collection-carousel">
          <button
            onclick={handlePrevious}
            class="carousel-btn prev"
            disabled={activeIndex === 0}
            aria-label="Previous Album"
          >
            <svg viewBox="0 0 24 24" fill="currentColor">
              <path d="M15.41 7.41L14 6l-6 6 6 6 1.41-1.41L10.83 12z"/>
            </svg>
          </button>

          <div class="carousel-container">
            <div class="carousel-track" style="transform: translateX(-{activeIndex * 25}%)">
              {#each records as record, index}
                <div class="album-card">
                  <button
                    onclick={() => {
                      activeIndex = Math.max(0, Math.min(index - 1, records.length - visibleCount));
                      handleRecordSelect(record.id);
                    }}
                    class="album-button"
                    class:active={record.id === currentRecordId}
                    aria-label="Play {record.title}"
                  >
                    <img
                      src={coverFile(record) || "/placeholder.svg"}
                      alt="{record.title} cover"
                      class="album-cover"
                    />
                    <div class="album-overlay">
                      <span>{record.id === currentRecordId ? 'Now Playing' : 'Play Album'}</span>
                    </div>
                  </button>
                  <div class="album-info">
                    <p class="album-title">{record.title}</p>
                    <p class="album-artist">{record.artist}</p>
                  </div>
                </div>
              {/each}
            </div>
          </div>

          <button
            onclick={handleNext}
            class="carousel-btn next"
            disabled={activeIndex >= records.length - visibleCount}
            aria-label="Next Album"
          >
            <svg viewBox="0 0 24 24" fill="currentColor">
              <path d="M10 6L8.59 7.41 13.17 12l-4.58 4.59L10 18l6-6z"/>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </div>
</section>

<!-- Dashboard Section -->
<section bind:this={dashboardSection} class="dashboard-section">
  <div class="container">
    <h2 class="section-title">Music Industry Analytics</h2>
    
    <div class="dashboard-grid">
      <!-- Genre Distribution -->
      <div class="dashboard-card">
        <h3>Genre Distribution</h3>
        <div class="chart-placeholder">
          <p>Flourish Chart: Genre popularity over time</p>
          <div class="flourish-embed" data-src="visualisation/placeholder-1"></div>
        </div>
      </div>

      <!-- Streaming Trends -->
      <div class="dashboard-card">
        <h3>Streaming Evolution</h3>
        <div class="chart-placeholder">
          <p>Flourish Chart: Streaming vs Physical sales timeline</p>
          <div class="flourish-embed" data-src="visualisation/placeholder-2"></div>
        </div>
      </div>

      <!-- Regional Preferences -->
      <div class="dashboard-card">
        <h3>Global Music Preferences</h3>
        <div class="chart-placeholder">
          <p>Flourish Chart: World map of genre preferences</p>
          <div class="flourish-embed" data-src="visualisation/placeholder-3"></div>
        </div>
      </div>

      <!-- Artist Impact -->
      <div class="dashboard-card">
        <h3>Artist Influence Network</h3>
        <div class="chart-placeholder">
          <p>Flourish Chart: Network graph of artist collaborations</p>
          <div class="flourish-embed" data-src="visualisation/placeholder-4"></div>
        </div>
      </div>
    </div>

    <div class="insights-section">
      <h3>Key Insights</h3>
      <div class="insights-grid">
        <div class="insight-card">
          <div class="insight-icon">🎵</div>
          <h4>Genre Evolution</h4>
          <p>Electronic music has seen a 340% increase in streaming over the past decade, while rock maintains steady popularity across all age groups.</p>
        </div>
        <div class="insight-card">
          <div class="insight-icon">🌍</div>
          <h4>Global Reach</h4>
          <p>Latin music has become the fastest-growing genre globally, with reggaeton and Latin pop crossing cultural boundaries.</p>
        </div>
        <div class="insight-card">
          <div class="insight-icon">📈</div>
          <h4>Vinyl Revival</h4>
          <p>Vinyl sales have grown for 16 consecutive years, with Gen Z driving 43% of purchases despite growing up in the digital age.</p>
        </div>
        <div class="insight-card">
          <div class="insight-icon">🎧</div>
          <h4>Listening Habits</h4>
          <p>The average listener discovers 5 new artists per month through algorithmic recommendations, but 67% still value human curation.</p>
        </div>
      </div>
    </div>
  </div>
</section>

<!-- Conclusion Section -->
<section class="conclusion-section">
  <div class="container">
    <div class="conclusion-content">
      <h2>The Beat Goes On</h2>
      <p class="conclusion-text">
        Music remains one of humanity's most powerful forms of expression, constantly evolving 
        while honoring its roots. From the warm crackle of vinyl to the infinite possibilities 
        of digital streaming, each era has contributed to the rich tapestry of musical history.
      </p>
      <p class="conclusion-text">
        As we look to the future, emerging technologies like AI-generated music, spatial audio, 
        and virtual concerts promise to reshape how we create and experience music. Yet the 
        fundamental human need for rhythm, melody, and emotional connection remains unchanged.
      </p>
      <div class="conclusion-cta">
        <p>The story of music is still being written. What chapter will you add?</p>
      </div>
    </div>
  </div>
</section>

<!-- Footer -->
<footer class="footer">
  <div class="container">
    <div class="footer-content">
      <div class="footer-section">
        <h4>About This Visualization</h4>
        <p>An interactive exploration of music industry data, combining historical context with modern analytics to tell the story of musical evolution.</p>
      </div>
      <div class="footer-section">
        <h4>Data Sources</h4>
        <ul>
          <li>Recording Industry Association of America (RIAA)</li>
          <li>International Federation of the Phonographic Industry (IFPI)</li>
          <li>Spotify Global Charts</li>
          <li>Billboard Chart Archives</li>
        </ul>
      </div>
      <div class="footer-section">
        <h4>Methodology</h4>
        <p>This visualization encodes multiple data dimensions into familiar vinyl record aesthetics, making complex industry data accessible and engaging.</p>
      </div>
    </div>
    <div class="footer-bottom">
      <p>&copy; 2024 Music Data Visualization. Created with passion for music and data.</p>
    </div>
  </div>
</footer>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    background-color: #0a0a0a;
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
    color: #ffffff;
    overflow-x: hidden;
    scroll-behavior: smooth;
  }

 .numerito {
  position: absolute;
  top: 75%;
  left: 74%;
  transform: translate(-50%, -50%);
 }
  /* Hero Section */
  .hero-section {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background: linear-gradient(135deg, #0a0a0a 0%, #1a1a1a 100%);
    position: relative;
  }

  .hero-content {
    text-align: center;
    z-index: 2;
  }

  .typewriter {
    font-size: clamp(2rem, 6vw, 4rem);
    font-weight: 700;
    color: #fbbf24;
    font-family: 'Courier New', monospace;
    border-right: 3px solid rgba(251, 191, 36, 0.8);
    animation: blink-cursor 1s steps(1) infinite;
    white-space: nowrap;
  }

  .hero-subtitle {
    font-size: 1.25rem;
    color: #d1d5db;
    margin-top: 2rem;
    transition: opacity 1s ease-in-out;
  }

  .scroll-indicator {
    position: absolute;
    bottom: 2rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    color: #9ca3af;
    transition: opacity 1s ease-in-out;
  }

  .scroll-arrow {
    width: 24px;
    height: 24px;
    border-right: 2px solid #fbbf24;
    border-bottom: 2px solid #fbbf24;
    transform: rotate(45deg);
    animation: bounce 2s infinite;
    margin-bottom: 0.5rem;
  }

  @keyframes blink-cursor {
    0%, 50% { border-color: transparent; }
    51%, 100% { border-color: rgba(251, 191, 36, 0.8); }
  }

  @keyframes bounce {
    0%, 20%, 50%, 80%, 100% { transform: rotate(45deg) translateY(0); }
    40% { transform: rotate(45deg) translateY(-10px); }
    60% { transform: rotate(45deg) translateY(-5px); }
  }

  /* Common Section Styles */
  section {
    min-height: 100vh;
    padding: 4rem 0;
  }

  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 2rem;
  }

  .section-title {
    font-size: 3rem;
    font-weight: 700;
    color: #fbbf24;
    text-align: center;
    margin-bottom: 3rem;
  }

  /* Industry Section */
  .industry-section {
    background: linear-gradient(135deg, #1a1a1a 0%, #2d2d2d 100%);
  }

  .industry-stats {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
    margin-bottom: 3rem;
  }

  .stat-card {
    background: rgba(251, 191, 36, 0.1);
    border: 1px solid rgba(251, 191, 36, 0.3);
    border-radius: 1rem;
    padding: 2rem;
    text-align: center;
  }

  .stat-number {
    font-size: 2.5rem;
    font-weight: 700;
    color: #fbbf24;
    margin-bottom: 0.5rem;
  }

  .stat-label {
    color: #d1d5db;
    font-size: 1rem;
  }

  .industry-narrative {
    max-width: 800px;
    margin: 0 auto 3rem;
    font-size: 1.125rem;
    line-height: 1.8;
    color: #d1d5db;
  }

  .timeline {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 2rem;
    margin-top: 3rem;
  }

  .timeline-item {
    text-align: center;
    padding: 1.5rem;
    background: rgba(0, 0, 0, 0.3);
    border-radius: 0.5rem;
    border: 1px solid rgba(251, 191, 36, 0.2);
  }

  .timeline-year {
    font-size: 1.5rem;
    font-weight: 700;
    color: #fbbf24;
    margin-bottom: 0.5rem;
  }

  .timeline-content {
    color: #d1d5db;
  }

  /* Visualization Section */
  .visualization-section {
    background: linear-gradient(135deg, #2d2d2d 0%, #1a1a1a 100%);
  }

  .viz-intro {
    max-width: 800px;
    margin: 0 auto 3rem;
    font-size: 1.125rem;
    line-height: 1.8;
    color: #d1d5db;
    text-align: center;
  }

  .legend-grid {
    display: grid;
    gap: 3rem;
  }

  .legend-section {
    background: rgba(0, 0, 0, 0.3);
    border: 1px solid rgba(251, 191, 36, 0.3);
    border-radius: 1rem;
    padding: 2rem;
  }

  .legend-title {
    font-size: 1.5rem;
    font-weight: 600;
    color: #fbbf24;
    margin-bottom: 1.5rem;
    text-align: center;
  }

  .genre-grid, .era-grid, .grooves-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
    gap: 1.5rem;
  }

  .genre-item, .era-item, .groove-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.75rem;
  }

  .vinyl-preview {
    position: relative;
    width: 80px;
    height: 80px;
    border-radius: 50%;
    overflow: hidden;
    border: 2px solid rgba(251, 191, 36, 0.3);
  }

  .vinyl-preview.large {
    width: 120px;
    height: 120px;
  }

  .vinyl-preview img {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .ranking-explanation {
    display: flex;
    align-items: center;
    gap: 2rem;
    flex-wrap: wrap;
    justify-content: center;
  }

  .ranking-number {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 700;
    color: #000;
    font-size: 1.25rem;
  }

  /* Player Section */
  .player-section {
    background: linear-gradient(135deg, #0a0a0a 0%, #1a1a1a 100%);
  }

  .player-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 3rem;
    margin-bottom: 3rem;
  }

  .turntable-container {
    display: flex;
    justify-content: center;
  }

  .turntable {
    background: linear-gradient(135deg, #374151 0%, #1f2937 100%);
    border-radius: 1rem;
    padding: 2rem;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
    position: relative;
    width: 100%;
    max-width: 500px;
  }

  .turntable-controls {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }

  .power-light {
    width: 12px;
    height: 12px;
    border-radius: 50%;
    background: #fbbf24;
    box-shadow: 0 0 10px rgba(251, 191, 36, 0.5);
  }

  .status-light {
    width: 12px;
    height: 12px;
    border-radius: 50%;
    background: #6b7280;
    transition: all 0.3s ease;
  }

  .status-light.playing {
    background: #10b981;
    box-shadow: 0 0 10px rgba(16, 185, 129, 0.5);
  }

  .record-area {
    position: relative;
    aspect-ratio: 1;
    background: #111827;
    border-radius: 50%;
    margin: 2rem 0;
    overflow: hidden;
    box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.5);
  }

  .vinyl-record {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .vinyl-layer {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .ranking-display {
    font-weight: 700;
    color: #000;
    font-size: 1.5rem;
  }

  .spindle {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 16px;
    height: 16px;
    background: #d1d5db;
    border-radius: 50%;
    z-index: 10;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  }

  .tonearm {
    position: absolute;
    top: 3rem;
    right: 3rem;
    transform-origin: bottom right;
    transition: transform 0.8s cubic-bezier(0.4, 0, 0.2, 1);
  }

  .tonearm-body {
    width: 8px;
    height: 120px;
    background: linear-gradient(to bottom, #d1d5db, #9ca3af);
    border-radius: 4px 4px 0 0;
  }

  .tonearm-head {
    position: absolute;
    bottom: -8px;
    right: -12px;
    width: 32px;
    height: 16px;
    background: linear-gradient(to right, #9ca3af, #6b7280);
    border-radius: 0 8px 8px 0;
  }

  .tonearm-base {
    position: absolute;
    bottom: -16px;
    left: -12px;
    width: 32px;
    height: 32px;
    background: linear-gradient(135deg, #d1d5db, #6b7280);
    border-radius: 50%;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  }

  .player-controls {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
  }

  .control-btn {
    width: 48px;
    height: 48px;
    border-radius: 50%;
    border: none;
    background: #fbbf24;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: all 0.2s ease;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  }

  .control-btn:hover:not(.disabled) {
    background: #f59e0b;
    transform: scale(1.05);
  }

  .control-btn:active:not(.disabled) {
    transform: scale(0.95);
  }

  .control-btn.disabled {
    background: #6b7280;
    cursor: not-allowed;
    opacity: 0.5;
  }

  .control-btn svg {
    width: 24px;
    height: 24px;
  }

  .volume-control {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    flex: 1;
    max-width: 200px;
  }

  .volume-icon {
    width: 20px;
    height: 20px;
    color: #fbbf24;
  }

  .volume-slider {
    flex: 1;
    height: 8px;
    border-radius: 4px;
    background: #374151;
    outline: none;
    cursor: pointer;
    -webkit-appearance: none;
    appearance: none;
  }

  .volume-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #fbbf24;
    cursor: pointer;
    border: 2px solid #f59e0b;
  }

  .volume-slider::-moz-range-thumb {
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #fbbf24;
    cursor: pointer;
    border: 2px solid #f59e0b;
  }

  /* Record Info */
  .record-info {
    background: rgba(0, 0, 0, 0.3);
    border: 1px solid rgba(251, 191, 36, 0.3);
    border-radius: 1rem;
    padding: 2rem;
    height: fit-content;
  }

  .record-title {
    font-size: 2rem;
    font-weight: 700;
    color: #fbbf24;
    margin-bottom: 0.5rem;
  }

  .record-artist {
    font-size: 1.25rem;
    color: #d1d5db;
    margin-bottom: 1.5rem;
  }

  .record-meta {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    margin-bottom: 1.5rem;
  }

  .meta-item {
    background: rgba(251, 191, 36, 0.1);
    padding: 0.75rem;
    border-radius: 0.5rem;
    display: flex;
    justify-content: space-between;
  }

  .meta-label {
    color: #fbbf24;
    font-weight: 500;
  }

  .meta-value {
    color: #d1d5db;
    font-weight: 600;
  }

  .record-description {
    background: rgba(0, 0, 0, 0.2);
    padding: 1rem;
    border-radius: 0.5rem;
    margin-bottom: 1.5rem;
  }

  .record-description p {
    color: #d1d5db;
    line-height: 1.6;
    margin: 0;
  }

  .tracklist h4 {
    color: #fbbf24;
    margin-bottom: 1rem;
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .tracklist ul {
    list-style: none;
    padding: 0;
    margin: 0;
    max-height: 200px;
    overflow-y: auto;
  }

  .track-item {
    display: flex;
    align-items: center;
    padding: 0.75rem;
    border-radius: 0.5rem;
    margin-bottom: 0.5rem;
    transition: background-color 0.2s ease;
  }

  .track-item:hover {
    background: rgba(251, 191, 36, 0.1);
  }

  .track-item.playing {
    background: rgba(251, 191, 36, 0.2);
  }

  .track-number {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 28px;
    height: 28px;
    background: rgba(251, 191, 36, 0.3);
    border-radius: 50%;
    color: #fbbf24;
    font-weight: 600;
    font-size: 0.875rem;
    margin-right: 0.75rem;
  }

  .track-name {
    flex: 1;
    color: #d1d5db;
    font-weight: 500;
  }

  .equalizer {
    display: flex;
    gap: 2px;
    align-items: end;
    height: 16px;
  }

  .eq-bar {
    width: 3px;
    background: #fbbf24;
    border-radius: 1px;
    animation: equalize 1.2s ease-in-out infinite;
  }

  @keyframes equalize {
    0%, 100% { height: 4px; }
    50% { height: 16px; }
  }

  .no-selection {
    text-align: center;
    padding: 3rem 1rem;
    color: #9ca3af;
  }

  .no-selection h3 {
    color: #fbbf24;
    margin-bottom: 1rem;
  }

  /* Record Collection */
  .record-collection {
    margin-top: 3rem;
  }

  .record-collection h3 {
    color: #fbbf24;
    font-size: 1.5rem;
    margin-bottom: 1.5rem;
    text-align: center;
  }

  .collection-carousel {
    position: relative;
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .carousel-btn {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    border: none;
    background: #fbbf24;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: all 0.2s ease;
    flex-shrink: 0;
    z-index: 10;
  }

  .carousel-btn:hover:not(:disabled) {
    background: #f59e0b;
    transform: scale(1.1);
  }

  .carousel-btn:disabled {
    background: #6b7280;
    cursor: not-allowed;
    opacity: 0.5;
  }

  .carousel-btn svg {
    width: 20px;
    height: 20px;
  }

  .carousel-container {
    flex: 1;
    overflow: hidden;
    border-radius: 1rem;
  }

  .carousel-track {
    display: flex;
    transition: transform 0.5s ease;
  }

  .album-card {
    width: 25%;
    flex-shrink: 0;
    padding: 1rem;
  }

  .album-button {
    position: relative;
    width: 100%;
    aspect-ratio: 1;
    border: none;
    border-radius: 0.5rem;
    overflow: hidden;
    cursor: pointer;
    transition: all 0.3s ease;
    background: none;
    padding: 0;
  }

  .album-button.active {
    box-shadow: 0 0 0 4px #fbbf24;
  }

  .album-button:hover {
    transform: scale(1.05);
  }

  .album-cover {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .album-overlay {
    position: absolute;
    inset: 0;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.3s ease;
    color: white;
    font-weight: 600;
  }

  .album-button:hover .album-overlay {
    opacity: 1;
  }

  .album-info {
    text-align: center;
    margin-top: 0.75rem;
  }

  .album-title {
    color: #fbbf24;
    font-weight: 600;
    font-size: 0.875rem;
    margin: 0 0 0.25rem 0;
  }

  .album-artist {
    color: #9ca3af;
    font-size: 0.75rem;
    margin: 0;
  }

  /* Dashboard Section */
  .dashboard-section {
    background: linear-gradient(135deg, #1a1a1a 0%, #2d2d2d 100%);
  }

  .dashboard-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
    gap: 2rem;
    margin-bottom: 3rem;
  }

  .dashboard-card {
    background: rgba(0, 0, 0, 0.3);
    border: 1px solid rgba(251, 191, 36, 0.3);
    border-radius: 1rem;
    padding: 2rem;
  }

  .dashboard-card h3 {
    color: #fbbf24;
    font-size: 1.25rem;
    margin-bottom: 1rem;
  }

  .chart-placeholder {
    height: 300px;
    background: rgba(251, 191, 36, 0.1);
    border-radius: 0.5rem;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #9ca3af;
    text-align: center;
  }

  .insights-section {
    margin-top: 3rem;
  }

  .insights-section h3 {
    color: #fbbf24;
    font-size: 2rem;
    text-align: center;
    margin-bottom: 2rem;
  }

  .insights-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
  }

  .insight-card {
    background: rgba(251, 191, 36, 0.1);
    border: 1px solid rgba(251, 191, 36, 0.3);
    border-radius: 1rem;
    padding: 2rem;
    text-align: center;
  }

  .insight-icon {
    font-size: 3rem;
    margin-bottom: 1rem;
  }

  .insight-card h4 {
    color: #fbbf24;
    font-size: 1.25rem;
    margin-bottom: 1rem;
  }

  .insight-card p {
    color: #d1d5db;
    line-height: 1.6;
  }

  /* Conclusion Section */
  .conclusion-section {
    background: linear-gradient(135deg, #0a0a0a 0%, #1a1a1a 100%);
    text-align: center;
  }

  .conclusion-content {
    max-width: 800px;
    margin: 0 auto;
  }

  .conclusion-content h2 {
    font-size: 3rem;
    color: #fbbf24;
    margin-bottom: 2rem;
  }

  .conclusion-text {
    font-size: 1.125rem;
    line-height: 1.8;
    color: #d1d5db;
    margin-bottom: 2rem;
  }

  .conclusion-cta {
    background: rgba(251, 191, 36, 0.1);
    border: 1px solid rgba(251, 191, 36, 0.3);
    border-radius: 1rem;
    padding: 2rem;
    margin-top: 3rem;
  }

  .conclusion-cta p {
    font-size: 1.25rem;
    color: #fbbf24;
    font-weight: 600;
    margin: 0;
  }

  /* Footer */
  .footer {
    background: #000;
    border-top: 1px solid rgba(251, 191, 36, 0.3);
    padding: 3rem 0 1rem;
  }

  .footer-content {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
    margin-bottom: 2rem;
  }

  .footer-section h4 {
    color: #fbbf24;
    font-size: 1.125rem;
    margin-bottom: 1rem;
  }

  .footer-section p {
    color: #9ca3af;
    line-height: 1.6;
  }

  .footer-section ul {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .footer-section li {
    color: #9ca3af;
    margin-bottom: 0.5rem;
  }

  .footer-bottom {
    border-top: 1px solid rgba(251, 191, 36, 0.2);
    padding-top: 2rem;
    text-align: center;
  }

  .footer-bottom p {
    color: #6b7280;
    margin: 0;
  }

  /* Responsive Design */
  @media (max-width: 768px) {
    .player-grid {
      grid-template-columns: 1fr;
    }

    .section-title {
      font-size: 2rem;
    }

    .typewriter {
      font-size: 2rem;
    }

    .hero-subtitle {
      font-size: 1rem;
    }

    .record-meta {
      grid-template-columns: 1fr;
    }

    .dashboard-grid {
      grid-template-columns: 1fr;
    }

    .carousel-container {
      margin: 0 1rem;
    }

    .album-card {
      width: 50%;
    }
  }

  @media (max-width: 480px) {
    .container {
      padding: 0 1rem;
    }

    .turntable {
      padding: 1rem;
    }

    .record-info {
      padding: 1rem;
    }

    .album-card {
      width: 100%;
    }
  }
</style>
