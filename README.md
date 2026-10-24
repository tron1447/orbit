[Uploading index (35).html…]()<!doctype html>
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
  #info{position:fixed;left:14px;top:14px;max-width:350px;padding:14px 16px;background:rgba(4,14,30,.8);border:1px solid rgba(42,212,255,.45);border-radius:12px;font-size:13px;line-height:1.55;z-index:5}
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
  <b>Quest:</b> vajuta all <b>ENTER VR</b>. Vasaku käe randmel on menüü!<br>
  <b>Vasak pulk</b> – hoob / kõndimine<br>
  <b>Parem pulk</b> – juhtimine / pööramine<br>
  <b>Parem päästik</b> – laser · nupud · jalgsi skanner/kaevur<br>
  <b>Vasak päästik</b> – pulsikiirus · jalgsi raketivöö<br>
  <b>A</b> autopiloot · <b>B</b> istu / tõuse / sisene<br>
  <b>X</b> välisvaade · <b>Y</b> järgmine sihtkoht
  <hr>
  <b>Arvutis:</b> klõpsa ekraanil, hiirega vaatad<br>
  <span class="k">W</span><span class="k">S</span> hoob/kõnd · <span class="k">A</span><span class="k">D</span> pööra · nooled kalle · <span class="k">Q</span><span class="k">E</span> veere<br>
  <span class="k">Shift</span> pulss · klõps/<span class="k">Space</span> laser (jalgsi Space = raketivöö)<br>
  <span class="k">P</span> autopiloot · <span class="k">F</span> istu/tõuse · <span class="k">V</span> välisvaade · <span class="k">T</span>/<span class="k">1</span>–<span class="k">8</span> siht<br>
  <span class="k">L</span> maandu/stardi · <span class="k">G</span> välju/sisene · <span class="k">H</span> hüperhüpe · <span class="k">M</span> muusika
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
const smooth = (a, b, x) => { const t = clamp((x - a) / (b - a), 0, 1); return t * t * (3 - 2 * t); };
const approach = (a, b, s) => a < b ? Math.min(b, a + s) : Math.max(b, a - s);
const nextFrame = () => new Promise(r => requestAnimationFrame(() => r()));
function mulberry(a) { return function () { a |= 0; a = a + 0x6D2B79F5 | 0; let t = Math.imul(a ^ a >>> 15, 1 | a); t = t + Math.imul(t ^ t >>> 7, 61 | t) ^ t; return ((t ^ t >>> 14) >>> 0) / 4294967296; }; }
const R0 = mulberry(20261024);
const rnd = (a = 0, b = 1) => a + R0() * (b - a);
const randUnit = (out = new V3(), r = R0) => { do { out.set(r() * 2 - 1, r() * 2 - 1, r() * 2 - 1); } while (out.lengthSq() > 1 || out.lengthSq() < 0.01); return out.normalize(); };
const _v = new V3(), _v2 = new V3(), _v3 = new V3(), _v4 = new V3(), _q = new Q4(), _m = new THREE.Matrix4(), _e = new THREE.Euler();
const UP = new V3(0, 1, 0);
const fwdOf = (o, out = new V3()) => out.set(0, 0, -1).applyQuaternion(o.quaternion);
function setLoad(f, t) { const b = $('barf'); if (!b) return; b.style.width = (f * 100) + '%'; if (t) $('load').querySelector('p').textContent = t; }
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
const SYL = ['ka', 'lu', 'mi', 'ro', 've', 'ti', 'sa', 'no', 'pe', 'ru', 'ki', 'ja', 'mo', 'la', 'su', 'vi', 'tõ', 'hä', 'õr', 'nu', 'kel', 'par', 'sil', 'tor', 'mä', 'ree'];
function genName(r, n = 2) { let s = ''; for (let i = 0; i < n; i++) s += SYL[(r() * SYL.length) | 0]; return s[0].toUpperCase() + s.slice(1); }

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
let activeScene = scene;
const camera = new THREE.PerspectiveCamera(72, innerWidth / innerHeight, 0.05, 60000);
camera.rotation.order = 'YXZ';
addEventListener('resize', () => { camera.aspect = innerWidth / innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth, innerHeight); });

const SUN_DIR = new V3(0.72, 0.28, 0.35).normalize();
const sunLight = new THREE.DirectionalLight('#fff0d8', 3.4);
sunLight.position.copy(SUN_DIR).multiplyScalar(100);
scene.add(sunLight);
scene.add(new THREE.HemisphereLight('#5868a0', '#140a1c', 0.45));
const SUNS = [['#fff0d8', '#fff6e0', '#ffe2a0', '#ffb060'], ['#ffd4a8', '#ffe0c0', '#ffb070', '#ff7030'], ['#d8e8ff', '#f0f6ff', '#a8c8ff', '#6090ff'], ['#ffc0b0', '#ffd0c0', '#ff8a6a', '#ff4a3a'], ['#e8ffd8', '#f6fff0', '#c8ffa0', '#80e060']];

/* =========================================================
   TEKSTUURID
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
  for (let y = 0; y < 256; y += 16) for (let x = 0; x < 256; x += 16) { g.fillStyle = ((x + y) / 16) % 2 ? '#30353c' : '#2a2f35'; g.fillRect(x + 1, y + 1, 14, 14); g.fillStyle = 'rgba(255,255,255,0.07)'; g.fillRect(x + 3, y + 7, 10, 2); }
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
const KEEP_TEX = new Set([GLOW, DOT, RING]);
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
  const m = new THREE.SpriteMaterial({ map: GLOW, color, blending: THREE.AdditiveBlending, transparent: true, depthWrite: false, toneMapped: false, opacity, fog: false });
  const s = new THREE.Sprite(m); s.scale.setScalar(size); return s;
}
function disposeTree(o, keepMaps = false) {
  o.traverse(c => {
    if (c.geometry) c.geometry.dispose();
    const ms = c.material ? (Array.isArray(c.material) ? c.material : [c.material]) : [];
    for (const m of ms) { if (!keepMaps) for (const k of ['map', 'emissiveMap']) if (m[k] && !KEEP_TEX.has(m[k])) m[k].dispose(); m.dispose(); }
  });
}

/* =========================================================
   GEOMEETRIA EHITAJA
   ========================================================= */
