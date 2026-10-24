[Uploading index (32).html…]()
<!doctype html>
<html lang="et">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1">
<title>Tähesadam VR</title>
<style>
  html,body{margin:0;height:100%;overflow:hidden;background:#01020a;font-family:system-ui,-apple-system,"Segoe UI",sans-serif;color:#d8f6ff}
  canvas{display:block}
  #load{position:fixed;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;background:radial-gradient(ellipse at center,#0b1a3a 0%,#01020a 70%);z-index:20;transition:opacity .6s}
  #load h1{font-size:46px;letter-spacing:.14em;margin:0 0 10px;color:#9ff0ff;text-shadow:0 0 18px #2ad4ff}
  #load p{opacity:.8;margin:4px}
  #bar{width:280px;height:6px;border-radius:3px;background:#123;overflow:hidden;margin-top:16px}
  #bar i{display:block;height:100%;width:0;background:linear-gradient(90deg,#2ad4ff,#b06cff);transition:width .2s}
  #info{position:fixed;left:14px;top:14px;max-width:340px;padding:14px 16px;background:rgba(4,14,30,.8);border:1px solid rgba(42,212,255,.45);border-radius:12px;font-size:13px;line-height:1.55;z-index:5}
  #info h2{margin:0 0 6px;font-size:18px;letter-spacing:.08em;color:#9ff0ff}
  #info b{color:#8ff0ff}
  #info .k{display:inline-block;min-width:14px;padding:0 5px;border:1px solid #3a7a90;border-radius:4px;background:#0a2030;font-size:11px;margin-right:2px}
  #info hr{border:0;border-top:1px solid #2a5a6a;margin:8px 0}
  #info button{margin-top:6px;background:#0a2a3a;color:#bff;border:1px solid #2a7a90;border-radius:6px;padding:3px 10px;cursor:pointer}
  #cross{position:fixed;left:50%;top:50%;width:10px;height:10px;margin:-7px 0 0 -7px;border:2px solid rgba(140,255,240,.85);border-radius:50%;pointer-events:none;z-index:4}
</style>
<script type="importmap">
{"imports":{
  "three":"https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
  "three/addons/":"https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
}}
</script>
</head>
<body>
<div id="load"><h1>TÄHESADAM</h1><p>Laadin galaktikat…</p><div id="bar"><i id="barf"></i></div></div>
<div id="info">
  <h2>TÄHESADAM VR</h2>
  <b>Quest:</b> vajuta all <b>ENTER VR</b>.<br>
  <b>Vasak pulk</b> – hoob / kõndimine<br>
  <b>Parem pulk</b> – juhtimine / pööramine<br>
  <b>Parem päästik</b> – laser ja nupud<br>
  <b>Vasak päästik</b> – pulsikiirus (hoia)<br>
  <b>A</b> autopiloot · <b>B</b> istu / tõuse<br>
  <b>X</b> välisvaade · <b>Y</b> järgmine sihtkoht
  <hr>
  <b>Arvutis:</b> klõpsa ekraanil, hiirega vaatad<br>
  <span class="k">W</span><span class="k">S</span> hoob / kõnd · <span class="k">A</span><span class="k">D</span> pööra · nooled kalle<br>
  <span class="k">Q</span><span class="k">E</span> veere · <span class="k">Shift</span> pulss · klõps / <span class="k">Space</span> laser<br>
  <span class="k">P</span> autopiloot · <span class="k">F</span> istu/tõuse · <span class="k">V</span> välisvaade<br>
  <span class="k">T</span> / <span class="k">1</span>–<span class="k">8</span> sihtkoht
  <br><button id="hideInfo">Peida</button>
</div>
<div id="cross"></div>

<script type="module">
import * as THREE from 'three';
import { VRButton } from 'three/addons/webxr/VRButton.js';
import { XRControllerModelFactory } from 'three/addons/webxr/XRControllerModelFactory.js';
import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js';

/* =========================================================
   ABIFUNKTSIOONID
   ========================================================= */
const PI = Math.PI, V3 = THREE.Vector3, Q4 = THREE.Quaternion;
const $ = id => document.getElementById(id);
const clamp = (v, a, b) => v < a ? a : v > b ? b : v;
const lerp = (a, b, t) => a + (b - a) * t;
const approach = (a, b, s) => a < b ? Math.min(b, a + s) : Math.max(b, a - s);
const nextFrame = () => new Promise(r => requestAnimationFrame(() => r()));
function mulberry(a) { return function () { a |= 0; a = a + 0x6D2B79F5 | 0; let t = Math.imul(a ^ a >>> 15, 1 | a); t = t + Math.imul(t ^ t >>> 7, 61 | t) ^ t; return ((t ^ t >>> 14) >>> 0) / 4294967296; }; }
const R0 = mulberry(20261024);
const rnd = (a = 0, b = 1) => a + R0() * (b - a);
const randUnit = (out = new V3()) => { do { out.set(rnd(-1, 1), rnd(-1, 1), rnd(-1, 1)); } while (out.lengthSq() > 1 || out.lengthSq() < 0.01); return out.normalize(); };
const _v = new V3(), _v2 = new V3(), _v3 = new V3(), _v4 = new V3(), _q = new Q4(), _q2 = new Q4(), _m = new THREE.Matrix4(), _e = new THREE.Euler();
const UP = new V3(0, 1, 0);
const fwdOf = (o, out = new V3()) => out.set(0, 0, -1).applyQuaternion(o.quaternion);
function setLoad(f, t) { $('barf').style.width = (f * 100) + '%'; if (t) $('load').querySelector('p').textContent = t; }
$('hideInfo').onclick = () => $('info').style.display = 'none';

function makeNoise(seed) {
  const r = mulberry(seed * 9973 + 17);
  const perm = new Uint8Array(256); for (let i = 0; i < 256; i++) perm[i] = i;
  for (let i = 255; i > 0; i--) { const j = (r() * (i + 1)) | 0; const t = perm[i]; perm[i] = perm[j]; perm[j] = t; }
  const val = new Float32Array(256); for (let i = 0; i < 256; i++) val[i] = r();
  const h = (x, y, z) => val[perm[(perm[(perm[x & 255] + y) & 255] + z) & 255]];
  function n3(x, y, z) {
    const X = Math.floor(x), Y = Math.floor(y), Z = Math.floor(z);
    const fx = x - X, fy = y - Y, fz = z - Z;
    const u = fx * fx * (3 - 2 * fx), v = fy * fy * (3 - 2 * fy), w = fz * fz * (3 - 2 * fz);
    const a = h(X, Y, Z), b = h(X + 1, Y, Z), c = h(X, Y + 1, Z), d = h(X + 1, Y + 1, Z);
    const e = h(X, Y, Z + 1), f = h(X + 1, Y, Z + 1), g = h(X, Y + 1, Z + 1), k = h(X + 1, Y + 1, Z + 1);
    const x1 = a + (b - a) * u, x2 = c + (d - c) * u, x3 = e + (f - e) * u, x4 = g + (k - g) * u;
    const y1 = x1 + (x2 - x1) * v, y2 = x3 + (x4 - x3) * v;
    return y1 + (y2 - y1) * w;
  }
  function fbm(x, y, z, oct = 5) { let s = 0, a = 0.5, f = 1, n = 0; for (let i = 0; i < oct; i++) { s += a * n3(x * f + i * 17.3, y * f + i * 5.1, z * f); n += a; a *= 0.5; f *= 2.03; } return s / n; }
  return { n3, fbm };
}
const con = (h, k = 2.3) => clamp((h - 0.5) * k + 0.5, 0, 1);

/* =========================================================
   RENDERDAJA JA STSEEN
   ========================================================= */
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.setSize(innerWidth, innerHeight);
renderer.outputColorSpace = THREE.SRGBColorSpace;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.15;
renderer.xr.enabled = true;
renderer.xr.setReferenceSpaceType('local-floor');
document.body.appendChild(renderer.domElement);

const scene = new THREE.Scene();
scene.background = new THREE.Color('#010209');
const camera = new THREE.PerspectiveCamera(72, innerWidth / innerHeight, 0.05, 60000);
camera.rotation.order = 'YXZ';
addEventListener('resize', () => { camera.aspect = innerWidth / innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth, innerHeight); });

const SUN_DIR = new V3(0.72, 0.28, 0.35).normalize();
const sunLight = new THREE.DirectionalLight('#fff0d8', 3.4);
sunLight.position.copy(SUN_DIR).multiplyScalar(100);
scene.add(sunLight);
scene.add(new THREE.HemisphereLight('#5868a0', '#140a1c', 0.45));

/* =========================================================
   TEKSTUURID (kõik joonistatakse koodis)
   ========================================================= */
function cnv(w, h) { const c = document.createElement('canvas'); c.width = w; c.height = h; return c; }
function ctex(c, srgb = true) { const t = new THREE.CanvasTexture(c); if (srgb) t.colorSpace = THREE.SRGBColorSpace; t.anisotropy = Math.min(8, renderer.capabilities.getMaxAnisotropy()); return t; }
function hx(h) { const n = parseInt(h.slice(1), 16); return [(n >> 16) & 255, (n >> 8) & 255, n & 255]; }
function rgbs(c, k = 1) { return `rgb(${clamp(c[0] * k, 0, 255) | 0},${clamp(c[1] * k, 0, 255) | 0},${clamp(c[2] * k, 0, 255) | 0})`; }
function hexA(h, a) { const c = hx(h); return `rgba(${c[0]},${c[1]},${c[2]},${a})`; }
function mix(a, b, t) { return [a[0] + (b[0] - a[0]) * t, a[1] + (b[1] - a[1]) * t, a[2] + (b[2] - a[2]) * t]; }
function rr(g, x, y, w, h, r) { g.beginPath(); g.moveTo(x + r, y); g.arcTo(x + w, y, x + w, y + h, r); g.arcTo(x + w, y + h, x, y + h, r); g.arcTo(x, y + h, x, y, r); g.arcTo(x, y, x + w, y, r); g.closePath(); }

function hazard(g, x, y, w, h) {
  g.save(); g.beginPath(); g.rect(x, y, w, h); g.clip();
  g.fillStyle = '#d6ae22'; g.fillRect(x, y, w, h); g.fillStyle = '#1b1b1b';
  for (let i = -h; i < w + h; i += 14) { g.beginPath(); g.moveTo(x + i, y); g.lineTo(x + i + 7, y); g.lineTo(x + i + 7 - h, y + h); g.lineTo(x + i - h, y + h); g.closePath(); g.fill(); }
  g.restore();
}
function hullTexture(base, accent, dark, seed, hz = true) {
  const c = cnv(512, 512), g = c.getContext('2d'), r = mulberry(seed);
  const B = hx(base), A = hx(accent), D = hx(dark);
  g.fillStyle = rgbs(B, 0.6); g.fillRect(0, 0, 512, 512);
  const panel = (x, y, w, h, d) => {
    if (d > 0 && (w > 40 || h > 40) && r() < 0.88) {
      if (w >= h) { const s = Math.max(8, Math.round(w * (0.3 + r() * 0.4) / 8) * 8); panel(x, y, s, h, d - 1); panel(x + s, y, w - s, h, d - 1); }
      else { const s = Math.max(8, Math.round(h * (0.3 + r() * 0.4) / 8) * 8); panel(x, y, w, s, d - 1); panel(x, y + s, w, h - s, d - 1); }
      return;
    }
    const t = r(); const col = t < 0.13 ? A : t < 0.21 ? D : B; const k = 0.84 + r() * 0.26;
    g.fillStyle = rgbs(col, k); g.fillRect(x + 1.5, y + 1.5, w - 3, h - 3);
    if (hz && r() < 0.07 && w > 30 && h > 14) hazard(g, x + 4, y + 4, w - 8, Math.min(h - 8, 20));
    if (r() < 0.22 && w > 24 && h > 24) { g.fillStyle = 'rgba(0,0,0,0.35)'; const n = 3 + ((r() * 4) | 0); for (let i = 0; i < n && 8 + i * 6 < h - 6; i++) g.fillRect(x + 6, y + 8 + i * 6, w - 12, 2.5); }
    if (r() < 0.35) { g.fillStyle = 'rgba(0,0,0,0.45)'; for (const [px, py] of [[x + 5, y + 5], [x + w - 7, y + 5], [x + 5, y + h - 7], [x + w - 7, y + h - 7]]) g.fillRect(px, py, 2, 2); }
    g.strokeStyle = 'rgba(0,0,0,0.6)'; g.lineWidth = 1.5; g.strokeRect(x + 0.75, y + 0.75, w - 1.5, h - 1.5);
    g.strokeStyle = 'rgba(255,255,255,0.22)'; g.beginPath(); g.moveTo(x + 2.5, y + h - 2.5); g.lineTo(x + 2.5, y + 2.5); g.lineTo(x + w - 2.5, y + 2.5); g.stroke();
  };
  panel(0, 0, 512, 512, 7);
  for (let i = 0; i < 2600; i++) { g.fillStyle = `rgba(${r() < 0.5 ? '0,0,0' : '90,70,50'},${r() * 0.07})`; g.fillRect(r() * 512, r() * 512, 1 + r() * 5, 1 + r() * 2); }
  const tx = ctex(c); tx.wrapS = tx.wrapT = THREE.RepeatWrapping; return tx;
}
function floorTex() {
  const c = cnv(256, 256), g = c.getContext('2d');
  g.fillStyle = '#23272d'; g.fillRect(0, 0, 256, 256);
  for (let y = 0; y < 256; y += 16) for (let x = 0; x < 256; x += 16) {
    g.fillStyle = ((x + y) / 16) % 2 ? '#30353c' : '#2a2f35'; g.fillRect(x + 1, y + 1, 14, 14);
    g.fillStyle = 'rgba(255,255,255,0.07)'; g.fillRect(x + 3, y + 7, 10, 2);
  }
  g.strokeStyle = '#121418'; g.lineWidth = 4; g.strokeRect(2, 2, 252, 252);
  const t = ctex(c); t.wrapS = t.wrapT = THREE.RepeatWrapping; return t;
}
function solarTex() {
  const c = cnv(256, 256), g = c.getContext('2d'); g.fillStyle = '#c8ccd0'; g.fillRect(0, 0, 256, 256);
  for (let y = 0; y < 256; y += 32) for (let x = 0; x < 256; x += 32) { const gr = g.createLinearGradient(x, y, x + 32, y + 32); gr.addColorStop(0, '#2a52a8'); gr.addColorStop(1, '#0f2458'); g.fillStyle = gr; g.fillRect(x + 2, y + 2, 28, 28); }
  const t = ctex(c); t.wrapS = t.wrapT = THREE.RepeatWrapping; return t;
}
function glowTex(stops, size = 128) {
  const c = cnv(size, size), g = c.getContext('2d'); const h = size / 2;
  const gr = g.createRadialGradient(h, h, 0, h, h, h); stops.forEach(([o, col]) => gr.addColorStop(o, col));
  g.fillStyle = gr; g.fillRect(0, 0, size, size); return ctex(c);
}
const GLOW = glowTex([[0, 'rgba(255,255,255,1)'], [0.18, 'rgba(255,255,255,0.65)'], [0.45, 'rgba(255,255,255,0.16)'], [1, 'rgba(255,255,255,0)']]);
const DOT = glowTex([[0, 'rgba(255,255,255,1)'], [0.35, 'rgba(255,255,255,0.8)'], [1, 'rgba(255,255,255,0)']], 32);
function ringTex() {
  const c = cnv(256, 256), g = c.getContext('2d');
  g.strokeStyle = '#fff'; g.lineWidth = 10; g.beginPath(); g.arc(128, 128, 110, 0, PI * 2); g.stroke();
  g.lineWidth = 6; for (let i = 0; i < 4; i++) { const a = i * PI / 2; g.beginPath(); g.moveTo(128 + Math.cos(a) * 82, 128 + Math.sin(a) * 82); g.lineTo(128 + Math.cos(a) * 124, 128 + Math.sin(a) * 124); g.stroke(); }
  return ctex(c);
}
const RING = ringTex();
function cloudTex(seed) {
  const c = cnv(256, 256), g = c.getContext('2d'), r = mulberry(seed);
  for (let i = 0; i < 70; i++) {
    const a = r() * PI * 2, d = Math.pow(r(), 0.7) * 80; const x = 128 + Math.cos(a) * d, y = 128 + Math.sin(a) * d * 0.8; const s = 20 + r() * 60;
    const gr = g.createRadialGradient(x, y, 0, x, y, s); gr.addColorStop(0, `rgba(255,255,255,${0.06 + r() * 0.1})`); gr.addColorStop(1, 'rgba(255,255,255,0)');
    g.fillStyle = gr; g.fillRect(0, 0, 256, 256);
  }
  return ctex(c);
}
function glowSprite(color, size, opacity = 1) {
  const m = new THREE.SpriteMaterial({ map: GLOW, color, blending: THREE.AdditiveBlending, transparent: true, depthWrite: false, toneMapped: false, opacity });
  const s = new THREE.Sprite(m); s.scale.setScalar(size); return s;
}

