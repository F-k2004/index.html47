<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<title>📡 Spacecraft Communication Network</title>
<style>
html,body{margin:0;overflow:hidden;background:#00030a;font-family:system-ui}
canvas{display:block}
.hud{
  position:absolute;left:16px;top:16px;
  padding:12px 16px;border-radius:14px;
  background:rgba(255,255,255,0.06);
  backdrop-filter:blur(10px);
  color:#d9f3ff;
  font-size:13px;
}
<style>
</head>
<body>

<canvas id="c"></canvas>
<div class="hud" id="hud"></div>

<script>
const c=document.getElementById("c");
const ctx=c.getContext("2d");
let w,h;
function resize(){w=c.width=innerWidth;h=c.height=innerHeight}
resize();addEventListener("resize",resize);

// Earth
const earth={x:w/2,y:h/2,r:60,mu:9000};

// Fleet
const targetAlt=180;
const maxLinkDist=160; // communication range
const colors=["#9ff0ff","#ffd29f","#c3ff9f","#ff9fd2"];
const fleet=[];

// Create spacecraft
for(let i=0;i<4;i++){
  fleet.push({
    x:earth.x,
    y:earth.y-(targetAlt+i*12),
    vx:2+i*0.12,
    vy:0,
    battery:100,
    integral:0,
    prevErr:0,
    color:colors[i]
  });
}

// PID constants
const Kp=0.015, Ki=0.0001, Kd=0.04;

// Gravity
function gravity(s){
  const dx=earth.x-s.x, dy=earth.y-s.y;
  const d=Math.hypot(dx,dy);
  const f=earth.mu/(d*d);
  s.vx+=f*dx/d;
  s.vy+=f*dy/d;
}

// Autopilot
function autopilot(s){
  const dx=s.x-earth.x, dy=s.y-earth.y;
  const dist=Math.hypot(dx,dy);
  const alt=dist-earth.r;

  const err=targetAlt-alt;
  s.integral+=err;
  const der=err-s.prevErr;
  s.prevErr=err;

  let out=Kp*err+Ki*s.integral+Kd*der;
  out=Math.max(-0.05,Math.min(0.05,out));

  const tx=-dy/dist, ty=dx/dist;
  s.vx+=tx*out;
  s.vy+=ty*out;

  s.battery-=Math.abs(out)*0.5;
  s.battery=Math.max(0,s.battery);
}

// Communication links
function drawLinks(){
  for(let i=0;i<fleet.length;i++){
    for(let j=i+1;j<fleet.length;j++){
      const a=fleet[i], b=fleet[j];
      const d=Math.hypot(a.x-b.x,a.y-b.y);
      if(d<maxLinkDist){
        ctx.strokeStyle=`rgba(120,220,255,${1-d/maxLinkDist})`;
        ctx.lineWidth=1;
        ctx.beginPath();
        ctx.moveTo(a.x,a.y);
        ctx.lineTo(b.x,b.y);
        ctx.stroke();
      }
    }
  }
}

// Update
function update(){
  fleet.forEach(s=>{
    gravity(s);
    autopilot(s);
    s.x+=s.vx;
    s.y+=s.vy;
  });
}

// Draw
function draw(){
  ctx.fillStyle="rgba(0,3,10,0.35)";
  ctx.fillRect(0,0,w,h);

  update();

  // Links
  drawLinks();

  // Earth
  ctx.beginPath();
  ctx.arc(earth.x,earth.y,earth.r,0,Math.PI*2);
  ctx.fillStyle="#0b3d91";
  ctx.fill();

  // Fleet
  fleet.forEach(s=>{
    ctx.beginPath();
    ctx.arc(s.x,s.y,4,0,Math.PI*2);
    ctx.fillStyle=s.color;
    ctx.fill();
  });

  // HUD
  let links=0;
  for(let i=0;i<fleet.length;i++){
    for(let j=i+1;j<fleet.length;j++){
      if(Math.hypot(fleet[i].x-fleet[j].x,fleet[i].y-fleet[j].y)<maxLinkDist){
        links++;
      }
    }
  }

  document.getElementById("hud").innerHTML=
    `📡 Active Links: ${links}<br>`+
    fleet.map((s,i)=>`🛰️ Ship ${i+1}: 🔋 ${s.battery.toFixed(0)}%`).join("<br>");

  requestAnimationFrame(draw);
}

draw();
</script>
</body>
</html>
