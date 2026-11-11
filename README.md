<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultra 4K Photorealistic Solar System</title>
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
            <div class="text-7xl">✨</div>
        </div>
        <div class="space-y-2">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 via-indigo-400 to-rose-400 tracking-wider">
                ULTRA 4K PHOTOREALISTIC SYSTEM
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 max-w-md mx-auto leading-relaxed">
                Đồ họa 4K chân thực, mây 3D độc lập, ánh sáng khí quyển rực rỡ & ánh đèn đô thị về đêm.
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-cyan-600 via-indigo-600 to-rose-600 font-orbitron font-bold text-xs sm:text-sm rounded-2xl text-white cinematic-glow transition active:scale-95 border border-cyan-400 flex items-center gap-3">
            <span>🚀</span> KHỞI ĐỘNG HỆ THỐNG 4K ULTRA
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
                <span class="text-[10px] font-bold tracking-widest text-slate-400">4K ULTRA OBSERVATORY</span>
                <div class="flex items-center gap-2">
                    <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-2.5 py-1 bg-rose-950/80 border border-rose-400 text-rose-300 rounded-lg text-[11px] font-bold hover:bg-rose-900 transition flex items-center gap-1.5 shadow-[0_0_12px_rgba(244,63,94,0.4)]">
                        <span id="audioIcon">🔊</span> <span id="audioStatus">Straywav: ON</span>
                    </button>
                    <button onclick="toggleHudInfo()" id="toggleHudBtn" class="px-2 py-1 bg-slate-800 border border-slate-600 text-slate-200 rounded-lg text-[11px] font-bold hover:bg-slate-700 transition">
                        👁️ Ẩn
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
                        Hệ thống hành tinh độ phân giải cao, chân thực đến từng chi tiết khí quyển.
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
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, previousMousePosition = { x: 0, y: 0 };
        let cameraDistance = 65, targetCameraDistance = 65;
        let cameraAngleX = 0.4, cameraAngleY = 0.0;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0;

        const planetDatabase = {
            sun: { name: "MẶT TRỜI (SUN)", desc: "Ngôi sao trung tâm plasma nhiệt độ cực cao.", dist: "0 AU", temp: "5,500 °C", radius: "696,340 km", zoomDist: 65 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Hành tinh đá với hố va chạm siêu nét.", dist: "0.39 AU", temp: "-180 đến 430 °C", radius: "2,439 km", zoomDist: 6 },
            venus: { name: "SAO KIM (VENUS)", desc: "Tầng mây axit dày đặc màu vàng cam rực rỡ.", dist: "0.72 AU", temp: "462 °C", radius: "6,051 km", zoomDist: 7 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Hành tinh sự sống với mây nổi 3D & ánh đèn đêm.", dist: "1.00 AU", temp: "15 °C", radius: "6,371 km", zoomDist: 8 },
            mars: { name: "SAO HỎA (MARS)", desc: "Hành tinh Đỏ sa mạc bụi oxit sắt.", dist: "1.52 AU", temp: "-63 °C", radius: "3,389 km", zoomDist: 7 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Gã khổng lồ khí với các dải bão xoáy cuộn 4K.", dist: "5.20 AU", temp: "-110 °C", radius: "69,911 km", zoomDist: 14 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Hệ thống vành đai băng trong suốt lấp lánh.", dist: "9.58 AU", temp: "-140 °C", radius: "58,232 km", zoomDist: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng xanh ngọc lam thanh lịch.", dist: "19.22 AU", temp: "-195 °C", radius: "25,362 km", zoomDist: 11 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Hành tinh gió bão xanh thẫm ngoài biên.", dist: "30.05 AU", temp: "-200 °C", radius: "24,622 km", zoomDist: 11 }
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

        function toggleSpaceAudio() {
            const bgAudio = document.getElementById('bgMusic');
            if (!bgAudio) return;
            if (bgAudio.paused) { bgAudio.play(); document.getElementById('audioStatus').innerText = "Straywav: ON"; }
            else { bgAudio.pause(); document.getElementById('audioStatus').innerText = "Straywav: OFF"; }
        }

        // TẠO TEXTURE 4K PROCEDURAL SIÊU NÉT
        function create4KTexture(type) {
            const canvas = document.createElement('canvas');
            canvas.width = 2048; canvas.height = 1024;
            const ctx = canvas.getContext('2d');

            if (type === 'sun') {
                const grad = ctx.createRadialGradient(1024, 512, 40, 1024, 512, 1000);
                grad.addColorStop(0, '#ffffff'); grad.addColorStop(0.25, '#fde047'); grad.addColorStop(0.65, '#f97316'); grad.addColorStop(1, '#7c2d12');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, 2048, 1024);
                // Đốm mặt trời bùng nổ plasma
                ctx.fillStyle = '#fee2e2';
                for(let i=0; i<400; i++) {
                    ctx.beginPath(); ctx.arc(Math.random()*2048, Math.random()*1024, Math.random()*15+2, 0, Math.PI*2); ctx.fill();
                }
            } else if (type === 'mercury') {
                ctx.fillStyle = '#475569'; ctx.fillRect(0, 0, 2048, 1024);
                ctx.fillStyle = '#1e293b';
                for(let i=0; i<800; i++) { ctx.beginPath(); ctx.arc(Math.random()*2048, Math.random()*1024, Math.random()*18+2, 0, Math.PI*2); ctx.fill(); }
            } else if (type === 'venus') {
                ctx.fillStyle = '#b45309'; ctx.fillRect(0, 0, 2048, 1024);
                ctx.fillStyle = '#fbbf24'; for(let i=0; i<150; i++) ctx.fillRect(0, Math.random()*1024, 2048, Math.random()*35+10);
            } else if (type === 'earth') {
                ctx.fillStyle = '#0284c7'; ctx.fillRect(0, 0, 2048, 1024);
                ctx.fillStyle = '#15803d';
                for(let i=0; i<80; i++) { ctx.beginPath(); ctx.ellipse(Math.random()*2048, Math.random()*1024, Math.random()*220+60, Math.random()*120+30, Math.random()*Math.PI, 0, Math.PI*2); ctx.fill(); }
                // Đèn đô thị lấp lánh ban đêm
                ctx.fillStyle = '#fef08a'; for(let i=0; i<500; i++) ctx.fillRect(Math.random()*2048, Math.random()*1024, 3, 3);
            } else if (type === 'clouds') {
                ctx.clearRect(0, 0, 2048, 1024);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.85)';
                for(let i=0; i<120; i++) { ctx.beginPath(); ctx.ellipse(Math.random()*2048, Math.random()*1024, Math.random()*180+50, Math.random()*50+15, 0, 0, Math.PI*2); ctx.fill(); }
            } else if (type === 'mars') {
                ctx.fillStyle = '#991b1b'; ctx.fillRect(0, 0, 2048, 1024);
                ctx.fillStyle = '#7f1d1d'; for(let i=0; i<350; i++) { ctx.beginPath(); ctx.arc(Math.random()*2048, Math.random()*1024, Math.random()*30+8, 0, Math.PI*2); ctx.fill(); }
            } else if (type === 'jupiter') {
                const colors = ['#78350f', '#b45309', '#f59e0b', '#fde047', '#d97706', '#92400e', '#78350f'];
                for(let y=0; y<1024; y+=16) { ctx.fillStyle = colors[Math.floor(Math.random()*colors.length)]; ctx.fillRect(0, y, 2048, 16); }
                // Vết đỏ lớn Sao Mộc
                ctx.fillStyle = '#dc2626'; ctx.beginPath(); ctx.ellipse(1300, 640, 140, 90, 0, 0, Math.PI*2); ctx.fill();
            } else if (type === 'saturn') {
                const colors = ['#ca8a04', '#eab308', '#fef08a', '#a16207', '#713f12', '#ca8a04'];
                for(let y=0; y<1024; y+=14) { ctx.fillStyle = colors[Math.floor(Math.random()*colors.length)]; ctx.fillRect(0, y, 2048, 14); }
            } else if (type === 'uranus') {
                ctx.fillStyle = '#0891b2'; ctx.fillRect(0, 0, 2048, 1024);
            } else if (type === 'neptune') {
                ctx.fillStyle = '#312e81'; ctx.fillRect(0, 0, 2048, 1024);
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
            scene.add(new THREE.AmbientLight(0x1e293b, 0.9));

            // Nền sao lấp lánh cao cấp
            const starsGeo = new THREE.BufferGeometry();
            const starPos = new Float32Array(6000 * 3);
            for(let i=0; i<6000*3; i++) starPos[i] = (Math.random() - 0.5) * 2500;
            starsGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            scene.add(new THREE.Points(starsGeo, new THREE.PointsMaterial({ color: 0xffffff, size: 0.95, transparent: true, opacity: 0.9 })));

            // MẶT TRỜI 4K
            const sunMat = new THREE.MeshBasicMaterial({ map: create4KTexture('sun') });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.8, 64, 64), sunMat);
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
                { id: 'neptune', size: 1.4, distance: 60, speed: 0.002, texture: 'neptune' }
            ];

            planetData.forEach(data => {
                const orbitGroup = new THREE.Group();
                scene.add(orbitGroup);

                const orbitLine = new THREE.Mesh(new THREE.RingGeometry(data.distance - 0.06, data.distance + 0.06, 128), new THREE.MeshBasicMaterial({ color: 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.15 }));
                orbitLine.rotation.x = Math.PI / 2;
                scene.add(orbitLine);

                const pMat = new THREE.MeshStandardMaterial({ roughness: 0.6, metalness: 0.1, map: create4KTexture(data.texture) });
                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size, 64, 64), pMat);
                planetMesh.position.x = data.distance;
                orbitGroup.add(planetMesh);

                // Khí quyển hào quang phát sáng
                if (data.hasAtmosphere) {
                    const atmosMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8, transparent: true, opacity: 0.22, side: THREE.BackSide });
                    const atmos = new THREE.Mesh(new THREE.SphereGeometry(data.size * 1.12, 32, 32), atmosMat);
                    planetMesh.add(atmos);
                }

                // Lớp mây 3D độc lập xoay quanh Trái Đất
                if (data.hasClouds) {
                    const cloudMat = new THREE.MeshStandardMaterial({ map: create4KTexture('clouds'), transparent: true, blending: THREE.AdditiveBlending, opacity: 0.75 });
                    const cloudMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size * 1.02, 32, 32), cloudMat);
                    planetMesh.add(cloudMesh);
                    planetMesh.userData.cloudMesh = cloudMesh;
                }

                // Vành đai Sao Thổ trong suốt lấp lánh
                if (data.hasRings) {
                    const ring = new THREE.Mesh(new THREE.RingGeometry(2.9, 5.4, 128), new THREE.MeshStandardMaterial({ color: 0xfef08a, side: THREE.DoubleSide, transparent: true, opacity: 0.85 }));
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
                targetCameraDistance = Math.max(3, Math.min(2000, targetCameraDistance + e.deltaY * 0.8));
            });

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

            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
                    p.angle += p.speed * timeSpeed;
                    p.mesh.position.x = Math.cos(p.angle) * p.distance;
                    p.mesh.position.z = Math.sin(p.angle) * p.distance;
                    p.mesh.rotation.y += 0.015 * timeSpeed;

                    if (p.mesh.userData.cloudMesh) {
                        p.mesh.userData.cloudMesh.rotation.y += 0.008 * timeSpeed;
                    }
                }
            });

            updateCameraPosition();
            renderer.render(scene, camera);
        }
    </script>
</body>
</html>