/* =========================================================
   GEOMEETRIA EHITAJA (paljud osad liidetakse üheks)
   ========================================================= */
function tBox(w, h, d, tile = 4) {
  const g = new THREE.BoxGeometry(w, h, d); const uv = g.attributes.uv;
  const dims = [[d, h], [d, h], [w, d], [w, d], [w, h], [w, h]]; const ou = Math.random(), ov = Math.random();
  for (let f = 0; f < 6; f++) for (let i = 0; i < 4; i++) { const k = f * 4 + i; uv.setXY(k, ou + uv.getX(k) * dims[f][0] / tile, ov + uv.getY(k) * dims[f][1] / tile); }
  return g;
}
function taper(g, tx, ty) { // kitsendab -Z otsa
  g.computeBoundingBox(); const p = g.attributes.position; const z0 = g.boundingBox.min.z, z1 = g.boundingBox.max.z;
  for (let i = 0; i < p.count; i++) { const t = (z1 - p.getZ(i)) / (z1 - z0); p.setX(i, p.getX(i) * (1 + (tx - 1) * t)); p.setY(i, p.getY(i) * (1 + (ty - 1) * t)); }
  g.computeVertexNormals(); return g;
}
const cylZ = (r, len, seg = 16, r2 = r) => new THREE.CylinderGeometry(r2, r, len, seg).rotateX(PI / 2);
const cylY = (r, len, seg = 16) => new THREE.CylinderGeometry(r, r, len, seg);
const disc = (r) => new THREE.CircleGeometry(r, 20);
class Builder {
  constructor() { this.parts = {}; }
  add(key, geo, x = 0, y = 0, z = 0, rx = 0, ry = 0, rz = 0) {
    let g = geo.index ? geo.toNonIndexed() : geo;
    g.applyMatrix4(_m.compose(_v.set(x, y, z), _q.setFromEuler(_e.set(rx, ry, rz)), _v2.set(1, 1, 1)));
    for (const k of Object.keys(g.attributes)) if (!['position', 'normal', 'uv'].includes(k)) g.deleteAttribute(k);
    (this.parts[key] ??= []).push(g); return this;
  }
  mirror(key, fn, x, y, z, rx = 0, ry = 0, rz = 0) { this.add(key, fn(), x, y, z, rx, ry, rz); this.add(key, fn(), -x, y, z, rx, -ry, -rz); return this; }
  geos() { const o = {}; for (const k in this.parts) o[k] = mergeGeometries(this.parts[k], false); return o; }
  build(mats) { return meshFrom(this.geos(), mats); }
}
function meshFrom(geos, mats) { const g = new THREE.Group(); for (const k in geos) g.add(new THREE.Mesh(geos[k], mats[k])); return g; }

function shipMats(base, accent, dark, seed) {
  return {
    hull: new THREE.MeshStandardMaterial({ map: hullTexture(base, accent, dark, seed), roughness: 0.45, metalness: 0.5 }),
    accent: new THREE.MeshStandardMaterial({ map: hullTexture(accent, base, dark, seed + 3), roughness: 0.55, metalness: 0.35 }),
    dark: new THREE.MeshStandardMaterial({ map: hullTexture(dark, '#5a5f68', '#1c1e22', seed + 5, false), roughness: 0.5, metalness: 0.7 }),
    glass: new THREE.MeshStandardMaterial({ color: '#0c1c28', roughness: 0.06, metalness: 0.95, emissive: '#0a4a66', emissiveIntensity: 0.6 }),
    glow: new THREE.MeshBasicMaterial({ color: new THREE.Color(0.6, 1.6, 2.4), toneMapped: false }),
    red: new THREE.MeshBasicMaterial({ color: new THREE.Color(2, 0.3, 0.2), toneMapped: false }),
  };
}

/* ---------- Kaubalaev (laiem, oranžide paneelidega) ---------- */
function buildHaulerGeo() {
  const b = new Builder(), r = mulberry(77);
  b.add('hull', tBox(12, 4, 22), 0, 0, 0);
  b.add('hull', tBox(9, 2, 14), 0, 3, 2);
  b.add('hull', taper(tBox(8, 2.2, 7), 0.7, 0.6), 0, -0.4, -14);
  b.add('hull', tBox(6, 2.4, 5), 0, 1.6, -11);
  b.add('glass', tBox(5, 1.1, 0.3), 0, 2.2, -13.4, -0.45);
  b.mirror('accent', () => tBox(5, 3, 16), 9, -0.5, 1);
  b.mirror('dark', () => tBox(3, 1.5, 8), 6.5, 0, 0);
  b.mirror('hull', () => tBox(7, 0.6, 10), 12.5, -0.6, 3, 0, 0, -0.12);
  b.mirror('accent', () => tBox(2.5, 0.9, 4), 15.6, -1.0, 5);
  for (const x of [-2.5, 0, 2.5]) { b.add('accent', cylZ(1.1, 8, 14), x, 4.9, 3); for (const z of [0, 6]) b.add('dark', cylZ(1.25, 0.4, 14), x, 4.9, z); }
  b.mirror('dark', () => cylZ(1.9, 4, 16, 2.2), 3.8, 0.2, 12);
  b.mirror('dark', () => cylZ(1.1, 3, 12), 9, -0.5, 10);
  b.mirror('glow', () => disc(1.6), 3.8, 0.2, 14.05);
  b.mirror('glow', () => disc(0.9), 9, -0.5, 11.55);
  b.add('dark', tBox(0.25, 3.5, 0.25), 3.2, 5.8, 7); b.add('dark', tBox(0.25, 2.5, 0.25), -3.6, 5.3, 9);
  b.mirror('dark', () => cylZ(0.2, 3, 6), 2, -0.6, -18);
  for (let i = 0; i < 26; i++) { const w = 0.5 + r() * 1.6, h = 0.3 + r() * 0.9, d = 0.5 + r() * 2; b.add(r() < 0.5 ? 'dark' : 'hull', tBox(w, h, d), (r() - 0.5) * 11, 2 + h / 2, (r() - 0.5) * 19); }
  return { geos: b.geos(), engines: [[3.8, 0.2, 14.6, 1.6], [-3.8, 0.2, 14.6, 1.6], [9, -0.5, 12, 0.9], [-9, -0.5, 12, 0.9]], navs: [[16.8, -1, 5, 'r'], [-16.8, -1, 5, 'g']] };
}
/* ---------- Hävitaja (pikk ja terav) ---------- */
function buildFighterGeo() {
  const b = new Builder();
  b.add('hull', tBox(4.2, 2.6, 20), 0, 0, 1);
  b.add('hull', taper(tBox(4.2, 2.6, 10), 0.3, 0.35), 0, -0.2, -14);
  b.add('glass', taper(tBox(2.4, 1.3, 6), 0.5, 0.4), 0, 1.75, -7.5);
  b.add('dark', tBox(2.2, 1.6, 15), 0, 1.9, 4);
  b.mirror('hull', () => tBox(9, 0.5, 8), 6, -0.4, 6, 0, -0.35, 0);
  b.mirror('dark', () => cylZ(1, 8, 12), 10.5, -0.4, 7.5);
  b.mirror('accent', () => tBox(0.4, 4, 5), 2.3, 2.8, 8, 0, 0, -0.3);
  b.mirror('hull', () => tBox(1.6, 2, 4.5), 3.1, 4.6, 8.5);
  b.mirror('dark', () => cylZ(1.15, 3, 14, 1.35), 1.3, 0, 12);
  b.mirror('dark', () => tBox(0.3, 0.3, 6), 1.2, -1.4, -6);
  b.mirror('accent', () => tBox(1.2, 0.6, 4), 4.5, -0.2, -2);
  b.mirror('glow', () => disc(1.0), 1.3, 0, 13.55);
  b.mirror('glow', () => disc(0.8), 10.5, -0.4, 11.55);
  return { geos: b.geos(), engines: [[1.3, 0, 14, 1.0], [-1.3, 0, 14, 1.0], [10.5, -0.4, 12, 0.8], [-10.5, -0.4, 12, 0.8]], navs: [[10.5, -0.4, 4, 'r'], [-10.5, -0.4, 4, 'g']] };
}

/* =========================================================
   MÄNGU OLEK
   ========================================================= */
const SAVE_KEY = 'tahesadam_v1';
const DEFAULT_S = () => ({ crystals: 0, credits: 0, fuel: 100, shield: 100, up: { laser: 0, engine: 0, tank: 0, shield: 0, magnet: 0 }, discovered: [], mined: 0, target: -1 });
const S = DEFAULT_S();
try { const d = JSON.parse(localStorage.getItem(SAVE_KEY) || 'null'); if (d) { Object.assign(S, d); S.up = Object.assign(DEFAULT_S().up, d.up || {}); } } catch (e) { }
const stat = {
  get maxFuel() { return 100 + 50 * S.up.tank; }, get maxShield() { return 100 + 50 * S.up.shield; },
  get dmg() { return 1 + S.up.laser; }, get pulse() { return 900 + 300 * S.up.engine; },
  get manual() { return 120 + 40 * S.up.engine; }, get magnet() { return 220 + 160 * S.up.magnet; }
};
function save() { try { localStorage.setItem(SAVE_KEY, JSON.stringify(S)); } catch (e) { } }
setInterval(save, 5000);
const R = { mode: 'seat', prevMode: 'seat', walkPos: null, autopilot: false, throttle: 0, speed: 0, boost: false, yaw: 0, pitch: 0, roll: 0,
  bump: new V3(), vel: new V3(), lastHit: -9, lastWarn: -9, faceT: 0, nearStation: false, inField: false, fireCd: 0, gun: 0, time: 0,
  msg: '', msgT: 0, queue: [], resetArm: 0, lowFuel: false, mouseFire: false, snapLock: false };

/* =========================================================
   MAAILM: TÄHED, UDUKOGUD, PÄIKE
   ========================================================= */
const sky = new THREE.Group(); scene.add(sky);
function buildSky() {
  const mk = (N, fn, size, op) => {
    const pos = new Float32Array(N * 3), col = new Float32Array(N * 3);
    for (let i = 0; i < N; i++) { const [p, c] = fn(i); pos.set([p.x, p.y, p.z], i * 3); col.set(c, i * 3); }
    const g = new THREE.BufferGeometry(); g.setAttribute('position', new THREE.BufferAttribute(pos, 3)); g.setAttribute('color', new THREE.BufferAttribute(col, 3));
    const pts = new THREE.Points(g, new THREE.PointsMaterial({ size, sizeAttenuation: false, map: DOT, vertexColors: true, transparent: true, opacity: op, depthWrite: false, blending: THREE.AdditiveBlending, toneMapped: false }));
    pts.frustumCulled = false; pts.renderOrder = -10; sky.add(pts);
  };
  const starCol = () => { const t = rnd(); const b = rnd(0.35, 1); return t < 0.15 ? [0.7 * b, 0.8 * b, 1.3 * b] : t < 0.25 ? [1.3 * b, 0.85 * b, 0.6 * b] : [b, b, b]; };
  mk(9000, () => [randUnit(new V3()).multiplyScalar(45000), starCol()], 2.2, 0.9);
  mk(500, () => [randUnit(new V3()).multiplyScalar(45000), starCol().map(v => v * 1.6)], 4.2, 1);
  // Linnutee vöö
  const n = new V3(0.25, 0.92, 0.3).normalize(), u = new V3().crossVectors(n, new V3(1, 0, 0)).normalize(), w = new V3().crossVectors(n, u);
  const gauss = () => (rnd() + rnd() + rnd() + rnd() - 2) / 2;
  mk(18000, () => { const t = rnd(0, PI * 2); const p = u.clone().multiplyScalar(Math.cos(t)).addScaledVector(w, Math.sin(t)).addScaledVector(n, gauss() * 0.16).normalize().multiplyScalar(44000); const b = rnd(0.2, 0.6); return [p, [b * 0.8, b * 0.85, b * 1.1]]; }, 1.8, 0.8);
  // udukogud
  const NEB = ['#6a34e0', '#167a8a', '#c02c7a', '#2a50d0', '#8a38d8', '#b85a30'];
  const ct = [cloudTex(1), cloudTex(2), cloudTex(3)];
  for (let i = 0; i < 18; i++) {
    const t = rnd(0, PI * 2); const p = u.clone().multiplyScalar(Math.cos(t)).addScaledVector(w, Math.sin(t)).addScaledVector(n, rnd(-0.22, 0.22)).normalize().multiplyScalar(42000);
    const m = new THREE.SpriteMaterial({ map: ct[i % 3], color: NEB[i % NEB.length], blending: THREE.AdditiveBlending, transparent: true, depthWrite: false, opacity: rnd(0.25, 0.5), toneMapped: false, rotation: rnd(0, PI * 2) });
    const s = new THREE.Sprite(m); s.position.copy(p); s.scale.setScalar(rnd(14000, 32000)); s.renderOrder = -9; sky.add(s);
  }
  // päike
  const sp = SUN_DIR.clone().multiplyScalar(44000);
  for (const [c, sz, op] of [['#fff6e0', 2600, 1], ['#ffe2a0', 9000, 0.9], ['#ffb060', 26000, 0.35]]) { const s = glowSprite(c, sz, op); s.position.copy(sp); s.renderOrder = -8; sky.add(s); }
}

/* =========================================================
   KOSMOSEJAAM
   ========================================================= */
const STATION_POS = new V3(0, 0, 0);
let station = null;
function buildStation() {
  const SM = shipMats('#cfd4d8', '#2f6fa8', '#3a3e44', 401);
  SM.solar = new THREE.MeshStandardMaterial({ map: solarTex(), roughness: 0.25, metalness: 0.8 });
  SM.lightW = new THREE.MeshBasicMaterial({ color: new THREE.Color(1.6, 1.25, 0.8), toneMapped: false });
  const st = new THREE.Group(); st.position.copy(STATION_POS); scene.add(st);
  const b = new Builder();
  b.add('hull', new THREE.SphereGeometry(60, 32, 20));
  b.add('hull', cylY(20, 330, 24));
  for (const y of [-120, 120]) b.add('accent', cylY(34, 26, 24), 0, y, 0);
  for (const y of [-165, 165]) b.add('dark', cylY(12, 10, 16), 0, y, 0);
  b.add('dark', tBox(300, 4, 4, 8), 0, 135, 0); b.add('dark', tBox(4, 4, 300, 8), 0, -135, 0);
  for (const s of [-1, 1]) { b.add('solar', tBox(110, 0.8, 44, 20), s * 95, 135, 0); b.add('solar', tBox(44, 0.8, 110, 20), 0, -135, s * 95); }
  b.add('dark', tBox(40, 22, 14), 0, 0, 57); b.add('glow', tBox(32, 14, 0.5), 0, 0, 64.3);
  for (let i = 0; i < 6; i++) b.add('lightW', tBox(1.4, 1.4, 1.4), -15 + i * 6, -12.5, 64);
  st.add(b.build(SM));
  const rb = new Builder();
  rb.add('hull', new THREE.TorusGeometry(220, 16, 18, 120), 0, 0, 0, PI / 2, 0, 0);
  for (let i = 0; i < 4; i++) { const a = i * PI / 2 + PI / 4; rb.add('dark', cylY(5, 200, 10), -Math.cos(a) * 120, 0, Math.sin(a) * 120, 0, a, PI / 2); }
  for (let i = 0; i < 16; i++) { const a = i / 16 * 2 * PI; rb.add('accent', tBox(14, 36, 14), Math.cos(a) * 220, 0, Math.sin(a) * 220, 0, PI / 2 - a, 0); }
  for (let i = 0; i < 64; i++) { const a = (i + 0.5) / 64 * 2 * PI; rb.add('lightW', tBox(5, 2.2, 0.6), Math.cos(a) * 236.3, 0, Math.sin(a) * 236.3, 0, PI / 2 - a, 0); }
  const ring = rb.build(SM); st.add(ring);
  const beacons = [];
  for (const y of [-172, 172]) { const s = glowSprite('#ff3030', 30); s.position.set(0, y, 0); st.add(s); beacons.push(s); }
  for (let i = 0; i < 4; i++) { const s = glowSprite('#40c0ff', 40, 0.8); s.position.set(0, 0, 66); st.add(s); }
  station = { st, ring, beacons };
}

