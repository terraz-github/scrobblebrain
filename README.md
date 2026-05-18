<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Scrobble Brain — Last.fm Guess Game</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;700;800&family=DM+Mono:ital,wght@0,400;0,500;1,400&display=swap" rel="stylesheet">
<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --bg: #0e0c0a;
  --surface: #1a1714;
  --surface2: #242019;
  --border: #3a342c;
  --text: #f0ebe3;
  --muted: #7a6f62;
  --accent: #e8a44a;
  --green: #6ab87a;
  --red: #e05555;
  --r: 12px;
}

body {
  font-family: 'Syne', sans-serif;
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 2rem 1rem 4rem;
}

body::before {
  content: '';
  position: fixed;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
  pointer-events: none;
  z-index: 0;
  opacity: 0.6;
}

* { position: relative; z-index: 1; }

header { text-align: center; margin-bottom: 2.5rem; }

.logo {
  font-size: clamp(2rem, 5vw, 3.2rem);
  font-weight: 800;
  letter-spacing: -0.03em;
  line-height: 1;
  color: var(--accent);
}
.logo span { color: var(--text); }

.tagline {
  font-family: 'DM Mono', monospace;
  font-size: 0.75rem;
  color: var(--muted);
  margin-top: 0.4rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

#setup { width: 100%; max-width: 480px; }

.panel {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--r);
  padding: 1.75rem;
}

.field { margin-bottom: 1.25rem; }

label {
  display: block;
  font-size: 0.75rem;
  font-family: 'DM Mono', monospace;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--muted);
  margin-bottom: 0.5rem;
}

input[type="text"], input[type="number"] {
  width: 100%;
  background: var(--surface2);
  border: 1px solid var(--border);
  border-radius: 8px;
  color: var(--text);
  font-family: 'Syne', sans-serif;
  font-size: 1rem;
  padding: 0.65rem 0.9rem;
  outline: none;
  transition: border-color 0.15s;
}
input:focus { border-color: var(--accent); }
input::placeholder { color: var(--muted); }

.settings-row { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }

.hint {
  font-size: 0.7rem;
  font-family: 'DM Mono', monospace;
  color: var(--muted);
  margin-top: 0.35rem;
}

.btn {
  display: block;
  width: 100%;
  background: var(--accent);
  color: #1a1200;
  font-family: 'Syne', sans-serif;
  font-weight: 700;
  font-size: 1rem;
  letter-spacing: 0.02em;
  border: none;
  border-radius: 8px;
  padding: 0.8rem;
  cursor: pointer;
  transition: background 0.15s, transform 0.1s;
  margin-top: 0.5rem;
}
.btn:hover { background: #f0b45a; }
.btn:active { transform: scale(0.98); }

.error-msg {
  background: rgba(224,85,85,0.12);
  border: 1px solid rgba(224,85,85,0.3);
  color: var(--red);
  border-radius: 8px;
  padding: 0.65rem 0.9rem;
  font-size: 0.85rem;
  margin-bottom: 1rem;
  display: none;
}

/* Loading */
#loading { display: none; width: 100%; max-width: 480px; }

.loading-panel {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--r);
  padding: 1.75rem;
}

.loading-title {
  font-family: 'DM Mono', monospace;
  font-size: 0.68rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--muted);
  margin-bottom: 1.25rem;
}

.progress-bar-wrap {
  background: var(--surface2);
  border: 1px solid var(--border);
  border-radius: 4px;
  height: 6px;
  overflow: hidden;
  margin-bottom: 0.85rem;
}

.progress-bar-fill {
  height: 100%;
  background: var(--accent);
  border-radius: 4px;
  width: 0%;
  transition: width 0.35s ease;
}

.loading-status {
  font-family: 'DM Mono', monospace;
  font-size: 0.82rem;
  color: var(--text);
  margin-bottom: 0.3rem;
  min-height: 1.2em;
}

.loading-detail {
  font-family: 'DM Mono', monospace;
  font-size: 0.7rem;
  color: var(--muted);
  min-height: 1.1em;
}

