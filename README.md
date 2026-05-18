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
    --accent2: #c96b2f;
    --green: #6ab87a;
    --red: #e05555;
    --radius: 12px;
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

  /* Grain overlay */
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

  header {
    text-align: center;
    margin-bottom: 2.5rem;
  }

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

  /* Setup panel */
  #setup {
    width: 100%;
    max-width: 480px;
  }

  .panel {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 1.75rem;
  }

  .field {
    margin-bottom: 1.25rem;
  }

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

  .settings-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }

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

  .btn.ghost {
    background: transparent;
    color: var(--muted);
    border: 1px solid var(--border);
    margin-top: 0.75rem;
  }

  .btn.ghost:hover { color: var(--text); border-color: var(--muted); background: transparent; }

  .error-msg {
    background: rgba(224, 85, 85, 0.12);
    border: 1px solid rgba(224, 85, 85, 0.3);
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
    text-align: center;
    padding: 3rem 0;
  }

  .spinner {
    width: 48px;
    height: 48px;
    border-radius: 50%;
    border: 3px solid var(--border);
    border-top-color: var(--accent);
    animation: spin 0.8s linear infinite;
    margin: 0 auto 1rem;
  }

  @keyframes spin { to { transform: rotate(360deg); } }

  .loading-text {
    font-family: 'DM Mono', monospace;
    font-size: 0.8rem;
    color: var(--muted);
    letter-spacing: 0.06em;
  }

  /* Game */
  #game {
    display: none;
    width: 100%;
    max-width: 680px;
  }

  .score-bar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1.5rem;
  }

  .score-item {
    text-align: center;
  }

  .score-label {
    font-family: 'DM Mono', monospace;
    font-size: 0.65rem;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    color: var(--muted);
    margin-bottom: 0.2rem;
  }

  .score-val {
    font-size: 1.5rem;
    font-weight: 800;
    letter-spacing: -0.02em;
  }

  .score-val.streak { color: var(--accent); }
  .score-val.correct { color: var(--green); }
  .score-val.wrong { color: var(--red); }

  .divider {
    width: 1px;
    height: 32px;
    background: var(--border);
  }

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

  @media (max-width: 500px) {
    .cards { grid-template-columns: 1fr; }
  }

  .track-card {
    background: var(--surface);
    border: 2px solid var(--border);
    border-radius: var(--radius);
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

  .track-card.selected { border-color: var(--accent); }

  .track-card.correct-answer {
    border-color: var(--green);
    background: rgba(106, 184, 122, 0.08);
  }

  .track-card.wrong-answer {
    border-color: var(--red);
    background: rgba(224, 85, 85, 0.08);
  }

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

  .track-art img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
  }

  .track-art-placeholder {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.4rem;
  }

  .track-name {
    font-weight: 700;
    font-size: 0.95rem;
    line-height: 1.3;
    margin-bottom: 0.3rem;
    color: var(--text);
  }

  .track-artist {
    font-family: 'DM Mono', monospace;
    font-size: 0.75rem;
    color: var(--muted);
    margin-bottom: 0.75rem;
  }

  .plays-row {
    display: flex;
    align-items: center;
    gap: 0.4rem;
  }

  .plays-label {
    font-family: 'DM Mono', monospace;
    font-size: 0.7rem;
    color: var(--muted);
    text-transform: uppercase;
    letter-spacing: 0.06em;
  }

  .plays-val {
    font-family: 'DM Mono', monospace;
    font-size: 1.1rem;
    font-weight: 500;
    color: var(--accent);
    opacity: 0;
    transition: opacity 0.3s;
  }

  .plays-val.hidden-count { opacity: 0; }
  .plays-val.show-count { opacity: 1; }

  .plays-mask {
    font-family: 'DM Mono', monospace;
    font-size: 1.1rem;
    font-weight: 500;
    color: var(--muted);
    letter-spacing: 0.15em;
  }

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
  .result-banner.win { background: rgba(106, 184, 122, 0.15); color: var(--green); border: 1px solid rgba(106, 184, 122, 0.3); }
  .result-banner.lose { background: rgba(224, 85, 85, 0.1); color: var(--red); border: 1px solid rgba(224, 85, 85, 0.25); }

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

  .bottom-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 1rem;
  }

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

  .pool-info {
    font-family: 'DM Mono', monospace;
    font-size: 0.7rem;
    color: var(--muted);
    letter-spacing: 0.04em;
  }

  /* VS badge */
  .vs-badge {
    text-align: center;
    font-family: 'DM Mono', monospace;
    font-size: 0.7rem;
    letter-spacing: 0.1em;
    color: var(--muted);
    padding: 0.5rem 0;
    display: none;
  }

  @media (max-width: 500px) {
    .vs-badge { display: block; }
  }

  /* Vinyl decoration on cards (subtle) */
  .track-card::after {
    content: '';
    position: absolute;
    bottom: -6px;
    right: -6px;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    border: 1.5px solid var(--border);
    opacity: 0.3;
    pointer-events: none;
    z-index: 0;
  }

  .or-divider {
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: 'DM Mono', monospace;
    font-size: 0.65rem;
    color: var(--muted);
    letter-spacing: 0.15em;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    z-index: 10;
    background: var(--bg);
    border: 1px solid var(--border);
    border-radius: 20px;
    padding: 4px 10px;
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
      <label for="username">Last.fm username</label>
      <input type="text" id="username" placeholder="e.g. rj" autocomplete="off" autocorrect="off" spellcheck="false">
    </div>

    <div class="settings-row">
      <div class="field">
        <label for="bucket-size">Percentile range</label>
        <input type="number" id="bucket-size" value="25" min="5" max="100" step="5">
        <div class="hint">tracks must be within this % of each other</div>
      </div>
      <div class="field">
        <label for="min-plays">Min plays</label>
        <input type="number" id="min-plays" value="10" min="1" max="999">
        <div class="hint">ignore tracks with fewer plays</div>
      </div>
    </div>

    <button class="btn" id="start-btn">load my tracks →</button>
  </div>
</div>

<div id="loading">
  <div class="spinner"></div>
  <div class="loading-text" id="loading-text">fetching your scrobbles...</div>
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
    <div class="cards" id="cards-container">
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
const API_KEY = '2f3407ec29f87e67a834702bbbb8c1da';
const BASE = 'https://ws.audioscrobbler.com/2.0/';

let tracks = [];
let bucketSize = 25;
let minPlays = 10;
let scoreCorrect = 0;
let scoreWrong = 0;
let streak = 0;
let resolved = false;
let currentPair = null;

const $ = id => document.getElementById(id);

async function fetchAllTopTracks(user) {
  const perPage = 1000;
  const url = `${BASE}?method=user.gettoptracks&user=${encodeURIComponent(user)}&api_key=${API_KEY}&format=json&limit=${perPage}&period=overall`;
  const res = await fetch(url);
  if (!res.ok) throw new Error('Network error');
  const data = await res.json();
  if (data.error) throw new Error(data.message || 'Last.fm error');
  const toptracks = data.toptracks;
  if (!toptracks || !toptracks.track) throw new Error('No tracks found');
  return toptracks.track;
}

function buildCardHTML(track, side) {
  const art = track.image && track.image[1] && track.image[1]['#text'];
  const artEl = art
    ? `<img src="${art}" alt="" loading="lazy">`
    : `<div class="track-art-placeholder">♪</div>`;
  return `
    <div class="track-art">${artEl}</div>
    <div>
      <div class="track-name">${esc(track.name)}</div>
      <div class="track-artist">${esc(track.artist.name)}</div>
    </div>
    <div class="plays-row">
      <span class="plays-mask" id="mask-${side}">• • •</span>
      <span class="plays-val" id="plays-${side}">${Number(track.playcount).toLocaleString()}</span>
    </div>
  `;
}

function esc(str) {
  return str.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}

function pickPair() {
  const plays = tracks.map(t => t.playcount);
  const min = Math.min(...plays);
  const max = Math.max(...plays);
  const range = max - min || 1;
  const bucketFrac = bucketSize / 100;

  let attempts = 0;
  while (attempts < 2000) {
    attempts++;
    const i = Math.floor(Math.random() * tracks.length);
    const j = Math.floor(Math.random() * tracks.length);
    if (i === j) continue;
    const a = tracks[i], b = tracks[j];
    if (a.playcount === b.playcount) continue;
    const diff = Math.abs(a.playcount - b.playcount) / range;
    if (diff <= bucketFrac) {
      return [a, b];
    }
  }
  return null;
}

function showPair() {
  resolved = false;
  const pair = pickPair();
  if (!pair) {
    $('result-banner').textContent = 'No valid pair found with these settings. Try a wider percentile range.';
    $('result-banner').className = 'result-banner show lose';
    return;
  }
  currentPair = pair;
  const [a, b] = pair;

  const cardA = $('card-a');
  const cardB = $('card-b');
  cardA.className = 'track-card';
  cardB.className = 'track-card';
  cardA.innerHTML = buildCardHTML(a, 'a');
  cardB.innerHTML = buildCardHTML(b, 'b');

  cardA.onclick = () => guess('a');
  cardB.onclick = () => guess('b');

  $('result-banner').className = 'result-banner';
  $('next-btn').className = 'next-btn';
}

function guess(side) {
  if (resolved) return;
  resolved = true;

  const [a, b] = currentPair;
  const winner = a.playcount > b.playcount ? 'a' : 'b';
  const correct = side === winner;

  // Reveal counts
  ['a','b'].forEach(s => {
    const mask = document.getElementById(`mask-${s}`);
    const val = document.getElementById(`plays-${s}`);
    if (mask) mask.style.display = 'none';
    if (val) val.classList.add('show-count');
  });

  const cardA = $('card-a');
  const cardB = $('card-b');
  cardA.classList.add('resolved');
  cardB.classList.add('resolved');

  if (correct) {
    scoreCorrect++;
    streak++;
    document.getElementById(`card-${side}`).classList.add('correct-answer');
    document.getElementById(`card-${winner === 'a' ? 'b' : 'a'}`).classList.add('wrong-answer');
    const banner = $('result-banner');
    banner.textContent = streak > 2 ? `✓ correct! ${streak} in a row 🔥` : '✓ correct!';
    banner.className = 'result-banner show win';
  } else {
    scoreWrong++;
    streak = 0;
    document.getElementById(`card-${side}`).classList.add('wrong-answer');
    document.getElementById(`card-${winner}`).classList.add('correct-answer');
    const banner = $('result-banner');
    banner.textContent = '✗ not quite';
    banner.className = 'result-banner show lose';
  }

  $('score-correct').textContent = scoreCorrect;
  $('score-wrong').textContent = scoreWrong;
  $('score-streak').textContent = streak;

  $('next-btn').className = 'next-btn show';
}

async function startGame() {
  const user = $('username').value.trim();
  if (!user) { showError('enter a username first'); return; }

  bucketSize = parseInt($('bucket-size').value) || 25;
  minPlays = parseInt($('min-plays').value) || 10;
  bucketSize = Math.max(5, Math.min(100, bucketSize));
  minPlays = Math.max(1, minPlays);

  hideError();
  $('setup').style.display = 'none';
  $('loading').style.display = 'block';

  try {
    $('loading-text').textContent = 'fetching your scrobbles...';
    const raw = await fetchAllTopTracks(user);
    $('loading-text').textContent = 'filtering tracks...';

    tracks = raw
      .map(t => ({ ...t, playcount: parseInt(t.playcount) }))
      .filter(t => t.playcount >= minPlays);

    if (tracks.length < 2) {
      throw new Error(`Only ${tracks.length} track(s) with ≥ ${minPlays} plays. Lower the minimum.`);
    }

    scoreCorrect = 0;
    scoreWrong = 0;
    streak = 0;

    $('loading').style.display = 'none';
    $('game').style.display = 'block';
    $('score-correct').textContent = '0';
    $('score-wrong').textContent = '0';
    $('score-streak').textContent = '0';
    $('pool-info').textContent = `${tracks.length} tracks · ±${bucketSize}% window`;

    showPair();
  } catch (e) {
    $('loading').style.display = 'none';
    $('setup').style.display = 'block';
    showError(e.message || 'something went wrong');
  }
}

function showError(msg) {
  const el = $('error-msg');
  el.textContent = msg;
  el.style.display = 'block';
}

function hideError() {
  $('error-msg').style.display = 'none';
}

$('start-btn').onclick = startGame;

$('username').addEventListener('keydown', e => {
  if (e.key === 'Enter') startGame();
});

$('next-btn').onclick = () => {
  showPair();
};

$('quit-btn').onclick = () => {
  $('game').style.display = 'none';
  $('setup').style.display = 'block';
};
</script>
</body>
</html>