/* =========================================================
   ASTEROIDIVÄLI
   ========================================================= */
const FIELD_POS = new V3(3500, 200, -5000);
const AST = [];
let rockIMs = [], debrisGeo, debrisMat;
function makeRockGeo(seed, detail) {
  const g = new THREE.IcosahedronGeometry(1, detail); const N = makeNoise(seed + 100); const pos = g.attributes.position;
  const col = new Float32Array(pos.count * 3); const r = mulberry(seed); const sq = [0.65 + r() * 0.4, 0.55 + r() * 0.35, 0.7 + r() * 0.4];
  const lin = v => Math.pow(v / 255, 2.2);
  for (let i = 0; i < pos.count; i++) {
    _v.fromBufferAttribute(pos, i).normalize();
    const n = N.fbm(_v.x * 1.4 + 3, _v.y * 1.4, _v.z * 1.4, 4), cr = N.n3(_v.x * 4, _v.y * 4, _v.z * 4);
    const d = 0.7 + n * 0.62 - (cr > 0.76 ? (cr - 0.76) * 1.3 : 0);
    pos.setXYZ(i, _v.x * d * sq[0], _v.y * d * sq[1], _v.z * d * sq[2]);
    const t = N.n3(_v.x * 3 + 9, _v.y * 3, _v.z * 3); const c = mix([70, 62, 56], [150, 138, 122], t); const dk = cr > 0.76 ? 0.65 : 1;
    col[i * 3] = lin(c[0] * dk); col[i * 3 + 1] = lin(c[1] * dk); col[i * 3 + 2] = lin(c[2] * dk);
  }
  g.setAttribute('color', new THREE.BufferAttribute(col, 3)); g.computeVertexNormals(); return g;
}
function buildField() {
  const mat = new THREE.MeshStandardMaterial({ vertexColors: true, roughness: 0.95, metalness: 0.05, flatShading: true });
  const geos = [0, 1, 2, 3, 4].map(i => makeRockGeo(i * 13, 2));
  rockIMs = geos.map(g => { const im = new THREE.InstancedMesh(g, mat, 30); im.frustumCulled = false; im.instanceMatrix.setUsage(THREE.DynamicDrawUsage); scene.add(im); return im; });
  for (let i = 0; i < 150; i++) {
    const a = { im: rockIMs[i % 5], idx: (i / 5) | 0, pos: new V3(), r: 0, hp: 0, alive: true, q: new Q4(), axis: randUnit(new V3()), spin: rnd(0.05, 0.3), respawn: 0 };
    placeRock(a, true); AST.push(a);
  }
  // kaugemad dekoratiivsed kivid
  const dg = makeRockGeo(999, 1); const dec = new THREE.InstancedMesh(dg, mat, 650); dec.frustumCulled = false;
  for (let i = 0; i < 650; i++) {
    const p = randUnit(_v).multiplyScalar(rnd(950, 1800)); p.y *= 0.45; p.add(FIELD_POS);
    _q.setFromEuler(_e.set(rnd(0, 6), rnd(0, 6), rnd(0, 6))); const s = rnd(3, 22);
    dec.setMatrixAt(i, _m.compose(p, _q, _v2.set(s, s, s)));
  }
  scene.add(dec);
  // roheline tolmuudu välja ümber
  for (let i = 0; i < 8; i++) { const s = new THREE.Sprite(new THREE.SpriteMaterial({ map: cloudTex(40 + i), color: '#8ab04a', blending: THREE.AdditiveBlending, transparent: true, depthWrite: false, opacity: 0.16, toneMapped: false })); s.position.copy(randUnit(_v).multiplyScalar(rnd(200, 1300))).add(FIELD_POS); s.scale.setScalar(rnd(1400, 2600)); scene.add(s); }
  debrisGeo = new THREE.IcosahedronGeometry(1, 0);
  debrisMat = new THREE.MeshStandardMaterial({ color: '#7a6e62', roughness: 1, flatShading: true });
}
function placeRock(a, init) {
  for (let k = 0; k < 20; k++) {
    randUnit(a.pos).multiplyScalar(Math.pow(rnd(), 0.5) * 880); a.pos.y *= 0.45; a.pos.add(FIELD_POS);
    if (init || a.pos.distanceTo(ship.position) > 350) break;
  }
  const t = rnd(); a.r = 5 + t * t * 26; a.hp = Math.max(2, Math.round(a.r / 3.2)); a.alive = true;
  a.q.setFromEuler(_e.set(rnd(0, 6), rnd(0, 6), rnd(0, 6)));
}
function updateAsteroids(dt) {
  for (const a of AST) {
    if (!a.alive) { a.respawn -= dt; if (a.respawn <= 0) placeRock(a, false); else { a.im.setMatrixAt(a.idx, _m.makeScale(0, 0, 0)); continue; } }
    _q.setFromAxisAngle(a.axis, a.spin * dt); a.q.multiply(_q);
    a.im.setMatrixAt(a.idx, _m.compose(a.pos, a.q, _v.set(a.r, a.r, a.r)));
  }
  for (const im of rockIMs) im.instanceMatrix.needsUpdate = true;
}

/* =========================================================
   PLANEEDID
   ========================================================= */
const PLANETS = [
  { name: 'Viridia', p: [5600, -300, -7200], r: 1100, kind: 'toxic', atm: '#b8f060', clouds: true },
  { name: 'Okeanos', p: [-6500, 600, -2800], r: 800, kind: 'ocean', atm: '#6ab4ff', clouds: true },
  { name: 'Kaljula', p: [-2600, -900, -10500], r: 550, kind: 'desert', atm: '#ffb070' },
  { name: 'Külmaste', p: [9500, 1400, 2500], r: 450, kind: 'ice', atm: '#aee6ff' },
  { name: 'Hõõguja', p: [-9000, -1600, 6500], r: 500, kind: 'lava', atm: '#ff6a2a' },
  { name: 'Vöölane', p: [2500, 2600, 15000], r: 2000, kind: 'gas', atm: '#ffd9a0', rings: true },
];
const PL = [];
const ST = k => k.map(([t, h]) => [t, hx(h)]);
const STOPS = {
  toxic: ST([[0, '#1a3418'], [0.35, '#3c6a24'], [0.6, '#8ab43c'], [0.8, '#c6d870'], [1, '#eef0b0']]),
  oceanW: ST([[0, '#061a4a'], [0.7, '#0f3f8a'], [1, '#2a78c0']]),
  oceanL: ST([[0, '#c2b280'], [0.08, '#3a8a3c'], [0.45, '#2a6a30'], [0.7, '#7a6a48'], [0.9, '#a89c8a'], [1, '#f4f6f8']]),
  desert: ST([[0, '#4a2412'], [0.3, '#8a4220'], [0.55, '#c27a3e'], [0.8, '#e0aa6a'], [1, '#f6deae']]),
  ice: ST([[0, '#4a6a8c'], [0.4, '#8ab6d8'], [0.75, '#d0e6f6'], [1, '#ffffff']]),
  lava: ST([[0, '#0e0a0a'], [0.5, '#2a1a16'], [1, '#4a342a']]),
  gas: ST([[0, '#7a4a2e'], [0.25, '#c9a27a'], [0.5, '#efdcb8'], [0.75, '#b07a52'], [1, '#f4ead6']]),
};
function ramp(st, t) { t = clamp(t, 0, 1); for (let i = 1; i < st.length; i++) if (t <= st[i][0]) { const a = st[i - 1], b = st[i]; return mix(a[1], b[1], (t - a[0]) / ((b[0] - a[0]) || 1)); } return st[st.length - 1][1]; }
async function planetTextures(p, seed) {
  const W = 512, H = 256, N = makeNoise(seed);
  const mkI = () => { const c = cnv(W, H); const g = c.getContext('2d'); const im = g.createImageData(W, H); return { c, g, im, d: im.data }; };
  const M = mkI(); const E = p.kind === 'lava' ? mkI() : null; const C = p.clouds ? mkI() : null;
  for (let y = 0; y < H; y++) {
    const lat = (y + 0.5) / H * PI, sl = Math.sin(lat), ny = Math.cos(lat);
    for (let x = 0; x < W; x++) {
      const lon = (x + 0.5) / W * 2 * PI, nx = -Math.cos(lon) * sl, nz = Math.sin(lon) * sl, i = (y * W + x) * 4;
      let col;
      switch (p.kind) {
        case 'toxic': { const w = N.fbm(nx * 1.5, ny * 1.5, nz * 1.5, 3); const h = con(N.fbm(nx * 2.4 + w, ny * 2.4 + w, nz * 2.4, 5)); col = ramp(STOPS.toxic, h); if (h < 0.3) col = mix(col, [160, 235, 70], (0.3 - h) * 2.2); break; }
        case 'ocean': { const h = con(N.fbm(nx * 1.7, ny * 1.7, nz * 1.7, 6), 2.6); col = h < 0.52 ? ramp(STOPS.oceanW, h / 0.52) : ramp(STOPS.oceanL, (h - 0.52) / 0.48); if (Math.abs(ny) > 0.83 + (N.n3(nx * 8, ny * 8, nz * 8) - 0.5) * 0.12) col = [232, 240, 248]; break; }
        case 'desert': { const h = con(N.fbm(nx * 2.2, ny * 2.2, nz * 2.2, 5)); col = ramp(STOPS.desert, h + Math.sin(ny * 34 + h * 9) * 0.04); break; }
        case 'ice': { const h = con(N.fbm(nx * 2, ny * 2, nz * 2, 5)); const cr = Math.abs(N.fbm(nx * 5 + 3, ny * 5, nz * 5, 4) - 0.5); col = ramp(STOPS.ice, h); if (cr < 0.018) col = mix(col, [50, 100, 165], 0.75 * (1 - cr / 0.018)); break; }
        case 'lava': { const h = con(N.fbm(nx * 2.2, ny * 2.2, nz * 2.2, 5)); const rid = 1 - Math.abs(N.fbm(nx * 3.2 + 5, ny * 3.2, nz * 3.2, 5) * 2 - 1); col = ramp(STOPS.lava, h); let e = 0; if (rid > 0.84) { e = Math.min(1, (rid - 0.84) / 0.12); col = mix(col, [255, 120, 30], e); } E.d[i] = 255 * e; E.d[i + 1] = 105 * e; E.d[i + 2] = 20 * e; E.d[i + 3] = 255; break; }
        case 'gas': { const w = N.fbm(nx * 2, ny * 6, nz * 2, 4); const t = ny * 2.2 + (w - 0.5) * 0.9; const band = (Math.sin(t * 11) + 1) / 2 * 0.6 + N.fbm(nx, ny * 14, nz, 3) * 0.4; col = ramp(STOPS.gas, band); const dx = lon - 3.6, dy = lat - 1.95; const sd = dx * dx / 0.09 + dy * dy / 0.012; if (sd < 1) col = mix(col, [200, 105, 66], (1 - sd) * 0.85); break; }
      }
      M.d[i] = col[0]; M.d[i + 1] = col[1]; M.d[i + 2] = col[2]; M.d[i + 3] = 255;
      if (C) { const cv = con(N.fbm(nx * 3 + 9, ny * 4.2, nz * 3, 5), 2.6); const a = clamp((cv - 0.5) / 0.28, 0, 1); const cc = p.kind === 'toxic' ? [220, 236, 150] : [255, 255, 255]; C.d[i] = cc[0]; C.d[i + 1] = cc[1]; C.d[i + 2] = cc[2]; C.d[i + 3] = a * (p.kind === 'toxic' ? 170 : 235); }
    }
    if (y % 32 === 31) await nextFrame();
  }
  const fin = o => { if (!o) return null; o.g.putImageData(o.im, 0, 0); return ctex(o.c); };
  return { map: fin(M), emis: fin(E), clouds: fin(C) };
}
function atmMat(color, power, intensity) {
  return new THREE.ShaderMaterial({
    uniforms: { c: { value: new THREE.Color(color) }, p: { value: power }, k: { value: intensity }, sunDir: { value: SUN_DIR } },
    vertexShader: `varying vec3 vN; varying vec3 vV; varying vec3 vW;
      void main(){ vec4 mv = modelViewMatrix*vec4(position,1.0); vN = normalize(normalMatrix*normal); vV = normalize(-mv.xyz); vW = normalize(mat3(modelMatrix)*normal); gl_Position = projectionMatrix*mv; }`,
    fragmentShader: `uniform vec3 c; uniform float p; uniform float k; uniform vec3 sunDir; varying vec3 vN; varying vec3 vV; varying vec3 vW;
      void main(){ float f = pow(1.0-max(dot(normalize(vN),normalize(vV)),0.0), p); float lit = 0.18+0.82*smoothstep(-0.35,0.45,dot(normalize(vW),sunDir)); gl_FragColor = vec4(c*f*k*lit,1.0); }`,
    blending: THREE.AdditiveBlending, transparent: true, depthWrite: false
  });
}
async function buildPlanets() {
  for (let i = 0; i < PLANETS.length; i++) {
    const p = PLANETS[i]; p.pos = new V3(...p.p);
    setLoad(0.25 + i * 0.1, `Loon planeeti ${p.name}…`); await nextFrame();
    const tx = await planetTextures(p, i * 31 + 7);
    const mat = new THREE.MeshStandardMaterial({ map: tx.map, roughness: 0.92, metalness: 0 });
    if (tx.emis) { mat.emissiveMap = tx.emis; mat.emissive = new THREE.Color(1, 1, 1); mat.emissiveIntensity = 1.6; }
    const mesh = new THREE.Mesh(new THREE.SphereGeometry(p.r, 64, 40), mat);
    mesh.position.copy(p.pos); mesh.rotation.set(rnd(-0.3, 0.3), rnd(0, 6), rnd(-0.3, 0.3)); scene.add(mesh); p.mesh = mesh;
    if (tx.clouds) { const cm = new THREE.Mesh(new THREE.SphereGeometry(p.r * 1.015, 64, 40), new THREE.MeshStandardMaterial({ map: tx.clouds, transparent: true, depthWrite: false, roughness: 1 })); mesh.add(cm); p.cloud = cm; }
    const atm = new THREE.Mesh(new THREE.SphereGeometry(p.r * 1.07, 64, 40), atmMat(p.atm, p.kind === 'toxic' ? 1.8 : 2.6, p.kind === 'toxic' ? 1.7 : 1.25));
    atm.position.copy(p.pos); scene.add(atm);
    if (p.rings) {
      const rg = new THREE.RingGeometry(p.r * 1.35, p.r * 2.35, 160, 1); const pa = rg.attributes.position, ua = rg.attributes.uv;
      for (let k = 0; k < pa.count; k++) { const d = Math.hypot(pa.getX(k), pa.getY(k)); ua.setXY(k, (d - p.r * 1.35) / (p.r * 1.0), 0.5); }
      const c = cnv(1024, 4), g = c.getContext('2d'), rr2 = mulberry(5);
      for (let x = 0; x < 1024; x++) { const a = (0.25 + rr2() * 0.55) * (Math.sin(x * 0.05) * 0.3 + 0.7) * (x < 40 || x > 1000 ? 0.2 : 1) * (x > 600 && x < 640 ? 0.1 : 1); const b = 180 + rr2() * 60; g.fillStyle = `rgba(${b},${b * 0.9 | 0},${b * 0.75 | 0},${a})`; g.fillRect(x, 0, 1, 4); }
      const ring = new THREE.Mesh(rg, new THREE.MeshStandardMaterial({ map: ctex(c), transparent: true, side: THREE.DoubleSide, depthWrite: false, roughness: 1 }));
      ring.position.copy(p.pos); ring.rotation.set(-PI / 2 + 0.62, 0.25, 0); scene.add(ring);
    }
    PL.push(p);
  }
}

