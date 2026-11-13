
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA Eyes 3D - Cinematic Unreal Bloom Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Three.js Core + Post-Processing Bloom -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/EffectComposer.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/RenderPass.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/ShaderPass.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/shaders/CopyShader.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/shaders/LuminosityHighPassShader.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/UnrealBloomPass.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Share+Tech+Mono&display=swap');
        body { font-family: 'Share Tech Mono', monospace; background: #000; color: #38bdf8; overflow: hidden; margin: 0; }
        .font-orbitron { font-family: 'Orbitron', sans-serif; }
        .hud-glass {
            background: rgba(5, 10, 22, 0.72);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(56, 189, 248, 0.28);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.8), inset 0 0 20px rgba(56, 189, 248, 0.08);
        }
        .planet-pill {
            background: rgba(15, 23, 42, 0.65);
            border: 1px solid rgba(148, 163, 184, 0.2);
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .planet-pill:hover, .planet-pill.active {
            border-color: #38bdf8;
            background: rgba(14, 165, 233, 0.22);
            box-shadow: 0 0 20px rgba(56, 189, 248, 0.45);
            transform: translateY(-2px);
        }
        .custom-scrollbar::-webkit-scrollbar { height: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: rgba(56, 189, 248, 0.5); border-radius: 99px; }
    </style>
</head>
<body class="w-screen h-screen select-none">

    <!-- MÀN HÌNH KHỞI ĐỘNG -->
    <div id="startOverlay" class="fixed inset-0 z-50 bg-[#02040a] flex flex-col items-center justify-center p-6 text-center space-y-6">
        <div class="relative w-28 h-28 flex items-center justify-center">
            <div class="absolute inset-0 rounded-full border border-amber-400/40 animate-ping"></div>
            <div class="w-20 h-20 rounded-full bg-gradient-to-tr from-amber-600 via-orange-400 to-yellow-200 shadow-[0_0_60px_rgba(251,191,36,0.8)]"></div>
        </div>
        <div class="space-y-2 max-w-lg">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-white tracking-widest">
                SOLAR SYSTEM <span class="text-transparent bg-clip-text bg-gradient-to-r from-amber-300 to-cyan-400">IMAX 3D</span>
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 leading-relaxed">
                Tích hợp Shader dung nham Mặt Trời, tán xạ khí quyển Rayleigh, vành đai băng đa tầng & hiệu ứng tỏa sáng điện ảnh Unreal Bloom.
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-3.5 bg-gradient-to-r from-amber-500 to-orange-600 hover:from-amber-400 hover:to-orange-500 font-orbitron font-bold text-xs sm:text-sm rounded-full text-slate-950 shadow-[0_0_30px_rgba(245,158,11,0.6)] transition active:scale-95">
            KHỞI CHẠY ĐỒ HỌA ĐIỆN ẢNH
        </button>
    </div>

    <!-- NHẠC NỀN -->
    <audio id="bgMusic" loop preload="auto">
        <source src="https://cdn.pixabay.com/download/audio/2022/11/22/audio_febc508520.mp3?filename=drift-phonk-126548.mp3" type="audio/mpeg">
    </audio>

    <!-- HUD GÓC TRÁI -->
    <header class="absolute top-4 left-4 z-20 hud-glass p-4 rounded-2xl w-80 space-y-3">
        <div class="flex justify-between items-center">
            <div class="flex items-center gap-2">
                <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span>
                <span class="text-[10px] font-bold tracking-widest text-slate-300">IMAX RENDER ENGINE</span>
            </div>
            <div class="flex gap-1.5">
                <button onclick="toggleSpaceAudio()" id="audioBtn" class="px-2 py-1 bg-slate-900/90 border border-slate-700 text-rose-300 rounded-lg text-[10px] font-bold hover:border-rose-400 transition">
                    🔊 Music
                </button>
                <button onclick="toggleHud()" id="hudToggleBtn" class="px-2 py-1 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-lg text-[10px] font-bold hover:border-cyan-400 transition">
                    👁️ Ẩn
                </button>
            </div>
        </div>

        <div id="hudBody" class="space-y-3 transition-all duration-300 overflow-hidden">
            <div class="grid grid-cols-2 gap-2 text-[10px]">
                <div class="bg-slate-950/60 p-2 rounded-xl border border-slate-800/80">
                    <div class="flex justify-between text-slate-400 mb-1"><span>⏳ Tốc độ</span><span id="speedVal" class="text-cyan-300">1.0x</span></div>
                    <input type="range" min="0" max="5" step="0.1" value="1" oninput="setSpeed(this.value)" class="w-full accent-cyan-400 h-1 cursor-pointer">
                </div>
                <div class="bg-slate-950/60 p-2 rounded-xl border border-slate-800/80">
                    <div class="flex justify-between text-slate-400 mb-1"><span>✨ Ánh sáng</span><span id="bloomVal" class="text-amber-300">1.2</span></div>
                    <input type="range" min="0" max="2.5" step="0.1" value="1.2" oninput="setBloom(this.value)" class="w-full accent-amber-400 h-1 cursor-pointer">
                </div>
            </div>

            <div class="border-t border-slate-800/80 pt-2.5 space-y-1">
                <div id="planetTitle" class="text-sm font-orbitron font-bold text-amber-300 tracking-wide">MẶT TRỜI (SOL)</div>
                <p id="planetDesc" class="text-[11px] text-slate-300 leading-relaxed">
                    Khối cầu plasma sôi sục với các dòng đối lưu nhiệt động lực học thời gian thực. Kéo chuột để xoay, lăn chuột để phóng to.
                </p>
                <div class="grid grid-cols-2 gap-1 pt-1.5 text-[10px] text-slate-400">
                    <div>Khoảng cách: <span id="planetDist" class="text-slate-200">0 AU</span></div>
                    <div>Nhiệt độ: <span id="planetTemp" class="text-amber-300">5,500 °C</span></div>
                </div>
            </div>
        </div>
    </header>

    <!-- THANH CHỌN HÀNH TINH DƯỚI ĐÁY -->
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
        let scene, camera, renderer, composer, bloomPass;
        let planets = {};
        let sunUniforms;
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, prevMouse = { x: 0, y: 0 };
        let camDist = 68, targetCamDist = 68;
        let camAngleX = 0.35, camAngleY = 0.2;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0;

        const planetDB = {
            sun: { name: "MẶT TRỜI (SOL)", desc: "Khối cầu plasma sôi sục với dòng đối lưu nhiệt động lực học GLSL thời gian thực.", dist: "0 AU", temp: "5,500 °C", zoom: 68 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Bề mặt đá bazan xám bạc lồi lõm hàng triệu hố thiên thạch.", dist: "0.39 AU", temp: "430 °C", zoom: 4.5 },
            venus: { name: "SAO KIM (VENUS)", desc: "Khí quyển siêu đặc cuộn xoáy mây axit sulfuric ánh vàng rực.", dist: "0.72 AU", temp: "462 °C", zoom: 6 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Tán xạ khí quyển xanh ngọc, mây 3D tách lớp & ánh đèn đô thị ban đêm.", dist: "1.00 AU", temp: "15 °C", zoom: 6.5 },
            mars: { name: "SAO HỎA (MARS)", desc: "Sa mạc oxit sắt đỏ rực với chỏm băng cực và hẻm núi sâu.", dist: "1.52 AU", temp: "-63 °C", zoom: 5.5 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Khí quyển dải băng tầng khí động học & cơn bão Vết Đỏ Lớn.", dist: "5.20 AU", temp: "-110 °C", zoom: 13 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Hệ thống vành đai băng đa tầng phân tách bởi khe Cassini sắc nét.", dist: "9.58 AU", temp: "-140 °C", zoom: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng khí metan xanh ngọc bích với vành đai dọc mờ ảo.", dist: "19.22 AU", temp: "-195 °C", zoom: 10 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Thế giới băng giá xanh thẫm với những luồng gió siêu thanh.", dist: "30.05 AU", temp: "-200 °C", zoom: 10 }
        };

        // SHADER DUNG NHAM MẶT TRỜI CHUYỂN ĐỘNG 3D (SIMPLEX NOISE)
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
                vec3 hotYellow = vec3(1.5, 1.25, 0.65);

                vec3 col = mix(darkSpot, midOrange, smoothstep(0.1, 0.55, plasma));
                col = mix(col, hotYellow, smoothstep(0.5, 0.9, plasma));

                // Hiệu ứng viền sáng rực (Limb brightening)
                float fresnel = pow(1.0 - max(dot(vNormal, vec3(0.0, 0.0, 1.0)), 0.0), 2.5);
                col += vec3(1.2, 0.5, 0.1) * fresnel * 0.8;

                gl_FragColor = vec4(col, 1.0);
            }
        `;

        // SHADER TÁN XẠ KHÍ QUYỂN HÀNH TINH (ATMOSPHERIC FRESNEL GLOW)
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

        // TẠO TEXTURE CHI TIẾT CAO + BUMP MAP TẠO ĐỘ LỒI LÕM 3D
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
                // Đại dương sâu thẳm đa sắc
                const seaGrad = ctx.createLinearGradient(0, 0, 0, h);
                seaGrad.addColorStop(0, '#e0f2fe'); // Băng cực Bắc
                seaGrad.addColorStop(0.12, '#0369a1');
                seaGrad.addColorStop(0.5, '#0284c7');
                seaGrad.addColorStop(0.88, '#0369a1');
                seaGrad.addColorStop(1, '#e0f2fe'); // Băng cực Nam
                ctx.fillStyle = seaGrad; ctx.fillRect(0, 0, w, h);

                // Lục địa + Dãy núi nổi khối (Bump)
                for (let i = 0; i < 65; i++) {
                    const cx = Math.random() * w, cy = 150 + Math.random() * (h - 300);
                    const rx = Math.random() * 190 + 50, ry = Math.random() * 110 + 30;
                    const rot = Math.random() * Math.PI;
                    ctx.fillStyle = i % 3 === 0 ? '#a16207' : '#15803d'; // Rừng xanh & sa mạc
                    ctx.beginPath(); ctx.ellipse(cx, cy, rx, ry, rot, 0, Math.PI * 2); ctx.fill();

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
                    bCtx.beginPath(); bCtx.arc(x, y, r, 0, Math.PI * 2); bCtx.fill();
                }
                // Chỏm băng 2 cực
                ctx.fillStyle = '#f8fafc';
                ctx.fillRect(0, 0, w, 45); ctx.fillRect(0, h - 45, w, 45);
            } else if (type === 'jupiter') {
                const palette = ['#451a03', '#92400e', '#d97706', '#fef3c7', '#b45309', '#78350f', '#fde68a'];
                let y = 0;
                while (y < h) {
                    const bandH = Math.random() * 28 + 8;
                    ctx.fillStyle = palette[Math.floor(Math.random() * palette.length)];
                    ctx.fillRect(0, y, w, bandH);
                    // Sóng nhiễu loạn biên giới các dải khí
                    for (let x = 0; x < w; x += 24) {
                        ctx.beginPath();
                        ctx.arc(x, y, Math.random() * 12 + 4, 0, Math.PI * 2);
                        ctx.fill();
                    }
                    y += bandH;
                }
                // Cơn bão Vết Đỏ Lớn (Great Red Spot)
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

        // TEXTURE VÀNH ĐAI SAO THỔ ĐA TẦNG CÓ KHE HỞ CASSINI
        function createSaturnRingTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 1024; canvas.height = 64;
            const ctx = canvas.getContext('2d');
            const grad = ctx.createLinearGradient(0, 0, 1024, 0);
            grad.addColorStop(0.0, 'rgba(254, 240, 138, 0.0)');
            grad.addColorStop(0.15, 'rgba(180, 150, 90, 0.35)');
            grad.addColorStop(0.35, 'rgba(254, 243, 199, 0.92)');
            grad.addColorStop(0.58, 'rgba(234, 179, 8, 0.85)');
            grad.addColorStop(0.63, 'rgba(0, 0, 0, 0.05)'); // Khe hở Cassini
            grad.addColorStop(0.68, 'rgba(253, 224, 71, 0.8)');
            grad.addColorStop(0.92, 'rgba(202, 138, 4, 0.45)');
            grad.addColorStop(1.0, 'rgba(254, 240, 138, 0.0)');
            ctx.fillStyle = grad;
            ctx.fillRect(0, 0, 1024, 64);
            return new THREE.CanvasTexture(canvas);
        }

        // TẠO NỀN VŨ TRỤ NGÀN SAO + TINH VÂN ĐA SẮC (NEBULA)
        function createDeepSpaceBackdrop() {
            // 1. Ngàn sao nhiều kích cỡ & nhiệt độ màu
            const starCount = 9000;
            const geo = new THREE.BufferGeometry();
            const pos = new Float32Array(starCount * 3);
            const cols = new Float32Array(starCount * 3);
            const starPalette = [new THREE.Color(0xffffff), new THREE.Color(0xbae6fd), new THREE.Color(0xfef08a), new THREE.Color(0xfecdd3)];

            for (let i = 0; i < starCount; i++) {
                const r = 600 + Math.random() * 2400;
                const theta = Math.random() * Math.PI * 2;
                const phi = Math.acos(2 * Math.random() - 1);
                pos[i * 3] = r * Math.sin(phi) * Math.cos(theta);
                pos[i * 3 + 1] = r * Math.sin(phi) * Math.sin(theta);
                pos[i * 3 + 2] = r * Math.cos(phi);

                const c = starPalette[i % starPalette.length];
                cols[i * 3] = c.r; cols[i * 3 + 1] = c.g; cols[i * 3 + 2] = c.b;
            }
            geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
            geo.setAttribute('color', new THREE.BufferAttribute(cols, 3));
            scene.add(new THREE.Points(geo, new THREE.PointsMaterial({ size: 2.2, vertexColors: true, transparent: true, opacity: 0.9 })));

            // 2. Tinh vân Nebula mờ ảo
            const nebCanvas = document.createElement('canvas');
            nebCanvas.width = 512; nebCanvas.height = 512;
            const nCtx = nebCanvas.getContext('2d');
            const nGrad = nCtx.createRadialGradient(256, 256, 10, 256, 256, 250);
            nGrad.addColorStop(0, 'rgba(56, 189, 248, 0.16)');
            nGrad.addColorStop(0.5, 'rgba(139, 92, 246, 0.08)');
            nGrad.addColorStop(1, 'rgba(0, 0, 0, 0)');
            nCtx.fillStyle = nGrad; nCtx.fillRect(0, 0, 512, 512);
            const nebTex = new THREE.CanvasTexture(nebCanvas);

            for (let i = 0; i < 18; i++) {
                const sprite = new THREE.Sprite(new THREE.SpriteMaterial({ map: nebTex, blending: THREE.AdditiveBlending, transparent: true, opacity: 0.55 }));
                sprite.position.set((Math.random() - 0.5) * 1800, (Math.random() - 0.5) * 900, (Math.random() - 0.5) * 1800);
                sprite.scale.set(900, 900, 1);
                scene.add(sprite);
            }
        }

        function init3D() {
            const container = document.getElementById('canvasContainer');
            scene = new THREE.Scene();

            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 10000);

            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.toneMapping = THREE.ReinhardToneMapping;
            renderer.toneMappingExposure = 1.15;
            container.appendChild(renderer.domElement);

            // CẤU HÌNH UNREAL BLOOM POST-PROCESSING (TỎA SÁNG ĐIỆN ẢNH)
            const renderScene = new THREE.RenderPass(scene, camera);
            bloomPass = new THREE.UnrealBloomPass(new THREE.Vector2(window.innerWidth, window.innerHeight), 1.2, 0.65, 0.35);
            composer = new THREE.EffectComposer(renderer);
            composer.addPass(renderScene);
            composer.addPass(bloomPass);

            // ÁNH SÁNG MẶT TRỜI VẬT LÝ (ĐỔ BÓNG ĐÊM SÂU THẲM)
            const sunLight = new THREE.PointLight(0xfff5e4, 3.2, 2000, 0.25);
            scene.add(sunLight);
            scene.add(new THREE.AmbientLight(0x0f172a, 0.35)); // Bóng tối mặt khuất sâu có chiều sâu

            createDeepSpaceBackdrop();

            // MẶT TRỜI GLSL SHADER
            sunUniforms = { uTime: { value: 0 } };
            const sunMat = new THREE.ShaderMaterial({
                uniforms: sunUniforms,
                vertexShader: sunVertexShader,
                fragmentShader: sunFragmentShader
            });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(5.2, 64, 64), sunMat);
            // Hào quang nhật hoa bao quanh Mặt Trời
            const corona = new THREE.Mesh(new THREE.SphereGeometry(6.8, 64, 64), createAtmosphereMaterial(0xff8c00, 1.35, 3.2));
            sun.add(corona);
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0 };

            // CẤU HÌNH 8 HÀNH TINH
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
                // Đường quỹ đạo thanh mảnh sang trọng
                const orbitGeo = new THREE.RingGeometry(cfg.dist - 0.04, cfg.dist + 0.04, 180);
                const orbitMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.14 });
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

                // Tán xạ khí quyển Rayleigh
                if (cfg.atmos) {
                    const atmosMesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size * 1.12, 48, 48), createAtmosphereMaterial(cfg.atmos, 1.15, 3.8));
                    mesh.add(atmosMesh);
                }

                // Lớp mây 3D Trái Đất
                if (cfg.clouds) {
                    const cloudMaps = generatePlanetMaps('clouds');
                    const cloudMat = new THREE.MeshStandardMaterial({
                        map: cloudMaps.map,
                        transparent: true,
                        opacity: 0.85,
                        depthWrite: false
                    });
                    const cloudMesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size * 1.022, 64, 64), cloudMat);
                    mesh.add(cloudMesh);
                    mesh.userData.cloudMesh = cloudMesh;
                }

                // Mặt Trăng quay quanh Trái Đất
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

                // Vành đai Sao Thổ UV Radial chuẩn xác
                if (cfg.rings) {
                    const rGeo = new THREE.RingGeometry(cfg.size * 1.35, cfg.size * 2.45, 128);
                    // Chỉnh lại UV để texture vành đai trải đều từ trong ra ngoài
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
                        opacity: 0.92,
                        roughness: 0.5
                    });
                    const ringMesh = new THREE.Mesh(rGeo, rMat);
                    ringMesh.rotation.x = Math.PI / 2;
                    mesh.add(ringMesh);
                }

                planets[cfg.id] = { mesh, distance: cfg.dist, speed: cfg.speed, angle: Math.random() * Math.PI * 2 };
            });

            // ĐIỀU KHIỂN CAMERA MƯỢT MÀ
            window.addEventListener('mousedown', e => { isDragging = true; prevMouse = { x: e.clientX, y: e.clientY }; });
            window.addEventListener('mousemove', e => {
                if (!isDragging) return;
                camAngleY -= (e.clientX - prevMouse.x) * 0.005;
                camAngleX = Math.max(-1.45, Math.min(1.45, camAngleX - (e.clientY - prevMouse.y) * 0.005));
                prevMouse = { x: e.clientX, y: e.clientY };
            });
            window.addEventListener('mouseup', () => isDragging = false);
            window.addEventListener('wheel', e => {
                targetCamDist = Math.max(2.5, Math.min(400, targetCamDist + e.deltaY * 0.06));
            });
            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
                composer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function startExperience() {
            document.getElementById('startOverlay').style.display = 'none';
            init3D();
            const bg = document.getElementById('bgMusic');
            if (bg) { bg.volume = 0.7; bg.play().catch(() => {}); }
        }

        function setSpeed(v) {
            timeSpeed = parseFloat(v);
            document.getElementById('speedVal').innerText = timeSpeed.toFixed(1) + 'x';
        }

        function setBloom(v) {
            if (bloomPass) bloomPass.strength = parseFloat(v);
            document.getElementById('bloomVal').innerText = parseFloat(v).toFixed(1);
        }

        function toggleHud() {
            isHudVisible = !isHudVisible;
            const body = document.getElementById('hudBody');
            body.style.maxHeight = isHudVisible ? '260px' : '0px';
            body.style.opacity = isHudVisible ? '1' : '0';
            document.getElementById('hudToggleBtn').innerText = isHudVisible ? '👁️ Ẩn' : '👁️ Hiện';
        }

        function toggleSpaceAudio() {
            const bg = document.getElementById('bgMusic');
            if (!bg) return;
            if (bg.paused) { bg.play(); document.getElementById('audioBtn').innerText = '🔊 Music'; }
            else { bg.pause(); document.getElementById('audioBtn').innerText = '🔇 Muted'; }
        }

        function selectPlanet(id) {
            focusedObject = id;
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

            // Camera theo tiêu điểm mượt
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
