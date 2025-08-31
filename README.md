 LED_Calculator_v2_1_3_PRO

<html lang="es">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Calculadora LED — v2.1.3 (Simple)</title>
<style>
  :root{ --bg:#0b0f14; --card:#121821; --ink:#e7eef7; --muted:#9fb1c7; --brand:#00c2ff; --accent:#7cff6b; }
  *{box-sizing:border-box}
  body{margin:0;background:var(--bg);color:var(--ink);font-family:system-ui,-apple-system,Segoe UI,Roboto,Inter,Arial,sans-serif}
  .wrap{max-width:1000px;margin:0 auto;padding:16px;display:grid;grid-template-columns:1fr 1fr;gap:16px}
  h1{grid-column:1/-1;margin:0 0 6px;font-size:clamp(20px,3.2vw,30px)}
  .muted{grid-column:1/-1;color:var(--muted);margin:0 0 12px}
  .card{background:var(--card);border:1px solid #1b2430;border-radius:14px;padding:14px}
  label{display:block;margin-top:10px;font-weight:600}
  select,input{width:100%;padding:10px;border-radius:10px;border:1px solid #2a3646;background:#0e151d;color:var(--ink);margin-top:6px}
  .row{display:grid;grid-template-columns:1fr 1fr;gap:10px}
  .toolbar{display:flex;gap:8px;flex-wrap:wrap;margin-top:12px}
  button{cursor:pointer;border:0;padding:10px 14px;border-radius:12px;background:var(--brand);color:#001014;font-weight:700}
  button.ghost{background:transparent;border:1px solid #2a3646;color:var(--ink)}
  button.secondary{background:var(--accent)}
  .result-grid{display:grid;grid-template-columns:1fr;gap:8px}
  .kv{background:#0e151d;border:1px solid #1c2836;border-radius:12px;padding:10px}
  .kv b{display:block;color:var(--muted);font-size:12px}
  .kv span{font-size:18px}
  canvas{width:100%;height:auto;border:1px dashed #2a3646;border-radius:12px;background:#0e151d;display:block}
  .foot{grid-column:1/-1;color:var(--muted);text-align:center;margin:8px 0 0;font-size:12px}
  @media (max-width:820px){ .wrap{grid-template-columns:1fr} .row{grid-template-columns:1fr} button, select, input{font-size:16px} }
</style>
</head>
<body>
  <div class="wrap">
    <h1>Calculadora de Pantallas LED <span style="color:#9fb1c7;font-weight:400;">— v2.1.3 Pro </span></h1>
    <p class="muted"> Calculadora • Tamaño de pantalla  • Plano legible •  Tensión fija (220V mono / 380V tri).</p>

    <div class="card">
      <h3 style="margin:0 0 6px;">Entrada</h3>
      <label for="moduleType">Modelo de pantalla</label>
      <select id="moduleType"></select>

      <div class="row">
        <div>
          <label for="inputMode">Modo de entrada</label>
          <select id="inputMode">
            <option value="modules">Módulos</option>
            <option value="meters">Medidas (m)</option>
          </select>
        </div>
        <div>
          <label for="pxPerOut">Capacidad por salida (px)</label>
          <input type="number" id="pxPerOut" min="200000" step="10000" value="650000" />
        </div>
      </div>

      <div class="row" id="byModules">
        <div>
          <label for="width">Módulos de ancho</label>
          <input type="number" id="width" min="1" step="1" value="3" />
        </div>
        <div>
          <label for="height">Módulos de alto</label>
          <input type="number" id="height" min="1" step="1" value="2" />
        </div>
      </div>

      <div class="row" id="byMeters" style="display:none;">
        <div>
          <label for="widthM">Ancho deseado (m)</label>
          <input type="number" id="widthM" min="0.5" step="0.01" value="5.0" />
        </div>
        <div>
          <label for="heightM">Alto deseado (m)</label>
          <input type="number" id="heightM" min="0.5" step="0.01" value="3.0" />
        </div>
      </div>

      <div class="row">
        <div>
          <label for="maxWH">Ancho × Alto máx por salida (px)</label>
          <input type="text" id="maxWH" value="3840×3840" />
        </div>
        <div>
          <label for="brightness">Brillo objetivo (nits)</label>
          <input type="number" id="brightness" min="300" step="50" value="1500" />
        </div>
      </div>

      <div class="toolbar">
        <button id="calculateBtn">Calcular</button>
        <button id="copyBtn" class="ghost">Copiar información</button>
        <button id="savePngBtn" class="secondary">Descargar PNG</button>
      </div>
    </div>

    <div class="card" id="diagramCard" style="display:none;">
      <h3 style="margin:0 0 6px;">Plano / Disposición</h3>
      <canvas id="diagram"></canvas>
    </div>

    <div class="card" id="result" style="display:none;">
      <h3 style="margin:0 0 6px;">Resumen</h3>
      <div class="result-grid" id="grid"></div>
    </div>

    <div class="foot">Funciona offline. © EFF COLOR</div>
  </div>

<script>
// Modelos: forzamos 500×1000 siempre vertical (mw=500, mh=1000)
const MODEL_LIST = [
  ["P-8 960×960 Outdoor",             {mw:960, mh:960, pitch:8.0, type:"Outdoor"}],
  ["P-6.66 960×960 Outdoor",          {mw:960, mh:960, pitch:6.66, type:"Outdoor"}],
  ["P-5 960×960 Outdoor",             {mw:960, mh:960, pitch:5.0, type:"Outdoor"}],
  ["P-4 960×960 Outdoor",             {mw:960, mh:960, pitch:4.0, type:"Outdoor"}],
  ["P-4 960×960 Indoor",              {mw:960, mh:960, pitch:4.0, type:"Indoor"}],
  ["P-4 640×480 Indoor",              {mw:640, mh:480, pitch:4.0, type:"Indoor"}],
  ["P-3.9 500×1000 Rental Outdoor",   {mw:500, mh:1000, pitch:3.9, type:"Outdoor", casePanels:6}],
  ["P-3.9 500×500 Rental Outdoor",    {mw:500, mh:500,  pitch:3.9, type:"Outdoor", casePanels:8}],
  ["P-3.9 500×1000 Rental Indoor",    {mw:500, mh:1000, pitch:3.9, type:"Indoor"}],
  ["P-3.03 960×960 Indoor",           {mw:960, mh:960, pitch:3.03, type:"Indoor"}],
  ["P-3.03 640×480 Indoor",           {mw:640, mh:480, pitch:3.03, type:"Indoor"}],
  ["P-2.9 500×1000 Outdoor",          {mw:500, mh:1000, pitch:2.9, type:"Outdoor"}],
  ["P-2.5 640×480 Indoor",            {mw:640, mh:480, pitch:2.5, type:"Indoor"}],
  ["P-1.8 640×480 Indoor",            {mw:640, mh:480, pitch:1.8, type:"Indoor"}],
];
const MODELS = Object.fromEntries(MODEL_LIST);

// UI
const moduleSelect = document.getElementById("moduleType");
MODEL_LIST.forEach(([name])=>{ const o=document.createElement("option"); o.value=name; o.textContent=name; moduleSelect.appendChild(o); });
moduleSelect.value = "P-5 960×960 Outdoor";

function byMetersToModules(wM, hM, cfg){
  const wMm=wM*1000, hMm=hM*1000;
  return {cols: Math.max(1, Math.round(wMm/cfg.mw)), rows: Math.max(1, Math.round(hMm/cfg.mh))};
}
function parseWH(txt){
  const t = (txt||"").toLowerCase().replace(/\s+/g,"").replace("×","x");
  const [w,h] = t.split("x").map(n=>parseInt(n)||0);
  return {maxW: w||3840, maxH: h||3840};
}

// Consumo: Outdoor ref 5000 nits ~ 900 W/m², Indoor ref 1500 nits ~ 450 W/m²
// Se escala linealmente con el brillo. Promedio ≈ 35% del pico.
function powerFor(model, areaM2, nits){
  const isOutdoor = (model.type||"").toLowerCase().includes("outdoor");
  const refNits = isOutdoor ? 5000 : 1500;
  const refWm2  = isOutdoor ? 900  : 450;
  const ratio = Math.max(0.05, Math.min(1, nits/refNits));
  const Wm2 = refWm2 * ratio;
  const W_peak = Math.round(Wm2 * areaM2);
  const W_avg  = Math.round(W_peak * 0.35);
  return {W_peak, W_avg};
}

// Térmicas sugeridas (monofásica @220V y trifásica @380V, PF=0.9)
function breakerSuggest(W_peak){
  const std = [10,16,20,25,32,40,50,63,80,100];
  const Vmono = 220, Vtri = 380, PF=0.9;
  const I_mono = W_peak / Vmono;
  const monoA = std.find(a => I_mono <= a) || std[std.length-1];
  const mono_label = "Monofásica: térmica " + monoA + "A @ 220V (pico ≈ " + I_mono.toFixed(1) + "A)";
  const I_tri = W_peak / (Math.sqrt(3) * Vtri * PF);
  const triA = std.find(a => I_tri <= a) || std[std.length-1];
  const tri_label = "Trifásica: térmica " + triA + "A (pico ≈ " + I_tri.toFixed(1) + "A @ 380V, PF 0.9)";
  const exceedsMono = I_mono > 32;
  return {I_mono, I_tri, monoA, triA, mono_label, tri_label, exceedsMono};
}

// Helpers
function safeModelKey(m){
  return m.replace(/\s+/g,'_').replace(/×/g,'x');
}
function nameFrom(last){
  const m = safeModelKey(last.modelKey);
  const w = Number(last.totalWm).toFixed(2);
  const h = Number(last.totalHm).toFixed(2);
  return `Plano_${m}_${w}x${h}m`;
}

let last = null;
const logo = new Image(); logo.src = "data:image/png;base64,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";

function calculate(){
  const modelKey = moduleSelect.value;
  const cfg = MODELS[modelKey];
  const mode = document.getElementById("inputMode").value;
  const pxPerOut = Math.max(100000, parseInt(document.getElementById("pxPerOut").value)||650000);
  const {maxW, maxH} = parseWH(document.getElementById("maxWH").value);
  const nits = Math.max(300, parseInt(document.getElementById("brightness").value)||1500);

  let cols, rows;
  if(mode==="meters"){
    const wM = parseFloat(document.getElementById("widthM").value)||1;
    const hM = parseFloat(document.getElementById("heightM").value)||1;
    const cr = byMetersToModules(wM,hM,cfg); cols=cr.cols; rows=cr.rows;
  }else{
    cols = Math.max(1, parseInt(document.getElementById("width").value)||1);
    rows = Math.max(1, parseInt(document.getElementById("height").value)||1);
  }

  const totalWmm = cols*cfg.mw, totalHmm = rows*cfg.mh;
  const totalWm = +(totalWmm/1000).toFixed(2), totalHm = +(totalHmm/1000).toFixed(2);
  const area = +(totalWm*totalHm).toFixed(2);
  const pxW = Math.round(totalWmm/cfg.pitch), pxH = Math.round(totalHmm/cfg.pitch);
  const pixels = pxW*pxH;

  const needByPixels = Math.max(1, Math.ceil(pixels/pxPerOut));
  const needByW = Math.max(1, Math.ceil(pxW/maxW));
  const needByH = Math.max(1, Math.ceil(pxH/maxH));
  const outputs = Math.max(needByPixels, needByW, needByH);

  let rec = "";
  if(outputs<=1) rec = "1 salida: TB1 / TB2 (NovaStar) o MSD300";
  else if(outputs===2) rec = "2 salidas: Serie Taurus (TB3–TB6) o 2×MSD300/MSD600";
  else if(outputs<=4) rec = outputs+" salidas: considerar VX600 (o 2×MSD600)";
  else rec = outputs+" salidas: considerar VX1000 (o múltiples VX600)";

  const {W_peak, W_avg} = powerFor(cfg, area, nits);
  const b = breakerSuggest(W_peak);

  last = {modelKey, cfg, cols, rows, totalWm, totalHm, area, pxW, pxH, pixels, outputs, rec,
          pxPerOut, maxW, maxH, nits, W_peak, W_avg, breaker: b};
  render();
}

function render(){
  if(!last) return;
  const grid = document.getElementById("grid");
  grid.innerHTML = "";
  const lines = [
    ["Modelo", last.modelKey],
    ["Tamaño panel (mm)", `${last.cfg.mw} × ${last.cfg.mh}`],
    ["Pitch (mm)", last.cfg.pitch],
    ["Módulos (ancho × alto)", `${last.cols} × ${last.rows}`],
    ["Dimensiones (m)", `${last.totalWm} × ${last.totalHm}`],
    ["Área (m²)", last.area],
    ["Resolución (px)", `${last.pxW} × ${last.pxH}`],
    ["Píxeles totales", last.pixels.toLocaleString()],
    ["Outputs estimados", String(last.outputs)],
    ["Sugerencia de envío/procesador", last.rec],
    ["Brillo objetivo (nits)", last.nits],
    ["Consumo promedio (W)", last.W_avg.toLocaleString()],
    ["Consumo pico (W)", last.W_peak.toLocaleString()],
    ["Térmica monofásica", last.breaker.mono_label + (last.breaker.exceedsMono ? " • Excede 32A, preferir trifásica" : "")],
    ["Térmica trifásica", last.breaker.tri_label],
    ["Límites por salida (px)", `${last.maxW} × ${last.maxH}; ${last.pxPerOut.toLocaleString()} px`],
  ];
  for(const [k,v] of lines){
    const el = document.createElement("div"); el.className="kv";
    el.innerHTML = `<b>${k}</b><span>${v}</span>`;
    grid.appendChild(el);
  }
  document.getElementById("result").style.display="block";
  drawDiagram();
}

// Helpers for dynamic sizing
function clamp(x, a, b){ return Math.max(a, Math.min(b, x)); }

function drawDiagram(){
  const cvs = document.getElementById("diagram");
  const parentW = cvs.parentElement.clientWidth - 2;
  const baseH = Math.max(320, Math.round(parentW * 0.64)); // alto para legibilidad
  const dpr = Math.max(1, window.devicePixelRatio || 1);
  cvs.width = Math.round(parentW * dpr);
  cvs.height = Math.round(baseH * dpr);
  cvs.style.width = parentW + "px";
  cvs.style.height = baseH + "px";
  const ctx = cvs.getContext("2d");
  ctx.setTransform(1,0,0,1,0,0);
  ctx.scale(dpr, dpr);
  ctx.clearRect(0,0,parentW,baseH);
  ctx.fillStyle = "#0e151d"; ctx.fillRect(0,0,parentW,baseH);

  const mw = last.cfg.mw, mh = last.cfg.mh;
  const cols = last.cols, rows = last.rows;
  const gridWmm = cols*mw, gridHmm = rows*mh;

  // padding adaptativo
  const padBase = 78;
  const pad = gridWmm*gridHmm < 1_000_000 ? 60 : padBase; // si < ~1m², menos margen
  const top = 56, bottom = 36, left = 30, right = 14;
  const availW = parentW - (pad + right);
  const availH = baseH - (pad + bottom + top);
  const scale = Math.min(availW / gridWmm, availH / gridHmm);

  const gridPxW = gridWmm * scale;
  const gridPxH = gridHmm * scale;
  const ox = Math.max(12, (parentW - gridPxW) / 2);
  const oy = Math.max(top + 8, (baseH - gridPxH) / 2);

  // módulos
  const cellW = mw * scale, cellH = mh * scale;
  for(let y=0;y<rows;y++){
    for(let x=0;x<cols;x++){
      const rx = ox + x*cellW, ry = oy + y*cellH;
      ctx.fillStyle = "#162230";
      ctx.fillRect(rx, ry, Math.max(1, cellW-2), Math.max(1, cellH-2));
      ctx.strokeStyle = "#2a3646"; ctx.lineWidth = 1;
      ctx.strokeRect(rx+0.5, ry+0.5, Math.max(0.5, cellW-3), Math.max(0.5, cellH-3));
    }
  }

  // Tamaño de panel dentro del primer módulo
  ctx.save();
  ctx.fillStyle = "#a8c1dc";
  const fontSize = clamp(Math.min(cellW, cellH) * 0.22, 9, 16);
  ctx.font = `${fontSize}px system-ui, Arial`;
  ctx.textAlign = "center";
  ctx.fillText(`${mw}×${mh} mm`, ox + cellW/2, oy + cellH/2 + fontSize*0.35);
  ctx.restore();

  // Reglas/cotas
  ctx.strokeStyle = "#2a3646"; ctx.lineWidth = 2;
  ctx.beginPath(); ctx.moveTo(ox, oy+gridPxH+10); ctx.lineTo(ox+gridPxW, oy+gridPxH+10); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(ox-10, oy); ctx.lineTo(ox-10, oy+gridPxH); ctx.stroke();

  // Etiquetas
  ctx.fillStyle = "#9fb1c7"; ctx.font = "12px system-ui, Arial"; ctx.textAlign = "left";
  ctx.fillText(`Resolución: ${last.pxW} × ${last.pxH} px`, ox, oy-14);
  ctx.textAlign = "center";
  ctx.fillText(`${cols} módulos  •  ${last.totalWm} m`, ox + gridPxW/2, oy+gridPxH+28);
  ctx.save(); ctx.translate(ox-34, oy + gridPxH/2); ctx.rotate(-Math.PI/2);
  ctx.fillText(`${rows} módulos  •  ${last.totalHm} m`, 0, 0); ctx.restore();

  // Watermark centrado proporcional
  function drawWM(){
    const nW = logo.naturalWidth || logo.width || 100;
    const nH = logo.naturalHeight || logo.height || 100;
    const ratio = nH / nW;
    const base = Math.min(gridPxW, gridPxH);
    const w = Math.max(120, Math.min(base * 0.55, gridPxW * 0.75));
    const h = w * ratio;
    const wx = ox + (gridPxW - w)/2;
    const wy = oy + (gridPxH - h)/2;
    ctx.globalAlpha = 0.20;
    ctx.drawImage(logo, wx, wy, w, h);
    ctx.globalAlpha = 1.0;
  }
  if(logo.complete) drawWM(); else logo.onload = drawWM;

  document.getElementById("diagramCard").style.display = "block";
}

function copyInfo(){
  if(!last){ alert("Primero calcula."); return; }
  const b = last.breaker;
  const header = nameFrom(last);
  const lines = [
    header,
    `MODELO: ${last.modelKey}`,
    `TAMAÑO PANEL (mm): ${last.cfg.mw} × ${last.cfg.mh}`,
    `PITCH (mm): ${last.cfg.pitch}`,
    `MÓDULOS (ANCHO × ALTO): ${last.cols} × ${last.rows}`,
    `DIMENSIONES (m): ${last.totalWm} × ${last.totalHm}`,
    `ÁREA (m²): ${last.area}`,
    `RESOLUCIÓN (px): ${last.pxW} × ${last.pxH}`,
    `PÍXELES TOTALES: ${last.pixels}`,
    `OUTPUTS ESTIMADOS: ${last.outputs}`,
    `SUGERENCIA: ${last.rec}`,
    `BRILLO OBJETIVO (nits): ${last.nits}`,
    `CONSUMO PROMEDIO (W): ${last.W_avg}`,
    `CONSUMO PICO (W): ${last.W_peak}`,
    `TÉRMICA MONOFÁSICA: ${b.mono_label}`,
    `TÉRMICA TRIFÁSICA: ${b.tri_label}`,
    `LÍMITES POR SALIDA (px): ${last.maxW} × ${last.maxH}; ${last.pxPerOut} px`
  ];
  navigator.clipboard.writeText(lines.join("\n")).then(()=>{ alert("Información copiada."); });
}

document.getElementById("inputMode").addEventListener("change", ()=>{
  const m = document.getElementById("inputMode").value;
  document.getElementById("byModules").style.display = (m==="modules"?"grid":"none");
  document.getElementById("byMeters").style.display = (m==="meters"?"grid":"none");
});
document.getElementById("calculateBtn").addEventListener("click", calculate);
document.getElementById("copyBtn").addEventListener("click", copyInfo);
document.getElementById("savePngBtn").addEventListener("click", ()=>{
  if(!last){ alert("Primero calcula."); return; }
  const cvs = document.getElementById("diagram");
  const link = document.createElement("a");
  link.download = nameFrom(last) + ".png";  // << nombre dinámico con punto decimal
  link.href = cvs.toDataURL("image/png");
  link.click();
});

let resizeTO=null;
window.addEventListener("resize", ()=>{
  if(!last) return;
  clearTimeout(resizeTO);
  resizeTO = setTimeout(drawDiagram, 120);
});
</script>
</body>
</html>
