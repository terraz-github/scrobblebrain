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
#loading {
  display: none;
  width: 100%;
  max-width: 480px;
}

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

.step-icon {
  font-size: 0.75rem;
  width: 14px;
  text-align: center;
  flex-shrink: 0;
}

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

.track-card {
  background: var(--surface);
  border: 2px solid var(--border);
  border-radius: var(--r);
  padding: 1.25rem;
  cursor: pointer;
  transition: border-color 0.15s, transform 0.12s, background 0.15s;
  user-select: none;
  min-height: 160px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
.track-card:hover:not(.resolved) {
  border-color: var(--accent);
  background: var(--surface2);
  transform: translateY(-2px);
}
.track-card.correct-answer { border-color: var(--green); background: rgba(106,184,122,0.08); }
.track-card.wrong-answer { border-color: var(--red); background: rgba(224,85,85,0.08); }
.track-card.resolved { cursor: default; }

.track-art {
  width: 52px;
  height: 52px;
  border-radius: 6px;
  background: var(--surface2);
  margin-bottom: 0.75rem;
  overflow: hidden;
  flex-shrink: 0;
}
.track-art img { width: 100%; height: 100%; object-fit: cover; display: block; }
.track-art-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.4rem;
}

.track-name { font-weight: 700; font-size: 0.95rem; line-height: 1.3; margin-bottom: 0.3rem; }
.track-artist { font-family: 'DM Mono', monospace; font-size: 0.75rem; color: var(--muted); margin-bottom: 0.75rem; }

.plays-row { display: flex; align-items: center; gap: 0.4rem; }
.plays-mask { font-family: 'DM Mono', monospace; font-size: 1.1rem; font-weight: 500; color: var(--muted); letter-spacing: 0.15em; }
.plays-val { font-family: 'DM Mono', monospace; font-size: 1.1rem; font-weight: 500; color: var(--accent); display: none; }
.plays-val.revealed { display: inline; }

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
  background: none;
  border: none;
  font-family: 'DM Mono', monospace;
  font-size: 0.7rem;
  color: var(--muted);
  cursor: pointer;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  text-decoration: underline;
  text-underline-offset: 3px;
  padding: 0;
}
.quit-btn:hover { color: var(--text); }

.pool-info { font-family: 'DM Mono', monospace; font-size: 0.7rem; color: var(--muted); }

.or-divider {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  z-index: 10;
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 20px;
  padding: 4px 10px;
  font-family: 'DM Mono', monospace;
  font-size: 0.65rem;
  color: var(--muted);
  letter-spacing: 0.15em;
  pointer-events: none;
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
    <div class="progress-bar-wrap">
      <div class="progress-bar-fill" id="progress-fill"></div>
    </div>
    <div class="loading-status" id="loading-status">connecting to last.fm...</div>
    <div class="loading-detail" id="loading-detail">&nbsp;</div>
    <div class="loading-steps">
      <div class="step" id="step-connect">
        <span class="step-icon">○</span>
        <span>connect to last.fm</span>
      </div>
      <div class="step" id="step-count">
        <span class="step-icon">○</span>
        <span id="step-count-label">fetch track count</span>
      </div>
      <div class="step" id="step-pages">
        <span class="step-icon">○</span>
        <span id="step-pages-label">load all pages</span>
      </div>
      <div class="step" id="step-filter">
        <span class="step-icon">○</span>
        <span id="step-filter-label">filter &amp; prepare</span>
      </div>
    </div>
  </div>
</div>

<div id="game">
  <div class="score-bar">
    <div class="score-item">
      <div class="score-label">correct</div>
      <div class="score-val correct" id="score-correct">0</div>
    </div>
    <div class="divider"></div>
    <div class="score-item">
      <div class="score-label">streak</div>
      <div class="score-val streak" id="score-streak">0</div>
    </div>
    <div class="divider"></div>
    <div class="score-item">
      <div class="score-label">wrong</div>
      <div class="score-val wrong" id="score-wrong">0</div>
    </div>
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
// Parse scrobble JSON exported from lastfmstats.com and count plays per track
function parseScrobbleJSON(json) {
  var data = JSON.parse(json);
  var scrobbles = data.scrobbles || data;
  if (!Array.isArray(scrobbles)) throw new Error('Could not find scrobbles array in JSON');

  var counts = {};
  var meta = {};
  scrobbles.forEach(function(s) {
    var key = s.track + '|||' + s.artist;
    counts[key] = (counts[key] || 0) + 1;
    if (!meta[key]) meta[key] = { name: s.track, artist: { name: s.artist }, image: [] };
  });

  return Object.keys(counts).map(function(k) {
    return Object.assign({}, meta[k], { playcount: counts[k] });
  });
}

// State
var tracks = [];
var bucketSize = 25;
var minPlays = 10;
var scoreCorrect = 0, scoreWrong = 0, streak = 0;
var resolved = false;
var currentPair = null;

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
  return String(str)
    .replace(/&/g,'&amp;').replace(/</g,'&lt;')
    .replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}