function tBox(w, h, d, tile = 4) {
  const g = new THREE.BoxGeometry(w, h, d); const uv = g.attributes.uv;
  const dims = [[d, h], [d, h], [w, d], [w, d], [w, h], [w, h]]; const ou = Math.random(), ov = Math.random();
  for (let f = 0; f < 6; f++) for (let i = 0; i < 4; i++) { const k = f * 4 + i; uv.setXY(k, ou + uv.getX(k) * dims[f][0] / tile, ov + uv.getY(k) * dims[f][1] / tile); }
  return g;
}
function taper(g, tx, ty) {
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
function buildHaulerGeo() {
  const b = new Builder(), r = mulberry(77);
  b.add('hull', tBox(12, 4, 22), 0, 0, 0); b.add('hull', tBox(9, 2, 14), 0, 3, 2);
  b.add('hull', taper(tBox(8, 2.2, 7), 0.7, 0.6), 0, -0.4, -14); b.add('hull', tBox(6, 2.4, 5), 0, 1.6, -11);
  b.add('glass', tBox(5, 1.1, 0.3), 0, 2.2, -13.4, -0.45);
  b.mirror('accent', () => tBox(5, 3, 16), 9, -0.5, 1); b.mirror('dark', () => tBox(3, 1.5, 8), 6.5, 0, 0);
  b.mirror('hull', () => tBox(7, 0.6, 10), 12.5, -0.6, 3, 0, 0, -0.12); b.mirror('accent', () => tBox(2.5, 0.9, 4), 15.6, -1.0, 5);
  for (const x of [-2.5, 0, 2.5]) { b.add('accent', cylZ(1.1, 8, 14), x, 4.9, 3); for (const z of [0, 6]) b.add('dark', cylZ(1.25, 0.4, 14), x, 4.9, z); }
  b.mirror('dark', () => cylZ(1.9, 4, 16, 2.2), 3.8, 0.2, 12); b.mirror('dark', () => cylZ(1.1, 3, 12), 9, -0.5, 10);
  b.mirror('glow', () => disc(1.6), 3.8, 0.2, 14.05); b.mirror('glow', () => disc(0.9), 9, -0.5, 11.55);
  b.add('dark', tBox(0.25, 3.5, 0.25), 3.2, 5.8, 7); b.add('dark', tBox(0.25, 2.5, 0.25), -3.6, 5.3, 9);
  b.mirror('dark', () => cylZ(0.2, 3, 6), 2, -0.6, -18);
  for (let i = 0; i < 26; i++) { const w = 0.5 + r() * 1.6, h = 0.3 + r() * 0.9, d = 0.5 + r() * 2; b.add(r() < 0.5 ? 'dark' : 'hull', tBox(w, h, d), (r() - 0.5) * 11, 2 + h / 2, (r() - 0.5) * 19); }
  return { geos: b.geos(), engines: [[3.8, 0.2, 14.6, 1.6], [-3.8, 0.2, 14.6, 1.6], [9, -0.5, 12, 0.9], [-9, -0.5, 12, 0.9]], navs: [[16.8, -1, 5, 'r'], [-16.8, -1, 5, 'g']] };
}
function buildFighterGeo() {
  const b = new Builder();
  b.add('hull', tBox(4.2, 2.6, 20), 0, 0, 1); b.add('hull', taper(tBox(4.2, 2.6, 10), 0.3, 0.35), 0, -0.2, -14);
  b.add('glass', taper(tBox(2.4, 1.3, 6), 0.5, 0.4), 0, 1.75, -7.5); b.add('dark', tBox(2.2, 1.6, 15), 0, 1.9, 4);
  b.mirror('hull', () => tBox(9, 0.5, 8), 6, -0.4, 6, 0, -0.35, 0); b.mirror('dark', () => cylZ(1, 8, 12), 10.5, -0.4, 7.5);
  b.mirror('accent', () => tBox(0.4, 4, 5), 2.3, 2.8, 8, 0, 0, -0.3); b.mirror('hull', () => tBox(1.6, 2, 4.5), 3.1, 4.6, 8.5);
  b.mirror('dark', () => cylZ(1.15, 3, 14, 1.35), 1.3, 0, 12); b.mirror('dark', () => tBox(0.3, 0.3, 6), 1.2, -1.4, -6);
  b.mirror('accent', () => tBox(1.2, 0.6, 4), 4.5, -0.2, -2);
  b.mirror('glow', () => disc(1.0), 1.3, 0, 13.55); b.mirror('glow', () => disc(0.8), 10.5, -0.4, 11.55);
  return { geos: b.geos(), engines: [[1.3, 0, 14, 1.0], [-1.3, 0, 14, 1.0], [10.5, -0.4, 12, 0.8], [-10.5, -0.4, 12, 0.8]], navs: [[10.5, -0.4, 4, 'r'], [-10.5, -0.4, 4, 'g']] };
}

/* =========================================================
   MÄNGU OLEK
   ========================================================= */
const SAVE_KEY = 'tahesadam_v1';
const DEFAULT_S = () => ({ crystals: 0, credits: 0, fuel: 100, shield: 100, up: { laser: 0, engine: 0, tank: 0, shield: 0, magnet: 0 }, discovered: [], mined: 0, target: -1, system: 0, scanned: [], monos: [], easy: false, music: true });
const S = DEFAULT_S();
try { const d = JSON.parse(localStorage.getItem(SAVE_KEY) || 'null'); if (d) { Object.assign(S, d); S.up = Object.assign(DEFAULT_S().up, d.up || {}); } } catch (e) { }
S.discovered = (S.discovered || []).map(n => n.includes(':') ? n : '0:' + n); S.scanned ??= []; S.monos ??= []; S.system ??= 0; S.music ??= true; S.easy ??= false;
const stat = {
  get maxFuel() { return 100 + 50 * S.up.tank; }, get maxShield() { return 100 + 50 * S.up.shield; },
  get dmg() { return 1 + S.up.laser; }, get pulse() { return 900 + 300 * S.up.engine; },
  get manual() { return 120 + 40 * S.up.engine; }, get magnet() { return 220 + 160 * S.up.magnet; }
};
function save() { try { localStorage.setItem(SAVE_KEY, JSON.stringify(S)); } catch (e) { } }
setInterval(save, 5000);
const R = { mode: 'seat', prevMode: 'seat', walkPos: null, autopilot: false, throttle: 0, speed: 0, boost: false, yaw: 0, pitch: 0, roll: 0,
  bump: new V3(), vel: new V3(), lastHit: -9, lastWarn: -9, faceT: 0, nearStation: false, nearPlanet: null, inField: false, fireCd: 0, gun: 0, time: 0,
  msg: '', msgT: 0, queue: [], resetArm: 0, lowFuel: false, mouseFire: false, snapLock: false,
  landed: null, busy: null, hyperT: 0, hyperReady: false, hyperTo: 0, spaceBack: new V3(), toolHeld: false, jetHeld: false, jv: 0, jy: 0,
  ev: new V3(), scanObj: null, scanT: 0, doorWarn: -9, fade: 0, fadeTarget: 0, fadeCb: null };
const flying = () => (R.mode === 'seat' || R.mode === 'outside') && !R.landed && !R.busy;
const onFoot = () => R.mode === 'eva' || R.mode === 'ground';

/* =========================================================
   TAEVAS
   ========================================================= */
const sky = new THREE.Group(); scene.add(sky);
const skyStars = new THREE.Group(); sky.add(skyStars);
const sunSprites = [];
function buildSky() {
  const mk = (N, fn, size, op) => {
    const pos = new Float32Array(N * 3), col = new Float32Array(N * 3);
    for (let i = 0; i < N; i++) { const [p, c] = fn(i); pos.set([p.x, p.y, p.z], i * 3); col.set(c, i * 3); }
    const g = new THREE.BufferGeometry(); g.setAttribute('position', new THREE.BufferAttribute(pos, 3)); g.setAttribute('color', new THREE.BufferAttribute(col, 3));
    const pts = new THREE.Points(g, new THREE.PointsMaterial({ size, sizeAttenuation: false, map: DOT, vertexColors: true, transparent: true, opacity: op, depthWrite: false, blending: THREE.AdditiveBlending, toneMapped: false }));
    pts.frustumCulled = false; pts.renderOrder = -10; skyStars.add(pts);
  };
  const starCol = () => { const t = rnd(); const b = rnd(0.35, 1); return t < 0.15 ? [0.7 * b, 0.8 * b, 1.3 * b] : t < 0.25 ? [1.3 * b, 0.85 * b, 0.6 * b] : [b, b, b]; };
  mk(9000, () => [randUnit(new V3()).multiplyScalar(45000), starCol()], 2.2, 0.9);
  mk(500, () => [randUnit(new V3()).multiplyScalar(45000), starCol().map(v => v * 1.6)], 4.2, 1);
  const n = new V3(0.25, 0.92, 0.3).normalize(), u = new V3().crossVectors(n, new V3(1, 0, 0)).normalize(), w = new V3().crossVectors(n, u);
  const gauss = () => (rnd() + rnd() + rnd() + rnd() - 2) / 2;
  mk(18000, () => { const t = rnd(0, PI * 2); const p = u.clone().multiplyScalar(Math.cos(t)).addScaledVector(w, Math.sin(t)).addScaledVector(n, gauss() * 0.16).normalize().multiplyScalar(44000); const b = rnd(0.2, 0.6); return [p, [b * 0.8, b * 0.85, b * 1.1]]; }, 1.8, 0.8);
  const NEB = ['#6a34e0', '#167a8a', '#c02c7a', '#2a50d0', '#8a38d8', '#b85a30'];
  const ct = [cloudTex(1), cloudTex(2), cloudTex(3)];
  for (let i = 0; i < 18; i++) {
    const t = rnd(0, PI * 2); const p = u.clone().multiplyScalar(Math.cos(t)).addScaledVector(w, Math.sin(t)).addScaledVector(n, rnd(-0.22, 0.22)).normalize().multiplyScalar(42000);
    const m = new THREE.SpriteMaterial({ map: ct[i % 3], color: NEB[i % NEB.length], blending: THREE.AdditiveBlending, transparent: true, depthWrite: false, opacity: rnd(0.25, 0.5), toneMapped: false, rotation: rnd(0, PI * 2) });
    const s = new THREE.Sprite(m); s.position.copy(p); s.scale.setScalar(rnd(14000, 32000)); s.renderOrder = -9; skyStars.add(s);
  }
  const sp = SUN_DIR.clone().multiplyScalar(44000);
  for (const [sz, op] of [[2600, 1], [9000, 0.9], [26000, 0.35]]) { const s = glowSprite('#ffffff', sz, op); s.position.copy(sp); s.renderOrder = -8; sky.add(s); sunSprites.push(s); }
}
function applySun(i) { const c = SUNS[i % SUNS.length]; sunLight.color.set(c[0]); sunSprites.forEach((s, k) => s.material.color.set(c[k + 1])); }

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
  b.add('hull', new THREE.SphereGeometry(60, 32, 20)); b.add('hull', cylY(20, 330, 24));
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
   ASTEROIDID
   ========================================================= */
const FIELD_POS = new V3(3500, 200, -5000);
const AST = [];
let rockIMs = [], rockMat, debrisGeo, debrisMat;
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
function buildRocks() {
  rockMat = new THREE.MeshStandardMaterial({ vertexColors: true, roughness: 0.95, metalness: 0.05, flatShading: true });
  const geos = [0, 1, 2, 3, 4].map(i => makeRockGeo(i * 13, 2));
  rockIMs = geos.map(g => { const im = new THREE.InstancedMesh(g, rockMat, 30); im.frustumCulled = false; im.instanceMatrix.setUsage(THREE.DynamicDrawUsage); scene.add(im); return im; });
  for (let i = 0; i < 150; i++) { const a = { im: rockIMs[i % 5], idx: (i / 5) | 0, pos: new V3(), r: 0, hp: 0, alive: true, q: new Q4(), axis: randUnit(new V3()), spin: rnd(0.05, 0.3), respawn: 0 }; AST.push(a); }
  debrisGeo = new THREE.IcosahedronGeometry(1, 0);
  debrisMat = new THREE.MeshStandardMaterial({ color: '#7a6e62', roughness: 1, flatShading: true });
}
function placeRock(a, init) {
  for (let k = 0; k < 20; k++) { randUnit(a.pos).multiplyScalar(Math.pow(rnd(), 0.5) * 880); a.pos.y *= 0.45; a.pos.add(FIELD_POS); if (init || a.pos.distanceTo(ship.position) > 350) break; }
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
   PÄIKESESÜSTEEMID JA PLANEEDID
   ========================================================= */
const ATM = { toxic: '#b8f060', ocean: '#6ab4ff', desert: '#ffb070', ice: '#aee6ff', lava: '#ff6a2a', gas: '#ffd9a0' };
const PLANETS0 = [
  { name: 'Viridia', p: [5600, -300, -7200], r: 1100, kind: 'toxic', clouds: true },
  { name: 'Okeanos', p: [-6500, 600, -2800], r: 800, kind: 'ocean', clouds: true },
  { name: 'Kaljula', p: [-2600, -900, -10500], r: 550, kind: 'desert' },
  { name: 'Külmaste', p: [9500, 1400, 2500], r: 450, kind: 'ice' },
  { name: 'Hõõguja', p: [-9000, -1600, 6500], r: 500, kind: 'lava' },
  { name: 'Vöölane', p: [2500, 2600, 15000], r: 2000, kind: 'gas', rings: true },
];
function genSystem(i) {
  if (i === 0) return { name: 'Kodusüsteem', sun: 0, skyRot: 0, field: new V3(3500, 200, -5000),
    planets: PLANETS0.map((p, k) => ({ name: p.name, pos: new V3(...p.p), r: p.r, kind: p.kind, atm: ATM[p.kind], clouds: !!p.clouds, rings: !!p.rings, seed: k * 31 + 7 })) };
  const r = mulberry(1000 + i * 7919); const n = 4 + ((r() * 3) | 0); const planets = []; const KINDS = ['toxic', 'ocean', 'desert', 'ice', 'lava'];
  for (let k = 0; k < n; k++) {
    const kind = (k === n - 1 && r() < 0.65) ? 'gas' : KINDS[(r() * 5) | 0]; const rad = kind === 'gas' ? 1400 + r() * 900 : 380 + r() * 750;
    let p = new V3();
    for (let t = 0; t < 40; t++) { p = randUnit(new V3(), r); p.y *= 0.25; p.normalize().multiplyScalar(3500 + r() * 12000); if (planets.every(q => q.pos.distanceTo(p) > q.r + rad + 2500) && p.length() > rad + 1800) break; }
    planets.push({ name: genName(r, 2 + (r() < 0.3 ? 1 : 0)), pos: p, r: rad, kind, atm: ATM[kind], clouds: kind === 'ocean' || kind === 'toxic' || (kind === 'ice' && r() < 0.5), rings: kind === 'gas' && r() < 0.75, seed: (r() * 1e6) | 0 });
  }
  let field = new V3();
  for (let t = 0; t < 40; t++) { field = randUnit(new V3(), r); field.y *= 0.2; field.normalize().multiplyScalar(3000 + r() * 4000); if (planets.every(q => q.pos.distanceTo(field) > q.r + 2600)) break; }
  return { name: genName(r, 2) + ' ' + ['Alfa', 'Beeta', 'Gamma', 'Delta', 'Prima'][(r() * 5) | 0], sun: 1 + ((r() * 4) | 0), skyRot: r() * 6, field, planets };
}
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
let sysG = null, SYSINFO = null; const PL = [];
async function buildSystem(i, loadUI = false) {
  const sys = genSystem(i); const g = new THREE.Group(); const newPL = [];
  for (let k = 0; k < sys.planets.length; k++) {
    const p = sys.planets[k]; if (loadUI) setLoad(0.25 + k * 0.1, `Loon planeeti ${p.name}…`); await nextFrame();
    const tx = await planetTextures(p, p.seed);
    const mat = new THREE.MeshStandardMaterial({ map: tx.map, roughness: 0.92, metalness: 0 });
    if (tx.emis) { mat.emissiveMap = tx.emis; mat.emissive = new THREE.Color(1, 1, 1); mat.emissiveIntensity = 1.6; }
    const mesh = new THREE.Mesh(new THREE.SphereGeometry(p.r, 64, 40), mat);
    mesh.position.copy(p.pos); mesh.rotation.set(rnd(-0.3, 0.3), rnd(0, 6), rnd(-0.3, 0.3)); g.add(mesh); p.mesh = mesh;
    if (tx.clouds) { const cm = new THREE.Mesh(new THREE.SphereGeometry(p.r * 1.015, 64, 40), new THREE.MeshStandardMaterial({ map: tx.clouds, transparent: true, depthWrite: false, roughness: 1 })); mesh.add(cm); p.cloud = cm; }
    const atm = new THREE.Mesh(new THREE.SphereGeometry(p.r * 1.07, 64, 40), atmMat(p.atm, p.kind === 'toxic' ? 1.8 : 2.6, p.kind === 'toxic' ? 1.7 : 1.25));
    atm.position.copy(p.pos); g.add(atm);
    if (p.rings) {
      const rg = new THREE.RingGeometry(p.r * 1.35, p.r * 2.35, 160, 1); const pa = rg.attributes.position, ua = rg.attributes.uv;
      for (let q = 0; q < pa.count; q++) { const d = Math.hypot(pa.getX(q), pa.getY(q)); ua.setXY(q, (d - p.r * 1.35) / p.r, 0.5); }
      const c = cnv(1024, 4), cg = c.getContext('2d'), r2 = mulberry(p.seed + 5);
      for (let x = 0; x < 1024; x++) { const a = (0.25 + r2() * 0.55) * (Math.sin(x * 0.05) * 0.3 + 0.7) * (x < 40 || x > 1000 ? 0.2 : 1) * (x > 600 && x < 640 ? 0.1 : 1); const b = 180 + r2() * 60; cg.fillStyle = `rgba(${b},${b * 0.9 | 0},${b * 0.75 | 0},${a})`; cg.fillRect(x, 0, 1, 4); }
      const ring = new THREE.Mesh(rg, new THREE.MeshStandardMaterial({ map: ctex(c), transparent: true, side: THREE.DoubleSide, depthWrite: false, roughness: 1 }));
      ring.position.copy(p.pos); ring.rotation.set(-PI / 2 + 0.62, 0.25, 0); g.add(ring);
    }
    newPL.push(p);
  }
  const dg = makeRockGeo(999 + i, 1); const dec = new THREE.InstancedMesh(dg, rockMat, 650); dec.frustumCulled = false;
  for (let k = 0; k < 650; k++) { const p = randUnit(_v).multiplyScalar(rnd(950, 1800)); p.y *= 0.45; p.add(sys.field); _q.setFromEuler(_e.set(rnd(0, 6), rnd(0, 6), rnd(0, 6))); const s = rnd(3, 22); dec.setMatrixAt(k, _m.compose(p, _q, _v2.set(s, s, s))); }
  g.add(dec);
  for (let k = 0; k < 8; k++) { const s = new THREE.Sprite(new THREE.SpriteMaterial({ map: cloudTex(40 + k), color: '#8ab04a', blending: THREE.AdditiveBlending, transparent: true, depthWrite: false, opacity: 0.16, toneMapped: false })); s.position.copy(randUnit(_v).multiplyScalar(rnd(200, 1300))).add(sys.field); s.scale.setScalar(rnd(1400, 2600)); g.add(s); }
  // vaheta vana süsteem uue vastu
  if (sysG) { scene.remove(sysG); sysG.traverse(o => { if (o.isInstancedMesh) { o.geometry.dispose(); o.material = null; } }); disposeTree(sysG); }
  sysG = g; scene.add(g); SYSINFO = sys; PL.length = 0; PL.push(...newPL);
  FIELD_POS.copy(sys.field); for (const a of AST) placeRock(a, true);
  applySun(sys.sun); skyStars.rotation.set(sys.skyRot * 0.3, sys.skyRot, 0);
  buildDest();
  if (holoDyn) rebuildHolo();
  for (const n of NPCS) if (n.mode === 'travel') { n.g.position.copy(randUnit(_v).multiplyScalar(rnd(700, 1600))).add(STATION_POS); pickNPCTarget(n); }
  for (const c of CRYS) { c.life = 0; c.m.visible = false; }
}

/* =========================================================
   MÄNGIJA LAEV
   ========================================================= */
const ship = new THREE.Group(); scene.add(ship);
ship.position.set(0, -190, 900);
const rig = new THREE.Group(); ship.add(rig); rig.add(camera);
const shipFX = { glows: [], stick: null, lever: null, reticle: null, tank: null, door: null, gear: null, tunnel: null };
const PM = {};
const LAND_H = 4.55;
function buildPlayerShip() {
  Object.assign(PM, shipMats('#d6d3cc', '#c25a1c', '#3a3c40', 909));
  PM.window = new THREE.MeshStandardMaterial({ color: '#9fd8ff', transparent: true, opacity: 0.1, roughness: 0.05, metalness: 0.3, depthWrite: false });
  const b = new Builder(), r = mulberry(5);
  b.add('hull', tBox(8.4, 2, 18.6), 0, -1.05, 1.9);
  b.add('hull', tBox(7.4, 1, 16.6), 0, 3.6, 2);
  b.mirror('hull', () => tBox(1.1, 1.1, 16.6), 3.65, 0.5, 2);
  b.mirror('hull', () => tBox(1.1, 0.9, 16.6), 3.65, 2.7, 2);
  for (const z of [-5.9, -4, -2, 0, 2, 4, 6, 8, 9.9]) b.mirror('dark', () => tBox(0.5, 1.3, 0.3), 3.4, 1.65, z);
  // tagasein uksega
  b.mirror('hull', () => tBox(3.35, 4.2, 0.8), 2.525, 1.05, 10.6);
  b.add('hull', tBox(1.7, 0.8, 0.8), 0, 2.75, 10.6); b.add('hull', tBox(1.7, 1.05, 0.8), 0, -0.525, 10.6);
  b.add('hull', taper(tBox(7.4, 1.4, 5.5), 0.6, 0.5), 0, 0.15, -8.8);
  b.add('dark', tBox(7.4, 0.35, 0.5), 0, 3.15, -6.2);
  b.mirror('dark', () => tBox(0.25, 2.3, 0.4), 1.55, 1.95, -6.2);
  b.mirror('dark', () => tBox(0.5, 2.3, 0.5), 3.25, 1.95, -6.2);
  b.mirror('accent', () => tBox(4, 2.6, 14), 6.3, -0.6, 3);
  b.mirror('dark', () => tBox(2, 1.2, 9), 4.6, -0.5, 2);
  b.mirror('hull', () => tBox(7, 0.5, 9), 11, -0.8, 4, 0, 0, -0.08);
  b.mirror('accent', () => tBox(1.5, 1.2, 5), 14.3, -1.1, 5.5);
  for (const x of [-2.2, 0, 2.2]) { b.add('accent', cylZ(0.9, 7, 14), x, 4.9, 4); for (const z of [1, 7]) b.add('dark', cylZ(1.02, 0.35, 14), x, 4.9, z); }
  for (let k = 0; k < 18; k++) { const w = 0.4 + r() * 1.2, h = 0.2 + r() * 0.6, d = 0.4 + r() * 1.6; b.add(r() < 0.6 ? 'dark' : 'hull', tBox(w, h, d), (r() - 0.5) * 6.5, 4.1 + h / 2, -5 + r() * 15); }
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
  // uks (kaob, kui maandud)
  const door = new THREE.Mesh(tBox(1.72, 2.36, 1.0, 2), PM.dark); door.position.set(0, 1.18, 10.5); ship.add(door); shipFX.door = door;
  // maandumisjalad ja kaldtee
  const gear = new THREE.Group(); gear.visible = false; ship.add(gear);
  for (const [x, z] of [[-3.4, -5], [3.4, -5], [-3.4, 7], [3.4, 7]]) { const l = new THREE.Mesh(cylY(0.25, 2.6, 10), PM.dark); l.position.set(x, -3.3, z); gear.add(l); const f = new THREE.Mesh(cylY(0.7, 0.2, 14), PM.dark); f.position.set(x, -4.5, z); gear.add(f); }
  const rampM = new THREE.Mesh(tBox(1.7, 0.15, 7.9, 2), new THREE.MeshStandardMaterial({ map: floorTex(), roughness: 0.6, metalness: 0.5 }));
  rampM.position.set(0, -LAND_H / 2, 14.25); rampM.rotation.x = Math.atan2(LAND_H, 6.5); rampM.visible = false; ship.add(rampM);
  shipFX.gear = gear; shipFX.ramp = rampM;
  // atmosfääri kuumus nina ees
  const flame = new THREE.Group(); flame.visible = false; ship.add(flame); shipFX.flame = [];
  for (const [x, y, z, s, c] of [[0, 0, -16, 18, '#ff7a2a'], [0, -0.5, -14, 10, '#ffd090'], [3.5, -0.5, -10, 9, '#ff5a1a'], [-3.5, -0.5, -10, 9, '#ff5a1a'], [0, 1.5, -18, 26, '#ff3a10']]) {
    const g = glowSprite(c, s, 0); g.position.set(x, y, z); g.userData.base = s; flame.add(g); shipFX.flame.push(g);
  }
  shipFX.flameG = flame;

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
  const ib = new Builder();
  ib.add('floor', tBox(6, 0.1, 16.2, 2), 0, -0.05, 2);
  ib.add('ceil', tBox(6, 0.1, 16.2, 2.5), 0, 3.05, 2);
  for (const s of [-1, 1]) {
    ib.add('wall', tBox(0.1, 1.05, 16.2, 2.5), s * 3, 0.525, 2);
    ib.add('wall', tBox(0.1, 0.8, 16.2, 2.5), s * 3, 2.65, 2);
    for (const z of [-6, -4, -2, 0, 2, 4, 6, 8, 10]) ib.add('trim', tBox(0.16, 1.25, 0.2), s * 2.98, 1.65, z);
    ib.add('trim', tBox(0.25, 0.08, 16.2), s * 2.9, 1.06, 2);
    ib.add('strip', tBox(0.05, 0.05, 16), s * 2.9, 2.96, 2);
    ib.add('stripO', tBox(0.04, 0.04, 16), s * 2.93, 0.06, 2);
    ib.add('wall', tBox(2.15, 3.1, 0.1), s * 1.925, 1.5, 10.05);
    ib.add('stripO', tBox(0.06, 2.4, 0.14), s * 0.88, 1.2, 10.0);
  }
  ib.add('wall', tBox(1.7, 0.75, 0.1), 0, 2.72, 10.05); ib.add('stripO', tBox(1.8, 0.06, 0.14), 0, 2.38, 10.0);
  ib.add('wall', tBox(6, 0.9, 0.1), 0, 0.45, -6.05);
  for (const zp of [-2, 4]) {
    for (const s of [-1, 1]) { ib.add('wall', tBox(2.1, 3.0, 0.15, 2.5), s * 1.95, 1.5, zp); ib.add('strip', tBox(0.06, 2.4, 0.18), s * 0.92, 1.2, zp); }
    ib.add('wall', tBox(1.8, 0.6, 0.15), 0, 2.7, zp); ib.add('strip', tBox(1.9, 0.06, 0.18), 0, 2.4, zp);
  }
  ib.add('trim', tBox(5.6, 0.85, 1.0), 0, 0.425, -5.5);
  ib.add('stripO', tBox(5.4, 0.03, 0.03), 0, 0.86, -4.99);
  ib.add('trim', tBox(0.5, 0.4, 0.5), 0, 0.2, -3.15);
  ib.add('seat', tBox(0.72, 0.15, 0.7), 0, 0.48, -3.15);
  ib.add('seat', tBox(0.72, 1.0, 0.15), 0, 1.02, -2.76, 0.12);
  ib.add('seat', tBox(0.45, 0.3, 0.12), 0, 1.66, -2.7, 0.12);
  ib.mirror('trim', () => tBox(0.12, 0.12, 0.6), 0.42, 0.72, -3.15);
  ib.mirror('trim', () => tBox(0.1, 0.3, 0.1), 0.42, 0.6, -3.15);
  ib.mirror('trim', () => tBox(0.25, 0.7, 0.35), 0.62, 0.35, -3.7);
  ib.add('trim', cylY(0.55, 0.9, 28), 0, 0.45, 1); ib.add('trim', cylY(0.6, 0.05, 28), 0, 0.92, 1); ib.add('strip', new THREE.TorusGeometry(0.6, 0.015, 6, 40), 0, 0.945, 1, PI / 2, 0, 0);
  ib.add('accent', tBox(0.6, 0.45, 2.4), -2.6, 0.225, 2.2); ib.add('seat', tBox(0.15, 0.6, 2.4), -2.88, 0.75, 2.2);
  ib.add('trim', cylY(0.22, 0.4, 12), 2.6, 0.2, 2.8); ib.add('trim', cylY(0.24, 0.04, 12), 2.6, 0.41, 2.8);
  ib.add('accent', tBox(1.0, 0.9, 0.9, 1.2), -2.35, 0.45, 5.4); ib.add('accent', tBox(0.7, 0.63, 0.63, 1.2), -2.35, 1.215, 5.4);
  ib.add('accent', tBox(0.8, 0.72, 0.72, 1.2), -2.35, 0.36, 6.3); ib.add('accent', tBox(0.9, 0.81, 0.81, 1.2), 2.35, 0.405, 5.5);
  ib.add('trim', tBox(1.2, 0.7, 1.0, 1.2), 2.35, 0.35, 6.3);
  ib.add('trim', tBox(1.3, 1.9, 1.5), -2.25, 0.95, 8); ib.add('stripO', cylY(0.16, 1.6, 12), -2.25, 0.95, 7.2); ib.add('ceil', tBox(1.3, 0.2, 1.5), -2.25, 2.0, 8);
  ib.add('trim', tBox(1.3, 1.9, 1.5), 2.25, 0.95, 8); ib.add('strip', tBox(0.04, 1.6, 1.3), 1.58, 0.95, 8);
  ib.add('trim', cylY(0.45, 0.25, 20), -2.35, 0.125, 9.45); ib.add('trim', cylY(0.45, 0.2, 20), -2.35, 2.0, 9.45); ib.add('strip', cylY(0.47, 0.03, 20), -2.35, 0.26, 9.45);
  ship.add(ib.build(IM));
  const tankGlass = new THREE.Mesh(cylY(0.42, 1.65, 24), new THREE.MeshStandardMaterial({ color: '#9fe8ff', transparent: true, opacity: 0.15, roughness: 0.05, depthWrite: false }));
  tankGlass.position.set(-2.35, 1.07, 9.45); ship.add(tankGlass);
  const tankIM = new THREE.InstancedMesh(new THREE.OctahedronGeometry(0.07, 0).scale(1, 1.6, 1), new THREE.MeshBasicMaterial({ color: new THREE.Color(0.3, 1.4, 1.8), toneMapped: false }), 40);
  const tr = mulberry(3); for (let k = 0; k < 40; k++) tankIM.setMatrixAt(k, _m.compose(_v.set(-2.35 + (tr() - 0.5) * 0.55, 0.33 + Math.floor(k / 8) * 0.09 + tr() * 0.05, 9.45 + (tr() - 0.5) * 0.55), _q.setFromEuler(_e.set(tr() * 3, tr() * 3, tr() * 3)), _v2.set(1, 1, 1)));
  tankIM.count = 0; ship.add(tankIM); shipFX.tank = tankIM;
  const leafMat = new THREE.MeshStandardMaterial({ color: '#3fa04a', roughness: 0.8 });
  for (let k = 0; k < 7; k++) { const lf = new THREE.Mesh(new THREE.ConeGeometry(0.06, 0.55, 6), leafMat); lf.position.set(2.6 + Math.cos(k) * 0.08, 0.68, 2.8 + Math.sin(k) * 0.08); lf.rotation.set(Math.sin(k * 2) * 0.5, 0, Math.cos(k * 2) * 0.5); ship.add(lf); }
  const knob = new THREE.MeshStandardMaterial({ color: '#c25a1c', roughness: 0.5 });
  const mkStick = (x) => { const g = new THREE.Group(); g.position.set(x, 0.72, -3.7); const s = new THREE.Mesh(cylY(0.025, 0.28, 8), IM.trim); s.position.y = 0.14; const k = new THREE.Mesh(new THREE.SphereGeometry(0.05, 12, 8), knob); k.position.y = 0.3; g.add(s, k); ship.add(g); return g; };
  shipFX.stick = mkStick(0.62); shipFX.lever = mkStick(-0.62);
  const l1 = new THREE.PointLight('#bfe8ff', 9, 9, 1.6); l1.position.set(0, 2.6, -3.2); ship.add(l1);
  const l2 = new THREE.PointLight('#ffe0c0', 9, 10, 1.6); l2.position.set(0, 2.6, 5.5); ship.add(l2);
  const ret = new THREE.Mesh(new THREE.PlaneGeometry(14, 14), new THREE.MeshBasicMaterial({ map: RING, color: new THREE.Color(0.4, 1.6, 1.2), transparent: true, blending: THREE.AdditiveBlending, depthWrite: false, toneMapped: false }));
  ret.position.set(0, 0.6, -300); ship.add(ret); shipFX.reticle = ret;
  ship.add(sign('KOKPIT', 0, 2.72, -1.91, 0)); ship.add(sign('LASTIRUUM', 0, 2.72, 3.91, PI));
  ship.add(sign('ELURUUM', 0, 2.72, 4.09, 0)); ship.add(sign('ELURUUM', 0, 2.72, -2.09, PI)); ship.add(sign('VÄLJAPÄÄS', 0, 2.7, 9.98, PI));
  // hüpertunnel
  const tc = cnv(256, 1024), tg = tc.getContext('2d'); tg.fillStyle = '#000'; tg.fillRect(0, 0, 256, 1024);
  for (let k = 0; k < 260; k++) { const x = Math.random() * 256, y = Math.random() * 1024, l = 60 + Math.random() * 260; const gr = tg.createLinearGradient(x, y, x, y + l); const c = ['#60e0ff', '#b080ff', '#ffffff', '#ff70d0'][k % 4]; gr.addColorStop(0, 'rgba(0,0,0,0)'); gr.addColorStop(0.5, c); gr.addColorStop(1, 'rgba(0,0,0,0)'); tg.fillStyle = gr; tg.fillRect(x, y, 2 + Math.random() * 3, l); }
  const tt = ctex(tc); tt.wrapS = tt.wrapT = THREE.RepeatWrapping; tt.repeat.set(5, 2);
  const tun = new THREE.Group(); tun.visible = false; ship.add(tun);
  const tDark = new THREE.Mesh(cylZ(34, 420, 32), new THREE.MeshBasicMaterial({ color: '#04020c', side: THREE.BackSide, transparent: true, opacity: 0 })); tun.add(tDark);
  const tStr = new THREE.Mesh(cylZ(30, 420, 32), new THREE.MeshBasicMaterial({ map: tt, side: THREE.BackSide, transparent: true, opacity: 0, blending: THREE.AdditiveBlending, depthWrite: false, toneMapped: false })); tun.add(tStr);
  const tEnd = glowSprite('#e8f4ff', 90, 0); tEnd.position.set(0, 0, -200); tun.add(tEnd);
  shipFX.tunnel = { g: tun, dark: tDark, str: tStr, end: tEnd, tex: tt };
}
function sign(text, x, y, z, ry) {
  const c = cnv(512, 96), g = c.getContext('2d');
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
  DEST.length = 0;
  DEST.push({ name: S.system === 0 ? 'Jaam Tähesadam' : 'Jaam ' + SYSINFO.name.split(' ')[0], type: 'station', pos: STATION_POS.clone(), stop: 560 });
  DEST.push({ name: 'Asteroidiväli', type: 'field', pos: FIELD_POS.clone(), stop: 1150 });
  for (const p of PL) DEST.push({ name: p.name, type: 'planet', pos: p.pos.clone(), stop: p.r * 1.7 + 250, planet: p });
  if (S.target >= DEST.length) S.target = -1;
}
const fmtDist = d => d >= 10000 ? Math.round(d / 1000) + ' km' : d >= 1000 ? (d / 1000).toFixed(1) + ' km' : Math.round(d) + ' m';
const discKey = p => S.system + ':' + p.name;

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
      if (_v3.distanceToSquared(a.pos) < a.r * a.r * 0.8) { b.life = 0; b.m.visible = false; a.hp -= stat.dmg; flash(_v3, a.r * 0.6, '#ffd090', 0.25); sfx.hit(); if (a.hp <= 0) breakRock(a); break; }
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
    if (dist < mr && !R.landed) { d.normalize(); const want = Math.abs(R.speed) + 160; c.vel.lerp(d.multiplyScalar(want), 1 - Math.exp(-dt * 3)); }
    else c.vel.multiplyScalar(Math.exp(-dt * 0.5));
    c.m.position.addScaledVector(c.vel, dt); c.m.rotation.y += dt * 2; c.m.rotation.x += dt * 0.7;
    if (dist < 16 && !R.landed) { c.life = 0; c.m.visible = false; S.crystals += c.value; S.mined += c.value; sfx.pick(); haptic('left', 0.3, 40); }
  }
  for (const d of DEBRIS) { if (d.life <= 0) continue; d.life -= dt; if (d.life <= 0) { d.m.visible = false; continue; } d.m.position.addScaledVector(d.vel, dt); d.m.rotation.x += d.spin.x * dt; d.m.rotation.y += d.spin.y * dt; d.m.scale.setScalar(d.s * Math.min(1, d.life)); }
  for (const f of FLASH) { if (f.life <= 0) continue; f.life -= dt; if (f.life <= 0) { f.s.visible = false; continue; } const t = 1 - f.life / f.max; f.s.scale.setScalar(f.size * (0.5 + t)); f.s.material.opacity = 1 - t; }
}

