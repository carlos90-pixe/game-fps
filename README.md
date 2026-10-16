
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>NEON STRIKE FPS</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#06101f;color:white;font-family:Arial;text-align:center}
header{padding:15px;background:#0d2038;color:#47f5e5}
h1{margin:5px}
.stats{display:flex;justify-content:center;gap:25px;padding:12px}
canvas{width:min(96vw,900px);height:auto;border:2px solid #47f5e5;background:#10253b;cursor:crosshair}
button{padding:12px 24px;background:#47f5e5;border:0;border-radius:8px;font-weight:bold;cursor:pointer;margin:8px}
#msg{color:#47f5e5;font-size:20px}
</style>
</head>
<body>
<header>
<h1>⚡ NEON STRIKE</h1>
<p>FPS — Arena de Treinamento</p>
</header>
<div class="stats">
<div>PONTOS: <b id="score">0</b></div>
<div>ALVOS: <b id="left">10</b></div>
<div>TEMPO: <b id="time">60</b>s</div>
</div>
<canvas id="game" width="900" height="500"></canvas>
<div><button id="start">INICIAR JOGO</button></div>
<p id="msg">Clique em iniciar para jogar!</p>
<p>Use WASD para andar. Mire com o mouse e clique nos drones para acertar.</p>

<script>
const c=document.getElementById('game');
const ctx=c.getContext('2d');
let targets=[],score=0,left=10,time=60,running=false;
let player={x:450,y:250};
let keys={};

function newTarget(){
 return {
  x:40+Math.random()*820,
  y:40+Math.random()*420,
  r:18+Math.random()*10,
  alive:true,
  color:'#ff4d8d'
 };
}

function reset(){
 score=0;left=10;time=60;
 targets=[];
 for(let i=0;i<10;i++)targets.push(newTarget());
 player={x:450,y:250};
 update();
}

function update(){
 document.getElementById('score').textContent=score;
 document.getElementById('left').textContent=left;
 document.getElementById('time').textContent=Math.ceil(time);
}

function draw(){
 ctx.clearRect(0,0,900,500);

 let grad=ctx.createLinearGradient(0,0,0,500);
 grad.addColorStop(0,'#102944');
 grad.addColorStop(1,'#040914');
 ctx.fillStyle=grad;
 ctx.fillRect(0,0,900,500);

 ctx.strokeStyle='#1c5267';
 for(let x=0;x<900;x+=50){
  ctx.beginPath();ctx.moveTo(x,0);ctx.lineTo(x,500);ctx.stroke();
 }
 for(let y=0;y<500;y+=50){
  ctx.beginPath();ctx.moveTo(0,y);ctx.lineTo(900,y);ctx.stroke();
 }

 targets.forEach(t=>{
  if(!t.alive)return;
  ctx.shadowBlur=20;
  ctx.shadowColor=t.color;
  ctx.fillStyle=t.color;
  ctx.beginPath();ctx.arc(t.x,t.y,t.r,0,Math.PI*2);ctx.fill();
  ctx.shadowBlur=0;
  ctx.fillStyle='#fff';
  ctx.beginPath();ctx.arc(t.x,t.y,t.r/3,0,Math.PI*2);ctx.fill();
  ctx.strokeStyle='#47f5e5';
  ctx.lineWidth=3;
  ctx.beginPath();ctx.arc(t.x,t.y,t.r+5,0,Math.PI*2);ctx.stroke();
 });

 // Mira
 ctx.strokeStyle='#47f5e5';
 ctx.lineWidth=2;
 ctx.beginPath();
 ctx.moveTo(438,250);ctx.lineTo(458,250);
 ctx.moveTo(448,240);ctx.lineTo(448,260);
 ctx.stroke();

 // Jogador
 ctx.fillStyle='#47f5e5';
 ctx.fillRect(player.x-10,player.y-10,20,20);

 // Arma
 ctx.fillStyle='#253e56';
 ctx.fillRect(410,450,80,50);
 ctx.fillStyle='#47f5e5';
 ctx.fillRect(438,430,24,40);
}

c.addEventListener('click',e=>{
 if(!running)return;
 const r=c.getBoundingClientRect();
 const x=(e.clientX-r.left)*900/r.width;
 const y=(e.clientY-r.top)*500/r.height;
 let hit=null;
 for(const t of targets){
  if(t.alive&&Math.hypot(t.x-x,t.y-y)<t.r+8){
   hit=t;break;
  }
 }
 if(hit){
  hit.alive=false;
  score+=100;
  left--;
  if(left===0)finish(true);
  update();
 }
 draw();
});

document.addEventListener('keydown',e=>{
 keys[e.key.toLowerCase()]=true;
});
document.addEventListener('keyup',e=>{
 keys[e.key.toLowerCase()]=false;
});

function finish(win){
 running=false;
 document.getElementById('msg').textContent=
 win?'🏆 MISSÃO COMPLETA! Pontuação: '+score:
 '⏰ TEMPO ESGOTADO! Pontuação: '+score;
 document.getElementById('start').textContent='JOGAR NOVAMENTE';
}

function loop(){
 if(!running)return;

 if(keys.w||keys.arrowup)player.y-=3;
 if(keys.s||keys.arrowdown)player.y+=3;
 if(keys.a||keys.arrowleft)player.x-=3;
 if(keys.d||keys.arrowright)player.x+=3;

 player.x=Math.max(12,Math.min(888,player.x));
 player.y=Math.max(12,Math.min(488,player.y));

 time-=1/60;
 if(time<=0){
  time=0;update();finish(false);draw();return;
 }

 update();draw();
 setTimeout(()=>requestAnimationFrame(loop),16);
}

document.getElementById('start').onclick=()=>{
 reset();
 running=true;
 document.getElementById('msg').textContent='🎯 Acerte todos os drones!';
 loop();
};

reset();
draw();
</script>
</body>
</html>