.loading-steps {
  margin-top: 1.4rem;
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
  border-top: 1px solid var(--border);
  padding-top: 1.2rem;
}

.step {
  display: flex;
  align-items: center;
  gap: 0.65rem;
  font-family: 'DM Mono', monospace;
  font-size: 0.72rem;
  color: var(--muted);
  transition: color 0.25s;
}
.step.done { color: var(--green); }
.step.active { color: var(--text); }
.step-icon { font-size: 0.75rem; width: 14px; text-align: center; flex-shrink: 0; }

@keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.3} }
.step.active .step-icon { animation: pulse 1s ease-in-out infinite; }

/* Game */
#game { display: none; width: 100%; max-width: 680px; }

.score-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.score-item { text-align: center; }

.score-label {
  font-family: 'DM Mono', monospace;
  font-size: 0.65rem;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--muted);
  margin-bottom: 0.2rem;
}

.score-val { font-size: 1.5rem; font-weight: 800; letter-spacing: -0.02em; }
.score-val.streak { color: var(--accent); }
.score-val.correct { color: var(--green); }
.score-val.wrong { color: var(--red); }

.divider { width: 1px; height: 32px; background: var(--border); }

.prompt {
  font-family: 'DM Mono', monospace;
  font-size: 0.75rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--muted);
  text-align: center;
  margin-bottom: 1rem;
}

.cards {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1rem;
}
@media (max-width: 500px) { .cards { grid-template-columns: 1fr; } }

/* Big art card */
.track-card {
  background: var(--surface);
  border: 2px solid var(--border);
  border-radius: var(--r);
  cursor: pointer;
  transition: border-color 0.15s, transform 0.12s;
  user-select: none;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}
.track-card:hover:not(.resolved) {
  border-color: var(--accent);
  transform: translateY(-2px);
}
.track-card.correct-answer { border-color: var(--green); }
.track-card.wrong-answer { border-color: var(--red); }
.track-card.resolved { cursor: default; }

/* Full-width square art area */
.track-art-wrap {
  width: 100%;
  aspect-ratio: 1 / 1;
  background: var(--surface2);
  overflow: hidden;
  position: relative;
  flex-shrink: 0;
}

/* Shimmer while loading */
@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}
.track-art-wrap.loading-art::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, transparent 25%, rgba(255,255,255,0.04) 50%, transparent 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

.track-art-wrap img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  opacity: 0;
  transition: opacity 0.4s;
}
.track-art-wrap img.ready { opacity: 1; }

.track-art-placeholder {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}
.ph-note { font-size: 2.5rem; opacity: 0.2; }
.ph-text { font-family: 'DM Mono', monospace; font-size: 0.6rem; color: var(--muted); opacity: 0.6; letter-spacing: 0.06em; }

/* Result overlay on art */
.art-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 3.5rem;
  opacity: 0;
  transition: opacity 0.2s;
  pointer-events: none;
}
.track-card.correct-answer .art-overlay { background: rgba(106,184,122,0.28); opacity: 1; }
.track-card.wrong-answer .art-overlay { background: rgba(224,85,85,0.28); opacity: 1; }