/* =========================================================
   KOSMOSETOLM
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
    for (const ax of ['x', 'y', 'z']) { const d = p[ax] - P[ax]; if (d > DB) p[ax] -= 2 * DB; else if (d < -DB) p[ax] += 2 * DB; }
    a[k * 6] = p.x; a[k * 6 + 1] = p.y; a[k * 6 + 2] = p.z;
    a[k * 6 + 3] = p.x - dir.x * len; a[k * 6 + 4] = p.y - dir.y * len; a[k * 6 + 5] = p.z - dir.z * len;
  }
  dust.geometry.attributes.position.needsUpdate = true;
  dust.material.opacity = 0.25 + clamp(sp / 600, 0, 0.45);
}

/* =========================================================
   HELI JA MUUSIKA
   ========================================================= */
let AC = null, master, engG, engF, musicBus;
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
  musicBus = AC.createGain(); musicBus.gain.value = S.music ? 1 : 0; musicBus.connect(master);
  const dl = AC.createDelay(); dl.delayTime.value = 0.42; const fb = AC.createGain(); fb.gain.value = 0.38; musicBus.connect(dl); dl.connect(fb).connect(dl); dl.connect(master);
}
function tone(f1, f2, dur, type = 'sine', vol = 0.2, delay = 0, bus = null) {
  if (!AC) return; const t = AC.currentTime + delay; const o = AC.createOscillator(), g = AC.createGain(); o.type = type;
  o.frequency.setValueAtTime(f1, t); o.frequency.exponentialRampToValueAtTime(Math.max(f2, 1), t + dur);
  g.gain.setValueAtTime(vol, t); g.gain.exponentialRampToValueAtTime(0.0001, t + dur); o.connect(g).connect(bus || master); o.start(t); o.stop(t + dur + 0.02);
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
  hyper: () => { tone(80, 1200, 3.5, 'sawtooth', 0.08); noiseBurst(3.5, 0.25, 3000); },
  chirp: () => { const f = 900 + Math.random() * 900; tone(f, f * 1.5, 0.12, 'sine', 0.08); tone(f * 1.2, f * 0.8, 0.15, 'sine', 0.07, 0.13); },
  scan: () => tone(1800, 1800, 0.03, 'sine', 0.03),
};
const CHORDS = { space: [[220, 261.6, 329.6], [174.6, 220, 261.6], [196, 246.9, 293.7], [164.8, 207.7, 246.9]], ground: [[261.6, 329.6, 392], [220, 277.2, 329.6], [293.7, 349.2, 440], [246.9, 311.1, 370]] };
const mus = { t: 1, i: 0 };
function pad(f, dur) {
  const t = AC.currentTime; const g = AC.createGain(); g.gain.setValueAtTime(0.0001, t); g.gain.linearRampToValueAtTime(0.03, t + 2.5); g.gain.linearRampToValueAtTime(0.0001, t + dur);
  const lp = AC.createBiquadFilter(); lp.type = 'lowpass'; lp.frequency.value = 900; lp.connect(g).connect(musicBus);
  for (const [type, mul] of [['triangle', 1], ['sine', 1.004]]) { const o = AC.createOscillator(); o.type = type; o.frequency.value = f * mul; o.connect(lp); o.start(t); o.stop(t + dur + 0.1); }
}
function updateMusic(dt) {
  if (!AC || !musicBus) return; musicBus.gain.setTargetAtTime(S.music ? 1 : 0, AC.currentTime, 0.5);
  if (!S.music) return; mus.t -= dt; if (mus.t > 0) return; mus.t = 7;
  const ch = CHORDS[R.landed ? 'ground' : 'space'][mus.i++ % 4];
  ch.forEach(f => pad(f / (R.landed ? 1 : 2), 7.5));
  const n = R.landed ? 4 : 2; for (let k = 0; k < n; k++) tone(ch[(Math.random() * 3) | 0] * 2, ch[0] * 2, 1.4, 'sine', 0.025, 1 + k * 1.3 + Math.random(), musicBus);
}
function updateAudio() {
  if (!AC || !engG) return; const f = clamp(Math.abs(R.speed) / stat.pulse, 0, 1);
  engG.gain.setTargetAtTime(onFoot() ? 0.02 : 0.06 + 0.3 * f + (R.mode === 'outside' ? 0.05 : 0), AC.currentTime, 0.2);
  engF.frequency.setTargetAtTime(140 + Math.abs(R.speed) * 1.1, AC.currentTime, 0.2);
}
function haptic(hand, v, ms) { const s = renderer.xr.getSession(); if (!s) return; for (const src of s.inputSources) if (src.handedness === hand) src.gamepad?.hapticActuators?.[0]?.pulse?.(v, ms); }