/* =========================================================
   MÄNGIJA LAEV (väljast + seest)
   ========================================================= */
const ship = new THREE.Group(); scene.add(ship);
ship.position.set(0, -190, 900);
const rig = new THREE.Group(); ship.add(rig); rig.add(camera);
const shipFX = { glows: [], stick: null, lever: null, reticle: null, tank: null };
const PM = {}; // mängija laeva materjalid
function buildPlayerShip() {
  Object.assign(PM, shipMats('#d6d3cc', '#c25a1c', '#3a3c40', 909));
  PM.window = new THREE.MeshStandardMaterial({ color: '#9fd8ff', transparent: true, opacity: 0.1, roughness: 0.05, metalness: 0.3, depthWrite: false });
  // ---- välimine kere ----
  const b = new Builder(), r = mulberry(5);
  b.add('hull', tBox(8.4, 2, 18.6), 0, -1.05, 1.9);
  b.add('hull', tBox(7.4, 1, 16.6), 0, 3.6, 2);
  b.mirror('hull', () => tBox(1.1, 1.1, 16.6), 3.65, 0.5, 2);
  b.mirror('hull', () => tBox(1.1, 0.9, 16.6), 3.65, 2.7, 2);
  for (const z of [-5.9, -4, -2, 0, 2, 4, 6, 8, 9.9]) b.mirror('dark', () => tBox(0.5, 1.3, 0.3), 3.4, 1.65, z);
  b.add('hull', tBox(8.4, 4.2, 0.8), 0, 1.05, 10.6);
  b.add('hull', taper(tBox(7.4, 1.4, 5.5), 0.6, 0.5), 0, 0.15, -8.8);
  b.add('dark', tBox(7.4, 0.35, 0.5), 0, 3.15, -6.2);
  b.mirror('dark', () => tBox(0.25, 2.3, 0.4), 1.55, 1.95, -6.2);
  b.mirror('dark', () => tBox(0.5, 2.3, 0.5), 3.25, 1.95, -6.2);
  b.mirror('accent', () => tBox(4, 2.6, 14), 6.3, -0.6, 3);
  b.mirror('dark', () => tBox(2, 1.2, 9), 4.6, -0.5, 2);
  b.mirror('hull', () => tBox(7, 0.5, 9), 11, -0.8, 4, 0, 0, -0.08);
  b.mirror('accent', () => tBox(1.5, 1.2, 5), 14.3, -1.1, 5.5);
  for (const x of [-2.2, 0, 2.2]) { b.add('accent', cylZ(0.9, 7, 14), x, 4.9, 4); for (const z of [1, 7]) b.add('dark', cylZ(1.02, 0.35, 14), x, 4.9, z); }
  for (let i = 0; i < 18; i++) { const w = 0.4 + r() * 1.2, h = 0.2 + r() * 0.6, d = 0.4 + r() * 1.6; b.add(r() < 0.6 ? 'dark' : 'hull', tBox(w, h, d), (r() - 0.5) * 6.5, 4.1 + h / 2, -5 + r() * 15); }
  b.add('dark', tBox(0.2, 2.2, 0.2), 2.8, 5.2, 8.5);
  b.mirror('dark', () => cylZ(1.5, 4, 18, 1.8), 2.4, 1.0, 12.5);
  b.mirror('dark', () => cylZ(1.0, 3, 14), 6.3, -0.6, 11.4);
  b.mirror('glow', () => disc(1.3), 2.4, 1.0, 14.55);
  b.mirror('glow', () => disc(0.85), 6.3, -0.6, 12.95);
  b.mirror('dark', () => cylZ(0.25, 4, 8), 3.2, 0.0, -10.5);
  b.mirror('dark', () => tBox(0.9, 0.9, 2), 3.2, 0.0, -8.6);
  b.mirror('red', () => tBox(0.4, 0.4, 0.4), 14.6, -0.5, 3);
  b.mirror('window', () => tBox(0.03, 1.2, 16), 3.12, 1.65, 2);
  b.add('window', tBox(6.2, 2.1, 0.03), 0, 1.95, -6.12);
  ship.add(b.build(PM));
  for (const [x, y, z, s] of [[2.4, 1, 15, 1.3], [-2.4, 1, 15, 1.3], [6.3, -0.6, 13.4, 0.85], [-6.3, -0.6, 13.4, 0.85]]) { const g = glowSprite(new THREE.Color(0.45, 0.85, 1.0), s * 4); g.position.set(x, y, z); g.userData.base = s * 4; ship.add(g); shipFX.glows.push(g); }

  // ---- sisemus ----
  const IM = {
    wall: new THREE.MeshStandardMaterial({ map: hullTexture('#8e969f', '#3f7391', '#565c64', 31, false), roughness: 0.6, metalness: 0.25 }),
    ceil: new THREE.MeshStandardMaterial({ map: hullTexture('#6a7178', '#3f7391', '#4a5056', 33, false), roughness: 0.7, metalness: 0.2 }),
    floor: new THREE.MeshStandardMaterial({ map: floorTex(), roughness: 0.65, metalness: 0.55 }),
    trim: new THREE.MeshStandardMaterial({ color: '#2c3036', roughness: 0.5, metalness: 0.6 }),
    accent: new THREE.MeshStandardMaterial({ map: hullTexture('#b65a24', '#7a3a18', '#3a2a22', 35, false), roughness: 0.7, metalness: 0.1 }),
    seat: new THREE.MeshStandardMaterial({ color: '#2a3a48', roughness: 0.8, metalness: 0.1 }),
    strip: new THREE.MeshBasicMaterial({ color: new THREE.Color(0.35, 1.0, 1.3), toneMapped: false }),
    stripO: new THREE.MeshBasicMaterial({ color: new THREE.Color(1.3, 0.55, 0.15), toneMapped: false }),
  };
  const i = new Builder();
  i.add('floor', tBox(6, 0.1, 16.2, 2), 0, -0.05, 2);
  i.add('ceil', tBox(6, 0.1, 16.2, 2.5), 0, 3.05, 2);
  for (const s of [-1, 1]) {
    i.add('wall', tBox(0.1, 1.05, 16.2, 2.5), s * 3, 0.525, 2);
    i.add('wall', tBox(0.1, 0.8, 16.2, 2.5), s * 3, 2.65, 2);
    for (const z of [-6, -4, -2, 0, 2, 4, 6, 8, 10]) i.add('trim', tBox(0.16, 1.25, 0.2), s * 2.98, 1.65, z);
    i.add('trim', tBox(0.25, 0.08, 16.2), s * 2.9, 1.06, 2);
    i.add('strip', tBox(0.05, 0.05, 16), s * 2.9, 2.96, 2);
    i.add('stripO', tBox(0.04, 0.04, 16), s * 2.93, 0.06, 2);
  }
  i.add('wall', tBox(6, 0.9, 0.1), 0, 0.45, -6.05);
  i.add('wall', tBox(6, 3.1, 0.1), 0, 1.5, 10.05);
  for (const zp of [-2, 4]) {
    for (const s of [-1, 1]) { i.add('wall', tBox(2.1, 3.0, 0.15, 2.5), s * 1.95, 1.5, zp); i.add('strip', tBox(0.06, 2.4, 0.18), s * 0.92, 1.2, zp); }
    i.add('wall', tBox(1.8, 0.6, 0.15), 0, 2.7, zp); i.add('strip', tBox(1.9, 0.06, 0.18), 0, 2.4, zp);
  }
  // kokpit
  i.add('trim', tBox(5.6, 0.85, 1.0), 0, 0.425, -5.5);
  i.add('stripO', tBox(5.4, 0.03, 0.03), 0, 0.86, -4.99);
  i.add('trim', tBox(0.5, 0.4, 0.5), 0, 0.2, -3.15);
  i.add('seat', tBox(0.72, 0.15, 0.7), 0, 0.48, -3.15);
  i.add('seat', tBox(0.72, 1.0, 0.15), 0, 1.02, -2.76, 0.12);
  i.add('seat', tBox(0.45, 0.3, 0.12), 0, 1.66, -2.7, 0.12);
  i.mirror('trim', () => tBox(0.12, 0.12, 0.6), 0.42, 0.72, -3.15);
  i.mirror('trim', () => tBox(0.1, 0.3, 0.1), 0.42, 0.6, -3.15);
  // pult kangidele
  i.mirror('trim', () => tBox(0.25, 0.7, 0.35), 0.62, 0.35, -3.7);
  // holokaardi laud
  i.add('trim', cylY(0.55, 0.9, 28), 0, 0.45, 1); i.add('trim', cylY(0.6, 0.05, 28), 0, 0.92, 1); i.add('strip', new THREE.TorusGeometry(0.6, 0.015, 6, 40), 0, 0.945, 1, PI / 2, 0, 0);
  // pink ja taim
  i.add('accent', tBox(0.6, 0.45, 2.4), -2.6, 0.225, 2.2); i.add('seat', tBox(0.15, 0.6, 2.4), -2.88, 0.75, 2.2);
  i.add('trim', cylY(0.22, 0.4, 12), 2.6, 0.2, 2.8); i.add('trim', cylY(0.24, 0.04, 12), 2.6, 0.41, 2.8);
  // lastiruum: kastid
  const cr = mulberry(9);
  for (const [x, z, s] of [[-2.35, 5.4, 1], [-2.35, 6.3, 0.8], [2.35, 5.5, 0.9], [-2.35, 5.4, 0.7]]) { i.add('accent', tBox(1.0 * s, 0.9 * s, 0.9 * s, 1.2), x, 0.45 * s + (s === 0.7 ? 0.9 : 0), z); }
  i.add('trim', tBox(1.2, 0.7, 1.0, 1.2), 2.35, 0.35 + 0.9 * 0, 6.2 + cr() * 0);
  // rafineerija (vasakul) ja uuendusterminal (paremal)
  i.add('trim', tBox(1.3, 1.9, 1.5), -2.25, 0.95, 8);
  i.add('stripO', cylY(0.16, 1.6, 12), -2.25, 0.95, 7.2);
  i.add('ceil', tBox(1.3, 0.2, 1.5), -2.25, 2.0, 8);
  i.add('trim', tBox(1.3, 1.9, 1.5), 2.25, 0.95, 8);
  i.add('strip', tBox(0.04, 1.6, 1.3), 1.58, 0.95, 8);
  // kristallimahuti
  i.add('trim', cylY(0.45, 0.25, 20), 0, 0.125, 9.45); i.add('trim', cylY(0.45, 0.2, 20), 0, 2.0, 9.45);
  i.add('strip', cylY(0.47, 0.03, 20), 0, 0.26, 9.45);
  ship.add(i.build(IM));
  // mahuti klaas
  const tankGlass = new THREE.Mesh(cylY(0.42, 1.65, 24), new THREE.MeshStandardMaterial({ color: '#9fe8ff', transparent: true, opacity: 0.15, roughness: 0.05, depthWrite: false }));
  tankGlass.position.set(0, 1.07, 9.45); ship.add(tankGlass);
  const tankIM = new THREE.InstancedMesh(new THREE.OctahedronGeometry(0.07, 0).scale(1, 1.6, 1), new THREE.MeshBasicMaterial({ color: new THREE.Color(0.3, 1.4, 1.8), toneMapped: false }), 40);
  const tr = mulberry(3); for (let k = 0; k < 40; k++) { tankIM.setMatrixAt(k, _m.compose(_v.set((tr() - 0.5) * 0.55, 0.33 + Math.floor(k / 8) * 0.09 + tr() * 0.05, 9.45 + (tr() - 0.5) * 0.55), _q.setFromEuler(_e.set(tr() * 3, tr() * 3, tr() * 3)), _v2.set(1, 1, 1))); }
  tankIM.count = 0; ship.add(tankIM); shipFX.tank = tankIM;
  // taim
  const leafMat = new THREE.MeshStandardMaterial({ color: '#3fa04a', roughness: 0.8 });
  for (let k = 0; k < 7; k++) { const lf = new THREE.Mesh(new THREE.ConeGeometry(0.06, 0.55, 6), leafMat); lf.position.set(2.6 + Math.cos(k) * 0.08, 0.68, 2.8 + Math.sin(k) * 0.08); lf.rotation.set(Math.sin(k * 2) * 0.5, 0, Math.cos(k * 2) * 0.5); ship.add(lf); }
  // juhtkang ja hoob (liiguvad)
  const knob = new THREE.MeshStandardMaterial({ color: '#c25a1c', roughness: 0.5 });
  const mkStick = (x) => { const g = new THREE.Group(); g.position.set(x, 0.72, -3.7); const s = new THREE.Mesh(cylY(0.025, 0.28, 8), IM.trim); s.position.y = 0.14; const k = new THREE.Mesh(new THREE.SphereGeometry(0.05, 12, 8), knob); k.position.y = 0.3; g.add(s, k); ship.add(g); return g; };
  shipFX.stick = mkStick(0.62); shipFX.lever = mkStick(-0.62);
  // valgused
  const l1 = new THREE.PointLight('#bfe8ff', 9, 9, 1.6); l1.position.set(0, 2.6, -3.2); ship.add(l1);
  const l2 = new THREE.PointLight('#ffe0c0', 9, 10, 1.6); l2.position.set(0, 2.6, 5.5); ship.add(l2);
  // sihik
  const ret = new THREE.Mesh(new THREE.PlaneGeometry(14, 14), new THREE.MeshBasicMaterial({ map: RING, color: new THREE.Color(0.4, 1.6, 1.2), transparent: true, blending: THREE.AdditiveBlending, depthWrite: false, toneMapped: false }));
  ret.position.set(0, 0.6, -300); ship.add(ret); shipFX.reticle = ret;
  // sildid
  ship.add(sign('KOKPIT', 0, 2.72, -1.91, 0)); ship.add(sign('LASTIRUUM', 0, 2.72, 3.91, PI));
  ship.add(sign('ELURUUM', 0, 2.72, 4.09, 0)); ship.add(sign('ELURUUM', 0, 2.72, -2.09, PI));
}
function sign(text, x, y, z, ry) {
  const c = cnv(512, 96), g = c.getContext('2d'); g.fillStyle = 'rgba(0,0,0,0)'; g.clearRect(0, 0, 512, 96);
  g.font = 'bold 60px system-ui,sans-serif'; g.textAlign = 'center'; g.textBaseline = 'middle'; g.fillStyle = '#9ff0ff'; g.shadowColor = '#2ad4ff'; g.shadowBlur = 12; g.fillText(text, 256, 50);
  const m = new THREE.Mesh(new THREE.PlaneGeometry(1.2, 0.225), new THREE.MeshBasicMaterial({ map: ctex(c), transparent: true, toneMapped: false, depthWrite: false }));
  m.position.set(x, y, z); m.rotation.y = ry; return m;
}

/* =========================================================
   SIHTKOHAD
   ========================================================= */
const DEST = [];
const TYPE_NAME = { station: 'jaam', field: 'kaevandus', planet: 'planeet' };
function buildDest() {
  DEST.push({ name: 'Jaam Tähesadam', type: 'station', pos: STATION_POS.clone(), stop: 560, color: '#7fd8ff' });
  DEST.push({ name: 'Asteroidiväli', type: 'field', pos: FIELD_POS.clone(), stop: 1150, color: '#c8b090' });
  for (const p of PL) DEST.push({ name: p.name, type: 'planet', pos: p.pos.clone(), stop: p.r * 1.7 + 250, planet: p, color: p.atm });
}
const fmtDist = d => d >= 10000 ? Math.round(d / 1000) + ' km' : d >= 1000 ? (d / 1000).toFixed(1) + ' km' : Math.round(d) + ' m';

/* =========================================================
   TEISED LAEVAD
   ========================================================= */
