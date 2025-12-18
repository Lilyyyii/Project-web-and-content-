[final page.html](https://github.com/user-attachments/files/24226468/final.page.html)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SG Air Quality Command Center | Deep Analysis</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&family=Inter:wght@400;600&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/echarts@5.4.3/dist/echarts.min.js"></script>
    
    <style>
        :root {
            --bg: #050505;
            --glass-bg: rgba(20, 20, 25, 0.65);
            --glass-border: 1px solid rgba(255, 255, 255, 0.1);
            --glass-blur: blur(15px);
            --accent: #00f3ff;
            --font-main: 'Inter', sans-serif;
            --font-mono: 'JetBrains Mono', monospace;
        }

        body { margin: 0; background: var(--bg); color: #e0e0e0; font-family: var(--font-main); overflow: hidden; }
        #canvas-wrapper { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; }

        /* UI Layer */
        .ui-layer {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 10;
            padding: 20px; box-sizing: border-box;
            display: grid;
            grid-template-columns: 320px 1fr 320px;
            grid-template-rows: 60px 1fr 80px;
            transition: opacity 0.5s ease, transform 0.5s ease;
        }
        .ui-layer.hidden { opacity: 0; pointer-events: none; transform: scale(1.05); }

        /* Toggle Button */
        #toggle-ui-btn {
            position: absolute; top: 24px; right: 24px; z-index: 100;
            background: rgba(0,0,0,0.5); border: 1px solid rgba(255,255,255,0.3);
            color: #fff; width: 40px; height: 40px; border-radius: 50%;
            cursor: pointer; display: flex; align-items: center; justify-content: center;
            transition: 0.3s; backdrop-filter: blur(5px);
        }
        #toggle-ui-btn:hover { background: #fff; color: #000; }
        #toggle-ui-btn svg { width: 20px; height: 20px; fill: currentColor; }

        /* Header */
        .header { grid-column: 1 / -1; display: flex; justify-content: space-between; align-items: flex-start; pointer-events: auto; }
        .brand h1 { margin: 0; font-size: 16px; font-weight: 700; letter-spacing: 2px; color: #fff; text-transform: uppercase; }
        .brand span { font-size: 10px; color: var(--accent); font-family: var(--font-mono); }
        .status-box { text-align: right; font-family: var(--font-mono); width: 300px; margin-right: 60px; }
        .status-text { font-size: 12px; color: var(--accent); min-height: 1.5em; }

        /* Cards & Interaction */
        .side-panel { display: flex; flex-direction: column; gap: 15px; margin-top: 40px; pointer-events: auto; }
        .left-col { grid-column: 1; }
        .right-col { grid-column: 3; }

        .card {
            background: var(--glass-bg); border: var(--glass-border); border-radius: 12px;
            padding: 16px; backdrop-filter: var(--glass-blur);
            transition: all 0.3s ease; position: relative; cursor: pointer; /* Changed to pointer */
        }
        .card:hover { 
            background: rgba(50,50,60,0.8); transform: translateX(5px); border-color: var(--accent);
            box-shadow: 0 0 20px rgba(0, 243, 255, 0.1); 
        }
        /* Click hint */
        .card::after {
            content: 'CLICK TO ANALYZE'; position: absolute; top: 10px; right: 10px;
            font-size: 8px; color: var(--accent); opacity: 0; transition: 0.3s; font-family: var(--font-mono);
        }
        .card:hover::after { opacity: 1; }
        
        .card-head { font-size: 10px; font-weight: 700; color: #888; margin-bottom: 10px; text-transform: uppercase; display: flex; justify-content: space-between; }
        .card-val { font-size: 24px; font-family: var(--font-mono); font-weight: 700; color: #fff; }

        /* Logs */
        #log-container {
            height: 180px; overflow: hidden; font-family: var(--font-mono); font-size: 10px;
            display: flex; flex-direction: column; justify-content: flex-end; position: relative;
        }
        #log-container::after {
            content: ''; position: absolute; top: 0; left: 0; width: 100%; height: 40%;
            background: linear-gradient(to bottom, var(--glass-bg) 0%, transparent 100%); pointer-events: none;
        }
        .log-line { display: flex; gap: 10px; padding: 2px 0; border-bottom: 1px solid rgba(255,255,255,0.02); animation: slideUp 0.3s ease-out; color: #aaa; }
        .log-line span:nth-child(2) { color: var(--accent); }
        @keyframes slideUp { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* Charts */
        .chart-box { width: 100%; height: 120px; pointer-events: none; }
        .chart-radar-box { width: 100%; height: 200px; pointer-events: none; }

        /* Dock */
        .dock-area { 
            grid-column: 1 / -1; grid-row: 3; 
            display: flex; justify-content: center; align-items: center; gap: 20px; pointer-events: auto;
        }
        .btn-dock {
            background: rgba(0,0,0,0.5); border: 1px solid #333; color: #888;
            padding: 10px 30px; border-radius: 4px; cursor: pointer;
            font-family: var(--font-mono); font-size: 12px; letter-spacing: 1px;
            transition: 0.3s; text-transform: uppercase; position: relative;
        }
        .btn-dock:hover { color: #fff; border-color: #666; background: rgba(255,255,255,0.05); }
        .btn-dock.active { 
            color: #000; background: var(--accent); border-color: var(--accent); 
            box-shadow: 0 0 15px var(--accent); font-weight: 700;
        }
        .progress-bar { position: absolute; bottom: 0; left: 0; height: 2px; background: var(--accent); width: 0%; transition: width 0.1s linear; }

        /* --- Analysis Modal (Deep Dive Layer) --- */
        .analysis-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85); backdrop-filter: blur(30px);
            z-index: 90; display: flex; justify-content: center; align-items: center;
            opacity: 0; pointer-events: none; transition: 0.4s cubic-bezier(0.16, 1, 0.3, 1);
        }
        .analysis-overlay.active { opacity: 1; pointer-events: auto; }
        
        .analysis-modal {
            width: 700px; max-width: 90%; background: #111; border: 1px solid #333;
            border-radius: 12px; padding: 40px; box-shadow: 0 20px 80px rgba(0,0,0,0.8);
            display: grid; grid-template-columns: 1fr 1fr; gap: 40px; position: relative;
        }
        .analysis-modal::before { content: ''; position: absolute; top:0; left:0; width:100%; height:2px; background: var(--accent); }
        
        .analysis-col h3 { font-family: var(--font-mono); font-size: 12px; color: #666; margin-bottom: 20px; }
        .stat-row { display: flex; justify-content: space-between; border-bottom: 1px solid #222; padding: 12px 0; font-size: 14px; }
        .stat-row span:last-child { font-family: var(--font-mono); color: var(--accent); }
        
        .close-btn {
            position: absolute; top: 20px; right: 20px; background: none; border: none; color: #fff; cursor: pointer; font-size: 20px;
        }
        .close-btn:hover { color: var(--accent); }

    </style>
</head>
<body>

    <div id="canvas-wrapper"></div>

    <button id="toggle-ui-btn" onclick="app.toggleUI()" title="Toggle HUD">
        <svg viewBox="0 0 24 24"><path d="M12 4.5C7 4.5 2.73 7.61 1 12c1.73 4.39 6 7.5 11 7.5s9.27-3.11 11-7.5c-1.73-4.39-6-7.5-11-7.5zM12 17c-2.76 0-5-2.24-5-5s2.24-5 5-5 5 2.24 5 5-2.24 5-5 5zm0-8c-1.66 0-3 1.34-3 3s1.34 3 3 3 3-1.34 3-3-1.34-3-3-3z"/></svg>
    </button>

    <div class="analysis-overlay" id="analysis-layer" onclick="if(event.target===this) app.closeAnalysis()">
        <div class="analysis-modal">
            <button class="close-btn" onclick="app.closeAnalysis()">×</button>
            <div class="analysis-col">
                <h3>DATA BREAKDOWN</h3>
                <div id="analysis-content">
                    </div>
            </div>
            <div class="analysis-col">
                <h3>AI INSIGHTS</h3>
                <p style="font-size:13px; color:#888; line-height:1.6;" id="analysis-text">
                    Scanning regional data...
                </p>
                <div style="margin-top:20px; padding:15px; background:rgba(255,255,255,0.05); border-radius:4px;">
                    <div style="font-size:10px; color:#666; margin-bottom:5px;">PREDICTED TREND</div>
                    <div style="color:#fff; font-family:var(--font-mono);" id="analysis-trend">--</div>
                </div>
            </div>
        </div>
    </div>

    <div class="ui-layer" id="ui-layer">
        <div class="header">
            <div class="brand">
                <h1>SG Air Quality Monitor</h1>
                <span>v4.0 // HIGH DENSITY PARTICLE CORE</span>
            </div>
            <div class="status-box">
                <div style="font-size:10px; color:#666; margin-bottom:4px;">SYSTEM MESSAGE</div>
                <div class="status-text" id="typewriter">INITIALIZING...</div>
            </div>
        </div>

        <div class="side-panel left-col">
            <div class="card" onclick="app.showAnalysis('pm25')">
                <div class="card-head"><span>PM2.5 Index</span><span id="region-badge">CENTRAL</span></div>
                <div style="display:flex; align-items:baseline;">
                    <div class="card-val" id="val-pm25">--</div>
                    <span style="font-size:12px; color:#666; margin-left:4px;">µg/m³</span>
                </div>
                <div style="font-size:10px; color:#666; margin-top:5px;" id="val-desc">Calibrating...</div>
            </div>
            <div class="card" onclick="app.showAnalysis('trend')">
                <div class="card-head"><span>24H Trend</span><span>LIVE</span></div>
                <div id="chart-wave" class="chart-box"></div>
            </div>
            <div class="card" style="flex:1;" onclick="app.showAnalysis('log')">
                <div class="card-head"><span>System Log</span><span>SCROLL</span></div>
                <div id="log-container"></div>
            </div>
        </div>

        <div class="side-panel right-col">
            <div class="card" onclick="app.showAnalysis('composition')">
                <div class="card-head"><span>Pollutants</span><span>NOW</span></div>
                <div id="chart-bar" class="chart-box"></div>
            </div>
            <div class="card" onclick="app.showAnalysis('radar')">
                <div class="card-head"><span>Multivariate</span><span>COMPARE</span></div>
                <div id="chart-radar" class="chart-radar-box"></div>
            </div>
        </div>

        <div class="dock-area">
            <div class="progress-bar" id="timeline-bar"></div>
            <button class="btn-dock active" onclick="app.switchRegion('central')" data-region="central">Central</button>
            <button class="btn-dock" onclick="app.switchRegion('east')" data-region="east">East</button>
            <button class="btn-dock" onclick="app.switchRegion('north')" data-region="north">North</button>
            <button class="btn-dock" onclick="app.switchRegion('west')" data-region="west">West</button>
        </div>
    </div>

    <script type="importmap">
        { "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js", "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/" } }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
        import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
        import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';

        // ==========================================
        // 1. DATA ENGINE (增加分析数据)
        // ==========================================
        const generateData = () => {
            const hours = 24;
            const createSeries = (base, variance) => Array.from({length: hours}, (_, i) => Math.max(10, Math.floor(base + Math.sin(i/hours*Math.PI*2)*10 + (Math.random()-0.5)*variance)));

            return {
                regions: {
                    central: { 
                        color: '#00f3ff', shape: 'sphere',
                        pm25: createSeries(45, 15), 
                        composition: [35, 25, 15, 10, 5, 10],
                        radar: [80, 70, 60, 50, 85],
                        analysis: {
                            source: "Vehicle Emissions (45%)",
                            risk: "Moderate",
                            trend: "Rising (+3.2% vs Yesterday)",
                            desc: "Central district shows elevated particulate matter consistent with peak hour traffic density. UV index is moderate."
                        }
                    },
                    east: { 
                        color: '#ff00ff', shape: 'cube',
                        pm25: createSeries(25, 10), 
                        composition: [20, 20, 30, 10, 10, 10],
                        radar: [50, 40, 70, 40, 50],
                        analysis: {
                            source: "Coastal Winds / Shipping",
                            risk: "Low",
                            trend: "Stable (-0.5% vs Yesterday)",
                            desc: "East coast benefits from sea breeze dispersion. Slight ozone increase detected due to industrial drift."
                        }
                    },
                    north: { 
                        color: '#00ff9d', shape: 'helix',
                        pm25: createSeries(35, 12), 
                        composition: [30, 30, 10, 15, 5, 10],
                        radar: [60, 60, 50, 70, 60],
                        analysis: {
                            source: "Transboundary Haze",
                            risk: "Low-Moderate",
                            trend: "Fluctuating",
                            desc: "Northern sectors detecting intermittent particulate spikes correlated with wind direction changes."
                        }
                    },
                    west: { 
                        color: '#ffa500', shape: 'galaxy',
                        pm25: createSeries(65, 20), 
                        composition: [50, 40, 20, 20, 15, 15],
                        radar: [90, 80, 60, 80, 95],
                        analysis: {
                            source: "Industrial Activity",
                            risk: "High Alert",
                            trend: "Spiking (+12% vs Yesterday)",
                            desc: "Jurong industrial zone showing significant SO2 and PM2.5 accumulation. Advisory issued for sensitive groups."
                        }
                    }
                }
            };
        };
        const DATA = generateData();
        const LOGS = ["Geometry synced", "Vertices aligned", "Buffer flushed", "Rendering...", "Sensor OK", "Uplink verified", "Packet received"];

        // ==========================================
        // 2. STATE & CHARTS
        // ==========================================
        const state = { region: 'central', timeIdx: 0, playing: true, uiVisible: true };
        
        const charts = {
            wave: echarts.init(document.getElementById('chart-wave')),
            bar: echarts.init(document.getElementById('chart-bar')),
            radar: echarts.init(document.getElementById('chart-radar'))
        };
        const chartOpt = { backgroundColor: 'transparent', animation: false };

        charts.wave.setOption({ ...chartOpt, grid: {top:10,bottom:0,left:0,right:0}, xAxis:{type:'category',show:false}, yAxis:{type:'value',show:false,min:0,max:100}, series:[{type:'line',smooth:0.4,showSymbol:false,areaStyle:{opacity:0.3}}] });
        charts.bar.setOption({ ...chartOpt, grid: {top:10,bottom:0,left:0,right:0}, xAxis:{type:'category',show:false}, yAxis:{type:'value',show:false}, series:[{type:'bar',barWidth:'60%',itemStyle:{borderRadius:[2,2,0,0]}}] });
        charts.radar.setOption({ ...chartOpt, radar:{indicator:[{name:'PM2.5',max:100},{name:'PM10',max:100},{name:'O3',max:100},{name:'NO2',max:100},{name:'PSI',max:100}], center:['50%','50%'], radius:'65%', splitLine:{lineStyle:{color:'rgba(255,255,255,0.1)'}}, axisName:{color:'#888'}}, series:[{type:'radar', areaStyle:{opacity:0.4}}] });

        // ==========================================
        // 3. THREE.JS PHYSICS (高密度粒子版本)
        // ==========================================
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x050505, 0.002);
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
        camera.position.z = 60;
        
        const renderer = new THREE.WebGLRenderer({alpha: true, antialias: false});
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.getElementById('canvas-wrapper').appendChild(renderer.domElement);

        // 核心修改：粒子数量翻倍，尺寸减半
        const pCount = 28000; // 之前的 3.5倍
        const geometry = new THREE.BufferGeometry();
        const pos = new Float32Array(pCount * 3);
        const targetPos = new Float32Array(pCount * 3);
        
        for(let i=0; i<pCount*3; i++) {
            pos[i] = (Math.random()-0.5) * 200;
            targetPos[i] = pos[i];
        }
        geometry.setAttribute('position', new THREE.BufferAttribute(pos, 3));
        
        // size 从 0.3 降到 0.15，制造细腻感
        const material = new THREE.PointsMaterial({ size: 0.15, color: 0x00f3ff, transparent: true, opacity: 0.8, blending: THREE.AdditiveBlending });
        const particles = new THREE.Points(geometry, material);
        scene.add(particles);

        const composer = new EffectComposer(renderer);
        composer.addPass(new RenderPass(scene, camera));
        composer.addPass(new UnrealBloomPass(new THREE.Vector2(window.innerWidth, window.innerHeight), 1.5, 0.4, 0.85));

        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2(99,99);
        const plane = new THREE.Plane(new THREE.Vector3(0,0,1), 0);
        const mouseTgt = new THREE.Vector3();
        document.addEventListener('mousemove', e => {
            mouse.x = (e.clientX/window.innerWidth)*2-1;
            mouse.y = -(e.clientY/window.innerHeight)*2+1;
        });

        const getShapePos = (shape, i) => {
            // 算法保持不变，但因为 pCount 变大，i/pCount 的步进更小，形状会自动变得更密
            if(shape === 'sphere') {
                const phi = Math.acos(-1 + (2*i)/pCount);
                const theta = Math.sqrt(pCount*Math.PI)*phi;
                const r = 35;
                return { x: r*Math.cos(theta)*Math.sin(phi), y: r*Math.sin(theta)*Math.sin(phi), z: r*Math.cos(phi) };
            }
            if(shape === 'cube') {
                const s = 50; return { x: (Math.random()-0.5)*s, y: (Math.random()-0.5)*s, z: (Math.random()-0.5)*s };
            }
            if(shape === 'helix') {
                const t = i/pCount; const a = t * Math.PI * 20; const r = 20;
                return { x: Math.cos(a)*r, y: (t-0.5)*100, z: Math.sin(a)*r };
            }
            if(shape === 'galaxy') {
                const a = i * 0.1; const r = Math.pow(i, 0.6) * 0.2; const arm = i % 3; const s = a + arm * (Math.PI * 2 / 3);
                return { x: Math.cos(s)*r, y: (Math.random()-0.5)*5, z: Math.sin(s)*r };
            }
            return {x:0, y:0, z:0};
        };

        const updateParticlesTarget = () => {
            const rData = DATA.regions[state.region];
            const pm25 = rData.pm25[state.timeIdx];
            const noise = pm25 / 150;
            for(let i=0; i<pCount; i++) {
                const p = getShapePos(rData.shape, i);
                targetPos[i*3] = p.x + (Math.random()-0.5) * noise * 5;
                targetPos[i*3+1] = p.y + (Math.random()-0.5) * noise * 5;
                targetPos[i*3+2] = p.z + (Math.random()-0.5) * noise * 5;
            }
            material.color.set(rData.color);
        };

        // ==========================================
        // 4. UI LOGIC (交互与分析层)
        // ==========================================
        const addLog = (msg) => {
            const c = document.getElementById('log-container');
            const d = document.createElement('div');
            d.className = 'log-line';
            d.innerHTML = `<span>${new Date().toLocaleTimeString().split(' ')[0]}</span> <span>${msg || LOGS[Math.floor(Math.random()*LOGS.length)]}</span>`;
            c.appendChild(d);
            if(c.children.length > 8) c.removeChild(c.firstChild);
        };

        const updateUI = () => {
            const rData = DATA.regions[state.region];
            const c = rData.color;
            const pm25 = rData.pm25[state.timeIdx];

            document.getElementById('val-pm25').innerText = pm25;
            document.getElementById('val-pm25').style.color = c;
            document.getElementById('region-badge').innerText = state.region.toUpperCase();
            document.getElementById('region-badge').style.color = c;
            document.getElementById('typewriter').style.color = c;
            
            let statusMsg = "", statusColor = "#fff";
            if(pm25 < 30) { statusMsg = "AIR QUALITY OPTIMAL"; statusColor = "#00ff9d"; }
            else if(pm25 < 60) { statusMsg = "MODERATE LEVELS"; statusColor = "#ffcc00"; }
            else { statusMsg = "CRITICAL ALERT"; statusColor = "#ff2a2a"; }
            const descEl = document.getElementById('val-desc');
            descEl.innerText = statusMsg;
            descEl.style.color = statusColor;

            charts.wave.setOption({
                series: [{ data: rData.pm25, lineStyle: { color: c }, areaStyle: { color: new echarts.graphic.LinearGradient(0,0,0,1, [{offset:0, color: c},{offset:1, color:'transparent'}]) }, markLine: { data: [{ xAxis: state.timeIdx }], lineStyle: { color: '#fff', type: 'dashed' }, symbol: 'none', animation: false } }]
            });

            const noisyComp = rData.composition.map(v => v + (Math.random()-0.5)*5);
            charts.bar.setOption({ series: [{ data: noisyComp, itemStyle: { color: c } }] });

            charts.radar.setOption({ series: [{ data: [ { value: [90,80,95,80,90], name: 'Guide', lineStyle:{type:'dashed', color:'#555'} }, { value: rData.radar, name: 'Current', itemStyle:{color:c}, areaStyle:{color:c, opacity:0.3} } ] }] });

            document.getElementById('timeline-bar').style.width = ((state.timeIdx/24)*100)+'%';
            document.getElementById('timeline-bar').style.backgroundColor = c;
            updateParticlesTarget();
        };

        window.app = {
            switchRegion: (r) => {
                state.region = r;
                document.querySelectorAll('.btn-dock').forEach(b => {
                    b.classList.toggle('active', b.dataset.region === r);
                    if(b.dataset.region === r) b.style.setProperty('--accent', DATA.regions[r].color);
                });
                document.getElementById('typewriter').innerText = `CALIBRATING ${r.toUpperCase()}...`;
                addLog(`Switched to ${r.toUpperCase()}`);
                updateUI();
            },
            toggleUI: () => {
                state.uiVisible = !state.uiVisible;
                const layer = document.getElementById('ui-layer');
                if(!state.uiVisible) {
                    layer.classList.add('hidden');
                    document.getElementById('typewriter').innerText = "OBSERVATION MODE";
                } else {
                    layer.classList.remove('hidden');
                    document.getElementById('typewriter').innerText = "HUD RESTORED";
                }
            },
            // --- Analysis Functions ---
            showAnalysis: (type) => {
                const rData = DATA.regions[state.region];
                const overlay = document.getElementById('analysis-layer');
                const content = document.getElementById('analysis-content');
                const text = document.getElementById('analysis-text');
                const trend = document.getElementById('analysis-trend');

                overlay.classList.add('active');
                state.playing = false; // Pause timeline when analyzing

                // Populate text
                text.innerText = rData.analysis.desc;
                trend.innerText = rData.analysis.trend;
                
                // Populate stats based on click type (Simulated deep data)
                let html = `
                    <div class="stat-row"><span>Primary Source</span><span>${rData.analysis.source}</span></div>
                    <div class="stat-row"><span>Risk Assessment</span><span style="color:${rData.pm25[state.timeIdx]>50?'#ff2a2a':'#00ff9d'}">${rData.analysis.risk}</span></div>
                    <div class="stat-row"><span>Avg Humidity</span><span>${60 + Math.floor(Math.random()*20)}%</span></div>
                    <div class="stat-row"><span>Wind Speed</span><span>${(Math.random()*10).toFixed(1)} km/h</span></div>
                    <div class="stat-row"><span>Sensor Uptime</span><span>99.9%</span></div>
                `;
                content.innerHTML = html;
            },
            closeAnalysis: () => {
                document.getElementById('analysis-layer').classList.remove('active');
                state.playing = true; // Resume timeline
            }
        };

        setInterval(() => { if(state.playing) { state.timeIdx = (state.timeIdx+1)%24; updateUI(); if(Math.random()>0.7) addLog(); } }, 800);

        const animate = () => {
            requestAnimationFrame(animate);
            raycaster.setFromCamera(mouse, camera);
            raycaster.ray.intersectPlane(plane, mouseTgt);
            const pArr = geometry.attributes.position.array;
            
            for(let i=0; i<pCount; i++) {
                const ix=i*3, iy=i*3+1, iz=i*3+2;
                pArr[ix] += (targetPos[ix] - pArr[ix]) * 0.08;
                pArr[iy] += (targetPos[iy] - pArr[iy]) * 0.08;
                pArr[iz] += (targetPos[iz] - pArr[iz]) * 0.08;
                
                const dx=pArr[ix]-mouseTgt.x, dy=pArr[iy]-mouseTgt.y, dz=pArr[iz]-mouseTgt.z;
                const distSq = dx*dx + dy*dy + dz*dz;
                if(distSq < 1500) {
                    const f = (1500 - distSq) / 1500;
                    pArr[ix] += dx * f * 0.5; pArr[iy] += dy * f * 0.5; pArr[iz] += dz * f * 0.5;
                }
            }
            geometry.attributes.position.needsUpdate = true;
            particles.rotation.y += 0.002;
            composer.render();
        };

        updateUI();
        animate();

        window.addEventListener('resize', () => {
            camera.aspect=window.innerWidth/window.innerHeight; camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth,window.innerHeight); composer.setSize(window.innerWidth,window.innerHeight);
            Object.values(charts).forEach(c=>c.resize());
        });
    </script>
</body>
</html>