/* =========================================================
   PANEELID
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
  click(uv) { const i = this.indexAt(uv); if (i < 0) return; const b = this.buttons[i]; if (b.dis) { sfx.deny(); return; } sfx.click(); b.fn(); for (const p of PANELS) p.last = ''; }
}
function place(p, x, y, z, rx = 0, ry = 0) { p.mesh.position.set(x, y, z); p.mesh.rotation.set(rx, ry, 0, 'YXZ'); ship.add(p.mesh); return p; }
const UPG = [['laser', 'Laser', 'kivid purunevad kiiremini'], ['engine', 'Mootor', 'suurem kiirus'], ['tank', 'Kütusepaak', '+50 kütust'], ['shield', 'Kilp', '+50 kilpi'], ['magnet', 'Magnet', 'tõmbab kristalle kaugemalt']];
const UPG_BASE = { laser: 60, engine: 80, tank: 60, shield: 60, magnet: 50 };
let msgPanel, wristPanel, fader;
function buildPanels() {
  place(new Panel({ w: 1.05, h: 0.7, W: 1024, H: 680, sig: () => [Math.round(R.speed / 2), Math.round(S.fuel), Math.round(S.shield), S.crystals, S.credits, R.autopilot, R.mode, S.up.tank, S.up.shield, !!R.landed].join('|'),
    draw: (g, P) => {
      panelBg(g, P.W, P.H, 'LAEVA OLEK'); let y = 138;
      const row = (k, v, c = '#f2fbff') => { txt(g, k, 40, y, 36, '#8fd8ea'); txt(g, String(v), P.W - 40, y, 42, c, 'right', true); y += 66; };
      row('Kiirus', Math.round(Math.abs(R.speed)) + ' m/s', Math.abs(R.speed) > stat.manual + 10 ? '#d0a8ff' : '#f2fbff');
      bar(g, 'Kütus', S.fuel / stat.maxFuel, S.fuel / stat.maxFuel < 0.15 ? '#ff5040' : '#ff9a30', y, P.W); y += 70;
      bar(g, 'Kilp', S.shield / stat.maxShield, '#2ab8ff', y, P.W); y += 74;
      row('Kristallid', S.crystals, '#7ff0ff'); row('Krediidid', S.credits + ' kr', '#ffd36a');
      row('Režiim', R.landed ? 'maandunud' : R.autopilot ? 'AUTOPILOOT' : R.mode === 'walk' ? 'hõljub' : 'käsitsi', R.autopilot ? '#7dff9a' : '#f2fbff');
    } }), -1.34, 1.38, -4.86, -0.3, 0.4);
  place(new Panel({ w: 1.3, h: 0.89, W: 1024, H: 700, sig: () => [S.target, R.autopilot, S.discovered.length, S.system, ...DEST.map(d => fmtDist(ship.position.distanceTo(d.pos)))].join('|'),
    draw: (g, P) => {
      panelBg(g, P.W, P.H, 'SIHTKOHAD');
      txt(g, SYSINFO ? SYSINFO.name : '', P.W - 40, 54, 32, '#9fc8d6', 'right');
      DEST.forEach((d, i) => { if (i > 7) return; const x = 36 + (i % 2) * 484, y = 108 + ((i / 2) | 0) * 112; const disc = d.type === 'planet' && S.discovered.includes(discKey(d.planet));
        P.btn(x, y, 468, 98, d.name + (disc ? ' ✓' : ''), () => setTarget(i), { on: S.target === i, fs: 36, sub: R.landed ? TYPE_NAME[d.type] : fmtDist(ship.position.distanceTo(d.pos)) + ' · ' + TYPE_NAME[d.type] }); });
      P.btn(36, 566, 952, 106, R.autopilot ? 'AUTOPILOOT SEES' : 'AUTOPILOOT SISSE', toggleAutopilot, { on: R.autopilot, col: '#7dff9a', fs: 44, dis: (S.target < 0 && !R.autopilot) || !!R.landed, sub: R.autopilot ? 'vajuta, et välja lülitada' : S.target < 0 ? 'vali enne sihtkoht' : 'lendab ise – võid ringi kõndida (A)' });
    } }), 0, 1.45, -4.98, -0.3, 0);
  place(new Panel({ w: 1.05, h: 0.7, W: 1024, H: 680, sig: () => [R.nearStation, R.nearPlanet?.name, R.landed?.name, R.busy, S.system, S.crystals, S.credits, Math.round(S.fuel), Math.round(S.shield), Math.round(R.throttle * 20), R.mode].join('|'),
    draw: (g, P) => {
      panelBg(g, P.W, P.H, 'JUHTIMINE');
      P.btn(36, 104, 464, 100, 'TÕUSE PÜSTI', () => toggleSeat(), { sub: 'kõnni laevas (B)', fs: 38 });
      P.btn(524, 104, 464, 100, 'VÄLISVAADE', () => toggleOutside(), { sub: 'vaata laeva (X)', fs: 38, dis: !!R.landed });
      P.btn(36, 216, 300, 104, 'PEATU', () => { R.throttle = 0; R.autopilot = false; say('Mootorid seisma.'); }, { col: '#ff8a5a', fs: 38, dis: !!R.landed });
      P.btn(362, 216, 300, 104, 'HÜPE EDASI', () => startHyper(S.system + 1), { col: '#c9a2ff', fs: 36, sub: S.easy ? 'tasuta' : '40 kütust', dis: !!R.landed || !!R.busy });
      P.btn(688, 216, 300, 104, 'HÜPE KOJU', () => startHyper(0), { col: '#c9a2ff', fs: 36, sub: 'kodusüsteemi', dis: S.system === 0 || !!R.landed || !!R.busy });
      if (R.landed) P.btn(36, 350, 952, 190, 'STARDI', takeoff, { sub: `lahku planeedilt ${R.landed.name} (või randmelt)`, col: '#7dff9a', fs: 60 });
      else if (R.nearStation) {
        txt(g, 'Jaam on lähedal', 512, 352, 34, '#ffd36a', 'center', true);
        P.btn(36, 380, 300, 150, 'MÜÜ', sell, { sub: `${S.crystals} × 10 kr`, dis: S.crystals <= 0, col: '#ffd36a', fs: 44 });
        P.btn(362, 380, 300, 150, 'TANGI', buyFuel, { sub: '30 kr', dis: S.credits < 30 || S.fuel >= stat.maxFuel - 0.5, col: '#ffa040', fs: 44 });
        P.btn(688, 380, 300, 150, 'PARANDA', repair, { sub: 'kilp tasuta', dis: S.shield >= stat.maxShield - 0.5, fs: 44 });
      } else if (R.nearPlanet) {
        const gas = R.nearPlanet.kind === 'gas';
        P.btn(36, 350, 952, 190, gas ? 'MAANDUDA EI SAA' : 'MAANDU', startLanding, { sub: gas ? 'gaasihiiglasel pole pinda' : `planeedile ${R.nearPlanet.name}`, dis: gas || !!R.busy, col: '#7dff9a', fs: 60 });
      } else { txt(g, 'Lenda planeedi juurde, et maanduda,', 512, 420, 34, '#7fa8b8', 'center'); txt(g, 'või jaama juurde, et kaubelda.', 512, 466, 34, '#7fa8b8', 'center'); }
      txt(g, 'Hoob', 40, 610, 34, '#8fd8ea'); rr(g, 170, 592, 818, 36, 10); g.fillStyle = 'rgba(255,255,255,0.08)'; g.fill();
      const z = 170 + 818 * 0.23; g.fillStyle = '#ffffff55'; g.fillRect(z - 1, 586, 3, 48);
      const tw = 818 * 0.77 * R.throttle; g.fillStyle = R.throttle >= 0 ? '#7dff9a' : '#ff8a5a'; g.fillRect(Math.min(z, z + tw), 596, Math.abs(tw), 28);
    } }), 1.34, 1.38, -4.86, -0.3, -0.4);
  place(new Panel({ w: 0.62, h: 0.24, W: 640, H: 248, sig: () => R.mode, draw: (g, P) => { P.btn(10, 10, 620, 228, R.mode === 'seat' ? 'ISTUD ROOLIS' : 'ISTU ROOLI', () => { if (R.mode !== 'seat') toggleSeat(); }, { fs: 64, on: R.mode === 'seat', sub: 'või vajuta B' }); } }), 0, 1.35, -2.66, 0, 0);
  place(new Panel({ w: 1.6, h: 1.25, W: 1024, H: 800, interactive: false, draw: (g, P) => {
    panelBg(g, P.W, P.H, 'KUIDAS MÄNGIDA', '#7dff9a'); const L = [
      ['1.', 'Vali sihtkoht ja lülita autopiloot sisse (ka randmelt).'], ['2.', 'Autopiloodi ajal võid püsti tõusta ja kõndida.'],
      ['3.', 'Asteroidiväljal tulista kive – need annavad kristalle.'], ['4.', 'Planeedi juures vajuta MAANDU ja mine välja!'],
      ['5.', 'Pinnal skaneeri loomi ja kaeva kristalle.'], ['6.', 'Kosmoses peatu ja kõnni tagauksest välja.'],
      ['7.', 'HÜPE viib uude päikesesüsteemi.'], ['', ''],
      ['Rand:', 'vasaku käe randmel on menüü'], ['Nupud:', 'A autopiloot, B istu/sisene, X vaade, Y siht']];
    let y = 140; for (const [a, b] of L) { txt(g, a, 44, y, 34, '#7dff9a', 'left', true); txt(g, b, 170, y, 34, '#e8f8ff'); y += 62; }
  } }), -1.95, 1.55, 3.9, 0, PI);
  place(new Panel({ w: 1.6, h: 1.25, W: 1024, H: 800, sig: () => [S.discovered.length, S.mined, S.credits, R.resetArm > 0, S.system, S.scanned.length, S.monos.length, S.music, S.easy].join('|'), draw: (g, P) => {
    panelBg(g, P.W, P.H, 'KAPTENI LOGI', '#c9a2ff'); let y = 130;
    txt(g, `${SYSINFO ? SYSINFO.name : ''} · süsteem ${S.system + 1}`, 44, y, 38, '#ffffff', 'left', true); y += 58;
    PL.forEach((p, k) => { const d = S.discovered.includes(discKey(p)); txt(g, (d ? '✓ ' : '? ') + (d ? p.name : 'tundmatu'), 70 + (k % 2) * 460, y + ((k / 2) | 0) * 52, 34, d ? '#7dff9a' : '#7a8a96'); }); y += 3 * 52 + 14;
    txt(g, `Liike skaneeritud: ${S.scanned.length}   Monoliite: ${S.monos.length}`, 44, y, 34, '#ffd36a'); y += 50;
    txt(g, `Kaevandatud kristalle: ${S.mined}   Krediite: ${S.credits} kr`, 44, y, 34, '#7ff0ff');
    P.btn(44, P.H - 150, 290, 110, 'MUUSIKA', () => { S.music = !S.music; initAudio(); save(); }, { on: S.music, sub: S.music ? 'sees' : 'väljas', col: '#7dff9a', fs: 38 });
    P.btn(367, P.H - 150, 290, 110, 'LIHTNE', () => { S.easy = !S.easy; save(); say(S.easy ? 'Lihtne režiim: pole kahju ega kütusekulu.' : 'Tavaline režiim.'); }, { on: S.easy, sub: S.easy ? 'sees' : 'väljas', col: '#7dff9a', fs: 38 });
    P.btn(690, P.H - 150, 290, 110, R.resetArm > 0 ? 'KINDEL?' : 'UUESTI', resetGame, { col: '#ff6a6a', sub: R.resetArm > 0 ? 'vajuta veel' : 'alusta algusest', fs: 38 });
  } }), 1.95, 1.55, 3.9, 0, PI);
  place(new Panel({ w: 0.9, h: 0.76, W: 900, H: 760, sig: () => [S.crystals, Math.round(S.fuel), S.up.tank].join('|'), draw: (g, P) => {
    panelBg(g, P.W, P.H, 'RAFINEERIJA', '#ffa040'); txt(g, 'Teeb kristallidest kütust', 40, 140, 34, '#ffd9b0');
    txt(g, `Kristalle: ${S.crystals}`, 40, 210, 46, '#7ff0ff', 'left', true); bar(g, 'Kütus', S.fuel / stat.maxFuel, '#ff9a30', 290, P.W);
    const full = S.fuel >= stat.maxFuel - 0.5;
    P.btn(40, 360, P.W - 80, 150, 'RAFINEERI', () => refine(1), { sub: '3 kristalli → +20 kütust', dis: S.crystals < 3 || full, col: '#ffa040', fs: 50 });
    P.btn(40, 540, P.W - 80, 150, 'RAFINEERI KÕIK', () => refine(999), { sub: 'kuni paak on täis', dis: S.crystals < 3 || full, col: '#ffa040', fs: 46 });
  } }), -1.585, 1.3, 8, 0, PI / 2);
  place(new Panel({ w: 0.9, h: 0.96, W: 900, H: 960, sig: () => [S.credits, ...Object.values(S.up)].join('|'), draw: (g, P) => {
    panelBg(g, P.W, P.H, 'UUENDUSED', '#7dff9a'); txt(g, `${S.credits} kr`, P.W - 40, 54, 40, '#ffd36a', 'right', true);
    UPG.forEach(([k, n, d], i) => { const y = 112 + i * 166, lv = S.up[k], price = UPG_BASE[k] * (lv + 1);
      txt(g, n, 40, y + 42, 44, '#ffffff', 'left', true); txt(g, d, 40, y + 92, 28, '#9fc8d6');
      for (let j = 0; j < 3; j++) { rr(g, 40 + j * 46, y + 118, 36, 16, 6); g.fillStyle = j < lv ? '#7dff9a' : 'rgba(255,255,255,0.15)'; g.fill(); }
      P.btn(P.W - 320, y + 22, 280, 116, lv >= 3 ? 'MAX' : `${price} kr`, () => buyUp(k), { dis: lv >= 3 || S.credits < price, col: '#7dff9a', fs: 44 }); });
  } }), 1.585, 1.3, 8, 0, -PI / 2);
  // väljapääsu paneel tagaukse kõrval
  place(new Panel({ w: 0.8, h: 0.4, W: 800, H: 400, sig: () => [!!R.landed, Math.round(R.speed)].join('|'), draw: (g, P) => {
    panelBg(g, P.W, P.H, null, '#ffa040');
    P.btn(30, 30, P.W - 60, P.H - 60, 'VÄLJU LAEVAST', () => exitShip(), { sub: R.landed ? 'mine planeedile' : 'kosmosekõnd', col: '#ffa040', fs: 60 });
  } }), 1.72, 1.45, 9.98, 0, PI);
  // käerandme menüü (kinnitub vasaku kontrolleri külge)
  wristPanel = new Panel({ w: 0.25, h: 0.2, W: 600, H: 480, sig: () => [S.target, R.autopilot, !!R.landed, R.mode, R.nearPlanet?.name, R.busy, DEST.length].join('|'), draw: (g, P) => {
    rr(g, 4, 4, P.W - 8, P.H - 8, 24); g.fillStyle = 'rgba(4,18,32,0.92)'; g.fill(); g.lineWidth = 5; g.strokeStyle = '#2ad4ff'; g.stroke();
    const tn = S.target >= 0 && DEST[S.target] ? DEST[S.target].name : 'vali siht';
    P.btn(14, 14, 90, 92, '<', () => setTarget(((S.target < 0 ? 0 : S.target) - 1 + DEST.length) % DEST.length), { fs: 52 });
    P.btn(P.W - 104, 14, 90, 92, '>', cycleTarget, { fs: 52 });
    let fs = 38; g.font = `bold ${fs}px system-ui`; while (g.measureText(tn).width > 360 && fs > 20) { fs -= 2; g.font = `bold ${fs}px system-ui`; }
    txt(g, tn, P.W / 2, 60, fs, '#c8ffd8', 'center', true);
    const bw = 280, bh = 106, x1 = 14, x2 = P.W - 14 - bw;
    P.btn(x1, 120, bw, bh, 'AUTOPILOOT', toggleAutopilot, { on: R.autopilot, col: '#7dff9a', fs: 36, sub: R.autopilot ? 'sees' : 'väljas', dis: !!R.landed || onFoot() });
    P.btn(x2, 120, bw, bh, 'HÜPE', () => startHyper(S.system + 1), { col: '#c9a2ff', fs: 38, sub: 'uus süsteem', dis: !!R.landed || !!R.busy || onFoot() });
    P.btn(x1, 236, bw, bh, R.landed ? 'STARDI' : 'MAANDU', landToggle, { col: '#7dff9a', fs: 38, dis: !R.landed && (!R.nearPlanet || R.nearPlanet.kind === 'gas' || onFoot()) });
    P.btn(x2, 236, bw, bh, onFoot() ? 'SISENE' : 'VÄLJU', exitEnter, { col: '#ffa040', fs: 38, sub: onFoot() ? 'laeva' : R.landed ? 'planeedile' : 'kosmosesse' });
    P.btn(x1, 352, bw, bh, R.mode === 'seat' ? 'TÕUSE' : 'ISTU', toggleSeat, { fs: 38 });
    P.btn(x2, 352, bw, bh, 'VAADE', toggleOutside, { fs: 38, dis: !!R.landed || onFoot() });
  } });
  wristPanel.mesh.position.set(0, 0.075, 0.03); wristPanel.mesh.rotation.set(-0.75, 0, 0);
  msgPanel = new Panel({ w: 1.5, h: 0.21, W: 1024, H: 144, interactive: false, overlay: true, draw: (g, P) => {
    if (!R.msg) return; rr(g, 8, 8, P.W - 16, P.H - 16, 30); g.fillStyle = 'rgba(4,18,32,0.88)'; g.fill(); g.lineWidth = 4; g.strokeStyle = '#2ad4ff'; g.stroke();
    let fs = 46; g.font = `bold ${fs}px system-ui,sans-serif`; while (g.measureText(R.msg).width > P.W - 70 && fs > 24) { fs -= 2; g.font = `bold ${fs}px system-ui,sans-serif`; }
    txt(g, R.msg, P.W / 2, P.H / 2 + 2, fs, '#ecfcff', 'center', true);
  } });
  msgPanel.mesh.position.set(0, -0.34, -1.6); msgPanel.mesh.visible = false; camera.add(msgPanel.mesh);
  fader = new THREE.Mesh(new THREE.SphereGeometry(0.3, 16, 12), new THREE.MeshBasicMaterial({ color: '#000', side: THREE.BackSide, transparent: true, opacity: 0, depthTest: false, depthWrite: false, fog: false }));
  fader.renderOrder = 3000; fader.visible = false; camera.add(fader);
}

/* ---------- holokaart ---------- */
const HK = 0.5 / 16000; let holoInner, holoDyn = null, holoShip, holoLine, holoSel;
function buildHolo() {
  const holo = new THREE.Group(); holo.position.set(0, 1.5, 1); ship.add(holo);
  holoInner = new THREE.Group(); holo.add(holoInner);
  const beam = new THREE.Mesh(new THREE.CylinderGeometry(0.62, 0.55, 0.6, 32, 1, true), new THREE.MeshBasicMaterial({ color: new THREE.Color(0.2, 0.7, 1), transparent: true, opacity: 0.045, blending: THREE.AdditiveBlending, depthWrite: false, side: THREE.DoubleSide, toneMapped: false }));
  beam.position.set(0, 1.25, 1); ship.add(beam);
  const sun = glowSprite('#ffe0a0', 0.12); sun.position.copy(SUN_DIR).multiplyScalar(0.62); holoInner.add(sun);
  holoShip = new THREE.Mesh(new THREE.ConeGeometry(0.014, 0.045, 8).rotateX(-PI / 2), new THREE.MeshBasicMaterial({ color: new THREE.Color(0.4, 2, 0.6), toneMapped: false })); holoInner.add(holoShip);
  holoLine = new THREE.Line(new THREE.BufferGeometry().setFromPoints([new V3(), new V3()]), new THREE.LineBasicMaterial({ color: 0x7dff9a, transparent: true, opacity: 0.8, toneMapped: false })); holoInner.add(holoLine);
  holoSel = new THREE.Sprite(new THREE.SpriteMaterial({ map: RING, color: new THREE.Color(0.5, 2, 0.8), transparent: true, depthWrite: false, toneMapped: false })); holoSel.scale.setScalar(0.09); holoSel.visible = false; holoInner.add(holoSel);
  rebuildHolo();
}
function rebuildHolo() {
  if (holoDyn) { holoInner.remove(holoDyn); for (let i = UIT.length - 1; i >= 0; i--) if (UIT[i].userData.dest !== undefined) UIT.splice(i, 1); disposeTree(holoDyn, true); }
  holoDyn = new THREE.Group(); holoInner.add(holoDyn);
  DEST.forEach((d, i) => {
    const p = d.pos.clone().multiplyScalar(HK); let vis;
    if (d.type === 'station') vis = new THREE.Mesh(new THREE.TorusGeometry(0.022, 0.006, 6, 20), new THREE.MeshBasicMaterial({ color: new THREE.Color(0.5, 1.4, 1.8), toneMapped: false }));
    else if (d.type === 'field') { const g = new THREE.BufferGeometry().setFromPoints([...Array(40)].map(() => randUnit(new V3()).multiplyScalar(rnd(0, 0.035)))); vis = new THREE.Points(g, new THREE.PointsMaterial({ color: '#d8c0a0', size: 0.006, toneMapped: false })); }
    else vis = new THREE.Mesh(new THREE.SphereGeometry(0.012 + d.planet.r / 2000 * 0.03, 16, 12), new THREE.MeshBasicMaterial({ map: d.planet.mesh.material.map, toneMapped: false }));
    vis.position.copy(p); holoDyn.add(vis);
    const hit = new THREE.Mesh(new THREE.SphereGeometry(0.06, 8, 6), new THREE.MeshBasicMaterial({ transparent: true, opacity: 0, depthWrite: false }));
    hit.position.copy(p); hit.userData.dest = i; holoDyn.add(hit); UIT.push(hit);
    const c = cnv(512, 96), g = c.getContext('2d'); txt(g, d.name, 256, 50, 50, '#bff6ff', 'center', true);
    const lab = new THREE.Sprite(new THREE.SpriteMaterial({ map: ctex(c), transparent: true, depthWrite: false, toneMapped: false })); lab.scale.set(0.2, 0.0375, 1); lab.position.copy(p).add(new V3(0, 0.07, 0)); holoDyn.add(lab);
  });
}
function updateHolo() {
  holoInner.quaternion.copy(ship.quaternion).invert();
  if (R.landed) return;
  holoShip.position.copy(ship.position).multiplyScalar(HK); holoShip.quaternion.copy(ship.quaternion);
  const pa = holoLine.geometry.attributes.position;
  if (S.target >= 0 && DEST[S.target]) { const tp = _v.copy(DEST[S.target].pos).multiplyScalar(HK); pa.setXYZ(0, holoShip.position.x, holoShip.position.y, holoShip.position.z); pa.setXYZ(1, tp.x, tp.y, tp.z); pa.needsUpdate = true; holoLine.visible = true; holoSel.visible = true; holoSel.position.copy(tp); holoSel.material.rotation = R.time; }
  else { holoLine.visible = false; holoSel.visible = false; }
}
let tMark, tLabel, tLabelTxt = '';
function buildMarker() {
  tMark = new THREE.Sprite(new THREE.SpriteMaterial({ map: RING, color: new THREE.Color(0.5, 2, 0.8), transparent: true, depthTest: false, depthWrite: false, toneMapped: false, opacity: 0.85 })); tMark.renderOrder = 900; scene.add(tMark);
  const c = cnv(512, 96); tLabel = new THREE.Sprite(new THREE.SpriteMaterial({ map: ctex(c), transparent: true, depthTest: false, depthWrite: false, toneMapped: false })); tLabel.userData.c = c; tLabel.renderOrder = 901; scene.add(tLabel);
}
function updateMarker() {
  if (S.target < 0 || !DEST[S.target] || R.landed || R.busy === 'hyper') { tMark.visible = tLabel.visible = false; return; }
  const d = DEST[S.target]; const camP = camera.getWorldPosition(_v); const dist = camP.distanceTo(d.pos);
  tMark.visible = tLabel.visible = true; tMark.position.copy(d.pos); tMark.scale.setScalar(dist * 0.045); tMark.material.rotation = R.time * 0.5;
  _v2.set(0, 1, 0).applyQuaternion(ship.quaternion); tLabel.position.copy(d.pos).addScaledVector(_v2, dist * 0.045); tLabel.scale.set(dist * 0.16, dist * 0.03, 1);
  const s = d.name + ' · ' + fmtDist(dist);
  if (s !== tLabelTxt) { tLabelTxt = s; const c = tLabel.userData.c, g = c.getContext('2d'); g.clearRect(0, 0, 512, 96); txt(g, s, 256, 50, 44, '#c8ffd8', 'center', true); tLabel.material.map.needsUpdate = true; }
}

