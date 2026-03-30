<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>高中化学 · 原电池微观探究实验</title>
    <style>
        *{margin:0;padding:0;box-sizing:border-box}
        body{font-family:'Microsoft YaHei',sans-serif;background:#0a0e17;color:#e6edf3;overflow:hidden}
        .top{background:linear-gradient(135deg,#0d1520,#131d2e);padding:15px;text-align:center;border-bottom:1px solid #1e2d3d}
        .top h1{font-size:1.8em;color:#58a6ff;text-shadow: 0 0 10px rgba(88,166,255,0.4)}
        .mode-tabs{display:flex;justify-content:center;gap:15px;margin-top:15px}
        .mode-tab{padding:10px 35px;border:1px solid #30363d;background:#0d1117;color:#8b949e;border-radius:25px;cursor:pointer;font-size:1.1em;transition:all .3s}
        .mode-tab.active{background:#1a3a5c;border-color:#58a6ff;color:#58a6ff;font-weight:bold;box-shadow: 0 0 15px rgba(88,166,255,0.3)}
        
        .main{display:flex;height:calc(100vh - 120px)}
        .canvas-area{flex:1;position:relative;background: radial-gradient(circle at center, #101825 0%, #0a0e17 100%); cursor: pointer;}
        canvas{display:block}

        .panel{width:360px;background:#0d1117;border-left:1px solid #1e2d3d;padding:20px;overflow-y:auto}
        .panel h3{color:#58a6ff;font-size:1.1em;margin:15px 0 10px;border-left:4px solid #58a6ff;padding-left:10px}
        .sel-row{display:flex;align-items:center;gap:10px;margin:12px 0}
        .sel-row select{flex:1;padding:10px;border:1px solid #30363d;background:#161b22;color:#e6edf3;border-radius:6px;font-size:1em}
        
        .speed-ctrl{background:#161b22;padding:15px;border-radius:8px;margin:15px 0}
        .speed-ctrl label{display:block;margin-bottom:8px;color:#8b949e;font-size:0.9em}
        .speed-ctrl input{width:100%; cursor: pointer; accent-color: #58a6ff;}

        .btn-row{display:flex;gap:10px;margin:10px 0}
        .btn-row button{flex:1;padding:12px;border:1px solid #30363d;background:#161b22;color:#c9d1d9;border-radius:6px;font-size:0.95em;cursor:pointer;transition:all 0.2s}
        .btn-on{background:#1a472a!important;border-color:#2ea043!important;color:#3fb950!important;font-weight:bold}
        .btn-off{background:#3d1a1a!important;border-color:#da3633!important;color:#f85149!important;font-weight:bold}
        .btn-micro-active{background:#1a3a5c!important;border-color:#58a6ff!important;color:#58a6ff!important;font-weight:bold;box-shadow: 0 0 10px rgba(88,166,255,0.2)}

        .info-box{background:#0d1520;border-radius:8px;padding:15px;margin:10px 0;line-height:1.6;border-left:4px solid #388bfd;font-size:0.95em}
        .error-box{border-left-color: #f85149 !important; background: #2a1111 !important;}
        .legend{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:15px;font-size:0.85em}
        .legend-item{display:flex;align-items:center;gap:8px}
        .dot{width:12px;height:12px;border-radius:50%}
    </style>
</head>
<body>

<div class="top">
    <h1>⚡ 原电池微观探究实验</h1>
    <div class="mode-tabs">
        <div id="tab-single" class="mode-tab active" onclick="setMode('single')">🧪 单液原电池</div>
        <div id="tab-double" class="mode-tab" onclick="setMode('double')">⚗️ 双液原电池 (U型盐桥)</div>
    </div>
</div>

<div class="main">
    <div class="canvas-area" id="container"><canvas id="cv"></canvas></div>
    <div class="panel">
        <h3>配置电极</h3>
        <div class="sel-row"><label>负极:</label><select id="selNeg" onchange="initExperiment()"></select></div>
        <div class="sel-row"><label>正极:</label><select id="selPos" onchange="initExperiment()"></select></div>

        <h3>控制台</h3>
        <div class="btn-row">
            <button id="btnSwitch" class="btn-off" onclick="toggleSwitch()">闭合开关</button>
            <button id="btnMicro" onclick="toggleMicro()">👁 开启微观探测</button>
        </div>
        <div class="btn-row">
            <button onclick="initExperiment()" style="background:#21262d">🔄 重置实验</button>
        </div>

        <h3>调节观察速度</h3>
        <div class="speed-ctrl">
            <label id="speedLabel">播放速度: 1.0x</label>
            <input type="range" id="speedRange" min="0.1" max="3" step="0.1" value="1.0" oninput="updateSpeed()">
        </div>

        <div class="info-box" id="dataPanel">请闭合开关</div>
        <div id="theoryPanel" class="info-box" style="border-left-color:#7ee787">等待探究开始...</div>

        <h3>图例说明</h3>
        <div class="legend" id="legendBox"></div>
    </div>
</div>

<script>
const cv = document.getElementById('cv');
const ctx = cv.getContext('2d');
let W, H;

const electrodes = {
    Mg: { name: '镁 Mg', color: '#e0e0e0', ion: 'Mg²⁺', ionColor: '#a5d6a7', v: -2.37 },
    Zn: { name: '锌 Zn', color: '#9e9e9e', ion: 'Zn²⁺', ionColor: '#4fc3f7', v: -0.76 },
    Fe: { name: '铁 Fe', color: '#78909c', ion: 'Fe²⁺', ionColor: '#80cbc4', v: -0.44 },
    Cu: { name: '铜 Cu', color: '#d4a24e', ion: 'Cu²⁺', ionColor: '#66bb6a', v: 0.34 },
    Ag: { name: '银 Ag', color: '#c8c8d0', ion: 'Ag⁺', ionColor: '#b0bec5', v: 0.80 },
    C:  { name: '石墨 C', color: '#333333', ion: '', ionColor: '', v: 0.40 }
};

let mode = 'single';
let switchOn = false;
let showMicro = false; // 微观开关状态
let globalSpeed = 1.0;
let negKey = 'Zn', posKey = 'Cu';
let layout = {};
let particles = { ions: [], electrons: [], microEvents: [], saltIons: [] };
let frame = 0;
let needleAngle = 0;

function isCellValid() { return electrodes[negKey].v < electrodes[posKey].v; }

function calcLayout() {
    W = cv.width = document.getElementById('container').clientWidth;
    H = cv.height = document.getElementById('container').clientHeight;
    const midY = H * 0.62;
    if (mode === 'single') {
        const bw = Math.min(W * 0.5, 500), bh = 360;
        layout = {
            beaker: { x: W/2 - bw/2, y: midY - bh/2, w: bw, h: bh, color: '#4fc3f722', label: '稀硫酸 (H₂SO₄)' },
            negElec: { x: W/2 - bw*0.28, y: midY - bh*0.4, w: 45, h: bh*0.75 },
            posElec: { x: W/2 + bw*0.28, y: midY - bh*0.4, w: 45, h: bh*0.75 },
            wireTop: H * 0.12, ammeter: { x: W/2, y: H * 0.12, r: 65 }, switchPos: { x: W * 0.28, y: H * 0.12 }
        };
    } else {
        const bw = Math.min(W * 0.35, 380), bh = 360;
        const bL_x = W * 0.25 - bw/2, bR_x = W * 0.75 - bw/2;
        layout = {
            leftBeaker: { x: bL_x, y: midY - bh/2, w: bw, h: bh, color: '#7cb34222', label: negKey+'SO₄ 溶液' },
            rightBeaker: { x: bR_x, y: midY - bh/2, w: bw, h: bh, color: '#03a9f422', label: posKey+'SO₄ 溶液' },
            negElec: { x: W*0.25, y: midY - bh*0.4, w: 45, h: bh*0.75 },
            posElec: { x: W*0.75, y: midY - bh*0.4, w: 45, h: bh*0.75 },
            uBridge: { leftX: bL_x + bw * 0.8, rightX: bR_x + bw * 0.2, topY: midY - bh/2 - 40, bottomY: midY - bh/2 + 60, width: 48 },
            wireTop: H * 0.12, ammeter: { x: W/2, y: H * 0.12, r: 65 }, switchPos: { x: W * 0.32, y: H * 0.12 }
        };
    }
}

function initParticles() {
    particles = { ions: [], electrons: [], microEvents: [], saltIons: [] };
    const addIons = (area, label, color, count) => {
        for(let i=0; i<count; i++) {
            particles.ions.push({
                x: area.x + 30 + Math.random()*(area.w-60), y: area.y + area.h*0.35 + Math.random()*(area.h*0.55),
                label, color, vx: (Math.random()-0.5)*0.8, vy: (Math.random()-0.5)*0.8, area
            });
        }
    };
    if (mode === 'single') {
        addIons(layout.beaker, 'H⁺', '#ffa726', 15);
        addIons(layout.beaker, 'SO₄²⁻', '#ef5350', 10);
    } else {
        addIons(layout.leftBeaker, electrodes[negKey].ion, electrodes[negKey].ionColor, 10);
        addIons(layout.leftBeaker, 'SO₄²⁻', '#ef5350', 10);
        addIons(layout.rightBeaker, electrodes[posKey].ion || 'Cu²⁺', electrodes[posKey].ionColor || '#66bb6a', 10);
        addIons(layout.rightBeaker, 'SO₄²⁻', '#ef5350', 10);
        const ub = layout.uBridge;
        const totalLen = (ub.bottomY - ub.topY) * 2 + (ub.rightX - ub.leftX);
        for(let i=0; i<20; i++) {
            particles.saltIons.push({ dist: (i / 20) * totalLen, type: i % 2 === 0 ? 'K⁺' : 'Cl⁻', color: i % 2 === 0 ? '#ce93d8' : '#81c784', offsetY: (Math.random() - 0.5) * 18 });
        }
    }
    for(let i=0; i<12; i++) particles.electrons.push({ t: i/12, speed: 0.005 });
}

function updateAndDrawParticles() {
    if (!showMicro) return; // 微观开关未开启，直接返回
    
    const spd = globalSpeed;
    particles.ions.forEach(ion => {
        ion.x += ion.vx * spd; ion.y += ion.vy * spd;
        if(ion.x < ion.area.x + 15 || ion.x > ion.area.x + ion.area.w - 15) ion.vx *= -1;
        if(ion.y < ion.area.y + ion.area.h*0.35 || ion.y > ion.area.y + ion.area.h - 15) ion.vy *= -1;
        ctx.fillStyle = ion.color + 'cc'; ctx.beginPath(); ctx.arc(ion.x, ion.y, 11, 0, Math.PI*2); ctx.fill();
        ctx.fillStyle = '#fff'; ctx.font = 'bold 11px Arial'; ctx.textAlign='center'; ctx.textBaseline='middle';
        ctx.fillText(ion.label, ion.x, ion.y);
    });

    if (mode === 'double') {
        const ub = layout.uBridge;
        const totalLen = (ub.bottomY - ub.topY) * 2 + (ub.rightX - ub.leftX);
        particles.saltIons.forEach(ion => {
            if (switchOn && isCellValid()) {
                ion.dist += (ion.type === 'K⁺' ? 0.6 : -0.6) * spd;
                if (ion.dist > totalLen) ion.dist = 0; if (ion.dist < 0) ion.dist = totalLen;
            }
            const pos = getUBridgePoint(ion.dist);
            let px = pos.x, py = pos.y;
            if (px === ub.leftX || px === ub.rightX) px += ion.offsetY; else py += ion.offsetY;
            ctx.fillStyle = ion.color; ctx.beginPath(); ctx.arc(px, py, 9, 0, Math.PI*2); ctx.fill();
            ctx.fillStyle = '#000'; ctx.font = 'bold 10px Arial'; ctx.textAlign='center'; ctx.fillText(ion.type, px, py+4);
        });
    }

    if (!switchOn || !isCellValid()) return;

    particles.electrons.forEach(e => {
        e.t += e.speed * spd; if(e.t > 1) e.t = 0;
        const p = getElectronPos(e.t); drawElectron(p.x, p.y, 0.7);
    });

    if (frame % Math.floor(60/spd) === 0) {
        const ne = layout.negElec, pe = layout.posElec;
        particles.microEvents.push({ x: ne.x, y: ne.y + 60 + Math.random()*(ne.h-120), type: 'oxidation', life: 100, ion: electrodes[negKey].ion, color: electrodes[negKey].ionColor });
        particles.microEvents.push({ x: pe.x, y: pe.y + 60 + Math.random()*(pe.h-120), type: 'reduction', life: 100, isH: (mode === 'single' || posKey === 'C') });
    }

    particles.microEvents.forEach((ev, i) => {
        ev.life -= spd; ctx.save(); ctx.globalAlpha = Math.min(1, ev.life / 20);
        const prog = 100 - ev.life;
        if (ev.type === 'oxidation') {
            drawElectron(ev.x - 12, ev.y - prog * 1.2, 0.8); drawElectron(ev.x + 12, ev.y - prog * 1.2, 0.8);
            ctx.fillStyle = ev.color; ctx.beginPath(); ctx.arc(ev.x + prog * 0.8, ev.y, 14, 0, Math.PI*2); ctx.fill();
            ctx.fillStyle = '#fff'; ctx.font = 'bold 12px Arial'; ctx.textAlign='center'; ctx.fillText(ev.ion, ev.x + prog * 0.8, ev.y + 5);
        } else {
            if (ev.isH) {
                if (prog < 50) {
                    ctx.fillStyle = '#ffa726'; ctx.beginPath(); ctx.arc(ev.x + 20, ev.y - 15, 11, 0, Math.PI*2); ctx.fill(); ctx.beginPath(); ctx.arc(ev.x + 20, ev.y + 15, 11, 0, Math.PI*2); ctx.fill();
                    drawElectron(ev.x, ev.y - (50 - prog) * 2, 0.8);
                } else {
                    ctx.strokeStyle = '#fff'; ctx.lineWidth = 2.5; ctx.beginPath(); ctx.arc(ev.x + 25, ev.y - (prog-50)*1.4, 16, 0, Math.PI*2); ctx.stroke();
                    ctx.fillStyle='#fff'; ctx.font='bold 12px Arial'; ctx.fillText('H₂', ev.x+25, ev.y - (prog-50)*1.4 + 5);
                }
            } else {
                drawElectron(ev.x, ev.y - (100-prog), 0.8);
                ctx.fillStyle = '#d4a24e'; ctx.beginPath(); ctx.arc(ev.x + 12, ev.y, 11, 0, Math.PI*2); ctx.fill();
            }
        }
        ctx.restore();
        if(ev.life <= 0) particles.microEvents.splice(i, 1);
    });
}

function getUBridgePoint(dist) {
    const ub = layout.uBridge;
    const h1 = ub.bottomY - ub.topY;
    const w = ub.rightX - ub.leftX;
    if (dist < h1) return { x: ub.leftX, y: ub.bottomY - dist };
    if (dist < h1 + w) return { x: ub.leftX + (dist - h1), y: ub.topY };
    return { x: ub.rightX, y: ub.topY + (dist - h1 - w) };
}

function drawElectron(x, y, scale = 1) {
    ctx.save(); ctx.translate(x, y); ctx.scale(scale, scale); ctx.fillStyle = '#ffeb3b'; ctx.shadowBlur = 10; ctx.shadowColor = '#ffeb3b'; ctx.beginPath(); ctx.arc(0, 0, 11, 0, Math.PI * 2); ctx.fill();
    ctx.fillStyle = '#000'; ctx.font = 'bold 12px Arial'; ctx.textAlign='center'; ctx.textBaseline='middle'; ctx.fillText('e⁻', 0, 0); ctx.restore();
}

function draw() {
    ctx.clearRect(0, 0, W, H);
    if (mode === 'single') drawBeaker(layout.beaker); else { drawBeaker(layout.leftBeaker); drawBeaker(layout.rightBeaker); drawUBridge(); }
    drawElectrode(layout.negElec, negKey, '负极 (氧化)'); drawElectrode(layout.posElec, posKey, '正极 (还原)');
    drawWire(); drawSwitch(); drawMeter();
    updateAndDrawParticles();
    frame++; requestAnimationFrame(draw);
}

function drawUBridge() {
    const ub = layout.uBridge; ctx.save();
    ctx.lineCap = 'round'; ctx.lineJoin = 'round';
    ctx.strokeStyle = 'rgba(200, 200, 230, 0.35)'; ctx.lineWidth = ub.width - 6;
    ctx.beginPath(); ctx.moveTo(ub.leftX, ub.bottomY); ctx.lineTo(ub.leftX, ub.topY); ctx.lineTo(ub.rightX, ub.topY); ctx.lineTo(ub.rightX, ub.bottomY); ctx.stroke();
    ctx.strokeStyle = 'rgba(150, 200, 255, 0.6)'; ctx.lineWidth = 3;
    ctx.beginPath(); ctx.moveTo(ub.leftX - ub.width/2, ub.bottomY); ctx.lineTo(ub.leftX - ub.width/2, ub.topY - ub.width/2); ctx.lineTo(ub.rightX + ub.width/2, ub.topY - ub.width/2); ctx.lineTo(ub.rightX + ub.width/2, ub.bottomY); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(ub.leftX + ub.width/2, ub.bottomY); ctx.lineTo(ub.leftX + ub.width/2, ub.topY + ub.width/2); ctx.lineTo(ub.rightX - ub.width/2, ub.topY + ub.width/2); ctx.lineTo(ub.rightX - ub.width/2, ub.bottomY); ctx.stroke();
    ctx.fillStyle = '#fff'; ctx.font = 'bold 15px Arial'; ctx.textAlign='center'; ctx.fillText("盐桥 (KCl 琼脂)", (ub.leftX+ub.rightX)/2, ub.topY - 45); ctx.restore();
}

function drawBeaker(b) {
    ctx.save(); ctx.fillStyle = b.color; ctx.beginPath(); ctx.roundRect(b.x, b.y + b.h*0.2, b.w, b.h*0.8, [0,0,15,15]); ctx.fill();
    ctx.strokeStyle = 'rgba(255,255,255,0.3)'; ctx.lineWidth = 4; ctx.beginPath(); ctx.roundRect(b.x, b.y, b.w, b.h, [0,0,15,15]); ctx.stroke();
    ctx.fillStyle = '#fff'; ctx.font = 'bold 22px Microsoft YaHei'; ctx.textAlign='center'; ctx.fillText(b.label, b.x + b.w/2, b.y + b.h + 38); ctx.restore();
}

function drawElectrode(e, key, label) {
    const info = electrodes[key]; const grad = ctx.createLinearGradient(e.x - e.w/2, 0, e.x + e.w/2, 0);
    grad.addColorStop(0, '#222'); grad.addColorStop(0.5, info.color); grad.addColorStop(1, '#222');
    ctx.fillStyle = grad; ctx.beginPath(); ctx.roundRect(e.x - e.w/2, e.y, e.w, e.h, 5); ctx.fill();
    ctx.fillStyle = label.includes('负') ? '#f0883e' : '#3fb950'; ctx.font = 'bold 20px Arial'; ctx.textAlign='center'; ctx.fillText(label, e.x, e.y - 52);
    ctx.fillStyle = '#fff'; ctx.fillText(info.name, e.x, e.y - 20);
}

function drawWire() {
    const wt = layout.wireTop; ctx.strokeStyle = switchOn && isCellValid() ? '#ffd54f' : '#444'; ctx.lineWidth = 6; ctx.beginPath();
    ctx.moveTo(layout.negElec.x, layout.negElec.y); ctx.lineTo(layout.negElec.x, wt); ctx.lineTo(layout.posElec.x, wt); ctx.lineTo(layout.posElec.x, layout.posElec.y); ctx.stroke();
}

function drawSwitch() {
    const s = layout.switchPos; ctx.save(); ctx.translate(s.x, s.y); ctx.fillStyle = '#555'; ctx.beginPath(); ctx.arc(-18, 0, 9, 0, Math.PI*2); ctx.fill(); ctx.beginPath(); ctx.arc(18, 0, 9, 0, Math.PI*2); ctx.fill();
    ctx.strokeStyle = switchOn ? (isCellValid() ? '#3fb950' : '#f85149') : '#f85149'; ctx.lineWidth = 7;
    if (switchOn && isCellValid()) { ctx.beginPath(); ctx.moveTo(-18, 0); ctx.lineTo(18, 0); ctx.stroke(); }
    else { ctx.beginPath(); ctx.moveTo(-18, 0); ctx.lineTo(12, -26); ctx.stroke(); } ctx.restore();
}

function drawMeter() {
    const m = layout.ammeter; ctx.save(); ctx.translate(m.x, m.y); ctx.fillStyle = '#111'; ctx.beginPath(); ctx.arc(0,0,m.r,0,Math.PI*2); ctx.fill();
    ctx.strokeStyle = '#555'; ctx.lineWidth = 4; ctx.stroke();
    const target = (switchOn && isCellValid()) ? -Math.PI/4 : 0; needleAngle += (target - needleAngle) * 0.1;
    ctx.rotate(needleAngle - Math.PI/2); ctx.strokeStyle = '#f85149'; ctx.lineWidth = 4; ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(0, -m.r*0.75); ctx.stroke(); ctx.restore();
}

function getElectronPos(t) {
    const ne = layout.negElec, pe = layout.posElec, wt = layout.wireTop;
    if (t < 0.3) return { x: ne.x, y: ne.y - (ne.y - wt) * (t/0.3) };
    else if (t < 0.7) return { x: ne.x + (pe.x - ne.x) * ((t-0.3)/0.4), y: wt };
    else return { x: pe.x, y: wt + (pe.y - wt) * ((t-0.7)/0.3) };
}

function updateSpeed() { globalSpeed = parseFloat(document.getElementById('speedRange').value); document.getElementById('speedLabel').textContent = `播放速度: ${globalSpeed.toFixed(1)}x`; }

function toggleSwitch() {
    switchOn = !switchOn;
    document.getElementById('btnSwitch').className = switchOn ? (isCellValid() ? 'btn-on' : 'btn-off') : 'btn-off';
    document.getElementById('btnSwitch').textContent = switchOn ? '断开开关' : '闭合开关';
    updateTextPanel();
}

function toggleMicro() {
    showMicro = !showMicro;
    const btn = document.getElementById('btnMicro');
    btn.className = showMicro ? 'btn-micro-active' : '';
    btn.textContent = showMicro ? '👁 关闭微观探测' : '👁 开启微观探测';
    updateTextPanel();
}

function setMode(m) { mode = m; switchOn = false; document.getElementById('tab-single').className = m === 'single' ? 'mode-tab active' : 'mode-tab'; document.getElementById('tab-double').className = m === 'double' ? 'mode-tab active' : 'mode-tab'; initExperiment(); }
function initExperiment() { negKey = document.getElementById('selNeg').value; posKey = document.getElementById('selPos').value; switchOn = false; document.getElementById('btnSwitch').className = 'btn-off'; calcLayout(); initParticles(); updateTextPanel(); }

function updateTextPanel() {
    const n = electrodes[negKey], p = electrodes[posKey], d = document.getElementById('dataPanel'), t = document.getElementById('theoryPanel');
    if (!isCellValid()) {
        d.innerHTML = "<b style='color:#f85149'>⚠ 无法形成原电池</b>";
        t.innerHTML = `<div style="color:#f85149"><b>错误提示：</b>负极必须比正极活泼。目前选择无法产生电势差驱动电子流动。</div>`;
        switchOn = false;
    } else {
        if (!switchOn) {
            d.innerHTML = "开关断开，电流为 0";
            t.innerHTML = showMicro ? "微观视角已开启，闭合开关观察反应过程" : "请闭合开关或开启微观探测观察原理";
        } else {
            d.innerHTML = `<b>理论电压:</b> ${(p.v - n.v).toFixed(2)} V<br><b>电子流向:</b> ${negKey} → ${posKey}`;
            let posR = (mode === 'single' || posKey==='C') ? "2H⁺ + 2e⁻ = H₂↑" : (posKey==='Cu'?"Cu²⁺ + 2e⁻ = Cu":`${p.ion} + e⁻ = ${posKey}`);
            t.innerHTML = `<div style="color:#f0883e"><b>负极 (氧化):</b> ${negKey} - 2e⁻ = ${n.ion}</div><div style="color:#3fb950; margin-top:10px"><b>正极 (还原):</b> ${posR}</div>`;
        }
    }
    updateLegend();
}

function updateLegend() {
    if (!showMicro) { document.getElementById('legendBox').innerHTML = ""; return; }
    const ni = electrodes[negKey];
    let h = `<div class="legend-item"><div class="dot" style="background:#ffeb3b"></div> 电子 (e⁻)</div><div class="legend-item"><div class="dot" style="background:#ffa726"></div> 氢离子 (H⁺)</div><div class="legend-item"><div class="dot" style="background:${ni.ionColor}"></div> ${ni.ion}</div><div class="legend-item"><div class="dot" style="background:#ef5350"></div> 硫酸根 (SO₄²⁻)</div>`;
    if(mode === 'double') h += `<div class="legend-item"><div class="dot" style="background:#ce93d8"></div> K⁺ (向正极)</div><div class="legend-item"><div class="dot" style="background:#81c784"></div> Cl⁻ (向负极)</div>`;
    document.getElementById('legendBox').innerHTML = h;
}

cv.addEventListener('mousedown', (e) => { const rect = cv.getBoundingClientRect(); const x = e.clientX - rect.left, y = e.clientY - rect.top; if (Math.abs(x - layout.switchPos.x) < 50 && Math.abs(y - layout.switchPos.y) < 40) toggleSwitch(); });
const sn = document.getElementById('selNeg'), sp = document.getElementById('selPos');
Object.keys(electrodes).forEach(k => { sn.add(new Option(electrodes[k].name, k)); sp.add(new Option(electrodes[k].name, k)); });
sn.value = 'Zn'; sp.value = 'Cu';
window.addEventListener('resize', () => { calcLayout(); initParticles(); });
initExperiment(); draw();
</script>
</body>
</html>
