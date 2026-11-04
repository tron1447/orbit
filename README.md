[index (7).html](https://github.com/user-attachments/files/34391401/index.7.html)
<!DOCTYPE html>
<html lang="et">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Zombie Defender MR</title>
<style>
:root{--bg:#0b1413;--fg:#e4f0ea;--ac:#6fd6b0;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme: dark){:root:not([data-theme="light"]){--bg:#0b1413;--fg:#e4f0ea;--ac:#6fd6b0}}
:root[data-theme="dark"]{--bg:#0b1413;--fg:#e4f0ea;--ac:#6fd6b0}
html,body{height:100%;margin:0;background:var(--bg);color:var(--fg);font-family:system-ui,sans-serif;overflow:hidden}
canvas{display:block}
#ui{position:fixed;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:12px;text-align:center;padding:24px;background:rgba(6,12,11,.88)}
#ui h1{margin:0;font-size:2rem;color:var(--ac)}
#ui p{margin:0;max-width:34rem;opacity:.85}
button{font:inherit;font-weight:600;padding:12px 22px;border:0;border-radius:10px;background:var(--ac);color:#06110e;cursor:pointer}
button.alt{background:transparent;color:var(--fg);border:1px solid var(--fg)}
button:disabled{opacity:.4}
</style>
</head>
<body>
<div id="ui">
  <h1>Zombie Defender MR</h1>
  <p>Pimedas toas ärkavad surnud. Võta taskulamp ja pistol, tulista pihta. Pähe tabamus on kahekordne. Jooksjad on kiired, tankid kannatavad palju. Elusid on 5.</p>
  <button id="mr" disabled>Mixed reality (kontrollin…)</button>
  <button id="flat" class="alt">Mängi tavarežiimis</button>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const renderer = new THREE.WebGLRenderer({antialias:true, alpha:true});
renderer.setPixelRatio(devicePixelRatio); renderer.setSize(innerWidth, innerHeight);
renderer.xr.enabled = true; renderer.xr.setReferenceSpaceType('local-floor');
document.body.prepend(renderer.domElement);

const scene = new THREE.Scene();
scene.fog = new THREE.FogExp2(0x2c4a47, .09);
const camera = new THREE.PerspectiveCamera(70, innerWidth/innerHeight, 0.05, 50);
camera.position.set(0,1.6,0); scene.add(camera);
scene.add(new THREE.HemisphereLight(0x9fd8cc, 0x101a14, .4));

const floor = new THREE.Mesh(new THREE.CircleGeometry(8,48), new THREE.MeshLambertMaterial({color:0x1c2b27}));
floor.rotation.x = -Math.PI/2; scene.add(floor);
const dim = new THREE.Mesh(new THREE.SphereGeometry(15,16,12), new THREE.MeshBasicMaterial({color:0x020a08,transparent:true,opacity:.5,side:THREE.BackSide,depthWrite:false}));
dim.renderOrder = -1; dim.visible = false; scene.add(dim);

// Pistol (Glock-laadne, barrel piki -Z)
const blk = new THREE.MeshLambertMaterial({color:0x151515}), gry = new THREE.MeshLambertMaterial({color:0x2b2b2b});
function bx(w,h,d,m,x,y,z,rx){ const b=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),m); b.position.set(x,y,z); if(rx) b.rotation.x=rx; return b; }
function makeGun(){
  const g = new THREE.Group();
  g.add(bx(.032,.036,.2,blk,0,.02,-.07));
  g.add(bx(.03,.03,.17,gry,0,-.012,-.055));
  g.add(bx(.034,.1,.045,gry,0,-.06,.02,.2));
  g.add(bx(.02,.012,.06,blk,0,-.035,-.03));
  g.add(bx(.008,.012,.012,blk,0,.044,-.15));
  g.add(bx(.01,.012,.012,blk,0,.044,.02));
  const fl = new THREE.PointLight(0xffaa44,0,3); fl.position.set(0,.02,-.2); g.add(fl);
  const spot = new THREE.SpotLight(0xdfffee,2.4,10,.38,.5,1.2); spot.position.set(0,0,-.1);
  spot.target.position.set(0,0,-5); g.add(spot); g.add(spot.target);
  g.userData = {fl, kick:0};
  return g;
}

// HUD
const hc = document.createElement('canvas'); hc.width=512; hc.height=128;
const hx = hc.getContext('2d'), htex = new THREE.CanvasTexture(hc);
const hud = new THREE.Sprite(new THREE.SpriteMaterial({map:htex,depthTest:false,transparent:true}));
hud.scale.set(.5,.125,1); hud.position.set(0,-.2,-.7); hud.renderOrder=999; camera.add(hud);
function drawHud(msg){
  hx.clearRect(0,0,512,128); hx.fillStyle='rgba(0,0,0,.55)'; hx.fillRect(0,0,512,128);
  hx.fillStyle='#6fd6b0'; hx.font='bold 34px sans-serif'; hx.textAlign='center';
  if(msg){ hx.fillText(msg,256,56); hx.font='24px sans-serif'; hx.fillStyle='#fff'; hx.fillText('Tulista, et uuesti alustada',256,98); }
  else { hx.fillText('♥ '+lives+'   ☠ '+score+'   Laine '+wave,256,56); hx.font='22px sans-serif'; hx.fillStyle='#fff'; hx.fillText('Pähe tabamus = 2x',256,98); }
  htex.needsUpdate = true;
}

// Heli
let AC;
function snd(kind){
  if(!AC) return; const t=AC.currentTime, g=AC.createGain(); g.connect(AC.destination);
  if(kind==='shot'){
    const b=AC.createBuffer(1,AC.sampleRate*.15,AC.sampleRate), d=b.getChannelData(0);
    for(let i=0;i<d.length;i++) d[i]=(Math.random()*2-1)*Math.pow(1-i/d.length,3);
    const s=AC.createBufferSource(); s.buffer=b; g.gain.value=.5; s.connect(g); s.start(t);
  } else {
    const o=AC.createOscillator(), f=AC.createBiquadFilter(); o.type='sawtooth'; f.type='lowpass'; f.frequency.value=260;
    o.frequency.setValueAtTime(90,t); o.frequency.linearRampToValueAtTime(55,t+.9);
    g.gain.setValueAtTime(0,t); g.gain.linearRampToValueAtTime(.18,t+.2); g.gain.linearRampToValueAtTime(0,t+.9);
    o.connect(f); f.connect(g); o.start(t); o.stop(t+1);
  }
}

// Zombid (kõhnad, küürus, rebenenud särk, valged juuksed, helendavad silmad)
const TYPES = {
  walker:{hp:2,sp:.3,sc:1,skin:0x6b7a4a,eye:0xc8ffd0},
  runner:{hp:1,sp:.8,sc:.9,skin:0x7d6a48,eye:0xff5a3a},
  tank:{hp:6,sp:.2,sc:1.35,skin:0x3f5238,eye:0xffd060}
};
function part(geo,m,x,y,z){ const p=new THREE.Mesh(geo,m); p.position.set(x,y,z); return p; }
function makeZombie(type){
  const T = TYPES[type], g = new THREE.Group(), body = new THREE.Group(); body.rotation.x = .2; g.add(body);
  const skin = new THREE.MeshLambertMaterial({color:T.skin}), shirt = new THREE.MeshLambertMaterial({color:0x9aa5a0}),
        pants = new THREE.MeshLambertMaterial({color:0x4a4a3a}), hair = new THREE.MeshLambertMaterial({color:0xd8dde0}),
        eyeM = new THREE.MeshBasicMaterial({color:T.eye}), dark = new THREE.MeshBasicMaterial({color:0x100808});
  body.add(part(new THREE.CylinderGeometry(.17,.12,.6,8),shirt,0,1.05,0));
  for(let i=-1;i<=1;i++) body.add(part(new THREE.BoxGeometry(.06,.18,.02),shirt,i*.09,.68,.1));
  const head = part(new THREE.SphereGeometry(.12,10,8),skin,0,1.5,.05); head.scale.set(.9,1.2,.95); body.add(head);
  body.add(part(new THREE.BoxGeometry(.09,.07,.07),dark,0,1.4,.12));
  body.add(part(new THREE.SphereGeometry(.022,6,6),eyeM,-.05,1.53,.14));
  body.add(part(new THREE.SphereGeometry(.022,6,6),eyeM,.05,1.53,.14));
  for(let i=0;i<8;i++) body.add(part(new THREE.BoxGeometry(.018,.22+Math.random()*.2,.018),hair,(i-3.5)*.03,1.4,-.08-Math.random()*.03));
  const arms=[], legs=[];
  for(const s of [-1,1]){
    const a = new THREE.Group(); a.position.set(s*.22,1.3,0); a.rotation.x=-1.25;
    a.add(part(new THREE.BoxGeometry(.07,.32,.07),skin,0,-.16,0)); a.add(part(new THREE.BoxGeometry(.06,.3,.06),skin,0,-.46,0));
    body.add(a); arms.push(a);
    const l = new THREE.Group(); l.position.set(s*.09,.78,0); l.add(part(new THREE.BoxGeometry(.09,.78,.09),pants,0,-.39,0)); g.add(l); legs.push(l);
  }
  g.scale.setScalar(T.sc);
  g.userData = {type, hp:T.hp, sp:T.sp, arms, legs, mats:[skin,shirt], phase:Math.random()*6, flash:0};
  return g;
}

// Olek
let lives, score, wave, spawnT, over, running=false, inXR=false, groanT=3;
const zombies=[], tracers=[], bits=[];
function reset(){ lives=5; score=0; wave=1; spawnT=1; over=false; zombies.splice(0).forEach(z=>scene.remove(z)); drawHud(); }
function spawn(){
  if(zombies.length>=12) return;
  const r=Math.random(); let type='walker';
  if(wave>=4 && r<.14) type='tank'; else if(wave>=2 && r<.35) type='runner';
  const z = makeZombie(type); z.userData.sp *= 1+wave*.06;
  const a = inXR ? Math.random()*Math.PI*2 : -Math.PI/2+(Math.random()-.5)*2;
  z.position.set(Math.cos(a)*5,0,Math.sin(a)*5); scene.add(z); zombies.push(z);
}
function blood(p){
  for(let i=0;i<9;i++){
    const m=new THREE.Mesh(new THREE.BoxGeometry(.025,.025,.025),new THREE.MeshBasicMaterial({color:0x8a0f0f}));
    m.position.copy(p); scene.add(m);
    bits.push({m,v:new THREE.Vector3((Math.random()-.5)*2,Math.random()*2,(Math.random()-.5)*2),t:.6});
  }
}
const ray = new THREE.Raycaster();
function shoot(o,d,gun){
  if(over){ reset(); return; }
  gun.userData.kick=1; gun.userData.fl.intensity=4; snd('shot');
  ray.set(o,d); const hits=ray.intersectObjects(zombies,true); let end=o.clone().add(d.clone().multiplyScalar(7));
  if(hits.length){
    end=hits[0].point; let z=hits[0].object; while(z.parent!==scene) z=z.parent;
    const head=(end.y-z.position.y)/z.scale.y>1.35;
    z.userData.hp-=head?2:1; z.userData.flash=1; blood(end);
    if(z.userData.hp<=0){ scene.remove(z); zombies.splice(zombies.indexOf(z),1); score++; if(score%8===0) wave++; }
  }
  const ln=new THREE.Line(new THREE.BufferGeometry().setFromPoints([o.clone().add(d.clone().multiplyScalar(.3)),end]),new THREE.LineBasicMaterial({color:0xffe066}));
  scene.add(ln); tracers.push({ln,t:.07}); drawHud();
}

const ctrls=[];
for(let i=0;i<2;i++){
  const c=renderer.xr.getController(i), gun=makeGun(); c.add(gun); c.visible=false;
  c.addEventListener('select',()=>{
    const o=new THREE.Vector3().setFromMatrixPosition(c.matrixWorld), d=new THREE.Vector3(0,0,-1).transformDirection(c.matrixWorld);
    shoot(o,d,gun);
  });
  scene.add(c); ctrls.push(c);
}
const flatGun = makeGun(); flatGun.position.set(.16,-.17,-.38); camera.add(flatGun);
renderer.domElement.addEventListener('pointerdown',e=>{
  if(inXR||!running) return;
  const r=new THREE.Raycaster(); r.setFromCamera(new THREE.Vector2(e.clientX/innerWidth*2-1,-(e.clientY/innerHeight)*2+1),camera);
  shoot(r.ray.origin.clone(),r.ray.direction.clone(),flatGun);
});

let last=0;
function loop(t){
  const dt=Math.min((t-last)/1000||0,.1); last=t;
  if(running&&!over){
    spawnT-=dt; if(spawnT<=0){ spawn(); spawnT=Math.max(.8,3-wave*.25); }
    groanT-=dt; if(groanT<=0&&zombies.length){ snd('groan'); groanT=2+Math.random()*3; }
    const cp=new THREE.Vector3(); camera.getWorldPosition(cp);
    for(const z of zombies.slice()){
      const u=z.userData, dx=cp.x-z.position.x, dz=cp.z-z.position.z, dist=Math.hypot(dx,dz);
      z.rotation.y=Math.atan2(dx,dz);
      z.position.x+=dx/dist*u.sp*dt; z.position.z+=dz/dist*u.sp*dt;
      u.phase+=dt*u.sp*14; z.position.y=Math.abs(Math.sin(u.phase))*.03;
      u.legs[0].rotation.x=Math.sin(u.phase)*.5; u.legs[1].rotation.x=-Math.sin(u.phase)*.5;
      u.arms[0].rotation.x=-1.25+Math.sin(u.phase*.7)*.12; u.arms[1].rotation.x=-1.25-Math.sin(u.phase*.7)*.12;
      u.flash=Math.max(0,u.flash-dt*4); u.mats.forEach(m=>m.emissive.setRGB(u.flash*.9,0,0));
      if(dist<.7){ scene.remove(z); zombies.splice(zombies.indexOf(z),1); lives--; drawHud(); if(lives<=0){ over=true; drawHud('Zombid võitsid · '+score+' ☠'); } }
    }
  }
  [flatGun,...ctrls.map(c=>c.children[0])].forEach(g=>{
    g.userData.kick=Math.max(0,g.userData.kick-dt*8); g.rotation.x=g.userData.kick*.25;
    g.userData.fl.intensity=Math.max(0,g.userData.fl.intensity-dt*60);
  });
  for(const b of bits.slice()){ b.t-=dt; b.v.y-=6*dt; b.m.position.addScaledVector(b.v,dt); if(b.t<=0){ scene.remove(b.m); bits.splice(bits.indexOf(b),1); } }
  for(const tr of tracers.slice()){ tr.t-=dt; if(tr.t<=0){ scene.remove(tr.ln); tracers.splice(tracers.indexOf(tr),1); } }
  renderer.render(scene,camera);
}
renderer.setAnimationLoop(loop);

function begin(xr){
  inXR=xr; floor.visible=!xr; dim.visible=xr; flatGun.visible=!xr; ctrls.forEach(c=>c.visible=xr);
  scene.fog.density = xr ? .05 : .09;
  scene.background = xr ? null : new THREE.Color(0x0d1a19);
  AC = AC || new (window.AudioContext||window.webkitAudioContext)();
  document.getElementById('ui').style.display='none'; reset(); running=true;
}
const mrBtn=document.getElementById('mr');
if(navigator.xr){
  navigator.xr.isSessionSupported('immersive-ar').then(ok=>{
    mrBtn.textContent=ok?'Alusta mixed reality’s':'MR pole selles seadmes toetatud'; mrBtn.disabled=!ok;
  }).catch(()=>{ mrBtn.textContent='MR pole saadaval'; });
} else mrBtn.textContent='WebXR puudub (proovi Quest Browserit)';
mrBtn.onclick=async()=>{
  try{
    const s=await navigator.xr.requestSession('immersive-ar',{optionalFeatures:['local-floor']});
    s.addEventListener('end',()=>{ running=false; inXR=false; document.getElementById('ui').style.display='flex'; floor.visible=true; dim.visible=false; scene.background=new THREE.Color(0x0d1a19); });
    await renderer.xr.setSession(s); begin(true);
  }catch(e){ mrBtn.textContent='Viga: '+e.message; }
};
document.getElementById('flat').onclick=()=>begin(false);
addEventListener('resize',()=>{ camera.aspect=innerWidth/innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth,innerHeight); });
</script>
</body>
</html>