const NPCS = []; let HAULER, FIGHTER;
function addNPC(T, mats, mode, o) {
  const g = meshFrom(T.geos, mats); g.scale.setScalar(o.scale || 1); scene.add(g);
  const glows = T.engines.map(e => { const s = glowSprite(new THREE.Color(0.45, 0.85, 1.0), e[3] * 5); s.position.set(e[0], e[1], e[2]); g.add(s); return s; });
  const navs = T.navs.map(([x, y, z, c]) => { const s = glowSprite(c === 'r' ? '#ff3a2a' : '#3aff6a', 3); s.position.set(x, y, z); g.add(s); return s; });
  const n = Object.assign({ g, glows, navs, mode, phase: rnd(0, 10) }, o);
  if (mode === 'travel') { g.position.copy(randUnit(_v).multiplyScalar(rnd(700, 1600))).add(STATION_POS); pickNPCTarget(n); }
  NPCS.push(n); return n;
}
function pickNPCTarget(n) { const d = DEST[(rnd() * DEST.length) | 0]; n.tp = randUnit(new V3()).multiplyScalar(d.stop + rnd(200, 700)).add(d.pos); }
function updateNPCs(dt) {
  for (const n of NPCS) {
    if (n.mode === 'orbit') {
      n.a += n.speed / n.R * dt;
      const p = _v.set(Math.cos(n.a) * n.R, n.h + Math.sin(n.a * 0.5) * n.bob, Math.sin(n.a) * n.R).add(STATION_POS);
      const p2 = _v2.set(Math.cos(n.a + 0.03) * n.R, n.h + Math.sin((n.a + 0.03) * 0.5) * n.bob, Math.sin(n.a + 0.03) * n.R).add(STATION_POS);
      n.g.position.copy(p); _m.lookAt(p, p2, UP); n.g.quaternion.setFromRotationMatrix(_m); n.g.rotateZ(n.bank || 0);
    } else {
      const p = n.g.position; const d = p.distanceTo(n.tp);
      _m.lookAt(p, n.tp, UP); _q.setFromRotationMatrix(_m); n.g.quaternion.rotateTowards(_q, 0.35 * dt);
      p.addScaledVector(fwdOf(n.g, _v), n.speed * dt);
      if (d < 200) pickNPCTarget(n);
    }
    for (let k = 0; k < n.glows.length; k++) n.glows[k].material.opacity = 0.75 + 0.25 * Math.sin(R.time * 31 + k * 2 + n.phase);
    const on = ((R.time + n.phase) % 1.5) < 0.18; for (const s of n.navs) s.visible = on;
  }
}

/* =========================================================
   LASERID, KRISTALLID, EFEKTID
   ========================================================= */
const BOLTS = [], CRYS = [], DEBRIS = [], FLASH = [];
function buildPools() {
  const bg = cylZ(0.2, 16, 6);
  const bm = new THREE.MeshBasicMaterial({ color: new THREE.Color(0.5, 1.8, 2.2), toneMapped: false, transparent: true, blending: THREE.AdditiveBlending, depthWrite: false });
  for (let k = 0; k < 40; k++) { const m = new THREE.Mesh(bg, bm); m.visible = false; scene.add(m); BOLTS.push({ m, vel: new V3(), life: 0, prev: new V3() }); }
  const cg = new THREE.OctahedronGeometry(1.4, 0).scale(1, 1.7, 1);
  const cm = [new THREE.MeshStandardMaterial({ color: '#50e8ff', emissive: '#18a0ff', emissiveIntensity: 1.6, roughness: 0.2, metalness: 0.3 }),
              new THREE.MeshStandardMaterial({ color: '#ffd24a', emissive: '#ff9020', emissiveIntensity: 1.6, roughness: 0.2, metalness: 0.3 })];
  for (let k = 0; k < 70; k++) { const m = new THREE.Mesh(cg, cm[0]); const gl = glowSprite('#40d8ff', 7, 0.8); m.add(gl); m.visible = false; scene.add(m); CRYS.push({ m, gl, vel: new V3(), life: 0, value: 1, mats: cm }); }
  for (let k = 0; k < 50; k++) { const m = new THREE.Mesh(debrisGeo, debrisMat); m.visible = false; scene.add(m); DEBRIS.push({ m, vel: new V3(), life: 0, s: 1, spin: new V3() }); }
  for (let k = 0; k < 14; k++) { const s = glowSprite('#ffc080', 10); s.visible = false; scene.add(s); FLASH.push({ s, life: 0, max: 1, size: 1 }); }
}
function flash(pos, size, color, life = 0.5) { const f = FLASH.find(f => f.life <= 0) || FLASH[0]; f.s.position.copy(pos); f.s.material.color.set(color); f.size = size; f.life = f.max = life; f.s.visible = true; }
function fire() {
  R.gun ^= 1; const b = BOLTS.find(b => b.life <= 0); if (!b) return;
  b.m.position.set(R.gun ? 3.2 : -3.2, 0, -13); ship.localToWorld(b.m.position);
  b.m.quaternion.copy(ship.quaternion); fwdOf(ship, b.vel).multiplyScalar(1900 + Math.max(0, R.speed)); b.life = 1.1; b.m.visible = true;
  sfx.laser(); haptic('right', 0.25, 25);
}
function updateLasers(dt) {
  const nearField = ship.position.distanceTo(FIELD_POS) < 3500;
  for (const b of BOLTS) {
    if (b.life <= 0) continue; b.life -= dt; if (b.life <= 0) { b.m.visible = false; continue; }
    b.prev.copy(b.m.position); b.m.position.addScaledVector(b.vel, dt);
    if (!nearField) continue;
    const d = _v.copy(b.m.position).sub(b.prev); const dd = d.lengthSq();
    for (const a of AST) {
      if (!a.alive) continue;
      const t = clamp(_v2.copy(a.pos).sub(b.prev).dot(d) / dd, 0, 1); _v3.copy(b.prev).addScaledVector(d, t);
      if (_v3.distanceToSquared(a.pos) < a.r * a.r * 0.8) {
        b.life = 0; b.m.visible = false; a.hp -= stat.dmg; flash(_v3, a.r * 0.6, '#ffd090', 0.25); sfx.hit();
        if (a.hp <= 0) breakRock(a);
        break;
      }
    }
  }
}
function breakRock(a) {
  a.alive = false; a.respawn = 45; sfx.boom(); haptic('right', 0.7, 120);
  flash(a.pos, a.r * 5, '#ffb060', 0.7); flash(a.pos, a.r * 3, '#ffffff', 0.3);
  for (let k = 0; k < 8; k++) { const d = DEBRIS.find(d => d.life <= 0); if (!d) break; d.m.position.copy(a.pos).addScaledVector(randUnit(_v), a.r * 0.4); d.vel.copy(randUnit(_v)).multiplyScalar(rnd(15, 50)); d.s = a.r * rnd(0.12, 0.3); d.life = rnd(1.5, 3); d.spin.set(rnd(-3, 3), rnd(-3, 3), rnd(-3, 3)); d.m.visible = true; }
  const n = 1 + Math.floor(a.r / 8);
  for (let k = 0; k < n; k++) { const c = CRYS.find(c => c.life <= 0); if (!c) break; const gold = rnd() < 0.15; c.value = gold ? 3 : 1; c.m.material = c.mats[gold ? 1 : 0]; c.gl.material.color.set(gold ? '#ffb040' : '#40d8ff'); c.m.position.copy(a.pos); c.vel.copy(randUnit(_v)).multiplyScalar(rnd(8, 25)); c.life = 60; c.m.visible = true; }
}
function updateCrystals(dt) {
  const P = ship.position; const mr = stat.magnet;
  for (const c of CRYS) {
    if (c.life <= 0) continue; c.life -= dt; if (c.life <= 0) { c.m.visible = false; continue; }
    const d = _v.copy(P).sub(c.m.position); const dist = d.length();
    if (dist < mr) { d.normalize(); const want = Math.abs(R.speed) + 160; c.vel.lerp(d.multiplyScalar(want), 1 - Math.exp(-dt * 3)); }
    else c.vel.multiplyScalar(Math.exp(-dt * 0.5));
    c.m.position.addScaledVector(c.vel, dt); c.m.rotation.y += dt * 2; c.m.rotation.x += dt * 0.7;
    if (dist < 16) { c.life = 0; c.m.visible = false; S.crystals += c.value; S.mined += c.value; sfx.pick(); haptic('left', 0.3, 40); }
  }
  for (const d of DEBRIS) { if (d.life <= 0) continue; d.life -= dt; if (d.life <= 0) { d.m.visible = false; continue; } d.m.position.addScaledVector(d.vel, dt); d.m.rotation.x += d.spin.x * dt; d.m.rotation.y += d.spin.y * dt; d.m.scale.setScalar(d.s * Math.min(1, d.life)); }
  for (const f of FLASH) { if (f.life <= 0) continue; f.life -= dt; if (f.life <= 0) { f.s.visible = false; continue; } const t = 1 - f.life / f.max; f.s.scale.setScalar(f.size * (0.5 + t)); f.s.material.opacity = 1 - t; }
}

/* =========================================================
   KOSMOSETOLM (kiiruse tunnetamiseks)
   ========================================================= */
const DN = 450, DB = 220; let dust, dustP = [];
function buildDust() {
  const pos = new Float32Array(DN * 6); for (let k = 0; k < DN; k++) dustP.push(new V3(rnd(-DB, DB), rnd(-DB, DB), rnd(-DB, DB)).add(ship.position));
  const g = new THREE.BufferGeometry(); g.setAttribute('position', new THREE.BufferAttribute(pos, 3));
  dust = new THREE.LineSegments(g, new THREE.LineBasicMaterial({ color: 0xa8d0ff, transparent: true, opacity: 0.5, toneMapped: false, depthWrite: false }));
  dust.frustumCulled = false; scene.add(dust);
}
function updateDust() {
  const P = ship.position, a = dust.geometry.attributes.position.array; const sp = R.vel.length();
  const len = clamp(sp * 0.03, 0.4, 70); const dir = sp > 1 ? _v2.copy(R.vel).normalize() : fwdOf(ship, _v2);
  for (let k = 0; k < DN; k++) {
    const p = dustP[k];
    for (const ax of ['x', 'y', 'z']) { let d = p[ax] - P[ax]; if (d > DB) p[ax] -= 2 * DB; else if (d < -DB) p[ax] += 2 * DB; }
    a[k * 6] = p.x; a[k * 6 + 1] = p.y; a[k * 6 + 2] = p.z;
    a[k * 6 + 3] = p.x - dir.x * len; a[k * 6 + 4] = p.y - dir.y * len; a[k * 6 + 5] = p.z - dir.z * len;
  }
  dust.geometry.attributes.position.needsUpdate = true;
  dust.material.opacity = 0.25 + clamp(sp / 600, 0, 0.45);
}

/* =========================================================
   HELI
   ========================================================= */
let AC = null, master, engG, engF;
function initAudio() {
  if (AC) { if (AC.state === 'suspended') AC.resume(); return; }
  try { AC = new (window.AudioContext || window.webkitAudioContext)(); } catch (e) { return; }
  master = AC.createGain(); master.gain.value = 0.6; master.connect(AC.destination);
  const buf = AC.createBuffer(1, AC.sampleRate * 2, AC.sampleRate), d = buf.getChannelData(0); let last = 0;
  for (let k = 0; k < d.length; k++) { last = (last + 0.02 * (Math.random() * 2 - 1)) / 1.02; d[k] = last * 3.5; }
  const src = AC.createBufferSource(); src.buffer = buf; src.loop = true;
  engF = AC.createBiquadFilter(); engF.type = 'lowpass'; engF.frequency.value = 200; engG = AC.createGain(); engG.gain.value = 0.1;
  src.connect(engF).connect(engG).connect(master); src.start();
  const o = AC.createOscillator(); o.type = 'sawtooth'; o.frequency.value = 46; const of = AC.createBiquadFilter(); of.type = 'lowpass'; of.frequency.value = 110; const hg = AC.createGain(); hg.gain.value = 0.045; o.connect(of).connect(hg).connect(master); o.start();
}
function tone(f1, f2, dur, type = 'sine', vol = 0.2, delay = 0) {
  if (!AC) return; const t = AC.currentTime + delay; const o = AC.createOscillator(), g = AC.createGain(); o.type = type;
  o.frequency.setValueAtTime(f1, t); o.frequency.exponentialRampToValueAtTime(Math.max(f2, 1), t + dur);
  g.gain.setValueAtTime(vol, t); g.gain.exponentialRampToValueAtTime(0.0001, t + dur); o.connect(g).connect(master); o.start(t); o.stop(t + dur + 0.02);
}
function noiseBurst(dur, vol, freq) {
  if (!AC) return; const t = AC.currentTime; const buf = AC.createBuffer(1, AC.sampleRate * dur, AC.sampleRate), d = buf.getChannelData(0);
  for (let k = 0; k < d.length; k++) d[k] = (Math.random() * 2 - 1) * (1 - k / d.length);
  const s = AC.createBufferSource(); s.buffer = buf; const f = AC.createBiquadFilter(); f.type = 'lowpass'; f.frequency.setValueAtTime(freq, t); f.frequency.exponentialRampToValueAtTime(80, t + dur);
  const g = AC.createGain(); g.gain.value = vol; s.connect(f).connect(g).connect(master); s.start();
}
const sfx = {
  laser: () => tone(1500, 240, 0.15, 'square', 0.05), hit: () => tone(320, 90, 0.1, 'triangle', 0.15), boom: () => noiseBurst(0.8, 0.6, 1000),
  pick: () => { tone(880, 880, 0.08, 'sine', 0.12); tone(1320, 1320, 0.12, 'sine', 0.12, 0.07); },
  click: () => tone(1200, 900, 0.05, 'square', 0.05), msg: () => { tone(660, 660, 0.08, 'sine', 0.08); tone(990, 990, 0.1, 'sine', 0.08, 0.09); },
  alarm: () => { tone(520, 520, 0.18, 'square', 0.07); tone(390, 390, 0.18, 'square', 0.07, 0.2); },
  fanfare: () => [523, 659, 784, 1046].forEach((f, i) => tone(f, f, 0.3, 'triangle', 0.12, i * 0.12)),
  deny: () => tone(220, 160, 0.2, 'sawtooth', 0.07),
};
function updateAudio() {
  if (!AC || !engG) return; const f = clamp(Math.abs(R.speed) / stat.pulse, 0, 1);
  engG.gain.setTargetAtTime(0.06 + 0.3 * f + (R.mode === 'outside' ? 0.05 : 0), AC.currentTime, 0.2);
  engF.frequency.setTargetAtTime(140 + Math.abs(R.speed) * 1.1, AC.currentTime, 0.2);
}
let etVoice = null;
function pickVoice() { try { etVoice = speechSynthesis.getVoices().find(v => v.lang && v.lang.toLowerCase().startsWith('et')) || null; } catch (e) { } }
if (window.speechSynthesis) { pickVoice(); speechSynthesis.onvoiceschanged = pickVoice; }
function haptic(hand, v, ms) { const s = renderer.xr.getSession(); if (!s) return; for (const src of s.inputSources) if (src.handedness === hand) src.gamepad?.hapticActuators?.[0]?.pulse?.(v, ms); }

/* =========================================================
   KASUTAJALIIDES: PANEELID
   ========================================================= */
