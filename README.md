
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA Eyes 3D - Ultimate Cosmic Cinematic Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/EffectComposer.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/RenderPass.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/ShaderPass.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/shaders/CopyShader.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/shaders/LuminosityHighPassShader.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/UnrealBloomPass.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/objects/Lensflare.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Share+Tech+Mono&display=swap');
        body { font-family: 'Share Tech Mono', monospace; background: #000; color: #38bdf8; overflow: hidden; margin: 0; }
        .font-orbitron { font-family: 'Orbitron', sans-serif; }
        .hud-glass {
            background: rgba(3, 7, 18, 0.85);
            backdrop-filter: blur(24px);
            border: 1px solid rgba(56, 189, 248, 0.4);
            box-shadow: 0 10px 40px rgba(0, 0, 0, 0.95), inset 0 0 30px rgba(56, 189, 248, 0.12);
        }
        .planet-pill {
            background: rgba(15, 23, 42, 0.7);
            border: 1px solid rgba(148, 163, 184, 0.25);
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .planet-pill:hover, .planet-pill.active {
            border-color: #38bdf8;
            background: rgba(14, 165, 233, 0.28);
            box-shadow: 0 0 25px rgba(56, 189, 248, 0.6);
            transform: translateY(-2px);
        }
        .custom-scrollbar::-webkit-scrollbar { height: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: rgba(56, 189, 248, 0.6); border-radius: 99px; }
    </style>
</head>
<body class="w-screen h-screen select-none">

    <!-- HUD ĐIỀU KHIỂN & ÂM THANH TỰ ĐỘNG -->
    <header class="absolute top-4 left-4 z-20 hud-glass p-4 rounded-2xl w-80 space-y-3">
        <div class="flex justify-between items-center">
            <div class="flex items-center gap-2">
                <span class="w-2 h-2 rounded-full bg-cyan-400 animate-pulse"></span>
                <span class="text-[10px] font-bold tracking-widest text-slate-300">GOD-TIER RENDER ENGINE</span>
            </div>
            <button onclick="toggleHud()" id="hudToggleBtn" class="px-2 py-1 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-lg text-[10px] font-bold hover:border-cyan-400 transition">
                👁️ Ẩn
            </button>
        </div>

        <div id="hudBody" class="space-y-3 transition-all duration-300 overflow-hidden">
            <!-- BẢNG ÂM THANH TỔNG HỢP KHÔNG GIAN -->
            <div class="bg-slate-950/80 p-2.5 rounded-xl border border-rose-500/40 space-y-2 shadow-[0_0_15px_rgba(244,63,94,0.15)]">
                <div class="flex justify-between items-center text-[10px]">
                    <span class="text-rose-300 font-bold flex items-center gap-1.5">🎵 AMBIENT SYNTH: <span class="text-white">Active</span></span>
                    <button onclick="toggleSpaceAudio()" id="audioBtn" class="px-2 py-1 bg-rose-950/80 border border-rose-500/60 text-rose-200 rounded-lg font-bold hover:bg-rose-900 transition">
                        ⏸ Tắt Nhạc
                    </button>
                </div>
                <div class="flex items-center justify-between text-[10px]">
                    <span class="text-slate-400">🔊 Vol:</span>
                    <input type="range" min="0" max="1" step="0.05" value="0.8" oninput="setVolume(this.value)" class="w-28 accent-rose-500 h-1 cursor-pointer">
                    <span id="volVal" class="text-rose-400 font-bold">80%</span>
                </div>
            </div>

            <div class="grid grid-cols-2 gap-2 text-[10px]">
                <button onclick="toggleFreeFly()" id="flyBtn" class="py-1.5 px-2 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-xl font-bold hover:border-cyan-400 transition">
                    🛸 Free-Fly: OFF
                </button>
                <button onclick="toggleSpacetimeGrid()" id="gridBtn" class="py-1.5 px-2 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-xl font-bold hover:border-cyan-400 transition">
                    🕸️ Lưới KG: OFF
                </button>
            </div>

            <div class="bg-slate-950/60 p-2 rounded-xl border border-slate-800/80 text-[10px]">
                <div class="flex justify-between text-slate-400 mb-1"><span>⏳ Tốc độ thời gian</span><span id="speedVal" class="text-cyan-300 font-bold">1.0x</span></div>
                <input type="range" min="0" max="5" step="0.1" value="1" oninput="setSpeed(this.value)" class="w-full accent-cyan-400 h-1 cursor-pointer">
            </div>

            <div class="border-t border-slate-800/80 pt-2.5 space-y-1">
                <div id="planetTitle" class="text-sm font-orbitron font-bold text-amber-300 tracking-wide">MẶT TRỜI (SOL)</div>
                <p id="planetDesc" class="text-[11px] text-slate-300 leading-relaxed">
                    Bật chế độ Free-Fly để camera tự do bay lượn điện ảnh qua hệ mặt trời.
                </p>
                <div class="grid grid-cols-2 gap-1 pt-1.5 text-[10px] text-slate-400">
                    <div>Khoảng cách: <span id="planetDist" class="text-slate-200">0 AU</span></div>
                    <div>Nhiệt độ: <span id="planetTemp" class="text-amber-300">5,500 °C</span></div>
                </div>
            </div>
        </div>
    </header>

    <!-- THANH CHỌN HÀNH TINH -->
    <nav class="absolute bottom-5 left-1/2 -translate-x-1/2 z-20 hud-glass px-3 py-2.5 rounded-2xl flex gap-2 overflow-x-auto max-w-[95vw] custom-scrollbar">
        <button id="btn-sun" onclick="selectPlanet('sun')" class="planet-pill active px-3 py-1.5 text-amber-300 rounded-xl text-xs font-bold whitespace-nowrap">☀️ Mặt Trời</button>
        <button id="btn-mercury" onclick="selectPlanet('mercury')" class="planet-pill px-3 py-1.5 text-slate-300 rounded-xl text-xs font-bold whitespace-nowrap">🪨 Sao Thủy</button>
        <button id="btn-venus" onclick="selectPlanet('venus')" class="planet-pill px-3 py-1.5 text-amber-200 rounded-xl text-xs font-bold whitespace-nowrap">🌕 Sao Kim</button>
        <button id="btn-earth" onclick="selectPlanet('earth')" class="planet-pill px-3 py-1.5 text-sky-400 rounded-xl text-xs font-bold whitespace-nowrap">🌍 Trái Đất</button>
        <button id="btn-mars" onclick="selectPlanet('mars')" class="planet-pill px-3 py-1.5 text-rose-400 rounded-xl text-xs font-bold whitespace-nowrap">🔴 Sao Hỏa</button>
        <button id="btn-jupiter" onclick="selectPlanet('jupiter')" class="planet-pill px-3 py-1.5 text-orange-300 rounded-xl text-xs font-bold whitespace-nowrap">🪐 Sao Mộc</button>
        <button id="btn-saturn" onclick="selectPlanet('saturn')" class="planet-pill px-3 py-1.5 text-yellow-200 rounded-xl text-xs font-bold whitespace-nowrap">🪐 Sao Thổ</button>
        <button id="btn-uranus" onclick="selectPlanet('uranus')" class="planet-pill px-3 py-1.5 text-cyan-300 rounded-xl text-xs font-bold whitespace-nowrap">💎 Thiên Vương</button>
        <button id="btn-neptune" onclick="selectPlanet('neptune')" class="planet-pill px-3 py-1.5 text-indigo-400 rounded-xl text-xs font-bold whitespace-nowrap">🔵 Hải Vương</button>
    </nav>

    <div id="canvasContainer" class="fixed inset-0 z-0"></div>

    <script>
        // ==========================================
        // BỘ TỔNG HỢP ÂM THANH KHÔNG GIAN THỜI GIAN THỰC
        // ==========================================
        let audioCtx, masterGain, isAudioPlaying = false, chordTimer;

        function initSpaceSynth() {
            if (audioCtx) {
                if (audioCtx.state === 'suspended') audioCtx.resume();
                return;
            }
            const AudioContext = window.AudioContext || window.webkitAudioContext;
            audioCtx = new AudioContext();

            masterGain = audioCtx.createGain();
            masterGain.gain.value = 0.8;
            masterGain.connect(audioCtx.destination);

            // Tạo dải âm trầm vũ trụ (Sub-Bass)
            const subOsc1 = audioCtx.createOscillator();
            const subOsc2 = audioCtx.createOscillator();
            const subFilter = audioCtx.createBiquadFilter();
            const subGain = audioCtx.createGain();

            subOsc1.type = 'sine';
            subOsc1.frequency.value = 55; // A1
            subOsc2.type = 'triangle';
            subOsc2.frequency.value = 110; // A2

            subFilter.type = 'lowpass';
            subFilter.frequency.value = 180;
            subGain.gain.value = 0.4;

            subOsc1.connect(subFilter);
            subOsc2.connect(subFilter);
            subFilter.connect(subGain);
            subGain.connect(masterGain);

            subOsc1.start();
            subOsc2.start();

            // Hợp âm không gian ngân vang liên tục
            const chords = [
                [220.00, 261.63, 329.63, 392.00], 
                [174.61, 220.00, 261.63, 329.63], 
                [130.81, 196.00, 246.94, 329.63], 
                [164.81, 196.00, 293.66, 392.00]
            ];
            let chordIndex = 0;

            function playPad() {
                if (!isAudioPlaying || audioCtx.state !== 'running') return;
                const now = audioCtx.currentTime;
                const freqs = chords[chordIndex % chords.length];
                chordIndex++;

                freqs.forEach((f, idx) => {
                    const osc = audioCtx.createOscillator();
                    const noteGain = audioCtx.createGain();
                    const filter = audioCtx.createBiquadFilter();

                    osc.type = idx % 2 === 0 ? 'sine' : 'triangle';
                    osc.frequency.setValueAtTime(f, now);

                    filter.type = 'lowpass';
                    filter.frequency.setValueAtTime(250, now);
                    filter.frequency.exponentialRampToValueAtTime(850, now + 3);
                    filter.frequency.exponentialRampToValueAtTime(200, now + 7);

                    noteGain.gain.setValueAtTime(0.0001, now);
                    noteGain.gain.linearRampToValueAtTime(0.12, now + 2.5);
                    noteGain.gain.exponentialRampToValueAtTime(0.0001, now + 7.5);

                    osc.connect(filter);
                    filter.connect(noteGain);
                    noteGain.connect(masterGain);

                    osc.start(now);
                    osc.stop(now + 7.5);
                });
            }

            isAudioPlaying = true;
            playPad();
            chordTimer = setInterval(playPad, 6000);
        }

        function setVolume(v) {
            const val = parseFloat(v);
            document.getElementById('volVal').innerText = Math.round(val * 100) + '%';
            if (masterGain && audioCtx) {
                masterGain.gain.setTargetAtTime(val, audioCtx.currentTime, 0.05);
            }
        }

        function toggleSpaceAudio() {
            if (!audioCtx) {
                initSpaceSynth();
                return;
            }
            if (audioCtx.state === 'running') {
                audioCtx.suspend();
                isAudioPlaying = false;
                document.getElementById('audioBtn').innerText = '▶ Bật Nhạc';
            } else {
                audioCtx.resume();
                isAudioPlaying = true;
                document.getElementById('audioBtn').innerText = '⏸ Tắt Nhạc';
            }
        }

        // ==========================================
        // ENGINE 3D THREE.JS
        // ==========================================
        let scene, camera, renderer, composer, bloomPass;
        let planets = {};
        let sunUniforms, milkyWayGroup, asteroidBelt, spacetimeGrid, cosmicDust;
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, prevMouse = { x: 0, y: 0 };
        let camDist = 72, targetCamDist = 72;
        let camAngleX = 0.35, camAngleY = 0.25;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0;
        let isFreeFly = false, freeFlyAngle = 0;

        const planetDB = {
            sun: { name: "MẶT TRỜI (SOL)", desc: "Khối cầu plasma sôi sục giữa không gian bao la.", dist: "0 AU", temp: "5,500 °C", zoom: 72 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Bề mặt đá bazan xám bạc lồi lõm hàng triệu hố thiên thạch.", dist: "0.39 AU", temp: "430 °C", zoom: 4.5 },
            venus: { name: "SAO KIM (VENUS)", desc: "Khí quyển siêu đặc cuộn xoáy mây axit sulfuric ánh vàng rực.", dist: "0.72 AU", temp: "462 °C", zoom: 6 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Tán xạ khí quyển xanh ngọc, mây 3D tách lớp & ánh đèn đô thị ban đêm.", dist: "1.00 AU", temp: "15 °C", zoom: 6.5 },
            mars: { name: "SAO HỎA (MARS)", desc: "Sa mạc oxit sắt đỏ rực với chỏm băng cực và hẻm núi sâu.", dist: "1.52 AU", temp: "-63 °C", zoom: 5.5 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Khí quyển dải băng tầng khí động học & cơn bão Vết Đỏ Lớn.", dist: "5.20 AU", temp: "-110 °C", zoom: 13 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Hệ thống vành đai băng đa tầng phân tách bởi khe Cassini sắc nét.", dist: "9.58 AU", temp: "-140 °C", zoom: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng khí metan xanh ngọc bích với vành đai dọc mờ ảo.", dist: "19.22 AU", temp: "-195 °C", zoom: 10 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Thế giới băng giá xanh thẫm với những luồng gió siêu thanh.", dist: "30.05 AU", temp: "-200 °C", zoom: 10 }
        };

        const sunVertexShader = `
            varying vec3 vNormal;
            varying vec3 vPosition;
            void main() {
                vNormal = normalize(normalMatrix * normal);
                vPosition = position;
                gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
            }
        `;

        const sunFragmentShader = `
            uniform float uTime;
            varying vec3 vNormal;
            varying vec3 vPosition;

            float hash(vec3 p) {
                p = fract(p * 0.3183099 + 0.1);
                p *= 17.0;
                return fract(p.x * p.y * p.z * (p.x + p.y + p.z));
            }

            float noise(vec3 x) {
                vec3 i = floor(x);
                vec3 f = fract(x);
                f = f * f * (3.0 - 2.0 * f);
                return mix(mix(mix(hash(i + vec3(0,0,0)), hash(i + vec3(1,0,0)), f.x),
                               mix(hash(i + vec3(0,1,0)), hash(i + vec3(1,1,0)), f.x), f.y),
                           mix(mix(hash(i + vec3(0,0,1)), hash(i + vec3(1,0,1)), f.x),
                               mix(hash(i + vec3(0,1,1)), hash(i + vec3(1,1,1)), f.x), f.y), f.z);
            }

            float fbm(vec3 p) {
                float v = 0.0;
                float a = 0.5;
                for (int i = 0; i < 5; i++) {
                    v += a * noise(p);
                    p = p * 2.05 + vec3(0.15);
                    a *= 0.5;
                }
                return v;
            }

            void main() {
                vec3 pos = vPosition * 1.4;
                float n1 = fbm(pos + vec3(uTime * 0.25, -uTime * 0.15, uTime * 0.2));
                float n2 = fbm(pos * 2.2 - vec3(uTime * 0.3));
                float plasma = smoothstep(0.25, 0.85, n1 * 0.65 + n2 * 0.45);

                vec3 darkSpot = vec3(0.55, 0.08, 0.01);
                vec3 midOrange = vec3(1.0, 0.38, 0.02);
                vec3 hotYellow = vec3(1.45, 1.2, 0.6);

                vec3 col = mix(darkSpot, midOrange, smoothstep(0.1, 0.55, plasma));
                col = mix(col, hotYellow, smoothstep(0.5, 0.9, plasma));

                float fresnel = pow(1.0 - max(dot(vNormal, vec3(0.0, 0.0, 1.0)), 0.0), 2.5);
                col += vec3(1.2, 0.5, 0.1) * fresnel * 0.8;

                gl_FragColor = vec4(col, 1.0);
            }
        `;

        function createAtmosphereMaterial(colorHex, intensity = 1.1, power = 3.5) {
            return new THREE.ShaderMaterial({
                uniforms: {
                    uColor: { value: new THREE.Color(colorHex) },
                    uIntensity: { value: intensity },
                    uPower: { value: power }
                },
                vertexShader: `
                    varying vec3 vNormal;
                    void main() {
                        vNormal = normalize(normalMatrix * normal);
                        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
                    }
                `,
                fragmentShader: `
                    uniform vec3 uColor;
                    uniform float uIntensity;
                    uniform float uPower;
                    varying vec3 vNormal;
                    void main() {
                        float glow = pow(0.68 - dot(vNormal, vec3(0.0, 0.0, 1.0)), uPower);
                        gl_FragColor = vec4(uColor * uIntensity, max(0.0, glow));
                    }
                `,
                side: THREE.BackSide,
                blending: THREE.AdditiveBlending,
                transparent: true,
                depthWrite: false
            });
        }

        function createFlareTextures() {
            const streakCanvas = document.createElement('canvas');
            streakCanvas.width = 512; streakCanvas.height = 64;
            const sCtx = streakCanvas.getContext('2d');
            const sGrad = sCtx.createLinearGradient(0, 32, 512, 32);
            sGrad.addColorStop(0.0, 'rgba(56, 189, 248, 0)');
            sGrad.addColorStop(0.35, 'rgba(56, 189, 248, 0.45)');
            sGrad.addColorStop(0.5, 'rgba(255, 251, 235, 0.95)');
            sGrad.addColorStop(0.65, 'rgba(56, 189, 248, 0.45)');
            sGrad.addColorStop(1.0, 'rgba(56, 189, 248, 0)');
            sCtx.fillStyle = sGrad;
            sCtx.fillRect(0, 26, 512, 12);

            const orbCanvas = document.createElement('canvas');
            orbCanvas.width = 128; orbCanvas.height = 128;
            const oCtx = orbCanvas.getContext('2d');
            const oGrad = oCtx.createRadialGradient(64, 64, 4, 64, 64, 60);
            oGrad.addColorStop(0, 'rgba(255, 255, 255, 0.9)');
            oGrad.addColorStop(0.4, 'rgba(56, 189, 248, 0.35)');
            oGrad.addColorStop(1, 'rgba(0, 0, 0, 0)');
            oCtx.fillStyle = oGrad;
            oCtx.fillRect(0, 0, 128, 128);

            return {
                streak: new THREE.CanvasTexture(streakCanvas),
                orb: new THREE.CanvasTexture(orbCanvas)
            };
        }

        function generatePlanetMaps(type) {
            const w = 2048, h = 1024;
            const diffCanvas = document.createElement('canvas');
            const bumpCanvas = document.createElement('canvas');
            diffCanvas.width = bumpCanvas.width = w;
            diffCanvas.height = bumpCanvas.height = h;
            const ctx = diffCanvas.getContext('2d');
            const bCtx = bumpCanvas.getContext('2d');

            bCtx.fillStyle = '#808080';
            bCtx.fillRect(0, 0, w, h);

            if (type === 'mercury') {
                ctx.fillStyle = '#64748b'; ctx.fillRect(0, 0, w, h);
                for (let i = 0; i < 1200; i++) {
                    const x = Math.random() * w, y = Math.random() * h, r = Math.random() * 22 + 2;
                    ctx.fillStyle = i % 2 === 0 ? 'rgba(30,41,59,0.45)' : 'rgba(148,163,184,0.35)';
                    ctx.beginPath(); ctx.arc(x, y, r, 0, Math.PI * 2); ctx.fill();
                    bCtx.fillStyle = i % 2 === 0 ? 'rgba(20,20,20,0.6)' : 'rgba(230,230,230,0.5)';
                    bCtx.beginPath(); bCtx.arc(x, y, r, 0, Math.PI * 2); bCtx.fill();
                }
            } else if (type === 'venus') {
                const grad = ctx.createLinearGradient(0, 0, 0, h);
                grad.addColorStop(0, '#92400e'); grad.addColorStop(0.5, '#f59e0b'); grad.addColorStop(1, '#78350f');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, w, h);
                for (let i = 0; i < 200; i++) {
                    ctx.fillStyle = `rgba(254, 243, 199, ${Math.random() * 0.18})`;
                    ctx.beginPath();
                    ctx.ellipse(Math.random() * w, Math.random() * h, Math.random() * 350 + 80, Math.random() * 25 + 6, 0, 0, Math.PI * 2);
                    ctx.fill();
                }
            } else if (type === 'earth') {
                const seaGrad = ctx.createLinearGradient(0, 0, 0, h);
                seaGrad.addColorStop(0, '#e0f2fe');
                seaGrad.addColorStop(0.12, '#0369a1');
                seaGrad.addColorStop(0.5, '#0284c7');
                seaGrad.addColorStop(0.88, '#0369a1');
                seaGrad.addColorStop(1, '#e0f2fe');
                ctx.fillStyle = seaGrad; ctx.fillRect(0, 0, w, h);

                for (let i = 0; i < 65; i++) {
                    const cx = Math.random() * w, cy = 150 + Math.random() * (h - 300);
                    const rx = Math.random() * 190 + 50, ry = Math.random() * 110 + 30;
                    const rot = Math.random() * Math.PI;
                    ctx.fillStyle = i % 3 === 0 ? '#a16207' : '#15803d';
                    ctx.beginPath(); ctx.ellipse(cx, cy, rx, ry, rot, 0, Math.PI * 2); ctx.fill();

                    ctx.fillStyle = '#fef08a';
                    for (let k = 0; k < 14; k++) {
                        ctx.fillRect(cx + (Math.random() - 0.5) * rx, cy + (Math.random() - 0.5) * ry, 2.5, 2.5);
                    }

                    bCtx.fillStyle = 'rgba(240, 240, 240, 0.75)';
                    bCtx.beginPath(); bCtx.ellipse(cx, cy, rx * 0.85, ry * 0.85, rot, 0, Math.PI * 2); bCtx.fill();
                }
            } else if (type === 'clouds') {
                ctx.clearRect(0, 0, w, h);
                for (let i = 0; i < 240; i++) {
                    ctx.fillStyle = `rgba(255, 255, 255, ${Math.random() * 0.45 + 0.15})`;
                    ctx.beginPath();
                    ctx.ellipse(Math.random() * w, Math.random() * h, Math.random() * 220 + 40, Math.random() * 35 + 8, (Math.random() - 0.5) * 0.4, 0, Math.PI * 2);
                    ctx.fill();
                }
            } else if (type === 'mars') {
                ctx.fillStyle = '#9a3412'; ctx.fillRect(0, 0, w, h);
                for (let i = 0; i < 500; i++) {
                    const x = Math.random() * w, y = Math.random() * h, r = Math.random() * 45 + 5;
                    ctx.fillStyle = i % 2 === 0 ? 'rgba(67, 20, 7, 0.4)' : 'rgba(234, 88, 12, 0.35)';
                    ctx.beginPath(); ctx.arc(x, y, r, 0, Math.PI * 2); ctx.fill();
                    bCtx.fillStyle = i % 2 === 0 ? 'rgba(40,40,40,0.5)' : 'rgba(210,210,210,0.5)';
                    bCtx.beginPath(); bCtx.arc(x, y, r, 0, Math.PI * 2); ctx.fill();
                }
                ctx.fillStyle = '#f8fafc';
                ctx.fillRect(0, 0, w, 45); ctx.fillRect(0, h - 45, w, 45);
            } else if (type === 'jupiter') {
                const palette = ['#451a03', '#92400e', '#d97706', '#fef3c7', '#b45309', '#78350f', '#fde68a'];
                let y = 0;
                while (y < h) {
                    const bandH = Math.random() * 28 + 8;
                    ctx.fillStyle = palette[Math.floor(Math.random() * palette.length)];
                    ctx.fillRect(0, y, w, bandH);
                    for (let x = 0; x < w; x += 24) {
                        ctx.beginPath();
                        ctx.arc(x, y, Math.random() * 12 + 4, 0, Math.PI * 2);
                        ctx.fill();
                    }
                    y += bandH;
                }
                const grsGrad = ctx.createRadialGradient(1350, 640, 10, 1350, 640, 125);
                grsGrad.addColorStop(0, '#7f1d1d'); grsGrad.addColorStop(0.6, '#dc2626'); grsGrad.addColorStop(1, 'rgba(217,119,6,0)');
                ctx.fillStyle = grsGrad;
                ctx.beginPath(); ctx.ellipse(1350, 640, 130, 80, 0, 0, Math.PI * 2); ctx.fill();
            } else if (type === 'saturn') {
                const palette = ['#713f12', '#a16207', '#eab308', '#fef08a', '#ca8a04', '#fde047'];
                let y = 0;
                while (y < h) {
                    const bandH = Math.random() * 22 + 6;
                    ctx.fillStyle = palette[Math.floor(Math.random() * palette.length)];
                    ctx.fillRect(0, y, w, bandH);
                    y += bandH;
                }
            } else if (type === 'uranus') {
                const grad = ctx.createLinearGradient(0, 0, 0, h);
                grad.addColorStop(0, '#cffafe'); grad.addColorStop(0.5, '#06b6d4'); grad.addColorStop(1, '#cffafe');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, w, h);
            } else if (type === 'neptune') {
                const grad = ctx.createLinearGradient(0, 0, 0, h);
                grad.addColorStop(0, '#1e1b4b'); grad.addColorStop(0.5, '#1d4ed8'); grad.addColorStop(1, '#1e1b4b');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, w, h);
                ctx.fillStyle = 'rgba(147, 197, 253, 0.35)';
                for (let i = 0; i < 40; i++) ctx.fillRect(Math.random() * w, Math.random() * h, Math.random() * 260 + 40, 6);
            }

            return {
                map: new THREE.CanvasTexture(diffCanvas),
                bumpMap: new THREE.CanvasTexture(bumpCanvas)
            };
        }

        function createSaturnRingTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 1024; canvas.height = 64;
            const ctx = canvas.getContext('2d');
            const grad = ctx.createLinearGradient(0, 0, 1024, 0);
            grad.addColorStop(0.0, 'rgba(254, 240, 138, 0.0)');
            grad.addColorStop(0.15, 'rgba(180, 150, 90, 0.35)');
            grad.addColorStop(0.35, 'rgba(254, 243, 199, 0.92)');
            grad.addColorStop(0.58, 'rgba(234, 179, 8, 0.85)');
            grad.addColorStop(0.63, 'rgba(0, 0, 0, 0.05)');
            grad.addColorStop(0.68, 'rgba(253, 224, 71, 0.8)');
            grad.addColorStop(0.92, 'rgba(202, 138, 4, 0.45)');
            grad.addColorStop(1.0, 'rgba(254, 240, 138, 0.0)');
            ctx.fillStyle = grad;
            ctx.fillRect(0, 0, 1024, 64);
            return new THREE.CanvasTexture(canvas);
        }

        function createMilkyWayBackdrop() {
            milkyWayGroup = new THREE.Group();
            milkyWayGroup.rotation.z = Math.PI / 5;

            const bandCount = 28000;
            const bandGeo = new THREE.BufferGeometry();
            const bandPos = new Float32Array(bandCount * 3);
            const bandCol = new Float32Array(bandCount * 3);

            const milkyColors = [
                new THREE.Color(0x38bdf8),
                new THREE.Color(0xa855f7),
                new THREE.Color(0xfde047),
                new THREE.Color(0xf472b6),
                new THREE.Color(0xffffff)
            ];

            for (let i = 0; i < bandCount; i++) {
                const angle = Math.random() * Math.PI * 2;
                const radius = 1400 + Math.random() * 1200;
                const spread = (Math.random() - 0.5) * (Math.random() - 0.5) * 650;

                bandPos[i * 3] = Math.cos(angle) * radius;
                bandPos[i * 3 + 1] = spread;
                bandPos[i * 3 + 2] = Math.sin(angle) * radius;

                const c = milkyColors[Math.floor(Math.random() * milkyColors.length)];
                const brightness = 0.4 + Math.random() * 0.6;
                bandCol[i * 3] = c.r * brightness;
                bandCol[i * 3 + 1] = c.g * brightness;
                bandCol[i * 3 + 2] = c.b * brightness;
            }
            bandGeo.setAttribute('position', new THREE.BufferAttribute(bandPos, 3));
            bandGeo.setAttribute('color', new THREE.BufferAttribute(bandCol, 3));
            milkyWayGroup.add(new THREE.Points(bandGeo, new THREE.PointsMaterial({
                size: 3.2,
                vertexColors: true,
                transparent: true,
                opacity: 0.82,
                blending: THREE.AdditiveBlending,
                depthWrite: false
            })));

            const dustCount = 4000;
            const dustGeo = new THREE.BufferGeometry();
            const dustPos = new Float32Array(dustCount * 3);
            for (let i = 0; i < dustCount; i++) {
                dustPos[i * 3] = (Math.random() - 0.5) * 350;
                dustPos[i * 3 + 1] = (Math.random() - 0.5) * 120;
                dustPos[i * 3 + 2] = (Math.random() - 0.5) * 350;
            }
            dustGeo.setAttribute('position', new THREE.BufferAttribute(dustPos, 3));
            cosmicDust = new THREE.Points(dustGeo, new THREE.PointsMaterial({ color: 0x7dd3fc, size: 0.8, transparent: true, opacity: 0.65, blending: THREE.AdditiveBlending }));
            scene.add(cosmicDust);

            scene.add(milkyWayGroup);
        }

        function createAsteroidBelt() {
            const count = 2200;
            const geo = new THREE.DodecahedronGeometry(0.11, 0);
            const mat = new THREE.MeshStandardMaterial({ color: 0x94a3b8, roughness: 0.85, metalness: 0.2 });
            asteroidBelt = new THREE.InstancedMesh(geo, mat, count);

            const dummy = new THREE.Object3D();
            for (let i = 0; i < count; i++) {
                const angle = Math.random() * Math.PI * 2;
                const radius = 30.2 + Math.random() * 3.6;
                const y = (Math.random() - 0.5) * 1.1;

                dummy.position.set(Math.cos(angle) * radius, y, Math.sin(angle) * radius);
                dummy.rotation.set(Math.random() * Math.PI, Math.random() * Math.PI, Math.random() * Math.PI);
                const scale = 0.4 + Math.random() * 1.4;
                dummy.scale.set(scale, scale * 0.7, scale);
                dummy.updateMatrix();
                asteroidBelt.setMatrixAt(i, dummy.matrix);
            }
            scene.add(asteroidBelt);
        }

        function createSpacetimeGrid() {
            const size = 160;
            const divisions = 48;
            const geo = new THREE.PlaneGeometry(size, size, divisions, divisions);
            geo.rotateX(-Math.PI / 2);

            const pos = geo.attributes.position;
            for (let i = 0; i < pos.count; i++) {
                const x = pos.getX(i);
                const z = pos.getZ(i);
                const dist = Math.sqrt(x * x + z * z);
                const dip = -18.0 / (Math.sqrt(dist * 0.35 + 1.2));
                pos.setY(i, dip - 3.5);
            }
            geo.computeVertexNormals();

            const mat = new THREE.MeshBasicMaterial({
                color: 0x0ea5e9,
                wireframe: true,
                transparent: true,
                opacity: 0.16
            });
            spacetimeGrid = new THREE.Mesh(geo, mat);
            spacetimeGrid.visible = false;
            scene.add(spacetimeGrid);
        }

        function init3D() {
            const container = document.getElementById('canvasContainer');
            scene = new THREE.Scene();

            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 10000);

            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.toneMapping = THREE.ReinhardToneMapping;
            renderer.toneMappingExposure = 1.2;
            container.appendChild(renderer.domElement);

            const renderScene = new THREE.RenderPass(scene, camera);
            bloomPass = new THREE.UnrealBloomPass(new THREE.Vector2(window.innerWidth, window.innerHeight), 1.25, 0.7, 0.3);
            composer = new THREE.EffectComposer(renderer);
            composer.addPass(renderScene);
            composer.addPass(bloomPass);

            const sunLight = new THREE.PointLight(0xfff5e4, 3.5, 2000, 0.25);
            scene.add(sunLight);
            scene.add(new THREE.AmbientLight(0x0f172a, 0.42));

            const flareTex = createFlareTextures();
            const lensflare = new THREE.Lensflare();
            lensflare.addElement(new THREE.LensflareElement(flareTex.streak, 720, 0, new THREE.Color(0x7dd3fc)));
            lensflare.addElement(new THREE.LensflareElement(flareTex.orb, 150, 0, new THREE.Color(0xfef08a)));
            lensflare.addElement(new THREE.LensflareElement(flareTex.orb, 65, 0.35, new THREE.Color(0x38bdf8)));
            lensflare.addElement(new THREE.LensflareElement(flareTex.orb, 95, 0.6, new THREE.Color(0x818cf8)));
            lensflare.addElement(new THREE.LensflareElement(flareTex.orb, 50, 0.85, new THREE.Color(0xf472b6)));
            sunLight.add(lensflare);

            createMilkyWayBackdrop();
            createAsteroidBelt();
            createSpacetimeGrid();

            sunUniforms = { uTime: { value: 0 } };
            const sunMat = new THREE.ShaderMaterial({
                uniforms: sunUniforms,
                vertexShader: sunVertexShader,
                fragmentShader: sunFragmentShader
            });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(5.2, 64, 64), sunMat);
            const corona = new THREE.Mesh(new THREE.SphereGeometry(6.8, 64, 64), createAtmosphereMaterial(0xff8c00, 1.4, 3.0));
            sun.add(corona);
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0 };

            const planetConfigs = [
                { id: 'mercury', size: 0.55, dist: 10, speed: 0.022, tilt: 0.03, atmos: null },
                { id: 'venus', size: 0.95, dist: 14.5, speed: 0.016, tilt: 3.09, atmos: 0xfbbf24 },
                { id: 'earth', size: 1.18, dist: 20, speed: 0.012, tilt: 0.41, atmos: 0x38bdf8, clouds: true, moon: true },
                { id: 'mars', size: 0.78, dist: 26, speed: 0.009, tilt: 0.44, atmos: 0xf97316 },
                { id: 'jupiter', size: 2.8, dist: 38, speed: 0.006, tilt: 0.05, atmos: 0xfdba74 },
                { id: 'saturn', size: 2.3, dist: 50, speed: 0.004, tilt: 0.47, atmos: 0xfef08a, rings: true },
                { id: 'uranus', size: 1.55, dist: 60, speed: 0.003, tilt: 1.71, atmos: 0x22d3ee },
                { id: 'neptune', size: 1.5, dist: 69, speed: 0.002, tilt: 0.49, atmos: 0x3b82f6 }
            ];

            planetConfigs.forEach(cfg => {
                const orbitGeo = new THREE.RingGeometry(cfg.dist - 0.06, cfg.dist + 0.06, 220);
                const orbitMat = new THREE.MeshBasicMaterial({ color: cfg.atmos || 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.22, blending: THREE.AdditiveBlending });
                const orbitRing = new THREE.Mesh(orbitGeo, orbitMat);
                orbitRing.rotation.x = Math.PI / 2;
                scene.add(orbitRing);

                const maps = generatePlanetMaps(cfg.id);
                const pMat = new THREE.MeshStandardMaterial({
                    map: maps.map,
                    bumpMap: maps.bumpMap,
                    bumpScale: 0.045,
                    roughness: 0.72,
                    metalness: 0.08
                });

                const mesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size, 64, 64), pMat);
                mesh.rotation.z = cfg.tilt;
                scene.add(mesh);

                if (cfg.atmos) {
                    const atmosMesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size * 1.12, 48, 48), createAtmosphereMaterial(cfg.atmos, 1.2, 3.6));
                    mesh.add(atmosMesh);
                }

                if (cfg.clouds) {
                    const cloudMaps = generatePlanetMaps('clouds');
                    const cloudMat = new THREE.MeshStandardMaterial({
                        map: cloudMaps.map,
                        transparent: true,
                        opacity: 0.88,
                        depthWrite: false
                    });
                    const cloudMesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size * 1.022, 64, 64), cloudMat);
                    mesh.add(cloudMesh);
                    mesh.userData.cloudMesh = cloudMesh;
                }

                if (cfg.moon) {
                    const moonPivot = new THREE.Group();
                    const moonMesh = new THREE.Mesh(
                        new THREE.SphereGeometry(0.28, 32, 32),
                        new THREE.MeshStandardMaterial({ map: generatePlanetMaps('mercury').map, roughness: 0.9 })
                    );
                    moonMesh.position.x = 2.3;
                    moonPivot.add(moonMesh);
                    mesh.add(moonPivot);
                    mesh.userData.moonPivot = moonPivot;
                }

                if (cfg.rings) {
                    const rGeo = new THREE.RingGeometry(cfg.size * 1.35, cfg.size * 2.45, 128);
                    const posAttr = rGeo.attributes.position;
                    const uvAttr = rGeo.attributes.uv;
                    const v3 = new THREE.Vector3();
                    for (let i = 0; i < posAttr.count; i++) {
                        v3.fromBufferAttribute(posAttr, i);
                        const u = (v3.length() - cfg.size * 1.35) / (cfg.size * 1.1);
                        uvAttr.setXY(i, u, 0.5);
                    }
                    const rMat = new THREE.MeshStandardMaterial({
                        map: createSaturnRingTexture(),
                        side: THREE.DoubleSide,
                        transparent: true,
                        opacity: 0.95,
                        roughness: 0.5
                    });
                    const ringMesh = new THREE.Mesh(rGeo, rMat);
                    ringMesh.rotation.x = Math.PI / 2;
                    mesh.add(ringMesh);
                }

                planets[cfg.id] = { mesh, distance: cfg.dist, speed: cfg.speed, angle: Math.random() * Math.PI * 2 };
            });

            window.addEventListener('mousedown', e => { 
                initSpaceSynth(); // Tự động kích hoạt âm thanh ngay khi click chuột đầu tiên
                if (e.clientY > 280 || e.clientX > 380) { isDragging = true; prevMouse = { x: e.clientX, y: e.clientY }; } 
            });
            window.addEventListener('keydown', () => initSpaceSynth()); // Kích hoạt âm thanh khi bấm phím bất kỳ

            window.addEventListener('mousemove', e => {
                if (!isDragging) return;
                camAngleY -= (e.clientX - prevMouse.x) * 0.005;
                camAngleX = Math.max(-1.45, Math.min(1.45, camAngleX - (e.clientY - prevMouse.y) * 0.005));
                prevMouse = { x: e.clientX, y: e.clientY };
            });
            window.addEventListener('mouseup', () => isDragging = false);
            window.addEventListener('wheel', e => {
                targetCamDist = Math.max(2.5, Math.min(450, targetCamDist + e.deltaY * 0.06));
            });
            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
                composer.setSize(window.innerWidth, window.innerHeight);
            });

            // Thử khởi tạo âm thanh ngay khi load
            setTimeout(() => initSpaceSynth(), 1000);

            animate();
        }

        window.onload = function() {
            init3D();
        };

        function setVolume(v) {
            const val = parseFloat(v);
            document.getElementById('volVal').innerText = Math.round(val * 100) + '%';
            if (masterGain && audioCtx) {
                masterGain.gain.setTargetAtTime(val, audioCtx.currentTime, 0.05);
            }
        }

        function toggleSpaceAudio() {
            if (!audioCtx) {
                initSpaceSynth();
                return;
            }
            if (audioCtx.state === 'running') {
                audioCtx.suspend();
                isAudioPlaying = false;
                document.getElementById('audioBtn').innerText = '▶ Bật Nhạc';
            } else {
                audioCtx.resume();
                isAudioPlaying = true;
                document.getElementById('audioBtn').innerText = '⏸ Tắt Nhạc';
            }
        }

        function toggleFreeFly() {
            isFreeFly = !isFreeFly;
            const btn = document.getElementById('flyBtn');
            btn.innerText = isFreeFly ? '🛸 Free-Fly: ON' : '🛸 Free-Fly: OFF';
            btn.className = isFreeFly
                ? 'py-1.5 px-2 bg-cyan-950/80 border border-cyan-500/60 text-cyan-200 rounded-xl font-bold transition'
                : 'py-1.5 px-2 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-xl font-bold hover:border-cyan-400 transition';
        }

        function toggleSpacetimeGrid() {
            if (!spacetimeGrid) return;
            spacetimeGrid.visible = !spacetimeGrid.visible;
            const btn = document.getElementById('gridBtn');
            btn.innerText = spacetimeGrid.visible ? '🕸️ Lưới KG: ON' : '🕸️ Lưới KG: OFF';
            btn.className = spacetimeGrid.visible
                ? 'py-1.5 px-2 bg-cyan-950/80 border border-cyan-500/60 text-cyan-200 rounded-xl font-bold transition'
                : 'py-1.5 px-2 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-xl font-bold hover:border-cyan-400 transition';
        }

        function setSpeed(v) {
            timeSpeed = parseFloat(v);
            document.getElementById('speedVal').innerText = timeSpeed.toFixed(1) + 'x';
        }

        function toggleHud() {
            isHudVisible = !isHudVisible;
            const body = document.getElementById('hudBody');
            body.style.maxHeight = isHudVisible ? '320px' : '0px';
            body.style.opacity = isHudVisible ? '1' : '0';
            document.getElementById('hudToggleBtn').innerText = isHudVisible ? '👁️ Ẩn' : '👁️ Hiện';
        }

        function selectPlanet(id) {
            focusedObject = id;
            isFreeFly = false;
            document.getElementById('flyBtn').innerText = '🛸 Free-Fly: OFF';
            document.getElementById('flyBtn').className = 'py-1.5 px-2 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-xl font-bold hover:border-cyan-400 transition';

            document.querySelectorAll('.planet-pill').forEach(b => b.classList.remove('active'));
            const btn = document.getElementById(`btn-${id}`);
            if (btn) btn.classList.add('active');

            const info = planetDB[id];
            if (info) {
                document.getElementById('planetTitle').innerText = info.name;
                document.getElementById('planetDesc').innerText = info.desc;
                document.getElementById('planetDist').innerText = info.dist;
                document.getElementById('planetTemp').innerText = info.temp;
                targetCamDist = info.zoom;
            }
        }

        function animate() {
            requestAnimationFrame(animate);

            if (sunUniforms) sunUniforms.uTime.value += 0.015 * timeSpeed;
            if (milkyWayGroup) milkyWayGroup.rotation.y += 0.00012 * timeSpeed;
            if (cosmicDust) cosmicDust.rotation.y += 0.0003 * timeSpeed;
            if (asteroidBelt && asteroidBelt.visible) asteroidBelt.rotation.y += 0.0018 * timeSpeed;

            if (isFreeFly) {
                freeFlyAngle += 0.003 * timeSpeed;
                currentLookAt.set(0, 0, 0);
                camDist = 85 + Math.sin(freeFlyAngle * 0.5) * 25;
                camera.position.x = Math.cos(freeFlyAngle) * camDist;
                camera.position.y = 25 + Math.sin(freeFlyAngle) * 15;
                camera.position.z = Math.sin(freeFlyAngle) * camDist;
                camera.lookAt(currentLookAt);
                composer.render();
                return;
            }

            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
                    p.angle += p.speed * timeSpeed;
                    p.mesh.position.x = Math.cos(p.angle) * p.distance;
                    p.mesh.position.z = Math.sin(p.angle) * p.distance;
                    p.mesh.rotation.y += 0.012 * timeSpeed;

                    if (p.mesh.userData.cloudMesh) p.mesh.userData.cloudMesh.rotation.y += 0.006 * timeSpeed;
                    if (p.mesh.userData.moonPivot) p.mesh.userData.moonPivot.rotation.y += 0.03 * timeSpeed;
                }
            });

            const targetPos = new THREE.Vector3();
            if (planets[focusedObject]) planets[focusedObject].mesh.getWorldPosition(targetPos);
            currentLookAt.lerp(targetPos, 0.08);
            camDist += (targetCamDist - camDist) * 0.08;

            camera.position.x = currentLookAt.x + camDist * Math.sin(camAngleY) * Math.cos(camAngleX);
            camera.position.y = currentLookAt.y + camDist * Math.sin(camAngleX);
            camera.position.z = currentLookAt.z + camDist * Math.cos(camAngleY) * Math.cos(camAngleX);
            camera.lookAt(currentLookAt);

            composer.render();
        }
    </script>
</body>
</html>
