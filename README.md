<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA 3D Cosmic Exploration - Solar System to Supercluster</title>
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
            <div class="w-32 h-32 rounded-full border-2 border-cyan-500/50 animate-ping absolute"></div>
            <div class="text-7xl">🌌</div>
        </div>
        <div class="space-y-2">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-rose-500 via-cyan-400 to-indigo-400 tracking-wider">
                UNIVERSAL SCALE 3D SIMULATOR
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 max-w-md mx-auto leading-relaxed">
                Khám phá từ Hệ Mặt Trời ➔ Dải Ngân Hà (Milky Way) ➔ Siêu Cụm Thiên Hà (Supercluster) vĩ đại.
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-rose-600 via-purple-600 to-cyan-600 font-orbitron font-bold text-xs sm:text-sm rounded-2xl text-white cinematic-glow transition active:scale-95 border border-rose-400 flex items-center gap-3">
            <span>🚀</span> BẮT ĐẦU DU HÀNH VŨ TRỤ VĨ MÔ
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
                <span class="text-[10px] font-bold tracking-widest text-slate-400">COSMIC COMMAND CENTER</span>
                <div class="flex items-center gap-2">
                    <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-2.5 py-1 bg-rose-950/80 border border-rose-400 text-rose-300 rounded-lg text-[11px] font-bold hover:bg-rose-900 transition flex items-center gap-1.5 shadow-[0_0_12px_rgba(244,63,94,0.4)]">
                        <span id="audioIcon">🔊</span> <span id="audioStatus">Straywav: ON</span>
                    </button>
                    <button onclick="toggleHudInfo()" id="toggleHudBtn" class="px-2 py-1 bg-slate-800 border border-slate-600 text-slate-200 rounded-lg text-[11px] font-bold hover:bg-slate-700 transition">
                        👁️ Ẩn
                    </button>
                </div>
            </div>

            <!-- NÚT CHUYỂN QUY MÔ VŨ TRỤ (COSMIC SCALE TOGGLE) -->
            <div class="bg-indigo-950/50 p-2.5 rounded-xl border border-indigo-500/40 space-y-2">
                <div class="text-[10px] text-indigo-300 font-bold uppercase tracking-wider">🌌 Tầm Nhìn Vũ Trụ (Zoom Out):</div>
                <div class="grid grid-cols-3 gap-1.5 text-[9px]">
                    <button onclick="setCosmicView('system')" id="viewSysBtn" class="py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-200 font-bold rounded-lg transition">
                        ☀️ Hệ Mặt Trời
                    </button>
                    <button onclick="setCosmicView('galaxy')" id="viewGalBtn" class="py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition">
                        🌀 Ngân Hà
                    </button>
                    <button onclick="setCosmicView('supercluster')" id="viewClusBtn" class="py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition">
                        🌌 Siêu Cụm
                    </button>
                </div>
            </div>

            <!-- ĐIỀU KHIỂN TÀU VŨ TRỤ -->
            <div class="bg-rose-950/40 p-2 rounded-xl border border-rose-500/40 flex justify-between items-center text-[10px]">
                <span class="text-rose-300 font-bold">🛸 Tàu Vũ Trụ Free-Roam:</span>
                <button onclick="toggleSpaceshipMode()" id="spaceshipBtn" class="px-2.5 py-1 bg-rose-900 border border-rose-400 text-rose-100 font-bold rounded-md hover:bg-rose-800 transition">
                    Bật Lái Tàu (WASD)
                </button>
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
                        Bấm chọn các chế độ quan sát phía trên để lùi ra ngoài không gian Ngân Hà & Siêu cụm thiên hà!
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
        let planets = {};
        let milkyWayPoints, superclusterGroup;
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, previousMousePosition = { x: 0, y: 0 };
        let cameraDistance = 65, targetCameraDistance = 65;
        let cameraAngleX = 0.4, cameraAngleY = 0.0;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0, isSpaceshipMode = false;
        let spaceshipPos = new THREE.Vector3(0, 15, 80);
        let keysPressed = {};
        let currentViewLevel = 'system';

        const planetDatabase = {
            sun: { name: "MẶT TRỜI (SUN)", desc: "Ngôi sao trung tâm Hệ Mặt Trời.", dist: "0 AU", temp: "5,500 °C", radius: "696,340 km", moons: "0", zoomDist: 65 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Hành tinh đá nhỏ nhất gần Mặt Trời.", dist: "0.39 AU", temp: "-180 đến 430 °C", radius: "2,439 km", moons: "0", zoomDist: 6 },
            venus: { name: "SAO KIM (VENUS)", desc: "Khí quyển CO2 cực kỳ nóng cháy.", dist: "0.72 AU", temp: "462 °C", radius: "6,051 km", moons: "0", zoomDist: 7 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Ngôi nhà duy nhất có sự sống.", dist: "1.00 AU", temp: "15 °C", radius: "6,371 km", moons: "1", zoomDist: 8 },
            mars: { name: "SAO HỎA (MARS)", desc: "Hành tinh Đỏ phủ oxit sắt.", dist: "1.52 AU", temp: "-63 °C", radius: "3,389 km", moons: "2", zoomDist: 7 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Gã khổng lồ khí lớn nhất.", dist: "5.20 AU", temp: "-110 °C", radius: "69,911 km", moons: "95", zoomDist: 14 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Vành đai băng hoành tráng nhất.", dist: "9.58 AU", temp: "-140 °C", radius: "58,232 km", moons: "146", zoomDist: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng xoay nghiêng.", dist: "19.22 AU", temp: "-195 °C", radius: "25,362 km", moons: "28", zoomDist: 11 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Hành tinh gió bão xa nhất.", dist: "30.05 AU", temp: "-200 °C", radius: "24,622 km", moons: "16", zoomDist: 11 }
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

        function setCosmicView(level) {
            currentViewLevel = level;
            isSpaceshipMode = false;
            
            document.getElementById('viewSysBtn').className = level === 'system' ? "py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-200 font-bold rounded-lg transition" : "py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition";
            document.getElementById('viewGalBtn').className = level === 'galaxy' ? "py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-200 font-bold rounded-lg transition" : "py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition";
            document.getElementById('viewClusBtn').className = level === 'supercluster' ? "py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-200 font-bold rounded-lg transition" : "py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition";

            if (level === 'system') {
                targetCameraDistance = 65;
                document.getElementById('planetTitle').innerText = "TOÀN CẢNH HỆ MẶT TRỜI";
                document.getElementById('planetDesc').innerText = "Hệ thống 8 hành tinh quay quanh Mặt Trời trung tâm.";
            } else if (level === 'galaxy') {
                targetCameraDistance = 800;
                document.getElementById('planetTitle').innerText = "DẢI NGÂN HÀ (MILKY WAY GALAXY)";
                document.getElementById('planetDesc').innerText = "Thiên hà xoắn ốc chứa hơn 100 tỷ ngôi sao, Hệ Mặt Trời chỉ là 1 điểm sáng nhỏ.";
            } else if (level === 'supercluster') {
                targetCameraDistance = 3500;
                document.getElementById('planetTitle').innerText = "SIÊU CỤM THIÊN HÀ (SUPERCLUSTER)";
                document.getElementById('planetDesc').innerText = "Mạng lưới vũ trụ gồm hàng ngàn Thiên Hà liên kết với nhau bằng hấp dẫn.";
            }
        }

        function toggleSpaceshipMode() {
            isSpaceshipMode = !isSpaceshipMode;
            const btn = document.getElementById("spaceshipBtn");
            if (isSpaceshipMode) {
                btn.className = "px-2.5 py-1 bg-cyan-600 border border-cyan-300 text-white font-bold rounded-md shadow-[0_0_10px_rgba(56,189,248,0.6)]";
                btn.innerText = "Đang Lái Tàu";
                spaceshipPos.copy(camera.position);
            } else {
                btn.className = "px-2.5 py-1 bg-rose-900 border border-rose-400 text-rose-100 font-bold rounded-md hover:bg-rose-800 transition";
                btn.innerText = "Bật Lái Tàu (WASD)";
            }
        }

        function toggleSpaceAudio() {
            const bgAudio = document.getElementById('bgMusic');
            if (!bgAudio) return;
            if (bgAudio.paused) { bgAudio.play(); document.getElementById('audioStatus').innerText = "Straywav: ON"; }
            else { bgAudio.pause(); document.getElementById('audioStatus').innerText = "Straywav: OFF"; }
        }

        function createMilkyWayGalaxy() {
            const particleCount = 25000;
            const geometry = new THREE.BufferGeometry();
            const positions = new Float32Array(particleCount * 3);
            const colors = new Float32Array(particleCount * 3);

            const arms = 4;
            const radius = 350;

            for (let i = 0; i < particleCount; i++) {
                const r = Math.pow(Math.random(), 2) * radius;
                const armAngle = ((i % arms) * 2 * Math.PI) / arms;
                const twist = r * 0.015;
                const angle = armAngle + twist;

                const spreadX = (Math.random() - 0.5) * (r * 0.15);
                const spreadY = (Math.random() - 0.5) * (r * 0.08);
                const spreadZ = (Math.random() - 0.5) * (r * 0.15);

                positions[i * 3] = Math.cos(angle) * r + spreadX;
                positions[i * 3 + 1] = spreadY;
                positions[i * 3 + 2] = Math.sin(angle) * r + spreadZ;

                const mixRatio = r / radius;
                colors[i * 3] = 1.0 - mixRatio * 0.6;
                colors[i * 3 + 1] = 0.7 + mixRatio * 0.3;
                colors[i * 3 + 2] = 0.3 + mixRatio * 0.7;
            }

            geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
            geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

            milkyWayPoints = new THREE.Points(geometry, new THREE.PointsMaterial({
                size: 1.2,
                vertexColors: true,
                transparent: true,
                opacity: 0.85,
                blending: THREE.AdditiveBlending
            }));
            scene.add(milkyWayPoints);
        }

        function createSupercluster() {
            superclusterGroup = new THREE.Group();
            const galaxyCount = 350;

            for (let i = 0; i < galaxyCount; i++) {
                const miniGeo = new THREE.BufferGeometry();
                const miniCnt = 120;
                const miniPos = new Float32Array(miniCnt * 3);

                for (let j = 0; j < miniCnt * 3; j++) miniPos[j] = (Math.random() - 0.5) * 15;
                miniGeo.setAttribute('position', new THREE.BufferAttribute(miniPos, 3));

                const colors = [0x38bdf8, 0xa855f7, 0xf43f5e, 0xfbbf24, 0x34d399];
                const miniMat = new THREE.PointsMaterial({ color: colors[i % colors.length], size: 1.5, transparent: true, opacity: 0.7 });

                const miniGalaxy = new THREE.Points(miniGeo, miniMat);
                miniGalaxy.position.set(
                    (Math.random() - 0.5) * 3000,
                    (Math.random() - 0.5) * 1500,
                    (Math.random() - 0.5) * 3000
                );
                superclusterGroup.add(miniGalaxy);
            }
            scene.add(superclusterGroup);
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
            } else {
                ctx.fillStyle = '#64748b'; ctx.fillRect(0, 0, 1024, 512);
            }
            return new THREE.CanvasTexture(canvas);
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

            createMilkyWayGalaxy();
            createSupercluster();

            const sunMat = new THREE.MeshBasicMaterial({ map: createPhotorealisticTexture('sun') });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.8, 48, 48), sunMat);
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(6.4, 32, 32), new THREE.MeshBasicMaterial({ color: 0xf59e0b, transparent: true, opacity: 0.35, side: THREE.BackSide })));
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0 };

            const planetData = [
                { id: 'mercury', size: 0.55, distance: 9, speed: 0.022 },
                { id: 'venus', size: 0.95, distance: 13, speed: 0.016 },
                { id: 'earth', size: 1.15, distance: 18, speed: 0.012, texture: 'earth' },
                { id: 'mars', size: 0.75, distance: 23, speed: 0.009 },
                { id: 'jupiter', size: 2.7, distance: 34, speed: 0.006 },
                { id: 'saturn', size: 2.2, distance: 44, speed: 0.004, hasRings: true },
                { id: 'uranus', size: 1.5, distance: 52, speed: 0.003 },
                { id: 'neptune', size: 1.4, distance: 60, speed: 0.002 }
            ];

            planetData.forEach(data => {
                const orbitGroup = new THREE.Group();
                scene.add(orbitGroup);

                const orbitLine = new THREE.Mesh(new THREE.RingGeometry(data.distance - 0.06, data.distance + 0.06, 128), new THREE.MeshBasicMaterial({ color: 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.15 }));
                orbitLine.rotation.x = Math.PI / 2;
                scene.add(orbitLine);

                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size, 48, 48), new THREE.MeshStandardMaterial({ roughness: 0.7, map: data.texture ? createPhotorealisticTexture(data.texture) : null }));
                planetMesh.position.x = data.distance;
                orbitGroup.add(planetMesh);

                if (data.hasRings) {
                    const ring = new THREE.Mesh(new THREE.RingGeometry(2.9, 5.2, 64), new THREE.MeshStandardMaterial({ color: 0xfef08a, side: THREE.DoubleSide, transparent: true, opacity: 0.85 }));
                    ring.rotation.x = Math.PI / 2.2;
                    planetMesh.add(ring);
                }

                planets[data.id] = { mesh: planetMesh, distance: data.distance, speed: data.speed, angle: Math.random() * Math.PI * 2 };
            });

            window.addEventListener('keydown', (e) => { keysPressed[e.key.toLowerCase()] = true; });
            window.addEventListener('keyup', (e) => { keysPressed[e.key.toLowerCase()] = false; });

            window.addEventListener('mousedown', (e) => { isDragging = true; previousMousePosition = { x: e.clientX, y: e.clientY }; });
            window.addEventListener('mousemove', (e) => {
                if (!isDragging) return;
                const deltaX = e.clientX - previousMousePosition.x;
                const deltaY = e.clientY - previousMousePosition.y;

                if (isSpaceshipMode) {
                    camera.rotation.y -= deltaX * 0.003;
                    camera.rotation.x -= deltaY * 0.003;
                } else {
                    cameraAngleY -= deltaX * 0.005;
                    cameraAngleX = Math.max(-1.4, Math.min(1.4, cameraAngleX - deltaY * 0.005));
                }
                previousMousePosition = { x: e.clientX, y: e.clientY };
            });

            window.addEventListener('mouseup', () => isDragging = false);
            window.addEventListener('wheel', (e) => {
                if (!isSpaceshipMode) targetCameraDistance = Math.max(3, Math.min(6000, targetCameraDistance + e.deltaY * 0.8));
            });

            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function updateCameraPosition() {
            if (isSpaceshipMode) {
                const speed = 1.2;
                let dir = new THREE.Vector3();
                camera.getWorldDirection(dir);
                
                if (keysPressed['w']) spaceshipPos.addScaledVector(dir, speed);
                if (keysPressed['s']) spaceshipPos.addScaledVector(dir, -speed);
                if (keysPressed['a']) spaceshipPos.addScaledVector(new THREE.Vector3(-dir.z, 0, dir.x).normalize(), speed);
                if (keysPressed['d']) spaceshipPos.addScaledVector(new THREE.Vector3(-dir.z, 0, dir.x).normalize(), -speed);
                if (keysPressed['r']) spaceshipPos.y += speed;
                if (keysPressed['f']) spaceshipPos.y -= speed;

                camera.position.copy(spaceshipPos);
                return;
            }

            let targetPos = new THREE.Vector3(0, 0, 0);
            if (currentViewLevel === 'system' && focusedObject && planets[focusedObject]) {
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
            setCosmicView('system');
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

            if (milkyWayPoints) milkyWayPoints.rotation.y += 0.0005 * timeSpeed;
            if (superclusterGroup) superclusterGroup.rotation.y += 0.0001 * timeSpeed;

            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
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
