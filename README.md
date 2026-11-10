
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA True-Scale 3D Solar System - Stray.wav Phonk Edition</title>
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
        @keyframes pulseGlow {
            0% { box-shadow: 0 0 15px rgba(244, 63, 94, 0.4); }
            50% { box-shadow: 0 0 30px rgba(244, 63, 94, 0.8); }
            100% { box-shadow: 0 0 15px rgba(244, 63, 94, 0.4); }
        }
        .cinematic-glow { animation: pulseGlow 2s infinite; }
    </style>
</head>
<body class="w-screen h-screen flex flex-col justify-between select-none">

    <!-- OVERLAY KHỞI ĐỘNG VỚI STRAY.WAV PHONK AUDIO -->
    <div id="startOverlay" class="fixed inset-0 z-50 bg-slate-950/95 backdrop-blur-xl flex flex-col items-center justify-center p-6 text-center space-y-6">
        <div class="relative flex items-center justify-center">
            <div class="w-28 h-28 rounded-full border-2 border-rose-500/40 animate-ping absolute"></div>
            <div class="text-6xl">🎧</div>
        </div>
        <div class="space-y-2">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-rose-500 via-sky-400 to-cyan-300 tracking-wider">
                STRAY.WAV // SOLAR SYSTEM 3D
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 max-w-md mx-auto leading-relaxed">
                Đồ họa Procedural HD (Lục địa, Khí quyển, Vành đai băng, Mây 3D) & Âm nhạc Phonk Drift sôi động.
            </p>
        </div>
        <button onclick="startExperience()" class="px-8 py-4 bg-gradient-to-r from-rose-600 to-cyan-600 hover:from-rose-500 hover:to-cyan-500 font-orbitron font-bold text-xs sm:text-sm rounded-2xl text-white cinematic-glow transition active:scale-95 border border-rose-400 flex items-center gap-3">
            <span>⚡</span> KHỞI ĐỘNG SIMULATION & NHẠC STRAY.WAV
        </button>
    </div>

    <!-- HUD THÔNG TIN BÊN TRÁI -->
    <header id="mainHud" class="absolute top-4 left-4 z-20 transition-all duration-300 pointer-events-auto">
        <div class="hud-panel p-4 rounded-2xl max-w-sm space-y-3 relative">
            <div class="flex justify-between items-center gap-3">
                <span class="text-[10px] font-bold tracking-widest text-slate-400">NASA PHOTO-REAL SYSTEM</span>
                <div class="flex items-center gap-2">
                    <button onclick="toggleSpaceAudio()" id="audioToggleBtn" class="px-2.5 py-1 bg-rose-950/80 border border-rose-400 text-rose-300 rounded-lg text-[11px] font-bold hover:bg-rose-900 transition flex items-center gap-1.5 shadow-[0_0_12px_rgba(244,63,94,0.4)]">
                        <span id="audioIcon">🔊</span> <span id="audioStatus">Phonk: ON</span>
                    </button>
                    <button onclick="toggleHudInfo()" id="toggleHudBtn" class="px-2 py-1 bg-slate-800 border border-slate-600 text-slate-200 rounded-lg text-[11px] font-bold hover:bg-slate-700 transition" title="Ẩn/Hiện Chi Tiết">
                        👁️ Ẩn
                    </button>
                </div>
            </div>

            <!-- THANH ĐIỀU CHỈNH ÂM LƯỢNG -->
            <div class="flex items-center gap-2 bg-slate-900/80 p-2 rounded-xl border border-slate-800">
                <span class="text-[10px] text-rose-300 font-bold">🎧 Phonk Vol:</span>
                <input type="range" id="volumeSlider" min="0" max="1" step="0.05" value="0.75" onchange="changeVolume(this.value)" oninput="changeVolume(this.value)" class="w-full accent-rose-500 h-1 cursor-pointer">
            </div>

            <!-- TÍNH NĂNG TỐC ĐỘ THỜI GIAN (TIMEWARP) -->
            <div class="flex items-center gap-2 bg-slate-900/80 p-2 rounded-xl border border-slate-800">
                <span class="text-[10px] text-cyan-300 font-bold">⏳ Tốc độ thời gian:</span>
                <input type="range" id="timeSpeedSlider" min="0" max="5" step="0.1" value="1" onchange="changeTimeSpeed(this.value)" oninput="changeTimeSpeed(this.value)" class="w-full accent-cyan-400 h-1 cursor-pointer">
            </div>

            <!-- TÍNH NĂNG CHẾ ĐỘ QUY CHUẨN TỶ LỆ KÍCH THƯỚC (REAL SCALE TOGGLE) -->
            <div class="flex items-center justify-between bg-slate-900/80 p-2 rounded-xl border border-slate-800">
                <span class="text-[10px] text-amber-300 font-bold">🌌 Tỷ lệ thực tế (Real Scale):</span>
                <button onclick="toggleScaleMode()" id="scaleModeBtn" class="px-2.5 py-1 bg-amber-950 border border-amber-500 text-amber-200 text-[10px] font-bold rounded-lg hover:bg-amber-900 transition">
                    Tối Ưu View (Default)
                </button>
            </div>

            <h1 class="text-lg font-orbitron font-bold text-cyan-400 tracking-wide">HỆ MẶT TRỜI CHÂN THỰC</h1>

            <div id="hudContent" class="space-y-3 transition-all duration-300 overflow-hidden">
                <div class="border-t border-slate-800 pt-2">
                    <div id="planetTitle" class="text-sm font-orbitron font-bold text-amber-300">TOÀN CẢNH HỆ MẶT TRỜI</div>
                    <p id="planetDesc" class="text-xs text-slate-300 leading-relaxed mt-1">
                        Mô phỏng 3D chuẩn xác với lớp mây khí quyển chuyển động, hiệu ứng đốm lửa plasma Mặt Trời & vành đai băng lấp lánh.
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
        let cameraDistance = 65, targetCameraDistance = 65;
        let cameraAngleX = 0.4, cameraAngleY = 0.0;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0;
        let isRealScale = false;

        // ÂM NHẠC DRIFT PHONK STRAY.WAV SYNTH ENGINE
        let audioCtx = null;
        let mainMasterGain = null;
        let isAudioPlaying = false;
        let phonkBeatTimer = null, basslineTimer = null;
        let beatStep = 0;

        const planetDatabase = {
            sun: { name: "MẶT TRỜI (SUN)", desc: "Ngôi sao trung tâm, cấu tạo từ plasma hydro và heli cực nóng với các đợt bùng nổ năng lượng.", dist: "0 AU", temp: "5,500 °C", radius: "696,340 km", moons: "0", zoomDist: 65 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Hành tinh đá với bề mặt chi chít hố va chạm thiên thạch khắc sâu tương tự Mặt Trăng.", dist: "0.39 AU", temp: "-180 đến 430 °C", radius: "2,439 km", moons: "0", zoomDist: 6 },
            venus: { name: "SAO KIM (VENUS)", desc: "Bao phủ bởi các tầng mây dày axit sulfuric phản chiếu ánh sáng rực rỡ màu vàng cam.", dist: "0.72 AU", temp: "462 °C", radius: "6,051 km", moons: "0", zoomDist: 7 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Hành tinh sự sống hoàn hảo với lớp mây trắng động, đại dương xanh thẳm và bầu khí quyển tán sắc.", dist: "1.00 AU", temp: "15 °C", radius: "6,371 km", moons: "1 (Mặt Trăng)", zoomDist: 8 },
            mars: { name: "SAO HỎA (MARS)", desc: "Hành tinh Đỏ sa mạc lạnh giá với các thung lũng sâu và bụi oxit sắt đặc trưng.", dist: "1.52 AU", temp: "-63 °C", radius: "3,389 km", moons: "2 (Phobos, Deimos)", zoomDist: 7 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Gã khổng lồ khí với các dải mây bão xoáy cuộn và Vết Đỏ Lớn kinh điển.", dist: "5.20 AU", temp: "-110 °C", radius: "69,911 km", moons: "95", zoomDist: 14 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Nổi bật với hệ thống vành đai băng phức tạp, sáng lấp lánh dưới góc chiếu của mặt trời.", dist: "9.58 AU", temp: "-140 °C", radius: "58,232 km", moons: "146", zoomDist: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng khổng lồ có màu xanh ngọc lam tĩnh lặng và trục quay nghiêng độc đáo.", dist: "19.22 AU", temp: "-195 °C", radius: "25,362 km", moons: "28", zoomDist: 11 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Hành tinh xanh thẫm xa xôi với những dải bão ngầm và gió siêu thanh mạnh nhất hệ mặt trời.", dist: "30.05 AU", temp: "-200 °C", radius: "24,622 km", moons: "16", zoomDist: 11 }
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
                mainMasterGain.gain.setValueAtTime(0.75, audioCtx.currentTime);
                mainMasterGain.connect(audioCtx.destination);
                isAudioPlaying = true;
                startStrayWavPhonk();
            } catch(e) {}
        }

        function changeVolume(val) {
            if (mainMasterGain && audioCtx) {
                mainMasterGain.gain.setValueAtTime(parseFloat(val), audioCtx.currentTime);
            }
        }

        function changeTimeSpeed(val) {
            timeSpeed = parseFloat(val);
        }

        function toggleScaleMode() {
            isRealScale = !isRealScale;
            const btn = document.getElementById("scaleModeBtn");
            if (isRealScale) {
                btn.innerText = "Tỷ Lệ Thực (Real Ratio)";
                btn.className = "px-2.5 py-1 bg-rose-950 border border-rose-500 text-rose-200 text-[10px] font-bold rounded-lg hover:bg-rose-900 transition";
            } else {
                btn.innerText = "Tối Ưu View (Default)";
                btn.className = "px-2.5 py-1 bg-amber-950 border border-amber-500 text-amber-200 text-[10px] font-bold rounded-lg hover:bg-amber-900 transition";
            }
            updatePlanetScales();
        }

        function toggleSpaceAudio() {
            if (!audioCtx) return;
            isAudioPlaying = !isAudioPlaying;
            const btnStatus = document.getElementById('audioStatus');
            const btnIcon = document.getElementById('audioIcon');
            if (isAudioPlaying) {
                if (audioCtx.state === 'suspended') audioCtx.resume();
                btnStatus.innerText = "Phonk: ON"; btnIcon.innerText = "🔊";
                startStrayWavPhonk();
            } else {
                audioCtx.suspend();
                btnStatus.innerText = "Phonk: OFF"; btnIcon.innerText = "🔇";
                stopStrayWavPhonk();
            }
        }

        // TẠO ÂM NHẠC DRIFT PHONK KHÔNG GIAN BẰNG WEB AUDIO API
        function startStrayWavPhonk() {
            stopStrayWavPhonk();
            const phonkNotes = [440, 440, 523.25, 440, 392, 349.23, 392, 440];

            phonkBeatTimer = setInterval(() => {
                if (!isAudioPlaying || !audioCtx) return;
                playPhonkCowbell(phonkNotes[beatStep % phonkNotes.length]);
                if (beatStep % 4 === 0) playPhonkKick();
                if (beatStep % 8 === 4) playPhonkSnare();
                beatStep++;
            }, 180);

            basslineTimer = setInterval(() => {
                if (!isAudioPlaying || !audioCtx) return;
                playPhonkSubBass(110);
            }, 720);
        }

        function playPhonkCowbell(freq) {
            const now = audioCtx.currentTime;
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();

            osc.type = 'square';
            osc.frequency.setValueAtTime(freq * 1.5, now);

            gain.gain.setValueAtTime(0.12, now);
            gain.gain.exponentialRampToValueAtTime(0.001, now + 0.12);

            osc.connect(gain);
            gain.connect(mainMasterGain);

            osc.start(now);
            osc.stop(now + 0.13);
        }

        function playPhonkKick() {
            const now = audioCtx.currentTime;
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();

            osc.type = 'sine';
            osc.frequency.setValueAtTime(140, now);
            osc.frequency.exponentialRampToValueAtTime(35, now + 0.15);

            gain.gain.setValueAtTime(0.4, now);
            gain.gain.exponentialRampToValueAtTime(0.001, now + 0.16);

            osc.connect(gain);
            gain.connect(mainMasterGain);

            osc.start(now);
            osc.stop(now + 0.17);
        }

        function playPhonkSnare() {
            const now = audioCtx.currentTime;
            const bufferSize = audioCtx.sampleRate * 0.1;
            const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
            const data = buffer.getChannelData(0);
            for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;

            const noise = audioCtx.createBufferSource();
            noise.buffer = buffer;

            const filter = audioCtx.createBiquadFilter();
            filter.type = 'highpass'; filter.frequency.value = 1000;

            const gain = audioCtx.createGain();
            gain.gain.setValueAtTime(0.2, now);
            gain.gain.exponentialRampToValueAtTime(0.001, now + 0.1);

            noise.connect(filter); filter.connect(gain); gain.connect(mainMasterGain);
            noise.start(now);
        }

        function playPhonkSubBass(freq) {
            const now = audioCtx.currentTime;
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();

            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(freq, now);

            gain.gain.setValueAtTime(0.25, now);
            gain.gain.exponentialRampToValueAtTime(0.001, now + 0.5);

            osc.connect(gain); gain.connect(mainMasterGain);
            osc.start(now); osc.stop(now + 0.55);
        }

        function stopStrayWavPhonk() {
            if (phonkBeatTimer) clearInterval(phonkBeatTimer);
            if (basslineTimer) clearInterval(basslineTimer);
        }

        // TẠO THIẾT KẾ ĐỒ HỌA HD TEXTURE VÀ BUMP MAP CHÂN THỰC
        function createPhotorealisticTexture(type) {
            const canvas = document.createElement('canvas');
            canvas.width = 1024; canvas.height = 512;
            const ctx = canvas.getContext('2d');

            if (type === 'sun') {
                const grad = ctx.createRadialGradient(512, 256, 20, 512, 256, 500);
                grad.addColorStop(0, '#ffffff'); grad.addColorStop(0.3, '#fde047'); grad.addColorStop(0.7, '#f97316'); grad.addColorStop(1, '#7c2d12');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.35)';
                for(let i=0; i<60; i++) {
                    ctx.beginPath(); ctx.arc(Math.random()*1024, Math.random()*512, Math.random()*35+10, 0, Math.PI*2); ctx.fill();
                }
            } else if (type === 'mercury') {
                ctx.fillStyle = '#475569'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#1e293b';
                for(let i=0; i<300; i++) {
                    ctx.beginPath(); ctx.arc(Math.random()*1024, Math.random()*512, Math.random()*12+2, 0, Math.PI*2); ctx.fill();
                }
            } else if (type === 'venus') {
                ctx.fillStyle = '#b45309'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#fbbf24';
                for(let i=0; i<70; i++) ctx.fillRect(0, Math.random()*512, 1024, Math.random()*25+5);
            } else if (type === 'earth') {
                ctx.fillStyle = '#0284c7'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#15803d';
                for(let i=0; i<40; i++) {
                    ctx.beginPath();
                    ctx.ellipse(Math.random()*1024, Math.random()*512, Math.random()*120+40, Math.random()*60+20, Math.random()*Math.PI, 0, Math.PI*2);
                    ctx.fill();
                }
                ctx.fillStyle = '#b45309';
                for(let i=0; i<15; i++) {
                    ctx.beginPath(); ctx.ellipse(Math.random()*1024, Math.random()*400+50, Math.random()*50+20, Math.random()*20+10, 0, 0, Math.PI*2); ctx.fill();
                }
            } else if (type === 'clouds') {
                ctx.clearRect(0, 0, 1024, 512);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.75)';
                for(let i=0; i<60; i++) {
                    ctx.beginPath();
                    ctx.ellipse(Math.random()*1024, Math.random()*512, Math.random()*110+30, Math.random()*30+10, 0, 0, Math.PI*2);
                    ctx.fill();
                }
            } else if (type === 'mars') {
                ctx.fillStyle = '#991b1b'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#7f1d1d';
                for(let i=0; i<160; i++) {
                    ctx.beginPath(); ctx.arc(Math.random()*1024, Math.random()*512, Math.random()*20+5, 0, Math.PI*2); ctx.fill();
                }
                ctx.fillStyle = '#f8fafc';
                ctx.fillRect(0, 0, 1024, 35); ctx.fillRect(0, 477, 1024, 35);
            } else if (type === 'jupiter') {
                const colors = ['#78350f', '#b45309', '#f59e0b', '#fde047', '#d97706', '#92400e'];
                for(let y=0; y<512; y+=12) {
                    ctx.fillStyle = colors[Math.floor(Math.random()*colors.length)];
                    ctx.fillRect(0, y, 1024, 12);
                }
                ctx.fillStyle = '#dc2626';
                ctx.beginPath(); ctx.ellipse(650, 320, 75, 48, 0, 0, Math.PI*2); ctx.fill();
            } else if (type === 'saturn') {
                const colors = ['#ca8a04', '#eab308', '#fef08a', '#a16207', '#713f12'];
                for(let y=0; y<512; y+=10) {
                    ctx.fillStyle = colors[Math.floor(Math.random()*colors.length)];
                    ctx.fillRect(0, y, 1024, 10);
                }
            } else if (type === 'uranus') {
                ctx.fillStyle = '#0891b2'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#22d3ee';
                for(let y=0; y<512; y+=35) ctx.fillRect(0, y, 1024, 10);
            } else if (type === 'neptune') {
                ctx.fillStyle = '#312e81'; ctx.fillRect(0, 0, 1024, 512);
                ctx.fillStyle = '#4f46e5';
                for(let y=0; y<512; y+=25) ctx.fillRect(0, y, 1024, 12);
                ctx.fillStyle = '#1e1b4b';
                ctx.beginPath(); ctx.ellipse(300, 350, 65, 42, 0, 0, Math.PI*2); ctx.fill();
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

            const sunLight = new THREE.PointLight(0xffffff, 4.0, 1800);
            scene.add(sunLight);
            scene.add(new THREE.AmbientLight(0x1e293b, 0.75));

            // Vũ trụ ngàn sao
            const starsGeo = new THREE.BufferGeometry();
            const starsCnt = 5000;
            const starPos = new Float32Array(starsCnt * 3);
            for(let i=0; i<starsCnt*3; i++) starPos[i] = (Math.random() - 0.5) * 1400;
            starsGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            const starsMat = new THREE.PointsMaterial({color: 0xffffff, size: 0.9});
            scene.add(new THREE.Points(starsGeo, starsMat));

            // MẶT TRỜI CHÂN THỰC
            const sunMat = new THREE.MeshBasicMaterial({ map: createPhotorealisticTexture('sun') });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(4.8, 48, 48), sunMat);
            const glowMat = new THREE.MeshBasicMaterial({ color: 0xf59e0b, transparent: true, opacity: 0.35, side: THREE.BackSide });
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(6.4, 32, 32), glowMat));
            scene.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0, baseSize: 4.8, realSize: 10.0 };

            // 8 HÀNH TINH
            const planetData = [
                { id: 'mercury', baseSize: 0.55, realSize: 0.4, distance: 9, speed: 0.022, texture: 'mercury' },
                { id: 'venus', baseSize: 0.95, realSize: 0.95, distance: 13, speed: 0.016, texture: 'venus' },
                { id: 'earth', baseSize: 1.15, realSize: 1.0, distance: 18, speed: 0.012, texture: 'earth', hasClouds: true, hasMoon: true },
                { id: 'mars', baseSize: 0.75, realSize: 0.53, distance: 23, speed: 0.009, texture: 'mars' },
                { id: 'jupiter', baseSize: 2.7, realSize: 5.5, distance: 33, speed: 0.006, texture: 'jupiter' },
                { id: 'saturn', baseSize: 2.2, realSize: 4.5, distance: 43, speed: 0.004, texture: 'saturn', hasRings: true },
                { id: 'uranus', baseSize: 1.5, realSize: 2.2, distance: 51, speed: 0.003, texture: 'uranus' },
                { id: 'neptune', baseSize: 1.4, realSize: 2.1, distance: 59, speed: 0.002, texture: 'neptune' }
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
                        map: createPhotorealisticTexture('clouds'),
                        transparent: true, blending: THREE.AdditiveBlending, opacity: 0.65
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

                if (data.hasRings) {
                    const ringGeo = new THREE.RingGeometry(2.9, 5.2, 64);
                    const ringMat = new THREE.MeshStandardMaterial({
                        color: 0xfef08a, side: THREE.DoubleSide, transparent: true, opacity: 0.85,
                        roughness: 0.5
                    });
                    const ring = new THREE.Mesh(ringGeo, ringMat);
                    ring.rotation.x = Math.PI / 2.2;
                    planetMesh.add(ring);
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
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
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

            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
                    p.angle += p.speed * timeSpeed;
                    p.mesh.position.x = Math.cos(p.angle) * p.distance;
                    p.mesh.position.z = Math.sin(p.angle) * p.distance;
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