/* Info strip below art */
.track-info {
  padding: 0.85rem 1rem 0.9rem;
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}
.track-name {
  font-weight: 700;
  font-size: 0.9rem;
  line-height: 1.3;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.track-artist {
  font-family: 'DM Mono', monospace;
  font-size: 0.7rem;
  color: var(--muted);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-bottom: 0.35rem;
}
.plays-row { display: flex; align-items: baseline; gap: 0.3rem; }
.plays-mask { font-family: 'DM Mono', monospace; font-size: 1rem; color: var(--muted); letter-spacing: 0.2em; }
.plays-val { font-family: 'DM Mono', monospace; font-size: 1rem; font-weight: 500; color: var(--accent); display: none; }
.plays-val.revealed { display: inline; }
.plays-unit { font-family: 'DM Mono', monospace; font-size: 0.65rem; color: var(--muted); display: none; }
.plays-unit.revealed { display: inline; }

.result-banner {
  text-align: center;
  padding: 0.75rem;
  border-radius: 8px;
  font-weight: 700;
  font-size: 1rem;
  letter-spacing: 0.02em;
  margin-bottom: 1rem;
  opacity: 0;
  transform: translateY(4px);
  transition: opacity 0.25s, transform 0.25s;
}
.result-banner.show { opacity: 1; transform: translateY(0); }
.result-banner.win { background: rgba(106,184,122,0.15); color: var(--green); border: 1px solid rgba(106,184,122,0.3); }
.result-banner.lose { background: rgba(224,85,85,0.1); color: var(--red); border: 1px solid rgba(224,85,85,0.25); }

.next-btn {
  display: block;
  width: 100%;
  background: var(--accent);
  color: #1a1200;
  font-family: 'Syne', sans-serif;
  font-weight: 700;
  font-size: 0.95rem;
  border: none;
  border-radius: 8px;
  padding: 0.75rem;
  cursor: pointer;
  opacity: 0;
  transform: translateY(4px);
  transition: opacity 0.25s, transform 0.25s, background 0.15s;
  pointer-events: none;
}
.next-btn.show { opacity: 1; transform: translateY(0); pointer-events: all; }
.next-btn:hover { background: #f0b45a; }

.bottom-row { display: flex; justify-content: space-between; align-items: center; margin-top: 1rem; }

.quit-btn {
  background: none; border: none;
  font-family: 'DM Mono', monospace;
  font-size: 0.7rem; color: var(--muted);
  cursor: pointer; letter-spacing: 0.06em;
  text-transform: uppercase; text-decoration: underline;
  text-underline-offset: 3px; padding: 0;
}
.quit-btn:hover { color: var(--text); }
.pool-info { font-family: 'DM Mono', monospace; font-size: 0.7rem; color: var(--muted); }

.or-divider {
  position: absolute; left: 50%; top: 50%;
  transform: translate(-50%, -50%);
  z-index: 10;
  background: var(--bg); border: 1px solid var(--border);
  border-radius: 20px; padding: 4px 10px;
  font-family: 'DM Mono', monospace; font-size: 0.65rem;
  color: var(--muted); letter-spacing: 0.15em; pointer-events: none;
}
</style>
</head>
<body>

<header>
  <div class="logo">scrobble<span>brain</span></div>
  <div class="tagline">do you know your own taste?</div>
</header>

<div id="setup">
  <div class="panel">
    <div class="error-msg" id="error-msg"></div>
    <div class="field">
      <label>lastfmstats.com export (.json)</label>
      <div id="drop-zone" style="border:2px dashed var(--border);border-radius:8px;padding:2rem 1rem;text-align:center;cursor:pointer;transition:border-color 0.15s;background:var(--surface2);">
        <div style="font-family:'DM Mono',monospace;font-size:0.8rem;color:var(--muted);margin-bottom:0.5rem;">drag &amp; drop your JSON file here</div>
        <div style="font-family:'DM Mono',monospace;font-size:0.7rem;color:var(--muted);">or click to browse</div>
      </div>
      <input type="file" id="json-file" accept=".json" style="display:none">
      <div class="hint" style="margin-top:0.5rem;">export from lastfmstats.com → ⚙ → Export</div>
    </div>
    <div class="settings-row">
      <div class="field">
        <label for="bucket-size">Percentile range</label>
        <input type="number" id="bucket-size" value="25" min="5" max="100" step="5">
        <div class="hint">tracks must be within this % of each other</div>
      </div>
      <div class="field">
        <label for="min-plays">Min plays</label>
        <input type="number" id="min-plays" value="10" min="1" max="9999">
        <div class="hint">ignore tracks with fewer plays</div>
      </div>
    </div>
    <button class="btn" id="start-btn" disabled style="opacity:0.4;cursor:not-allowed;">load a json file first</button>
  </div>
</div>

<div id="loading">
  <div class="loading-panel">
    <div class="loading-title">loading your library</div>
    <div class="progress-bar-wrap"><div class="progress-bar-fill" id="progress-fill"></div></div>
    <div class="loading-status" id="loading-status">parsing data…</div>
    <div class="loading-detail" id="loading-detail">&nbsp;</div>
    <div class="loading-steps">
      <div class="step" id="step-parse"><span class="step-icon">○</span><span id="step-parse-label">parse scrobble data</span></div>
      <div class="step" id="step-count"><span class="step-icon">○</span><span id="step-count-label">count unique tracks</span></div>
      <div class="step" id="step-filter"><span class="step-icon">○</span><span id="step-filter-label">filter &amp; prepare pool</span></div>
    </div>
  </div>
</div>

<div id="game">
  <div class="score-bar">
    <div class="score-item"><div class="score-label">correct</div><div class="score-val correct" id="score-correct">0</div></div>
    <div class="divider"></div>
    <div class="score-item"><div class="score-label">streak</div><div class="score-val streak" id="score-streak">0</div></div>
    <div class="divider"></div>
    <div class="score-item"><div class="score-label">wrong</div><div class="score-val wrong" id="score-wrong">0</div></div>
  </div>

  <div class="prompt">which have you played more?</div>

  <div style="position: relative;">
    <div class="cards">
      <div class="track-card" id="card-a"></div>
      <div class="track-card" id="card-b"></div>
    </div>
    <div class="or-divider">OR</div>
  </div>

  <div class="result-banner" id="result-banner"></div>
  <button class="next-btn" id="next-btn">next round →</button>

  <div class="bottom-row">
    <button class="quit-btn" id="quit-btn">← change user</button>
    <div class="pool-info" id="pool-info"></div>
  </div>
</div>

<script>
// ── Rate-limited iTunes art fetcher ────────────────────────────────────────
// iTunes Search API: ~20 req/min → 1 request every 3.5s to stay safe.
// We only ever fetch the 2 tracks currently on screen (on demand), then
// lookahead-fetch the next pair while the user is reading the result.
// Cache persists across rounds so each track is fetched at most once.

var artCache = {};   // key → url string | null
var artQueue = [];   // [{key, track, artist, resolve}]
var artTimer = null;
var RATE_MS = 3500;  // ms between requests

function artKey(name, artist) { return name + '|||' + artist; }

function fetchArt(name, artist) {
  var key = artKey(name, artist);
  if (artCache.hasOwnProperty(key)) return Promise.resolve(artCache[key]);

  return new Promise(function(resolve) {
    artQueue.push({ key: key, name: name, artist: artist, resolve: resolve });
    if (!artTimer) drainQueue();
  });
}

function drainQueue() {
  if (!artQueue.length) { artTimer = null; return; }
  var item = artQueue.shift();

  // already resolved by a parallel call
  if (artCache.hasOwnProperty(item.key)) {
    item.resolve(artCache[item.key]);
    artTimer = setTimeout(drainQueue, 0);
    return;
  }

  var q = encodeURIComponent(item.artist + ' ' + item.name);
  fetch('https://itunes.apple.com/search?term=' + q + '&entity=song&limit=5&media=music')
    .then(function(r) { return r.json(); })
    .then(function(data) {
      var url = null;
      if (data.results && data.results.length) {
        var hit = data.results.find(function(r) {
          return r.artistName && r.artistName.toLowerCase() === item.artist.toLowerCase();
        }) || data.results[0];
        if (hit && hit.artworkUrl100) {
          url = hit.artworkUrl100.replace('100x100bb', '600x600bb');
        }
      }
      artCache[item.key] = url;
      item.resolve(url);
    })
    .catch(function() {
      artCache[item.key] = null;
      item.resolve(null);
    })
    .finally(function() {
      artTimer = setTimeout(drainQueue, RATE_MS);
    });
}

// ── JSON parser ────────────────────────────────────────────────────────────
function parseScrobbleJSON(json) {
  var data = JSON.parse(json);
  var scrobbles = data.scrobbles || data;
  if (!Array.isArray(scrobbles)) throw new Error('Could not find scrobbles array in JSON');

  var counts = {}, meta = {};
  scrobbles.forEach(function(s) {
    var k = s.track + '|||' + s.artist;
    counts[k] = (counts[k] || 0) + 1;
    if (!meta[k]) meta[k] = { name: s.track, artist: { name: s.artist } };
  });
  return Object.keys(counts).map(function(k) {
    return Object.assign({}, meta[k], { playcount: counts[k] });
  });
}

// ── State ──────────────────────────────────────────────────────────────────
var tracks = [];
var bucketSize = 25, minPlays = 10;
var scoreCorrect = 0, scoreWrong = 0, streak = 0;
var resolved = false, currentPair = null;
var nextPair = null; // lookahead

function $(id) { return document.getElementById(id); }

function setProgress(pct, status, detail) {
  $('progress-fill').style.width = pct + '%';
  if (status != null) $('loading-status').textContent = status;
  if (detail !== undefined) $('loading-detail').textContent = detail || '\u00a0';
}
function setStep(id, state) {
  var el = $(id);
  el.className = 'step' + (state === 'active' ? ' active' : state === 'done' ? ' done' : '');
  el.querySelector('.step-icon').textContent = state === 'done' ? '✓' : state === 'active' ? '●' : '○';
}
function esc(str) {
  return String(str).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}

// ── Card rendering ─────────────────────────────────────────────────────────
function buildCard(track, side) {
  var key = artKey(track.name, track.artist.name);
  var cached = artCache.hasOwnProperty(key) ? artCache[key] : undefined;

  var artInner;
  if (cached) {
    artInner = cached
      ? '<img src="' + esc(cached) + '" id="art-img-' + side + '" alt="">'
      : placeholder();
  } else {
    artInner = placeholder();
  }

  return '<div class="track-art-wrap' + (cached === undefined ? ' loading-art' : '') + '" id="art-wrap-' + side + '">'
       + artInner
       + '<div class="art-overlay" id="art-ol-' + side + '"></div>'
       + '</div>'
       + '<div class="track-info">'
       + '<div class="track-name">' + esc(track.name) + '</div>'
       + '<div class="track-artist">' + esc(track.artist.name) + '</div>'
       + '<div class="plays-row">'
       + '<span class="plays-mask" id="mask-' + side + '">• • •</span>'
       + '<span class="plays-val" id="plays-' + side + '">' + Number(track.playcount).toLocaleString() + '</span>'
       + '<span class="plays-unit" id="plays-u-' + side + '"> plays</span>'
       + '</div></div>';
}

function placeholder() {
  return '<div class="track-art-placeholder"><div class="ph-note">♪</div><div class="ph-text">loading art…</div></div>';
}

function noArtPlaceholder() {
  return '<div class="track-art-placeholder"><div class="ph-note">♪</div><div class="ph-text">no art found</div></div>';
}

// Attach art to a rendered card (called after fetch resolves)
function injectArt(side, url) {
  var wrap = $('art-wrap-' + side);
  if (!wrap) return; // card gone
  wrap.classList.remove('loading-art');
  // clear existing content (placeholder or old img) before overlay
  var overlay = $('art-ol-' + side);
  while (wrap.firstChild !== overlay) wrap.removeChild(wrap.firstChild);

  if (url) {
    var img = document.createElement('img');
    img.alt = '';
    img.id = 'art-img-' + side;
    img.onload = function() { img.classList.add('ready'); };
    img.onerror = function() { img.parentNode && img.parentNode.insertBefore(noArtEl(), img); img.remove(); };
    img.src = url;
    wrap.insertBefore(img, overlay);
  } else {
    var ph = document.createElement('div');
    ph.innerHTML = noArtPlaceholder();
    wrap.insertBefore(ph.firstChild, overlay);
  }
}

function noArtEl() {
  var d = document.createElement('div');
  d.innerHTML = noArtPlaceholder();
  return d.firstChild;
}

// ── Pair picker ────────────────────────────────────────────────────────────
function pickPair() {
  var plays = tracks.map(function(t) { return t.playcount; });
  var min = Math.min.apply(null, plays), max = Math.max.apply(null, plays);
  var range = max - min || 1, bf = bucketSize / 100;
  for (var i = 0; i < 3000; i++) {
    var ai = Math.floor(Math.random() * tracks.length);
    var bi = Math.floor(Math.random() * tracks.length);
    if (ai === bi) continue;
    var a = tracks[ai], b = tracks[bi];
    if (a.playcount === b.playcount) continue;
    if (Math.abs(a.playcount - b.playcount) / range <= bf) return [a, b];
  }
  return null;
}

// ── Show pair ──────────────────────────────────────────────────────────────
function showPair() {
  resolved = false;
  var pair = nextPair || pickPair();
  nextPair = null;

  if (!pair) {
    $('result-banner').textContent = 'No valid pair found — try a wider percentile range.';
    $('result-banner').className = 'result-banner show lose';
    return;
  }
  currentPair = pair;

  $('card-a').className = 'track-card';
  $('card-b').className = 'track-card';
  $('card-a').innerHTML = buildCard(pair[0], 'a');
  $('card-b').innerHTML = buildCard(pair[1], 'b');
  $('card-a').onclick = function() { guess('a'); };
  $('card-b').onclick = function() { guess('b'); };
  $('result-banner').className = 'result-banner';
  $('next-btn').className = 'next-btn';

  // Wire up existing cached images
  ['a','b'].forEach(function(s, i) {
    var t = pair[i];
    var key = artKey(t.name, t.artist.name);
    if (artCache.hasOwnProperty(key)) {
      // already cached — wire the img fade-in
      var img = $('art-img-' + s);
      if (img) {
        if (img.complete && img.naturalWidth) { img.classList.add('ready'); }
        else { img.onload = function() { img.classList.add('ready'); }; }
      }
    } else {
      // fetch on demand
      fetchArt(t.name, t.artist.name).then(function(url) { injectArt(s, url); });
    }
  });
}

// ── Guess ──────────────────────────────────────────────────────────────────
function guess(side) {
  if (resolved) return;
  resolved = true;
  var a = currentPair[0], b = currentPair[1];
  var winner = a.playcount > b.playcount ? 'a' : 'b';
  var correct = side === winner;
  var loser = winner === 'a' ? 'b' : 'a';

  ['a','b'].forEach(function(s) {
    var mask = $('mask-' + s), val = $('plays-' + s), unit = $('plays-u-' + s);
    if (mask) mask.style.display = 'none';
    if (val) val.classList.add('revealed');
    if (unit) unit.classList.add('revealed');
  });

  $('card-a').classList.add('resolved');
  $('card-b').classList.add('resolved');

  if (correct) {
    scoreCorrect++; streak++;
    $('card-' + side).classList.add('correct-answer');
    $('card-' + loser).classList.add('wrong-answer');
    $('art-ol-' + side).textContent = '✓';
    $('art-ol-' + loser).textContent = '✗';
    $('result-banner').textContent = streak > 2 ? '✓ correct! ' + streak + ' in a row 🔥' : '✓ correct!';
    $('result-banner').className = 'result-banner show win';
  } else {
    scoreWrong++; streak = 0;
    $('card-' + side).classList.add('wrong-answer');
    $('card-' + winner).classList.add('correct-answer');
    $('art-ol-' + side).textContent = '✗';
    $('art-ol-' + winner).textContent = '✓';
    $('result-banner').textContent = '✗ not quite';
    $('result-banner').className = 'result-banner show lose';
  }

  $('score-correct').textContent = scoreCorrect;
  $('score-wrong').textContent = scoreWrong;
  $('score-streak').textContent = streak;
  $('next-btn').className = 'next-btn show';

  // Lookahead: pick and pre-fetch art for the next pair while user reads result
  var lp = pickPair();
  if (lp) {
    nextPair = lp;
    lp.forEach(function(t) {
      var key = artKey(t.name, t.artist.name);
      if (!artCache.hasOwnProperty(key)) fetchArt(t.name, t.artist.name);
    });
  }
}

// ── File handling ──────────────────────────────────────────────────────────
var loadedJSON = null;
var dropZone = $('drop-zone'), fileInput = $('json-file');

function handleFile(file) {
  if (!file || !file.name.endsWith('.json')) { showError('please select a .json file'); return; }
  var reader = new FileReader();
  reader.onload = function(e) {
    try {
      loadedJSON = e.target.result;
      var parsed = JSON.parse(loadedJSON);
      var scrobbles = parsed.scrobbles || parsed;
      if (!Array.isArray(scrobbles) || !scrobbles.length) throw new Error('No scrobbles found');
      dropZone.style.borderColor = 'var(--green)';
      dropZone.innerHTML = '<div style="font-family:monospace;font-size:0.8rem;color:var(--green);">✓ ' + file.name + '</div>'
        + '<div style="font-family:monospace;font-size:0.7rem;color:var(--muted);margin-top:0.3rem;">' + scrobbles.length.toLocaleString() + ' scrobbles loaded</div>';
      var btn = $('start-btn');
      btn.disabled = false; btn.style.opacity = '1'; btn.style.cursor = 'pointer';
      btn.textContent = 'start game →';
      hideError();
    } catch(err) {
      showError('invalid JSON: ' + (err.message || 'could not parse'));
      loadedJSON = null;
    }
  };
  reader.readAsText(file);
}

dropZone.addEventListener('click', function() { fileInput.click(); });
fileInput.addEventListener('change', function() { if (fileInput.files[0]) handleFile(fileInput.files[0]); });
dropZone.addEventListener('dragover', function(e) { e.preventDefault(); dropZone.style.borderColor='var(--accent)'; });
dropZone.addEventListener('dragleave', function() { dropZone.style.borderColor='var(--border)'; });
dropZone.addEventListener('drop', function(e) {
  e.preventDefault(); dropZone.style.borderColor='var(--border)';
  handleFile(e.dataTransfer.files[0]);
});

// ── Start game ─────────────────────────────────────────────────────────────
function startGame() {
  if (!loadedJSON) { showError('load a json file first'); return; }
  bucketSize = Math.max(5, Math.min(100, parseInt($('bucket-size').value) || 25));
  minPlays   = Math.max(1, parseInt($('min-plays').value) || 10);
  hideError();
  $('setup').style.display = 'none';
  $('loading').style.display = 'block';
  ['step-parse','step-count','step-filter'].forEach(function(s) { setStep(s,'pending'); });

  try {
    setStep('step-parse','active');
    setProgress(20, 'parsing scrobble data…', '');
    var allTracks = parseScrobbleJSON(loadedJSON);

    setStep('step-parse','done'); setStep('step-count','active');
    $('step-count-label').textContent = allTracks.length.toLocaleString() + ' unique tracks found';
    setProgress(55, 'found ' + allTracks.length.toLocaleString() + ' unique tracks', '');

    setStep('step-count','done'); setStep('step-filter','active');
    setProgress(75, 'filtering tracks…', 'min ' + minPlays + ' plays · ±' + bucketSize + '% window');

    tracks = allTracks.filter(function(t) { return t.playcount >= minPlays; });
    if (tracks.length < 2) throw new Error('Only ' + tracks.length + ' track(s) with ≥' + minPlays + ' plays. Try lowering the minimum.');

    $('step-filter-label').textContent = tracks.length.toLocaleString() + ' tracks in pool';
    setStep('step-filter','done');
    setProgress(100, 'ready! ' + tracks.length.toLocaleString() + ' tracks', '');

    setTimeout(function() {
      scoreCorrect = 0; scoreWrong = 0; streak = 0; nextPair = null;
      $('score-correct').textContent = '0';
      $('score-wrong').textContent = '0';
      $('score-streak').textContent = '0';
      $('pool-info').textContent = tracks.length.toLocaleString() + ' tracks · ±' + bucketSize + '% window';
      $('loading').style.display = 'none';
      $('game').style.display = 'block';
      showPair();
    }, 500);

  } catch(e) {
    $('loading').style.display = 'none';
    $('setup').style.display = 'block';
    showError(e.message || 'something went wrong');
  }
}

function showError(msg) { var el=$('error-msg'); el.textContent=msg; el.style.display='block'; }
function hideError() { $('error-msg').style.display='none'; }

$('start-btn').onclick = startGame;
$('next-btn').onclick = showPair;
$('quit-btn').onclick = function() { $('game').style.display='none'; $('setup').style.display='block'; };
</script>
</body>
</html>
