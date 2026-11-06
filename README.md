[index (13).html](https://github.com/user-attachments/files/34497889/index.13.html)
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

<canvas id="skyTex" width="16" height="256" style="display:none"></canvas>
<canvas id="winTex" width="128" height="128" style="display:none"></canvas>
<canvas id="vig" width="256" height="256" style="display:none"></canvas>
<a-scene life renderer="antialias:true" fog="type:linear;color:#c99a8e;near:100;far:1000" background="color:#c99a8e">
  <a-sky src="#skyTex" radius="2500"></a-sky>
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

  <a-light id="amb" type="ambient" color="#ffe2cc" intensity="0.8"></a-light>
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
    <a-text id="score" value="Võidud: 0   Kukkumised: 0" position="0 2.35 1.15" rotation="0 180 0" align="center" width="2.2" color="#aab"></a-text>
    <a-text id="win" value="Päästsid kassi!" visible="false" position="0 1.6 1.15" rotation="0 180 0" align="center" width="3" color="#ffd86b"></a-text>
    <a-box id="btn" class="clickable" position="-1.15 1.2 0" width="0.06" height="0.2" depth="0.2" color="#d22"></a-box>
    <a-text id="floor" value="Korrus 0" position="0 2.1 1.15" rotation="0 180 0" align="center" width="3" color="#fff"></a-text>
  </a-entity>

  <!-- Laud (150 m kõrgusel) -->
  <a-box id="plank" position="0 -0.012 -2.1" width="0.22" height="0.025" depth="1.8" color="#b07a3c"></a-box>
  <a-entity id="cat" position="0 0 -2.7">
    <a-entity id="catbody" scale="1.1 1.1 1.1">
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
  <a-octahedron id="gem" position="0 0.35 -2.7" radius="0.12" visible="false"
    material="color:#4df0ff;emissive:#2ab;emissiveIntensity:0.8;metalness:0.5"
    animation="property:rotation;to:0 360 0;dur:3000;loop:true;easing:linear"
    animation__bob="property:position;from:0 0.32 -2.7;to:0 0.44 -2.7;dur:900;dir:alternate;loop:true;easing:easeInOutSine"></a-octahedron>
  <a-cylinder id="beacon" position="0 3 -2.7" radius="0.1" height="6"
              material="shader:flat;color:#ffd86b;opacity:0.2;transparent:true;side:double"></a-cylinder>

  <a-entity id="city"><a-plane rotation="-90 0 0" position="0 -0.05 0" width="3000" height="3000" color="#333"></a-plane></a-entity>
</a-scene>

<script>
const $ = s => document.querySelector(s);
const FALL_H = 400;
let state = 'idle'; // idle -> ascending -> up -> falling -> landed
let shakeAmp = 0, flicker = false, hum, creakIv, ambWind, rumble, fear = 0, rescued = false, catHome = false, catDist = 99, liftGone = false;
const setPos = (el, x, y, z) => { el.object3D.position.set(x, y, z); el.setAttribute('position', { x, y, z }); };
function recenter() { const c = $('#cam').object3D.position; setPos($('#rig'), -c.x, 0, -c.z); }

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
$('a-scene').addEventListener('enter-vr', () => { audio(); setTimeout(recenter, 700); });

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
setInterval(() => { if (state === 'up' && !rescued) { if (ending === 'gem') chime(); else meow(.1 + .3 * Math.max(0, 1 - catDist / 4)); } }, 3000);

let ending = 'trap', lastEnding = '', wins = 0, falls = 0, fadeIv;
function score() { $('#score').setAttribute('value', 'Võidud: ' + wins + '   Kukkumised: ' + falls); }
function fadeFlash() {
  let o = 1; clearInterval(fadeIv);
  fadeIv = setInterval(() => { o -= .03; $('#flash').setAttribute('material', 'opacity', Math.max(0, o)); if (o <= 0) clearInterval(fadeIv); }, 50);
}
function chime() {
  if (!AC) return;
  [1568, 2093].forEach((fr, i) => { const now = AC.currentTime + i * .12, o = AC.createOscillator(), g = AC.createGain(); o.frequency.value = fr;
    g.gain.setValueAtTime(.12, now); g.gain.exponentialRampToValueAtTime(.001, now + .9); o.connect(g); g.connect(master); o.start(now); o.stop(now + 1); });
}
function pickEnding() {   // iga ring on laua otsas midagi muud
  const c = ['trap', 'rescue', 'gem'].filter(x => x !== lastEnding); ending = lastEnding = c[Math.floor(Math.random() * c.length)];
  $('#cat').setAttribute('visible', ending !== 'gem'); $('#gem').setAttribute('visible', ending === 'gem');
  $('#beacon').setAttribute('material', 'color', ending === 'gem' ? '#4df0ff' : '#ffd86b'); $('#beacon').setAttribute('visible', true);
}
function saveCat() {   // LÕPP 2: kass on päriselt hädas, päästad ta
  meow(.35); setTimeout(ding, 150);
  $('#catsay').setAttribute('value', 'Aitäh!'); $('#beacon').setAttribute('visible', false);
  $('#win').setAttribute('value', 'Päästsid kassi!');
  $('#catbody').setAttribute('animation__hop', { property: 'position', from: '0 0 0', to: '0 0.07 0', dur: 220, dir: 'alternate', loop: true, easing: 'easeInOutSine' });
  $('#cat').setAttribute('animation__walk', { property: 'position', to: '0 0 -1.0', dur: 2500, easing: 'linear' });
}
function grabGem() {   // LÕPP 3: aare, aga varesed ründavad ja laud praksub
  ding(); thump(70, .8, .6); catHome = true;
  $('#gem').setAttribute('visible', false); $('#beacon').setAttribute('visible', false);
  $('#win').setAttribute('value', 'Aare on sinu! Jookse liftini!'); $('#win').setAttribute('visible', true);
  const L = $('a-scene').components.life;
  for (let i = 0; i < 4; i++) setTimeout(() => { if (state === 'up') L.swoopNow(); }, 500 + i * 1100);
  setTimeout(crack, 400); setTimeout(crack, 1800); setTimeout(() => L.heliNow(), 1000);
}
function rescue() {   // kutsutakse, kui jõuad laua otsa
  rescued = true;
  if (ending === 'trap') trapCat(); else if (ending === 'rescue') saveCat(); else grabGem();
}
function trapCat() {   // LÕKS: kass meelitas sind kohale, lift kukub minema, väljapääsu pole
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
  const cat = $('#cat'); ['animation__walk', 'animation__walk2'].forEach(a => cat.removeAttribute(a)); setPos(cat, 0, 0, -2.7);
  $('#catbody').removeAttribute('animation__hop'); setPos($('#catbody'), 0, 0, 0);
  rescued = false; catHome = false; liftGone = false; $('#catsay').setAttribute('value', 'Aita mind!');
  ['#eyeL', '#eyeR'].forEach(id => $(id).setAttribute('color', '#111')); 
  const lift = $('#lift'); lift.removeAttribute('animation__drop'); setPos(lift, 0, 0, 0);
  $('#win').setAttribute('visible', false); pickEnding();
}
function descend() {   // kass on liftis ja sina ka -> sõidame alla
  state = 'descending'; hum = startHum(); shakeAmp = .01;
  creakIv = setInterval(() => { if (Math.random() < .5) creak(); }, 900);
  $('#door').setAttribute('animation__open', { property: 'position', to: '0 1.25 -1.2', dur: 900 });
  if (ambWind) { stopNode(ambWind, .5); ambWind = null; }
  $('#city').setAttribute('animation__down', { property: 'position', to: '0 0 0', dur: 9000, easing: 'easeInOutSine' });
}
$('#city').addEventListener('animationcomplete__down', () => {
  hum.stop(); clearInterval(creakIv); shakeAmp = 0; $('#lift').object3D.position.set(0, 0, 0); ding(); wins++; score();
  const city = $('#city'); ['animation__up1', 'animation__up2', 'animation__down'].forEach(a => city.removeAttribute(a)); setPos(city, 0, 0, 0);
  $('#door').setAttribute('animation__open', { property: 'position', to: '0 4 -1.2', dur: 1500 });
  $('#btn').setAttribute('color', '#d22'); $('#floor').setAttribute('value', 'Korrus 0');
  resetCat(); state = 'idle';
});


/* ---------- ÜLEVAL: pilved, varesed, helikopter, laua kõikumine ---------- */
function caw() {
  if (!AC) return;
  for (let i = 0; i < 3; i++) {
    const now = AC.currentTime + i * .22, o = AC.createOscillator(), g = AC.createGain(), f = AC.createBiquadFilter();
    o.type = 'sawtooth'; o.frequency.setValueAtTime(720, now); o.frequency.exponentialRampToValueAtTime(380, now + .16);
    f.type = 'bandpass'; f.frequency.value = 1100; f.Q.value = 2;
    g.gain.setValueAtTime(.0001, now); g.gain.exponentialRampToValueAtTime(.25, now + .02); g.gain.exponentialRampToValueAtTime(.0001, now + .18);
    o.connect(f); f.connect(g); g.connect(master); o.start(now); o.stop(now + .2);
  }
}
function crack() { thump(180, .6, .12); creak(); const pl = $('#plank').object3D; pl.position.y -= .03; setTimeout(() => { pl.position.y += .03; }, 130); }
function heliSound(dur) {
  if (!AC) return; const now = AC.currentTime, s = AC.createBufferSource(), f = AC.createBiquadFilter(), g = AC.createGain();
  s.buffer = noiseBuf; s.loop = true; f.type = 'lowpass'; f.frequency.value = 260; g.gain.value = .5;
  const lfo = AC.createOscillator(), lg = AC.createGain(); lfo.frequency.value = 13; lg.gain.value = .5; lfo.connect(lg); lg.connect(g.gain);
  const env = AC.createGain(); env.gain.setValueAtTime(0, now); env.gain.linearRampToValueAtTime(.9, now + dur / 2); env.gain.linearRampToValueAtTime(0, now + dur);
  const pan = AC.createStereoPanner(); pan.pan.setValueAtTime(-1, now); pan.pan.linearRampToValueAtTime(1, now + dur);
  s.connect(f); f.connect(g); g.connect(env); env.connect(pan); pan.connect(master);
  s.start(now); lfo.start(now); s.stop(now + dur + .2); lfo.stop(now + dur + .2);
}
AFRAME.registerComponent('life', {
  init() {
    const T = AFRAME.THREE, sc = this.el.object3D; this.v = new T.Vector3();
    let sd = 11; const r = () => (sd = (sd * 16807) % 2147483647) / 2147483647;
    const mk = (c, o) => new T.MeshBasicMaterial({ color: c, transparent: o < 1, opacity: o, depthWrite: o >= 1 });
    this.clouds = new T.Group(); sc.add(this.clouds);
    const cg = new T.SphereGeometry(1, 10, 6), cm = mk(0xffffff, .7);
    for (let i = 0; i < 22; i++) {
      const g = new T.Group(), ang = r() * 6.28, rad = 40 + r() * 280, y = i < 5 ? 6 + r() * 28 : -(50 + r() * 320);
      for (let k = 0; k < 4; k++) { const m = new T.Mesh(cg, cm); m.scale.set(22 + r() * 25, 7 + r() * 6, 16 + r() * 18); m.position.set((k - 1.5) * 20, r() * 4, r() * 14); g.add(m); }
      g.position.set(Math.cos(ang) * rad, y, Math.sin(ang) * rad); this.clouds.add(g);
    }
    const bm = mk(0x111111, 1), wg = new T.BoxGeometry(.5, .012, .14), bg = new T.BoxGeometry(.1, .07, .24);
    const bird = () => { const g = new T.Group(), lw = new T.Mesh(wg, bm), rw = new T.Mesh(wg, bm); g.add(new T.Mesh(bg, bm)); lw.position.x = -.27; rw.position.x = .27; g.add(lw, rw); g.userData = { lw, rw }; return g; };
    this.birds = [];
    for (let i = 0; i < 4; i++) { const b = bird(); Object.assign(b.userData, { a: r() * 6.28, rad: 9 + r() * 10, h: 1 + r() * 7, sp: .0004 + r() * .0003 }); sc.add(b); this.birds.push(b); }
    this.sw = bird(); this.sw.visible = false; sc.add(this.sw); this.swT = -1;
    const hm = mk(0x30343a, 1), h = new T.Group(), tail = new T.Mesh(new T.BoxGeometry(3, .25, .25), hm);
    h.add(new T.Mesh(new T.BoxGeometry(3, 1.2, 1.2), hm)); tail.position.x = -2.8; h.add(tail);
    this.rotor = new T.Mesh(new T.BoxGeometry(8, .05, .2), hm); this.rotor.position.y = .8; h.add(this.rotor); h.visible = false; sc.add(h); this.heli = h; this.heliT = -1;
  },
  swoopNow() {
    const T = AFRAME.THREE; $('#cam').object3D.getWorldPosition(this.v);
    this.p0 = this.v.clone().add(new T.Vector3(8, .1, .35)); this.p1 = this.v.clone().add(new T.Vector3(-8, 0, -.35));
    this.swT = 0; this.sw.visible = true; caw();
  },
  heliNow() { this.heliT = 0; heliSound(14); },
  tick(t, dt) {
    const up = state === 'up' || state === 'falling';
    this.clouds.children.forEach(g => { g.position.x += dt * .004; if (g.position.x > 340) g.position.x = -340; });
    this.birds.forEach(b => {
      const u = b.userData; u.a += u.sp * dt; b.visible = up;
      b.position.set(Math.cos(u.a) * u.rad, u.h + Math.sin(u.a * 3) * 1.2, -2 + Math.sin(u.a) * u.rad); b.rotation.y = -u.a;
      const f = Math.sin(t * .02 + u.a * 5) * .6; u.lw.rotation.z = f; u.rw.rotation.z = -f;
    });
    if (this.swT >= 0) {
      this.swT += dt / 1800;
      if (this.swT >= 1) { this.swT = -1; this.sw.visible = false; }
      else { this.sw.position.lerpVectors(this.p0, this.p1, this.swT); this.sw.rotation.y = -Math.PI / 2; const f = Math.sin(t * .04) * .8; this.sw.userData.lw.rotation.z = f; this.sw.userData.rw.rotation.z = -f; }
    }
    if (this.heliT >= 0) {
      this.heliT += dt / 14000;
      if (this.heliT >= 1) { this.heliT = -1; this.heli.visible = false; }
      else { this.heli.visible = true; this.heli.position.set(-260 + 520 * this.heliT, 14, -110); this.rotor.rotation.y += dt * .05; }
    }
    if (!this.pl) this.pl = $('#plank'); if (this.pl) this.pl.object3D.rotation.z = up ? Math.sin(t / 650) * .006 : 0;
  }
});
setInterval(() => { const L = $('a-scene').components.life; if (L && state === 'up' && Math.random() < .55) L.swoopNow(); }, 9000);
setInterval(() => { const L = $('a-scene').components.life; if (L && state === 'up' && L.heliT < 0 && Math.random() < .4) L.heliNow(); }, 20000);

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

/* ---------- LINN (valgustatud akendega) ---------- */
(function () {
  const T = AFRAME.THREE, cv = $('#winTex'), c = cv.getContext('2d'), city = $('#city');
  let sd = 7; const r = () => (sd = (sd * 16807) % 2147483647) / 2147483647;
  c.fillStyle = '#586075'; c.fillRect(0, 0, 128, 128);
  for (let i = 0; i < 4; i++) for (let j = 0; j < 4; j++) {
    c.fillStyle = r() < .38 ? (r() < .5 ? '#ffd27a' : '#fff0b8') : '#1d2536'; c.fillRect(i * 32 + 7, j * 32 + 8, 18, 16);
  }
  const base = new T.CanvasTexture(cv); base.wrapS = base.wrapT = T.RepeatWrapping;
  if (T.SRGBColorSpace) base.colorSpace = T.SRGBColorSpace;
  const geo = new T.BoxGeometry(1, 1, 1), redM = new T.MeshBasicMaterial({ color: 0xff2020 }), redG = new T.SphereGeometry(1.6, 8, 6);
  for (let i = 0; i < 420; i++) {
    const ang = r() * Math.PI * 2, dist = 18 + r() * 600, h = 15 + r() * (dist > 120 ? 500 : 300), w = 6 + r() * 26;
    const t = base.clone(); t.needsUpdate = true; t.repeat.set(w / 16, h / 16); t.offset.set(r(), r());
    const g = (150 + r() * 105) / 255, m = new T.Mesh(geo, new T.MeshBasicMaterial({ map: t, color: new T.Color(g, g, g * 1.05) }));
    m.scale.set(w, h, w); m.position.set(Math.cos(ang) * dist, h / 2, Math.sin(ang) * dist); city.object3D.add(m);
    if (h > 200) { const l = new T.Mesh(redG, redM); l.position.set(m.position.x, h + 2, m.position.z); city.object3D.add(l); }
  }
  const sk = $('#skyTex').getContext('2d'), gr = sk.createLinearGradient(0, 0, 0, 256);
  [[0, '#1c2a55'], [.3, '#6a5f92'], [.47, '#e0997d'], [.5, '#c99a8e'], [1, '#c99a8e']].forEach(([o, col]) => gr.addColorStop(o, col));
  sk.fillStyle = gr; sk.fillRect(0, 0, 16, 256);
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
  amb.setAttribute('light', 'intensity', (flicker || (state === 'ascending' && Math.random() < .04)) ? (Math.random() < .5 ? .08 : .8) : (rescued && ending === 'trap' ? .4 : .8));
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
  clearInterval(fadeIv); falls++; score(); $('#flash').setAttribute('material', 'opacity', 1);   // kõik läheb valgeks
  setTimeout(restart, 2200);
});

function restart() {   // nagu mängus: uuesti algusest
  const rig = $('#rig'), city = $('#city'), door = $('#door');
  rig.removeAttribute('animation__fall'); recenter();
  ['animation__up1', 'animation__up2'].forEach(a => city.removeAttribute(a)); setPos(city, 0, 0, 0);
  door.removeAttribute('animation__open'); setPos(door, 0, 1.25, -1.2); resetCat();
  $('#btn').setAttribute('color', '#d22'); $('#floor').setAttribute('value', 'Korrus 0');
  state = 'idle';
  fadeFlash();
}

/* ---------- KONTROLL: kas oled liftis või laual ---------- */
$('a-scene').addEventListener('loaded', () => {
  const fl = $('#flash').getObject3D('mesh'); if (fl) fl.renderOrder = 999;
  const vg = $('#vign').getObject3D('mesh'); if (vg) vg.renderOrder = 998;
  const cam = $('#cam').object3D, v = new THREE.Vector3();
  setInterval(() => {
    cam.getWorldPosition(v);
    const rig = $('#rig').object3D.position;
    if (state === 'idle' || state === 'ascending' || state === 'descending') { fear = 0; if (Math.abs(v.x) > 1.0 || Math.abs(v.z) > 1.0) recenter(); return; }
    if (state !== 'up') { fear = state === 'falling' ? 1 : 0; return; }
    const inLift = !liftGone && Math.abs(v.x) < 1.2 && v.z > -1.2 && v.z < 1.2;
    const onPlank = Math.abs(v.x) < 0.18 && v.z <= -1.2 && v.z > -3.1;
    if (!inLift && !onPlank) { fall(); return; }
    fear = (rescued && ending === 'trap') ? 1 : onPlank ? .25 + .75 * Math.min(1, Math.max(0, (-v.z - 1.2) / 1.8)) : 0;
    catDist = Math.hypot(v.x, v.z + 2.7);
    if (!rescued && catDist < 0.7) rescue();
    if (rescued && catHome && inLift) { descend(); return; }
    if (ambWind) ambWind.g.gain.setTargetAtTime(.05 + .15 * fear, AC.currentTime, .3);
    if (onPlank && Math.random() < .03) creak();
    if (onPlank && fear > .5 && Math.random() < .012) crack();
  }, 100);
});
pickEnding(); score();
</script>
</body>
</html>
