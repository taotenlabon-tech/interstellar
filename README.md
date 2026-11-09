<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA Solar System 3D & Interstellar Audio</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap');
        body { font-family: 'Share Tech Mono', monospace; background: #000000; color: #38bdf8; overflow: hidden; margin: 0; }
        .hud-panel { background: rgba(3, 7, 18, 0.9); backdrop-filter: blur(12px); border: 1px solid rgba(56, 189, 248, 0.4); box-shadow: 0 0 25px rgba(56, 189, 248, 0.25); }
    </style>
</head>
<body class="w-screen h-screen flex flex-col justify-between select-none">

    <!-- HEADER / THÔNG TIN NASA -->
    <header class="absolute top-4 left-4 z-20 hud-panel p-4 rounded-2xl max-w-sm space-y-2 pointer-events-none">
        <div class="text-xs tracking-widest text-slate-400">NASA EXPLORATION LAB // 2026</div>
        <h1 class="text-xl font-bold text-cyan-400">KHÁM PHÁ HỆ MẶT TRỜI 3D</h1>
        <p class="text-xs text-slate-300 leading-relaxed">
            Chọn hành tinh bên dưới để bay camera đến gần, xem thông số khoa học và tận hưởng nhạc nền không gian Interstellar!
        </p>
    </header>

    <!-- NÚT BẬT/TẮT NHẠC NỀN INTERSTELLAR -->
    <div class="absolute top-4 right-4 z-20">
        <button id="audioToggleBtn" onclick="toggleAudio()" class="hud-panel px-4 py-2.5 rounded-xl text-xs font-bold text-cyan-300 hover:bg-cyan-950 transition flex items-center gap-2 border border-cyan-500/50">
            <span>🎵</span> <span id="audioStatus">Bật Nhạc Interstellar: OFF</span>
        </button>
    </div>

    <!-- BẢNG THÔNG TIN KHÁM PHÁ HÀNH TINH (HIỆN KHI CHỌN) -->
    <div id="planetInfoBox" class="absolute top-28 left-4 z-20 hud-panel p-5 rounded-2xl max-w-xs space-y-3 hidden transition-all duration-500">
        <div class="flex justify-between items-center border-b border-slate-700 pb-2">
            <h2 id="infoTitle" class="text-lg font-bold text-cyan-300">Tên Hành Tinh</h2>
            <button onclick="resetCamera()" class="text-xs text-slate-400 hover:text-white">✕ Thoát</button>
        </div>
        <p id="infoDesc" class="text-xs text-slate-300 leading-relaxed">Mô tả chi tiết...</p>
        <div class="text-[11px] text-cyan-400 space-y-1 bg-slate-950/80 p-2.5 rounded-xl border border-slate-800">
            <div>📏 Đường kính: <span id="infoSize" class="text-white font-bold">--</span></div>
            <div>🌡️ Nhiệt độ bề mặt: <span id="infoTemp" class="text-white font-bold">--</span></div>
            <div>🛰️ Đặc điểm: <span id="infoFeature" class="text-white font-bold">--</span></div>
        </div>
    </div>

    <!-- THANH CHỌN HÀNH TINH -->
    <div class="absolute bottom-6 left-1/2 -translate-x-1/2 z-20 hud-panel px-6 py-3 rounded-2xl flex gap-3 overflow-x-auto max-w-[95vw]">
        <button onclick="focusPlanet('sun')" class="px-3.5 py-2 bg-amber-950/80 border border-amber-500 text-amber-300 rounded-xl text-xs font-bold hover:bg-amber-950 transition">☀️ Mặt Trời</button>
        <button onclick="focusPlanet('earth')" class="px-3.5 py-2 bg-blue-950/80 border border-blue-500 text-blue-300 rounded-xl text-xs font-bold hover:bg-blue-950 transition">🌍 Trái Đất</button>
        <button onclick="focusPlanet('mars')" class="px-3.5 py-2 bg-rose-950/80 border border-rose-500 text-rose-300 rounded-xl text-xs font-bold hover:bg-rose-950 transition">🔴 Sao Hỏa</button>
        <button onclick="focusPlanet('jupiter')" class="px-3.5 py-2 bg-orange-950/80 border border-orange-500 text-orange-300 rounded-xl text-xs font-bold hover:bg-orange-950 transition">🪐 Sao Mộc</button>
        <button onclick="focusPlanet('saturn')" class="px-3.5 py-2 bg-yellow-950/80 border border-yellow-500 text-yellow-300 rounded-xl text-xs font-bold hover:bg-yellow-950 transition">🪐 Sao Thổ</button>
    </div>

    <!-- CANVAS 3D -->
    <div id="canvasContainer" class="absolute inset-0 z-0"></div>

    <script>
        let scene, camera, renderer;
        let planets = {};
        let focusedObject = null;
        let isDragging = false, previousMousePosition = { x: 0, y: 0 };
        let cameraDistance = 35;
        let cameraAngleX = 0.5, cameraAngleY = 0.0;
        let targetCameraPos = new THREE.Vector3(0, 0, 0);

        // Dữ liệu khám phá khoa học hành tinh
        const planetInfoData = {
            earth: { title: "Trái Đất (Earth)", desc: "Hành tinh xanh chứa sự sống duy nhất được biết đến trong hệ mặt trời, với 71% bề mặt phủ nước.", size: "12,742 km", temp: "15°C (Trung bình)", feature: "1 Mặt Trăng vệ tinh, Khí quyển giàu Oxi." },
            mars: { title: "Sao Hỏa (Mars)", desc: "Hành tinh đỏ với bề mặt chứa nhiều oxit sắt, bão cát khổng lồ và ngọn núi lửa cao nhất hệ mặt trời Olympus Mons.", size: "6,779 km", temp: "-60°C", feature: "2 vệ tinh (Phobos & Deimos)." },
            jupiter: { title: "Sao Mộc (Jupiter)", desc: "Hành tinh lớn nhất trong hệ mặt trời, là một khối khí khổng lồ nổi tiếng với Vết Đỏ Lớn - một cơn bão khổng lồ.", size: "139,820 km", temp: "-110°C", feature: "Hơn 95 mặt trăng vệ tinh." },
            saturn: { title: "Sao Thổ (Saturn)", desc: "Nổi tiếng với hệ thống vành đai băng đá và bụi khổng lồ cực kỳ rực rỡ và lộng lẫy.", size: "116,460 km", temp: "-140°C", feature: "Vành đai băng đá lớn nhất." }
        };

        // Trình phát nhạc nền Interstellar (Web Audio API Synthesizer êm dịu)
        let audioCtx = null;
        let isAudioPlaying = false;
        let audioInterval = null;

        function toggleAudio() {
            if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            if (audioCtx.state === 'suspended') audioCtx.resume();

            isAudioPlaying = !isAudioPlaying;
            const statusEl = document.getElementById('audioStatus');
            if (isAudioPlaying) {
                statusEl.innerText = "Bật Nhạc Interstellar: ON";
                statusEl.className = "text-cyan-300 font-bold";
                startInterstellarDrone();
            } else {
                statusEl.innerText = "Bật Nhạc Interstellar: OFF";
                statusEl.className = "";
                if (audioInterval) clearInterval(audioInterval);
            }
        }

        function startInterstellarDrone() {
            // Mô phỏng âm thanh organ/ambient êm dịu kiểu Interstellar (Chords tuần tự)
            const chords = [
                [130.81, 196.00, 261.63], // C3, G3, C4
                [110.00, 164.81, 220.00], // A2, E3, A3
                [92.50, 138.59, 185.00],  // F#2, C#3, F#3
                [98.00, 146.83, 196.00]   // G2, D3, G3
            ];
            let chordIdx = 0;

            function playChord() {
                if (!isAudioPlaying) return;
                const notes = chords[chordIdx];
                chordIdx = (chordIdx + 1) % chords.length;

                notes.forEach(freq => {
                    try {
                        const osc = audioCtx.createOscillator();
                        const gain = audioCtx.createGain();
                        osc.type = 'sine';
                        osc.frequency.setValueAtTime(freq, audioCtx.currentTime);

                        gain.gain.setValueAtTime(0.001, audioCtx.currentTime);
                        gain.gain.linearRampToValueAtTime(0.04, audioCtx.currentTime + 1.5);
                        gain.gain.linearRampToValueAtTime(0.001, audioCtx.currentTime + 5.5);

                        osc.connect(gain);
                        gain.connect(audioCtx.destination);
                        osc.start();
                        osc.stop(audioCtx.currentTime + 5.5);
                    } catch(e) {}
                });
            }

            playChord();
            audioInterval = setInterval(playChord, 5000);
        }

        function init() {
            const container = document.getElementById('canvasContainer');
            scene = new THREE.Scene();

            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 2000);
            updateCameraPosition();

            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            container.appendChild(renderer.domElement);

            // Ánh sáng Mặt Trời
            const sunLight = new THREE.PointLight(0xffffff, 2.8, 600);
            scene.add(sunLight);
            scene.add(new THREE.AmbientLight(0x222222));

            // Nền sao trời
            const starsGeo = new THREE.BufferGeometry();
            const starsCnt = 2500;
            const starPos = new Float32Array(starsCnt * 3);
            for(let i=0; i<starsCnt*3; i++) starPos[i] = (Math.random() - 0.5) * 900;
            starsGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            scene.add(new THREE.Points(starsGeo, new THREE.PointsMaterial({color: 0xffffff, size: 0.8})));

            // 1. MẶT TRỜI
            const sun = new THREE.Mesh(new THREE.SphereGeometry(3.8, 32, 32), new THREE.MeshBasicMaterial({ color: 0xfbbf24 }));
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0 };

            // 2. HÀNH TINH
            const planetData = [
                { id: 'earth', size: 1.0, distance: 10, color: 0x38bdf8, speed: 0.012 },
                { id: 'mars', size: 0.7, distance: 15, color: 0xf43f5e, speed: 0.009 },
                { id: 'jupiter', size: 2.3, distance: 22, color: 0xd97706, speed: 0.006 },
                { id: 'saturn', size: 1.8, distance: 30, color: 0xfde047, speed: 0.004, hasRings: true }
            ];

            planetData.forEach(data => {
                const orbitGroup = new THREE.Group();
                scene.add(orbitGroup);

                // Đường quỹ đạo sáng mờ
                const orbitLine = new THREE.Mesh(new THREE.RingGeometry(data.distance - 0.04, data.distance + 0.04, 64), new THREE.MeshBasicMaterial({ color: 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.12 }));
                orbitLine.rotation.x = Math.PI / 2;
                scene.add(orbitLine);

                const planetMesh = new THREE.Mesh(new THREE.SphereGeometry(data.size, 24, 24), new THREE.MeshStandardMaterial({ color: data.color, roughness: 0.4 }));
                planetMesh.position.x = data.distance;
                orbitGroup.add(planetMesh);

                if (data.hasRings) {
                    const ring = new THREE.Mesh(new THREE.RingGeometry(2.3, 3.6, 32), new THREE.MeshBasicMaterial({ color: 0xca8a04, side: THREE.DoubleSide, transparent: true, opacity: 0.75 }));
                    ring.rotation.x = Math.PI / 2;
                    planetMesh.add(ring);
                }

                planets[data.id] = { mesh: planetMesh, distance: data.distance, speed: data.speed, angle: Math.random() * Math.PI * 2 };
            });

            // Tương tác chuột xoay camera
            window.addEventListener('mousedown', (e) => { isDragging = true; previousMousePosition = { x: e.clientX, y: e.clientY }; });
            window.addEventListener('mousemove', (e) => {
                if (!isDragging) return;
                cameraAngleY -= (e.clientX - previousMousePosition.x) * 0.005;
                cameraAngleX = Math.max(-1.4, Math.min(1.4, cameraAngleX - (e.clientY - previousMousePosition.y) * 0.005));
                previousMousePosition = { x: e.clientX, y: e.clientY };
            });
            window.addEventListener('mouseup', () => isDragging = false);
            window.addEventListener('wheel', (e) => {
                cameraDistance = Math.max(4, Math.min(70, cameraDistance + e.deltaY * 0.03));
            });

            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function focusPlanet(id) {
            focusedObject = id === 'sun' ? null : id;
            const infoBox = document.getElementById('planetInfoBox');

            if (id === 'sun' || !planetInfoData[id]) {
                infoBox.classList.add('hidden');
            } else {
                const data = planetInfoData[id];
                document.getElementById('infoTitle').innerText = data.title;
                document.getElementById('infoDesc').innerText = data.desc;
                document.getElementById('infoSize').innerText = data.size;
                document.getElementById('infoTemp').innerText = data.temp;
                document.getElementById('infoFeature').innerText = data.feature;
                infoBox.classList.remove('hidden');
            }
        }

        function resetCamera() {
            focusedObject = null;
            document.getElementById('planetInfoBox').classList.add('hidden');
        }

        function updateCameraPosition() {
            let targetPos = new THREE.Vector3(0, 0, 0);
            if (focusedObject && planets[focusedObject]) {
                const worldPos = new THREE.Vector3();
                planets[focusedObject].mesh.getWorldPosition(worldPos);
                targetPos.lerp(worldPos, 0.1); // Bay mượt mà tới hành tinh
            }

            const idealX = targetPos.x + cameraDistance * Math.sin(cameraAngleY) * Math.cos(cameraAngleX);
            const idealY = targetPos.y + cameraDistance * Math.sin(cameraAngleX);
            const idealZ = targetPos.z + cameraDistance * Math.cos(cameraAngleY) * Math.cos(cameraAngleX);

            camera.position.set(idealX, idealY, idealZ);
            camera.lookAt(targetPos);
        }

        function animate() {
            requestAnimationFrame(animate);

            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
                    p.angle += p.speed;
                    p.mesh.position.x = Math.cos(p.angle) * p.distance;
                    p.mesh.position.z = Math.sin(p.angle) * p.distance;
                    p.mesh.rotation.y += 0.02;
                }
            });

            updateCameraPosition();
            renderer.render(scene, camera);
        }

        window.onload = init;
    </script>
</body>
</html>
