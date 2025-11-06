<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>100 cimarrones dijeron — Plantilla editable</title>
<style>
  :root{
    --bg:#052A6B; --accent:#FFD400; --card:#083E8C;
    --text:#ffffff; --muted:#cfe3ff;
    --shadow: 0 8px 20px rgba(0,0,0,0.4);
    font-family: "Segoe UI", Roboto, Arial, sans-serif;
  }
  body{margin:0;background:linear-gradient(180deg,var(--bg),#02306a);color:var(--text);}
  .container{max-width:1100px;margin:24px auto;padding:20px;}
  .header{display:flex;align-items:center;gap:20px;margin-bottom:18px;}
  .logo{background:var(--accent);color:#022a4f;padding:14px 20px;border-radius:8px;font-weight:700;font-size:20px;box-shadow:var(--shadow);}
  .title{font-size:28px;font-weight:700;}
  .subtitle{color:var(--muted);font-size:14px;}
  .board{background:var(--card);padding:18px;border-radius:12px;box-shadow:var(--shadow);}
  .round-title{display:flex;justify-content:space-between;align-items:center;margin-bottom:12px;}
  .answers{display:grid;grid-template-columns:repeat(3,1fr);gap:12px;}
  .answer{background:rgba(255,255,255,0.06);padding:12px;border-radius:8px;min-height:60px;display:flex;align-items:center;justify-content:center;font-weight:700;font-size:16px;cursor:pointer;user-select:none;position:relative;overflow:hidden;}
  .answer.hidden::after{content:"";position:absolute;inset:0;background:linear-gradient(90deg,rgba(2,42,79,0.65),rgba(2,42,79,0.85));}
  .answer .points{position:absolute;right:8px;top:6px;background:rgba(0,0,0,0.25);padding:4px 8px;border-radius:6px;font-size:12px;}
  .controls{display:flex;gap:8px;margin-top:14px;}
  button{background:var(--accent);border:none;padding:10px 14px;border-radius:8px;font-weight:700;cursor:pointer;}
  .btn-muted{background:#ffffff11;color:var(--muted);font-weight:600;}
  .scoreboard{margin-top:14px;background:#021f45;padding:12px;border-radius:8px;color:var(--muted);}
  .small{font-size:13px;color:var(--muted)}
  footer{margin-top:18px;color:var(--muted);font-size:13px;}
  input.editable{width:100%;background:transparent;border:0;color:var(--text);font-weight:700;font-size:16px;}
  .topbar{display:flex;gap:8px;align-items:center;}
  .round-wrapper{margin-bottom:18px;border-radius:10px;padding:14px;background:linear-gradient(180deg,#0b3a76,#08315f);}
</style>
</head>
<body>
<div class="container">
  <div class="header">
    <div class="logo">100 CIMARRONES DIJERON</div>
    <div>
      <div class="title">Plantilla editable — "100 cimarrones dijeron"</div>
      <div class="subtitle">Edita las preguntas, respuestas y puntos directamente en esta página. Descárgala y súbela a Genially.</div>
    </div>
  </div>

  <div class="board" id="board">
    <div class="round-title">
      <div>
        <div style="font-size:16px;font-weight:700">Ronda <span id="round-number">1</span></div>
        <div class="small">Haz clic en cada recuadro para revelar la respuesta.</div>
      </div>
      <div class="topbar">
        <label class="small">Pregunta:</label>
        <input class="editable" id="question" value="Escribe aquí la pregunta de la ronda 1" />
      </div>
    </div>

    <!-- answers grid -->
    <div class="round-wrapper" id="rounds-container">
      <!-- rounds will be injected by JS -->
    </div>

    <div class="controls">
      <button id="prev" class="btn-muted">Anterior</button>
      <button id="next">Siguiente</button>
      <button id="reset" class="btn-muted">Reiniciar ronda</button>
      <button id="reveal-all" class="btn-muted">Revelar todo</button>
      <div style="flex:1"></div>
      <div class="scoreboard">
        <div>Puntos acumulados: <span id="score">0</span></div>
        <div class="small">Haz clic en una respuesta para sumar sus puntos (editable).</div>
      </div>
    </div>

    <footer>
      Tip: Para personalizar, edita el texto en los cuadros y los valores de puntos. Si lo subes a Genially, inserta este HTML como recurso o copia el código en una nueva página HTML.
    </footer>
  </div>
</div>

<script>
/* ========== Configuración editable ========== */
const NUM_ROUNDS = 5;
const ANSWERS_PER_ROUND = 6;

/* Si quieres, cambia aquí los textos por defecto para cada ronda */
const defaultData = Array.from({length:NUM_ROUNDS}, (_,r)=>({
  question: "Pregunta de ejemplo para la ronda " + (r+1),
  answers: Array.from({length:ANSWERS_PER_ROUND}, (_,i)=>({
    text: "Respuesta " + (i+1),
    points: (ANSWERS_PER_ROUND - i) * 10
  }))
}));

/* ========== SONIDOS (sintetizados con WebAudio) ========== */
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

function playTone(freq, type='sine', duration=0.15, dest=audioCtx.destination, gain=0.18) {
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.type = type;
  o.frequency.value = freq;
  o.connect(g);
  g.connect(dest);
  g.gain.value = gain;
  o.start();
  o.stop(audioCtx.currentTime + duration);
}

/* campana correcta */
function playCorrect() {
  // bell-like arpeggio
  playTone(880,'sine',0.12, audioCtx.destination, 0.16);
  setTimeout(()=>playTone(660,'sine',0.12, audioCtx.destination, 0.14), 80);
  setTimeout(()=>playTone(520,'sine',0.12, audioCtx.destination, 0.12), 160);
}

/* error sound */
function playWrong() {
  // short buzz down
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.type = 'sawtooth';
  o.frequency.setValueAtTime(200, audioCtx.currentTime);
  o.frequency.exponentialRampToValueAtTime(80, audioCtx.currentTime + 0.18);
  g.gain.value = 0.18;
  o.connect(g); g.connect(audioCtx.destination);
  o.start(); o.stop(audioCtx.currentTime + 0.18);
}

/* applause simulation (quick sequence of tones) */
function playApplause() {
  for(let i=0;i<8;i++){
    setTimeout(()=>playTone(600 + Math.random()*400,'square',0.08, audioCtx.destination, 0.08), i*70);
  }
}

/* ========== Lógica de la plantilla ========== */
let currentRound = 0;
let score = 0;
let rounds = JSON.parse(JSON.stringify(defaultData)); // deep copy

const roundsContainer = document.getElementById('rounds-container');
const questionInput = document.getElementById('question');
const roundNumberSpan = document.getElementById('round-number');
const scoreSpan = document.getElementById('score');

function buildRoundElements() {
  roundsContainer.innerHTML = '';
  for(let r=0;r<NUM_ROUNDS;r++){
    const wrapper = document.createElement('div');
    wrapper.className = 'round-wrapper';
    wrapper.style.display = r===currentRound ? 'block' : 'none';
    wrapper.dataset.roundIndex = r;
    // question editable for each round
    const qlabel = document.createElement('div');
    qlabel.style.marginBottom = '10px';
    const qinp = document.createElement('input');
    qinp.className = 'editable';
    qinp.value = rounds[r].question;
    qinp.oninput = (e)=>{ rounds[r].question = e.target.value; if(r===currentRound) questionInput.value = e.target.value; };
    qlabel.appendChild(qinp);
    wrapper.appendChild(qlabel);

    const answersGrid = document.createElement('div');
    answersGrid.className = 'answers';
    rounds[r].answers.forEach((ans, i)=>{
      const a = document.createElement('div');
      a.className = 'answer hidden';
      a.dataset.revealed = 'false';
      a.innerHTML = '<div class="text" contenteditable="true" oninput="this.closest(\'.answer\').dataset.text=this.innerText">'+ans.text+'</div><div class="points" contenteditable="true" oninput="this.closest(\'.answer\').dataset.points=this.innerText">'+ans.points+'</div>';
      // click behavior: reveal and optionally add points
      a.addEventListener('click',(ev)=>{
        const revealed = a.dataset.revealed === 'true';
        if(!revealed){
          revealAnswer(a);
        } else {
          // if already revealed, add points to score (parse int)
          const p = parseInt(a.dataset.points || '0') || 0;
          score += p;
          updateScore();
          playCorrect();
        }
      });
      // store metadata
      a.dataset.text = ans.text;
      a.dataset.points = ans.points;
      answersGrid.appendChild(a);
    });
    wrapper.appendChild(answersGrid);
    roundsContainer.appendChild(wrapper);
  }
}

function revealAnswer(element){
  element.dataset.revealed = 'true';
  element.classList.remove('hidden');
  playCorrect();
}

function revealAllInRound(roundIdx){
  const wrapper = roundsContainer.querySelector(`.round-wrapper[data-round-index="${roundIdx}"]`);
  wrapper.querySelectorAll('.answer').forEach(el=>{
    el.dataset.revealed='true'; el.classList.remove('hidden');
  });
  playApplause();
}

function resetRound(roundIdx){
  const wrapper = roundsContainer.querySelector(`.round-wrapper[data-round-index="${roundIdx}"]`);
  wrapper.querySelectorAll('.answer').forEach(el=>{
    el.dataset.revealed='false'; el.classList.add('hidden');
  });
}

function updateScore(){ scoreSpan.innerText = score; }

document.getElementById('next').addEventListener('click',()=>{
  const old = currentRound;
  currentRound = Math.min(NUM_ROUNDS-1, currentRound+1);
  changeRound(old,currentRound);
});
document.getElementById('prev').addEventListener('click',()=>{
  const old = currentRound;
  currentRound = Math.max(0, currentRound-1);
  changeRound(old,currentRound);
});
document.getElementById('reset').addEventListener('click',()=>{
  resetRound(currentRound);
  playWrong();
});
document.getElementById('reveal-all').addEventListener('click',()=>{
  revealAllInRound(currentRound);
});

function changeRound(oldIdx,newIdx){
  const oldEl = roundsContainer.querySelector(`.round-wrapper[data-round-index="${oldIdx}"]`);
  const newEl = roundsContainer.querySelector(`.round-wrapper[data-round-index="${newIdx}"]`);
  if(oldEl) oldEl.style.display = 'none';
  if(newEl) newEl.style.display = 'block';
  currentRound = newIdx;
  roundNumberSpan.innerText = (currentRound+1);
  // sync question input with round
  questionInput.value = rounds[currentRound].question;
}

questionInput.addEventListener('input',(e)=>{
  rounds[currentRound].question = e.target.value;
  // reflect in round wrapper's editable input
  const wrapper = roundsContainer.querySelector(`.round-wrapper[data-round-index="${currentRound}"]`);
  if(wrapper) wrapper.querySelector('input.editable').value = e.target.value;
});

buildRoundElements();
updateScore();

/* Allow editing of answer text and points by contenteditable; save back to dataset every 700ms */
setInterval(()=>{
  roundsContainer.querySelectorAll('.answer').forEach(a=>{
    const txtEl = a.querySelector('.text');
    const ptsEl = a.querySelector('.points');
    if(txtEl) a.dataset.text = txtEl.innerText.trim();
    if(ptsEl) a.dataset.points = ptsEl.innerText.trim();
  });
},700);

/* Keyboard shortcuts for convenience */
document.addEventListener('keydown',(e)=>{
  if(e.key === 'ArrowRight') document.getElementById('next').click();
  if(e.key === 'ArrowLeft') document.getElementById('prev').click();
  if(e.key === 'r') document.getElementById('reset').click();
});

</script>
</body>
</html>
