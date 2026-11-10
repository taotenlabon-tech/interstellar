<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA 3D Interactive Physics & Gravity Sandbox</title>
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
            <div class="text-7xl">💥</div>
        </div>
        <div class="space-y-2">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-rose-500 via-purple-400 to-cyan-400 tracking-wider">
                INTERACTIVE PHYSICS SANDBOX
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 max-w-md mx-auto leading-relaxed">
                Trải nghiệm tính năng thả Hố đen hút vật thể, Phóng vệ tinh nhân tạo, Kích hoạt Nhật Thực & Du hành thời gian.
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-rose-600 via-purple-600 to-cyan-600 font-orbitron font-bold text-xs sm:text-sm rounded-2xl text-white cinematic-glow transition active:scale-95 border border-rose-400 flex items-center gap-3">
            <span>⚡</span> TRẢI NGHIỆM SANDBOX VẬT LÝ VŨ TRỤ
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
                <span class="text-[10px] font-bold tracking-widest text-slate-400">PHYSICS ENGINE COMMAND</span>
                <div class="flex items-center gap-2">
                    <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-2.5 py-1 bg-rose-950/80 border border-rose-400 text-rose-300 rounded-lg text-[11px] font-bold hover:bg-rose-900 transition flex items-center gap-1.5 shadow-[0_0_12px_rgba(244,63,94,0.4)]">
                        <span id="audioIcon">🔊</span> <span id="audioStatus">Straywav: ON</span>
                    </button>
                    <button onclick="toggleHudInfo()" id="toggleHudBtn" class="px-2 py-1 bg-slate-800 border border-slate-600 text-slate-200 rounded-lg text-[11px] font-bold hover:bg-slate-700 transition">
                        👁️ Ẩn
                    </button>
                </div>
            </div>

            <!-- BẢNG ĐIỀU KHIỂN VẬT LÝ & TƯƠNG TÁC -->
            <div class="bg-rose-950/40 p-2.5 rounded-xl border border-rose-500/40 space-y-2">
                <div class="text-[10px] text-rose-300 font-bold uppercase tracking-wider">💥 Tương Tác Vật Lý Thực Chiến:</div>
                <div class="grid grid-cols-2 gap-2 text-[10px]">
                    <button onclick="spawnGravityBomb()" class="py-1.5 bg-rose-900 border border-rose-400 text-rose-100 font-bold rounded-lg hover:bg-rose-800 transition">
                        🕳️ Thả Hố Đen Hút
                    </button>
                    <button onclick="spawnSatellite()" class="py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-100 font-bold rounded-lg hover:bg-cyan-800 transition">
                        🛰️ Phóng Vệ Tinh
                    </button>
                </div>
                <div class="grid grid-cols-2 gap-2 text-[10px]">
                    <button onclick="triggerEclipse()" class="py-1.5 bg-purple-900 border border-purple-400 text-purple-100 font-bold rounded-lg hover:bg-purple-800 transition">
                        🌘 Căn Chỉnh Nhật Thực
                    </button>
                    <button onclick="spawnCometShower()" class="py-1.5 bg-amber-900 border border-amber-400 text-amber-100 font-bold rounded-lg hover:bg-amber-800 transition">
                        ☄️ Mưa Thiên Thạch
                    </button>
                </div>
            </div>

            <!-- CỖ MÁY THỜI GIAN -->
            <div class="bg-cyan-950/40 p-2.5 rounded-xl border border-cyan-500/40 space-y-2">
                <div class="text-[10px] text-cyan-300 font-bold uppercase tracking-wider">⏳ Tua Nhanh Thời Gian:</div>
                <div class="grid grid-cols-3 gap-1.5 text-[9px]">
                    <button onclick="travelTime(-1000)" class="py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-100 font-bold rounded-lg hover:bg-cyan-800 transition">
                        ⏪ -1k Năm
                    </button>
                    <button onclick="resetTime()" class="py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg hover:bg-slate-800 transition">
                        ⏹️ Hiện Tại
                    </button>
                    <button onclick="travelTime(1000)" class="py-1.5 bg-rose-950 border border-rose-500 text-rose-200 font-bold rounded-lg hover:bg-rose-900 transition">
                        ⏩ +1k Năm
                    </button>
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
                        Sử dụng bảng điều khiển vật lý phía trên để thử nghiệm các hiện tượng vũ trụ!
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
        let milkyWayPoints, andromedaGalaxy, superclusterGroup;
        let activeGravityBombs = [], activeSatellites = [], customMeteors = [];
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, previousMousePosition = { x: 0, y: 0 };
        let cameraDistance = 65, targetCameraDistance = 65;
        let cameraAngleX = 0.4, cameraAngleY = 0.0;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0;
        let timeOffset = 0;
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

        function travelTime(years) {
            timeOffset += years * 0.05;
            document.getElementById('planetTitle').innerText = `MỐC THỜI GIAN: ${years > 0 ? '+' : ''}${years} NĂM`;
            document.getElementById('planetDesc').innerText = `Đã tua nhanh quỹ đạo các hành tinh tới mốc ${years > 0 ? 'tương lai' : 'quá khứ'}.`;
        }

        function resetTime() {
            timeOffset = 0;
            document.getElementById('planetTitle').innerText = "MỐC THỜI GIAN: HIỆN TẠI";
            document.getElementById('planetDesc').innerText = "Đã đưa hệ thống trở về thời gian thực.";
        }

        // TÍNH NĂNG 1: THẢ HỐ ĐEN MINI HÚT VẬT THỂ
        function spawnGravityBomb() {
            const bombGroup = new THREE.Group();
            const core = new THREE.Mesh(new THREE.SphereGeometry(1.2, 16, 16), new THREE.MeshBasicMaterial({ color: 0x000000 }));
            const ring = new THREE.Mesh(new THREE.RingGeometry(1.5, 3, 32), new THREE.MeshBasicMaterial({ color: 0xf43f5e, side: THREE.DoubleSide }));
            ring.rotation.x = Math.PI / 2;
            bombGroup.add(core); bombGroup.add(ring);

            // Thả ngẫu nhiên gần khu vực các hành tinh trong
            const angle = Math.random() * Math.PI * 2;
            const dist = 15 + Math.random() * 15;
            bombGroup.position.set(Math.cos(angle) * dist, (Math.random() - 0.5) * 5, Math.sin(angle) * dist);
            scene.add(bombGroup);
            activeGravityBombs.push(bombGroup);

            document.getElementById('planetTitle').innerText = "VŨ KHÍ TRỌNG LỰC ĐÃ KÍCH HOẠT";
            document.getElementById('planetDesc').innerText = "Hố đen mini đang hút các vật thể và thiên thạch lân cận!";
        }

        // TÍNH NĂNG 2: PHÓNG VỆ TINH NHÂN TẠO QUANH TRÁI ĐẤT
        function spawnSatellite() {
            if (!planets['earth']) return;
            const sat = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.3, 0.5), new THREE.MeshStandardMaterial({ color: 0xf8fafc, metalness: 0.9 }));
            planets['earth'].mesh.add(sat);
            activeSatellites.push({ mesh: sat, angle: Math.random() * Math.PI * 2, dist: 2.2 });

            document.getElementById('planetTitle').innerText = "PHÓNG VỆ TINH THÀNH CÔNG";
            document.getElementById('planetDesc').innerText = "Vệ tinh nhân tạo đang quay vòng quanh quỹ đạo Trái Đất.";
        }

        // TÍNH NĂNG 3: CĂN CHỈNH NHẬT THỰC (ECLIPSE)
        function triggerEclipse() {
            if (!planets['earth'] || !planets['sun']) return;
            planets['earth'].angle = 0; // Đưa Trái Đất thẳng hàng
            document.getElementById('planetTitle').innerText = "HIỆN TƯỢNG NHẬT THỰC";
            document.getElementById('planetDesc').innerText = "Mặt Trăng và Trái Đất đã được căn chỉnh thẳng hàng chắn ánh sáng Mặt Trời.";
        }

        // TÍNH NĂNG 4: BẮN MƯA THIÊN THẠCH
        function spawnCometShower() {
            for(let i=0; i<5; i++) {
                const meteor = new THREE.Mesh(new THREE.DodecahedronGeometry(0.25, 1), new THREE.MeshBasicMaterial({ color: 0x38bdf8 }));
                meteor.position.set((Math.random() - 0.5) * 50, (Math.random() - 0.5) * 15, -70);
                scene.add(meteor);
                customMeteors.push({ mesh: meteor, velocity: new THREE.Vector3((Math.random() - 0.5) * 0.6, (Math.random() - 0.5) * 0.1, 1.0 + Math.random() * 0.5) });
            }
            document.getElementById('planetTitle').innerText = "MƯA THIÊN THẠCH XUYÊN KHÔNG";
            document.getElementById('planetDesc').innerText = "Các khối thiên thạch băng đang bay xẹt qua hệ thống.";
        }

        function toggleSpaceAudio() {
            const bgAudio = document.getElementById('bgMusic');
            if (!bgAudio) return;
            if (bgAudio.paused) { bgAudio.play(); document.getElementById('audioStatus').innerText = "Straywav: ON"; }
            else { bgAudio.pause(); document.getElementById('audioStatus').innerText = "Straywav: OFF"; }
        }

        function createPlanetTexture(type) {
            const canvas = document.createElement('canvas');
            canvas.width = 1024; canvas.height = 512;
            const ctx = canvas.getContext('2d');

            if (type === 'sun') {
                const grad = ctx.createRadialGradient(512, 256, 20, 512, 256, 500);
                grad.addColorStop(0, '#ffffff'); grad.addColorStop(0.3, '#fde047'); grad.addColorStop(0.7, '#f97316'); grad.addColorStop(1, '#7c2d12');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, 1024, 512);
            } else if (type === 'earth') {
                ctx.fillStyle = '#0284c7'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#15803d'; for(let i=0; i<45; i++) ctx.beginPath(), ctx.ellipse(Math.random()*1024, Math.random()*512, Math.random()*130+40, Math.random()*70+20, Math.random()*Math.PI, 0, Math.PI*2), ctx.fill();
                ctx.fillStyle = '#fef08a'; for(let i=0; i<250; i++) ctx.fillRect(Math.random()*1024, Math.random()*512, 2, 2);
            } else if (type === 'clouds') {
                ctx.clearRect(0, 0, 1024, 512);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.75)';
                for(let i=0; i<70; i++) ctx.beginPath(), ctx.ellipse(Math.random()*1024, Math.random()*512, Math.random()*110+30, Math.random()*30+10, 0, 0, Math.PI*2), ctx.fill();
            } else {
                ctx.fillStyle = '#64748b'; ctx.fillRect(0, 0, 1024, 512);
            }
            return new THREE.CanvasTexture(canvas);
        }

        function createMilkyWayGalaxy() {
            const count = 25000;
            const geo = new THREE.BufferGeometry();
            const pos = new Float32Array(count * 3);
            const col = new Float32Array(count * 3);
            for (let i = 0; i < count; i++) {
                const r = Math.pow(Math.random(), 2) * 350;
                const angle = (i % 4) * (Math.PI / 2) + (r * 0.015);
                pos[i * 3] = Math.cos(angle) * r + (Math.random() - 0.5) * 30;
                pos[i * 3 + 1] = (Math.random() - 0.5) * 20;
                pos[i * 3 + 2] = Math.sin(angle) * r + (Math.random() - 0.5) * 30;
                col[i * 3] = 0.8; col[i * 3 + 1] = 0.8; col[i * 3 + 2] = 1.0;
            }
            geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
            geo.setAttribute('color', new THREE.BufferAttribute(col, 3));
            milkyWayPoints = new THREE.Points(geo, new THREE.PointsMaterial({ size: 1.2, vertexColors: true, transparent: true, opacity: 0.85, blending: THREE.AdditiveBlending }));
            scene.add(milkyWayPoints);
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

            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.8, 48, 48), new THREE.MeshBasicMaterial({ map: createPlanetTexture('sun') }));
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(6.4, 32, 32), new THREE.MeshBasicMaterial({ color: 0xf59e0b, transparent: true, opacity: 0.35, side: THREE.BackSide })));
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0 };

            const planetData = [
                { id: 'mercury', size: 0.55, distance: 9, speed: 0.022 },
                { id: 'venus', size: 0.95, distance: 13, speed: 0.016 },
                { id: 'earth', size: 1.15, distance: 18, speed: 0.012, texture: 'earth', hasClouds: true, hasMoon: true },
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

                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size, 48, 48), new THREE.MeshStandardMaterial({ roughness: 0.7, map: data.texture ? createPlanetTexture(data.texture) : null }));
                planetMesh.position.x = data.distance;
                orbitGroup.add(planetMesh);

                if (data.hasMoon) {
                    const moon = new THREE.Mesh(new THREE.SphereGeometry(0.3, 16, 16), new THREE.MeshStandardMaterial({ color: 0xcbd5e1 }));
                    moon.position.x = 2.4;
                    planetMesh.add(moon);
                    planetMesh.userData.moon = moon;
                }

                if (data.hasRings) {
                    const ring = new THREE.Mesh(new THREE.RingGeometry(2.9, 5.2, 64), new THREE.MeshStandardMaterial({ color: 0xfef08a, side: THREE.DoubleSide, transparent: true, opacity: 0.85 }));
                    ring.rotation.x = Math.PI / 2.2;
                    planetMesh.add(ring);
                }

                planets[data.id] = { mesh: planetMesh, distance: data.distance, speed: data.speed, angle: Math.random() * Math.PI * 2 };
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
            window.addEventListener('wheel', (e) => { targetCameraDistance = Math.max(3, Math.min(1500, targetCameraDistance + e.deltaY * 0.8)); });

            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function updateCameraPosition() {
            let targetPos = new THREE.Vector3(0, 0, 0);
            if (focusedObject && planets[focusedObject]) {
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

            if (milkyWayPoints) milkyWayPoints.rotation.y += 0.0005 * timeSpeed;

            // Xoay vệ tinh nhân tạo quanh Trái Đất
            activeSatellites.forEach(s => {
                s.angle += 0.03 * timeSpeed;
                s.mesh.position.set(Math.cos(s.angle) * s.dist, 0, Math.sin(s.angle) * s.dist);
            });

            // Hố đen hút vật thể xung quanh
            activeGravityBombs.forEach(b => {
                b.rotation.z += 0.05;
                Object.keys(planets).forEach(id => {
                    const p = planets[id];
                    if (id !== 'sun' && p.mesh) {
                        let worldPos = new THREE.Vector3();
                        p.mesh.getWorldPosition(worldPos);
                        if (worldPos.distanceTo(b.position) < 15) {
                            p.mesh.position.lerp(b.position, 0.02); // Hút dần vào tâm hố đen
                        }
                    }
                });
            });

            // Cập nhật thiên thạch bắn
            for (let i = customMeteors.length - 1; i >= 0; i--) {
                const m = customMeteors[i];
                m.mesh.position.addScaledVector(m.velocity, timeSpeed);
                if (m.mesh.position.length() > 300) { scene.remove(m.mesh); customMeteors.splice(i, 1); }
            }

            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
                    p.angle += (p.speed * timeSpeed) + (timeOffset * 0.001);
                    p.mesh.position.x = Math.cos(p.angle) * p.distance;
                    p.mesh.position.z = Math.sin(p.angle) * p.distance;
                    p.mesh.rotation.y += 0.015 * timeSpeed;
                    if (p.mesh.userData.moon) p.mesh.userData.moon.rotation.y += 0.02 * timeSpeed;
                }
            });

            updateCameraPosition();
            renderer.render(scene, camera);
        }
    </script>
</body>
</html>