// iTunes art cache: key = "artist|||track"
var artCache = {};

function fetchArt(track, side) {
  var key = track.artist.name + '|||' + track.name;
  if (artCache[key]) {
    applyArt(side, artCache[key]);
    return;
  }
  if (artCache[key] === null) return; // known miss
  var query = encodeURIComponent(track.artist.name + ' ' + track.name);
  var url = 'https://itunes.apple.com/search?term=' + query + '&media=music&limit=1&callback=itunesCb_' + side + '_' + Date.now();
  var cbName = 'itunesCb_' + side + '_' + Date.now();
  url = 'https://itunes.apple.com/search?term=' + query + '&media=music&limit=1&callback=' + cbName;
  window[cbName] = function(data) {
    delete window[cbName];
    if (script.parentNode) script.parentNode.removeChild(script);
    if (data && data.results && data.results.length > 0) {
      var artUrl = data.results[0].artworkUrl100.replace('100x100', '300x300');
      artCache[key] = artUrl;
      applyArt(side, artUrl);
    } else {
      artCache[key] = null;
    }
  };
  var script = document.createElement('script');
  script.src = url;
  script.onerror = function() { if (script.parentNode) script.parentNode.removeChild(script); };
  document.head.appendChild(script);
}

function applyArt(side, url) {
  var el = document.getElementById('art-' + side);
  if (!el) return;
  el.innerHTML = '<img src="' + esc(url) + '" alt="" style="width:100%;height:100%;object-fit:cover;display:block;border-radius:6px;">';
}

function buildCardHTML(track, side) {
  return '<div class="track-art" id="art-' + side + '"><div class="track-art-placeholder">\u266a</div></div>'
    + '<div><div class="track-name">' + esc(track.name) + '</div>'
    + '<div class="track-artist">' + esc(track.artist.name) + '</div></div>'
    + '<div class="plays-row">'
    + '<span class="plays-mask" id="mask-' + side + '">\u2022 \u2022 \u2022</span>'
    + '<span class="plays-val" id="plays-' + side + '">' + Number(track.playcount).toLocaleString() + '</span>'
    + '</div>';
}

function pickPair() {
  var plays = tracks.map(function(t){ return t.playcount; });
  var min = Math.min.apply(null, plays);
  var max = Math.max.apply(null, plays);
  var range = max - min || 1;
  var bucketFrac = bucketSize / 100;
  for (var i = 0; i < 3000; i++) {
    var ai = Math.floor(Math.random() * tracks.length);
    var bi = Math.floor(Math.random() * tracks.length);
    if (ai === bi) continue;
    var a = tracks[ai], b = tracks[bi];
    if (a.playcount === b.playcount) continue;
    if (Math.abs(a.playcount - b.playcount) / range <= bucketFrac) return [a, b];
  }
  return null;
}

function showPair() {
  resolved = false;
  var pair = pickPair();
  if (!pair) {
    $('result-banner').textContent = 'No valid pair found — try a wider percentile range.';
    $('result-banner').className = 'result-banner show lose';
    return;
  }
  currentPair = pair;
  var a = pair[0], b = pair[1];
  $('card-a').className = 'track-card';
  $('card-b').className = 'track-card';
  $('card-a').innerHTML = buildCardHTML(a, 'a');
  $('card-b').innerHTML = buildCardHTML(b, 'b');
  $('card-a').onclick = function(){ guess('a'); };
  $('card-b').onclick = function(){ guess('b'); };
  fetchArt(a, 'a');
  fetchArt(b, 'b');
  $('result-banner').className = 'result-banner';
  $('next-btn').className = 'next-btn';
}

function guess(side) {
  if (resolved) return;
  resolved = true;
  var a = currentPair[0], b = currentPair[1];
  var winner = a.playcount > b.playcount ? 'a' : 'b';
  var correct = side === winner;

  ['a','b'].forEach(function(s) {
    var mask = document.getElementById('mask-' + s);
    var val = document.getElementById('plays-' + s);
    if (mask) mask.style.display = 'none';
    if (val) val.classList.add('revealed');
  });

  $('card-a').classList.add('resolved');
  $('card-b').classList.add('resolved');

  if (correct) {
    scoreCorrect++;
    streak++;
    document.getElementById('card-' + side).classList.add('correct-answer');
    document.getElementById('card-' + (winner === 'a' ? 'b' : 'a')).classList.add('wrong-answer');
    $('result-banner').textContent = streak > 2 ? '\u2713 correct! ' + streak + ' in a row \uD83D\uDD25' : '\u2713 correct!';
    $('result-banner').className = 'result-banner show win';
  } else {
    scoreWrong++;
    streak = 0;
    document.getElementById('card-' + side).classList.add('wrong-answer');
    document.getElementById('card-' + winner).classList.add('correct-answer');
    $('result-banner').textContent = '\u2717 not quite';
    $('result-banner').className = 'result-banner show lose';
  }

  $('score-correct').textContent = scoreCorrect;
  $('score-wrong').textContent = scoreWrong;
  $('score-streak').textContent = streak;
  $('next-btn').className = 'next-btn show';
}

