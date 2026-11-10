
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA 3D Solar System - Ultimate Straywav Phonk Edition</title>
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
            <div class="text-7xl">🌌</div>
        </div>
        <div class="space-y-2">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-rose-500 via-cyan-400 to-amber-300 tracking-wider">
                NASA EYES // ULTIMATE PHONK 3D
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 max-w-md mx-auto leading-relaxed">
                Tích hợp Đai tiểu hành tinh, Đèn đêm Trái Đất, Bão mặt trời, Trạm ISS, Chế độ Cinematic & Nhạc Straywav.
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-rose-600 via-purple-600 to-cyan-600 font-orbitron font-bold text-xs sm:text-sm rounded-2xl text-white cinematic-glow transition active:scale-95 border border-rose-400 flex items-center gap-3">
            <span>⚡</span> KHỞI ĐỘNG HỆ THỐNG ULTIMATE 3D
        </button>
    </div>

    <!-- AUDIO PLAYER -->
    <audio id="bgMusic" loop preload="auto">
        <source src="https://cdn.pixabay.com/download/audio/2022/11/22/audio_febc508520.mp3?filename=drift-phonk-126548.mp3" type="audio/mpeg">
    </audio>

    <!-- HUD TRUNG TÂM ĐIỀU HÀNH -->
    <header id="mainHud" class="absolute top-4 left-4 z-20 transition-all duration-300 pointer-events-auto">
        <div class="hud-panel p-4 rounded-2xl max-w-sm space-y-3 relative">
            <div class="flex justify-between items-center gap-3">
                <span class="text-[10px] font-bold tracking-widest text-slate-400">NASA ULTIMATE SYSTEM</span>
                <div class="flex items-center gap-2">
                    <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-2.5 py-1 bg-rose-950/80 border border-rose-400 text-rose-300 rounded-lg text-[11px] font-bold hover:bg-rose-900 transition flex items-center gap-1.5 shadow-[0_0_12px_rgba(244,63,94,0.4)]">
                        <span id="audioIcon">🔊</span> <span id="audioStatus">Straywav: ON</span>
                    </button>
                    <button onclick="toggleHudInfo()" id="toggleHudBtn" class="px-2 py-1 bg-slate-800 border border-slate-600 text-slate-200 rounded-lg text-[11px] font-bold hover:bg-slate-700 transition">
                        👁️ Ẩn
                    </button>
                </div>
            </div>

            <!-- THANH ĐIỀU CHỈNH AUDIO & TỐC ĐỘ -->
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

            <!-- CÁC NÚT TÍNH NĂNG ĐẶC BIỆT -->
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
                        Hiển thị 8 Hành Tinh, Đai Tiểu Hành Tinh, Trạm ISS, Bão Mặt Trời & hiệu ứng mây 3D.
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
        let timeSpeed = 1.0, isRealScale = false, isCinematic = false;
        let cinematicTimer = 0, cinematicIndex = 0;
        let solarFlares = [];

        const planetDatabase = {
            sun: { name: "MẶT TRỜI (SUN)", desc: "Ngôi sao trung tâm, cấu tạo từ plasma hydro cực nóng với bão lửa bùng nổ liên tục.", dist: "0 AU", temp: "5,500 °C", radius: "696,340 km", moons: "0", zoomDist: 65 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Hành tinh đá với bề mặt chi chít hố va chạm thiên thạch khắc sâu tương tự Mặt Trăng.", dist: "0.39 AU", temp: "-180 đến 430 °C", radius: "2,439 km", moons: "0", zoomDist: 6 },
            venus: { name: "SAO KIM (VENUS)", desc: "Bao phủ bởi tầng mây dày axit sulfuric phản chiếu ánh sáng màu vàng cam rực rỡ.", dist: "0.72 AU", temp: "462 °C", radius: "6,051 km", moons: "0", zoomDist: 7 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Hành tinh duy nhất có sự sống, trạm ISS quay quanh & lớp đèn đêm thành phố sáng rực.", dist: "1.00 AU", temp: "15 °C", radius: "6,371 km", moons: "1 + Trạm ISS", zoomDist: 8 },
            mars: { name: "SAO HỎA (MARS)", desc: "Hành tinh Đỏ sa mạc lạnh giá với núi lửa khổng lồ và bụi oxit sắt đặc trưng.", dist: "1.52 AU", temp: "-63 °C", radius: "3,389 km", moons: "2 (Phobos, Deimos)", zoomDist: 7 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Gã khổng lồ khí với các dải mây bão xoáy cuộn và Vết Đỏ Lớn kinh điển.", dist: "5.20 AU", temp: "-110 °C", radius: "69,911 km", moons: "95", zoomDist: 14 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Nổi bật với hệ thống vành đai băng phức tạp sáng lấp lánh phản quang ấn tượng.", dist: "9.58 AU", temp: "-140 °C", radius: "58,232 km", moons: "146", zoomDist: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng khổng lồ màu xanh ngọc lam tĩnh lặng với trục quay nghiêng độc đáo.", dist: "19.22 AU", temp: "-195 °C", radius: "25,362 km", moons: "28", zoomDist: 11 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Hành tinh xanh thẫm xa xôi với những dải bão ngầm và gió siêu thanh mạnh nhất.", dist: "30.05 AU", temp: "-200 °C", radius: "24,622 km", moons: "16", zoomDist: 11 }
        };

        function toggleHudInfo() {
            isHudVisible = !isHudVisible;
            const content = document.getElementById("hudContent");
            const btn = document.getElementById("toggleHudBtn");
            if (isHudVisible) {
                content.style.maxHeight = "300px"; content.style.opacity = "1"; content.style.marginTop = "0.75rem"; btn.innerText = "👁️ Ẩn";
            } else {
                content.style.maxHeight = "0px"; content.style.opacity = "0"; content.style.marginTop = "0px"; btn.innerText = "👁️ Hiện Thông Tin";
            }
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
            const btn = document.getElementById("scaleModeBtn");
            btn.innerText = isRealScale ? "🌌 Scale: Real" : "🌌 Scale: Default";
            updatePlanetScales();
        }

        function toggleCinematicMode() {
            isCinematic = !isCinematic;
            const btn = document.getElementById("cinematicBtn");
            if (isCinematic) {
                btn.className = "w-1/2 py-1.5 bg-rose-950 border border-rose-500 text-rose-200 text-[10px] font-bold rounded-xl animate-pulse";
                btn.innerText = "🎬 Stop Tour";
            } else {
                btn.className = "w-1/2 py-1.5 bg-purple-950 border border-purple-500 text-purple-200 text-[10px] font-bold rounded-xl hover:bg-purple-900 transition";
                btn.innerText = "🎬 Cinematic Tour";
            }
        }

        function toggleSpaceAudio() {
            const bgAudio = document.getElementById('bgMusic');
            const btnStatus = document.getElementById('audioStatus');
            const btnIcon = document.getElementById('audioIcon');
            if (!bgAudio) return;
            if (bgAudio.paused) {
                bgAudio.play(); btnStatus.innerText = "Straywav: ON"; btnIcon.innerText = "🔊";
            } else {
                bgAudio.pause(); btnStatus.innerText = "Straywav: OFF"; btnIcon.innerText = "🔇";
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
                ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
                for(let i=0; i<80; i++) {
                    ctx.beginPath(); ctx.arc(Math.random()*1024, Math.random()*512, Math.random()*30+10, 0, Math.PI*2); ctx.fill();
                }
            } else if (type === 'mercury') {
                ctx.fillStyle = '#475569'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#1e293b';
                for(let i=0; i<300; i++) { ctx.beginPath(); ctx.arc(Math.random()*1024, Math.random()*512, Math.random()*12+2, 0, Math.PI*2); ctx.fill(); }
            } else if (type === 'venus') {
                ctx.fillStyle = '#b45309'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#fbbf24';
                for(let i=0; i<80; i++) ctx.fillRect(0, Math.random()*512, 1024, Math.random()*25+5);
            } else if (type === 'earth') {
                ctx.fillStyle = '#0284c7'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#15803d';
                for(let i=0; i<45; i++) {
                    ctx.beginPath(); ctx.ellipse(Math.random()*1024, Math.random()*512, Math.random()*130+40, Math.random()*70+20, Math.random()*Math.PI, 0, Math.PI*2); ctx.fill();
                }
                // Đèn đêm thành phố (City Night Lights)
                ctx.fillStyle = '#fef08a';
                for(let i=0; i<250; i++) { ctx.fillRect(Math.random()*1024, Math.random()*512, 2, 2); }
            } else if (type === 'clouds') {
                ctx.clearRect(0, 0, 1024, 512);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.75)';
                for(let i=0; i<70; i++) {
                    ctx.beginPath(); ctx.ellipse(Math.random()*1024, Math.random()*512, Math.random()*110+30, Math.random()*30+10, 0, 0, Math.PI*2); ctx.fill();
                }
            } else if (type === 'mars') {
                ctx.fillStyle = '#991b1b'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#7f1d1d';
                for(let i=0; i<180; i++) { ctx.beginPath(); ctx.arc(Math.random()*1024, Math.random()*512, Math.random()*20+5, 0, Math.PI*2); ctx.fill(); }
                ctx.fillStyle = '#f8fafc'; ctx.fillRect(0, 0, 1024, 35); ctx.fillRect(0, 477, 1024, 35);
            } else if (type === 'jupiter') {
                const colors = ['#78350f', '#b45309', '#f59e0b', '#fde047', '#d97706', '#92400e'];
                for(let y=0; y<512; y+=12) { ctx.fillStyle = colors[Math.floor(Math.random()*colors.length)]; ctx.fillRect(0, y, 1024, 12); }
                ctx.fillStyle = '#dc2626'; ctx.beginPath(); ctx.ellipse(650, 320, 75, 48, 0, 0, Math.PI*2); ctx.fill();
            } else if (type === 'saturn') {
                const colors = ['#ca8a04', '#eab308', '#fef08a', '#a16207', '#713f12'];
                for(let y=0; y<512; y+=10) { ctx.fillStyle = colors[Math.floor(Math.random()*colors.length)]; ctx.fillRect(0, y, 1024, 10); }
            } else if (type === 'uranus') {
                ctx.fillStyle = '#0891b2'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#22d3ee'; for(let y=0; y<512; y+=35) ctx.fillRect(0, y, 1024, 10);
            } else if (type === 'neptune') {
                ctx.fillStyle = '#312e81'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#4f46e5'; for(let y=0; y<512; y+=25) ctx.fillRect(0, y, 1024, 12);
                ctx.fillStyle = '#1e1b4b'; ctx.beginPath(); ctx.ellipse(300, 350, 65, 42, 0, 0, Math.PI*2); ctx.fill();
            }
            return new THREE.CanvasTexture(canvas);
        }

        function init3D() {
            const container = document.getElementById('canvasContainer');
            scene = new THREE.Scene();

            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 2500);
            
            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.toneMapping = THREE.ACESFilmicToneMapping;
            container.appendChild(renderer.domElement);

            const sunLight = new THREE.PointLight(0xffffff, 4.2, 1800);
            scene.add(sunLight);
            scene.add(new THREE.AmbientLight(0x1e293b, 0.75));

            // Ngàn sao
            const starsGeo = new THREE.BufferGeometry();
            const starsCnt = 5000;
            const starPos = new Float32Array(starsCnt * 3);
            for(let i=0; i<starsCnt*3; i++) starPos[i] = (Math.random() - 0.5) * 1400;
            starsGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            scene.add(new THREE.Points(starsGeo, new THREE.PointsMaterial({color: 0xffffff, size: 0.9})));

            // MẶT TRỜI CÓ BÃO PLASMA (SOLAR FLARES)
            const sunMat = new THREE.MeshBasicMaterial({ map: createPhotorealisticTexture('sun') });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.8, 48, 48), sunMat);
            const glowMat = new THREE.MeshBasicMaterial({ color: 0xf59e0b, transparent: true, opacity: 0.35, side: THREE.BackSide });
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(6.4, 32, 32), glowMat));
            
            // Thêm đốm bão lửa Plasma
            for(let i=0; i<6; i++) {
                const flare = new THREE.Mesh(new THREE.TorusGeometry(5.2 + Math.random()*0.4, 0.1, 8, 24), new THREE.MeshBasicMaterial({color: 0xef4444, transparent: true, opacity: 0.6}));
                flare.rotation.x = Math.random() * Math.PI;
                flare.rotation.y = Math.random() * Math.PI;
                sun.add(flare);
                solarFlares.push(flare);
            }
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0, baseSize: 4.8, realSize: 10.0 };

            // ĐAI TIỂU HÀNH TINH (ASTEROID BELT GIỮA MARS & JUPITER)
            const astGeo = new THREE.DodecahedronGeometry(0.12, 1);
            const astMat = new THREE.MeshStandardMaterial({ color: 0x64748b, roughness: 0.9 });
            for(let i=0; i<400; i++) {
                const ast = new THREE.Mesh(astGeo, astMat);
                const radius = 27 + Math.random() * 4;
                const angle = Math.random() * Math.PI * 2;
                ast.position.set(Math.cos(angle)*radius, (Math.random()-0.5)*1.5, Math.sin(angle)*radius);
                scene.add(ast);
                asteroidBelt.push({ mesh: ast, angle: angle, speed: 0.002 + Math.random()*0.003, radius: radius });
            }

            // 8 HÀNH TINH
            const planetData = [
                { id: 'mercury', baseSize: 0.55, realSize: 0.4, distance: 9, speed: 0.022, texture: 'mercury' },
                { id: 'venus', baseSize: 0.95, realSize: 0.95, distance: 13, speed: 0.016, texture: 'venus' },
                { id: 'earth', baseSize: 1.15, realSize: 1.0, distance: 18, speed: 0.012, texture: 'earth', hasClouds: true, hasMoon: true, hasISS: true },
                { id: 'mars', baseSize: 0.75, realSize: 0.53, distance: 23, speed: 0.009, texture: 'mars' },
                { id: 'jupiter', baseSize: 2.7, realSize: 5.5, distance: 34, speed: 0.006, texture: 'jupiter' },
                { id: 'saturn', baseSize: 2.2, realSize: 4.5, distance: 44, speed: 0.004, texture: 'saturn', hasRings: true },
                { id: 'uranus', baseSize: 1.5, realSize: 2.2, distance: 52, speed: 0.003, texture: 'uranus' },
                { id: 'neptune', baseSize: 1.4, realSize: 2.1, distance: 60, speed: 0.002, texture: 'neptune' }
            ];

            planetData.forEach(data => {
                const orbitGroup = new THREE.Group();
                scene.add(orbitGroup);

                const orbitGeo = new THREE.RingGeometry(data.distance - 0.06, data.distance + 0.06, 128);
                const orbitMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.12 });
                const orbitLine = new THREE.Mesh(orbitGeo, orbitMat);
                orbitLine.rotation.x = Math.PI / 2;
                scene.add(orbitLine);

                const pMat = new THREE.MeshStandardMaterial({
                    roughness: 0.7, metalness: 0.1,
                    map: createPhotorealisticTexture(data.texture)
                });
                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.baseSize, 48, 48), pMat);
                planetMesh.position.x = data.distance;
                orbitGroup.add(planetMesh);

                if (data.hasClouds) {
                    const cloudMat = new THREE.MeshStandardMaterial({
                        map: createPhotorealisticTexture('clouds'), transparent: true, blending: THREE.AdditiveBlending, opacity: 0.65
                    });
                    const cloudMesh = new THREE.Mesh(new THREE.SphereGeometry(data.baseSize * 1.02, 32, 32), cloudMat);
                    planetMesh.add(cloudMesh);
                    planetMesh.userData.cloudMesh = cloudMesh;
                }

                if (data.hasMoon) {
                    const moonMesh = new THREE.Mesh(new THREE.SphereGeometry(0.3, 24, 24), new THREE.MeshStandardMaterial({ color: 0xcbd5e1, roughness: 0.9 }));
                    moonMesh.position.x = 2.4;
                    planetMesh.add(moonMesh);
                    planetMesh.userData.moonMesh = moonMesh;
                }

                // Trạm Vũ Trụ ISS
                if (data.hasISS) {
                    const issMesh = new THREE.Mesh(new THREE.BoxGeometry(0.15, 0.05, 0.3), new THREE.MeshBasicMaterial({ color: 0x38bdf8 }));
                    issMesh.position.x = 1.6;
                    planetMesh.add(issMesh);
                    planetMesh.userData.issMesh = issMesh;
                }

                if (data.hasRings) {
                    const ringGeo = new THREE.RingGeometry(2.9, 5.2, 64);
                    const ringMat = new THREE.MeshStandardMaterial({ color: 0xfef08a, side: THREE.DoubleSide, transparent: true, opacity: 0.85, roughness: 0.5 });
                    const ring = new THREE.Mesh(ringGeo, ringMat);
                    ring.rotation.x = Math.PI / 2.2;
                    planetMesh.add(ring);
                    planetMesh.userData.ringMesh = ring;
                }

                planets[data.id] = {
                    mesh: planetMesh, distance: data.distance, speed: data.speed,
                    angle: Math.random() * Math.PI * 2, baseSize: data.baseSize, realSize: data.realSize
                };
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
                targetCameraDistance = Math.max(3, Math.min(140, targetCameraDistance + e.deltaY * 0.04));
            });
            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function updatePlanetScales() {
            Object.keys(planets).forEach(id => {
                const p = planets[id];
                const targetSize = isRealScale ? p.realSize : p.baseSize;
                p.mesh.scale.set(targetSize / p.baseSize, targetSize / p.baseSize, targetSize / p.baseSize);
            });
        }

        function updateCameraPosition() {
            // TỰ ĐỘNG CHUYỂN HÀNH TINH TRONG CHẾ ĐỘ CINEMATIC
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
                
                const zooms = { sun: 65, mercury: 6, venus: 7, earth: 8, mars: 7, jupiter: 14, saturn: 16, uranus: 11, neptune: 11 };
                targetCameraDistance = zooms[id] || 10;
            }
        }

        function animate() {
            requestAnimationFrame(animate);

            // Xoay bão lửa mặt trời
            solarFlares.forEach(flare => { flare.rotation.z += 0.01 * timeSpeed; });

            // Xoay đai tiểu hành tinh
            asteroidBelt.forEach(ast => {
                ast.angle += ast.speed * timeSpeed;
                ast.mesh.position.x = Math.cos(ast.angle) * ast.radius;
                ast.mesh.position.z = Math.sin(ast.angle) * ast.radius;
                ast.mesh.rotation.y += 0.01;
            });

            // Cập nhật vị trí các hành tinh
            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
                    p.angle += p.speed * timeSpeed;
                    p.mesh.position.x = Math.cos(p.angle) * p.distance;
                    p.mesh.position.z = Math.sin(p.angle) * p.distance;
                    p.mesh.rotation.y += 0.015 * timeSpeed;

                    if (p.mesh.userData.cloudMesh) p.mesh.userData.cloudMesh.rotation.y += 0.008 * timeSpeed;
                    if (p.mesh.userData.moonMesh) p.mesh.userData.moonMesh.rotation.y += 0.02 * timeSpeed;
                    if (p.mesh.userData.issMesh) p.mesh.userData.issMesh.rotation.y += 0.04 * timeSpeed;
                }
            });

            updateCameraPosition();
            renderer.render(scene, camera);
        }
    </script>
</body>
</html>