const PANELS = [], UIT = [];
function txt(g, s, x, y, size, col, align = 'left', bold = false) { g.font = `${bold ? 'bold ' : ''}${size}px system-ui,sans-serif`; g.fillStyle = col; g.textAlign = align; g.textBaseline = 'middle'; g.fillText(s, x, y); }
function panelBg(g, W, H, title, col = '#2ad4ff') {
  rr(g, 6, 6, W - 12, H - 12, 26); const gr = g.createLinearGradient(0, 0, 0, H); gr.addColorStop(0, 'rgba(6,26,44,0.94)'); gr.addColorStop(1, 'rgba(3,12,24,0.94)');
  g.fillStyle = gr; g.fill(); g.lineWidth = 5; g.strokeStyle = hexA(col, 0.9); g.stroke();
  g.fillStyle = hexA(col, 0.05); for (let y = 14; y < H - 14; y += 6) g.fillRect(14, y, W - 28, 2);
  if (title) { txt(g, title, 36, 54, 46, col, 'left', true); g.fillStyle = hexA(col, 0.7); g.fillRect(36, 86, W - 72, 3); }
}
function bar(g, label, f, col, y, W) {
  txt(g, label, 40, y, 36, '#8fd8ea'); const x = 250, w = W - 290, h = 36;
  rr(g, x, y - h / 2, w, h, 10); g.fillStyle = 'rgba(255,255,255,0.08)'; g.fill();
  if (f > 0.005) { rr(g, x, y - h / 2, Math.max(20, w * clamp(f, 0, 1)), h, 10); g.fillStyle = col; g.fill(); }
  txt(g, Math.round(clamp(f, 0, 1) * 100) + '%', x + w / 2, y + 1, 28, '#ffffff', 'center', true);
}
class Panel {
  constructor({ w, h, W, H, draw, sig = null, interactive = true, overlay = false }) {
    Object.assign(this, { W, H, draw, sig, last: '', buttons: [], hover: -1, hoverNext: -1 });
    this.c = cnv(W, H); this.g = this.c.getContext('2d'); this.t = ctex(this.c);
    this.mesh = new THREE.Mesh(new THREE.PlaneGeometry(w, h), new THREE.MeshBasicMaterial({ map: this.t, transparent: true, toneMapped: false, depthTest: !overlay, depthWrite: !overlay }));
    if (overlay) this.mesh.renderOrder = 1000;
    this.mesh.userData.panel = this; PANELS.push(this); if (interactive) UIT.push(this.mesh); this.refresh();
  }
  refresh() { this.g.clearRect(0, 0, this.W, this.H); this.buttons = []; this.draw(this.g, this); this.t.needsUpdate = true; }
  check() { if (!this.sig) return; const s = this.sig(); if (s !== this.last) { this.last = s; this.refresh(); } }
  btn(x, y, w, h, label, fn, o = {}) {
    const i = this.buttons.length; this.buttons.push({ x, y, w, h, fn, dis: !!o.dis }); const g = this.g;
    const col = o.col || '#2ad4ff', hov = this.hover === i && !o.dis;
    rr(g, x, y, w, h, Math.min(18, h / 3));
    g.fillStyle = o.dis ? 'rgba(70,80,90,0.45)' : o.on ? hexA(col, 0.55) : hov ? hexA(col, 0.42) : hexA(col, 0.14); g.fill();
    g.lineWidth = hov ? 6 : 3; g.strokeStyle = o.dis ? 'rgba(120,130,140,0.6)' : col; g.stroke();
    const fs = o.fs || 40; txt(g, label, x + w / 2, y + h / 2 + (o.sub ? -fs * 0.32 : 0), fs, o.dis ? '#8a96a0' : '#f4fcff', 'center', true);
    if (o.sub) txt(g, o.sub, x + w / 2, y + h / 2 + fs * 0.5, fs * 0.62, o.dis ? '#76828c' : '#b4e6f2', 'center');
  }
  indexAt(uv) { const x = uv.x * this.W, y = (1 - uv.y) * this.H; return this.buttons.findIndex(b => x >= b.x && x <= b.x + b.w && y >= b.y && y <= b.y + b.h); }
  click(uv) { const i = this.indexAt(uv); if (i < 0) return; const b = this.buttons[i]; if (b.dis) { sfx.deny(); return; } sfx.click(); b.fn(); this.last = ''; for (const p of PANELS) p.last = ''; }
}
function place(p, x, y, z, rx = 0, ry = 0) { p.mesh.position.set(x, y, z); p.mesh.rotation.set(rx, ry, 0, 'YXZ'); ship.add(p.mesh); return p; }

const UPG = [['laser', 'Laser', 'kivid purunevad kiiremini'], ['engine', 'Mootor', 'suurem kiirus'], ['tank', 'Kütusepaak', '+50 kütust'], ['shield', 'Kilp', '+50 kilpi'], ['magnet', 'Magnet', 'tõmbab kristalle kaugemalt']];
const UPG_BASE = { laser: 60, engine: 80, tank: 60, shield: 60, magnet: 50 };
let msgPanel;
function buildPanels() {
  // vasak ekraan: laeva olek
  place(new Panel({ w: 1.05, h: 0.7, W: 1024, H: 680, sig: () => [Math.round(R.speed / 2), Math.round(S.fuel), Math.round(S.shield), S.crystals, S.credits, R.autopilot, R.mode, S.up.tank, S.up.shield].join('|'),
    draw: (g, P) => {
      panelBg(g, P.W, P.H, 'LAEVA OLEK'); let y = 138;
      const row = (k, v, c = '#f2fbff') => { txt(g, k, 40, y, 36, '#8fd8ea'); txt(g, String(v), P.W - 40, y, 42, c, 'right', true); y += 66; };
      row('Kiirus', Math.round(Math.abs(R.speed)) + ' m/s', Math.abs(R.speed) > stat.manual + 10 ? '#d0a8ff' : '#f2fbff');
      bar(g, 'Kütus', S.fuel / stat.maxFuel, S.fuel / stat.maxFuel < 0.15 ? '#ff5040' : '#ff9a30', y, P.W); y += 70;
      bar(g, 'Kilp', S.shield / stat.maxShield, '#2ab8ff', y, P.W); y += 74;
      row('Kristallid', S.crystals, '#7ff0ff'); row('Krediidid', S.credits + ' kr', '#ffd36a');
      row('Režiim', R.autopilot ? 'AUTOPILOOT' : R.mode === 'walk' ? 'hõljub' : 'käsitsi', R.autopilot ? '#7dff9a' : '#f2fbff');
    } }), -1.34, 1.38, -4.86, -0.3, 0.4);
  // keskmine: sihtkohad + autopiloot
  place(new Panel({ w: 1.3, h: 0.89, W: 1024, H: 700, sig: () => [S.target, R.autopilot, S.discovered.length, ...DEST.map(d => fmtDist(ship.position.distanceTo(d.pos)))].join('|'),
    draw: (g, P) => {
      panelBg(g, P.W, P.H, 'SIHTKOHAD');
      DEST.forEach((d, i) => { const x = 36 + (i % 2) * 484, y = 108 + ((i / 2) | 0) * 112; const disc = d.type === 'planet' && S.discovered.includes(d.name);
        P.btn(x, y, 468, 98, d.name + (disc ? ' ✓' : ''), () => setTarget(i), { on: S.target === i, fs: 36, sub: fmtDist(ship.position.distanceTo(d.pos)) + ' · ' + TYPE_NAME[d.type] }); });
      P.btn(36, 566, 952, 106, R.autopilot ? 'AUTOPILOOT SEES' : 'AUTOPILOOT SISSE', toggleAutopilot, { on: R.autopilot, col: '#7dff9a', fs: 44, dis: S.target < 0 && !R.autopilot, sub: R.autopilot ? 'vajuta, et välja lülitada' : S.target < 0 ? 'vali enne sihtkoht' : 'lendab ise – võid ringi kõndida' });
    } }), 0, 1.45, -4.98, -0.3, 0);
  // parem: juhtimine + jaam
  place(new Panel({ w: 1.05, h: 0.7, W: 1024, H: 680, sig: () => [R.nearStation, S.crystals, S.credits, Math.round(S.fuel), Math.round(S.shield), Math.round(R.throttle * 20), R.mode].join('|'),
    draw: (g, P) => {
      panelBg(g, P.W, P.H, 'JUHTIMINE');
      P.btn(36, 108, 464, 112, 'TÕUSE PÜSTI', () => toggleSeat(), { sub: 'kõnni laevas (B)' });
      P.btn(524, 108, 464, 112, 'VÄLISVAADE', () => toggleOutside(), { sub: 'vaata laeva (X)' });
      P.btn(36, 236, 464, 96, 'PEATU', () => { R.throttle = 0; R.autopilot = false; say('Mootorid seisma.'); }, { col: '#ff8a5a' });
      P.btn(524, 236, 464, 96, 'JÄRGMINE SIHT', () => cycleTarget(), { sub: '(Y)', fs: 36 });
      if (R.nearStation) {
        txt(g, 'Tähesadama jaam on lähedal', 512, 370, 34, '#ffd36a', 'center', true);
        P.btn(36, 400, 300, 128, 'MÜÜ', sell, { sub: `${S.crystals} × 10 kr`, dis: S.crystals <= 0, col: '#ffd36a' });
        P.btn(362, 400, 300, 128, 'TANGI', buyFuel, { sub: '30 kr', dis: S.credits < 30 || S.fuel >= stat.maxFuel - 0.5, col: '#ffa040' });
        P.btn(688, 400, 300, 128, 'PARANDA', repair, { sub: 'kilp tasuta', dis: S.shield >= stat.maxShield - 0.5 });
      } else { txt(g, 'Lenda jaama juurde, et müüa', 512, 420, 34, '#7fa8b8', 'center'); txt(g, 'kristalle ja tankida.', 512, 466, 34, '#7fa8b8', 'center'); }
      txt(g, 'Hoob', 40, 600, 34, '#8fd8ea'); rr(g, 170, 582, 818, 36, 10); g.fillStyle = 'rgba(255,255,255,0.08)'; g.fill();
      const z = 170 + 818 * 0.23; g.fillStyle = '#ffffff55'; g.fillRect(z - 1, 576, 3, 48);
      const tw = 818 * 0.77 * R.throttle; g.fillStyle = R.throttle >= 0 ? '#7dff9a' : '#ff8a5a'; g.fillRect(Math.min(z, z + tw), 586, Math.abs(tw), 28);
    } }), 1.34, 1.38, -4.86, -0.3, -0.4);
  // istme taga: istu
  place(new Panel({ w: 0.62, h: 0.24, W: 640, H: 248, sig: () => R.mode, draw: (g, P) => { P.btn(10, 10, 620, 228, R.mode === 'seat' ? 'ISTUD ROOLIS' : 'ISTU ROOLI', () => { if (R.mode !== 'seat') toggleSeat(); }, { fs: 64, on: R.mode === 'seat', sub: 'või vajuta B' }); } }), 0, 1.35, -2.66, 0, 0);
  // abi (elutoas)
  place(new Panel({ w: 1.6, h: 1.25, W: 1024, H: 800, interactive: false, draw: (g, P) => {
    panelBg(g, P.W, P.H, 'KUIDAS MÄNGIDA', '#7dff9a'); const L = [
      ['1.', 'Vali kokpitis sihtkoht ja lülita autopiloot sisse.'], ['2.', 'Autopiloodi ajal võid püsti tõusta ja kõndida.'],
      ['3.', 'Asteroidiväljal tulista kive – need annavad kristalle.'], ['4.', 'Rafineerija teeb kristallidest kütust.'],
      ['5.', 'Jaamas müü kristalle ja osta uuendusi.'], ['6.', 'Lenda planeetide juurde ja avasta need kõik!'],
      ['', ''], ['Pulgad:', 'vasak – kõnd / hoob, parem – pööra'], ['Nupud:', 'A autopiloot, B istu, X välisvaade, Y siht'], ['Päästik:', 'parem – laser ja nupud, vasak – pulss']];
    let y = 140; for (const [a, b] of L) { txt(g, a, 44, y, 36, '#7dff9a', 'left', true); txt(g, b, 190, y, 36, '#e8f8ff'); y += 64; }
  } }), -1.95, 1.55, 3.9, 0, PI);
  // kapteni logi
  place(new Panel({ w: 1.6, h: 1.25, W: 1024, H: 800, sig: () => [S.discovered.length, S.mined, S.credits, R.resetArm > 0].join('|'), draw: (g, P) => {
    panelBg(g, P.W, P.H, 'KAPTENI LOGI', '#c9a2ff'); let y = 140;
    txt(g, `Avastatud planeete: ${S.discovered.length} / ${PL.length}`, 44, y, 40, '#ffffff', 'left', true); y += 66;
    PL.forEach((p, k) => { const d = S.discovered.includes(p.name); txt(g, (d ? '✓ ' : '? ') + (d ? p.name : 'tundmatu'), 70 + (k % 2) * 460, y + ((k / 2) | 0) * 58, 38, d ? '#7dff9a' : '#7a8a96'); }); y += 3 * 58 + 20;
    txt(g, `Kaevandatud kristalle: ${S.mined}`, 44, y, 36, '#7ff0ff'); y += 56; txt(g, `Krediite: ${S.credits} kr`, 44, y, 36, '#ffd36a');
    if (S.discovered.length >= PL.length) txt(g, 'Kõik planeedid avastatud – suurepärane, kapten!', 512, y + 70, 34, '#7dff9a', 'center', true);
    P.btn(44, P.H - 132, P.W - 88, 96, R.resetArm > 0 ? 'KINDEL? Vajuta uuesti' : 'ALUSTA UUESTI', resetGame, { col: '#ff6a6a', fs: 38 });
  } }), 1.95, 1.55, 3.9, 0, PI);
  // rafineerija
  place(new Panel({ w: 0.9, h: 0.76, W: 900, H: 760, sig: () => [S.crystals, Math.round(S.fuel), S.up.tank].join('|'), draw: (g, P) => {
    panelBg(g, P.W, P.H, 'RAFINEERIJA', '#ffa040'); txt(g, 'Teeb kristallidest kütust', 40, 140, 34, '#ffd9b0');
    txt(g, `Kristalle: ${S.crystals}`, 40, 210, 46, '#7ff0ff', 'left', true); bar(g, 'Kütus', S.fuel / stat.maxFuel, '#ff9a30', 290, P.W);
    const full = S.fuel >= stat.maxFuel - 0.5;
    P.btn(40, 360, P.W - 80, 150, 'RAFINEERI', () => refine(1), { sub: '3 kristalli → +20 kütust', dis: S.crystals < 3 || full, col: '#ffa040', fs: 50 });
    P.btn(40, 540, P.W - 80, 150, 'RAFINEERI KÕIK', () => refine(999), { sub: 'kuni paak on täis', dis: S.crystals < 3 || full, col: '#ffa040', fs: 46 });
  } }), -1.585, 1.3, 8, 0, PI / 2);
  // uuendused
  place(new Panel({ w: 0.9, h: 0.96, W: 900, H: 960, sig: () => [S.credits, ...Object.values(S.up)].join('|'), draw: (g, P) => {
    panelBg(g, P.W, P.H, 'UUENDUSED', '#7dff9a'); txt(g, `${S.credits} kr`, P.W - 40, 54, 40, '#ffd36a', 'right', true);
    UPG.forEach(([k, n, d], i) => { const y = 112 + i * 166, lv = S.up[k], price = UPG_BASE[k] * (lv + 1);
      txt(g, n, 40, y + 42, 44, '#ffffff', 'left', true); txt(g, d, 40, y + 92, 28, '#9fc8d6');
      for (let j = 0; j < 3; j++) { rr(g, 40 + j * 46, y + 118, 36, 16, 6); g.fillStyle = j < lv ? '#7dff9a' : 'rgba(255,255,255,0.15)'; g.fill(); }
      P.btn(P.W - 320, y + 22, 280, 116, lv >= 3 ? 'MAX' : `${price} kr`, () => buyUp(k), { dis: lv >= 3 || S.credits < price, col: '#7dff9a', fs: 44 }); });
  } }), 1.585, 1.3, 8, 0, -PI / 2);
  // teated (alati nähtaval, pea ees)
  msgPanel = new Panel({ w: 1.5, h: 0.21, W: 1024, H: 144, interactive: false, overlay: true, draw: (g, P) => {
    if (!R.msg) return; rr(g, 8, 8, P.W - 16, P.H - 16, 30); g.fillStyle = 'rgba(4,18,32,0.88)'; g.fill(); g.lineWidth = 4; g.strokeStyle = '#2ad4ff'; g.stroke();
    let fs = 46; g.font = `bold ${fs}px system-ui,sans-serif`; while (g.measureText(R.msg).width > P.W - 70 && fs > 24) { fs -= 2; g.font = `bold ${fs}px system-ui,sans-serif`; }
    txt(g, R.msg, P.W / 2, P.H / 2 + 2, fs, '#ecfcff', 'center', true);
  } });
  msgPanel.mesh.position.set(0, -0.34, -1.6); msgPanel.mesh.visible = false; camera.add(msgPanel.mesh);
}

