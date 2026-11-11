
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interstellar 3D Engine - Gargantua Black Hole & Solar System</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Share+Tech+Mono&display=swap');
        body { font-family: 'Share Tech Mono', monospace; background: #000000; color: #38bdf8; overflow: hidden; margin: 0; }
        .font-orbitron { font-family: 'Orbitron', sans-serif; }
        .hud-panel { background: rgba(3, 7, 18, 0.92); backdrop-filter: blur(16px); border: 1px solid rgba(56, 189, 248, 0.4); box-shadow: 0 0 30px rgba(56, 189, 248, 0.25); }
        .planet-btn { background: rgba(15, 23, 42, 0.85); border: 1px solid rgba(56, 189, 248, 0.3); transition: all 0.25s ease; }
        .planet-btn:hover, .planet-btn.active { border-color: #38bdf8; box-shadow: 0 0 18px rgba(56, 189, 248, 0.6); transform: translateY(-2px); background: rgba(14, 165, 233, 0.2); }
        .custom-scrollbar::-webkit-scrollbar { height: 6px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: rgba(15, 23, 42, 0.6); border-radius: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #0284c7; border-radius: 4px; }
        @keyframes pulseGlow {
            0% { box-shadow: 0 0 15px rgba(244, 63, 94, 0.4); }
            50% { box-shadow: 0 0 35px rgba(244, 63, 94, 0.9); }
            100% { box-shadow: 0 0 15px rgba(244, 63, 94, 0.4); }
        }
        .cinematic-glow { animation: pulseGlow 1.8s infinite; }
    </style>
</head>
<body class="w-screen h-screen flex flex-col justify-between select-none">

    <!-- OVERLAY KHỞI ĐỘNG -->
    <div id="startOverlay" class="fixed inset-0 z-50 bg-slate-950/95 backdrop-blur-xl flex flex-col items-center justify-center p-6 text-center space-y-6">
        <div class="relative flex items-center justify-center">
            <div class="w-32 h-32 rounded-full border-2 border-amber-500/50 animate-ping absolute"></div>
            <div class="text-7xl">🕳️</div>
        </div>
        <div class="space-y-2">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-amber-400 via-rose-500 to-purple-500 tracking-wider">
                GARGANTUA CINEMATIC ENGINE
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 max-w-md mx-auto leading-relaxed">
                Mô phỏng Hố đen điện ảnh Shader GLSL, đĩa bồi tích phát sáng volumetric & vụ hút 1,000 năm thời gian.
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-amber-600 via-rose-600 to-purple-600 font-orbitron font-bold text-xs sm:text-sm rounded-2xl text-white cinematic-glow transition active:scale-95 border border-amber-400 flex items-center gap-3">
            <span>⚡</span> VÀO TRẢI NGHIỆM ĐIỆN ẢNH HIGH-END
        </button>
    </div>

    <!-- AUDIO PLAYER -->
    <audio id="bgMusic" loop preload="auto">
        <source src="https://cdn.pixabay.com/download/audio/2022/11/22/audio_febc508520.mp3?filename=drift-phonk-126548.mp3" type="audio/mpeg">
    </audio>

    <!-- HUD ĐIỀU KHIỂN -->
    <header id="mainHud" class="absolute top-4 left-4 z-20 transition-all duration-300 pointer-events-auto">
        <div class="hud-panel p-4 rounded-2xl max-w-sm space-y-3 relative">
            <div class="flex justify-between items-center gap-3">
                <span class="text-[10px] font-bold tracking-widest text-slate-400">INTERSTELLAR SYSTEM COMMAND</span>
                <div class="flex items-center gap-2">
                    <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-2.5 py-1 bg-rose-950/80 border border-rose-400 text-rose-300 rounded-lg text-[11px] font-bold hover:bg-rose-900 transition flex items-center gap-1.5 shadow-[0_0_12px_rgba(244,63,94,0.4)]">
                        <span id="audioIcon">🔊</span> <span id="audioStatus">Straywav: ON</span>
                    </button>
                    <button onclick="toggleHudInfo()" id="toggleHudBtn" class="px-2 py-1 bg-slate-800 border border-slate-600 text-slate-200 rounded-lg text-[11px] font-bold hover:bg-slate-700 transition">
                        👁️ Ẩn
                    </button>
                </div>
            </div>

            <!-- CHẾ ĐỘ THỜI GIAN & HỐ ĐEN -->
            <div class="bg-rose-950/40 p-2.5 rounded-xl border border-rose-500/40 space-y-2">
                <div class="text-[10px] text-rose-300 font-bold uppercase tracking-wider">🕳️ Hố Đen & Lực Hấp Dẫn 1,000 Năm:</div>
                <div class="grid grid-cols-2 gap-2 text-[10px]">
                    <button onclick="triggerBlackHoleDecay()" id="decayBtn" class="py-2 bg-rose-700 border border-rose-400 text-white font-bold rounded-lg hover:bg-rose-600 transition shadow-[0_0_15px_rgba(244,63,94,0.5)]">
                        ⏩ Tua 1000 Năm (Hút)
                    </button>
                    <button onclick="resetSolarSystem()" class="py-2 bg-slate-800 border border-slate-600 text-slate-200 font-bold rounded-lg hover:bg-slate-700 transition">
                        🔄 Khôi Phục Hệ
                    </button>
                </div>
            </div>

            <!-- QUAN SÁT TIÊU ĐIỂM -->
            <div class="bg-indigo-950/50 p-2.5 rounded-xl border border-indigo-500/40 space-y-2">
                <div class="text-[10px] text-indigo-300 font-bold uppercase tracking-wider">🎯 Góc Nhìn Tiêu Điểm:</div>
                <div class="grid grid-cols-2 gap-1.5 text-[9px]">
                    <button onclick="selectTarget('sun')" id="btn-target-sun" class="py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-200 font-bold rounded-lg transition">
                        ☀️ Hệ Mặt Trời
                    </button>
                    <button onclick="selectTarget('gargantua')" id="btn-target-bh" class="py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition">
                        🕳️ Hố Đen Gargantua
                    </button>
                </div>
            </div>

            <div class="grid grid-cols-2 gap-2 text-[10px]">
                <div class="bg-slate-900/80 p-2 rounded-xl border border-slate-800 space-y-1">
                    <span class="text-rose-300 font-bold">🎧 Nhạc Vol:</span>
                    <input type="range" id="volumeSlider" min="0" max="1" step="0.05" value="0.75" onchange="changeVolume(this.value)" class="w-full accent-rose-500 h-1 cursor-pointer">
                </div>
                <div class="bg-slate-900/80 p-2 rounded-xl border border-slate-800 space-y-1">
                    <span class="text-cyan-300 font-bold">⏳ Tốc độ thời gian:</span>
                    <input type="range" id="timeSpeedSlider" min="0" max="5" step="0.1" value="1" onchange="changeTimeSpeed(this.value)" class="w-full accent-cyan-400 h-1 cursor-pointer">
                </div>
            </div>

            <div id="hudContent" class="space-y-3 transition-all duration-300 overflow-hidden">
                <div class="border-t border-slate-800 pt-2">
                    <div id="planetTitle" class="text-sm font-orbitron font-bold text-amber-300">TOÀN CẢNH HỆ MẶT TRỜI</div>
                    <p id="planetDesc" class="text-xs text-slate-300 leading-relaxed mt-1">
                        Nhấn nút "Tua 1000 Năm (Hút)" để theo dõi toàn bộ các hành tinh bị xoắn ốc nuốt chửng vào Hố Đen Gargantua!
                    </p>
                </div>
            </div>
        </div>
    </header>

    <!-- THANH CHỌN HÀNH TINH -->
    <div class="absolute bottom-6 left-1/2 -translate-x-1/2 z-20 hud-panel px-4 py-3 rounded-2xl flex gap-2 overflow-x-auto max-w-[95vw] custom-scrollbar">
        <button id="btn-sun" onclick="selectPlanet('sun')" class="planet-btn active px-3 py-1.5 text-amber-300 rounded-xl text-xs font-bold whitespace-nowrap">☀️ Mặt Trời</button>
        <button id="btn-mercury" onclick="selectPlanet('mercury')" class="planet-btn px-3 py-1.5 text-slate-300 rounded-xl text-xs font-bold whitespace-nowrap">🪨 Sao Thủy</button>
        <button id="btn-venus" onclick="selectPlanet('venus')" class="planet-btn px-3 py-1.5 text-amber-200 rounded-xl text-xs font-bold whitespace-nowrap">🌕 Sao Kim</button>
        <button id="btn-earth" onclick="selectPlanet('earth')" class="planet-btn px-3 py-1.5 text-blue-400 rounded-xl text-xs font-bold whitespace-nowrap">🌍 Trái Đất</button>
        <button id="btn-mars" onclick="selectPlanet('mars')" class="planet-btn px-3 py-1.5 text-rose-400 rounded-xl text-xs font-bold whitespace-nowrap">🔴 Sao Hỏa</button>
        <button id="btn-jupiter" onclick="selectPlanet('jupiter')" class="planet-btn px-3 py-1.5 text-orange-300 rounded-xl text-xs font-bold whitespace-nowrap">🪐 Sao Mộc</button>
        <button id="btn-saturn" onclick="selectPlanet('saturn')" class="planet-btn px-3 py-1.5 text-yellow-300 rounded-xl text-xs font-bold whitespace-nowrap">🪐 Sao Thổ</button>
        <button id="btn-uranus" onclick="selectPlanet('uranus')" class="planet-btn px-3 py-1.5 text-cyan-300 rounded-xl text-xs font-bold whitespace-nowrap">💎 Sao Thiên Vương</button>
        <button id="btn-neptune" onclick="selectPlanet('neptune')" class="planet-btn px-3 py-1.5 text-indigo-400 rounded-xl text-xs font-bold whitespace-nowrap">🔵 Sao Hải Vương</button>
    </div>

    <!-- CANVAS 3D -->
    <div id="canvasContainer" class="absolute inset-0 z-0"></div>

    <script>
        let scene, camera, renderer;
        let planets = {}, orbits = {};
        let gargantuaGroup, blackHoleShaderMat;
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, previousMousePosition = { x: 0, y: 0 };
        let cameraDistance = 65, targetCameraDistance = 65;
        let cameraAngleX = 0.4, cameraAngleY = 0.0;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0;
        let isDecaying = false;
        let decayProgress = 0.0;

        const planetDatabase = {
            sun: { name: "MẶT TRỜI (SUN)", desc: "Ngôi sao trung tâm Hệ Mặt Trời.", dist: "0 AU", temp: "5,500 °C", radius: "696,340 km", zoomDist: 65 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Hành tinh đá nhỏ nhất gần Mặt Trời.", dist: "0.39 AU", temp: "-180 đến 430 °C", radius: "2,439 km", zoomDist: 6 },
            venus: { name: "SAO KIM (VENUS)", desc: "Khí quyển CO2 cực kỳ nóng cháy.", dist: "0.72 AU", temp: "462 °C", radius: "6,051 km", zoomDist: 7 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Hành tinh sự sống xanh lam mượt mà.", dist: "1.00 AU", temp: "15 °C", radius: "6,371 km", zoomDist: 8 },
            mars: { name: "SAO HỎA (MARS)", desc: "Hành tinh Đỏ sa mạc lạnh giá.", dist: "1.52 AU", temp: "-63 °C", radius: "3,389 km", zoomDist: 7 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Gã khổng lồ khí lớn nhất.", dist: "5.20 AU", temp: "-110 °C", radius: "69,911 km", zoomDist: 14 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Vành đai băng sáng lấp lánh.", dist: "9.58 AU", temp: "-140 °C", radius: "58,232 km", zoomDist: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng xanh ngọc lam.", dist: "19.22 AU", temp: "-195 °C", radius: "25,362 km", zoomDist: 11 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Hành tinh xanh thẫm xa nhất.", dist: "30.05 AU", temp: "-200 °C", radius: "24,622 km", zoomDist: 11 },
            gargantua: { name: "HỐ ĐEN GARGANTUA", desc: "Hố đen siêu trọng với vành bồi tích nhiệt lượng phát sáng điện ảnh.", dist: "150 AU", temp: "10,000,000 °C", radius: "Khổng lồ", zoomDist: 40 }
        };

        // GLSL SHADER CHO HỐ ĐEN INTERSTELLAR (GARGANTUA ACCRETION DISK)
        const GargantuaShader = {
            uniforms: {
                uTime: { value: 0 },
                uColorInner: { value: new THREE.Color(0xfffbef) },
                uColorOuter: { value: new THREE.Color(0xff4500) }
            },
            vertexShader: `
                varying vec2 vUv;
                varying vec3 vPosition;
                void main() {
                    vUv = uv;
                    vPosition = position;
                    gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
                }
            `,
            fragmentShader: `
                uniform float uTime;
                uniform vec3 uColorInner;
                uniform vec3 uColorOuter;
                varying vec2 vUv;
                varying vec3 vPosition;

                void main() {
                    vec2 uv = vUv - 0.5;
                    float dist = length(uv) * 2.0;
                    
                    // Chân trời sự kiện đen tuyệt đối
                    if (dist < 0.35) {
                        gl_FragColor = vec4(0.0, 0.0, 0.0, 1.0);
                        return;
                    }

                    // Vành bồi tích phát sáng xoáy sáng
                    float angle = atan(uv.y, uv.x);
                    float noise = sin(angle * 8.0 + uTime * 3.0) * 0.08 + cos(dist * 20.0 - uTime * 2.0) * 0.05;
                    float ringPattern = smoothstep(0.35, 0.45, dist + noise) * (1.0 - smoothstep(0.85, 1.0, dist));

                    vec3 color = mix(uColorInner, uColorOuter, (dist - 0.35) / 0.65);
                    float alpha = ringPattern * 0.95;

                    gl_FragColor = vec4(color * (1.5 + noise * 2.0), alpha);
                }
            `
        };

        function toggleHudInfo() {
            isHudVisible = !isHudVisible;
            const content = document.getElementById("hudContent");
            const btn = document.getElementById("toggleHudBtn");
            content.style.maxHeight = isHudVisible ? "300px" : "0px";
            content.style.opacity = isHudVisible ? "1" : "0";
            btn.innerText = isHudVisible ? "👁️ Ẩn" : "👁️ Hiện";
        }

        function startExperience() {
            document.getElementById('startOverlay').style.display = 'none';
            init3D();
            const bgAudio = document.getElementById('bgMusic');
            if (bgAudio) { bgAudio.volume = 0.75; bgAudio.play().catch(() => {}); }
        }

        function changeVolume(val) {
            const bgAudio = document.getElementById('bgMusic');
            if (bgAudio) bgAudio.volume = parseFloat(val);
        }

        function changeTimeSpeed(val) { timeSpeed = parseFloat(val); }

        function triggerBlackHoleDecay() {
            isDecaying = true;
            document.getElementById('planetTitle').innerText = "CẢNH BÁO: TUA 1000 NĂM (VỤ HÚT TRỌNG LỰC)";
            document.getElementById('planetDesc').innerText = "Hố Đen Gargantua đang gia tăng sức hút, kéo toàn bộ các hành tinh xoắn ốc chìm dần vào Chân trời sự kiện!";
        }

        function resetSolarSystem() {
            isDecaying = false;
            decayProgress = 0.0;
            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.mesh) p.mesh.visible = true;
            });
            document.getElementById('planetTitle').innerText = "ĐÃ KHÔI PHỤC HỆ MẶT TRỜI";
            document.getElementById('planetDesc').innerText = "Trở lại trạng thái quỹ đạo ban đầu.";
        }

        function selectTarget(target) {
            if (target === 'gargantua') {
                focusedObject = 'gargantua';
                targetCameraDistance = 40;
                document.getElementById('planetTitle').innerText = "HỐ ĐEN GARGANTUA";
                document.getElementById('planetDesc').innerText = "Hố đen điện ảnh với đĩa bồi tích GLSL rực rỡ nhiệt lượng.";
            } else {
                selectPlanet('sun');
            }
        }

        function createPhotorealisticTexture(type) {
            const canvas = document.createElement('canvas');
            canvas.width = 1024; canvas.height = 512;
            const ctx = canvas.getContext('2d');

            if (type === 'sun') {
                const grad = ctx.createRadialGradient(512, 256, 20, 512, 256, 500);
                grad.addColorStop(0, '#ffffff'); grad.addColorStop(0.3, '#fde047'); grad.addColorStop(0.7, '#f97316'); grad.addColorStop(1, '#7c2d12');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, 1024, 512);
            } else if (type === 'mercury') {
                ctx.fillStyle = '#64748b'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#334155'; for(let i=0; i<250; i++) ctx.fillRect(Math.random()*1024, Math.random()*512, Math.random()*8+2, Math.random()*8+2);
            } else if (type === 'venus') {
                ctx.fillStyle = '#d97706'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#fef08a'; for(let i=0; i<60; i++) ctx.fillRect(0, Math.random()*512, 1024, Math.random()*20+5);
            } else if (type === 'earth') {
                ctx.fillStyle = '#0284c7'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#15803d'; for(let i=0; i<45; i++) { ctx.beginPath(); ctx.ellipse(Math.random()*1024, Math.random()*512, Math.random()*130+40, Math.random()*70+20, Math.random()*Math.PI, 0, Math.PI*2); ctx.fill(); }
                ctx.fillStyle = '#fef08a'; for(let i=0; i<250; i++) ctx.fillRect(Math.random()*1024, Math.random()*512, 2, 2);
            } else if (type === 'clouds') {
                ctx.clearRect(0, 0, 1024, 512);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.75)';
                for(let i=0; i<70; i++) { ctx.beginPath(); ctx.ellipse(Math.random()*1024, Math.random()*512, Math.random()*110+30, Math.random()*30+10, 0, 0, Math.PI*2); ctx.fill(); }
            } else if (type === 'mars') {
                ctx.fillStyle = '#b91c1c'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#7f1d1d'; for(let i=0; i<150; i++) ctx.fillRect(Math.random()*1024, Math.random()*512, Math.random()*15+3, Math.random()*15+3);
            } else if (type === 'jupiter') {
                const colors = ['#78350f', '#b45309', '#f59e0b', '#fde047', '#d97706'];
                for(let y=0; y<512; y+=12) { ctx.fillStyle = colors[Math.floor(Math.random()*colors.length)]; ctx.fillRect(0, y, 1024, 12); }
                ctx.fillStyle = '#dc2626'; ctx.beginPath(); ctx.ellipse(650, 320, 75, 48, 0, 0, Math.PI*2); ctx.fill();
            } else if (type === 'saturn') {
                const colors = ['#ca8a04', '#eab308', '#fef08a', '#a16207'];
                for(let y=0; y<512; y+=10) { ctx.fillStyle = colors[Math.floor(Math.random()*colors.length)]; ctx.fillRect(0, y, 1024, 10); }
            } else if (type === 'uranus') {
                ctx.fillStyle = '#06b6d4'; ctx.fillRect(0, 0, 1024, 512);
            } else if (type === 'neptune') {
                ctx.fillStyle = '#1e1b4b'; ctx.fillRect(0, 0, 1024, 512);
            }
            return new THREE.CanvasTexture(canvas);
        }

        // KHỔNG LỒ HỐ ĐEN INTERSTELLAR (GARGANTUA)
        function createGargantuaBlackHole() {
            gargantuaGroup = new THREE.Group();
            gargantuaGroup.position.set(120, 20, -100);

            // Chân trời sự kiện đen tuyền
            const coreGeo = new THREE.SphereGeometry(6, 64, 64);
            const coreMat = new THREE.MeshBasicMaterial({ color: 0x000000 });
            const core = new THREE.Mesh(coreGeo, coreMat);
            gargantuaGroup.add(core);

            // Đĩa bồi tích Shader GLSL
            const diskGeo = new THREE.PlaneGeometry(36, 36);
            blackHoleShaderMat = new THREE.ShaderMaterial({
                uniforms: GargantuaShader.uniforms,
                vertexShader: GargantuaShader.vertexShader,
                fragmentShader: GargantuaShader.fragmentShader,
                transparent: true,
                side: THREE.DoubleSide,
                blending: THREE.AdditiveBlending
            });
            const disk = new THREE.Mesh(diskGeo, blackHoleShaderMat);
            disk.rotation.x = Math.PI / 2.3;
            gargantuaGroup.add(disk);

            // Vòng thấu kính hấp dẫn bao quanh (Gravitational Lens Halo)
            const haloGeo = new THREE.SphereGeometry(7.2, 32, 32);
            const haloMat = new THREE.MeshBasicMaterial({ color: 0xf97316, transparent: true, opacity: 0.25, side: THREE.BackSide });
            gargantuaGroup.add(new THREE.Mesh(haloGeo, haloMat));

            scene.add(gargantuaGroup);
        }

        function init3D() {
            const container = document.getElementById('canvasContainer');
            scene = new THREE.Scene();

            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 10000);
            
            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.toneMapping = THREE.ACESFilmicToneMapping;
            container.appendChild(renderer.domElement);

            const sunLight = new THREE.PointLight(0xffffff, 4.5, 2500);
            scene.add(sunLight);
            scene.add(new THREE.AmbientLight(0x1e293b, 0.8));

            // Ngàn sao sáng thanh lịch
            const starsGeo = new THREE.BufferGeometry();
            const starPos = new Float32Array(5000 * 3);
            for(let i=0; i<5000*3; i++) starPos[i] = (Math.random() - 0.5) * 2000;
            starsGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            scene.add(new THREE.Points(starsGeo, new THREE.PointsMaterial({ color: 0xffffff, size: 0.9 })));

            createGargantuaBlackHole();

            // MẶT TRỜI
            const sunMat = new THREE.MeshBasicMaterial({ map: createPhotorealisticTexture('sun') });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.8, 48, 48), sunMat);
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(6.4, 32, 32), new THREE.MeshBasicMaterial({ color: 0xf59e0b, transparent: true, opacity: 0.35, side: THREE.BackSide })));
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0, initialDist: 0 };

            const planetData = [
                { id: 'mercury', size: 0.55, distance: 9, speed: 0.022, texture: 'mercury' },
                { id: 'venus', size: 0.95, distance: 13, speed: 0.016, texture: 'venus' },
                { id: 'earth', size: 1.15, distance: 18, speed: 0.012, texture: 'earth', hasClouds: true },
                { id: 'mars', size: 0.75, distance: 23, speed: 0.009, texture: 'mars' },
                { id: 'jupiter', size: 2.7, distance: 34, speed: 0.006, texture: 'jupiter' },
                { id: 'saturn', size: 2.2, distance: 44, speed: 0.004, texture: 'saturn', hasRings: true },
                { id: 'uranus', size: 1.5, distance: 52, speed: 0.003, texture: 'uranus' },
                { id: 'neptune', size: 1.4, distance: 60, speed: 0.002, texture: 'neptune' }
            ];

            planetData.forEach(data => {
                const orbitLine = new THREE.Mesh(new THREE.RingGeometry(data.distance - 0.06, data.distance + 0.06, 128), new THREE.MeshBasicMaterial({ color: 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.15 }));
                orbitLine.rotation.x = Math.PI / 2;
                scene.add(orbitLine);
                orbits[data.id] = orbitLine;

                const pMat = new THREE.MeshStandardMaterial({ roughness: 0.7, map: createPhotorealisticTexture(data.texture) });
                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size, 48, 48), pMat);
                planetMesh.position.x = data.distance;
                scene.add(planetMesh);

                if (data.hasClouds) {
                    const cloudMat = new THREE.MeshStandardMaterial({ map: createPhotorealisticTexture('clouds'), transparent: true, blending: THREE.AdditiveBlending, opacity: 0.65 });
                    const cloudMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size * 1.02, 32, 32), cloudMat);
                    planetMesh.add(cloudMesh);
                }

                if (data.hasRings) {
                    const ring = new THREE.Mesh(new THREE.RingGeometry(2.9, 5.2, 64), new THREE.MeshStandardMaterial({ color: 0xfef08a, side: THREE.DoubleSide, transparent: true, opacity: 0.85 }));
                    ring.rotation.x = Math.PI / 2.2;
                    planetMesh.add(ring);
                }

                planets[data.id] = { mesh: planetMesh, distance: data.distance, initialDist: data.distance, speed: data.speed, angle: Math.random() * Math.PI * 2 };
            });

            window.addEventListener('mousedown', (e) => { isDragging = true; previousMousePosition = { x: e.clientX, y: e.clientY }; });
            window.addEventListener('mousemove', (e) => {
                if (!isDragging) return;
                const deltaX = e.clientX - previousMousePosition.x;
                const deltaY = e.clientY - previousMousePosition.y;

                cameraAngleY -= deltaX * 0.005;
                cameraAngleX = Math.max(-1.4, Math.min(1.4, cameraAngleX - deltaY * 0.005));
                previousMousePosition = { x: e.clientX, y: e.clientY };
            });

            window.addEventListener('mouseup', () => isDragging = false);
            window.addEventListener('wheel', (e) => {
                targetCameraDistance = Math.max(3, Math.min(1500, targetCameraDistance + e.deltaY * 0.8));
            });

            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function updateCameraPosition() {
            let targetPos = new THREE.Vector3(0, 0, 0);
            
            if (focusedObject === 'gargantua' && gargantuaGroup) {
                gargantuaGroup.getWorldPosition(targetPos);
            } else if (focusedObject && planets[focusedObject]) {
                planets[focusedObject].mesh.getWorldPosition(targetPos);
            }
            
            currentLookAt.lerp(targetPos, 0.07);
            cameraDistance += (targetCameraDistance - cameraDistance) * 0.08;

            camera.position.x = currentLookAt.x + cameraDistance * Math.sin(cameraAngleY) * Math.cos(cameraAngleX);
            camera.position.y = currentLookAt.y + cameraDistance * Math.sin(cameraAngleX);
            camera.position.z = currentLookAt.z + cameraDistance * Math.cos(cameraAngleY) * Math.cos(cameraAngleX);
            camera.lookAt(currentLookAt);
        }

        function selectPlanet(id) {
            focusedObject = id;
            document.querySelectorAll('.planet-btn').forEach(btn => btn.classList.remove('active'));
            const activeBtn = document.getElementById(`btn-${id}`);
            if (activeBtn) activeBtn.classList.add('active');

            const info = planetDatabase[id];
            if (info) {
                document.getElementById('planetTitle').innerText = info.name;
                document.getElementById('planetDesc').innerText = info.desc;
                targetCameraDistance = info.zoomDist;
            }
        }

        function animate() {
            requestAnimationFrame(animate);

            // Cập nhật Shader Hố Đen theo thời gian
            if (blackHoleShaderMat) blackHoleShaderMat.uniforms.uTime.value += 0.02 * timeSpeed;
            if (gargantuaGroup) gargantuaGroup.rotation.y += 0.002 * timeSpeed;

            // Xử lý cơ chế Hố Đen hút 1,000 năm sau
            if (isDecaying && gargantuaGroup) {
                decayProgress += 0.002 * timeSpeed;
                const bhWorldPos = new THREE.Vector3();
                gargantuaGroup.getWorldPosition(bhWorldPos);

                Object.keys(planets).forEach(id => {
                    const p = planets[id];
                    if (p.mesh && p.mesh.visible) {
                        p.mesh.position.lerp(bhWorldPos, decayProgress * 0.05);
                        if (p.mesh.position.distanceTo(bhWorldPos) < 7.0) {
                            p.mesh.visible = false; // Bị nuốt chửng hoàn toàn
                        }
                    }
                });
            }

            // Cập nhật quay quỹ đạo bình thường
            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0 && !isDecaying) {
                    p.angle += p.speed * timeSpeed;
                    p.mesh.position.x = Math.cos(p.angle) * p.distance;
                    p.mesh.position.z = Math.sin(p.angle) * p.distance;
                    p.mesh.rotation.y += 0.015 * timeSpeed;
                }
            });

            updateCameraPosition();
            renderer.render(scene, camera);
        }
    </script>
</body>
</html>
