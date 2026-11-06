[index (11).html](https://github.com/user-attachments/files/34496921/index.11.html)
<!DOCTYPE html>
<html lang="et">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Plank Walk VR</title>
<script src="https://cdn.jsdelivr.net/npm/aframe@1.5.0/dist/aframe-master.min.js"></script>
<style>
  #hud{position:fixed;left:12px;bottom:12px;max-width:340px;padding:10px 14px;border-radius:10px;
       background:rgba(0,0,0,.6);color:#fff;font:14px/1.4 system-ui,sans-serif;z-index:10}
</style>
</head>
<body>
<div id="hud">Klõpsa punast nuppu liftis (või Space). Üleval kutsub kass laua otsas sind... WASD + hiir = töölaud. VR-is vajuta alt paremalt VR-nuppu.</div>

<canvas id="vig" width="256" height="256" style="display:none"></canvas>
<a-scene renderer="antialias:true" fog="type:linear;color:#bcd4e6;near:100;far:1000" background="color:#bcd4e6">
  <a-entity cursor="rayOrigin:mouse" raycaster="objects:.clickable"></a-entity>

  <a-entity id="shaker">
    <a-entity id="rig" wasd-controls="acceleration:20">
      <a-camera id="cam" fov="80" position="0 1.6 0" look-controls>
        <a-plane id="flash" position="0 0 -0.25" width="4" height="4"
                 material="color:#fff;shader:flat;opacity:0;transparent:true;depthTest:false;depthWrite:false"></a-plane>
        <a-plane id="vign" position="0 0 -0.24" width="1.4" height="1.4"
                 material="src:#vig;shader:flat;transparent:true;opacity:0;depthTest:false;depthWrite:false"></a-plane>
      </a-camera>
      <a-entity laser-controls="hand:left" raycaster="objects:.clickable" stick-move></a-entity>
      <a-entity laser-controls="hand:right" raycaster="objects:.clickable"></a-entity>
    </a-entity>
  </a-entity>

  <a-light id="amb" type="ambient" color="#fff" intensity="0.8"></a-light>
  <a-light type="directional" position="-3 8 4" intensity="0.6"></a-light>

  <!-- Lift (väriseb sõidu ajal) -->
  <a-entity id="lift">
    <a-box position="0 -0.05 0" width="2.4" height="0.1" depth="2.4" color="#555"></a-box>
    <a-box position="-1.2 1.25 0" width="0.05" height="2.5" depth="2.4" color="#9aa"></a-box>
    <a-box position="1.2 1.25 0" width="0.05" height="2.5" depth="2.4" color="#9aa"></a-box>
    <a-box position="0 1.25 1.2" width="2.4" height="2.5" depth="0.05" color="#9aa"></a-box>
    <a-box position="0 2.55 0" width="2.5" height="0.1" depth="2.5" color="#667"></a-box>
    <a-box position="0 2.47 0" width="0.7" height="0.04" depth="0.3" color="#ffe9a8" material="shader:flat"></a-box>
    <a-box id="door" position="0 1.25 -1.2" width="2.4" height="2.5" depth="0.05" color="#778"></a-box>
    <a-text id="win" value="Päästsid kassi!" visible="false" position="0 1.6 1.15" rotation="0 180 0" align="center" width="3" color="#ffd86b"></a-text>
    <a-box id="btn" class="clickable" position="-1.15 1.2 0" width="0.06" height="0.2" depth="0.2" color="#d22"></a-box>
    <a-text id="floor" value="Korrus 0" position="0 2.1 1.15" rotation="0 180 0" align="center" width="3" color="#fff"></a-text>
  </a-entity>

  <!-- Laud (150 m kõrgusel) -->
  <a-box position="0 -0.015 -3.45" width="0.3" height="0.03" depth="4.5" color="#b07a3c"></a-box>
  <a-entity id="cat" position="0 0 -5.3">
    <a-entity id="catbody" scale="1.4 1.4 1.4">
      <a-box position="0 0.11 0" width="0.2" height="0.16" depth="0.34" color="#e9892b"></a-box>
      <a-sphere position="0 0.24 0.17" radius="0.09" color="#f09a3e"></a-sphere>
      <a-cone position="-0.055 0.33 0.17" radius-bottom="0.035" radius-top="0" height="0.07" color="#f09a3e"></a-cone>
      <a-cone position="0.055 0.33 0.17" radius-bottom="0.035" radius-top="0" height="0.07" color="#f09a3e"></a-cone>
      <a-sphere id="eyeL" position="-0.035 0.255 0.25" radius="0.014" color="#111"></a-sphere>
      <a-sphere id="eyeR" position="0.035 0.255 0.25" radius="0.014" color="#111"></a-sphere>
      <a-sphere position="0 0.23 0.262" radius="0.012" color="#f4a"></a-sphere>
      <a-cylinder position="0 0.2 -0.22" radius="0.016" height="0.3" rotation="35 0 0" color="#e9892b"
                  animation="property:rotation;from:35 -25 0;to:35 25 0;dur:700;dir:alternate;loop:true;easing:easeInOutSine"></a-cylinder>
    </a-entity>
    <a-text id="catsay" value="Aita mind!" position="0 0.85 0" align="center" width="3.2" color="#fff" look-at="#cam"></a-text>
  </a-entity>
  <a-cylinder id="beacon" position="0 3 -5.3" radius="0.12" height="6"
              material="shader:flat;color:#ffd86b;opacity:0.2;transparent:true;side:double"></a-cylinder>

  <a-entity id="city"><a-plane rotation="-90 0 0" position="0 -0.05 0" width="3000" height="3000" color="#333"></a-plane></a-entity>
</a-scene>

<script>
const $ = s => document.querySelector(s);
const FALL_H = 400;
let state = 'idle'; // idle -> ascending -> up -> falling -> landed
let shakeAmp = 0, flicker = false, hum, creakIv, ambWind, rumble, fear = 0, rescued = false, catHome = false, catDist = 99, liftGone = false;
const setPos = (el, x, y, z) => el.setAttribute('position', { x, y, z });

/* ---------- HELI (WebAudio, failid pole vaja) ---------- */
let AC, master, noiseBuf;
function audio() {
  if (!AC) {
    AC = new (window.AudioContext || window.webkitAudioContext)();
    master = AC.createGain(); master.gain.value = 0.8; master.connect(AC.destination);
    noiseBuf = AC.createBuffer(1, AC.sampleRate * 2, AC.sampleRate);
    const d = noiseBuf.getChannelData(0); for (let i = 0; i < d.length; i++) d[i] = Math.random() * 2 - 1;
  }
  if (AC.state === 'suspended') AC.resume();
}
['pointerdown', 'keydown', 'click'].forEach(e => window.addEventListener(e, audio));
$('a-scene').addEventListener('enter-vr', audio);

function noise(type, freq, q, vol) {
  const s = AC.createBufferSource(); s.buffer = noiseBuf; s.loop = true;
  const f = AC.createBiquadFilter(); f.type = type; f.frequency.value = freq; f.Q.value = q;
  const g = AC.createGain(); g.gain.value = vol;
  s.connect(f); f.connect(g); g.connect(master); s.start(); return { s, f, g };
}
function osc(type, freq, vol) {
  const o = AC.createOscillator(); o.type = type; o.frequency.value = freq;
  const g = AC.createGain(); g.gain.value = vol; o.connect(g); g.connect(master); o.start(); return { o, g };
}
function stopNode(n, t = 0.4) {
  if (!n || !AC) return; const now = AC.currentTime;
  n.g.gain.cancelScheduledValues(now); n.g.gain.setTargetAtTime(0, now, t / 3);
  [n.s, n.o, n.l].forEach(x => x && x.stop(now + t + 0.3));
}
function startHum() {
  const a = osc('sine', 48, .35), b = osc('triangle', 96, .12), n = noise('lowpass', 150, 1, .5);
  return { stop() { [a, b, n].forEach(x => stopNode(x)); } };
}
function creak() {
  if (!AC) return; const now = AC.currentTime;
  const o = AC.createOscillator(), g = AC.createGain(), f = AC.createBiquadFilter();
  o.type = 'sawtooth'; const f0 = 180 + Math.random() * 300;
  o.frequency.setValueAtTime(f0, now); o.frequency.linearRampToValueAtTime(f0 * (0.6 + Math.random() * 0.8), now + 0.4);
  f.type = 'bandpass'; f.frequency.value = 700 + Math.random() * 900; f.Q.value = 6;
  g.gain.setValueAtTime(0, now); g.gain.linearRampToValueAtTime(.12, now + .05); g.gain.linearRampToValueAtTime(0, now + .45);
  o.connect(f); f.connect(g); g.connect(master); o.start(now); o.stop(now + .5);
}
function thump(freq, vol, dur) {
  const now = AC.currentTime, o = AC.createOscillator(), g = AC.createGain();
  o.frequency.setValueAtTime(freq, now); o.frequency.exponentialRampToValueAtTime(30, now + dur);
  g.gain.setValueAtTime(vol, now); g.gain.exponentialRampToValueAtTime(0.001, now + dur);
  o.connect(g); g.connect(master); o.start(now); o.stop(now + dur);
}
function clank() {
  const now = AC.currentTime;
  [310, 470, 733].forEach(fr => {
    const o = AC.createOscillator(), g = AC.createGain(); o.type = 'square'; o.frequency.value = fr;
    g.gain.setValueAtTime(.12, now); g.gain.exponentialRampToValueAtTime(.001, now + .35);
    o.connect(g); g.connect(master); o.start(now); o.stop(now + .4);
  });
}
function ding() {
  [880, 1318].forEach((fr, i) => {
    const now = AC.currentTime, o = AC.createOscillator(), g = AC.createGain(); o.frequency.value = fr;
    g.gain.setValueAtTime(.2 / (i + 1), now); g.gain.exponentialRampToValueAtTime(.001, now + 1.6);
    o.connect(g); g.connect(master); o.start(now); o.stop(now + 1.7);
  });
}
function impact() {
  thump(90, 1, .6);
  const n = noise('lowpass', 900, 1, .9), now = AC.currentTime;
  n.g.gain.setValueAtTime(.9, now); n.g.gain.exponentialRampToValueAtTime(.001, now + .5); n.s.stop(now + .6);
  const t = osc('sine', 3200, .0), g = t.g.gain; g.setValueAtTime(.05, now + .3); g.exponentialRampToValueAtTime(.0005, now + 3);
  t.o.stop(now + 3.1);
}



/* ---------- KASS: põhjus laua peale minna ---------- */
function meow(vol = .3) {
  if (!AC) return; const now = AC.currentTime, o = AC.createOscillator(), g = AC.createGain(), f = AC.createBiquadFilter();
  o.type = 'sawtooth'; o.frequency.setValueAtTime(520, now); o.frequency.linearRampToValueAtTime(880, now + .18); o.frequency.linearRampToValueAtTime(480, now + .6);
  f.type = 'bandpass'; f.Q.value = 3; f.frequency.setValueAtTime(900, now); f.frequency.linearRampToValueAtTime(1800, now + .2); f.frequency.linearRampToValueAtTime(900, now + .6);
  g.gain.setValueAtTime(0, now); g.gain.linearRampToValueAtTime(vol, now + .08); g.gain.linearRampToValueAtTime(0, now + .65);
  o.connect(f); f.connect(g); g.connect(master); o.start(now); o.stop(now + .7);
}
setInterval(() => { if (state === 'up' && !rescued) meow(.1 + .3 * Math.max(0, 1 - catDist / 6)); }, 3000);

function rescue() {   // LÕKS: kass meelitas sind kohale, lift kukub minema, väljapääsu pole
  rescued = true; meow(.35); thump(55, .9, .9);
  $('#catsay').setAttribute('value', 'Siit pole väljapääsu...');
  $('#beacon').setAttribute('material', 'color', '#ff2222');
  ['#eyeL', '#eyeR'].forEach(id => $(id).setAttribute('color', '#ff2200'));
  flicker = true; setTimeout(() => { flicker = false; }, 2500);
  setTimeout(() => { thump(80, 1, .5); clank(); $('#door').setAttribute('animation__open', { property: 'position', to: '0 1.25 -1.2', dur: 150 }); }, 1200);
  setTimeout(() => {
    liftGone = true; const lift = $('#lift'); setPos(lift, 0, 0, 0);
    lift.setAttribute('animation__drop', { property: 'position', to: '0 -400 0', dur: 3500, easing: 'easeInQuad' });
    for (let i = 0; i < 6; i++) setTimeout(creak, i * 250);
    clank();
  }, 2000);
  setTimeout(() => { thump(60, 1, .9); setTimeout(() => thump(45, .8, 1.1), 250); }, 5800);
}
$('#cat').addEventListener('animationcomplete__walk', () => {
  if (!rescued) return;
  $('#cat').setAttribute('animation__walk2', { property: 'position', to: '0.7 0 0.5', dur: 900, easing: 'easeInOutSine' });
});
$('#cat').addEventListener('animationcomplete__walk2', () => {
  if (!rescued) return;
  catHome = true; $('#catbody').removeAttribute('animation__hop'); setPos($('#catbody'), 0, 0, 0);
  meow(.2); $('#win').setAttribute('visible', true);
});
function resetCat() {
  const cat = $('#cat'); ['animation__walk', 'animation__walk2'].forEach(a => cat.removeAttribute(a)); setPos(cat, 0, 0, -5.3);
  $('#catbody').removeAttribute('animation__hop'); setPos($('#catbody'), 0, 0, 0);
  rescued = false; catHome = false; liftGone = false; $('#catsay').setAttribute('value', 'Aita mind!');
  ['#eyeL', '#eyeR'].forEach(id => $(id).setAttribute('color', '#111')); $('#beacon').setAttribute('material', 'color', '#ffd86b');
  const lift = $('#lift'); lift.removeAttribute('animation__drop'); setPos(lift, 0, 0, 0);
  $('#beacon').setAttribute('visible', true); $('#win').setAttribute('visible', false);
}
function descend() {   // kass on liftis ja sina ka -> sõidame alla
  state = 'descending'; hum = startHum(); shakeAmp = .01;
  creakIv = setInterval(() => { if (Math.random() < .5) creak(); }, 900);
  $('#door').setAttribute('animation__open', { property: 'position', to: '0 1.25 -1.2', dur: 900 });
  if (ambWind) { stopNode(ambWind, .5); ambWind = null; }
  $('#city').setAttribute('animation__down', { property: 'position', to: '0 0 0', dur: 9000, easing: 'easeInOutSine' });
}
$('#city').addEventListener('animationcomplete__down', () => {
  hum.stop(); clearInterval(creakIv); shakeAmp = 0; $('#lift').object3D.position.set(0, 0, 0); ding();
  const city = $('#city'); ['animation__up1', 'animation__up2', 'animation__down'].forEach(a => city.removeAttribute(a)); setPos(city, 0, 0, 0);
  $('#door').setAttribute('animation__open', { property: 'position', to: '0 4 -1.2', dur: 1500 });
  $('#btn').setAttribute('color', '#d22'); $('#floor').setAttribute('value', 'Korrus 0');
  resetCat(); state = 'idle';
});

/* ---------- HIRM: südamelöögid + äärte tumenemine ---------- */
(function () {
  const c = $('#vig').getContext('2d'), g = c.createRadialGradient(128, 128, 15, 128, 128, 80);
  g.addColorStop(0, 'rgba(0,0,0,0)'); g.addColorStop(1, 'rgba(70,0,0,1)');
  c.fillStyle = g; c.fillRect(0, 0, 256, 256);
})();
function beat() {
  if (AC && fear > 0 && (state === 'falling' || state === 'up')) {
    const v = .2 + .6 * fear; thump(75, v, .16); setTimeout(() => thump(60, v * .8, .16), 170);
  }
  setTimeout(beat, state === 'falling' ? 420 : 900 - 450 * fear);
}
beat();

/* ---------- KONTROLLERI LIIKUMINE (VR vasak kang) ---------- */
AFRAME.registerComponent('stick-move', {
  init() { this.a = [0, 0]; this.d = new THREE.Vector3();
    this.el.addEventListener('thumbstickmoved', e => { this.a = [e.detail.x, e.detail.y]; }); },
  tick(t, dt) {
    if (state !== 'up') return;
    const [ax, ay] = this.a; if (Math.abs(ax) + Math.abs(ay) < 0.15) return;
    $('#cam').object3D.getWorldDirection(this.d);
    const fx = -this.d.x, fz = -this.d.z, l = Math.hypot(fx, fz) || 1;
    const nx = fx / l, nz = fz / l, s = 1.2 * dt / 1000, p = $('#rig').object3D.position;
    p.x += (nx * -ay + -nz * ax) * s; p.z += (nz * -ay + nx * ax) * s;
  }
});

/* ---------- LINN ---------- */
(function () {
  let seed = 7; const r = () => (seed = (seed * 16807) % 2147483647) / 2147483647;
  const city = $('#city');
  for (let i = 0; i < 420; i++) {
    const ang = r() * Math.PI * 2, dist = 18 + r() * 600;
    const h = 15 + r() * (dist > 120 ? 500 : 300), w = 6 + r() * 26;
    const b = document.createElement('a-box');
    b.setAttribute('position', `${Math.cos(ang) * dist} ${h / 2} ${Math.sin(ang) * dist}`);
    b.setAttribute('width', w); b.setAttribute('depth', w); b.setAttribute('height', h);
    const g = 90 + Math.floor(r() * 90);
    b.setAttribute('color', `rgb(${g},${g + 8},${g + 18})`);
    city.appendChild(b);
  }
})();

/* ---------- LIFT (vilgub, väriseb, jõnksatab) ---------- */
function ascend() {
  if (state !== 'idle') return;
  audio(); state = 'ascending'; $('#door').setAttribute('animation__open', { property: 'position', to: '0 1.25 -1.2', dur: 600 }); $('#btn').setAttribute('color', '#2a2');
  hum = startHum(); shakeAmp = 0.012;
  creakIv = setInterval(() => { if (Math.random() < .7) creak(); }, 650);
  $('#city').setAttribute('animation__up1', { property: 'position', to: '0 -150 0', dur: 6000, easing: 'easeInQuad' });
}
$('#city').addEventListener('animationcomplete__up1', () => {
  // jõnks: lift peatub, tuled vilguvad
  hum.stop(); thump(120, .9, .4); clank(); shakeAmp = 0.05; flicker = true;
  setTimeout(() => clank(), 700);
  setTimeout(() => {
    flicker = false; shakeAmp = 0.02; hum = startHum();
    $('#city').setAttribute('animation__up2', { property: 'position', to: `0 -${FALL_H} 0`, dur: 7000, easing: 'easeOutQuad' });
  }, 1700);
});
$('#city').addEventListener('animationcomplete__up2', () => {
  hum.stop(); clearInterval(creakIv); shakeAmp = 0; ding();
  $('#lift').object3D.position.set(0, 0, 0);
  $('#door').setAttribute('animation__open', { property: 'position', to: '0 4 -1.2', dur: 1500 });
  ambWind = noise('bandpass', 500, .5, .05);
  state = 'up';
});
$('#btn').addEventListener('click', ascend);
window.addEventListener('keydown', e => { if (e.code === 'Space') ascend(); });

setInterval(() => {  // värin, tulede vilkumine, korruse number
  const lift = $('#lift').object3D, amb = $('#amb');
  if (shakeAmp > 0 && state !== 'falling') lift.position.set((Math.random() - .5) * shakeAmp * 2, (Math.random() - .5) * shakeAmp, (Math.random() - .5) * shakeAmp * 2);
  amb.setAttribute('light', 'intensity', (flicker || (state === 'ascending' && Math.random() < .04)) ? (Math.random() < .5 ? .08 : .8) : (rescued ? .4 : .8));
  if (state === 'ascending' || state === 'descending') $('#floor').setAttribute('value', 'Korrus ' + Math.max(0, Math.round(-$('#city').object3D.position.y / 3)));
}, 80);

/* ---------- KUKKUMINE ---------- */
function fall() {
  state = 'falling';
  const rig = $('#rig'), p = rig.object3D.position;
  setPos(rig, p.x, p.y, p.z);
  rig.setAttribute('animation__fall', { property: 'position', to: `${p.x} -${FALL_H} ${p.z}`, dur: 8500, easing: 'easeInQuad' });
  const t = AC.currentTime;
  if (ambWind) {
    ambWind.g.gain.cancelScheduledValues(t); ambWind.g.gain.setValueAtTime(ambWind.g.gain.value, t);
    ambWind.g.gain.linearRampToValueAtTime(1.0, t + 8.5);
    ambWind.f.frequency.cancelScheduledValues(t); ambWind.f.frequency.setValueAtTime(500, t);
    ambWind.f.frequency.linearRampToValueAtTime(3500, t + 8.5);
  }
  rumble = osc('sine', 38, 0); rumble.g.gain.setValueAtTime(0, t); rumble.g.gain.linearRampToValueAtTime(.7, t + 8);
}
setInterval(() => {  // kukkumise efektid: värin, nähtavusväli laieneb
  if (state !== 'falling') return;
  const prog = Math.min(1, -$('#rig').object3D.position.y / FALL_H), a = 0.01 + 0.12 * prog;
  $('#shaker').object3D.position.set((Math.random() - .5) * a, (Math.random() - .5) * a, (Math.random() - .5) * a);
  $('#cam').setAttribute('camera', 'fov', 80 + 55 * prog);
  $('#vign').setAttribute('material', 'opacity', Math.min(1, prog * 1.3));
}, 30);

$('#rig').addEventListener('animationcomplete__fall', () => {
  if (state !== 'falling') return;
  state = 'landed';
  stopNode(rumble, .05); stopNode(ambWind, .1); ambWind = null;
  $('#shaker').object3D.position.set(0, 0, 0); $('#cam').setAttribute('camera', 'fov', 80);
  impact(); $('#vign').setAttribute('material', 'opacity', 0);
  $('#flash').setAttribute('material', 'opacity', 1);   // kõik läheb valgeks
  setTimeout(restart, 2200);
});

function restart() {   // nagu mängus: uuesti algusest
  const rig = $('#rig'), city = $('#city'), door = $('#door');
  rig.removeAttribute('animation__fall'); setPos(rig, 0, 0, 0);
  ['animation__up1', 'animation__up2'].forEach(a => city.removeAttribute(a)); setPos(city, 0, 0, 0);
  door.removeAttribute('animation__open'); setPos(door, 0, 1.25, -1.2); resetCat();
  $('#btn').setAttribute('color', '#d22'); $('#floor').setAttribute('value', 'Korrus 0');
  state = 'idle';
  $('#flash').setAttribute('animation__fade', { property: 'material.opacity', from: 1, to: 0, dur: 1800, easing: 'linear' });
}

/* ---------- KONTROLL: kas oled liftis või laual ---------- */
$('a-scene').addEventListener('loaded', () => {
  const fl = $('#flash').getObject3D('mesh'); if (fl) fl.renderOrder = 999;
  const vg = $('#vign').getObject3D('mesh'); if (vg) vg.renderOrder = 998;
  const cam = $('#cam').object3D, v = new THREE.Vector3();
  setInterval(() => {
    cam.getWorldPosition(v);
    const rig = $('#rig').object3D.position;
    if (state === 'idle' || state === 'ascending' || state === 'descending') { fear = 0; if (Math.abs(v.x) > 1.0 || Math.abs(v.z) > 1.0) setPos($('#rig'), 0, 0, 0); return; }
    if (state !== 'up') { fear = state === 'falling' ? 1 : 0; return; }
    const inLift = !liftGone && Math.abs(v.x) < 1.2 && v.z > -1.2 && v.z < 1.2;
    const onPlank = Math.abs(v.x) < 0.25 && v.z <= -1.2 && v.z > -5.8;
    if (!inLift && !onPlank) { fall(); return; }
    fear = rescued ? 1 : onPlank ? .25 + .75 * Math.min(1, Math.max(0, (-v.z - 1.2) / 4.3)) : 0;
    catDist = Math.hypot(v.x, v.z + 5.3);
    if (!rescued && catDist < 0.8) rescue();
    if (ambWind) ambWind.g.gain.setTargetAtTime(.05 + .15 * fear, AC.currentTime, .3);
    if (onPlank && Math.random() < .03) creak();
  }, 100);
});
</script>
</body>
</html>
