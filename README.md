<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Восьмитресс</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
#vosmi-tress{
  background:#111;
  color:#eee;
  font-family:monospace;
  display:flex;
  justify-content:center;
}
#vosmi-tress .game{
  display:flex;
  flex-direction:column;
  align-items:center;
  padding:10px;
}
#vosmi-tress .field{
  display:grid;
  grid-template-columns:repeat(10,1fr);
  grid-template-rows:repeat(16,1fr);
  width:90vw;
  max-width:260px;
  aspect-ratio:10/16;
  background:#000;
  border:2px solid #555;
}
#vosmi-tress .cell{
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:1rem;
}
#vosmi-tress .guide{color:#333}
#vosmi-tress .controls{
  width:90vw;
  max-width:260px;
  margin-top:8px;
  display:grid;
  grid-template-columns:repeat(3,1fr);
  gap:6px;
}
#vosmi-tress button{
  padding:8px;
  font-size:1rem;
  background:#222;
  color:#eee;
  border:1px solid #555;
  cursor:pointer;
}
#vosmi-tress button:active{background:#444}
#vosmi-tress .panel{
  width:90vw;
  max-width:260px;
  margin-top:8px;
  border:2px solid #555;
  padding:6px;
  font-size:0.9rem;
}
</style>
</head>

<body>
<div id="vosmi-tress">
  <div class="game">
    <div class="field" id="field"></div>

    <div class="controls">
      <button data-a="left">◀</button>
      <button data-a="rot">⟳</button>
      <button data-a="right">▶</button>
      <button data-a="down">▼</button>
      <button data-a="pause">▶⏸</button>
      <button data-a="demo">DEMO</button>
    </div>

    <div class="panel">
      <b>Восьмитресс</b><br>
      Уровень: <span id="lvl">1</span>/5<br>
      Принты: <span id="prt">0.0</span>
    </div>
  </div>
</div>

<script>
(()=>{

/* ===== НАСТРОЙКИ ===== */
const W=10,H=16;
const SPEED=[800,600,450,320,220];

const SHAPES=[
  [["[","]","[","]"]],
  [["{","}"],["{","}"]],
  [["[","]"],["["]]
];

const OCTA=[
{x:3,y:6,m:[[1,1,1,1],[1,0,0,1],[1,0,0,1],[1,1,1,1]]},
{x:2,y:5,m:[[0,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]]},
{x:2,y:5,m:[[0,1,1,1,0],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[0,1,1,1,0]]},
{x:1,y:4,m:[[0,1,1,1,1,0],[1,0,0,0,0,1],[1,0,0,0,0,1],[1,0,0,0,0,1],[0,1,1,1,1,0]]},
{x:1,y:4,m:[[0,1,1,1,1,0],[1,0,0,0,0,1],[1,0,0,0,0,1],[1,0,0,0,0,1],[1,0,0,0,0,1],[0,1,1,1,1,0]]}
];

/* ===== СОСТОЯНИЕ ===== */
const field=document.getElementById('field');
const lvlUI=document.getElementById('lvl');
const prtUI=document.getElementById('prt');

let grid, piece, px, py;
let level=1, prints=0;
let paused=true, demo=false, timer;

/* ===== ИНИЦИАЛИЗАЦИЯ ===== */
function reset(){
  grid=Array.from({length:H},()=>Array(W).fill(""));
  spawn();
}
function spawn(){
  piece=SHAPES[Math.random()*SHAPES.length|0];
  px=3; py=0;
}

/* ===== ЛОГИКА ===== */
const collide=(x,y,p=piece)=>
  p.some((r,dy)=>r.some((v,dx)=>{
    if(!v) return false;
    const X=x+dx,Y=y+dy;
    return X<0||X>=W||Y>=H||(Y>=0&&grid[Y][X]);
  }));

function freeze(){
  piece.forEach((r,dy)=>r.forEach((v,dx)=>{
    if(v) grid[py+dy][px+dx]=v;
  }));
  if(!demo) checkLevel();
  spawn();
}

function rotate(){
  const r=piece[0].map((_,i)=>piece.map(a=>a[i]).reverse());
  if(!collide(px,py,r)) piece=r;
}

function tick(){
  if(paused) return;
  if(!collide(px,py+1)) py++;
  else freeze();
  draw();
}

/* ===== УРОВНИ ===== */
function checkLevel(){
  const o=OCTA[level-1];
  let ok=true;
  o.m.forEach((r,y)=>r.forEach((v,x)=>{
    if(v&&!grid[o.y+y]?.[o.x+x]) ok=false;
  }));
  if(ok){
    prints+=1.5;
    level++;
    if(level>5){
      paused=true;
      clearInterval(timer);
      alert("Восьмитресс пройден!");
      return;
    }
    reset();
    clearInterval(timer);
    timer=setInterval(tick,SPEED[level-1]);
  }
  updateUI();
}

/* ===== ОТРИСОВКА ===== */
function draw(){
  field.innerHTML="";
  const o=OCTA[level-1];
  for(let y=0;y<H;y++)for(let x=0;x<W;x++){
    const d=document.createElement("div");
    d.className="cell";
    if(o?.m[y-o.y]?.[x-o.x]) d.classList.add("guide"),d.textContent="[]";
    if(grid[y][x]) d.textContent=grid[y][x];
    field.appendChild(d);
  }
  piece.forEach((r,dy)=>r.forEach((v,dx)=>{
    if(v){
      const i=(py+dy)*W+(px+dx);
      field.children[i]&&(field.children[i].textContent=v);
    }
  }));
}

const updateUI=()=>{
  lvlUI.textContent=level;
  prtUI.textContent=prints.toFixed(1);
};

/* ===== УПРАВЛЕНИЕ ===== */
document.querySelectorAll("#vosmi-tress button").forEach(b=>{
  b.onclick=()=>{
    const a=b.dataset.a;
    if(a==="left"&&!paused&&!collide(px-1,py)) px--;
    if(a==="right"&&!paused&&!collide(px+1,py)) px++;
    if(a==="down"&&!paused) tick();
    if(a==="rot"&&!paused) rotate();
    if(a==="pause") paused=!paused;
    if(a==="demo"){
      demo=true; paused=false; level=1; prints=0;
      reset(); clearInterval(timer);
      timer=setInterval(tick,120);
    }
    draw();
  };
});

/* ===== СТАРТ ===== */
reset(); updateUI(); draw();
timer=setInterval(tick,SPEED[0]);

})();
</script>
</body>
</html>
