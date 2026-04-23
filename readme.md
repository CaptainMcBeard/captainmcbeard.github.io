<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Arcane Labyrinth Prototype</title>
<style>
:root{
  --bg:#070707;
  --fg:#d2d2d2;
  --dim:#8e8e8e;
  --line:#242424;
  --line-2:#444;
  --panel:#0d0d0d;
  --panel-2:#111;
  --bright:#f5f5f5;
  --hit:#ffffff;
  --warn:#c4c4c4;
  --miss:#575757;
}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--fg);font-family:Courier New,monospace}
.shell{max-width:1100px;margin:0 auto;padding:18px}
.window{border:2px solid var(--line-2);background:#050505;box-shadow:0 0 0 1px #141414 inset;padding:14px}
.titlebar{display:flex;justify-content:space-between;align-items:center;gap:12px;border:1px solid var(--line-2);background:var(--panel);padding:10px 12px;margin-bottom:12px}
.titleblock{display:flex;flex-direction:column;gap:4px}
.gamename{font-size:20px;color:var(--bright);letter-spacing:1px}
.subtitle{font-size:12px;color:var(--dim)}
button,input{background:#111;color:#ddd;border:1px solid var(--line-2);padding:8px 10px;font-family:inherit;margin-right:6px;margin-bottom:6px}
button:hover:not(:disabled),input:focus{border-color:#777;outline:none}
button:disabled,input:disabled{opacity:.45;cursor:not-allowed}
.grid{display:grid;grid-template-columns:320px 1fr;gap:12px;margin-bottom:12px}
.panel{border:1px solid var(--line-2);background:var(--panel);padding:10px}
.panel-title{font-size:12px;color:var(--dim);letter-spacing:1px;margin-bottom:8px;text-transform:uppercase}
.stats{display:grid;gap:6px;font-size:14px}
.stat-line{display:flex;justify-content:space-between;gap:8px;border-bottom:1px dotted #242424;padding-bottom:4px}
.statusbox{min-height:62px;display:grid;gap:8px}
#status{min-height:24px;padding:8px 10px;border:1px solid #2f2f2f;background:var(--panel-2);color:#f0f0f0;transition:background .18s ease,border-color .18s ease,color .18s ease,box-shadow .18s ease}
#status.dim{color:#9c9c9c}
#status.good{border-color:#666;background:#121212;box-shadow:0 0 0 1px rgba(255,255,255,.04) inset}
#status.bad{border-color:#5a5a5a;background:#101010;color:#f2f2f2}
#status.pulse{animation:statusPulse .28s ease}
#turnSummary{min-height:24px;padding:8px 10px;border:1px solid #1f1f1f;background:#0b0b0b;color:var(--dim)}
.board-panel{border:2px solid var(--bright);background:var(--panel);padding:18px 18px 20px;margin-bottom:18px;box-shadow:0 0 0 1px #1a1a1a inset, 0 0 18px rgba(255,255,255,0.06);position:relative;overflow:hidden;min-height:320px}
.board-panel::before{content:"";position:absolute;inset:0;pointer-events:none;opacity:.16;background-image:linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px),linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px),radial-gradient(circle at center, rgba(255,255,255,0.06), transparent 60%);background-size:24px 24px,24px 24px,100% 100%;background-position:center center;}
#board{position:relative;min-height:250px}
.cell{width:56px;height:56px;border:2px solid #3a3a3a;display:inline-flex;align-items:center;justify-content:center;margin:3px;font-size:22px;font-weight:700;letter-spacing:1px;transition:transform .18s ease,border-color .18s ease,background .18s ease,color .18s ease,box-shadow .18s ease,opacity .18s ease}
.row{display:flex;align-items:center;justify-content:center;margin-bottom:6px}
.cast-row{position:absolute;display:flex;align-items:center;justify-content:center;transition:transform .24s ease,opacity .24s ease,filter .24s ease;--rot:0deg;--scale:1;--blur:0px;--float-x:0px;--float-y:0px;transform:translate(var(--float-x), var(--float-y)) rotate(var(--rot)) scale(var(--scale));}
.cast-row.scattered{opacity:.7;filter:drop-shadow(0 0 6px rgba(255,255,255,0.04)) blur(var(--blur));animation:floatDrift 6s ease-in-out infinite}
.cast-row.latest-solved{opacity:1;filter:drop-shadow(0 0 8px rgba(255,255,255,0.10))}
.green{color:var(--bright);border-color:var(--bright);background:#141414;box-shadow:0 0 8px rgba(255,255,255,.08) inset}
.yellow{color:var(--warn);border-style:dashed;background:#101010}
.gray{color:var(--miss)}
.locked{border-color:#9a9a9a;background:#0d0d0d;box-shadow:0 0 0 1px rgba(255,255,255,0.04) inset}
.active-row{outline:1px solid #777;padding:4px 4px;display:inline-flex;background:rgba(255,255,255,0.025);box-shadow:0 0 12px rgba(255,255,255,0.06);animation:activeRowPulse 1.7s ease-in-out infinite}
.last-row{outline:1px solid #aaa;padding:4px 4px;display:inline-flex;background:rgba(255,255,255,0.04);box-shadow:0 0 14px rgba(255,255,255,0.10)}
.hit-flash{transform:scale(1.08)}
.solve-flash{box-shadow:0 0 18px rgba(255,255,255,.26),0 0 28px rgba(255,255,255,.12) inset;animation:solveSettle .48s ease}
.bank-panel{border:1px solid var(--line-2);background:var(--panel);padding:10px;margin-bottom:12px}
.bank-label{font-size:12px;color:var(--dim);margin-bottom:8px;text-transform:uppercase;letter-spacing:1px}
.bank-grid{display:grid;grid-template-columns:repeat(13, minmax(0, 1fr));gap:4px}
.bank-grid span{display:flex;align-items:center;justify-content:center;height:24px;border:1px solid #232323;background:#090909;color:var(--fg);transition:opacity .12s ease,color .12s ease,transform .12s ease,border-color .12s ease}
.bank-grid .hit{color:var(--bright);border-color:#7b7b7b;background:#111;transform:translateY(-1px)}
.bank-grid .miss{color:var(--miss);border-color:#191919;background:#080808}
.info-row{display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-bottom:12px}
#choices{min-height:36px;padding:8px 10px;border:1px solid var(--line-2);background:var(--panel)}
#controls{border:1px solid var(--line-2);background:var(--panel);padding:10px;margin-bottom:12px}
.summary-panel{border:1px solid var(--line-2);background:var(--panel);padding:12px;margin-bottom:12px;display:none}
.summary-panel.show{display:block}
.summary-title{font-size:18px;color:var(--bright);margin-bottom:8px}
.summary-grid{display:grid;grid-template-columns:1fr 1fr;gap:8px 16px;margin:10px 0}
.summary-line{display:flex;justify-content:space-between;border-bottom:1px dotted #2a2a2a;padding-bottom:4px}
.summary-note{color:var(--dim);margin-top:8px}
#controls .panel-title{margin-bottom:10px}
pre{white-space:pre-wrap;border:1px solid var(--line-2);background:var(--panel);padding:10px;margin:0}
.small{font-size:12px;color:var(--dim)}
.toggle{font-size:12px;padding:6px 8px}
.footer-grid{display:grid;grid-template-columns:2fr 1fr;gap:12px}
.legend{font-size:12px;color:var(--dim);line-height:1.5}
@keyframes statusPulse{0%{transform:scale(1)}50%{transform:scale(1.01)}100%{transform:scale(1)}}
@keyframes activeRowPulse{0%{transform:translateY(0);box-shadow:0 0 10px rgba(255,255,255,0.04)}50%{transform:translateY(-1px);box-shadow:0 0 16px rgba(255,255,255,0.08)}100%{transform:translateY(0);box-shadow:0 0 10px rgba(255,255,255,0.04)}}
@keyframes solveSettle{0%{transform:scale(1.04);filter:brightness(1.15)}100%{transform:scale(1);filter:brightness(1)}}
@keyframes floatDrift{
  0%{transform:translate(0px, 0px) rotate(var(--rot)) scale(var(--scale))}
  50%{transform:translate(2px, -6px) rotate(var(--rot)) scale(var(--scale))}
  100%{transform:translate(0px, 0px) rotate(var(--rot)) scale(var(--scale))}
}
@media (max-width:900px){
  .cell{width:50px;height:50px;font-size:20px}
  .grid,.info-row,.footer-grid,.summary-grid{grid-template-columns:1fr}
  .bank-grid{grid-template-columns:repeat(9, minmax(0, 1fr))}
}
@media (prefers-reduced-motion:reduce){
  .cell,.bank-grid span,#status{transition:none}
  #status.pulse{animation:none}
}
</style>
</head>
<body>
<div class="shell">
  <div class="window">
    <div class="titlebar">
      <div class="titleblock">
        <div class="gamename">ARCANE LABYRINTH</div>
        <div class="subtitle">Apprentice Build / Framed Retro UI</div>
      </div>
      <button id="soundBtn" class="toggle" onclick="toggleSound()">SOUND: ON</button>
    </div>

    <div class="summary-panel" id="summaryPanel">
      <div class="summary-title" id="summaryTitle">RUN COMPLETE</div>
      <div class="summary-grid" id="summaryGrid"></div>
      <div class="summary-note" id="summaryNote"></div>
    </div>

    <div class="grid">
      <div class="panel">
        <div class="panel-title">Run Status</div>
        <div class="stats" id="statsPanel"></div>
      </div>
      <div class="panel statusbox">
        <div class="panel-title">Chamber Feedback</div>
        <div id="status" class="dim"></div>
        <div id="turnSummary">Awaiting your next cast.</div>
      </div>
    </div>

    <div class="board-panel">
      <div class="panel-title">Casting Board</div>
      <div id="board"></div>
    </div>

    <div class="bank-panel">
      <div class="bank-label">Known letters for this chamber</div>
      <div class="bank-grid" id="bank"></div>
    </div>

    <div class="info-row">
      <div id="choices">No active choices.</div>
      <div class="panel legend">
        <div class="panel-title">Legend</div>
        [G] fixed letter / [Y] shifting letter / gray = absent<br>
        Locked row shows stabilized rune slots for your next cast.
      </div>
    </div>

    <div id="controls">
      <div class="panel-title">Actions</div>
      <input id="input" maxlength="5" placeholder="CAST WORD" />
      <button id="castBtn" onclick="guess()">CAST</button>
      <button id="insightBtn" onclick="reveal()">INSIGHT</button>
      <button id="choiceABtn" onclick="choice(0)">CHOICE A</button>
      <button id="choiceBBtn" onclick="choice(1)">CHOICE B</button>
      <button id="continueBtn" onclick="next()">CONTINUE</button>
      <button onclick="reset()">RESET</button>
    </div>

    <div class="footer-grid">
      <div>
        <div class="panel-title">Run Log</div>
        <pre id="log"></pre>
      </div>
      <div>
        <div class="panel-title small">Prototype checks</div>
        <pre id="tests"></pre>
      </div>
    </div>
  </div>
</div>
<script>
const LETTERS = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".split("");
const POOLS = {
  enemy:["CRYPT","SKULL","BLADE","GLOOM"],
  trap:["SPIKE","CHAIN","GLYPH"],
  chest:["CACHE","VAULT","CROWN"],
  boss:["RUNIC","ABYSS"],
  elite:["WRATH","GRAVE","SHADE","HEXED"]
};
const RUN = ["enemy","trap","enemy","chest","elite","rest","boss1","boss2"];
const FEEL_DELAY = 220;
let state;
let audioCtx = null;

function sleep(ms){ return new Promise(resolve => setTimeout(resolve, ms)); }

function ensureAudio(){
  if(!audioCtx){
    const Ctx = window.AudioContext || window.webkitAudioContext;
    if(Ctx) audioCtx = new Ctx();
  }
  if(audioCtx && audioCtx.state === "suspended") audioCtx.resume();
}

function tone(freq, duration, volume = 0.02, type = "square"){
  if(!state || !state.soundEnabled) return;
  ensureAudio();
  if(!audioCtx) return;
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.type = type;
  osc.frequency.value = freq;
  gain.gain.value = volume;
  osc.connect(gain);
  gain.connect(audioCtx.destination);
  const now = audioCtx.currentTime;
  gain.gain.setValueAtTime(volume, now);
  gain.gain.exponentialRampToValueAtTime(0.0001, now + duration);
  osc.start(now);
  osc.stop(now + duration);
}

function playGuessSound(greens, yellows){
  if(greens > 0) tone(520 + greens * 30, 0.08, 0.022);
  if(yellows > 0) setTimeout(() => tone(360 + yellows * 20, 0.06, 0.016), 60);
}
function playBacklashSound(){ tone(180, 0.11, 0.025, "sawtooth"); }
function playSolveSound(){ tone(660, 0.08, 0.02); setTimeout(() => tone(820, 0.1, 0.02), 70); }
function playChoiceSound(){ tone(420, 0.06, 0.015); }

function toggleSound(){
  state.soundEnabled = !state.soundEnabled;
  document.getElementById("soundBtn").textContent = `SOUND: ${state.soundEnabled ? "ON" : "OFF"}`;
  if(state.soundEnabled) tone(520, 0.05, 0.014);
}

function setBusy(v){ state.busy = v; renderControls(); }

function setStatus(text, mode = "dim"){
  const el = document.getElementById("status");
  el.textContent = text;
  el.className = mode;
  void el.offsetWidth;
  el.classList.add("pulse");
}

function setTurnSummary(text){ document.getElementById("turnSummary").textContent = text; }

function reset(){
  state = {
    hp:20,
    runes:1,
    node:0,
    log:[],
    buffs:[],
    choices:null,
    enc:null,
    letterState:{},
    gameWon:false,
    busy:false,
    lastGuessIndex:-1,
    soundEnabled:true,
    lastResolution:null,
    summary:{
      guessesCast:0,
      insightUses:0,
      chambersCleared:0,
      damageTaken:0,
      result:null
    }
  };
  LETTERS.forEach(l => state.letterState[l] = "unused");
  startNode();
  document.getElementById("soundBtn").textContent = "SOUND: ON";
  renderSummary();
}

function chamberLabel(type){
  if(type === "enemy") return "Hostile sigil";
  if(type === "trap") return "Trap sigil";
  if(type === "chest") return "Sealed cache";
  if(type === "elite") return "Elite ward";
  if(type === "rest") return "Sanctuary";
  if(type === "boss1") return "Master ward";
  if(type === "boss2") return "True name";
  return type;
}

function chamberDisplayName(type){
  if(type === "enemy") return "Enemy";
  if(type === "trap") return "Trap";
  if(type === "chest") return "Chest";
  if(type === "elite") return "Elite";
  if(type === "rest") return "Rest";
  if(type === "boss1") return "Boss I";
  if(type === "boss2") return "Boss II";
  return type;
}

function resetLetterBank(){ LETTERS.forEach(l => state.letterState[l] = "unused"); }

function finalizeRun(success){
  state.gameWon = success;
  state.summary.result = success ? "Victory" : "Defeat";
  renderSummary();
}

function renderSummary(){
  const panel = document.getElementById("summaryPanel");
  const title = document.getElementById("summaryTitle");
  const grid = document.getElementById("summaryGrid");
  const note = document.getElementById("summaryNote");

  if(!state || !state.summary.result){
    panel.classList.remove("show");
    grid.innerHTML = "";
    note.textContent = "";
    return;
  }

  panel.classList.add("show");
  title.textContent = state.summary.result === "Victory" ? "RUN COMPLETE" : "RUN ENDED";
  grid.innerHTML = `
    <div class="summary-line"><span>Result</span><span>${state.summary.result}</span></div>
    <div class="summary-line"><span>HP Remaining</span><span>${state.hp}</span></div>
    <div class="summary-line"><span>Guesses Cast</span><span>${state.summary.guessesCast}</span></div>
    <div class="summary-line"><span>Insight Uses</span><span>${state.summary.insightUses}</span></div>
    <div class="summary-line"><span>Chambers Cleared</span><span>${state.summary.chambersCleared}</span></div>
    <div class="summary-line"><span>Damage Taken</span><span>${state.summary.damageTaken}</span></div>
  `;
  note.textContent = state.summary.result === "Victory"
    ? "You reached the end of the labyrinth and completed the current vertical slice."
    : "The run ended before the labyrinth was cleared. Reset to begin another attempt.";
}

function startNode(){
  const type = RUN[state.node];
  let word = null;
  if(type.includes("boss")) word = POOLS.boss[Math.random() * POOLS.boss.length | 0];
  else if(type === "elite") word = POOLS.elite[Math.random() * POOLS.elite.length | 0];
  else if(type === "rest") word = null;
  else word = POOLS[type]?.[Math.random() * POOLS[type].length | 0] ?? null;

  resetLetterBank();

  state.enc = { type, word, guesses:[], locked:[null,null,null,null,null], done:false, max:(type === "trap" ? 4 : type === "chest" ? 3 : 6) };
  state.lastGuessIndex = -1;
  state.lastResolution = null;
  state.log.push("-- " + type.toUpperCase() + " CHAMBER --");

  if(type === "rest"){
    state.log.push("A quiet sanctuary. Choose to recover vitality or restore insight.");
    state.enc.done = true;
    state.choices = ["Heal","Rune"];
    setStatus("Sanctuary found. Choose how to prepare.", "good");
    setTurnSummary("No casting needed here. Resolve your recovery choice.");
  } else {
    setStatus(chamberLabel(type) + ". Weave a five-letter incantation.", "dim");
    setTurnSummary("Awaiting cast.");
  }

  render();
  focusInput();
}

function focusInput(){ const input = document.getElementById("input"); if(!input.disabled) input.focus(); }

function evalGuess(g,a){
  const r = Array(5).fill(".");
  const used = [0,0,0,0,0];
  for(let i=0;i<5;i++) if(g[i] === a[i]){ r[i] = "G"; used[i] = 1; }
  for(let i=0;i<5;i++){
    if(r[i] === "G") continue;
    for(let j=0;j<5;j++) if(!used[j] && g[i] === a[j]){ r[i] = "Y"; used[j] = 1; break; }
  }
  return r;
}

function updateLetterBank(g,r){
  for(let i=0;i<5;i++){
    const l = g[i];
    if(r[i] === "G" || r[i] === "Y") state.letterState[l] = "hit";
    else if(state.letterState[l] === "unused") state.letterState[l] = "miss";
  }
}

function applyDamage(amount){
  if(amount <= 0) return;
  state.hp -= amount;
  state.summary.damageTaken += amount;
}

async function guess(){
  const input = document.getElementById("input");
  const g = input.value.toUpperCase().replace(/[^A-Z]/g, "");
  if(g.length !== 5 || state.enc.done || !state.enc.word || state.busy) return;

  for(let i=0;i<5;i++){
    if(state.enc.locked[i] && g[i] !== state.enc.locked[i]){
      state.log.push("Locked rune mismatch at slot " + (i + 1) + ".");
      setStatus("That rune slot is locked. Match the revealed letter.", "bad");
      setTurnSummary("Revise the cast to match your fixed rune slots.");
      render();
      return;
    }
  }

  setBusy(true);
  state.summary.guessesCast += 1;
  const r = evalGuess(g, state.enc.word);
  state.enc.guesses.push({g,r});
  state.lastGuessIndex = state.enc.guesses.length - 1;
  updateLetterBank(g,r);
  for(let i=0;i<5;i++) if(r[i] === "G") state.enc.locked[i] = g[i];
  state.log.push("Weave: " + g + " " + r.join(""));

  const greens = r.filter(x => x === "G").length;
  const yellows = r.filter(x => x === "Y").length;
  const misses = 5 - greens - yellows;
  state.lastResolution = {greens, yellows, misses};
  playGuessSound(greens, yellows);
  setStatus(`Incantation resolves: ${greens} fixed, ${yellows} shifting.`, greens > 0 || yellows > 0 ? "good" : "bad");
  setTurnSummary(`Result: ${greens} green, ${yellows} yellow, ${misses} gray.`);
  render();
  await sleep(FEEL_DELAY);

  if(g === state.enc.word){
    state.log.push("INCANTATION COMPLETE");
    state.enc.done = true;
    state.summary.chambersCleared += 1;
    playSolveSound();
    if(state.enc.type === "chest"){
      state.choices = ["Heal","Rune"];
      setStatus("The cache opens. Choose your reward.", "good");
      setTurnSummary("Successful cast. Claim your reward before proceeding.");
    } else {
      setStatus("Spell complete. The chamber yields.", "good");
      setTurnSummary("Successful cast. Proceed when ready.");
    }
    render();
    input.value = "";
    setBusy(false);
    focusInput();
    return;
  }

  if(state.enc.type === "enemy" || state.enc.type === "elite" || state.enc.type.includes("boss")){
    const hits = r.filter(x => x !== ".").length;
    let dmg = [4,3,3,2,1][hits] || 0;
    if(state.enc.type === "elite") dmg += 1;
    applyDamage(dmg);
    state.log.push("Backlash: " + dmg);
    playBacklashSound();
    setStatus("Arcane backlash strikes for " + dmg + ".", "bad");
    setTurnSummary(`You connected ${hits} letters. Backlash dealt ${dmg} damage.`);
    render();
    await sleep(FEEL_DELAY);
  }

  if(state.enc.type === "trap"){
    applyDamage(1);
    state.log.push("Trap lashes out: 1");
    playBacklashSound();
    setStatus("The trap bites back for 1.", "bad");
    setTurnSummary("Trap pressure is constant. Find the pattern quickly.");
    render();
    await sleep(FEEL_DELAY);
  }

  if(state.hp <= 0){
    state.hp = 0;
    state.log.push("YOU FALL");
    state.enc.done = true;
    setStatus("Your spell breaks. You fall in the labyrinth.", "bad");
    setTurnSummary("Run ended.");
    finalizeRun(false);
  }

  if(state.enc.guesses.length >= state.enc.max && !state.enc.done){
    state.log.push("FAILED: " + state.enc.word);
    state.enc.done = true;
    setStatus("The chamber rejects you. The word was " + state.enc.word + ".", "bad");
    setTurnSummary("Encounter failed. Continue if the run still stands.");
  }

  if(!state.enc.done) setStatus("Adjust your casting and try again.", "dim");

  input.value = "";
  render();
  setBusy(false);
  focusInput();
}

function reveal(){
  if(state.runes <= 0 || state.enc.done || !state.enc.word || state.busy) return;
  const i = state.enc.locked.findIndex(x => !x);
  if(i >= 0){
    state.enc.locked[i] = state.enc.word[i];
    state.runes--;
    state.summary.insightUses += 1;
    state.log.push("Insight reveals slot " + (i + 1) + ": " + state.enc.word[i]);
    playChoiceSound();
    setStatus("Insight steadies the pattern at slot " + (i + 1) + ".", "good");
    setTurnSummary("A locked rune has been added to your active row.");
  }
  render();
  focusInput();
}

function choice(i){
  if(!state.choices || state.busy) return;
  const c = state.choices[i];
  if(!c) return;
  state.log.push("Chosen: " + c);
  if(c === "Heal") state.hp = Math.min(20, state.hp + 5);
  if(c === "Rune") state.runes++;
  state.choices = null;
  playChoiceSound();
  setStatus(c === "Heal" ? "Vitality restored." : "Insight restored.", "good");
  setTurnSummary("Choice resolved. Continue to the next chamber.");
  render();
  focusInput();
}

function next(){
  if(!state.enc.done || state.busy) return;
  state.node++;
  if(state.node >= RUN.length){
    state.log.push("MASTER SAVED");
    playSolveSound();
    setStatus("Your master is saved. The labyrinth falls silent.", "good");
    setTurnSummary("Run complete.");
    finalizeRun(true);
    render();
    return;
  }
  startNode();
}

function renderControls(){
  const doneWithRun = state.node >= RUN.length || state.gameWon || !!state.summary.result;
  const hasChoices = !!state.choices;
  document.getElementById("input").disabled = state.enc.type === "rest" || doneWithRun || state.busy || hasChoices;
  document.getElementById("castBtn").disabled = state.enc.type === "rest" || doneWithRun || state.busy || hasChoices;
  document.getElementById("insightBtn").disabled = state.runes <= 0 || !state.enc.word || state.enc.done || state.busy || hasChoices || !!state.summary.result;
  document.getElementById("choiceABtn").disabled = !hasChoices || state.busy;
  document.getElementById("choiceBBtn").disabled = !hasChoices || state.busy;
  document.getElementById("continueBtn").disabled = !state.enc.done || hasChoices || state.busy || doneWithRun;
}

function render(){
  document.getElementById("statsPanel").innerHTML = `
    <div class="stat-line"><span>HP</span><span>${state.hp}</span></div>
    <div class="stat-line"><span>Runes</span><span>${state.runes}</span></div>
    <div class="stat-line"><span>Node</span><span>${Math.min(state.node + 1, RUN.length)} / ${RUN.length}</span></div>
    <div class="stat-line"><span>Chamber</span><span>${chamberDisplayName(state.enc.type)}</span></div>
  `;

  const b = document.getElementById("board");
  b.innerHTML = "";

  const boardWidth = b.clientWidth || 700;
  const rowWidth = (56 * 5) + (3 * 2 * 5) + 8;
  const centerLeft = Math.max(0, Math.round((boardWidth - rowWidth) / 2));
  const topBase = 26;
  const scatterSlots = [
    {x:-160,y:18,rot:-1.5},
    {x:150,y:-6,rot:1.2},
    {x:-210,y:88,rot:-0.8},
    {x:185,y:98,rot:1.6},
    {x:0,y:-34,rot:0.4},
    {x:-120,y:148,rot:-1.2}
  ];

  state.enc.guesses.forEach((row, idx) => {
    const d = document.createElement("div");
    const isLatest = idx === state.lastGuessIndex;
    const isSolvedLatest = state.enc.done && isLatest && row.g === state.enc.word;

    // Age factor for scattered rows (0 = newest previous cast, 1 = oldest cast)
    const age = state.enc.guesses.length > 1 ? (state.enc.guesses.length - 1 - idx) / (state.enc.guesses.length - 1) : 0;

    d.className = "cast-row " + (isSolvedLatest ? "last-row latest-solved" : "scattered");

    const slot = scatterSlots[idx % scatterSlots.length];
    let left = centerLeft + slot.x;
    let top = topBase + slot.y;
    let rotate = slot.rot;

    if(isSolvedLatest){
      left = centerLeft;
      top = 150;
      rotate = 0;
    }

    // Apply scaling + fade based on age (older = smaller + dimmer + further out)
    const scale = isSolvedLatest ? 1 : 0.96 - (age * 0.46);
    const opacity = isSolvedLatest ? 1 : 0.92 - (age * 0.5);
    const drift = isSolvedLatest ? 0 : 24 + (age * 44);
    const blur = isSolvedLatest ? 0 : 0.5 + (age * 2.2);

    d.style.left = `${left + (slot.x > 0 ? drift : -drift)}px`;
    d.style.top = `${top + (slot.y > 0 ? drift * 0.6 : -drift * 0.6)}px`;
    d.style.setProperty("--rot", `${rotate}deg`);
    d.style.setProperty("--scale", `${scale}`);
    d.style.setProperty("--blur", `${blur}px`);
    d.style.opacity = opacity;
    d.style.zIndex = isSolvedLatest ? "20" : `${10 - idx}`;

    row.g.split("").forEach((c, i) => {
      const cell = document.createElement("div");
      cell.className = "cell" + (row.r[i] === "G" ? " green hit-flash" : row.r[i] === "Y" ? " yellow hit-flash" : " gray");
      if(isSolvedLatest) cell.className += " solve-flash";
      cell.textContent = c;
      d.appendChild(cell);
    });
    b.appendChild(d);
  });

  if(!state.enc.done && state.enc.word && !state.summary.result){
    const active = document.createElement("div");
    active.className = "cast-row active-row";
    active.style.left = `${centerLeft}px`;
    active.style.top = `150px`;
    active.style.setProperty("--rot", "0deg");
    active.style.setProperty("--scale", "1");
    for(let i=0;i<5;i++){
      const cell = document.createElement("div");
      cell.className = "cell" + (state.enc.locked[i] ? " locked" : " gray");
      cell.textContent = state.enc.locked[i] || "_";
      active.appendChild(cell);
    }
    b.appendChild(active);
  }

  const bank = document.getElementById("bank");
  bank.innerHTML = "";
  LETTERS.forEach(l => {
    const span = document.createElement("span");
    span.textContent = l;
    const s = state.letterState[l];
    if(s === "hit") span.className = "hit";
    else if(s === "miss") span.className = "miss";
    bank.appendChild(span);
  });

  const choices = document.getElementById("choices");
  choices.textContent = state.choices ? `CHOICE A: ${state.choices[0]}   |   CHOICE B: ${state.choices[1]}` : "No active choices.";

  document.getElementById("log").textContent = state.log.join("\n");
  renderControls();
  renderSummary();
}

function cloneStateForTests(src){
  return {
    hp:src.hp, runes:src.runes, node:src.node, log:[...src.log], buffs:[...src.buffs], choices:src.choices ? [...src.choices] : null,
    enc:src.enc ? { type:src.enc.type, word:src.enc.word, guesses:src.enc.guesses.map(g => ({ g:g.g, r:[...g.r] })), locked:[...src.enc.locked], done:src.enc.done, max:src.enc.max } : null,
    letterState:{...src.letterState}, gameWon:src.gameWon, busy:src.busy, lastGuessIndex:src.lastGuessIndex, soundEnabled:src.soundEnabled,
    lastResolution:src.lastResolution ? {...src.lastResolution} : null,
    summary:{...src.summary}
  };
}

function runTests(){
  const results = [];
  function test(name, fn){ try{ fn(); results.push("PASS: " + name); }catch(err){ results.push("FAIL: " + name + " -> " + err.message); } }
  function assert(condition, message){ if(!condition) throw new Error(message); }

  const savedState = cloneStateForTests(state);

  test("evalGuess marks full match as all greens", () => {
    const r = evalGuess("CRYPT", "CRYPT");
    assert(r.join("") === "GGGGG", "expected GGGGG, got " + r.join(""));
  });

  test("evalGuess handles mixed matches", () => {
    const r = evalGuess("CARVE", "CRYPT");
    assert(r[0] === "G", "expected first letter green");
    assert(r.includes("Y") || r.includes("."), "expected mixed result");
  });

  test("startNode gives elite a word", () => {
    const prior = cloneStateForTests(state);
    state = {hp:20,runes:1,node:4,log:[],buffs:[],choices:null,enc:null,letterState:{},gameWon:false,busy:false,lastGuessIndex:-1,soundEnabled:false,lastResolution:null,summary:{guessesCast:0,insightUses:0,chambersCleared:0,damageTaken:0,result:null}};
    LETTERS.forEach(l => state.letterState[l] = "unused");
    startNode();
    assert(typeof state.enc.word === "string" && state.enc.word.length === 5, "elite word missing");
    state = prior;
  });

  test("startNode makes rest a choice chamber with no word", () => {
    const prior = cloneStateForTests(state);
    state = {hp:20,runes:1,node:5,log:[],buffs:[],choices:null,enc:null,letterState:{},gameWon:false,busy:false,lastGuessIndex:-1,soundEnabled:false,lastResolution:null,summary:{guessesCast:0,insightUses:0,chambersCleared:0,damageTaken:0,result:null}};
    LETTERS.forEach(l => state.letterState[l] = "unused");
    startNode();
    assert(state.enc.word === null, "rest should not have a word");
    assert(state.enc.done === true, "rest should be immediately done");
    assert(Array.isArray(state.choices) && state.choices.length === 2, "rest choices missing");
    state = prior;
  });

  test("boss nodes always have valid words", () => {
    const prior = cloneStateForTests(state);
    state = {hp:20,runes:1,node:6,log:[],buffs:[],choices:null,enc:null,letterState:{},gameWon:false,busy:false,lastGuessIndex:-1,soundEnabled:false,lastResolution:null,summary:{guessesCast:0,insightUses:0,chambersCleared:0,damageTaken:0,result:null}};
    LETTERS.forEach(l => state.letterState[l] = "unused");
    startNode();
    assert(typeof state.enc.word === "string" && state.enc.word.length === 5, "boss word missing");
    state.node = 7;
    startNode();
    assert(typeof state.enc.word === "string" && state.enc.word.length === 5, "boss phase 2 word missing");
    state = prior;
  });

  test("reveal locks a letter and consumes a rune", () => {
    const prior = cloneStateForTests(state);
    state = {hp:20,runes:1,node:0,log:[],buffs:[],choices:null,enc:{type:"enemy",word:"CRYPT",guesses:[],locked:[null,null,null,null,null],done:false,max:6},letterState:{},gameWon:false,busy:false,lastGuessIndex:-1,soundEnabled:false,lastResolution:null,summary:{guessesCast:0,insightUses:0,chambersCleared:0,damageTaken:0,result:null}};
    LETTERS.forEach(l => state.letterState[l] = "unused");
    reveal();
    assert(state.runes === 0, "rune should be consumed");
    assert(state.enc.locked.some(Boolean), "a locked slot should be revealed");
    assert(state.summary.insightUses === 1, "insight use should be counted");
    state = prior;
  });

  test("reset clears leaked locked letters and letter bank state", () => {
    const prior = cloneStateForTests(state);
    state.enc.locked = ["C", null, null, null, null];
    state.letterState.C = "hit";
    reset();
    assert(state.enc.locked.every(v => v === null), "locked letters should reset");
    assert(state.letterState.C === "unused", "letter bank should reset to unused");
    state = prior;
    render();
  });

  test("startNode resets letter bank for a new chamber", () => {
    const prior = cloneStateForTests(state);
    state = {hp:20,runes:1,node:0,log:[],buffs:[],choices:null,enc:null,letterState:{},gameWon:false,busy:false,lastGuessIndex:-1,soundEnabled:false,lastResolution:null,summary:{guessesCast:0,insightUses:0,chambersCleared:0,damageTaken:0,result:null}};
    LETTERS.forEach(l => state.letterState[l] = "unused");
    state.letterState.A = "miss";
    state.letterState.C = "hit";
    startNode();
    assert(state.letterState.A === "unused", "new chamber should clear prior miss state");
    assert(state.letterState.C === "unused", "new chamber should clear prior hit state");
    state = prior;
  });

  test("finalizeRun records a visible summary result", () => {
    const prior = cloneStateForTests(state);
    state.summary.result = null;
    finalizeRun(true);
    assert(state.summary.result === "Victory", "summary result should be Victory");
    state = prior;
  });

  test("unfinished encounter keeps only the live input row centered", () => {
    const prior = cloneStateForTests(state);
    state = {hp:20,runes:1,node:0,log:[],buffs:[],choices:null,enc:{type:"enemy",word:"GLOOM",guesses:[{g:"CRYPT",r:[".",".",".",".","."]}],locked:["G",null,null,null,null],done:false,max:6},letterState:{},gameWon:false,busy:false,lastGuessIndex:0,soundEnabled:false,lastResolution:null,summary:{guessesCast:1,insightUses:0,chambersCleared:0,damageTaken:0,result:null}};
    LETTERS.forEach(l => state.letterState[l] = "unused");
    render();
    const activeRows = document.querySelectorAll('#board .active-row');
    assert(activeRows.length === 1, "should render exactly one active row");
    const scatteredRows = document.querySelectorAll('#board .scattered');
    assert(scatteredRows.length === 1, "previous guess should remain scattered");
    state = prior;
  });

  state = savedState;
  document.getElementById("tests").textContent = results.join("\n");
  render();
}

reset();
runTests();
</script>
</body>
</html>
