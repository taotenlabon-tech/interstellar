
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA Solar System 3D - HD Graphics & Interstellar Audio</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap');
        body { font-family: 'Share Tech Mono', monospace; background: #000000; color: #38bdf8; overflow: hidden; margin: 0; }
        .hud-panel { background: rgba(3, 7, 18, 0.9); backdrop-filter: blur(12px); border: 1px solid rgba(56, 189, 248, 0.4); box-shadow: 0 0 25px rgba(56, 189, 248, 0.25); }
        .planet-btn { background: rgba(15, 23, 42, 0.85); border: 1px solid rgba(56, 189, 248, 0.3); transition: all 0.2s ease; }
        .planet-btn:hover { border-color: #38bdf8; box-shadow: 0 0 15px rgba(56, 189, 248, 0.5); transform: translateY(-2px); }
    </style>
</head>
<body class="w-screen h-screen flex flex-col justify-between select-none">

    <!-- MÀN HÌNH KHỞI ĐỘNG (GIÚP MỞ KHÓA ÂM THANH TRÌNH DUYỆT) -->
    <div id="startOverlay" class="fixed inset-0 z-50 bg-black/95 flex flex-col items-center justify-center p-6 text-center space-y-5">
        <div class="text-6xl animate-pulse">🌌</div>
        <h1 class="font-bold text-2xl sm:text-3xl text-cyan-400 tracking-wider">NASA SOLAR SYSTEM 3D EXPLORATION</h1>
        <p class="text-xs text-slate-400 max-w-md leading-relaxed">
            Nhấn nút bên dưới để khởi động mô phỏng 3D hệ mặt trời 8 hành tinh và kích hoạt âm thanh không gian Interstellar.
        </p>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 font-bold text-sm rounded-2xl text-white shadow-[0_0_25px_rgba(56,189,248,0.6)] transition active:scale-95 border border-cyan-400">
            🚀 KHỞI ĐỘNG HỆ THỐNG & BẬT NHẠC
        </button>
    </div>

    <!-- HUD THÔNG TIN NASA & NÚT ÂM NHẠC -->
    <header class="absolute top-4 left-4 z-20 hud-panel p-5 rounded-2xl max-w-sm space-y-3 pointer-events-auto">
        <div class="flex justify-between items-center">
            <span class="text-[10px] tracking-widest text-slate-400">NASA EXPLORATION HD</span>
            <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-3 py-1.5 bg-cyan-950 border border-cyan-400 text-cyan-300 rounded-xl text-xs font-bold hover:bg-cyan-900 transition flex items-center gap-2 shadow-[0_0_15px_rgba(56,189,248,0.5)]">
                <span>🔊</span> <span id="audioStatus">Đang Phát Nhạc</span>
            </button>
        </div>
        <h1 class="text-xl font-bold text-cyan-400 tracking-wide">HỆ MẶT TRỜI 3D (8 HÀNH TINH)</h1>
        <div id="planetInfo" class="text-xs text-slate-300 leading-relaxed border-t border-slate-800 pt-2.5">
            Đang xem: <b class="text-amber-400">Toàn Cảnh Hệ Mặt Trời</b>. Bấm chọn các hành tinh phía dưới để Zoom tự động!
        </div>
    </header>

    <!-- THANH CHỌN HÀNH TINH -->
    <div class="absolute bottom-6 left-1/2 -translate-x-1/2 z-20 hud-panel px-6 py-3.5 rounded-2xl flex gap-2.5 overflow-x-auto max-w-[95vw]">
        <button onclick="focusPlanet('overview', 'Toàn cảnh Hệ Mặt Trời bao gồm 8 hành tinh quay quanh Mặt Trời.')" class="planet-btn px-3 py-1.5 text-amber-300 rounded-xl text-xs font-bold">☀️ Toàn Cảnh</button>
        <button onclick="focusPlanet('mercury', 'Sao Thủy: Hành tinh nhỏ nhất và gần Mặt Trời nhất.')" class="planet-btn px-3 py-1.5 text-slate-300 rounded-xl text-xs font-bold">🪨 Sao Thủy</button>
        <button onclick="focusPlanet('venus', 'Sao Kim: Hành tinh nóng nhất với khí quyển CO2 dày đặc.')" class="planet-btn px-3 py-1.5 text-amber-200 rounded-xl text-xs font-bold">🌕 Sao Kim</button>
        <button onclick="focusPlanet('earth', 'Trái Đất: Ngôi nhà xanh duy nhất có sự sống và nước.')" class="planet-btn px-3 py-1.5 text-blue-400 rounded-xl text-xs font-bold">🌍 Trái Đất</button>
        <button onclick="focusPlanet('mars', 'Sao Hỏa: Hành tinh Đỏ phủ đầy oxit sắt.')" class="planet-btn px-3 py-1.5 text-rose-400 rounded-xl text-xs font-bold">🔴 Sao Hỏa</button>
        <button onclick="focusPlanet('jupiter', 'Sao Mộc: Gã khổng lồ khí lớn nhất hệ mặt trời.')" class="planet-btn px-3 py-1.5 text-orange-300 rounded-xl text-xs font-bold">🪐 Sao Mộc</button>
        <button onclick="focusPlanet('saturn', 'Sao Thổ: Nổi bật với vành đai băng lấp lánh ngoạn mục.')" class="planet-btn px-3 py-1.5 text-yellow-300 rounded-xl text-xs font-bold">🪐 Sao Thổ</button>
        <button onclick="focusPlanet('uranus', 'Sao Thiên Vương: Hành tinh băng xanh lam nghiêng độc đáo.')" class="planet-btn px-3 py-1.5 text-cyan-300 rounded-xl text-xs font-bold">💎 Sao Thiên Vương</button>
        <button onclick="focusPlanet('neptune', 'Sao Hải Vương: Bão siêu tốc ở khoảng cách xa nhất.')" class="planet-btn px-3 py-1.5 text-indigo-400 rounded-xl text-xs font-bold">🔵 Sao Hải Vương</button>
    </div>

    <!-- CANVAS 3D KHÔNG GIAN -->
    <div id="canvasContainer" class="absolute inset-0 z-0"></div>

    <script>
        let scene, camera, renderer;
        let planets = {};
        let focusedObject = 'overview';
        let isDragging = false, previousMousePosition = { x: 0, y: 0 };
        let cameraDistance = 55;
        let targetCameraDistance = 55;
        let cameraAngleX = 0.4, cameraAngleY = 0.0;
        let currentLookAt = new THREE.Vector3(0, 0, 0);

        // HỆ THỐNG ÂM THANH INTERSTELLAR (WEB AUDIO API)
        let audioCtx = null;
        let isAudioPlaying = false;
        let ambientTimer = null;

        function startExperience() {
            document.getElementById('startOverlay').style.display = 'none';
            init3D();
            
            // Khởi tạo AudioContext sau khi người dùng click
            try {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                if (audioCtx.state === 'suspended') audioCtx.resume();
                isAudioPlaying = true;
                startInterstellarSound();
            } catch(e) {}
        }

        function toggleSpaceAudio() {
            if (!audioCtx) return;
            isAudioPlaying = !isAudioPlaying;
            const btnStatus = document.getElementById('audioStatus');
            if (isAudioPlaying) {
                if (audioCtx.state === 'suspended') audioCtx.resume();
                btnStatus.innerText = "Đang Phát Nhạc";
                startInterstellarSound();
            } else {
                audioCtx.suspend();
                btnStatus.innerText = "Đã Tắt Nhạc";
                stopInterstellarSound();
            }
        }

        function startInterstellarSound() {
            playInterstellarChord();
            if (ambientTimer) clearInterval(ambientTimer);
            ambientTimer = setInterval(() => {
                if (isAudioPlaying) playInterstellarChord();
            }, 7500);
        }

        function playInterstellarChord() {
            if (!isAudioPlaying || !audioCtx) return;
            const now = audioCtx.currentTime;
            const frequencies = [130.81, 164.81, 196.00, 261.63, 329.63, 392.00];

            frequencies.forEach((freq, idx) => {
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();

                osc.type = idx % 2 === 0 ? 'sine' : 'triangle';
                osc.frequency.setValueAtTime(freq, now);

                gain.gain.setValueAtTime(0.0001, now);
                gain.gain.linearRampToValueAtTime(0.035, now + 2.5);
                gain.gain.exponentialRampToValueAtTime(0.0001, now + 7.0);

                osc.connect(gain);
                gain.connect(audioCtx.destination);

                osc.start(now);
                osc.stop(now + 7.2);
            });
        }

        function stopInterstellarSound() {
            if (ambientTimer) clearInterval(ambientTimer);
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

            const sunLight = new THREE.PointLight(0xffffff, 3.2, 1000);
            scene.add(sunLight);
            scene.add(new THREE.AmbientLight(0x223344, 0.8));

            // Ngàn sao vũ trụ
            const starsGeo = new THREE.BufferGeometry();
            const starsCnt = 3500;
            const starPos = new Float32Array(starsCnt * 3);
            for(let i=0; i<starsCnt*3; i++) starPos[i] = (Math.random() - 0.5) * 1000;
            starsGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            const starsMat = new THREE.PointsMaterial({color: 0xffffff, size: 0.9});
            scene.add(new THREE.Points(starsGeo, starsMat));

            // MẶT TRỜI HD PHÁT SÁNG
            const sunMat = new THREE.MeshBasicMaterial({ color: 0xffb703 });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.2, 32, 32), sunMat);
            const glowMat = new THREE.MeshBasicMaterial({ color: 0xfbbf24, transparent: true, opacity: 0.25, side: THREE.BackSide });
            const sunGlow = new THREE.Mesh(new THREE.SphereGeometry(5.4, 32, 32), glowMat);
            sun.add(sunGlow);
            scene.add(sun);
            planets['overview'] = { mesh: sun, distance: 0, speed: 0, zoomDist: 55 };

            // 8 HÀNH TINH
            const planetData = [
                { id: 'mercury', size: 0.5, distance: 8, color: 0x94a3b8, speed: 0.022, zoomDist: 4 },
                { id: 'venus', size: 0.9, distance: 12, color: 0xfde047, speed: 0.016, zoomDist: 5 },
                { id: 'earth', size: 1.0, distance: 17, color: 0x38bdf8, speed: 0.012, zoomDist: 6 },
                { id: 'mars', size: 0.7, distance: 22, color: 0xf43f5e, speed: 0.009, zoomDist: 5 },
                { id: 'jupiter', size: 2.5, distance: 30, color: 0xca8a04, speed: 0.006, zoomDist: 10 },
                { id: 'saturn', size: 2.0, distance: 39, color: 0xeab308, speed: 0.004, zoomDist: 11, hasRings: true },
                { id: 'uranus', size: 1.4, distance: 47, color: 0x22d3ee, speed: 0.003, zoomDist: 8 },
                { id: 'neptune', size: 1.3, distance: 55, color: 0x6366f1, speed: 0.002, zoomDist: 8 }
            ];

            planetData.forEach(data => {
                const orbitGroup = new THREE.Group();
                scene.add(orbitGroup);

                const orbitGeo = new THREE.RingGeometry(data.distance - 0.05, data.distance + 0.05, 128);
                const orbitMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.15 });
                const orbitLine = new THREE.Mesh(orbitGeo, orbitMat);
                orbitLine.rotation.x = Math.PI / 2;
                scene.add(orbitLine);

                const pMat = new THREE.MeshStandardMaterial({ color: data.color, roughness: 0.4, metalness: 0.2 });
                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size, 32, 32), pMat);
                planetMesh.position.x = data.distance;
                orbitGroup.add(planetMesh);

                if (data.hasRings) {
                    const ringGeo = new THREE.RingGeometry(2.6, 4.2, 64);
                    const ringMat = new THREE.MeshBasicMaterial({ color: 0xfef08a, side: THREE.DoubleSide, transparent: true, opacity: 0.75 });
                    const ring = new THREE.Mesh(ringGeo, ringMat);
                    ring.rotation.x = Math.PI / 2.2;
                    planetMesh.add(ring);
                }

                planets[data.id] = { mesh: planetMesh, group: orbitGroup, distance: data.distance, speed: data.speed, angle: Math.random() * Math.PI * 2, zoomDist: data.zoomDist };
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
                targetCameraDistance = Math.max(3, Math.min(110, targetCameraDistance + e.deltaY * 0.04));
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

        function focusPlanet(id, infoText) {
            focusedObject = id;
            document.getElementById('planetInfo').innerHTML = `Đang chọn: <b class="text-cyan-300 font-bold">${id.toUpperCase()}</b><br>${infoText}`;
            if (planets[id]) {
                targetCameraDistance = planets[id].zoomDist;
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
