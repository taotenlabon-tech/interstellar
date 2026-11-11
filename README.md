<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA 3D High-Tech Cosmic Sandbox</title>
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
            0% { box-shadow: 0 0 15px rgba(56, 189, 248, 0.4); }
            50% { box-shadow: 0 0 35px rgba(56, 189, 248, 0.9); }
            100% { box-shadow: 0 0 15px rgba(56, 189, 248, 0.4); }
        }
        .cinematic-glow { animation: pulseGlow 1.8s infinite; }
    </style>
</head>
<body class="w-screen h-screen flex flex-col justify-between select-none">

    <!-- OVERLAY KHỞI ĐỘNG -->
    <div id="startOverlay" class="fixed inset-0 z-50 bg-slate-950/95 backdrop-blur-xl flex flex-col items-center justify-center p-6 text-center space-y-6">
        <div class="relative flex items-center justify-center">
            <div class="w-32 h-32 rounded-full border-2 border-cyan-500/50 animate-ping absolute"></div>
            <div class="text-7xl">🚀</div>
        </div>
        <div class="space-y-2">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 via-indigo-400 to-rose-400 tracking-wider">
                HIGH-TECH COSMIC SANDBOX
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 max-w-md mx-auto leading-relaxed">
                Click chuột bất kỳ đâu để tạo Sao Chổi, phóng Trạm ISS & thả Tàu thám hiểm vũ trụ real-time!
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-cyan-600 via-indigo-600 to-rose-600 font-orbitron font-bold text-xs sm:text-sm rounded-2xl text-white cinematic-glow transition active:scale-95 border border-cyan-400 flex items-center gap-3">
            <span>⚡</span> KHỞI ĐỘNG TRẠM VŨ TRỤ SANDBOX
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
                <span class="text-[10px] font-bold tracking-widest text-slate-400">COSMIC INTERACTIVE HUD</span>
                <div class="flex items-center gap-2">
                    <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-2.5 py-1 bg-rose-950/80 border border-rose-400 text-rose-300 rounded-lg text-[11px] font-bold hover:bg-rose-900 transition flex items-center gap-1.5 shadow-[0_0_12px_rgba(244,63,94,0.4)]">
                        <span id="audioIcon">🔊</span> <span id="audioStatus">Straywav: ON</span>
                    </button>
                    <button onclick="toggleHudInfo()" id="toggleHudBtn" class="px-2 py-1 bg-slate-800 border border-slate-600 text-slate-200 rounded-lg text-[11px] font-bold hover:bg-slate-700 transition">
                        👁️ Ẩn
                    </button>
                </div>
            </div>

            <!-- CHỨC NĂNG TƯƠNG TÁC HAY -->
            <div class="bg-cyan-950/40 p-2.5 rounded-xl border border-cyan-500/40 space-y-2">
                <div class="text-[10px] text-cyan-300 font-bold uppercase tracking-wider">🛸 Tính Năng Tương Tác Cực Đỉnh:</div>
                <div class="grid grid-cols-2 gap-2 text-[10px]">
                    <button onclick="toggleExplorerShip()" id="shipBtn" class="py-2 bg-indigo-900 border border-indigo-400 text-indigo-100 font-bold rounded-lg hover:bg-indigo-800 transition">
                        🚀 Phóng Tàu Thám Hiểm
                    </button>
                    <button onclick="spawnISSSpaceStation()" class="py-2 bg-emerald-900 border border-emerald-400 text-emerald-100 font-bold rounded-lg hover:bg-emerald-800 transition">
                        🛰️ Phóng Trạm ISS
                    </button>
                </div>
                <div class="text-[9px] text-amber-300 italic text-center pt-1">
                    👉 *Mẹo: Click chuột vào bất kỳ đâu trên màn hình để giội Sao Chổi!*
                </div>
            </div>

            <div class="grid grid-cols-2 gap-2 text-[10px]">
                <div class="bg-slate-900/80 p-2 rounded-xl border border-slate-800 space-y-1">
                    <span class="text-rose-300 font-bold">🎧 Nhạc Vol:</span>
                    <input type="range" id="volumeSlider" min="0" max="1" step="0.05" value="0.75" onchange="changeVolume(this.value)" class="w-full accent-rose-500 h-1 cursor-pointer">
                </div>
                <div class="bg-slate-900/80 p-2 rounded-xl border border-slate-800 space-y-1">
                    <span class="text-cyan-300 font-bold">⏳ Tốc độ quay:</span>
                    <input type="range" id="timeSpeedSlider" min="0" max="5" step="0.1" value="1" onchange="changeTimeSpeed(this.value)" class="w-full accent-cyan-400 h-1 cursor-pointer">
                </div>
            </div>

            <div id="hudContent" class="space-y-3 transition-all duration-300 overflow-hidden">
                <div class="border-t border-slate-800 pt-2">
                    <div id="planetTitle" class="text-sm font-orbitron font-bold text-amber-300">TOÀN CẢNH HỆ MẶT TRỜI</div>
                    <p id="planetDesc" class="text-xs text-slate-300 leading-relaxed mt-1">
                        Hệ thống hành tinh sắc nét. Click chuột vào không gian để giội sao chổi lửa!
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
        let explorerShip, issStation;
        let customComets = [];
        let raycaster = new THREE.Raycaster();
        let mouse = new THREE.Vector2();
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, previousMousePosition = { x: 0, y: 0 };
        let cameraDistance = 65, targetCameraDistance = 65;
        let cameraAngleX = 0.4, cameraAngleY = 0.0;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0;
        let isShipActive = false;
        let shipAngle = 0;

        const planetDatabase = {
            sun: { name: "MẶT TRỜI (SUN)", desc: "Ngôi sao trung tâm tỏa sáng rực rỡ.", dist: "0 AU", temp: "5,500 °C", radius: "696,340 km", zoomDist: 65 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Hành tinh đá nhỏ nhất gần Mặt Trời.", dist: "0.39 AU", temp: "-180 đến 430 °C", radius: "2,439 km", zoomDist: 6 },
            venus: { name: "SAO KIM (VENUS)", desc: "Khí quyển CO2 cực kỳ nóng cháy.", dist: "0.72 AU", temp: "462 °C", radius: "6,051 km", zoomDist: 7 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Hành tinh sự sống xanh lam mượt mà.", dist: "1.00 AU", temp: "15 °C", radius: "6,371 km", zoomDist: 8 },
            mars: { name: "SAO HỎA (MARS)", desc: "Hành tinh Đỏ sa mạc lạnh giá.", dist: "1.52 AU", temp: "-63 °C", radius: "3,389 km", zoomDist: 7 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Gã khổng lồ khí lớn nhất.", dist: "5.20 AU", temp: "-110 °C", radius: "69,911 km", zoomDist: 14 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Vành đai băng sáng lấp lánh.", dist: "9.58 AU", temp: "-140 °C", radius: "58,232 km", zoomDist: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng xanh ngọc lam.", dist: "19.22 AU", temp: "-195 °C", radius: "25,362 km", zoomDist: 11 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Hành tinh gió bão xa nhất.", dist: "30.05 AU", temp: "-200 °C", radius: "24,622 km", zoomDist: 11 }
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

        // TÍNH NĂNG 1: THẢ TÀU THÁM HIỂM VŨ TRỤ
        function toggleExplorerShip() {
            isShipActive = !isShipActive;
            const btn = document.getElementById('shipBtn');
            if (isShipActive) {
                if (!explorerShip) {
                    const group = new THREE.Group();
                    const body = new THREE.Mesh(new THREE.ConeGeometry(0.4, 1.2, 8), new THREE.MeshStandardMaterial({ color: 0x38bdf8, metalness: 0.8 }));
                    body.rotation.x = Math.PI / 2;
                    const wings = new THREE.Mesh(new THREE.BoxGeometry(1.5, 0.1, 0.4), new THREE.MeshStandardMaterial({ color: 0xf43f5e }));
                    group.add(body); group.add(wings);
                    scene.add(group);
                    explorerShip = group;
                }
                explorerShip.visible = true;
                btn.className = "py-2 bg-rose-600 border border-rose-300 text-white font-bold rounded-lg shadow-[0_0_12px_rgba(244,63,94,0.6)]";
                btn.innerText = "🚀 Đang Bay Thám Hiểm";
                document.getElementById('planetTitle').innerText = "TÀU THÁM HIỂM ĐÃ KÍCH HOẠT";
                document.getElementById('planetDesc').innerText = "Tàu vũ trụ đang tự động tuần tra qua các quỹ đạo hành tinh.";
            } else {
                if (explorerShip) explorerShip.visible = false;
                btn.className = "py-2 bg-indigo-900 border border-indigo-400 text-indigo-100 font-bold rounded-lg hover:bg-indigo-800 transition";
                btn.innerText = "🚀 Phóng Tàu Thám Hiểm";
            }
        }

        // TÍNH NĂNG 2: PHÓNG TRẠM VŨ TRỤ ISS QUANH TRÁI ĐẤT
        function spawnISSSpaceStation() {
            if (!planets['earth']) return;
            if (!issStation) {
                const issGroup = new THREE.Group();
                const core = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.1, 0.8), new THREE.MeshStandardMaterial({ color: 0xf8fafc, metalness: 0.9 }));
                core.rotation.z = Math.PI / 2;
                const solarPanels = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.02, 0.3), new THREE.MeshStandardMaterial({ color: 0x0284c7 }));
                issGroup.add(core); issGroup.add(solarPanels);
                planets['earth'].mesh.add(issGroup);
                issStation = issGroup;
                issStation.position.set(2.2, 0, 0);
            }
            document.getElementById('planetTitle').innerText = "TRẠM VŨ TRỤ ISS";
            document.getElementById('planetDesc').innerText = "Trạm không gian ISS đã đi vào quỹ đạo Trái Đất.";
        }

        // TÍNH NĂNG 3: CLICK CHUỘT TẠO SAO CHỔI LỬA
        function onDocumentMouseDown(event) {
            // Nếu click vào menu HUD thì bỏ qua
            if (event.clientY < 280 && event.clientX < 400) return;
            
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
            raycaster.setFromCamera(mouse, camera);

            // Tạo sao chổi xẹt qua điểm click
            const cometGeo = new THREE.SphereGeometry(0.35, 16, 16);
            const cometMat = new THREE.MeshBasicMaterial({ color: 0xf97316 });
            const comet = new THREE.Mesh(cometGeo, cometMat);
            
            const spawnPos = new THREE.Vector3();
            raycaster.ray.at(50, spawnPos);
            comet.position.copy(spawnPos);

            scene.add(comet);
            customComets.push({
                mesh: comet,
                dir: new THREE.Vector3((Math.random() - 0.5) * 1.5, -0.8, (Math.random() - 0.5) * 1.5).normalize()
            });
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

            // Nền sao rực rỡ lấp lánh
            const starsGeo = new THREE.BufferGeometry();
            const starPos = new Float32Array(4000 * 3);
            for(let i=0; i<4000*3; i++) starPos[i] = (Math.random() - 0.5) * 1800;
            starsGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            scene.add(new THREE.Points(starsGeo, new THREE.PointsMaterial({ color: 0x38bdf8, size: 0.9 })));

            // MẶT TRỜI
            const sunMat = new THREE.MeshBasicMaterial({ map: createPhotorealisticTexture('sun') });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.8, 48, 48), sunMat);
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(6.4, 32, 32), new THREE.MeshBasicMaterial({ color: 0xf59e0b, transparent: true, opacity: 0.35, side: THREE.BackSide })));
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0 };

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
                const orbitGroup = new THREE.Group();
                scene.add(orbitGroup);

                const orbitLine = new THREE.Mesh(new THREE.RingGeometry(data.distance - 0.06, data.distance + 0.06, 128), new THREE.MeshBasicMaterial({ color: 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.15 }));
                orbitLine.rotation.x = Math.PI / 2;
                scene.add(orbitLine);

                const pMat = new THREE.MeshStandardMaterial({ roughness: 0.7, map: createPhotorealisticTexture(data.texture) });
                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size, 48, 48), pMat);
                planetMesh.position.x = data.distance;
                orbitGroup.add(planetMesh);

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

                planets[data.id] = { mesh: planetMesh, distance: data.distance, speed: data.speed, angle: Math.random() * Math.PI * 2 };
            });

            window.addEventListener('mousedown', (e) => {
                isDragging = true;
                previousMousePosition = { x: e.clientX, y: e.clientY };
                onDocumentMouseDown(e);
            });

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
            
            if (isShipActive && explorerShip) {
                explorerShip.getWorldPosition(targetPos);
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

            // Xoay Tàu thám hiểm
            if (isShipActive && explorerShip) {
                shipAngle += 0.008 * timeSpeed;
                explorerShip.position.x = Math.cos(shipAngle) * 38;
                explorerShip.position.z = Math.sin(shipAngle) * 38;
                explorerShip.position.y = Math.sin(shipAngle * 2) * 5;
                explorerShip.rotation.y = -shipAngle + Math.PI / 2;
            }

            // Xoay Trạm ISS quanh Trái Đất
            if (issStation) {
                issStation.rotation.y += 0.02 * timeSpeed;
            }

            // Xử lý các quả Sao Chổi được click ra
            for (let i = customComets.length - 1; i >= 0; i--) {
                const c = customComets[i];
                c.mesh.position.addScaledVector(c.dir, 1.2 * timeSpeed);
                if (c.mesh.position.length() > 300) {
                    scene.remove(c.mesh);
                    customComets.splice(i, 1);
                }
            }

            // Cập nhật quay quỹ đạo hành tinh
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