var loadedJSON = null;

// File picker / drag-drop wiring
var dropZone = $('drop-zone');
var fileInput = $('json-file');

function handleFile(file) {
  if (!file || !file.name.endsWith('.json')) {
    showError('please select a .json file');
    return;
  }
  var reader = new FileReader();
  reader.onload = function(e) {
    try {
      loadedJSON = e.target.result;
      // Quick validation
      var parsed = JSON.parse(loadedJSON);
      var scrobbles = parsed.scrobbles || parsed;
      if (!Array.isArray(scrobbles) || scrobbles.length === 0) throw new Error('No scrobbles found');
      dropZone.style.borderColor = 'var(--green)';
      dropZone.innerHTML = '<div style="font-family:monospace;font-size:0.8rem;color:var(--green);">✓ ' + file.name + '</div>'
        + '<div style="font-family:monospace;font-size:0.7rem;color:var(--muted);margin-top:0.3rem;">' + scrobbles.length.toLocaleString() + ' scrobbles loaded</div>';
      var btn = $('start-btn');
      btn.disabled = false;
      btn.style.opacity = '1';
      btn.style.cursor = 'pointer';
      btn.textContent = 'start game →';
      hideError();
    } catch(err) {
      showError('invalid JSON: ' + (err.message || 'could not parse file'));
      loadedJSON = null;
    }
  };
  reader.readAsText(file);
}

dropZone.addEventListener('click', function() { fileInput.click(); });
fileInput.addEventListener('change', function() { if (fileInput.files[0]) handleFile(fileInput.files[0]); });
dropZone.addEventListener('dragover', function(e) { e.preventDefault(); dropZone.style.borderColor = 'var(--accent)'; });
dropZone.addEventListener('dragleave', function() { dropZone.style.borderColor = 'var(--border)'; });
dropZone.addEventListener('drop', function(e) {
  e.preventDefault();
  dropZone.style.borderColor = 'var(--border)';
  handleFile(e.dataTransfer.files[0]);
});

function startGame() {
  if (!loadedJSON) { showError('load a json file first'); return; }

  bucketSize = Math.max(5, Math.min(100, parseInt($('bucket-size').value) || 25));
  minPlays   = Math.max(1, parseInt($('min-plays').value) || 10);

  hideError();
  $('setup').style.display = 'none';
  $('loading').style.display = 'block';

  ['step-connect','step-count','step-pages','step-filter'].forEach(function(s){ setStep(s, 'pending'); });
  $('step-pages-label').textContent = 'count plays per track';
  $('step-filter-label').textContent = 'filter & prepare';

  try {
    setStep('step-connect', 'active');
    setProgress(20, 'parsing scrobble data...', '');

    var allTracks = parseScrobbleJSON(loadedJSON);

    setStep('step-connect', 'done');
    setStep('step-count', 'active');
    setProgress(50, 'found ' + allTracks.length.toLocaleString() + ' unique tracks', '');
    $('step-count-label').textContent = allTracks.length.toLocaleString() + ' unique tracks found';
    setStep('step-count', 'done');
    setStep('step-pages', 'active');
    setProgress(75, 'counting plays...', '');
    $('step-pages-label').textContent = 'plays counted';
    setStep('step-pages', 'done');
    setStep('step-filter', 'active');
    setProgress(90, 'filtering tracks...', 'min ' + minPlays + ' plays · ±' + bucketSize + '% window');

    tracks = allTracks.filter(function(t){ return t.playcount >= minPlays; });

    if (tracks.length < 2) {
      throw new Error('Only ' + tracks.length + ' track(s) with ≥ ' + minPlays + ' plays. Try lowering the minimum.');
    }

    setProgress(100,
      'ready! ' + tracks.length.toLocaleString() + ' tracks in pool',
      allTracks.length + ' total, ' + (allTracks.length - tracks.length) + ' below min plays'
    );
    $('step-filter-label').textContent = tracks.length.toLocaleString() + ' tracks ready';
    setStep('step-filter', 'done');

    setTimeout(function() {
      scoreCorrect = 0; scoreWrong = 0; streak = 0;
      $('score-correct').textContent = '0';
      $('score-wrong').textContent = '0';
      $('score-streak').textContent = '0';
      $('pool-info').textContent = tracks.length.toLocaleString() + ' tracks · ±' + bucketSize + '% window';
      $('loading').style.display = 'none';
      $('game').style.display = 'block';
      showPair();
    }, 600);

  } catch(e) {
    $('loading').style.display = 'none';
    $('setup').style.display = 'block';
    showError(e.message || 'something went wrong');
  }
}

function showError(msg) {
  var el = $('error-msg');
  el.textContent = msg;
  el.style.display = 'block';
}

function hideError() {
  $('error-msg').style.display = 'none';
}

$('start-btn').onclick = startGame;
$('next-btn').onclick = showPair;
$('quit-btn').onclick = function() {
  $('game').style.display = 'none';
  $('setup').style.display = 'block';
};
</script>
</body>
</html>
