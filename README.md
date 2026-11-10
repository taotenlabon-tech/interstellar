
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA 3D Solar System - Ultimate Spaceship & Comet Edition</title>
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
            <div class="w-32 h-32 rounded-full border-2 border-rose-500/50 animate-ping absolute"></div>
            <div class="text-7xl">🛸</div>
        </div>
        <div class="space-y-2">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-rose-500 via-cyan-400 to-amber-300 tracking-wider">
                NASA EYES // ULTIMATE SPACESHIP FLIGHT
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 max-w-md mx-auto leading-relaxed">
                Đã sửa lỗi xoay góc nhìn khi lái tàu. Dùng Chuột để liếc nhìn & W,A,S,D để bay lượn tự do!
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-rose-600 via-purple-600 to-cyan-600 font-orbitron font-bold text-xs sm:text-sm rounded-2xl text-white cinematic-glow transition active:scale-95 border border-rose-400 flex items-center gap-3">
            <span>⚡</span> CẤT CÁNH KHÁM PHÁ VŨ TRỤ
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
                <span class="text-[10px] font-bold tracking-widest text-slate-400">FLIGHT COMMAND CENTER</span>
                <div class="flex items-center gap-2">
                    <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-2.5 py-1 bg-rose-950/80 border border-rose-400 text-rose-300 rounded-lg text-[11px] font-bold hover:bg-rose-900 transition flex items-center gap-1.5 shadow-[0_0_12px_rgba(244,63,94,0.4)]">
                        <span id="audioIcon">🔊</span> <span id="audioStatus">Straywav: ON</span>
                    </button>
                    <button onclick="toggleHudInfo()" id="toggleHudBtn" class="px-2 py-1 bg-slate-800 border border-slate-600 text-slate-200 rounded-lg text-[11px] font-bold hover:bg-slate-700 transition">
                        👁️ Ẩn
                    </button>
                </div>
            </div>

            <!-- ĐIỀU KHIỂN TÀU VŨ TRỤ -->
            <div class="bg-rose-950/40 p-2.5 rounded-xl border border-rose-500/40 space-y-1.5">
                <div class="flex justify-between items-center">
                    <span class="text-[10px] text-rose-300 font-bold uppercase tracking-wider">🛸 Chế độ Lái Tàu:</span>
                    <button onclick="toggleSpaceshipMode()" id="spaceshipBtn" class="px-2 py-0.5 bg-rose-900 border border-rose-400 text-rose-100 text-[10px] font-bold rounded-md hover:bg-rose-800 transition">
                        Bật Lái Tàu
                    </button>
                </div>
                <div class="text-[9px] text-slate-300 leading-tight">
                    <b class="text-cyan-300">W, A, S, D</b> bay | <b class="text-cyan-300">R / F</b> lên xuống | <b class="text-cyan-300">Giữ Chuột Trái + Rê</b> để đổi hướng nhìn.
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

            <div class="flex gap-2">
                <button onclick="toggleCinematicMode()" id="cinematicBtn" class="w-1/2 py-1.5 bg-purple-950 border border-purple-500 text-purple-200 text-[10px] font-bold rounded-xl hover:bg-purple-900 transition">
                    🎬 Cinematic Tour
                </button>
                <button onclick="toggleScaleMode()" id="scaleModeBtn" class="w-1/2 py-1.5 bg-amber-950 border border-amber-500 text-amber-200 text-[10px] font-bold rounded-xl hover:bg-amber-900 transition">
                    🌌 Scale: Default
                </button>
            </div>

            <h1 class="text-lg font-orbitron font-bold text-cyan-400 tracking-wide">HỆ MẶT TRỜI CHÂN THỰC</h1>

            <div id="hudContent" class="space-y-3 transition-all duration-300 overflow-hidden">
                <div class="border-t border-slate-800 pt-2">
                    <div id="planetTitle" class="text-sm font-orbitron font-bold text-amber-300">TOÀN CẢNH HỆ MẶT TRỜI</div>
                    <p id="planetDesc" class="text-xs text-slate-300 leading-relaxed mt-1">
                        Khám phá không gian tự do với tàu vũ trụ, hiệu ứng khí quyển và sao chổi bay qua.
                    </p>
                </div>

                <div class="grid grid-cols-2 gap-2 text-[11px] border-t border-slate-800/80 pt-2 text-slate-300">
                    <div class="bg-slate-900/80 p-2 rounded-xl border border-slate-800">
                        <div class="text-[9px] text-slate-400 uppercase">Khoảng Cách Gốc</div>
                        <div id="statDistance" class="font-bold text-cyan-300">0 AU</div>
                    </div>
                    <div class="bg-slate-900/80 p-2 rounded-xl border border-slate-800">
                        <div class="text-[9px] text-slate-400 uppercase">Nhiệt Độ Trung Bình</div>
                        <div id="statTemp" class="font-bold text-rose-300">5,500 °C</div>
                    </div>
                    <div class="bg-slate-900/80 p-2 rounded-xl border border-slate-800">
                        <div class="text-[9px] text-slate-400 uppercase">Bán Kính Tương Đối</div>
                        <div id="statRadius" class="font-bold text-amber-300">696,340 km</div>
                    </div>
                    <div class="bg-slate-900/80 p-2 rounded-xl border border-slate-800">
                        <div class="text-[9px] text-slate-400 uppercase">Số Lượng Mặt Trăng</div>
                        <div id="statMoons" class="font-bold text-purple-300">0</div>
                    </div>
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
        let planets = {}, asteroidBelt = [];
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, previousMousePosition = { x: 0, y: 0 };
        let cameraDistance = 65, targetCameraDistance = 65;
        let cameraAngleX = 0.4, cameraAngleY = 0.0;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0, isRealScale = false, isCinematic = false, isSpaceshipMode = false;
        let cinematicTimer = 0, cinematicIndex = 0;
        let spaceshipPos = new THREE.Vector3(0, 15, 80);
        let keysPressed = {};
        let comet;
        let cometAngle = 0;

        const planetDatabase = {
            sun: { name: "MẶT TRỜI (SUN)", desc: "Ngôi sao trung tâm chứa plasma hydro cực nóng.", dist: "0 AU", temp: "5,500 °C", radius: "696,340 km", moons: "0", zoomDist: 65 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Hành tinh đá với bề mặt chi chít hố va chạm.", dist: "0.39 AU", temp: "-180 đến 430 °C", radius: "2,439 km", moons: "0", zoomDist: 6 },
            venus: { name: "SAO KIM (VENUS)", desc: "Bao phủ bởi tầng mây dày axit sulfuric màu vàng cam.", dist: "0.72 AU", temp: "462 °C", radius: "6,051 km", moons: "0", zoomDist: 7 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Hành tinh sự sống với tầng khí quyển xanh lam.", dist: "1.00 AU", temp: "15 °C", radius: "6,371 km", moons: "1 + Trạm ISS", zoomDist: 8 },
            mars: { name: "SAO HỎA (MARS)", desc: "Hành tinh Đỏ sa mạc lạnh giá với bụi oxit sắt.", dist: "1.52 AU", temp: "-63 °C", radius: "3,389 km", moons: "2", zoomDist: 7 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Gã khổng lồ khí với các dải mây bão xoáy cuộn.", dist: "5.20 AU", temp: "-110 °C", radius: "69,911 km", moons: "95", zoomDist: 14 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Hệ thống vành đai băng phức tạp sáng lấp lánh.", dist: "9.58 AU", temp: "-140 °C", radius: "58,232 km", moons: "146", zoomDist: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng khổng lồ màu xanh ngọc lam.", dist: "19.22 AU", temp: "-195 °C", radius: "25,362 km", moons: "28", zoomDist: 11 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Hành tinh xanh thẫm với gió siêu thanh mạnh nhất.", dist: "30.05 AU", temp: "-200 °C", radius: "24,622 km", moons: "16", zoomDist: 11 }
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

        function toggleScaleMode() {
            isRealScale = !isRealScale;
            document.getElementById("scaleModeBtn").innerText = isRealScale ? "🌌 Scale: Real" : "🌌 Scale: Default";
            Object.keys(planets).forEach(id => {
                const p = planets[id];
                const targetSize = isRealScale ? p.realSize : p.baseSize;
                p.mesh.scale.set(targetSize / p.baseSize, targetSize / p.baseSize, targetSize / p.baseSize);
            });
        }

        function toggleCinematicMode() {
            isCinematic = !isCinematic;
            isSpaceshipMode = false;
            document.getElementById("cinematicBtn").className = isCinematic ? "w-1/2 py-1.5 bg-rose-950 border border-rose-500 text-rose-200 text-[10px] font-bold rounded-xl animate-pulse" : "w-1/2 py-1.5 bg-purple-950 border border-purple-500 text-purple-200 text-[10px] font-bold rounded-xl hover:bg-purple-900 transition";
            document.getElementById("cinematicBtn").innerText = isCinematic ? "🎬 Stop Tour" : "🎬 Cinematic Tour";
        }

        function toggleSpaceshipMode() {
            isSpaceshipMode = !isSpaceshipMode;
            isCinematic = false;
            const btn = document.getElementById("spaceshipBtn");
            if (isSpaceshipMode) {
                btn.className = "px-2 py-0.5 bg-cyan-600 border border-cyan-300 text-white text-[10px] font-bold rounded-md shadow-[0_0_10px_rgba(56,189,248,0.6)]";
                btn.innerText = "Đang Lái Tàu";
                spaceshipPos.copy(camera.position);
            } else {
                btn.className = "px-2 py-0.5 bg-rose-900 border border-rose-400 text-rose-100 text-[10px] font-bold rounded-md hover:bg-rose-800 transition";
                btn.innerText = "Bật Lái Tàu";
            }
        }

        function toggleSpaceAudio() {
            const bgAudio = document.getElementById('bgMusic');
            if (!bgAudio) return;
            if (bgAudio.paused) { bgAudio.play(); document.getElementById('audioStatus').innerText = "Straywav: ON"; }
            else { bgAudio.pause(); document.getElementById('audioStatus').innerText = "Straywav: OFF"; }
        }

        function createPhotorealisticTexture(type) {
            const canvas = document.createElement('canvas');
            canvas.width = 1024; canvas.height = 512;
            const ctx = canvas.getContext('2d');

            if (type === 'sun') {
                const grad = ctx.createRadialGradient(512, 256, 20, 512, 256, 500);
                grad.addColorStop(0, '#ffffff'); grad.addColorStop(0.3, '#fde047'); grad.addColorStop(0.7, '#f97316'); grad.addColorStop(1, '#7c2d12');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, 1024, 512);
            } else if (type === 'earth') {
                ctx.fillStyle = '#0284c7'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#15803d';
                for(let i=0; i<45; i++) { ctx.beginPath(); ctx.ellipse(Math.random()*1024, Math.random()*512, Math.random()*130+40, Math.random()*70+20, Math.random()*Math.PI, 0, Math.PI*2); ctx.fill(); }
                ctx.fillStyle = '#fef08a'; for(let i=0; i<250; i++) ctx.fillRect(Math.random()*1024, Math.random()*512, 2, 2);
            } else if (type === 'clouds') {
                ctx.clearRect(0, 0, 1024, 512);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.75)';
                for(let i=0; i<70; i++) { ctx.beginPath(); ctx.ellipse(Math.random()*1024, Math.random()*512, Math.random()*110+30, Math.random()*30+10, 0, 0, Math.PI*2); ctx.fill(); }
            } else {
                ctx.fillStyle = '#64748b'; ctx.fillRect(0, 0, 1024, 512);
            }
            return new THREE.CanvasTexture(canvas);
        }

        function init3D() {
            const container = document.getElementById('canvasContainer');
            scene = new THREE.Scene();

            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 3000);
            
            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.toneMapping = THREE.ACESFilmicToneMapping;
            container.appendChild(renderer.domElement);

            const sunLight = new THREE.PointLight(0xffffff, 4.5, 2000);
            scene.add(sunLight);
            scene.add(new THREE.AmbientLight(0x1e293b, 0.8));

            // Ngàn sao
            const starsGeo = new THREE.BufferGeometry();
            const starsCnt = 6000;
            const starPos = new Float32Array(starsCnt * 3);
            for(let i=0; i<starsCnt*3; i++) starPos[i] = (Math.random() - 0.5) * 1800;
            starsGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            scene.add(new THREE.Points(starsGeo, new THREE.PointsMaterial({color: 0xffffff, size: 0.9})));

            // SAO CHỔI (COMET)
            const cometGeo = new THREE.SphereGeometry(0.4, 16, 16);
            const cometMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8 });
            comet = new THREE.Mesh(cometGeo, cometMat);
            scene.add(comet);

            // MẶT TRỜI
            const sunMat = new THREE.MeshBasicMaterial({ map: createPhotorealisticTexture('sun') });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.8, 48, 48), sunMat);
            const glowMat = new THREE.MeshBasicMaterial({ color: 0xf59e0b, transparent: true, opacity: 0.35, side: THREE.BackSide });
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(6.4, 32, 32), glowMat));
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0, baseSize: 4.8, realSize: 10.0 };

            // ĐAI TIỂU HÀNH TINH
            for(let i=0; i<450; i++) {
                const ast = new THREE.Mesh(new THREE.DodecahedronGeometry(0.12, 1), new THREE.MeshStandardMaterial({ color: 0x64748b }));
                const radius = 27 + Math.random() * 4, angle = Math.random() * Math.PI * 2;
                ast.position.set(Math.cos(angle)*radius, (Math.random()-0.5)*1.5, Math.sin(angle)*radius);
                scene.add(ast);
                asteroidBelt.push({ mesh: ast, angle: angle, speed: 0.002 + Math.random()*0.003, radius: radius });
            }

            // 8 HÀNH TINH
            const planetData = [
                { id: 'mercury', baseSize: 0.55, realSize: 0.4, distance: 9, speed: 0.022, texture: 'mercury' },
                { id: 'venus', baseSize: 0.95, realSize: 0.95, distance: 13, speed: 0.016, texture: 'venus' },
                { id: 'earth', baseSize: 1.15, realSize: 1.0, distance: 18, speed: 0.012, texture: 'earth', hasClouds: true, hasMoon: true, hasAtmosphere: true },
                { id: 'mars', baseSize: 0.75, realSize: 0.53, distance: 23, speed: 0.009, texture: 'mars' },
                { id: 'jupiter', baseSize: 2.7, realSize: 5.5, distance: 34, speed: 0.006, texture: 'jupiter' },
                { id: 'saturn', baseSize: 2.2, realSize: 4.5, distance: 44, speed: 0.004, texture: 'saturn', hasRings: true },
                { id: 'uranus', baseSize: 1.5, realSize: 2.2, distance: 52, speed: 0.003, texture: 'uranus' },
                { id: 'neptune', baseSize: 1.4, realSize: 2.1, distance: 60, speed: 0.002, texture: 'neptune' }
            ];

            planetData.forEach(data => {
                const orbitGroup = new THREE.Group();
                scene.add(orbitGroup);

                const orbitCurve = new THREE.EllipseCurve(0, 0, data.distance, data.distance * 0.95, 0, 2 * Math.PI, false, 0);
                const points = orbitCurve.getPoints(128);
                const orbitGeo = new THREE.BufferGeometry().setFromPoints(points.map(p => new THREE.Vector3(p.x, 0, p.y)));
                const orbitLine = new THREE.Line(orbitGeo, new THREE.LineBasicMaterial({ color: 0x38bdf8, transparent: true, opacity: 0.15 }));
                scene.add(orbitLine);

                const pMat = new THREE.MeshStandardMaterial({ roughness: 0.7, metalness: 0.1, map: createPhotorealisticTexture(data.texture) });
                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.baseSize, 48, 48), pMat);
                planetMesh.position.x = data.distance;
                orbitGroup.add(planetMesh);

                if (data.hasAtmosphere) {
                    const atmosMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8, transparent: true, opacity: 0.25, side: THREE.BackSide });
                    const atmos = new THREE.Mesh(new THREE.SphereGeometry(data.baseSize * 1.15, 32, 32), atmosMat);
                    planetMesh.add(atmos);
                }

                if (data.hasClouds) {
                    const cloudMat = new THREE.MeshStandardMaterial({ map: createPhotorealisticTexture('clouds'), transparent: true, blending: THREE.AdditiveBlending, opacity: 0.65 });
                    const cloudMesh = new THREE.Mesh(new THREE.SphereGeometry(data.baseSize * 1.02, 32, 32), cloudMat);
                    planetMesh.add(cloudMesh);
                    planetMesh.userData.cloudMesh = cloudMesh;
                }

                if (data.hasMoon) {
                    const moonMesh = new THREE.Mesh(new THREE.SphereGeometry(0.3, 24, 24), new THREE.MeshStandardMaterial({ color: 0xcbd5e1 }));
                    moonMesh.position.x = 2.4;
                    planetMesh.add(moonMesh);
                    planetMesh.userData.moonMesh = moonMesh;
                }

                if (data.hasRings) {
                    const ringGeo = new THREE.RingGeometry(2.9, 5.2, 64);
                    const ringMat = new THREE.MeshStandardMaterial({ color: 0xfef08a, side: THREE.DoubleSide, transparent: true, opacity: 0.85 });
                    const ring = new THREE.Mesh(ringGeo, ringMat);
                    ring.rotation.x = Math.PI / 2.2;
                    planetMesh.add(ring);
                }

                planets[data.id] = { mesh: planetMesh, distance: data.distance, speed: data.speed, angle: Math.random() * Math.PI * 2, baseSize: data.baseSize, realSize: data.realSize };
            });

            window.addEventListener('keydown', (e) => { keysPressed[e.key.toLowerCase()] = true; });
            window.addEventListener('keyup', (e) => { keysPressed[e.key.toLowerCase()] = false; });

            window.addEventListener('mousedown', (e) => {
                isDragging = true;
                previousMousePosition = { x: e.clientX, y: e.clientY };
            });

            window.addEventListener('mousemove', (e) => {
                if (!isDragging) return;
                const deltaX = e.clientX - previousMousePosition.x;
                const deltaY = e.clientY - previousMousePosition.y;

                if (isSpaceshipMode) {
                    // Xoay góc nhìn khi đang lái tàu
                    camera.rotation.y -= deltaX * 0.003;
                    camera.rotation.x -= deltaY * 0.003;
                } else {
                    // Xoay quanh hành tinh thông thường
                    cameraAngleY -= deltaX * 0.005;
                    cameraAngleX = Math.max(-1.4, Math.min(1.4, cameraAngleX - deltaY * 0.005));
                }
                previousMousePosition = { x: e.clientX, y: e.clientY };
            });

            window.addEventListener('mouseup', () => isDragging = false);
            window.addEventListener('wheel', (e) => { if (!isSpaceshipMode) targetCameraDistance = Math.max(3, Math.min(140, targetCameraDistance + e.deltaY * 0.04)); });

            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function updateCameraPosition() {
            if (isSpaceshipMode) {
                const speed = 0.6;
                let dir = new THREE.Vector3();
                camera.getWorldDirection(dir);
                
                if (keysPressed['w']) spaceshipPos.addScaledVector(dir, speed);
                if (keysPressed['s']) spaceshipPos.addScaledVector(dir, -speed);
                if (keysPressed['a']) {
                    const sideDir = new THREE.Vector3(-dir.z, 0, dir.x).normalize();
                    spaceshipPos.addScaledVector(sideDir, speed);
                }
                if (keysPressed['d']) {
                    const sideDir = new THREE.Vector3(-dir.z, 0, dir.x).normalize();
                    spaceshipPos.addScaledVector(sideDir, -speed);
                }
                if (keysPressed['r']) spaceshipPos.y += speed;
                if (keysPressed['f']) spaceshipPos.y -= speed;

                camera.position.copy(spaceshipPos);
                return;
            }

            if (isCinematic) {
                cinematicTimer += 0.016;
                if (cinematicTimer > 5) {
                    cinematicTimer = 0;
                    const keys = Object.keys(planets);
                    cinematicIndex = (cinematicIndex + 1) % keys.length;
                    selectPlanet(keys[cinematicIndex]);
                }
            }

            let targetPos = new THREE.Vector3(0, 0, 0);
            if (focusedObject && planets[focusedObject]) {
                const worldPos = new THREE.Vector3();
                planets[focusedObject].mesh.getWorldPosition(worldPos);
                targetPos.copy(worldPos);
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
                document.getElementById('statDistance').innerText = info.dist;
                document.getElementById('statTemp').innerText = info.temp;
                document.getElementById('statRadius').innerText = info.radius;
                document.getElementById('statMoons').innerText = info.moons;
                targetCameraDistance = info.zoomDist;
            }
        }

        function animate() {
            requestAnimationFrame(animate);

            // Cập nhật Sao Chổi bay qua hệ thống
            cometAngle += 0.008 * timeSpeed;
            comet.position.set(Math.cos(cometAngle) * 50, 5, Math.sin(cometAngle) * 70);

            asteroidBelt.forEach(ast => {
                ast.angle += ast.speed * timeSpeed;
                ast.mesh.position.x = Math.cos(ast.angle) * ast.radius;
                ast.mesh.position.z = Math.sin(ast.angle) * ast.radius * 0.95;
            });

            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
                    p.angle += p.speed * timeSpeed;
                    p.mesh.position.x = Math.cos(p.angle) * p.distance;
                    p.mesh.position.z = Math.sin(p.angle) * p.distance * 0.95;
                    p.mesh.rotation.y += 0.015 * timeSpeed;

                    if (p.mesh.userData.cloudMesh) p.mesh.userData.cloudMesh.rotation.y += 0.008 * timeSpeed;
                    if (p.mesh.userData.moonMesh) p.mesh.userData.moonMesh.rotation.y += 0.02 * timeSpeed;
                }
            });

            updateCameraPosition();
            renderer.render(scene, camera);
        }
    </script>
</body>
</html>