/* ---------- holokaart ---------- */
const HK = 0.5 / 16000; let holoInner, holoShip, holoLine, holoSel;
function buildHolo() {
  const holo = new THREE.Group(); holo.position.set(0, 1.5, 1); ship.add(holo);
  holoInner = new THREE.Group(); holo.add(holoInner);
  const beam = new THREE.Mesh(new THREE.CylinderGeometry(0.62, 0.55, 0.6, 32, 1, true), new THREE.MeshBasicMaterial({ color: new THREE.Color(0.2, 0.7, 1), transparent: true, opacity: 0.045, blending: THREE.AdditiveBlending, depthWrite: false, side: THREE.DoubleSide, toneMapped: false }));
  beam.position.set(0, 1.25, 1); ship.add(beam);
  DEST.forEach((d, i) => {
    const p = d.pos.clone().multiplyScalar(HK);
    let vis;
    if (d.type === 'station') vis = new THREE.Mesh(new THREE.TorusGeometry(0.022, 0.006, 6, 20), new THREE.MeshBasicMaterial({ color: new THREE.Color(0.5, 1.4, 1.8), toneMapped: false }));
    else if (d.type === 'field') { const g = new THREE.BufferGeometry().setFromPoints([...Array(40)].map(() => randUnit(new V3()).multiplyScalar(rnd(0, 0.035)))); vis = new THREE.Points(g, new THREE.PointsMaterial({ color: '#d8c0a0', size: 0.006, toneMapped: false })); }
    else { const p0 = d.planet; vis = new THREE.Mesh(new THREE.SphereGeometry(0.012 + p0.r / 2000 * 0.03, 16, 12), new THREE.MeshBasicMaterial({ map: p0.mesh.material.map, toneMapped: false })); }
    vis.position.copy(p); holoInner.add(vis);
    const hit = new THREE.Mesh(new THREE.SphereGeometry(0.06, 8, 6), new THREE.MeshBasicMaterial({ transparent: true, opacity: 0, depthWrite: false }));
    hit.position.copy(p); hit.userData.dest = i; holoInner.add(hit); UIT.push(hit);
    const c = cnv(512, 96), g = c.getContext('2d'); txt(g, d.name, 256, 50, 50, '#bff6ff', 'center', true);
    const lab = new THREE.Sprite(new THREE.SpriteMaterial({ map: ctex(c), transparent: true, depthWrite: false, toneMapped: false })); lab.scale.set(0.2, 0.0375, 1); lab.position.copy(p).add(new V3(0, 0.07, 0)); holoInner.add(lab);
  });
  const sun = glowSprite('#ffe0a0', 0.12); sun.position.copy(SUN_DIR).multiplyScalar(0.62); holoInner.add(sun);
  holoShip = new THREE.Mesh(new THREE.ConeGeometry(0.014, 0.045, 8).rotateX(-PI / 2), new THREE.MeshBasicMaterial({ color: new THREE.Color(0.4, 2, 0.6), toneMapped: false })); holoInner.add(holoShip);
  holoLine = new THREE.Line(new THREE.BufferGeometry().setFromPoints([new V3(), new V3()]), new THREE.LineBasicMaterial({ color: 0x7dff9a, transparent: true, opacity: 0.8, toneMapped: false })); holoInner.add(holoLine);
  holoSel = new THREE.Sprite(new THREE.SpriteMaterial({ map: RING, color: new THREE.Color(0.5, 2, 0.8), transparent: true, depthWrite: false, toneMapped: false })); holoSel.scale.setScalar(0.09); holoSel.visible = false; holoInner.add(holoSel);
}
function updateHolo() {
  holoInner.quaternion.copy(ship.quaternion).invert();
  holoShip.position.copy(ship.position).multiplyScalar(HK); holoShip.quaternion.copy(ship.quaternion);
  const pa = holoLine.geometry.attributes.position;
  if (S.target >= 0) { const tp = _v.copy(DEST[S.target].pos).multiplyScalar(HK); pa.setXYZ(0, holoShip.position.x, holoShip.position.y, holoShip.position.z); pa.setXYZ(1, tp.x, tp.y, tp.z); pa.needsUpdate = true; holoLine.visible = true; holoSel.visible = true; holoSel.position.copy(tp); holoSel.material.rotation = R.time; }
  else { holoLine.visible = false; holoSel.visible = false; }
}

/* ---------- sihtmärgi marker ruumis ---------- */
let tMark, tLabel, tLabelTxt = '';
function buildMarker() {
  tMark = new THREE.Sprite(new THREE.SpriteMaterial({ map: RING, color: new THREE.Color(0.5, 2, 0.8), transparent: true, depthTest: false, depthWrite: false, toneMapped: false, opacity: 0.85 })); tMark.renderOrder = 900; scene.add(tMark);
  const c = cnv(512, 96); tLabel = new THREE.Sprite(new THREE.SpriteMaterial({ map: ctex(c), transparent: true, depthTest: false, depthWrite: false, toneMapped: false })); tLabel.userData.c = c; tLabel.renderOrder = 901; scene.add(tLabel);
}
function updateMarker() {
  if (S.target < 0) { tMark.visible = tLabel.visible = false; return; }
  const d = DEST[S.target]; const camP = camera.getWorldPosition(_v); const dist = camP.distanceTo(d.pos);
  tMark.visible = tLabel.visible = true; tMark.position.copy(d.pos); tMark.scale.setScalar(dist * 0.045); tMark.material.rotation = R.time * 0.5;
  _v2.set(0, 1, 0).applyQuaternion(ship.quaternion); tLabel.position.copy(d.pos).addScaledVector(_v2, dist * 0.045); tLabel.scale.set(dist * 0.16, dist * 0.03, 1);
  const s = d.name + ' · ' + fmtDist(dist);
  if (s !== tLabelTxt) { tLabelTxt = s; const c = tLabel.userData.c, g = c.getContext('2d'); g.clearRect(0, 0, 512, 96); txt(g, s, 256, 50, 44, '#c8ffd8', 'center', true); tLabel.material.map.needsUpdate = true; }
}

/* =========================================================
   TEGEVUSED
   ========================================================= */
function say(t) {
  if (R.msg === t && R.msgT > 0) return;
  if (R.msgT > 1.2) { if (!R.queue.includes(t)) { R.queue.push(t); if (R.queue.length > 3) R.queue.shift(); } return; }
  showMsg(t);
}
function showMsg(t) {
  R.msg = t; R.msgT = 4; msgPanel.refresh(); msgPanel.mesh.visible = true; sfx.msg();
  if (etVoice) { try { speechSynthesis.cancel(); const u = new SpeechSynthesisUtterance(t); u.voice = etVoice; u.lang = etVoice.lang; speechSynthesis.speak(u); } catch (e) { } }
}
function updateMsg(dt) {
  if (R.msgT > 0) { R.msgT -= dt; msgPanel.mesh.material.opacity = clamp(R.msgT * 2, 0, 1); if (R.msgT <= 0) { msgPanel.mesh.visible = false; R.msg = ''; } }
  else if (R.queue.length) showMsg(R.queue.shift());
}
function setTarget(i) { if (i < 0 || i >= DEST.length) return; S.target = i; R.faceT = 0; say(`Sihtkoht: ${DEST[i].name} (${fmtDist(ship.position.distanceTo(DEST[i].pos))})`); }
function cycleTarget() { setTarget((S.target + 1) % DEST.length); }
function toggleAutopilot() {
  if (R.autopilot) { R.autopilot = false; R.throttle = 0; say('Autopiloot välja.'); return; }
  if (S.target < 0) { say('Vali enne sihtkoht!'); sfx.deny(); return; }
  const d = DEST[S.target];
  if (ship.position.distanceTo(d.pos) < d.stop + 100) { say(`Oled juba kohal: ${d.name}`); return; }
  R.autopilot = true; R.faceT = 0; say(`Autopiloot sees → ${d.name}. Võid laevas kõndida!`);
}
function setMode(m) {
  if (R.mode === 'walk') R.walkPos = rig.position.clone();
  R.mode = m;
  if (m === 'seat') { rig.position.set(0, 0, -3.3); rig.rotation.set(0, 0, 0); }
  else if (m === 'walk') { if (R.walkPos) rig.position.copy(R.walkPos); else rig.position.set(0.95, 0, -3.2); rig.rotation.set(0, 0, 0); }
  else { rig.position.set(0, 7.5, 44); rig.rotation.set(0, 0, 0); }
  applyDeskCam();
}
function applyDeskCam() { if (renderer.xr.isPresenting) return; camera.position.set(0, R.mode === 'seat' ? 1.3 : 1.65, 0); camera.rotation.set(R.mode === 'outside' ? -0.16 : 0, 0, 0); }
function toggleSeat() {
  if (R.mode === 'seat') { R.walkPos = null; setMode('walk'); if (!R.autopilot) R.throttle = 0; say(R.autopilot ? 'Autopiloot juhib. Kõnni ringi!' : 'Tõusid püsti. Laev jääb seisma.'); }
  else { setMode('seat'); say('Istud roolis.'); }
}
function toggleOutside() { if (R.mode === 'outside') setMode(R.prevMode || 'seat'); else { R.prevMode = R.mode; setMode('outside'); say('Välisvaade. Vajuta X, et tagasi minna.'); } }
function sell() { const n = S.crystals; if (n <= 0) return; S.crystals = 0; S.credits += n * 10; sfx.fanfare(); say(`Müüdud ${n} kristalli: +${n * 10} kr`); save(); }
function buyFuel() { if (S.credits < 30) return; S.credits -= 30; S.fuel = stat.maxFuel; say('Paak on täis!'); save(); }
function repair() { S.shield = stat.maxShield; say('Kilp parandatud!'); save(); }
function refine(n) { let k = 0; while (n-- > 0 && S.crystals >= 3 && S.fuel < stat.maxFuel - 0.5) { S.crystals -= 3; S.fuel = Math.min(stat.maxFuel, S.fuel + 20); k++; } if (k) { sfx.pick(); say(`Rafineeritud! +${k * 20} kütust`); save(); } }
function buyUp(k) { const lv = S.up[k], price = UPG_BASE[k] * (lv + 1); if (lv >= 3 || S.credits < price) return; S.credits -= price; S.up[k]++; if (k === 'tank') S.fuel += 50; if (k === 'shield') S.shield += 50; sfx.fanfare(); say(`${UPG.find(u => u[0] === k)[1]} uuendatud! Tase ${S.up[k]}`); save(); }
function resetGame() {
  if (R.resetArm <= 0) { R.resetArm = 4; return; }
  R.resetArm = 0; Object.assign(S, DEFAULT_S()); save(); R.autopilot = false; R.throttle = 0; R.speed = 0;
  ship.position.set(0, -190, 900); ship.quaternion.identity(); say('Uus mäng algab! Edu, kapten!');
}

/* =========================================================
   SISEND
   ========================================================= */
const IN = { lx: 0, ly: 0, rx: 0, ry: 0 }; const btnPrev = {}; const keys = {};
const dz = v => Math.abs(v) < 0.15 ? 0 : v;
function readXR() {
  IN.lx = IN.ly = IN.rx = IN.ry = 0; const s = renderer.xr.getSession(); if (!s) return;
  for (const src of s.inputSources) {
    const gp = src.gamepad; if (!gp) continue; const h = src.handedness; const ax = gp.axes;
    const x = ax.length >= 4 ? ax[2] : (ax[0] || 0), y = ax.length >= 4 ? ax[3] : (ax[1] || 0);
    if (h === 'left') { IN.lx = dz(x); IN.ly = dz(y); } else if (h === 'right') { IN.rx = dz(x); IN.ry = dz(y); }
    for (const bi of [4, 5]) { const p = !!gp.buttons[bi]?.pressed, k = h + bi; if (p && !btnPrev[k]) onButton(h, bi); btnPrev[k] = p; }
  }
}
function onButton(h, bi) {
  if (h === 'right' && bi === 4) toggleAutopilot(); else if (h === 'right' && bi === 5) toggleSeat();
  else if (h === 'left' && bi === 4) toggleOutside(); else if (h === 'left' && bi === 5) cycleTarget();
}
addEventListener('keydown', e => {
  if (keys[e.code]) return; keys[e.code] = true; initAudio();
  switch (e.code) { case 'KeyP': toggleAutopilot(); break; case 'KeyF': toggleSeat(); break; case 'KeyV': toggleOutside(); break; case 'KeyT': cycleTarget(); break; }
  if (/^Digit[1-8]$/.test(e.code)) setTarget(+e.code.slice(5) - 1);
  if (e.code === 'Space') e.preventDefault();
});
addEventListener('keyup', e => { keys[e.code] = false; });
function readKeys() {
  if (renderer.xr.isPresenting) return; const ax = (a, b) => (keys[a] ? 1 : 0) - (keys[b] ? 1 : 0);
  if (R.mode === 'walk') { IN.lx = ax('KeyD', 'KeyA'); IN.ly = ax('KeyS', 'KeyW'); }
  else { IN.ly = ax('KeyS', 'KeyW'); IN.lx = ax('KeyE', 'KeyQ'); IN.rx = clamp(ax('ArrowRight', 'ArrowLeft') + ax('KeyD', 'KeyA'), -1, 1); IN.ry = ax('ArrowUp', 'ArrowDown'); }
}
const cvs = renderer.domElement;
cvs.addEventListener('mousedown', e => {
  initAudio();
  if (document.pointerLockElement !== cvs) { cvs.requestPointerLock?.(); return; }
  if (e.button === 0) { const h = castCenter(); if (h) activateHit(h); else R.mouseFire = true; }
});
addEventListener('mouseup', () => R.mouseFire = false);
addEventListener('mousemove', e => {
  if (document.pointerLockElement !== cvs || renderer.xr.isPresenting) return;
  rig.rotation.y -= e.movementX * 0.0025; camera.rotation.x = clamp(camera.rotation.x - e.movementY * 0.0025, -1.4, 1.4);
});
const ray = new THREE.Raycaster(); ray.far = 8;
function cast(o, d) { ray.set(o, d); const h = ray.intersectObjects(UIT, false); return h[0] || null; }
function castCenter() { camera.updateMatrixWorld(); return cast(camera.getWorldPosition(new V3()), camera.getWorldDirection(new V3())); }
function castCtrl(c) { _m.identity().extractRotation(c.matrixWorld); return cast(new V3().setFromMatrixPosition(c.matrixWorld), new V3(0, 0, -1).applyMatrix4(_m)); }
function activateHit(h) { const P = h.object.userData.panel; if (P) P.click(h.uv); else if (h.object.userData.dest !== undefined) { sfx.click(); setTarget(h.object.userData.dest); } }
const ctrls = [];
function buildControllers() {
  const cmf = new XRControllerModelFactory();
  for (let k = 0; k < 2; k++) {
    const c = renderer.xr.getController(k); c.userData = { hand: null, src: null, held: false, ui: false };
    c.addEventListener('connected', e => { c.userData.hand = e.data.handedness; c.userData.src = e.data; });
    c.addEventListener('disconnected', () => { c.userData.src = null; });
    c.addEventListener('selectstart', () => { initAudio(); c.userData.held = true; const h = castCtrl(c); c.userData.ui = !!h; if (h) activateHit(h); });
    c.addEventListener('selectend', () => { c.userData.held = false; c.userData.ui = false; });
    const line = new THREE.Line(new THREE.BufferGeometry().setFromPoints([new V3(), new V3(0, 0, -1)]), new THREE.LineBasicMaterial({ color: 0x7ff0ff, transparent: true, opacity: 0.6, toneMapped: false }));
    c.add(line); c.userData.line = line;
    const dot = new THREE.Mesh(new THREE.SphereGeometry(0.012, 8, 6), new THREE.MeshBasicMaterial({ color: 0xbfffff, toneMapped: false })); dot.visible = false; rig.add(dot); c.userData.dot = dot;
    rig.add(c); const gr = renderer.xr.getControllerGrip(k); gr.add(cmf.createControllerModel(gr)); rig.add(gr); ctrls.push(c);
  }
}
function updatePointers() {
  for (const P of PANELS) P.hoverNext = -1;
  const mark = h => { const P = h.object.userData.panel; if (P) P.hoverNext = P.indexAt(h.uv); };
  if (renderer.xr.isPresenting) {
    for (const c of ctrls) {
      const L = c.userData.line; if (!c.userData.src) { L.visible = false; c.userData.dot.visible = false; continue; }
      L.visible = true; const h = castCtrl(c);
      if (h) { L.scale.z = h.distance; L.material.opacity = 0.9; c.userData.dot.visible = true; c.userData.dot.position.copy(rig.worldToLocal(h.point.clone())); mark(h); }
      else { L.scale.z = R.mode === 'walk' ? 2.5 : 1.0; L.material.opacity = 0.3; c.userData.dot.visible = false; }
    }
  } else { const h = castCenter(); if (h) mark(h); $('cross').style.borderColor = h ? '#ffd36a' : 'rgba(140,255,240,.85)'; }
  for (const P of PANELS) if (P.hoverNext !== P.hover) { P.hover = P.hoverNext; P.refresh(); }
}

