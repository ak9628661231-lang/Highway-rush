
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>Arrow Puzzle</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#f4f6fb;color:#202a3b;font-family:Arial,sans-serif}
.app{max-width:520px;margin:auto;padding:20px 16px;min-height:100dvh}
h1{text-align:center;font-size:30px;margin:10px 0}
.sub{text-align:center;color:#718096;margin-bottom:24px}
.stats{display:grid;grid-template-columns:repeat(3,1fr);gap:8px;margin-bottom:20px}
.stat{background:white;border:1px solid #dce3ee;border-radius:14px;padding:12px 4px;text-align:center}
.stat span{display:block;color:#748096;font-size:12px;margin-bottom:5px}
.stat b{font-size:23px}
#board{display:grid;gap:5px;background:#dce4ef;border:3px solid #dce4ef;border-radius:17px;padding:6px;touch-action:manipulation}
.cell{position:relative;aspect-ratio:1;background:#fff;border-radius:9px;min-width:0}
.arrow{width:100%;height:100%;border:0;border-radius:9px;background:#f8fafc;color:#3978db;font-size:clamp(23px,7vw,37px);font-weight:bold;display:flex;align-items:center;justify-content:center;cursor:pointer;transition:transform .18s,opacity .18s}
.arrow:active{transform:scale(.91)}
.arrow.blocked{background:#ffe7e8;color:#e34e62}
.arrow.hinted{outline:3px solid #f6b73c;background:#fff5cf}
.arrow.gone{opacity:0;transform:translate(0,-100%)}
#message{text-align:center;min-height:48px;padding:16px 2px;font-size:15px;color:#526176}
.controls{display:grid;grid-template-columns:1fr 1fr;gap:10px}
button.action{border:0;border-radius:13px;padding:15px 5px;font-size:16px;font-weight:bold;background:#304b70;color:white}
button.action:active{opacity:.8}
#win{display:none;text-align:center;background:#dff8e8;color:#187644;border-radius:14px;padding:16px;margin:14px 0;font-weight:bold}
.help{background:white;border:1px solid #dce3ee;border-radius:14px;padding:14px;line-height:1.55;font-size:14px;color:#66758b;margin-top:20px}
</style>
</head>
<body>
<div class="app">
  <h1>➡️ Arrow Puzzle</h1>
  <div class="sub">Clear the board one arrow at a time!</div>

  <div class="stats">
    <div class="stat"><span>LEVEL</span><b id="level">1</b></div>
    <div class="stat"><span>SCORE</span><b id="score">0</b></div>
    <div class="stat"><span>ARROWS LEFT</span><b id="left">0</b></div>
  </div>

  <div id="board"></div>
  <div id="message">Tap an arrow with a clear path!</div>
  <div id="win">🎉 Level complete!</div>

  <div class="controls">
    <button class="action" id="hint">💡 Hint</button>
    <button class="action" id="restart">↻ Restart</button>
  </div>

  <div class="help">
    <b>How to play</b><br>
    Tap an arrow to move it off the board in the direction it points.
    If another arrow blocks its path, it cannot move.
    Remove every arrow to win!
  </div>
</div>

<script>
const board = document.getElementById("board");
const message = document.getElementById("message");
const win = document.getElementById("win");

let level = 1;
let score = 0;
let arrows = [];
let size = 6;
let busy = false;
let hinted = null;

const dirs = [
  {name:"R",symbol:"➜",dx:1,dy:0},
  {name:"L",symbol:"⬅",dx:-1,dy:0},
  {name:"D",symbol:"⬇",dx:0,dy:1},
  {name:"U",symbol:"⬆",dx:0,dy:-1}
];

function key(x,y){return x+","+y}

function occupied(x,y,list){
  return list.some(a=>a.x===x && a.y===y);
}

function pathClear(a,list){
  let x=a.x+a.dx;
  let y=a.y+a.dy;

  while(x>=0 && x<size && y>=0 && y<size){
    if(occupied(x,y,list)) return false;
    x+=a.dx;
    y+=a.dy;
  }
  return true;
}

/* Har puzzle ka solution generation ke samay hi tay hota hai.
   Naya arrow tabhi add hota hai jab uska exit path khula ho.
   Isliye arrows ko ulte creation order mein nikalna hamesha
   ek valid solution hota hai. */
function makePuzzle(){
  size=6;
  arrows=[];
  hinted=null;
  busy=false;

  const target=Math.min(12+(level-1)*2,30);
  let attempts=0;

  while(arrows.length<target && attempts<10000){
    attempts++;
    const x=Math.floor(Math.random()*size);
    const y=Math.floor(Math.random()*size);

    if(occupied(x,y,arrows)) continue;

    const shuffled=[...dirs].sort(()=>Math.random()-.5);
    let placed=false;

    for(const d of shuffled){
      const a={id:Math.random().toString(36).slice(2),
        x,y,dx:d.dx,dy:d.dy,symbol:d.symbol,name:d.name};

      if(pathClear(a,arrows)){
        arrows.push(a);
        placed=true;
        break;
      }
    }
    if(!placed) continue;
  }

  render();
  message.textContent="Find a free path and tap an arrow!";
  win.style.display="none";
}

function render(){
  board.style.gridTemplateColumns=`repeat(${size},minmax(0,1fr))`;
  board.innerHTML="";

  const cells=new Map();
  arrows.forEach(a=>cells.set(key(a.x,a.y),a));

  for(let y=0;y<size;y++){
    for(let x=0;x<size;x++){
      const cell=document.createElement("div");
      cell.className="cell";

      const a=cells.get(key(x,y));
      if(a){
        const btn=document.createElement("button");
        btn.className="arrow";
        btn.textContent=a.symbol;
        btn.setAttribute("aria-label","Arrow "+a.name);

        if(!pathClear(a,arrows)) btn.classList.add("blocked");
        if(hinted===a.id) btn.classList.add("hinted");

        btn.onclick=()=>tapArrow(a.id);
        cell.appendChild(btn);
      }
      board.appendChild(cell);
    }
  }

  document.getElementById("level").textContent=level;
  document.getElementById("score").textContent=score;
  document.getElementById("left").textContent=arrows.length;
}

function tapArrow(id){
  if(busy) return;

  const a=arrows.find(item=>item.id===id);
  if(!a) return;

  if(!pathClear(a,arrows)){
    message.textContent="🚧 Blocked! Remove the arrow in its path first.";
    const btn=[...board.querySelectorAll(".arrow")]
      .find(b=>b.getAttribute("aria-label")==="Arrow "+a.name);
    if(btn){
      btn.animate(
        [{transform:"translateX(-4px)"},
         {transform:"translateX(4px)"},
         {transform:"translateX(0)"}],
        {duration:180}
      );
    }
    return;
  }

  arrows=arrows.filter(item=>item.id!==id);
  score+=10;
  hinted=null;
  render();
  message.textContent="Nice move! +10 points";

  if(arrows.length===0){
    busy=true;
    win.style.display="block";
    message.textContent="You cleared every arrow!";
    win.textContent="🎉 Level "+level+" complete!";

    const next=document.createElement("button");
    next.className="action";
    next.textContent="Next Level ➜";
    next.style.width="100%";
    next.style.marginTop="12px";
    next.onclick=()=>{
      level++;
      makePuzzle();
      next.remove();
    };
    win.appendChild(next);
  }
}

document.getElementById("hint").onclick=()=>{
  if(busy || arrows.length===0) return;

  /* Reverse creation order always gives a valid move. */
  const a=arrows[arrows.length-1];
  hinted=a.id;
  render();
  message.textContent="💡 Try the highlighted arrow!";
};

document.getElementById("restart").onclick=()=>{
  makePuzzle();
};

makePuzzle();
</script>
</body>
</html>
