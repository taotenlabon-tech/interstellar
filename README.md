<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA 3D Cosmic Sandbox - Time Travel & Exoplanets</title>
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
            <div class="text-7xl">⏳</div>
        </div>
        <div class="space-y-2">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 via-indigo-400 to-rose-400 tracking-wider">
                COSMIC TIME MACHINE & EXOPLANETS
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 max-w-md mx-auto leading-relaxed">
                Khám phá vị trí Hệ Mặt Trời 1,000 năm trước & 1,000 năm sau, du hành ngoại hành tinh và Thiên hà Andromeda.
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-cyan-600 via-indigo-600 to-rose-600 font-orbitron font-bold text-xs sm:text-sm rounded-2xl text-white cinematic-glow transition active:scale-95 border border-cyan-400 flex items-center gap-3">
            <span>⚡</span> KÍCH HOẠT CỖ MÁY THỜI GIAN VŨ TRỤ
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
                <span class="text-[10px] font-bold tracking-widest text-slate-400">CHRONOS COMMAND CENTER</span>
                <div class="flex items-center gap-2">
                    <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-2.5 py-1 bg-rose-950/80 border border-rose-400 text-rose-300 rounded-lg text-[11px] font-bold hover:bg-rose-900 transition flex items-center gap-1.5 shadow-[0_0_12px_rgba(244,63,94,0.4)]">
                        <span id="audioIcon">🔊</span> <span id="audioStatus">Straywav: ON</span>
                    </button>
                    <button onclick="toggleHudInfo()" id="toggleHudBtn" class="px-2 py-1 bg-slate-800 border border-slate-600 text-slate-200 rounded-lg text-[11px] font-bold hover:bg-slate-700 transition">
                        👁️ Ẩn
                    </button>
                </div>
            </div>

            <!-- CỖ MÁY THỜI GIAN (TIME TRAVEL 1000 NĂM) -->
            <div class="bg-cyan-950/40 p-2.5 rounded-xl border border-cyan-500/40 space-y-2">
                <div class="text-[10px] text-cyan-300 font-bold uppercase tracking-wider">⏳ Cỗ Máy Thời Gian (1,000 Năm):</div>
                <div class="grid grid-cols-3 gap-1.5 text-[9px]">
                    <button onclick="travelTime(-1000)" class="py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-100 font-bold rounded-lg hover:bg-cyan-800 transition">
                        ⏪ 1k Năm Trước
                    </button>
                    <button onclick="resetTime()" class="py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg hover:bg-slate-800 transition">
                        ⏹️ Hiện Tại
                    </button>
                    <button onclick="travelTime(1000)" class="py-1.5 bg-rose-950 border border-rose-500 text-rose-200 font-bold rounded-lg hover:bg-rose-900 transition">
                        ⏩ 1k Năm Sau
                    </button>
                </div>
            </div>

            <!-- TƯƠNG TÁC SANDBOX -->
            <div class="bg-purple-950/40 p-2.5 rounded-xl border border-purple-500/40 space-y-2">
                <div class="text-[10px] text-purple-300 font-bold uppercase tracking-wider">☄️ Tương Tác Vũ Trụ:</div>
                <div class="grid grid-cols-2 gap-2 text-[10px]">
                    <button onclick="spawnComet()" class="py-1.5 bg-purple-900 border border-purple-400 text-purple-100 font-bold rounded-lg hover:bg-purple-800 transition">
                        ☄️ Bắn Thiên Thạch
                    </button>
                    <button onclick="setCosmicView('blackhole')" id="viewBhBtn" class="py-1.5 bg-rose-950 border border-rose-500 text-rose-200 font-bold rounded-lg transition">
                        🕳️ Xem Hố Đen
                    </button>
                </div>
            </div>

            <!-- QUY MÔ & THIÊN HÀ -->
            <div class="bg-indigo-950/50 p-2.5 rounded-xl border border-indigo-500/40 space-y-2">
                <div class="text-[10px] text-indigo-300 font-bold uppercase tracking-wider">🌌 Quy Mô & Thiên Hà:</div>
                <div class="grid grid-cols-2 gap-1.5 text-[9px]">
                    <button onclick="setCosmicView('system')" id="viewSysBtn" class="py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-200 font-bold rounded-lg transition">
                        ☀️ Hệ Mặt Trời
                    </button>
                    <button onclick="setCosmicView('galaxy')" id="viewGalBtn" class="py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition">
                        🌀 Ngân Hà
                    </button>
                    <button onclick="setCosmicView('andromeda')" id="viewAndBtn" class="py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition">
                        ✨ Thiên Hà Andromeda
                    </button>
                    <button onclick="setCosmicView('supercluster')" id="viewClusBtn" class="py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition">
                        🌌 Siêu Cụm
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
                        Sử dụng Cỗ Máy Thời Gian để tua nhanh vị trí các hành tinh sau 1,000 năm!
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
        <button id="btn-kepler186f" onclick="selectPlanet('kepler186f')" class="planet-btn px-3 py-1.5 text-emerald-300 rounded-xl text-xs font-bold whitespace-nowrap">🌿 Kepler-186f</button>
        <button id="btn-trappist1e" onclick="selectPlanet('trappist1e')" class="planet-btn px-3 py-1.5 text-teal-300 rounded-xl text-xs font-bold whitespace-nowrap">💎 TRAPPIST-1e</button>
    </div>

    <!-- CANVAS 3D -->
    <div id="canvasContainer" class="absolute inset-0 z-0"></div>

    <script>
        let scene, camera, renderer;
        let planets = {};
        let milkyWayPoints, andromedaGalaxy, superclusterGroup;
        let blackHoleMesh;
        let customMeteors = [];
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
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Hành tinh gió bão xa nhất.", dist: "30.05 AU", temp: "-200 °C", radius: "24,622 km", moons: "16", zoomDist: 11 },
            kepler186f: { name: "KEPLER-186F (NGOẠI HÀNH TINH)", desc: "Ngoại hành tinh nằm ở hệ sao khác (cách 582 năm ánh sáng).", dist: "69 AU (Giả lập)", temp: "-50 °C", radius: "7,000 km", moons: "0", zoomDist: 8 },
            trappist1e: { name: "TRAPPIST-1E (NGOẠI HÀNH TINH)", desc: "Ngoại hành tinh đá tiềm năng sống ở hệ sao TRAPPIST-1.", dist: "77 AU (Giả lập)", temp: "-20 °C", radius: "5,800 km", moons: "0", zoomDist: 7 },
            blackhole: { name: "HỐ ĐEN SIÊU TRỌNG (BLACK HOLE)", desc: "Nơi trọng lực lớn đến mức không gì thoát ra được.", dist: "Rìa Ngân Hà", temp: "0 °C", radius: "Vô cực", moons: "0", zoomDist: 25 },
            andromeda: { name: "THIÊN HÀ TIÊN NỮ (ANDROMEDA)", desc: "Thiên hà xoắn ốc lớn gần Dải Ngân Hà nhất.", dist: "4,500 Đơn vị", temp: "Vũ trụ", radius: "110,000 NĂM ÁNH SÁNG", moons: "Nhiều thiên hà lùn", zoomDist: 1200 }
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
            timeOffset += years * 0.05; // Mỗi 1000 năm tương ứng với bước dịch chuyển góc quỹ đạo
            document.getElementById('planetTitle').innerText = `MỐC THỜI GIAN: ${years > 0 ? '+' : ''}${years} NĂM`;
            document.getElementById('planetDesc').innerText = `Đã dịch chuyển quỹ đạo các hành tinh tới mốc thời gian ${years > 0.5 ? 'tương lai' : 'quá khứ'} 1,000 năm.`;
        }

        function resetTime() {
            timeOffset = 0;
            document.getElementById('planetTitle').innerText = "MỐC THỜI GIAN: HIỆN TẠI (2026)";
            document.getElementById('planetDesc').innerText = "Đã đưa hệ thống trở về thời gian thực.";
        }

        function setCosmicView(level) {
            currentViewLevel = level;
            document.getElementById('viewSysBtn').className = level === 'system' ? "py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-200 font-bold rounded-lg transition" : "py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition";
            document.getElementById('viewGalBtn').className = level === 'galaxy' ? "py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-200 font-bold rounded-lg transition" : "py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition";
            document.getElementById('viewAndBtn').className = level === 'andromeda' ? "py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-200 font-bold rounded-lg transition" : "py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition";
            document.getElementById('viewClusBtn').className = level === 'supercluster' ? "py-1.5 bg-cyan-900 border border-cyan-400 text-cyan-200 font-bold rounded-lg transition" : "py-1.5 bg-slate-900 border border-slate-700 text-slate-300 font-bold rounded-lg transition";
            document.getElementById('viewBhBtn').className = level === 'blackhole' ? "py-1.5 bg-rose-600 border border-rose-300 text-white font-bold rounded-lg shadow-[0_0_10px_rgba(244,63,94,0.6)]" : "py-1.5 bg-rose-950 border border-rose-500 text-rose-200 font-bold rounded-lg transition";

            if (level === 'system') targetCameraDistance = 65;
            else if (level === 'galaxy') targetCameraDistance = 800;
            else if (level === 'andromeda') { targetCameraDistance = 600; focusedObject = 'andromeda'; }
            else if (level === 'supercluster') targetCameraDistance = 3500;
            else if (level === 'blackhole') { targetCameraDistance = 35; focusedObject = 'blackhole'; }

            const info = planetDatabase[level === 'blackhole' ? 'blackhole' : level === 'andromeda' ? 'andromeda' : 'sun'];
            if (info) {
                document.getElementById('planetTitle').innerText = info.name;
                document.getElementById('planetDesc').innerText = info.desc;
            }
        }

        function spawnComet() {
            const meteor = new THREE.Mesh(new THREE.DodecahedronGeometry(0.3, 1), new THREE.MeshBasicMaterial({ color: 0xf43f5e }));
            meteor.position.set((Math.random() - 0.5) * 60, (Math.random() - 0.5) * 20, -80);
            scene.add(meteor);
            customMeteors.push({ mesh: meteor, velocity: new THREE.Vector3((Math.random() - 0.5) * 0.8, (Math.random() - 0.5) * 0.2, 0.8 + Math.random() * 0.5) });
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
            } else if (type === 'kepler186f') {
                ctx.fillStyle = '#065f46'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#047857'; for(let i=0; i<50; i++) ctx.beginPath(), ctx.arc(Math.random()*1024, Math.random()*512, Math.random()*80+20, 0, Math.PI*2), ctx.fill();
            } else if (type === 'trappist1e') {
                ctx.fillStyle = '#0f766e'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#14b8a6'; for(let i=0; i<60; i++) ctx.fillRect(0, Math.random()*512, 1024, Math.random()*15+5);
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
            const particleCount = 25000;
            const geometry = new THREE.BufferGeometry();
            const positions = new Float32Array(particleCount * 3);
            const colors = new Float32Array(particleCount * 3);
            const arms = 4, radius = 350;

            for (let i = 0; i < particleCount; i++) {
                const r = Math.pow(Math.random(), 2) * radius;
                const armAngle = ((i % arms) * 2 * Math.PI) / arms;
                const angle = armAngle + (r * 0.015);
                positions[i * 3] = Math.cos(angle) * r + (Math.random() - 0.5) * (r * 0.15);
                positions[i * 3 + 1] = (Math.random() - 0.5) * (r * 0.08);
                positions[i * 3 + 2] = Math.sin(angle) * r + (Math.random() - 0.5) * (r * 0.15);
                const mix = r / radius;
                colors[i * 3] = 1.0 - mix * 0.6; colors[i * 3 + 1] = 0.7 + mix * 0.3; colors[i * 3 + 2] = 0.3 + mix * 0.7;
            }
            geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
            geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));
            milkyWayPoints = new THREE.Points(geometry, new THREE.PointsMaterial({ size: 1.2, vertexColors: true, transparent: true, opacity: 0.85, blending: THREE.AdditiveBlending }));
            scene.add(milkyWayPoints);
        }

        function createAndromedaGalaxy() {
            const count = 18000;
            const geo = new THREE.BufferGeometry();
            const pos = new Float32Array(count * 3);
            const col = new Float32Array(count * 3);
            for (let i = 0; i < count; i++) {
                const r = Math.pow(Math.random(), 2) * 280;
                const angle = i * 0.05;
                pos[i * 3] = Math.cos(angle) * r + 800;
                pos[i * 3 + 1] = (Math.random() - 0.5) * (r * 0.1);
                pos[i * 3 + 2] = Math.sin(angle) * r - 600;
                col[i * 3] = 0.8; col[i * 3 + 1] = 0.8; col[i * 3 + 2] = 1.0;
            }
            geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
            geo.setAttribute('color', new THREE.BufferAttribute(col, 3));
            andromedaGalaxy = new THREE.Points(geo, new THREE.PointsMaterial({ size: 1.2, vertexColors: true, transparent: true, opacity: 0.8, blending: THREE.AdditiveBlending }));
            scene.add(andromedaGalaxy);
        }

        function createSupercluster() {
            superclusterGroup = new THREE.Group();
            for (let i = 0; i < 350; i++) {
                const miniGeo = new THREE.BufferGeometry();
                const miniPos = new Float32Array(120 * 3);
                for (let j = 0; j < 120 * 3; j++) miniPos[j] = (Math.random() - 0.5) * 15;
                miniGeo.setAttribute('position', new THREE.BufferAttribute(miniPos, 3));
                const colors = [0x38bdf8, 0xa855f7, 0xf43f5e, 0xfbbf24, 0x34d399];
                const miniGalaxy = new THREE.Points(miniGeo, new THREE.PointsMaterial({ color: colors[i % colors.length], size: 1.5, transparent: true, opacity: 0.7 }));
                miniGalaxy.position.set((Math.random() - 0.5) * 3000, (Math.random() - 0.5) * 1500, (Math.random() - 0.5) * 3000);
                superclusterGroup.add(miniGalaxy);
            }
            scene.add(superclusterGroup);
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
            createAndromedaGalaxy();
            createSupercluster();

            const bhGroup = new THREE.Group();
            bhGroup.position.set(250, 40, -150);
            bhGroup.add(new THREE.Mesh(new THREE.SphereGeometry(3, 32, 32), new THREE.MeshBasicMaterial({ color: 0x000000 })));
            const acc = new THREE.Mesh(new THREE.RingGeometry(3.5, 8, 64), new THREE.MeshBasicMaterial({ color: 0xf97316, side: THREE.DoubleSide, transparent: true, opacity: 0.9 }));
            acc.rotation.x = Math.PI / 2.2;
            bhGroup.add(acc);
            scene.add(bhGroup);
            blackHoleMesh = bhGroup;

            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.8, 48, 48), new THREE.MeshBasicMaterial({ map: createPlanetTexture('sun') }));
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(6.4, 32, 32), new THREE.MeshBasicMaterial({ color: 0xf59e0b, transparent: true, opacity: 0.35, side: THREE.BackSide })));
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0 };

            const planetData = [
                { id: 'mercury', size: 0.55, distance: 9, speed: 0.022, texture: 'mercury' },
                { id: 'venus', size: 0.95, distance: 13, speed: 0.016, texture: 'venus' },
                { id: 'earth', size: 1.15, distance: 18, speed: 0.012, texture: 'earth', hasClouds: true, hasAtmosphere: true },
                { id: 'mars', size: 0.75, distance: 23, speed: 0.009, texture: 'mars' },
                { id: 'jupiter', size: 2.7, distance: 34, speed: 0.006, texture: 'jupiter' },
                { id: 'saturn', size: 2.2, distance: 44, speed: 0.004, texture: 'saturn', hasRings: true },
                { id: 'uranus', size: 1.5, distance: 52, speed: 0.003, texture: 'uranus' },
                { id: 'neptune', size: 1.4, distance: 60, speed: 0.002, texture: 'neptune' },
                { id: 'kepler186f', size: 1.2, distance: 72, speed: 0.0015, texture: 'kepler186f', hasAtmosphere: true },
                { id: 'trappist1e', size: 1.0, distance: 82, speed: 0.001, texture: 'trappist1e', hasAtmosphere: true }
            ];

            planetData.forEach(data => {
                const orbitGroup = new THREE.Group();
                scene.add(orbitGroup);

                const orbitLine = new THREE.Mesh(new THREE.RingGeometry(data.distance - 0.06, data.distance + 0.06, 128), new THREE.MeshBasicMaterial({ color: 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.15 }));
                orbitLine.rotation.x = Math.PI / 2;
                scene.add(orbitLine);

                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size, 48, 48), new THREE.MeshStandardMaterial({ roughness: 0.7, map: createPlanetTexture(data.texture) }));
                planetMesh.position.x = data.distance;
                orbitGroup.add(planetMesh);

                if (data.hasAtmosphere) {
                    const atmos = new THREE.Mesh(new THREE.SphereGeometry(data.size * 1.15, 32, 32), new THREE.MeshBasicMaterial({ color: 0x34d399, transparent: true, opacity: 0.25, side: THREE.BackSide }));
                    planetMesh.add(atmos);
                }

                if (data.hasClouds) {
                    const cloudMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size * 1.02, 32, 32), new THREE.MeshStandardMaterial({ map: createPlanetTexture('clouds'), transparent: true, blending: THREE.AdditiveBlending, opacity: 0.65 }));
                    planetMesh.add(cloudMesh);
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
            window.addEventListener('wheel', (e) => { targetCameraDistance = Math.max(3, Math.min(6000, targetCameraDistance + e.deltaY * 0.8)); });

            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function updateCameraPosition() {
            let targetPos = new THREE.Vector3(0, 0, 0);
            if (currentViewLevel === 'blackhole' && blackHoleMesh) {
                blackHoleMesh.getWorldPosition(targetPos);
            } else if (currentViewLevel === 'andromeda' && andromedaGalaxy) {
                andromedaGalaxy.getWorldPosition(targetPos);
            } else if (currentViewLevel === 'system' && focusedObject && planets[focusedObject]) {
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
            if (andromedaGalaxy) andromedaGalaxy.rotation.y += 0.0003 * timeSpeed;
            if (superclusterGroup) superclusterGroup.rotation.y += 0.0001 * timeSpeed;
            if (blackHoleMesh) blackHoleMesh.rotation.z += 0.02 * timeSpeed;

            for (let i = customMeteors.length - 1; i >= 0; i--) {
                const m = customMeteors[i];
                m.mesh.position.addScaledVector(m.velocity, timeSpeed);
                if (m.mesh.position.length() > 500) { scene.remove(m.mesh); customMeteors.splice(i, 1); }
            }

            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
                    p.angle += (p.speed * timeSpeed) + (timeOffset * 0.001);
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