/* =========================================================
   PLANEEDI PIND (maandumine)
   ========================================================= */
const SURF_PAL = {
  toxic: { ground: ['#24401a', '#4e7a26', '#9ab83c', '#d8e07a'], sky: '#b8e070', zen: '#3a6a1a', fog: '#a8cc60', tree: '#b8e040', trunk: '#4a3a20', glow: '#e8ff70', rough: 1 },
  ocean: { ground: ['#d8c898', '#4a9a42', '#2e7a34', '#7a7a70', '#f0f4f8'], sky: '#a8d8ff', zen: '#2a62c0', fog: '#b8dcf8', water: '#1a5ab0', tree: '#2e8a3a', trunk: '#5a3a22', glow: '#80ffe0', rough: 1.2 },
  desert: { ground: ['#7a3a1a', '#b8703a', '#e0a868', '#f4d8a8'], sky: '#f4c898', zen: '#b06030', fog: '#ecc090', tree: '#6a8a3a', trunk: '#4a6a2a', glow: '#ffb040', rough: 0.9 },
  ice: { ground: ['#7a9ac0', '#b8d4ee', '#e6f2ff', '#ffffff'], sky: '#d8ecff', zen: '#6a9ad8', fog: '#e4f0ff', tree: '#a8e0ff', trunk: '#6a8ab0', glow: '#a0f0ff', rough: 1.1 },
  lava: { ground: ['#120c0a', '#2a1a14', '#4a3024', '#6a4a3a'], sky: '#7a2a12', zen: '#200a06', fog: '#5a2010', lava: '#ff5a10', tree: '#3a2a2a', trunk: '#1a1210', glow: '#ff7a20', rough: 1.3 },
};
const CSYL = ['puh', 'kõr', 'mõm', 'sil', 'tup', 'nök', 'vur', 'pii', 'lol', 'mumm', 'kik', 'sä', 'tsir', 'bub', 'kärp', 'hü'];
const CSUF = ['ik', 'ats', 'u', 'lane', 'kene', 'nõks', 'ur', 'ok'];
const creatureName = r => { const s = CSYL[(r() * CSYL.length) | 0] + CSUF[(r() * CSUF.length) | 0]; return s[0].toUpperCase() + s.slice(1); };
let SURF = null;
function makeSpecies(r, fly) {
  const hue = r();
  const sp = { name: creatureName(r), fly, size: fly ? 0.5 + r() * 0.6 : 0.5 + r() * 1.6, legs: fly ? 0 : (r() < 0.2 ? 0 : r() < 0.7 ? 4 : 6), neck: r() * 0.8, horns: r() < 0.5, ears: r() < 0.5, tail: r() < 0.7,
    speed: 1.5 + r() * 3, shy: r() < 0.65, body: [1 + r() * 0.6, 0.8 + r() * 0.4, 1.2 + r() * 0.8] };
  const c1 = new THREE.Color().setHSL(hue, 0.5 + r() * 0.35, 0.42 + r() * 0.2), c2 = new THREE.Color().setHSL((hue + 0.3 + r() * 0.4) % 1, 0.65, 0.55);
  sp.mats = { c1: new THREE.MeshStandardMaterial({ color: c1, roughness: 0.7 }), c2: new THREE.MeshStandardMaterial({ color: c2, roughness: 0.6 }), eye: new THREE.MeshStandardMaterial({ color: '#ffffff', roughness: 0.3 }), pupil: new THREE.MeshBasicMaterial({ color: '#05060a' }) };
  const b = new Builder(); const [bx, by, bz] = sp.body; const legL = sp.legs ? 0.7 : 0; const cy = fly ? 0 : legL + by * 0.4;
  b.add('c1', new THREE.SphereGeometry(1, 14, 10).scale(bx * 0.6, by * 0.5, bz * 0.7), 0, cy, 0);
  const hy = cy + by * 0.3 + sp.neck * 0.5, hz = -bz * 0.62;
  b.add('c1', new THREE.SphereGeometry(0.42, 12, 10), 0, hy, hz);
  if (sp.neck > 0.3) b.add('c1', cylY(0.14, sp.neck * 0.8, 8), 0, cy + by * 0.2 + sp.neck * 0.25, hz * 0.8, -0.5, 0, 0);
  if (sp.tail) b.add('c1', new THREE.ConeGeometry(0.15, 0.8, 8), 0, cy + 0.1, bz * 0.75, PI / 2 + 0.5, 0, 0);
  if (sp.horns) b.mirror('c2', () => new THREE.ConeGeometry(0.07, 0.35, 6), 0.18, hy + 0.38, hz + 0.05, 0, 0, -0.3);
  if (sp.ears) b.mirror('c2', () => new THREE.ConeGeometry(0.13, 0.3, 6).scale(1, 1, 0.35), 0.3, hy + 0.3, hz + 0.12, 0, 0, -0.7);
  for (let k = 0; k < 4; k++) b.add('c2', new THREE.SphereGeometry(0.1 + r() * 0.08, 8, 6), (r() - 0.5) * bx * 0.6, cy + by * 0.42, (r() - 0.5) * bz * 0.8);
  b.mirror('eye', () => new THREE.SphereGeometry(0.13, 10, 8), 0.17, hy + 0.08, hz - 0.33);
  b.mirror('pupil', () => new THREE.SphereGeometry(0.065, 8, 6), 0.18, hy + 0.09, hz - 0.44);
  sp.geos = b.geos(); sp.cy = cy; sp.legL = legL;
  sp.legGeo = cylY(0.08, Math.max(0.1, legL), 6).translate(0, -legL / 2, 0);
  sp.wingGeo = new THREE.BoxGeometry(1.3, 0.05, 0.7).translate(0.65, 0, 0);
  const legPos = sp.legs === 6 ? [[1, -1], [-1, -1], [1, 0], [-1, 0], [1, 1], [-1, 1]] : [[1, -1], [-1, -1], [1, 1], [-1, 1]];
  sp.legPos = sp.legs ? legPos.map(([sx, sz]) => [sx * bx * 0.33, legL, sz * bz * 0.42]) : [];
  return sp;
}
function makeCreature(sp) {
  const g = new THREE.Group(); const body = new THREE.Group(); g.add(body);
  for (const k in sp.geos) body.add(new THREE.Mesh(sp.geos[k], sp.mats[k]));
  const legs = sp.legPos.map(([x, y, z]) => { const l = new THREE.Mesh(sp.legGeo, sp.mats.c2); l.position.set(x, y, z); g.add(l); return l; });
  const wings = sp.fly ? [0, 1].map(k => { const w = new THREE.Mesh(sp.wingGeo, sp.mats.c2); w.position.set((k ? -1 : 1) * sp.body[0] * 0.3, sp.cy + 0.1, 0); if (k) w.rotation.y = PI; body.add(w); return w; }) : [];
  g.scale.setScalar(sp.size); return { g, body, legs, wings, sp, ph: Math.random() * 6, tx: 0, tz: 0, wait: 0, yaw: Math.random() * 6, home: new V3(), hover: 4 + Math.random() * 6 };
}
async function buildSurface(p) {
  const r = mulberry(p.seed * 3 + 11); const N = makeNoise(p.seed + 5); const pal = SURF_PAL[p.kind];
  const sc = new THREE.Scene(); sc.background = new THREE.Color(pal.fog); sc.fog = new THREE.Fog(pal.fog, 90, 1000); sc.environment = scene.environment;
  const sl = new THREE.DirectionalLight(sunLight.color, 2.8); sl.position.copy(SUN_DIR).multiplyScalar(100); sc.add(sl);
  sc.add(new THREE.HemisphereLight(pal.sky, pal.ground[0], 1.0));
  const water = pal.water ? -8 : pal.lava ? -12 : -999;
  const H = (x, z) => { const h = (N.fbm(x * 0.0032 + 50, 3, z * 0.0032, 5) - 0.5) * 130 * pal.rough + (N.fbm(x * 0.015, 7, z * 0.015, 3) - 0.5) * 10; const f = smooth(30, 100, Math.hypot(x, z - 3)); return lerp(0, h, f); };
  const SIZE = 1400, SEG = 150; const tg = new THREE.PlaneGeometry(SIZE, SIZE, SEG, SEG).rotateX(-PI / 2);
  const pa = tg.attributes.position; const col = new Float32Array(pa.count * 3); const gst = ST(pal.ground.map((h, k, a) => [k / (a.length - 1), h])); const lin = v => Math.pow(v / 255, 2.2);
  for (let k = 0; k < pa.count; k++) {
    const x = pa.getX(k), z = pa.getZ(k), y = H(x, z); pa.setY(k, y);
    let c = ramp(gst, clamp((y + 50) / 110 + (N.n3(x * 0.05, 1, z * 0.05) - 0.5) * 0.25, 0, 1));
    if (y < water + 2) c = mix(c, [90, 80, 60], 0.5);
    col[k * 3] = lin(c[0]); col[k * 3 + 1] = lin(c[1]); col[k * 3 + 2] = lin(c[2]);
  }
  tg.setAttribute('color', new THREE.BufferAttribute(col, 3)); tg.computeVertexNormals();
  sc.add(new THREE.Mesh(tg, new THREE.MeshStandardMaterial({ vertexColors: true, flatShading: true, roughness: 1 })));
  await nextFrame();
  if (pal.water) { const w = new THREE.Mesh(new THREE.PlaneGeometry(SIZE, SIZE).rotateX(-PI / 2), new THREE.MeshStandardMaterial({ color: pal.water, transparent: true, opacity: 0.85, roughness: 0.15, metalness: 0.3 })); w.position.y = water; sc.add(w); }
  if (pal.lava) { const w = new THREE.Mesh(new THREE.PlaneGeometry(SIZE, SIZE).rotateX(-PI / 2), new THREE.MeshBasicMaterial({ color: new THREE.Color(pal.lava).multiplyScalar(1.6), toneMapped: false })); w.position.y = water; sc.add(w); }
  // taevakuppel
  const dg = new THREE.SphereGeometry(1300, 32, 16); const dc = new Float32Array(dg.attributes.position.count * 3); const ca = new THREE.Color(pal.fog), cb = new THREE.Color(pal.zen);
  for (let k = 0; k < dg.attributes.position.count; k++) { const t = clamp(dg.attributes.position.getY(k) / 1300, 0, 1); const c = ca.clone().lerp(cb, Math.pow(t, 0.6)); dc.set([c.r, c.g, c.b], k * 3); }
  dg.setAttribute('color', new THREE.BufferAttribute(dc, 3)); sc.add(new THREE.Mesh(dg, new THREE.MeshBasicMaterial({ vertexColors: true, side: THREE.BackSide, fog: false, depthWrite: false })));
  const sun = glowSprite(sunLight.color, 260); sun.position.copy(SUN_DIR).multiplyScalar(1100); sc.add(sun);
  const other = PL.find(q => q !== p && q.kind === 'gas') || PL.find(q => q !== p);
  if (other) { const moon = new THREE.Mesh(new THREE.SphereGeometry(140, 32, 20), new THREE.MeshBasicMaterial({ map: other.mesh.material.map, fog: false, transparent: true, opacity: 0.85 })); moon.position.set(-500, 700, -800); sc.add(moon); }
  // taimed, kivid
  const okSpot = (x, z) => !(Math.abs(x) < 22 && z > -26 && z < 30) && H(x, z) > water + 1;
  const trunkIM = new THREE.InstancedMesh(cylY(0.3, 1, 6), new THREE.MeshStandardMaterial({ color: pal.trunk, roughness: 1 }), 260);
  const crownIM = new THREE.InstancedMesh(new THREE.IcosahedronGeometry(1, 0), new THREE.MeshStandardMaterial({ color: '#ffffff', roughness: 0.9, flatShading: true }), 260);
  const glowIM = new THREE.InstancedMesh(new THREE.SphereGeometry(0.35, 10, 8), new THREE.MeshBasicMaterial({ color: new THREE.Color(pal.glow).multiplyScalar(1.5), toneMapped: false }), 160);
  const stemIM = new THREE.InstancedMesh(cylY(0.05, 1, 5), new THREE.MeshStandardMaterial({ color: pal.trunk }), 160);
  const rockIM = new THREE.InstancedMesh(new THREE.IcosahedronGeometry(1, 0), new THREE.MeshStandardMaterial({ color: pal.ground[1], roughness: 1, flatShading: true }), 200);
  const tc = new THREE.Color(pal.tree);
  for (let k = 0; k < 260; k++) {
    let x, z; for (let t = 0; t < 10; t++) { x = (r() - 0.5) * 1200; z = (r() - 0.5) * 1200; if (okSpot(x, z)) break; }
    const y = H(x, z), h = 2 + r() * 6; trunkIM.setMatrixAt(k, _m.compose(_v.set(x, y + h / 2, z), _q.identity(), _v2.set(1, h, 1)));
    const cs = 1.5 + r() * 3; crownIM.setMatrixAt(k, _m.compose(_v.set(x, y + h + cs * 0.6, z), _q.setFromEuler(_e.set(r(), r() * 6, r())), _v2.set(cs, cs * (0.7 + r() * 0.8), cs)));
    crownIM.setColorAt(k, tc.clone().offsetHSL((r() - 0.5) * 0.08, 0, (r() - 0.5) * 0.15));
  }
  for (let k = 0; k < 160; k++) { let x, z; for (let t = 0; t < 10; t++) { x = (r() - 0.5) * 900; z = (r() - 0.5) * 900; if (okSpot(x, z)) break; } const y = H(x, z), h = 0.4 + r() * 1.4; stemIM.setMatrixAt(k, _m.compose(_v.set(x, y + h / 2, z), _q.identity(), _v2.set(1, h, 1))); const s = 0.6 + r() * 1.2; glowIM.setMatrixAt(k, _m.compose(_v.set(x, y + h, z), _q.identity(), _v2.set(s, s * 0.8, s))); }
  for (let k = 0; k < 200; k++) { const x = (r() - 0.5) * 1200, z = (r() - 0.5) * 1200; const s = 0.5 + r() * r() * 6; rockIM.setMatrixAt(k, _m.compose(_v.set(x, H(x, z) + s * 0.3, z), _q.setFromEuler(_e.set(r() * 6, r() * 6, r() * 6)), _v2.set(s, s * 0.7, s))); }
  sc.add(trunkIM, crownIM, glowIM, stemIM, rockIM);
  await nextFrame();
  // kristallimaardlad
  const DEPN = 28; const depIM = new THREE.InstancedMesh(new THREE.OctahedronGeometry(0.6, 0).scale(1, 2, 1), new THREE.MeshBasicMaterial({ color: '#ffffff', toneMapped: false }), DEPN * 4);
  const deposits = [];
  for (let k = 0; k < DEPN; k++) {
    let x, z; for (let t = 0; t < 15; t++) { const a = r() * PI * 2, d = 40 + r() * 400; x = Math.cos(a) * d; z = Math.sin(a) * d; if (okSpot(x, z)) break; }
    const y = H(x, z); const gold = r() < 0.25; const c = gold ? new THREE.Color(2.2, 1.5, 0.3) : new THREE.Color(0.4, 1.8, 2.4); const idx = [];
    for (let q = 0; q < 4; q++) { const i = k * 4 + q; idx.push(i); depIM.setMatrixAt(i, _m.compose(_v.set(x + (r() - 0.5) * 1.6, y + 0.6, z + (r() - 0.5) * 1.6), _q.setFromEuler(_e.set((r() - 0.5) * 0.8, r() * 6, (r() - 0.5) * 0.8)), _v2.setScalar(0.7 + r() * 0.8))); depIM.setColorAt(i, c); }
    deposits.push({ c: new V3(x, y + 1, z), idx, hp: 3, val: gold ? 6 : 3, alive: true });
  }
  sc.add(depIM);
  // monoliidid
  const mc = cnv(128, 512), mg = mc.getContext('2d'); mg.fillStyle = '#000'; mg.fillRect(0, 0, 128, 512); mg.strokeStyle = '#fff'; mg.lineWidth = 6;
  for (let y = 30; y < 490; y += 46) { mg.beginPath(); mg.moveTo(30 + Math.random() * 20, y); for (let s = 0; s < 3; s++) mg.lineTo(20 + Math.random() * 88, y + Math.random() * 30); mg.stroke(); }
  const monoMat = new THREE.MeshStandardMaterial({ color: '#1a1a22', roughness: 0.4, metalness: 0.6, emissive: new THREE.Color(pal.glow), emissiveMap: ctex(mc), emissiveIntensity: 2.2 });
  const monos = [];
  for (let k = 0; k < 2; k++) { const a = r() * PI * 2, d = 140 + r() * 260; const x = Math.cos(a) * d, z = Math.sin(a) * d; const y = Math.max(H(x, z), water + 0.5); const m = new THREE.Mesh(tBox(2.4, 8, 1.2), monoMat); m.position.set(x, y + 4, z); m.rotation.y = r() * 3; sc.add(m); const gl = glowSprite(pal.glow, 10, 0.7); gl.position.set(x, y + 9, z); sc.add(gl); monos.push({ pos: new V3(x, y, z), key: `${S.system}:${p.name}:m${k}` }); }
  // olendid
  const species = [makeSpecies(r, false), makeSpecies(r, false), makeSpecies(r, r() < 0.6)];
  const creatures = [];
  for (const sp of species) for (let k = 0; k < 4; k++) {
    const c = makeCreature(sp); let x, z; for (let t = 0; t < 15; t++) { const a = r() * PI * 2, d = 25 + r() * 260; x = Math.cos(a) * d; z = Math.sin(a) * d; if (okSpot(x, z)) break; }
    c.home.set(x, 0, z); c.g.position.set(x, H(x, z), z); c.tx = x; c.tz = z; sc.add(c.g); creatures.push(c);
  }
  // pilved, millest laev läbi lendab
  const cts = [cloudTex(71), cloudTex(72), cloudTex(73)]; const cloudCol = p.kind === 'lava' ? '#5a3a30' : p.kind === 'toxic' ? '#e0f0b0' : '#ffffff';
  for (let k = 0; k < 34; k++) { const s = new THREE.Sprite(new THREE.SpriteMaterial({ map: cts[k % 3], color: cloudCol, transparent: true, depthWrite: false, opacity: 0.9 })); const a = r() * PI * 2, d = 80 + r() * 700; s.position.set(Math.cos(a) * d, 170 + r() * 110, Math.sin(a) * d * 0.6 + 250); s.scale.setScalar(160 + r() * 220); sc.add(s); }
  // tolm maandumiskohas
  const dust = new THREE.Sprite(new THREE.SpriteMaterial({ map: cts[0], color: pal.ground[1], transparent: true, depthWrite: false, opacity: 0 })); dust.position.set(0, 2.5, 2); dust.scale.setScalar(55); sc.add(dust);
  const beam = new THREE.Line(new THREE.BufferGeometry().setFromPoints([new V3(), new V3(0, 0, -1)]), new THREE.LineBasicMaterial({ color: 0x7dff9a, toneMapped: false, fog: false }));
  return { scene: sc, H, water, deposits, depIM, monos, creatures, planet: p, beam, dust };
}
function surfaceGround(x, z) { let g = SURF.H(x, z); if (Math.abs(x) < 1.2 && z >= 10.9 && z <= 17.5) g = Math.max(g, LAND_H * (17.5 - z) / 6.6); return g; }
function updateSurface(dt) {
  if (!SURF) return;
  const pl = R.mode === 'ground' ? rig.position : null;
  for (const c of SURF.creatures) {
    const p = c.g.position, sp = c.sp; let tx = c.tx, tz = c.tz, spd = sp.speed, flee = false;
    if (pl && sp.shy && Math.hypot(p.x - pl.x, p.z - pl.z) < 9) { tx = p.x + (p.x - pl.x) * 3; tz = p.z + (p.z - pl.z) * 3; spd *= 2.2; flee = true; }
    const dx = tx - p.x, dz = tz - p.z, d = Math.hypot(dx, dz); let moving = false;
    if (d < 1.5 && !flee) { c.wait -= dt; if (c.wait <= 0) { c.tx = c.home.x + rnd(-60, 60); c.tz = c.home.z + rnd(-60, 60); c.wait = rnd(1, 5); if (SURF.H(c.tx, c.tz) < SURF.water + 1) { c.tx = c.home.x; c.tz = c.home.z; } } }
    else { moving = true; const st = Math.min(d, spd * dt); const nx = p.x + dx / d * st, nz = p.z + dz / d * st; if (SURF.H(nx, nz) > SURF.water + 0.5 || sp.fly) { p.x = nx; p.z = nz; } else { c.tx = c.home.x; c.tz = c.home.z; } const want = Math.atan2(-dx, -dz); let df = want - c.yaw; while (df > PI) df -= 2 * PI; while (df < -PI) df += 2 * PI; c.yaw += df * Math.min(1, dt * 5); }
    c.g.rotation.y = c.yaw; c.ph += dt * (moving ? spd * 3 : 1);
    let y = SURF.H(p.x, p.z);
    if (sp.fly) { y = Math.max(y, SURF.water) + c.hover + Math.sin(c.ph * 0.5) * 0.8; for (const w of c.wings) w.rotation.z = Math.sin(R.time * 11 + c.hover) * 0.6; }
    else if (!sp.legs) { y += moving ? Math.abs(Math.sin(c.ph * 1.3)) * 0.7 * sp.size : 0; }
    p.y = y; c.body.position.y = moving ? Math.sin(c.ph * 2) * 0.04 : 0;
    c.legs.forEach((l, k) => l.rotation.x = moving ? Math.sin(c.ph + (k % 2) * PI) * 0.6 : l.rotation.x * 0.9);
    if (pl && !flee && Math.random() < dt * 0.05 && Math.hypot(p.x - pl.x, p.z - pl.z) < 20) sfx.chirp();
  }
  if (R.mode === 'ground') {
    for (const m of SURF.monos) if (!S.monos.includes(m.key) && Math.hypot(m.pos.x - rig.position.x, m.pos.z - rig.position.z) < 7) { S.monos.push(m.key); S.credits += 40; sfx.fanfare(); say('Leidsid iidse monoliidi! +40 kr'); save(); }
  }
}

