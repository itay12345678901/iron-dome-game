<!DOCTYPE html>
<html lang="he">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Iron Dome Game</title>
<style>
 body{font-family:Arial,Helvetica,sans-serif;direction:rtl;margin:0;background:#f0f0f0}
 #game-container{position:relative;width:500px;height:1060px;margin:20px auto;
   background:url('https://upload.wikimedia.org/wikipedia/commons/6/6c/Israel_Wikivoyage_map.png') center/contain no-repeat;
   border:1px solid #ccc;overflow:hidden}
 #controls{position:fixed;top:20px;right:20px;width:230px;background:#fff;padding:10px;
   border-radius:8px;box-shadow:0 0 4px #999;text-align:center;z-index:1000}
 #money{font-size:20px;margin-bottom:6px}
 button{width:95%;margin-top:8px;padding:6px 0;border:none;border-radius:4px;
   background:#007bff;color:#fff;cursor:pointer;font-weight:bold}
 #dome-pool-img{width:50px;cursor:grab}
 .map-area{position:absolute;width:50px;height:53px;border:1px solid rgba(255,0,0,.1);
   transition:.2s;background:transparent;cursor:pointer;z-index:50}
 .map-area.alert{background:rgba(255,0,0,.4)}
 .iron-dome{position:absolute;width:50px;height:50px;cursor:grab;z-index:200}
 .coverage-circle{position:absolute;width:200px;height:200px;border-radius:50%;
   border:2px solid rgba(0,150,0,.5);pointer-events:none;z-index:150}
 .missile{position:absolute;width:60px;height:120px;
   background:url('https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSgYkeYBVD6AUpk2hrWxADgBrPy5LLpkigGmQ&s') center/contain no-repeat;
   pointer-events:none;z-index:300;transform-origin:center}
 .missile-target{position:absolute;width:30px;height:30px;border-radius:50%;
   background:rgba(255,0,0,.7);pointer-events:none;z-index:250}
</style>
</head>
<body>

<div id="controls">
 <div id="money">₪10,000</div>
 <img id="dome-img" src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ3U-wZPAFLSrFqAVhwC-XC9R26YwFLvMOD6w&s" draggable="true">
 <div>גרור למפה – ₪1,000</div>
 <button id="clearBtn">נקה אזעקות</button>
 <button id="upgradeBtn">שפר סיכוי יירוט (₪10,000)</button>
 <div id="stats">יירוטים: <span id="int">0</span><br>פגיעות: <span id="hit">0</span></div>
</div>

<div id="game-container"></div>

<audio id="beep" src="https://actions.google.com/sounds/v1/alarms/beep_short.ogg" preload="auto"></audio>
<audio id="boom" src="https://actions.google.com/sounds/v1/explosions/explosion.ogg" preload="auto"></audio>

<script>
/* מצב */
let money=10000, intercepts=0, hits=0, chance=0.70;
const moneyEl=document.getElementById('money'),
      intEl=document.getElementById('int'),hitEl=document.getElementById('hit');
function updateMoney(d=0){money+=d;moneyEl.textContent='₪'+money.toLocaleString();}
function updStats(){intEl.textContent=intercepts;hitEl.textContent=hits;}
updateMoney(0);updStats();

/* לוח אזורים */
const cont=document.getElementById('game-container'), cols=10, rows=20,
      aW=cont.clientWidth/cols,aH=cont.clientHeight/rows, areas=[], activeTargets=[];
for(let r=0;r<rows;r++)for(let c=0;c<cols;c++){
  const a=document.createElement('div');
  a.className='map-area';
  a.style.left=(c*aW)+'px';a.style.top=(r*aH)+'px';
  a.onclick=()=>{
    a.classList.toggle('alert');
    if(a.classList.contains('alert')){
      const t=activeTargets.find(t=>t.area===a && !t.alerted);
      if(t){updateMoney(50);t.alerted=true;}
    }
  };
  cont.appendChild(a);areas.push(a);
}

/* כיפת ברזל */
const domes=[], domeImg=document.getElementById('dome-img');
domeImg.ondragstart=e=>{if(money<1000){e.preventDefault();alert('אין כסף');}else e.dataTransfer.setData('d','1');};
cont.ondragover=e=>e.preventDefault();
cont.ondrop=e=>{
  if(!e.dataTransfer.getData('d'))return;
  const r=cont.getBoundingClientRect();placeDome(e.clientX-r.left,e.clientY-r.top);updateMoney(-1000);
};
function placeDome(x,y){
  const d=document.createElement('img');d.src=domeImg.src;d.className='iron-dome';
  d.style.left=(x-25)+'px';d.style.top=(y-25)+'px';cont.appendChild(d);
  const c=document.createElement('div');c.className='coverage-circle';
  c.style.left=(x-100)+'px';c.style.top=(y-100)+'px';cont.appendChild(c);
  domes.push({x,y,r:100, circ:c});
  d.onmousedown=ev=>{
    const sx=ev.clientX,sy=ev.clientY,ox=d.offsetLeft,oy=d.offsetTop;
    document.onmousemove=mv=>{
      let nx=ox+(mv.clientX-sx),ny=oy+(mv.clientY-sy);
      nx=Math.max(0,Math.min(cont.clientWidth-50,nx));
      ny=Math.max(0,Math.min(cont.clientHeight-50,ny));
      d.style.left=nx+'px';d.style.top=ny+'px';
      const obj=domes.find(o=>o.circ===c);obj.x=nx+25;obj.y=ny+25;
      c.style.left=(obj.x-100)+'px';c.style.top=(obj.y-100)+'px';
    };
    document.onmouseup=()=>document.onmousemove=document.onmouseup=null;
  };
}

/* כפתורים */
document.getElementById('clearBtn').onclick=()=>areas.forEach(a=>a.classList.remove('alert'));
document.getElementById('upgradeBtn').onclick=()=>{
  if(money<10000){alert('אין מספיק כסף');return;}
  updateMoney(-10000);
  chance=chance+(1-chance)/2;chance=Math.min(1,chance);
  alert('הסיכוי ליירוט: '+Math.round(chance*100)+'%');
};

/* טילים */
const beep=document.getElementById('beep'),boom=document.getElementById('boom');
let delay=30000;                                         // מתחיל 30 ש'
function spawn(){
  const area=areas[Math.floor(Math.random()*areas.length)],
        tx=area.offsetLeft+aW/2,ty=area.offsetTop+aH/2;
  const side=['top','bottom','left','right'][Math.floor(Math.random()*4)];
  let sx,sy;
  if(side==='top'){sx=Math.random()*cont.clientWidth;sy=-120;}
  if(side==='bottom'){sx=Math.random()*cont.clientWidth;sy=cont.clientHeight+120;}
  if(side==='left'){sx=-60;sy=Math.random()*cont.clientHeight;}
  if(side==='right'){sx=cont.clientWidth+60;sy=Math.random()*cont.clientHeight;}

  const mis=document.createElement('div');mis.className='missile';
  mis.style.left=(sx-30)+'px';mis.style.top=(sy-60)+'px';
  mis.style.transform=`rotate(${Math.atan2(ty-sy,tx-sx)*180/Math.PI+90}deg)`;cont.appendChild(mis);
  const dot=document.createElement('div');dot.className='missile-target';
  dot.style.left=(tx-15)+'px';dot.style.top=(ty-15)+'px';cont.appendChild(dot);

  activeTargets.push({area,alerted:false});

  requestAnimationFrame(()=>{mis.style.transition=`left 10s linear,top 10s linear`;
    mis.style.left=(tx-30)+'px';mis.style.top=(ty-60)+'px';});

  setTimeout(()=>{
      const protected=domes.some(d=>Math.hypot(d.x-tx,d.y-ty)<=d.r);
      if(protected && Math.random()<chance){intercepts++;updStats();updateMoney(100);beep.currentTime=0;beep.play();}
      else{hits++;updStats();boom.currentTime=0;boom.play();}
      mis.remove();dot.remove();
      const idx=activeTargets.findIndex(o=>o.area===area);if(idx>-1)activeTargets.splice(idx,1);
  },10000);
  setTimeout(spawn,delay);
}
spawn();

/* שינוי קצב לאחר 30 ש' → 5 ש' → 3 ש' → 2 ש' */
setTimeout(()=>delay=5000,30000);
setTimeout(()=>delay=3000,60000);
setTimeout(()=>delay=2000,90000);
</script>
</body>
</html>
