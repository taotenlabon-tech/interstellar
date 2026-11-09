<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA Solar System 3D - Hans Zimmer Interstellar Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Share+Tech+Mono&display=swap');
        body { font-family: 'Share Tech Mono', monospace; background: #000000; color: #38bdf8; overflow: hidden; margin: 0; }
        .font-orbitron { font-family: 'Orbitron', sans-serif; }
        .hud-panel { background: rgba(3, 7, 18, 0.92); backdrop-filter: blur(16px); border: 1px solid rgba(56, 189, 248, 0.4); box-shadow: 0 0 30px rgba(56, 189, 248, 0.25); }
        .planet-btn { background: rgba(15, 23, 42, 0.85); border: 1px solid rgba(56, 189, 248, 0.3); transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1); }
        .planet-btn:hover, .planet-btn.active { border-color: #38bdf8; box-shadow: 0 0 18px rgba(56, 189, 248, 0.6); transform: translateY(-2px); background: rgba(14, 165, 233, 0.2); }
        .custom-scrollbar::-webkit-scrollbar { height: 6px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: rgba(15, 23, 42, 0.6); border-radius: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #0284c7; border-radius: 4px; }
    </style>
</head>
<body class="w-screen h-screen flex flex-col justify-between select-none">

    <!-- MÀN HÌNH KHỞI ĐỘNG CÓ BẬT ÂM THANH -->
    <div id="startOverlay" class="fixed inset-0 z-50 bg-slate-950/95 backdrop-blur-xl flex flex-col items-center justify-center p-6 text-center space-y-6">
        <div class="relative flex items-center justify-center">
            <div class="w-24 h-24 rounded-full border-2 border-cyan-400/30 animate-ping absolute"></div>
            <div class="text-6xl">🎹</div>
        </div>
        <div class="space-y-2">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 via-sky-300 to-amber-300 tracking-wider">
                INTERSTELLAR SOUND EXPERIENCE
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 max-w-md mx-auto leading-relaxed">
                Khám phá 8 Hành Tinh 3D với Giai Điệu Organ & Nhịp Đếm Đồng Hồ Chuẩn Phong Cách Hans Zimmer.
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 font-orbitron font-bold text-xs sm:text-sm rounded-2xl text-white shadow-[0_0_30px_rgba(56,189,248,0.5)] transition active:scale-95 border border-cyan-300 flex items-center gap-3">
            <span>🚀</span> KHỞI ĐỘNG HỆ THỐNG & BẬT NHẠC LỚN
        </button>
    </div>

    <!-- HUD THÔNG TIN BÊN TRÁI -->
    <header id="mainHud" class="absolute top-4 left-4 z-20 transition-all duration-300 pointer-events-auto">
        <div class="hud-panel p-4 rounded-2xl max-w-sm space-y-3 relative">
            <div class="flex justify-between items-center gap-3">
                <span class="text-[10px] font-bold tracking-widest text-slate-400">NASA DATA SYSTEM</span>
                <div class="flex items-center gap-2">
                    <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-2.5 py-1 bg-cyan-950/80 border border-cyan-400 text-cyan-300 rounded-lg text-[11px] font-bold hover:bg-cyan-900 transition flex items-center gap-1.5 shadow-[0_0_12px_rgba(56,189,248,0.4)]">
                        <span id="audioIcon">🔊</span> <span id="audioStatus">Tắt Nhạc</span>
                    </button>
                    <button onclick="toggleHudInfo()" id="toggleHudBtn" class="px-2 py-1 bg-slate-800 border border-slate-600 text-slate-200 rounded-lg text-[11px] font-bold hover:bg-slate-700 transition" title="Ẩn/Hiện Chi Tiết">
                        👁️ Ẩn
                    </button>
                </div>
            </div>

            <!-- THANH ĐIỀU CHỈNH ÂM LƯỢNG -->
            <div class="flex items-center gap-2 bg-slate-900/80 p-2 rounded-xl border border-slate-800">
                <span class="text-[10px] text-slate-300 font-bold">🔊 Âm lượng:</span>
                <input type="range" id="volumeSlider" min="0" max="1" step="0.05" value="0.7" onchange="changeVolume(this.value)" oninput="changeVolume(this.value)" class="w-full accent-cyan-400 h-1 cursor-pointer">
            </div>

            <h1 class="text-lg font-orbitron font-bold text-cyan-400 tracking-wide">HỆ MẶT TRỜI 3D</h1>

            <div id="hudContent" class="space-y-3 transition-all duration-300 overflow-hidden">
                <div class="border-t border-slate-800 pt-2">
                    <div id="planetTitle" class="text-sm font-orbitron font-bold text-amber-300">TOÀN CẢNH HỆ MẶT TRỜI</div>
                    <p id="planetDesc" class="text-xs text-slate-300 leading-relaxed mt-1">
                        Hệ Mặt Trời bao gồm Mặt Trời ở trung tâm và 8 hành tinh quay xung quanh trên các quỹ đạo elip.
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

    <!-- THANH CHỌN HÀNH TINH BÊN DƯỚI -->
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
        let focusedObject = 'sun';
        let isHudVisible = true;
        let isDragging = false, previousMousePosition = { x: 0, y: 0 };
        let cameraDistance = 60, targetCameraDistance = 60;
        let cameraAngleX = 0.4, cameraAngleY = 0.0;
        let currentLookAt = new THREE.Vector3(0, 0, 0);

        // HỆ THỐNG ÂM NHẠC HANS ZIMMER INTERSTELLAR ENGINE
        let audioCtx = null;
        let mainMasterGain = null;
        let isAudioPlaying = false;
        let melodyTimer = null, tickTimer = null;

        const planetDatabase = {
            sun: { name: "MẶT TRỜI (SUN)", desc: "Ngôi sao ở trung tâm hệ mặt trời, chứa 99.8% khối lượng của toàn bộ hệ thống.", dist: "0 AU", temp: "5,500 °C", radius: "696,340 km", moons: "0", zoomDist: 60 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Hành tinh nhỏ nhất và gần Mặt Trời nhất, không có bầu khí quyển giữ nhiệt.", dist: "0.39 AU", temp: "-180 đến 430 °C", radius: "2,439 km", moons: "0", zoomDist: 6 },
            venus: { name: "SAO KIM (VENUS)", desc: "Hành tinh nóng nhất do hiệu ứng nhà kính cực đại với bầu khí quyển CO2 dày đặc.", dist: "0.72 AU", temp: "462 °C", radius: "6,051 km", moons: "0", zoomDist: 7 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Hành tinh duy nhất được biết đến có sự sống, bề mặt phủ 71% là đại dương nước lỏng.", dist: "1.00 AU", temp: "15 °C", radius: "6,371 km", moons: "1 (Mặt Trăng)", zoomDist: 8 },
            mars: { name: "SAO HỎA (MARS)", desc: "Hành tinh Đỏ chứa nhiều oxit sắt, sở hữu ngọn núi lửa Olympus Mons lớn nhất hệ mặt trời.", dist: "1.52 AU", temp: "-63 °C", radius: "3,389 km", moons: "2 (Phobos, Deimos)", zoomDist: 7 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Gã khổng lồ khí lớn nhất, nổi tiếng với cơn bão Vết Đỏ Lớn tồn tại hàng trăm năm.", dist: "5.20 AU", temp: "-110 °C", radius: "69,911 km", moons: "95", zoomDist: 14 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Hành tinh có hệ thống vành đai băng lấp lánh và hoành tráng nhất trong vũ trụ.", dist: "9.58 AU", temp: "-140 °C", radius: "58,232 km", moons: "146", zoomDist: 15 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng khổng lồ có trục tự quay nghiêng tới 98 độ gần như nằm ngang.", dist: "19.22 AU", temp: "-195 °C", radius: "25,362 km", moons: "28", zoomDist: 11 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Hành tinh xa nhất, có những cơn gió siêu tốc lên tới 2,100 km/h trong vũ trụ.", dist: "30.05 AU", temp: "-200 °C", radius: "24,622 km", moons: "16", zoomDist: 11 }
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
            try {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                if (audioCtx.state === 'suspended') audioCtx.resume();
                mainMasterGain = audioCtx.createGain();
                mainMasterGain.gain.setValueAtTime(0.7, audioCtx.currentTime);
                mainMasterGain.connect(audioCtx.destination);
                isAudioPlaying = true;
                startInterstellarMusic();
            } catch(e) {}
        }

        function changeVolume(val) {
            if (mainMasterGain && audioCtx) {
                mainMasterGain.gain.setValueAtTime(parseFloat(val), audioCtx.currentTime);
            }
        }

        function toggleSpaceAudio() {
            if (!audioCtx) return;
            isAudioPlaying = !isAudioPlaying;
            const btnStatus = document.getElementById('audioStatus');
            const btnIcon = document.getElementById('audioIcon');
            if (isAudioPlaying) {
                if (audioCtx.state === 'suspended') audioCtx.resume();
                btnStatus.innerText = "Tắt Nhạc"; btnIcon.innerText = "🔊";
                startInterstellarMusic();
            } else {
                audioCtx.suspend();
                btnStatus.innerText = "Bật Nhạc"; btnIcon.innerText = "🔇";
                stopInterstellarMusic();
            }
        }

        // TỰ ĐỘNG PHÁT CHUỖI GIAI ĐIỆU ORGAN CORNFIELD CHASE (INTERSTELLAR)
        function startInterstellarMusic() {
            stopInterstellarMusic();

            // Giai điệu organ Hans Zimmer nổi tiếng (Tần số nốt nhạc Hz)
            const melodyNotes = [
                { f: 261.63, d: 0.4 }, { f: 293.66, d: 0.4 }, { f: 329.63, d: 0.4 }, { f: 392.00, d: 0.6 },
                { f: 440.00, d: 0.6 }, { f: 392.00, d: 0.4 }, { f: 329.63, d: 0.6 }, { f: 261.63, d: 0.8 }
            ];

            let noteIdx = 0;
            melodyTimer = setInterval(() => {
                if (!isAudioPlaying || !audioCtx) return;
                const note = melodyNotes[noteIdx];
                playOrganNote(note.f, note.d);
                noteIdx = (noteIdx + 1) % melodyNotes.length;
            }, 550);

            // Tiếng gõ nhịp đồng hồ Interstellar Ticking (Mỗi 1.25s)
            tickTimer = setInterval(() => {
                if (!isAudioPlaying || !audioCtx) return;
                playClockTick();
            }, 1250);
        }

        function playOrganNote(freq, duration) {
            const now = audioCtx.currentTime;
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();

            // Sóng SAWTOOTH kết hợp TRIANGLE tạo chất âm Organ rực rỡ
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(freq, now);

            gain.gain.setValueAtTime(0.001, now);
            gain.gain.linearRampToValueAtTime(0.25, now + 0.08);
            gain.gain.exponentialRampToValueAtTime(0.001, now + duration);

            osc.connect(gain);
            gain.connect(mainMasterGain);

            osc.start(now);
            osc.stop(now + duration);
        }

        function playClockTick() {
            const now = audioCtx.currentTime;
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();

            osc.type = 'sine';
            osc.frequency.setValueAtTime(1200, now);
            osc.frequency.exponentialRampToValueAtTime(200, now + 0.05);

            gain.gain.setValueAtTime(0.18, now);
            gain.gain.exponentialRampToValueAtTime(0.001, now + 0.05);

            osc.connect(gain);
            gain.connect(mainMasterGain);

            osc.start(now);
            osc.stop(now + 0.06);
        }

        function stopInterstellarMusic() {
            if (melodyTimer) clearInterval(melodyTimer);
            if (tickTimer) clearInterval(tickTimer);
        }

        function createProceduralTexture(type) {
            const canvas = document.createElement('canvas');
            canvas.width = 512; canvas.height = 512;
            const ctx = canvas.getContext('2d');

            if (type === 'sun') {
                const grad = ctx.createRadialGradient(256, 256, 10, 256, 256, 256);
                grad.addColorStop(0, '#fff5c4'); grad.addColorStop(0.5, '#fbbf24'); grad.addColorStop(1, '#d97706');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, 512, 512);
            } else if (type === 'earth') {
                ctx.fillStyle = '#1d4ed8'; ctx.fillRect(0, 0, 512, 512);
                ctx.fillStyle = '#15803d';
                for(let i=0; i<15; i++) {
                    ctx.beginPath(); ctx.arc(Math.random()*512, Math.random()*512, Math.random()*80 + 30, 0, Math.PI*2); ctx.fill();
                }
            } else if (type === 'jupiter') {
                const grad = ctx.createLinearGradient(0, 0, 0, 512);
                grad.addColorStop(0, '#78350f'); grad.addColorStop(0.3, '#f59e0b');
                grad.addColorStop(0.5, '#fde047'); grad.addColorStop(0.7, '#d97706');
                grad.addColorStop(1, '#451a03');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, 512, 512);
            } else {
                ctx.fillStyle = '#64748b'; ctx.fillRect(0, 0, 512, 512);
            }
            return new THREE.CanvasTexture(canvas);
        }

        function init3D() {
            const container = document.getElementById('canvasContainer');
            scene = new THREE.Scene();

            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 2000);
            
            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.toneMapping = THREE.ACESFilmicToneMapping;
            container.appendChild(renderer.domElement);

            const sunLight = new THREE.PointLight(0xffffff, 3.5, 1200);
            scene.add(sunLight);
            scene.add(new THREE.AmbientLight(0x334155, 0.8));

            const starsGeo = new THREE.BufferGeometry();
            const starsCnt = 4000;
            const starPos = new Float32Array(starsCnt * 3);
            for(let i=0; i<starsCnt*3; i++) starPos[i] = (Math.random() - 0.5) * 1100;
            starsGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            const starsMat = new THREE.PointsMaterial({color: 0xffffff, size: 0.95});
            scene.add(new THREE.Points(starsGeo, starsMat));

            const sunMat = new THREE.MeshBasicMaterial({ map: createProceduralTexture('sun') });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.5, 32, 32), sunMat);
            const glowMat = new THREE.MeshBasicMaterial({ color: 0xfbbf24, transparent: true, opacity: 0.25, side: THREE.BackSide });
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(5.8, 32, 32), glowMat));
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0 };

            const planetData = [
                { id: 'mercury', size: 0.5, distance: 9, color: 0x94a3b8, speed: 0.022 },
                { id: 'venus', size: 0.9, distance: 13, color: 0xfde047, speed: 0.016 },
                { id: 'earth', size: 1.1, distance: 18, color: 0x38bdf8, speed: 0.012, texture: 'earth', hasMoon: true },
                { id: 'mars', size: 0.7, distance: 23, color: 0xf43f5e, speed: 0.009 },
                { id: 'jupiter', size: 2.6, distance: 32, color: 0xd97706, speed: 0.006, texture: 'jupiter' },
                { id: 'saturn', size: 2.1, distance: 41, color: 0xeab308, speed: 0.004, hasRings: true },
                { id: 'uranus', size: 1.4, distance: 49, color: 0x22d3ee, speed: 0.003 },
                { id: 'neptune', size: 1.3, distance: 57, color: 0x6366f1, speed: 0.002 }
            ];

            planetData.forEach(data => {
                const orbitGroup = new THREE.Group();
                scene.add(orbitGroup);

                const orbitGeo = new THREE.RingGeometry(data.distance - 0.06, data.distance + 0.06, 128);
                const orbitMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.15 });
                const orbitLine = new THREE.Mesh(orbitGeo, orbitMat);
                orbitLine.rotation.x = Math.PI / 2;
                scene.add(orbitLine);

                const pMat = new THREE.MeshStandardMaterial({
                    color: data.color, roughness: 0.4, metalness: 0.2,
                    map: data.texture ? createProceduralTexture(data.texture) : null
                });
                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size, 32, 32), pMat);
                planetMesh.position.x = data.distance;
                orbitGroup.add(planetMesh);

                if (data.hasMoon) {
                    const moonMesh = new THREE.Mesh(new THREE.SphereGeometry(0.28, 16, 16), new THREE.MeshStandardMaterial({ color: 0xe2e8f0 }));
                    moonMesh.position.x = 2.0;
                    planetMesh.add(moonMesh);
                }

                if (data.hasRings) {
                    const ringGeo = new THREE.RingGeometry(2.8, 4.5, 64);
                    const ringMat = new THREE.MeshBasicMaterial({ color: 0xfef08a, side: THREE.DoubleSide, transparent: true, opacity: 0.75 });
                    const ring = new THREE.Mesh(ringGeo, ringMat);
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
            window.addEventListener('wheel', (e) => {
                targetCameraDistance = Math.max(3, Math.min(120, targetCameraDistance + e.deltaY * 0.04));
            });

            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function updateCameraPosition() {
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

            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
                    p.angle += p.speed;
                    p.mesh.position.x = Math.cos(p.angle) * p.distance;
                    p.mesh.position.z = Math.sin(p.angle) * p.distance;
                    p.mesh.rotation.y += 0.015;
                }
            });

            updateCameraPosition();
            renderer.render(scene, camera);
        }
    </script>
</body>
</html>
