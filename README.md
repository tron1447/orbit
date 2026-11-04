[index (6).html](https://github.com/user-attachments/files/34391061/index.6.html)
<!DOCTYPE html>
<html lang="et">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Zombie Defender MR</title>
<style>
:root{--bg:#10140f;--fg:#e8f0e0;--ac:#7bd35a;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme: dark){:root:not([data-theme="light"]){--bg:#10140f;--fg:#e8f0e0;--ac:#7bd35a}}
:root[data-theme="dark"]{--bg:#10140f;--fg:#e8f0e0;--ac:#7bd35a}
html,body{height:100%;margin:0;background:var(--bg);color:var(--fg);font-family:system-ui,sans-serif;overflow:hidden}
canvas{display:block}
#ui{position:fixed;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:12px;text-align:center;padding:24px;background:rgba(10,14,9,.85)}
#ui h1{margin:0;font-size:2rem;color:var(--ac)}
#ui p{margin:0;max-width:32rem;opacity:.85}
button{font:inherit;font-weight:600;padding:12px 22px;border:0;border-radius:10px;background:var(--ac);color:#0b1208;cursor:pointer}
button.alt{background:transparent;color:var(--fg);border:1px solid var(--fg)}
button:disabled{opacity:.4}
</style>
</head>
<body>
<div id="ui">
  <h1>Zombie Defender MR</h1>
  <p>Zombid tulevad su toast sisse. Tulista kontrolleri päästikuga (või ekraanil klikates / puudutades) ja ära lase neil endale ligi. Elusid on 5.</p>
  <button id="mr" disabled>Mixed reality (kontrollin…)</button>
  <button id="flat" class="alt">Mängi tavarežiimis</button>
</div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
const renderer = new THREE.WebGLRenderer({antialias:true, alpha:true});
renderer.setPixelRatio(devicePixelRatio);
renderer.setSize(innerWidth, innerHeight);
renderer.xr.enabled = true;
renderer.xr.setReferenceSpaceType('local-floor');
document.body.prepend(renderer.domElement);

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(70, innerWidth/innerHeight, 0.05, 50);
camera.position.set(0,1.6,0);
scene.add(camera);
scene.add(new THREE.HemisphereLight(0xbfd8ff, 0x203018, 1.1));
const sun = new THREE.DirectionalLight(0xffffff, .6); sun.position.set(1,3,1); scene.add(sun);

const floor = new THREE.Mesh(new THREE.CircleGeometry(6,48), new THREE.MeshBasicMaterial({color:0x1a2616}));
floor.rotation.x = -Math.PI/2; scene.add(floor);

// HUD
const hc = document.createElement('canvas'); hc.width = 512; hc.height = 128;
const hctx = hc.getContext('2d'), htex = new THREE.CanvasTexture(hc);
const hud = new THREE.Sprite(new THREE.SpriteMaterial({map:htex, depthTest:false, transparent:true}));
hud.scale.set(.5,.125,1); hud.position.set(0,-.2,-.7); hud.renderOrder = 999; camera.add(hud);
function drawHud(msg){
  hctx.clearRect(0,0,512,128);
  hctx.fillStyle='rgba(0,0,0,.55)'; hctx.fillRect(0,0,512,128);
  hctx.fillStyle='#7bd35a'; hctx.font='bold 34px sans-serif'; hctx.textAlign='center';
  if(msg){ hctx.fillText(msg,256,56); hctx.font='24px sans-serif'; hctx.fillStyle='#fff'; hctx.fillText('Tulista, et uuesti alustada',256,98); }
  else {
    hctx.fillText('♥ '+lives+'   ☠ '+score+'   Laine '+wave,256,56);
    hctx.font='22px sans-serif'; hctx.fillStyle='#fff'; hctx.fillText('Hoia nad eemal!',256,98);
  }
  htex.needsUpdate = true;
}

// Mängu olek
let lives, score, wave, spawnT, alive, over, inXR = false;
const zombies = [], tracers = [];
function reset(){ lives=5; score=0; wave=1; spawnT=1; over=false; zombies.splice(0).forEach(z=>scene.remove(z)); drawHud(); }

const skin = new THREE.MeshLambertMaterial({color:0x6fa05a});
const cloth = new THREE.MeshLambertMaterial({color:0x4a5a7a});
function box(w,h,d,m,x,y,z){ const b=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),m); b.position.set(x,y,z); return b; }
function makeZombie(){
  const g = new THREE.Group();
  g.add(box(.45,.65,.25,cloth,0,1.0,0));
  g.add(box(.3,.3,.3,skin,0,1.5,0));
  g.add(box(.18,.7,.18,cloth,-.12,.35,0));
  g.add(box(.18,.7,.18,cloth,.12,.35,0));
  const al = box(.14,.14,.6,skin,-.3,1.2,.3), ar = box(.14,.14,.6,skin,.3,1.2,.3);
  g.add(al); g.add(ar);
  g.userData = {hp:2, arms:[al,ar], phase:Math.random()*6, speed:.3+wave*.06};
  return g;
}
function spawn(){
  const z = makeZombie();
  const a = inXR ? Math.random()*Math.PI*2 : -Math.PI/2 + (Math.random()-.5)*2;
  z.position.set(Math.cos(a)*4, 0, Math.sin(a)*4);
  scene.add(z); zombies.push(z);
}
function kill(z){ scene.remove(z); zombies.splice(zombies.indexOf(z),1); score++; if(score%8===0) wave++; }

const ray = new THREE.Raycaster();
function shoot(origin, dir){
  if(over){ reset(); return; }
  ray.set(origin, dir);
  const hits = ray.intersectObjects(zombies, true);
  let end = origin.clone().add(dir.clone().multiplyScalar(6));
  if(hits.length){
    end = hits[0].point;
    let z = hits[0].object; while(z.parent !== scene) z = z.parent;
    z.userData.hp--;
    z.children.forEach(c=>{ if(c.material===skin) { c.material = c.material.clone(); c.material.color.set(0xff3030); } });
    if(z.userData.hp<=0) kill(z);
  }
  const line = new THREE.Line(new THREE.BufferGeometry().setFromPoints([origin.clone().add(dir.clone().multiplyScalar(.3)), end]), new THREE.LineBasicMaterial({color:0xffe066}));
  scene.add(line); tracers.push({line, t:.08});
  drawHud();
}

// XR-kontrollerid (ka puutetundlik ekraan telefonis)
for(let i=0;i<2;i++){
  const c = renderer.xr.getController(i);
  c.addEventListener('select', ()=>{
    const o = new THREE.Vector3().setFromMatrixPosition(c.matrixWorld);
    const d = new THREE.Vector3(0,0,-1).transformDirection(c.matrixWorld);
    shoot(o,d);
  });
  scene.add(c);
}
// Tavarežiim
renderer.domElement.addEventListener('pointerdown', e=>{
  if(inXR || !running) return;
  const m = new THREE.Vector2(e.clientX/innerWidth*2-1, -(e.clientY/innerHeight)*2+1);
  const r = new THREE.Raycaster(); r.setFromCamera(m,camera);
  shoot(r.ray.origin.clone().add(new THREE.Vector3(0,-.1,0)), r.ray.direction.clone());
});

let running = false, last = 0;
function loop(t){
  const dt = Math.min((t-last)/1000||0, .1); last = t;
  if(running && !over){
    spawnT -= dt;
    if(spawnT<=0){ spawn(); spawnT = Math.max(.7, 3 - wave*.25); }
    const cp = new THREE.Vector3(); camera.getWorldPosition(cp);
    for(const z of zombies.slice()){
      const u = z.userData;
      const dx = cp.x - z.position.x, dz = cp.z - z.position.z, dist = Math.hypot(dx,dz);
      z.rotation.y = Math.atan2(dx,dz);
      z.position.x += dx/dist*u.speed*dt; z.position.z += dz/dist*u.speed*dt;
      u.phase += dt*5; z.position.y = Math.abs(Math.sin(u.phase))*.04;
      u.arms.forEach((a,i)=>a.rotation.x = Math.sin(u.phase+i)*.15);
      if(dist<.7){ scene.remove(z); zombies.splice(zombies.indexOf(z),1); lives--; drawHud(); if(lives<=0){ over=true; drawHud('Zombid võitsid · '+score+' ☠'); } }
    }
  }
  for(const tr of tracers.slice()){ tr.t-=dt; if(tr.t<=0){ scene.remove(tr.line); tracers.splice(tracers.indexOf(tr),1);} }
  renderer.render(scene,camera);
}
renderer.setAnimationLoop(loop);

function begin(xr){
  inXR = xr; floor.visible = !xr;
  scene.background = xr ? null : new THREE.Color(0x0b100a);
  document.getElementById('ui').style.display='none';
  reset(); running = true;
}
const mrBtn = document.getElementById('mr');
if(navigator.xr){
  navigator.xr.isSessionSupported('immersive-ar').then(ok=>{
    mrBtn.textContent = ok ? 'Alusta mixed reality’s' : 'MR pole selles seadmes toetatud';
    mrBtn.disabled = !ok;
  }).catch(()=>{ mrBtn.textContent='MR pole saadaval'; });
} else mrBtn.textContent = 'WebXR puudub (proovi Quest Browserit)';
mrBtn.onclick = async ()=>{
  try{
    const s = await navigator.xr.requestSession('immersive-ar',{optionalFeatures:['local-floor']});
    s.addEventListener('end', ()=>{ running=false; inXR=false; document.getElementById('ui').style.display='flex'; floor.visible=true; scene.background=new THREE.Color(0x0b100a); });
    await renderer.xr.setSession(s); begin(true);
  }catch(e){ mrBtn.textContent='Viga: '+e.message; }
};
document.getElementById('flat').onclick = ()=>begin(false);
addEventListener('resize', ()=>{ camera.aspect=innerWidth/innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth,innerHeight); });
</script>
</body>
</html>