/* =========================================================
   LENDAMINE, AUTOPILOOT, KÕNDIMINE
   ========================================================= */
function updateControls(dt) {
  const fly = R.mode !== 'walk';
  let fireNow = R.mouseFire || keys.Space, boost = keys.ShiftLeft || keys.ShiftRight;
  for (const c of ctrls) if (c.userData.held && !c.userData.ui) { if (c.userData.hand === 'right') fireNow = true; if (c.userData.hand === 'left') boost = true; }
  R.boost = fly && boost;
  R.fireCd -= dt; if (fly && fireNow && R.fireCd <= 0) { fire(); R.fireCd = 0.14; }
}
function updateFlight(dt) {
  const fuelOK = S.fuel > 0.01;
  if (R.autopilot && R.mode !== 'walk' && (Math.abs(IN.rx) > 0.6 || Math.abs(IN.ry) > 0.6)) { R.autopilot = false; say('Käsijuhtimine – autopiloot välja.'); }
  if (R.autopilot) autopilot(dt, fuelOK);
  else if (R.mode !== 'walk') {
    R.throttle = clamp(R.throttle - IN.ly * dt * 0.6, -0.3, 1);
    const k = 1 - Math.exp(-dt * 4);
    R.yaw = lerp(R.yaw, -IN.rx * 0.75, k); R.pitch = lerp(R.pitch, IN.ry * 0.6, k); R.roll = lerp(R.roll, -IN.lx * 1.1, k);
    ship.quaternion.multiply(_q.setFromEuler(_e.set(R.pitch * dt, R.yaw * dt, R.roll * dt, 'YXZ'))).normalize();
    let tgt = R.throttle * stat.manual; if (R.boost && fuelOK) tgt = stat.pulse; if (!fuelOK) tgt = clamp(tgt, -40, 40);
    R.speed = approach(R.speed, tgt, (tgt > R.speed ? (R.boost ? 380 : 60) : 260) * dt);
    if (R.faceT > 0) faceTarget(dt);
  } else { R.speed = approach(R.speed, 0, 50 * dt); R.yaw = R.pitch = R.roll = 0; if (R.faceT > 0) faceTarget(dt); }
  const a = Math.abs(R.speed);
  if (a > 200) S.fuel = Math.max(0, S.fuel - dt * (a / stat.pulse) * 1.1); else if (a > 1) S.fuel = Math.max(0, S.fuel - dt * 0.05 * a / stat.manual);
  R.vel.copy(fwdOf(ship, _v)).multiplyScalar(R.speed).add(R.bump);
  ship.position.addScaledVector(R.vel, dt); R.bump.multiplyScalar(Math.exp(-dt * 2));
}
function faceTarget(dt) { R.faceT -= dt; if (S.target < 0) return; _m.lookAt(ship.position, DEST[S.target].pos, _v3.set(0, 1, 0).applyQuaternion(ship.quaternion)); _q.setFromRotationMatrix(_m); ship.quaternion.rotateTowards(_q, 0.25 * dt); }
function autopilot(dt, fuelOK) {
  const d = DEST[S.target]; const distC = ship.position.distanceTo(d.pos);
  const aim = _v4.copy(ship.position).sub(d.pos).normalize().multiplyScalar(d.stop).add(d.pos);
  const dist = ship.position.distanceTo(aim);
  _m.lookAt(ship.position, aim, _v3.set(0, 1, 0).applyQuaternion(ship.quaternion)); _q.setFromRotationMatrix(_m);
  const ang = ship.quaternion.angleTo(_q); ship.quaternion.rotateTowards(_q, dt * 0.45);
  let tgt = Math.min(fuelOK ? stat.pulse : 120, Math.sqrt(2 * 140 * Math.max(0, dist - 20)) + 6);
  if (ang > 0.3) tgt = Math.min(tgt, ang > 0.9 ? 40 : 140);
  R.speed = approach(R.speed, tgt, (tgt > R.speed ? 170 : 320) * dt);
  if (dist < 60 || distC < d.stop + 20) {
    R.autopilot = false; R.throttle = 0; R.faceT = 6;
    const extra = d.type === 'field' ? ' Lenda sisse ja tulista kive!' : d.type === 'station' ? ' Müü kristalle juhtpaneelilt.' : ' Milline vaade!';
    say(`Kohal: ${d.name}.${extra}`);
  }
}
function hitShip(dmg) {
  if (R.time - R.lastHit < 0.7) return; R.lastHit = R.time; S.shield = Math.max(0, S.shield - dmg); sfx.alarm(); haptic('left', 0.8, 150); haptic('right', 0.8, 150);
  say(S.shield > 0 ? 'Ettevaatust! Kilp sai pihta.' : 'Kilp on tühi! Lenda ettevaatlikult.');
}
function shipCollisions(dt) {
  const P = ship.position, SR = 12;
  if (P.distanceTo(FIELD_POS) < 1500) for (const a of AST) {
    if (!a.alive) continue; const m = a.r + SR; const d2 = P.distanceToSquared(a.pos); if (d2 > m * m) continue;
    const d = Math.sqrt(d2); _v.copy(P).sub(a.pos).normalize(); P.addScaledVector(_v, m - d); R.bump.addScaledVector(_v, Math.max(20, Math.abs(R.speed) * 0.4)); R.speed *= 0.35; hitShip(Math.min(30, 8 + a.r));
  }
  for (const p of PL) { const m = p.r * 1.03 + SR + 20; const d = P.distanceTo(p.pos); if (d < m) { _v.copy(P).sub(p.pos).normalize(); P.addScaledVector(_v, m - d); R.speed = Math.min(R.speed, 20); if (R.time - R.lastWarn > 4) { R.lastWarn = R.time; say('Liiga lähedal planeedile!'); } } }
  const L = _v.copy(P).sub(STATION_POS); const ll = L.length(); if (ll > 420) return; const rh = Math.hypot(L.x, L.z); let push = null;
  if (ll < 62 + SR) push = _v2.copy(L).normalize().multiplyScalar(62 + SR - ll);
  else if (rh < 20 + SR && Math.abs(L.y) < 170) push = _v2.set(L.x, 0, L.z).normalize().multiplyScalar(20 + SR - rh);
  else { const dr = rh - 220, dd = Math.hypot(dr, L.y); if (dd < 18 + SR) push = _v2.set(L.x / rh * dr, L.y, L.z / rh * dr).normalize().multiplyScalar(18 + SR - dd); }
  if (push) { P.add(push); R.speed *= 0.4; R.bump.addScaledVector(push.normalize(), 15); hitShip(5); }
}
const OBST = [[-2.8, 2.8, -6.2, -4.95], [-0.65, 0.65, 0.35, 1.65], [-3, -2.25, 0.9, 3.5], [2.3, 3, 2.5, 3.1], [-3, -1.75, 4.9, 6.9], [1.75, 3, 4.9, 6.8], [-3, -1.55, 7.2, 8.8], [1.55, 3, 7.2, 8.8], [-0.55, 0.55, 8.9, 10]];
function walkable(x, z) {
  const pr = 0.22; if (x < -2.75 || x > 2.75 || z < -4.85 || z > 9.75) return false;
  for (const zp of [-2, 4]) if (Math.abs(z - zp) < 0.08 + pr && Math.abs(x) > 0.9 - pr) return false;
  for (const o of OBST) if (x > o[0] - pr && x < o[1] + pr && z > o[2] - pr && z < o[3] + pr) return false;
  return true;
}
function updateWalk(dt) {
  if (R.mode !== 'walk') return;
  camera.getWorldDirection(_v); _v.applyQuaternion(_q.copy(ship.quaternion).invert()); _v.y = 0; if (_v.lengthSq() < 1e-4) _v.set(0, 0, -1); _v.normalize();
  const rx = -_v.z, rz = _v.x, sp = 2.4 * dt;
  const mx = (_v.x * -IN.ly + rx * IN.lx) * sp, mz = (_v.z * -IN.ly + rz * IN.lx) * sp; const p = rig.position;
  if (walkable(p.x + mx, p.z + mz)) { p.x += mx; p.z += mz; } else if (walkable(p.x + mx, p.z)) p.x += mx; else if (walkable(p.x, p.z + mz)) p.z += mz;
  if (renderer.xr.isPresenting) { if (Math.abs(IN.rx) > 0.7) { if (!R.snapLock) { rig.rotation.y += IN.rx > 0 ? -PI / 6 : PI / 6; R.snapLock = true; } } else if (Math.abs(IN.rx) < 0.3) R.snapLock = false; }
}
function checkProximity() {
  const P = ship.position;
  const near = P.distanceTo(STATION_POS) < 1100; if (near && !R.nearStation) say('Tere tulemast Tähesadama jaama!'); R.nearStation = near;
  const inF = P.distanceTo(FIELD_POS) < 1300; if (inF && !R.inField) say('Asteroidiväli! Tulista kive ja korja kristallid.'); R.inField = inF;
  for (const p of PL) if (!S.discovered.includes(p.name) && P.distanceTo(p.pos) < p.r * 2.6 + 600) { S.discovered.push(p.name); S.credits += 50; sfx.fanfare(); say(`Uus planeet avastatud: ${p.name}! +50 kr`); save(); }
  if (S.fuel < 15 && !R.lowFuel) { R.lowFuel = true; say('Kütus on madal! Rafineeri kristalle lastiruumis.'); } else if (S.fuel > 30) R.lowFuel = false;
  if (R.time - R.lastHit > 3 && S.shield < stat.maxShield) S.shield = Math.min(stat.maxShield, S.shield + 0.8);
}
function updateShipFX(dt) {
  const f = clamp(Math.abs(R.speed) / stat.pulse, 0, 1), base = 0.35 + clamp(Math.abs(R.speed) / stat.manual, 0, 1) * 0.5 + f * 1.6;
  for (const g of shipFX.glows) { g.scale.setScalar(g.userData.base * base * (0.92 + Math.random() * 0.16)); }
  shipFX.stick.rotation.set(-IN.ry * 0.35, 0, -IN.rx * 0.35); shipFX.lever.rotation.x = -R.throttle * 0.6;
  shipFX.reticle.visible = R.mode !== 'walk';
  shipFX.tank.count = Math.min(40, S.crystals);
}

/* =========================================================
   KÄIVITAMINE
   ========================================================= */
let ready = false; const clock = new THREE.Clock(); let accUI = 0, accP = 0;
function loop() {
  const dt = Math.min(clock.getDelta(), 0.05); if (!ready) { renderer.render(scene, camera); return; }
  step(dt);
  renderer.render(scene, camera);
}
function step(dt) {
  R.time += dt;
  readXR(); readKeys(); updateControls(dt);
  updateFlight(dt); shipCollisions(dt); ship.updateMatrixWorld(true);
  updateWalk(dt); updateLasers(dt); updateAsteroids(dt); updateCrystals(dt); updateNPCs(dt);
  station.ring.rotation.y += dt * 0.03; for (const b of station.beacons) b.visible = (R.time % 2) < 0.25;
  for (const p of PL) { p.mesh.rotation.y += dt * 0.004; if (p.cloud) p.cloud.rotation.y += dt * 0.002; }
  sky.position.copy(ship.position); updateDust();
  scene.updateMatrixWorld();
  updateHolo(); updateMarker(); updateShipFX(dt);
  accP += dt; if (accP > 0.4) { accP = 0; checkProximity(); }
  if (R.resetArm > 0) R.resetArm -= dt;
  updatePointers();
  accUI += dt; if (accUI > 0.2) { accUI = 0; for (const p of PANELS) p.check(); }
  updateMsg(dt); updateAudio();
}
renderer.setAnimationLoop(loop);

async function init() {
  setLoad(0.05, 'Süütan tähed…'); await nextFrame();
  const pm = new THREE.PMREMGenerator(renderer); const es = new THREE.Scene();
  const ec = cnv(256, 128), eg = ec.getContext('2d'), gr = eg.createLinearGradient(0, 0, 0, 128); gr.addColorStop(0, '#25305e'); gr.addColorStop(0.5, '#0b0e20'); gr.addColorStop(1, '#22102c'); eg.fillStyle = gr; eg.fillRect(0, 0, 256, 128);
  const et = ctex(ec); es.add(new THREE.Mesh(new THREE.SphereGeometry(50, 32, 16), new THREE.MeshBasicMaterial({ map: et, side: THREE.BackSide })));
  const es2 = new THREE.Mesh(new THREE.SphereGeometry(6, 16, 8), new THREE.MeshBasicMaterial({ color: new THREE.Color(40, 34, 26) })); es2.position.copy(SUN_DIR).multiplyScalar(40); es.add(es2);
  scene.environment = pm.fromScene(es, 0.04).texture;
  buildSky();
  setLoad(0.12, 'Ehitan kosmosejaama…'); await nextFrame(); buildStation();
  setLoad(0.18, 'Paigutan asteroide…'); await nextFrame(); buildField();
  await buildPlanets();
  setLoad(0.88, 'Ehitan laevu…'); await nextFrame();
  buildDest();
  HAULER = buildHaulerGeo(); FIGHTER = buildFighterGeo();
  const pal = [shipMats('#c9c9c4', '#b8521f', '#3a3c40', 501), shipMats('#d0d4d8', '#2f6fa8', '#34383e', 502), shipMats('#9aa3a8', '#5e4f6e', '#2a2a30', 503), shipMats('#a4a8ac', '#a83a2a', '#2a2a30', 504)];
  addNPC(FIGHTER, pal[2], 'orbit', { R: 380, h: 60, bob: 40, a: 0, speed: 120, bank: -0.35 });
  addNPC(FIGHTER, pal[3], 'orbit', { R: 520, h: -50, bob: 30, a: 2.5, speed: 140, bank: -0.3 });
  addNPC(HAULER, pal[0], 'orbit', { R: 300, h: 110, bob: 10, a: 4.2, speed: 28, bank: -0.05 });
  addNPC(HAULER, pal[1], 'orbit', { R: 640, h: 20, bob: 20, a: 1.2, speed: 45, bank: -0.08 });
  addNPC(HAULER, pal[0], 'travel', { speed: 160 }); addNPC(HAULER, pal[1], 'travel', { speed: 140 });
  addNPC(FIGHTER, pal[2], 'travel', { speed: 260 }); addNPC(FIGHTER, pal[3], 'travel', { speed: 300 });
  buildPlayerShip(); buildPools(); buildDust(); buildPanels(); buildHolo(); buildMarker(); buildControllers();
  setMode('seat');
  document.body.appendChild(VRButton.createButton(renderer, { optionalFeatures: ['local-floor', 'bounded-floor', 'hand-tracking'] }));
  renderer.xr.addEventListener('sessionstart', () => { initAudio(); $('info').style.display = 'none'; $('cross').style.display = 'none'; camera.position.set(0, 0, 0); camera.rotation.set(0, 0, 0); });
  renderer.xr.addEventListener('sessionend', () => { $('cross').style.display = ''; applyDeskCam(); });
  setLoad(1, 'Valmis!'); await nextFrame();
  $('load').style.opacity = 0; setTimeout(() => $('load').remove(), 700);
  ready = true;
  say(S.mined > 0 || S.discovered.length ? 'Tere tagasi, kapten!' : 'Tere, kapten! Vali sihtkoht ja lülita autopiloot sisse.');
  window.G = { S, R, ship, rig, camera, setMode, setTarget, toggleAutopilot, toggleSeat, keys, NPCS, DEST, renderer, scene, AST, CRYS, breakRock, fire, IN, refine, sell, buyUp, walkable, sim: (sec) => { for (let t = 0; t < sec; t += 0.05) step(0.05); } };
}
init();
</script>
</body>
</html>