/* =========================================================
   TEGEVUSED
   ========================================================= */
function say(t) {
  if (R.msg === t && R.msgT > 0) return;
  if (R.msgT > 1.2) { if (!R.queue.includes(t)) { R.queue.push(t); if (R.queue.length > 3) R.queue.shift(); } return; }
  showMsg(t);
}
function showMsg(t) { R.msg = t; R.msgT = 4; msgPanel.refresh(); msgPanel.mesh.visible = true; sfx.msg(); }
function updateMsg(dt) {
  if (R.msgT > 0) { R.msgT -= dt; msgPanel.mesh.material.opacity = clamp(R.msgT * 2, 0, 1); if (R.msgT <= 0) { msgPanel.mesh.visible = false; R.msg = ''; } }
  else if (R.queue.length) showMsg(R.queue.shift());
}
function fadeTo(v, cb) { R.fadeTarget = v; R.fadeCb = cb || null; }
function updateFade(dt) {
  if (R.fade !== R.fadeTarget) { R.fade = approach(R.fade, R.fadeTarget, dt * 1.2); }
  else if (R.fadeCb) { const cb = R.fadeCb; R.fadeCb = null; cb(); }
  fader.visible = R.fade > 0.001; fader.material.opacity = R.fade;
}
function setTarget(i) { if (i < 0 || i >= DEST.length) return; S.target = i; R.faceT = 0; say(`Sihtkoht: ${DEST[i].name}` + (R.landed ? '' : ` (${fmtDist(ship.position.distanceTo(DEST[i].pos))})`)); }
function cycleTarget() { if (DEST.length) setTarget((S.target + 1) % DEST.length); }
function toggleAutopilot() {
  if (R.landed) { say('Stardi enne planeedilt!'); return; }
  if (onFoot()) { say('Mine enne laeva tagasi!'); return; }
  if (R.busy) return;
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
  else if (m === 'outside') { rig.position.set(0, 7.5, 44); rig.rotation.set(0, 0, 0); }
  applyDeskCam();
}
function applyDeskCam() { if (renderer.xr.isPresenting) return; camera.position.set(0, R.mode === 'seat' ? 1.3 : 1.65, 0); camera.rotation.set(R.mode === 'outside' ? -0.16 : 0, 0, 0); }
function toggleSeat() {
  if (onFoot()) { enterShip(); return; }
  if (R.mode === 'seat') { R.walkPos = null; setMode('walk'); if (!R.autopilot) R.throttle = 0; say(R.landed ? 'Mine tagauksest välja!' : R.autopilot ? 'Autopiloot juhib. Kõnni ringi!' : 'Tõusid püsti. Laev jääb seisma.'); }
  else { setMode('seat'); say('Istud roolis.'); }
}
function toggleOutside() {
  if (onFoot() || R.busy === 'hyper' || (R.landed && !R.busy)) return;
  if (R.mode === 'outside') setMode(R.prevMode || 'seat'); else { R.prevMode = R.mode; setMode('outside'); say('Välisvaade. Vajuta X, et tagasi minna.'); }
}
function exitShip() {
  if (onFoot() || R.busy) return;
  if (R.landed) {
    if (R.mode === 'walk') R.walkPos = rig.position.clone();
    SURF.scene.add(rig); R.mode = 'ground'; rig.position.set(0, LAND_H * (17.5 - 12.6) / 6.6, 12.6); rig.rotation.set(0, PI, 0); R.jv = 0; R.jy = 0; rig.add(SURF.beam);
    applyDeskCam(); say(`Tere tulemast planeedile ${R.landed.name}! Skaneeri loomi.`); return;
  }
  if (Math.abs(R.speed) > 8 || R.autopilot) { if (R.time - R.doorWarn > 3) { R.doorWarn = R.time; say('Peata laev enne väljumist!'); } return; }
  R.autopilot = false; R.throttle = 0; R.speed = 0; R.bump.set(0, 0, 0);
  if (R.mode === 'walk') R.walkPos = rig.position.clone();
  ship.updateMatrixWorld(true); scene.add(rig); R.mode = 'eva';
  rig.position.copy(ship.localToWorld(new V3(0, 0.6, 13.5))); rig.quaternion.copy(ship.quaternion); rig.rotateY(PI); R.ev.set(0, 0, 0); rig.add(EVA_BEAM);
  applyDeskCam(); say('Kosmosekõnd! Pulgaga liigud, B viib laeva tagasi.');
}
function enterShip() {
  if (!onFoot()) return;
  ship.add(rig); R.mode = 'walk'; rig.position.set(0, 0, 9.2); rig.rotation.set(0, 0, 0); R.walkPos = null;
  if (SURF) rig.remove(SURF.beam); rig.remove(EVA_BEAM);
  applyDeskCam(); say('Tagasi laevas.');
}
function exitEnter() { if (onFoot()) enterShip(); else exitShip(); }
function landToggle() { if (R.landed) takeoff(); else startLanding(); }
/* ---------- nähtav maandumine ja start ---------- */
const LS = { t: 0, p: null, surf: null, entered: false, done: false, gear: false };
const LP0 = new V3(0, 300, 650), LP1 = new V3(0, 140, 280), LP2 = new V3(0, 35, 0);
const TP0 = new V3(0, 35, 0), TP1 = new V3(0, 140, -220), TP2 = new V3(0, 460, -950);
const QI = new Q4();
function setFade(v) { R.fade = R.fadeTarget = clamp(v, 0, 1); }
function setFlame(v) {
  shipFX.flameG.visible = v > 0.01;
  for (const g of shipFX.flame) { g.material.opacity = v; g.scale.setScalar(g.userData.base * (0.7 + v * 0.5) * (0.85 + Math.random() * 0.3)); }
}
function bez(a, b, c, u, out) { const k = 1 - u; return out.set(k * k * a.x + 2 * k * u * b.x + u * u * c.x, k * k * a.y + 2 * k * u * b.y + u * u * c.y, k * k * a.z + 2 * k * u * b.z + u * u * c.z); }
function shipAlong(a, b, c, u, levelOut) {
  bez(a, b, c, u, ship.position);
  const k = 1 - u; _v.set(2 * k * (b.x - a.x) + 2 * u * (c.x - b.x), 2 * k * (b.y - a.y) + 2 * u * (c.y - b.y), 2 * k * (b.z - a.z) + 2 * u * (c.z - b.z));
  _m.lookAt(ship.position, _v2.copy(ship.position).add(_v), UP); _q.setFromRotationMatrix(_m);
  ship.quaternion.slerpQuaternions(_q, QI, levelOut);
}
function startLanding() {
  const p = R.nearPlanet; if (!p || R.landed || R.busy || onFoot()) return;
  if (p.kind === 'gas') { say('Gaasihiiglasele ei saa maanduda!'); return; }
  R.busy = 'land'; R.autopilot = false; R.throttle = 0;
  Object.assign(LS, { t: 0, p, surf: null, entered: false, done: false, gear: false });
  R.spaceBack.copy(ship.position).sub(p.pos).normalize();
  fader.material.color.set(p.atm);
  buildSurface(p).then(s => { LS.surf = s; });
  say(`Lähenen planeedile ${p.name}…`);
}
function landSpace(dt) {
  const p = LS.p; LS.t += dt; const d = ship.position.distanceTo(p.pos);
  _m.lookAt(ship.position, p.pos, _v3.set(0, 1, 0).applyQuaternion(ship.quaternion)); _q.setFromRotationMatrix(_m);
  const ang = ship.quaternion.angleTo(_q); ship.quaternion.rotateTowards(_q, 1.3 * dt);
  if (!LS.entered && d < p.r * 1.6) { LS.entered = true; say('Sisenen atmosfääri…'); haptic('left', 0.5, 300); haptic('right', 0.5, 300); }
  setFlame(smooth(p.r * 1.9, p.r * 1.22, d));
  if (!LS.done && (d < p.r * 1.16 || LS.t > 10)) LS.done = true;
  if (LS.done) {
    R.speed = approach(R.speed, 0, 1500 * dt); setFade(R.fade + dt * 1.6);
    if (R.fade >= 1 && LS.surf) swapToSurface();
  } else {
    R.speed = approach(R.speed, ang > 0.5 ? 150 : clamp((d - p.r) * 0.9, 250, 3000), 900 * dt);
    setFade(smooth(p.r * 1.5, p.r * 1.16, d) * 0.9);
  }
}
function swapToSurface() {
  SURF = LS.surf; SURF.scene.add(ship); ship.position.copy(LP0); ship.quaternion.identity(); ship.updateMatrixWorld(true);
  activeScene = SURF.scene; R.busy = 'land2'; LS.t = 0; R.speed = 0; R.vel.set(0, 0, 0); R.bump.set(0, 0, 0);
  shipFX.gear.visible = false; mus.t = 0.5;
  if (!S.discovered.includes(discKey(LS.p))) { S.discovered.push(discKey(LS.p)); S.credits += 50; }
  say('Läbime pilvi – maandumine algab!');
}
function finishLanding() {
  R.landed = LS.p; R.busy = null; shipFX.ramp.visible = true; shipFX.door.visible = false; shipFX.gear.position.y = 0;
  ship.position.set(0, LAND_H, 0); ship.quaternion.identity(); setFlame(0);
  if (R.mode === 'outside') setMode('seat');
  sfx.boom(); haptic('left', 0.6, 200); haptic('right', 0.6, 200);
  say('Maandusid! Tõuse püsti ja mine tagauksest välja.'); save();
}
function takeoff() {
  if (!R.landed || R.busy) return;
  if (onFoot()) enterShip();
  R.busy = 'takeoff'; LS.t = 0; LS.p = R.landed;
  shipFX.ramp.visible = false; shipFX.door.visible = true; fader.material.color.set(LS.p.atm);
  say('Start! Hoia kinni…');
}
function swapToSpace() {
  const p = LS.p; scene.add(ship); const dir = R.spaceBack; ship.position.copy(p.pos).addScaledVector(dir, p.r * 1.3);
  _m.lookAt(ship.position, _v.copy(ship.position).add(dir), UP); ship.quaternion.setFromRotationMatrix(_m);
  shipFX.gear.visible = false; activeScene = scene;
  const old = SURF; SURF = null; old.scene.remove(old.beam); disposeTree(old.scene, false);
  R.landed = null; R.busy = 'takeoff2'; LS.t = 0; R.speed = 500; mus.t = 0.5;
}
function updateLandSeq(dt) {
  const st = R.busy;
  if (st === 'land2') {
    LS.t += dt; const t = LS.t; setFade(1 - smooth(0, 1.3, t));
    if (t < 6.5) { const s = t / 6.5; shipAlong(LP0, LP1, LP2, 1 - Math.pow(1 - s, 2.2), smooth(4, 6.5, t)); setFlame(0.6 * (1 - smooth(0, 2.5, t))); SURF.dust.material.opacity = 0; }
    else {
      const s = clamp((t - 6.5) / 3.5, 0, 1); ship.quaternion.identity(); ship.position.set(0, lerp(LP2.y, LAND_H, smooth(0, 1, s)), 0);
      if (!LS.gear) { LS.gear = true; shipFX.gear.visible = true; say('Maandumisjalad välja!'); }
      shipFX.gear.position.y = 2.5 * (1 - smooth(0, 0.45, s));
      SURF.dust.material.opacity = 0.75 * (1 - clamp((ship.position.y - LAND_H) / 22, 0, 1));
      if (s >= 1) finishLanding();
    }
  } else if (st === 'takeoff') {
    LS.t += dt; const t = LS.t;
    if (t < 3) {
      ship.quaternion.identity(); ship.position.set(0, lerp(LAND_H, TP0.y, smooth(0, 3, t)), 0);
      shipFX.gear.position.y = 2.5 * smooth(1, 2.5, t); shipFX.gear.visible = t < 2.5;
      SURF.dust.material.opacity = 0.75 * (1 - clamp((ship.position.y - LAND_H) / 22, 0, 1));
    } else {
      const s = clamp((t - 3) / 5, 0, 1); shipAlong(TP0, TP1, TP2, s * s, 1 - smooth(3, 4.5, t));
      SURF.dust.material.opacity = Math.max(0, SURF.dust.material.opacity - dt * 0.5);
      setFlame(smooth(6, 8, t) * 0.7); setFade(smooth(6.6, 8, t));
      if (t >= 8.1) swapToSpace();
    }
  } else if (st === 'takeoff2') {
    LS.t += dt; const t = LS.t; setFade(1 - smooth(0, 1.3, t)); setFlame(0.7 * (1 - smooth(0, 2, t))); R.speed = approach(R.speed, 300, 200 * dt);
    if (t > 2) { R.busy = null; setFlame(0); say('Tagasi kosmoses. Head reisi, kapten!'); }
  }
  if (!R.busy && SURF && SURF.dust.material.opacity > 0) SURF.dust.material.opacity = Math.max(0, SURF.dust.material.opacity - dt * 0.35);
}
function startHyper(to) {
  if (R.busy || R.landed) return;
  if (onFoot()) { say('Mine enne laeva tagasi!'); return; }
  if (to === S.system) return;
  for (const p of PL) if (ship.position.distanceTo(p.pos) < p.r + 1500) { say('Liiga lähedal planeedile – lenda eemale!'); sfx.deny(); return; }
  const cost = S.easy ? 0 : 40; if (S.fuel < cost) { say('Hüperhüppeks on vaja 40 kütust!'); sfx.deny(); return; }
  S.fuel -= cost; R.busy = 'hyper'; R.hyperT = 0; R.hyperReady = false; R.hyperTo = to; R.autopilot = false; R.throttle = 0;
  if (R.mode === 'outside') setMode('seat');
  say('Hüperhüpe! Hoia kinni…'); sfx.hyper(); haptic('left', 1, 400); haptic('right', 1, 400);
}
function updateHyper(dt) {
  if (R.busy !== 'hyper') return;
  R.hyperT += dt; const T = shipFX.tunnel; T.g.visible = true;
  const tIn = clamp(R.hyperT / 1.2, 0, 1);
  if (R.hyperT > 1.3 && !R.hyperStarted) { R.hyperStarted = true; (async () => { S.system = R.hyperTo; S.target = -1; await buildSystem(S.system); R.hyperReady = true; save(); })(); }
  let out = 0;
  if (R.hyperReady && R.hyperT > 4.5) { if (!R.hyperExit) { R.hyperExit = R.hyperT; ship.position.set(0, -190, 900); ship.quaternion.identity(); R.speed = 700; } out = clamp((R.hyperT - R.hyperExit) / 1.2, 0, 1); }
  const v = tIn * (1 - out);
  T.dark.material.opacity = v; T.str.material.opacity = v; T.end.material.opacity = v; T.tex.offset.y -= dt * (0.5 + tIn * 3);
  if (!R.hyperExit) R.speed = approach(R.speed, 2500, 1500 * dt);
  if (out >= 1) { T.g.visible = false; R.busy = null; R.hyperStarted = false; R.hyperExit = 0; sfx.fanfare(); say(`Tere tulemast: ${SYSINFO.name}!`); }
}
function sell() { const n = S.crystals; if (n <= 0) return; S.crystals = 0; S.credits += n * 10; sfx.fanfare(); say(`Müüdud ${n} kristalli: +${n * 10} kr`); save(); }
function buyFuel() { if (S.credits < 30) return; S.credits -= 30; S.fuel = stat.maxFuel; say('Paak on täis!'); save(); }
function repair() { S.shield = stat.maxShield; say('Kilp parandatud!'); save(); }
function refine(n) { let k = 0; while (n-- > 0 && S.crystals >= 3 && S.fuel < stat.maxFuel - 0.5) { S.crystals -= 3; S.fuel = Math.min(stat.maxFuel, S.fuel + 20); k++; } if (k) { sfx.pick(); say(`Rafineeritud! +${k * 20} kütust`); save(); } }
function buyUp(k) { const lv = S.up[k], price = UPG_BASE[k] * (lv + 1); if (lv >= 3 || S.credits < price) return; S.credits -= price; S.up[k]++; if (k === 'tank') S.fuel += 50; if (k === 'shield') S.shield += 50; sfx.fanfare(); say(`${UPG.find(u => u[0] === k)[1]} uuendatud! Tase ${S.up[k]}`); save(); }
function resetGame() {
  if (R.resetArm <= 0) { R.resetArm = 4; return; }
  R.resetArm = 0; const music = S.music; Object.assign(S, DEFAULT_S()); S.music = music; save(); location.reload();
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
  switch (e.code) {
    case 'KeyP': toggleAutopilot(); break; case 'KeyF': toggleSeat(); break; case 'KeyV': toggleOutside(); break; case 'KeyT': cycleTarget(); break;
    case 'KeyL': landToggle(); break; case 'KeyG': exitEnter(); break; case 'KeyH': startHyper(S.system + 1); break; case 'KeyM': S.music = !S.music; save(); break;
  }
  if (/^Digit[1-8]$/.test(e.code)) setTarget(+e.code.slice(5) - 1);
  if (e.code === 'Space') e.preventDefault();
});
addEventListener('keyup', e => { keys[e.code] = false; });
function readKeys() {
  if (renderer.xr.isPresenting) return; const ax = (a, b) => (keys[a] ? 1 : 0) - (keys[b] ? 1 : 0);
  if (R.mode === 'walk' || onFoot()) { IN.lx = ax('KeyD', 'KeyA'); IN.ly = ax('KeyS', 'KeyW'); IN.ry = ax('KeyE', 'KeyQ'); }
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
  if (R.mode === 'eva') rig.rotateY(-e.movementX * 0.0025); else rig.rotation.y -= e.movementX * 0.0025;
  camera.rotation.x = clamp(camera.rotation.x - e.movementY * 0.0025, -1.4, 1.4);
});
const ray = new THREE.Raycaster(); ray.far = 8;
function cast(o, d) { ray.set(o, d); const h = ray.intersectObjects(UIT.filter(m => m.parent && m.visible), false); return h[0] || null; }
function castCenter() { camera.updateMatrixWorld(); return cast(camera.getWorldPosition(new V3()), camera.getWorldDirection(new V3())); }
function ctrlRay(c, o, d) { _m.identity().extractRotation(c.matrixWorld); o.setFromMatrixPosition(c.matrixWorld); d.set(0, 0, -1).applyMatrix4(_m); }
function castCtrl(c) { const o = new V3(), d = new V3(); ctrlRay(c, o, d); return cast(o, d); }
function activateHit(h) { const P = h.object.userData.panel; if (P) P.click(h.uv); else if (h.object.userData.dest !== undefined) { sfx.click(); setTarget(h.object.userData.dest); } }
const ctrls = [], grips = [];
let EVA_BEAM;
function buildControllers() {
  const cmf = new XRControllerModelFactory();
  for (let k = 0; k < 2; k++) {
    const c = renderer.xr.getController(k); c.userData = { hand: null, src: null, held: false, ui: false };
    const gr = renderer.xr.getControllerGrip(k);
    c.addEventListener('connected', e => { c.userData.hand = e.data.handedness; c.userData.src = e.data; if (e.data.handedness === 'left') gr.add(wristPanel.mesh); });
    c.addEventListener('disconnected', () => { c.userData.src = null; });
    c.addEventListener('selectstart', () => { initAudio(); c.userData.held = true; const h = castCtrl(c); c.userData.ui = !!h; if (h) activateHit(h); });
    c.addEventListener('selectend', () => { c.userData.held = false; c.userData.ui = false; });
    const line = new THREE.Line(new THREE.BufferGeometry().setFromPoints([new V3(), new V3(0, 0, -1)]), new THREE.LineBasicMaterial({ color: 0x7ff0ff, transparent: true, opacity: 0.6, toneMapped: false }));
    c.add(line); c.userData.line = line;
    const dot = new THREE.Mesh(new THREE.SphereGeometry(0.012, 8, 6), new THREE.MeshBasicMaterial({ color: 0xbfffff, toneMapped: false })); dot.visible = false; rig.add(dot); c.userData.dot = dot;
    rig.add(c); gr.add(cmf.createControllerModel(gr)); rig.add(gr); ctrls.push(c); grips.push(gr);
  }
  EVA_BEAM = new THREE.Line(new THREE.BufferGeometry().setFromPoints([new V3(), new V3(0, 0, -1)]), new THREE.LineBasicMaterial({ color: 0xffa040, toneMapped: false }));
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
   LENDAMINE, KÕNDIMINE, TÖÖRIIST
   ========================================================= */
function updateControls(dt) {
  let fireNow = R.mouseFire || (!onFoot() && keys.Space), boost = keys.ShiftLeft || keys.ShiftRight;
  let jet = onFoot() && keys.Space;
  for (const c of ctrls) if (c.userData.held && !c.userData.ui) { if (c.userData.hand === 'right') fireNow = true; if (c.userData.hand === 'left') { boost = true; jet = true; } }
  R.boost = flying() && boost; R.jetHeld = jet; R.toolHeld = onFoot() && fireNow;
  R.fireCd -= dt; if (flying() && fireNow && R.fireCd <= 0) { fire(); R.fireCd = 0.14; }
}
function updateFlight(dt) {
  if (R.landed || SURF) { R.speed = 0; R.vel.set(0, 0, 0); return; }
  const fuelOK = S.fuel > 0.01;
  if (R.busy === 'hyper') { R.vel.copy(fwdOf(ship, _v)).multiplyScalar(R.speed); ship.position.addScaledVector(R.vel, dt); if (R.hyperExit) R.speed = approach(R.speed, 0, 500 * dt); return; }
  if (R.busy === 'land') landSpace(dt);
  else if (R.autopilot && flying() && (Math.abs(IN.rx) > 0.6 || Math.abs(IN.ry) > 0.6)) { R.autopilot = false; say('Käsijuhtimine – autopiloot välja.'); }
  if (R.busy) { }
  else if (R.autopilot) autopilot(dt, fuelOK);
  else if (flying()) {
    R.throttle = clamp(R.throttle - IN.ly * dt * 0.6, -0.3, 1);
    const k = 1 - Math.exp(-dt * 4);
    R.yaw = lerp(R.yaw, -IN.rx * 0.75, k); R.pitch = lerp(R.pitch, IN.ry * 0.6, k); R.roll = lerp(R.roll, -IN.lx * 1.1, k);
    ship.quaternion.multiply(_q.setFromEuler(_e.set(R.pitch * dt, R.yaw * dt, R.roll * dt, 'YXZ'))).normalize();
    let tgt = R.throttle * stat.manual; if (R.boost && fuelOK) tgt = stat.pulse; if (!fuelOK) tgt = clamp(tgt, -40, 40);
    R.speed = approach(R.speed, tgt, (tgt > R.speed ? (R.boost ? 380 : 60) : 260) * dt);
    if (R.faceT > 0) faceTarget(dt);
  } else { R.speed = approach(R.speed, 0, (R.mode === 'eva' ? 200 : 50) * dt); R.yaw = R.pitch = R.roll = 0; if (R.faceT > 0) faceTarget(dt); }
  const a = Math.abs(R.speed);
  if (!S.easy && !R.busy) { if (a > 200) S.fuel = Math.max(0, S.fuel - dt * (a / stat.pulse) * 1.1); else if (a > 1) S.fuel = Math.max(0, S.fuel - dt * 0.05 * a / stat.manual); }
  R.vel.copy(fwdOf(ship, _v)).multiplyScalar(R.speed).add(R.bump);
  ship.position.addScaledVector(R.vel, dt); R.bump.multiplyScalar(Math.exp(-dt * 2));
}
function faceTarget(dt) { R.faceT -= dt; if (S.target < 0 || !DEST[S.target]) return; _m.lookAt(ship.position, DEST[S.target].pos, _v3.set(0, 1, 0).applyQuaternion(ship.quaternion)); _q.setFromRotationMatrix(_m); ship.quaternion.rotateTowards(_q, 0.25 * dt); }
function autopilot(dt, fuelOK) {
  const d = DEST[S.target]; if (!d) { R.autopilot = false; return; } const distC = ship.position.distanceTo(d.pos);
  const aim = _v4.copy(ship.position).sub(d.pos).normalize().multiplyScalar(d.stop).add(d.pos);
  const dist = ship.position.distanceTo(aim);
  _m.lookAt(ship.position, aim, _v3.set(0, 1, 0).applyQuaternion(ship.quaternion)); _q.setFromRotationMatrix(_m);
  const ang = ship.quaternion.angleTo(_q); ship.quaternion.rotateTowards(_q, dt * 0.45);
  let tgt = Math.min(fuelOK ? stat.pulse : 120, Math.sqrt(2 * 140 * Math.max(0, dist - 20)) + 6);
  if (ang > 0.3) tgt = Math.min(tgt, ang > 0.9 ? 40 : 140);
  R.speed = approach(R.speed, tgt, (tgt > R.speed ? 170 : 320) * dt);
  if (dist < 60 || distC < d.stop + 20) {
    R.autopilot = false; R.throttle = 0; R.faceT = 6;
    const extra = d.type === 'field' ? ' Lenda sisse ja tulista kive!' : d.type === 'station' ? ' Müü kristalle juhtpaneelilt.' : d.planet.kind === 'gas' ? ' Milline vaade!' : ' Vajuta MAANDU!';
    say(`Kohal: ${d.name}.${extra}`);
  }
}
function hitShip(dmg) {
  if (R.time - R.lastHit < 0.7) return; R.lastHit = R.time; if (S.easy) return; S.shield = Math.max(0, S.shield - dmg); sfx.alarm(); haptic('left', 0.8, 150); haptic('right', 0.8, 150);
  say(S.shield > 0 ? 'Ettevaatust! Kilp sai pihta.' : 'Kilp on tühi! Lenda ettevaatlikult.');
}
function shipCollisions() {
  if (R.landed || SURF || R.busy === 'hyper') return;
  const P = ship.position, SR = 12;
  if (P.distanceTo(FIELD_POS) < 1500) for (const a of AST) {
    if (!a.alive) continue; const m = a.r + SR; const d2 = P.distanceToSquared(a.pos); if (d2 > m * m) continue;
    const d = Math.sqrt(d2); _v.copy(P).sub(a.pos).normalize(); P.addScaledVector(_v, m - d); R.bump.addScaledVector(_v, Math.max(20, Math.abs(R.speed) * 0.4)); R.speed *= 0.35; hitShip(Math.min(30, 8 + a.r));
  }
  for (const p of PL) { const m = p.r * 1.03 + SR + 20; const d = P.distanceTo(p.pos); if (d < m) { _v.copy(P).sub(p.pos).normalize(); P.addScaledVector(_v, m - d); R.speed = Math.min(R.speed, 20); if (R.time - R.lastWarn > 4 && R.busy !== 'land') { R.lastWarn = R.time; say('Liiga lähedal planeedile!'); } } }
  const L = _v.copy(P).sub(STATION_POS); const ll = L.length(); if (ll > 420) return; const rh = Math.hypot(L.x, L.z); let push = null;
  if (ll < 62 + SR) push = _v2.copy(L).normalize().multiplyScalar(62 + SR - ll);
  else if (rh < 20 + SR && Math.abs(L.y) < 170) push = _v2.set(L.x, 0, L.z).normalize().multiplyScalar(20 + SR - rh);
  else { const dr = rh - 220, dd = Math.hypot(dr, L.y); if (dd < 18 + SR) push = _v2.set(L.x / rh * dr, L.y, L.z / rh * dr).normalize().multiplyScalar(18 + SR - dd); }
  if (push) { P.add(push); R.speed *= 0.4; R.bump.addScaledVector(push.normalize(), 15); hitShip(5); }
}
const OBST = [[-2.8, 2.8, -6.2, -4.95], [-0.65, 0.65, 0.35, 1.65], [-3, -2.25, 0.9, 3.5], [2.3, 3, 2.5, 3.1], [-3, -1.75, 4.9, 6.9], [1.75, 3, 4.9, 6.8], [-3, -1.55, 7.2, 8.8], [1.55, 3, 7.2, 8.8], [-3, -1.8, 8.95, 10]];
function walkable(x, z) {
  const pr = 0.22; if (x < -2.75 || x > 2.75 || z < -4.85 || z > 9.85) return false;
  for (const zp of [-2, 4]) if (Math.abs(z - zp) < 0.08 + pr && Math.abs(x) > 0.9 - pr) return false;
  for (const o of OBST) if (x > o[0] - pr && x < o[1] + pr && z > o[2] - pr && z < o[3] + pr) return false;
  return true;
}
function headDir() { camera.getWorldDirection(_v); return _v; }
function snapTurn(fn) { if (!renderer.xr.isPresenting) return; if (Math.abs(IN.rx) > 0.7) { if (!R.snapLock) { fn(IN.rx > 0 ? -PI / 6 : PI / 6); R.snapLock = true; } } else if (Math.abs(IN.rx) < 0.3) R.snapLock = false; }
function updateWalk(dt) {
  if (R.mode !== 'walk') return;
  headDir(); _v.applyQuaternion(_q.copy(ship.quaternion).invert()); _v.y = 0; if (_v.lengthSq() < 1e-4) _v.set(0, 0, -1); _v.normalize();
  const rx = -_v.z, rz = _v.x, sp = 2.4 * dt;
  const mx = (_v.x * -IN.ly + rx * IN.lx) * sp, mz = (_v.z * -IN.ly + rz * IN.lx) * sp; const p = rig.position;
  if (walkable(p.x + mx, p.z + mz)) { p.x += mx; p.z += mz; } else if (walkable(p.x + mx, p.z)) p.x += mx; else if (walkable(p.x, p.z + mz)) p.z += mz;
  snapTurn(a => rig.rotation.y += a);
  if (p.z > 9.6 && Math.abs(p.x) < 0.75) { exitShip(); if (R.mode === 'walk') p.z = 9.3; }
}
function updateGround(dt) {
  if (R.mode !== 'ground' || !SURF) return;
  headDir(); _v.y = 0; if (_v.lengthSq() < 1e-4) _v.set(0, 0, -1); _v.normalize();
  const rx = -_v.z, rz = _v.x, sp = 4.5 * dt; const p = rig.position;
  let nx = p.x + (_v.x * -IN.ly + rx * IN.lx) * sp, nz = p.z + (_v.z * -IN.ly + rz * IN.lx) * sp;
  nx = clamp(nx, -620, 620); nz = clamp(nz, -620, 620);
  const onRamp = Math.abs(nx) < 1.2 && nz > 10.9;
  if (!onRamp && Math.abs(nx) < 8.6 && nz > -12 && nz < 11.4 && R.jy < 5) { nx = p.x; nz = p.z; }
  if (Math.abs(nx) < 1.2 && nz < 11.0 && nz > 9) { enterShip(); return; }
  if (R.jetHeld) R.jv = Math.min(6, R.jv + 16 * dt); else R.jv -= 9.8 * dt;
  R.jy = clamp(R.jy + R.jv * dt, 0, 30); if (R.jy <= 0) R.jv = Math.max(0, R.jv);
  p.x = nx; p.z = nz; p.y = Math.max(surfaceGround(nx, nz), SURF.water > -900 ? SURF.water : -999) + R.jy;
  snapTurn(a => rig.rotation.y += a);
}
function updateEVA(dt) {
  if (R.mode !== 'eva') return;
  const f = headDir().clone(); const right = new V3().crossVectors(f, _v2.set(0, 1, 0).applyQuaternion(rig.quaternion)).normalize(); const up = new V3(0, 1, 0).applyQuaternion(rig.quaternion);
  const acc = new V3().addScaledVector(f, -IN.ly).addScaledVector(right, IN.lx).addScaledVector(up, -IN.ry * (renderer.xr.isPresenting ? 1 : -1)).multiplyScalar(R.jetHeld ? 12 : 6);
  R.ev.addScaledVector(acc, dt).multiplyScalar(Math.exp(-dt * 0.7)); if (R.ev.length() > 15) R.ev.setLength(15);
  const toShip = _v3.copy(ship.position).sub(rig.position); if (toShip.length() > 220) R.ev.addScaledVector(toShip.normalize(), 20 * dt);
  rig.position.addScaledVector(R.ev, dt);
  const L = ship.worldToLocal(rig.position.clone());
  if (Math.abs(L.x) < 4.4 && L.y > -2.4 && L.y < 4.4 && L.z > -11.8 && L.z < 11.2) {
    const pen = [4.4 - Math.abs(L.x), Math.min(L.y + 2.4, 4.4 - L.y), Math.min(L.z + 11.8, 11.2 - L.z)]; const i = pen.indexOf(Math.min(...pen));
    if (i === 0) L.x = Math.sign(L.x) * 4.45; else if (i === 1) L.y = L.y + 2.4 < 4.4 - L.y ? -2.45 : 4.45; else L.z = L.z + 11.8 < 11.2 - L.z ? -11.85 : 11.25;
    rig.position.copy(ship.localToWorld(L)); R.ev.multiplyScalar(0.3);
  }
  if (L.z > 11 && L.z < 14 && Math.abs(L.x) < 1.6 && Math.abs(L.y - 1.2) < 1.8 && R.ev.dot(fwdOf(ship, _v4)) > 0.3) enterShip();
  if (renderer.xr.isPresenting) snapTurn(a => rig.rotateY(a));
  for (const c of CRYS) if (c.life > 0 && c.m.position.distanceTo(rig.position) < 4) { c.life = 0; c.m.visible = false; S.crystals += c.value; S.mined += c.value; sfx.pick(); }
}
function updateTool(dt) {
  const beam = R.mode === 'ground' && SURF ? SURF.beam : R.mode === 'eva' ? EVA_BEAM : null;
  if (!beam) return;
  beam.visible = R.toolHeld; if (!R.toolHeld) { R.scanObj = null; R.scanT = 0; return; }
  const o = new V3(), d = new V3();
  const rc = ctrls.find(c => c.userData.hand === 'right' && c.userData.src);
  if (renderer.xr.isPresenting && rc) ctrlRay(rc, o, d); else { camera.getWorldPosition(o); camera.getWorldDirection(d); o.addScaledVector(_v.set(0, -0.25, 0), 1); }
  let best = null, bt = R.mode === 'eva' ? 120 : 60;
  const test = (c, rad, obj) => { const t = _v.copy(c).sub(o).dot(d); if (t < 0 || t > bt) return; if (_v2.copy(o).addScaledVector(d, t).distanceTo(c) < rad) { bt = t; best = obj; } };
  if (R.mode === 'ground') { for (const c of SURF.creatures) test(_v4.copy(c.g.position).add(_v3.set(0, (c.sp.cy + 0.3) * c.sp.size, 0)), Math.max(1.2, c.sp.size * 1.3), c); for (const dp of SURF.deposits) if (dp.alive) test(dp.c, 2.2, dp); }
  else for (const a of AST) if (a.alive) test(a.pos, a.r, a);
  const end = o.clone().addScaledVector(d, bt);
  const pa = beam.geometry.attributes.position; const lo = rig.worldToLocal(o.clone()), le = rig.worldToLocal(end);
  pa.setXYZ(0, lo.x, lo.y, lo.z); pa.setXYZ(1, le.x, le.y, le.z); pa.needsUpdate = true; beam.geometry.computeBoundingSphere();
  if (best && best.sp) {
    beam.material.color.set('#7dff9a');
    if (R.scanObj !== best) { R.scanObj = best; R.scanT = 0; }
    if (R.scanT >= 0) { R.scanT += dt; if (Math.random() < dt * 10) sfx.scan(); haptic('right', 0.15, 20);
      if (R.scanT > 1.2) { R.scanT = -99; const key = `${S.system}:${SURF.planet.name}:${best.sp.name}`; if (S.scanned.includes(key)) say(`Juba skaneeritud: ${best.sp.name}`); else { S.scanned.push(key); S.credits += 25; sfx.fanfare(); sfx.chirp(); say(`Uus liik: ${best.sp.name}! +25 kr`); save(); } } }
  } else if (best && best.idx) {
    beam.material.color.set('#ffa040'); best.hp -= dt * 1.5 * stat.dmg; haptic('right', 0.3, 20); if (Math.random() < dt * 8) sfx.hit();
    if (best.hp <= 0) { best.alive = false; for (const i of best.idx) SURF.depIM.setMatrixAt(i, _m.makeScale(0, 0, 0)); SURF.depIM.instanceMatrix.needsUpdate = true; S.crystals += best.val; S.mined += best.val; sfx.pick(); say(`+${best.val} kristalli!`); }
  } else if (best && best.im) {
    beam.material.color.set('#ffa040'); best.hp -= dt * 2 * stat.dmg; if (Math.random() < dt * 8) sfx.hit(); if (best.hp <= 0) breakRock(best);
  } else { beam.material.color.set('#7ff0ff'); R.scanObj = null; }
}
function checkProximity() {
  if (R.landed || SURF || R.busy === 'hyper' || R.busy === 'takeoff2') { R.nearStation = false; R.nearPlanet = null; return; }
  const P = ship.position;
  const near = P.distanceTo(STATION_POS) < 1100; if (near && !R.nearStation) say('Tere tulemast jaama!'); R.nearStation = near;
  const inF = P.distanceTo(FIELD_POS) < 1300; if (inF && !R.inField) say('Asteroidiväli! Tulista kive ja korja kristallid.'); R.inField = inF;
  if (R.busy !== 'land') { R.nearPlanet = null; for (const p of PL) if (P.distanceTo(p.pos) < p.r * 2.2 + 800) R.nearPlanet = p; }
  for (const p of PL) if (!S.discovered.includes(discKey(p)) && P.distanceTo(p.pos) < p.r * 2.6 + 600) { S.discovered.push(discKey(p)); S.credits += 50; sfx.fanfare(); say(`Uus planeet avastatud: ${p.name}! +50 kr`); save(); }
  if (S.fuel < 15 && !R.lowFuel) { R.lowFuel = true; say('Kütus on madal! Rafineeri kristalle lastiruumis.'); } else if (S.fuel > 30) R.lowFuel = false;
  if (R.time - R.lastHit > 3 && S.shield < stat.maxShield) S.shield = Math.min(stat.maxShield, S.shield + 0.8);
}
function updateShipFX(dt) {
  const f = clamp(Math.abs(R.speed) / stat.pulse, 0, 1), base = (R.busy === 'land2' || R.busy === 'takeoff') ? 1.3 : R.landed ? 0.2 : 0.35 + clamp(Math.abs(R.speed) / stat.manual, 0, 1) * 0.5 + f * 1.6;
  for (const g of shipFX.glows) g.scale.setScalar(g.userData.base * base * (0.92 + Math.random() * 0.16));
  shipFX.stick.rotation.set(-IN.ry * 0.35, 0, -IN.rx * 0.35); shipFX.lever.rotation.x = -R.throttle * 0.6;
  shipFX.reticle.visible = flying();
  shipFX.tank.count = Math.min(40, S.crystals);
}

/* =========================================================
   KÄIVITAMINE
   ========================================================= */
let ready = false; const clock = new THREE.Clock(); let accUI = 0, accP = 0;
function loop() {
  const dt = Math.min(clock.getDelta(), 0.05); if (!ready) { renderer.render(activeScene, camera); return; }
  step(dt); renderer.render(activeScene, camera);
}
function step(dt) {
  R.time += dt;
  readXR(); readKeys(); updateControls(dt);
  updateLandSeq(dt); updateFlight(dt); shipCollisions(); updateHyper(dt); ship.updateMatrixWorld(true);
  updateWalk(dt); updateGround(dt); updateEVA(dt);
  updateLasers(dt); updateAsteroids(dt); updateCrystals(dt); updateNPCs(dt); updateSurface(dt);
  station.ring.rotation.y += dt * 0.03; for (const b of station.beacons) b.visible = (R.time % 2) < 0.25;
  for (const p of PL) { p.mesh.rotation.y += dt * 0.004; if (p.cloud) p.cloud.rotation.y += dt * 0.002; }
  sky.position.copy(ship.position); if (!R.landed) updateDust();
  activeScene.updateMatrixWorld();
  updateTool(dt); updateHolo(); updateMarker(); updateShipFX(dt);
  accP += dt; if (accP > 0.4) { accP = 0; checkProximity(); }
  if (R.resetArm > 0) R.resetArm -= dt;
  updatePointers();
  accUI += dt; if (accUI > 0.2) { accUI = 0; for (const p of PANELS) p.check(); }
  updateMsg(dt); updateFade(dt); updateAudio(); updateMusic(dt);
}
renderer.setAnimationLoop(loop);

async function init() {
  setLoad(0.05, 'Süütan tähed…'); await nextFrame();
  const pm = new THREE.PMREMGenerator(renderer); const es = new THREE.Scene();
  const ec = cnv(256, 128), eg = ec.getContext('2d'), gr = eg.createLinearGradient(0, 0, 0, 128); gr.addColorStop(0, '#25305e'); gr.addColorStop(0.5, '#0b0e20'); gr.addColorStop(1, '#22102c'); eg.fillStyle = gr; eg.fillRect(0, 0, 256, 128);
  es.add(new THREE.Mesh(new THREE.SphereGeometry(50, 32, 16), new THREE.MeshBasicMaterial({ map: ctex(ec), side: THREE.BackSide })));
  const es2 = new THREE.Mesh(new THREE.SphereGeometry(6, 16, 8), new THREE.MeshBasicMaterial({ color: new THREE.Color(40, 34, 26) })); es2.position.copy(SUN_DIR).multiplyScalar(40); es.add(es2);
  scene.environment = pm.fromScene(es, 0.04).texture;
  buildSky();
  setLoad(0.12, 'Ehitan kosmosejaama…'); await nextFrame(); buildStation();
  setLoad(0.18, 'Paigutan asteroide…'); await nextFrame(); buildRocks();
  await buildSystem(S.system, true);
  setLoad(0.9, 'Ehitan laevu…'); await nextFrame();
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
  say(S.mined > 0 || S.discovered.length ? 'Tere tagasi, kapten!' : 'Tere, kapten! Vasaku käe randmel on menüü.');
  window.G = { S, R, ship, rig, camera, DEST, PL, setMode, setTarget, toggleAutopilot, toggleSeat, startLanding, takeoff, startHyper, exitShip, enterShip, keys, sim: (sec) => { for (let t = 0; t < sec; t += 0.05) step(0.05); } };
}
init();
</script>
</body>
</html>
