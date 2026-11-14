
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA Eyes 3D - Cosmic & Earth Civilization Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/EffectComposer.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/RenderPass.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/ShaderPass.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/shaders/CopyShader.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/shaders/LuminosityHighPassShader.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/postprocessing/UnrealBloomPass.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/objects/Lensflare.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Share+Tech+Mono&display=swap');
        body { font-family: 'Share Tech Mono', monospace; background: #000; color: #38bdf8; overflow: hidden; margin: 0; }
        .font-orbitron { font-family: 'Orbitron', sans-serif; }
        .hud-glass {
            background: rgba(3, 7, 18, 0.85);
            backdrop-filter: blur(24px);
            border: 1px solid rgba(56, 189, 248, 0.4);
            box-shadow: 0 10px 40px rgba(0, 0, 0, 0.95), inset 0 0 30px rgba(56, 189, 248, 0.12);
        }
        .planet-pill {
            background: rgba(15, 23, 42, 0.7);
            border: 1px solid rgba(148, 163, 184, 0.25);
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .planet-pill:hover, .planet-pill.active {
            border-color: #38bdf8;
            background: rgba(14, 165, 233, 0.28);
            box-shadow: 0 0 25px rgba(56, 189, 248, 0.6);
            transform: translateY(-2px);
        }
        .upgrade-card {
            background: rgba(15, 23, 42, 0.85);
            border: 1px solid rgba(56, 189, 248, 0.3);
            transition: all 0.2s ease;
        }
        .upgrade-card:hover:not(:disabled) {
            border-color: #34d399;
            background: rgba(6, 78, 59, 0.45);
            transform: translateY(-2px);
            box-shadow: 0 0 20px rgba(52, 211, 153, 0.35);
        }
        .upgrade-card:disabled {
            opacity: 0.45;
            cursor: not-allowed;
        }
        .custom-scrollbar::-webkit-scrollbar { height: 4px; width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: rgba(56, 189, 248, 0.6); border-radius: 99px; }
    </style>
</head>
<body class="w-screen h-screen select-none">

    <!-- MÀN HÌNH CHỜ KHỞI ĐỘNG -->
    <div id="startOverlay" class="fixed inset-0 z-50 bg-[#010206] flex flex-col items-center justify-center p-6 text-center space-y-6">
        <div class="relative w-32 h-32 flex items-center justify-center">
            <div class="absolute inset-0 rounded-full border-2 border-cyan-400/50 animate-ping"></div>
            <div class="w-24 h-24 rounded-full bg-gradient-to-tr from-amber-500 via-orange-500 to-cyan-300 shadow-[0_0_100px_rgba(56,189,248,0.9)]"></div>
        </div>
        <div class="space-y-2 max-w-lg">
            <h1 class="font-orbitron font-black text-2xl sm:text-5xl text-white tracking-widest">
                COSMIC <span class="text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 via-emerald-400 to-indigo-400">CIVILIZATION 3D</span>
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 leading-relaxed">
                Khám phá Hệ Mặt Trời 3D & Đổ bộ xuống Trái Đất để kiến tạo, nâng cấp nền văn minh sự sống!
            </p>
        </div>
        <button onclick="startExperience()" class="px-9 py-4 bg-gradient-to-r from-cyan-500 via-emerald-500 to-indigo-500 hover:brightness-110 font-orbitron font-bold text-xs sm:text-sm rounded-full text-white shadow-[0_0_60px_rgba(56,189,248,0.8)] transition active:scale-95">
            ⚡ KHỞI ĐỘNG VŨ TRỤ
        </button>
    </div>

    <!-- =====================================================================
         GIAO DIỆN CHẾ ĐỘ VŨ TRỤ (SPACE MODE HUD)
    ===================================================================== -->
    <div id="spaceUI">
        <header class="absolute top-4 left-4 z-20 hud-glass p-4 rounded-2xl w-80 space-y-3">
            <div class="flex justify-between items-center">
                <div class="flex items-center gap-2">
                    <span class="w-2 h-2 rounded-full bg-cyan-400 animate-pulse"></span>
                    <span class="text-[10px] font-bold tracking-widest text-slate-300">COSMIC ENGINE 3D</span>
                </div>
                <div class="flex gap-1.5">
                    <button onclick="toggleSpaceAudio()" id="audioBtn" class="px-2 py-1 bg-slate-900/90 border border-slate-700 text-rose-300 rounded-lg text-[10px] font-bold hover:border-rose-400 transition">
                        🔊 Music: ON
                    </button>
                    <button onclick="toggleHud()" id="hudToggleBtn" class="px-2 py-1 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-lg text-[10px] font-bold hover:border-cyan-400 transition">
                        👁️ Ẩn
                    </button>
                </div>
            </div>

            <div id="hudBody" class="space-y-3 transition-all duration-300 overflow-hidden">
                <div class="grid grid-cols-2 gap-2 text-[10px]">
                    <button onclick="toggleFreeFly()" id="flyBtn" class="py-1.5 px-2 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-xl font-bold hover:border-cyan-400 transition">
                        🛸 Free-Fly: OFF
                    </button>
                    <button onclick="toggleSpacetimeGrid()" id="gridBtn" class="py-1.5 px-2 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-xl font-bold hover:border-cyan-400 transition">
                        🕸️ Lưới KG: OFF
                    </button>
                </div>

                <div class="grid grid-cols-2 gap-2 text-[10px]">
                    <div class="bg-slate-950/60 p-2 rounded-xl border border-slate-800/80">
                        <div class="flex justify-between text-slate-400 mb-1"><span>🔊 Âm lượng</span><span id="volVal" class="text-rose-400 font-bold">85%</span></div>
                        <input type="range" min="0" max="1" step="0.05" value="0.85" oninput="setVolume(this.value)" class="w-full accent-rose-500 h-1 cursor-pointer">
                    </div>
                    <div class="bg-slate-950/60 p-2 rounded-xl border border-slate-800/80">
                        <div class="flex justify-between text-slate-400 mb-1"><span>⏳ Tốc độ</span><span id="speedVal" class="text-cyan-300">1.0x</span></div>
                        <input type="range" min="0" max="5" step="0.1" value="1" oninput="setSpeed(this.value)" class="w-full accent-cyan-400 h-1 cursor-pointer">
                    </div>
                </div>

                <div class="border-t border-slate-800/80 pt-2.5 space-y-1.5">
                    <div id="planetTitle" class="text-sm font-orbitron font-bold text-amber-300 tracking-wide">MẶT TRỜI (SOL)</div>
                    <p id="planetDesc" class="text-[11px] text-slate-300 leading-relaxed">
                        Bấm chọn Trái Đất bên dưới để mở khóa cổng đổ bộ xuống nền văn minh sự sống!
                    </p>
                    <div class="grid grid-cols-2 gap-1 pt-1 text-[10px] text-slate-400">
                        <div>Khoảng cách: <span id="planetDist" class="text-slate-200">0 AU</span></div>
                        <div>Nhiệt độ: <span id="planetTemp" class="text-amber-300">5,500 °C</span></div>
                    </div>

                    <!-- NÚT ĐỔ BỘ TRÁI ĐẤT (CHỈ HIỆN KHI CHỌN TRÁI ĐẤT) -->
                    <button id="landEarthBtn" onclick="enterEarthMode()" class="hidden w-full mt-2 py-2.5 px-3 bg-gradient-to-r from-emerald-500 to-cyan-500 hover:brightness-110 text-slate-950 font-orbitron font-black text-xs rounded-xl shadow-[0_0_25px_rgba(16,185,129,0.7)] transition animate-bounce">
                        🌍 ĐỔ BỘ XUỐNG TRÁI ĐẤT
                    </button>
                </div>
            </div>
        </header>

        <!-- THANH CHỌN HÀNH TINH -->
        <nav class="absolute bottom-5 left-1/2 -translate-x-1/2 z-20 hud-glass px-3 py-2.5 rounded-2xl flex gap-2 overflow-x-auto max-w-[95vw] custom-scrollbar">
            <button id="btn-sun" onclick="selectPlanet('sun')" class="planet-pill active px-3 py-1.5 text-amber-300 rounded-xl text-xs font-bold whitespace-nowrap">☀️ Mặt Trời</button>
            <button id="btn-mercury" onclick="selectPlanet('mercury')" class="planet-pill px-3 py-1.5 text-slate-300 rounded-xl text-xs font-bold whitespace-nowrap">🪨 Sao Thủy</button>
            <button id="btn-venus" onclick="selectPlanet('venus')" class="planet-pill px-3 py-1.5 text-amber-200 rounded-xl text-xs font-bold whitespace-nowrap">🌕 Sao Kim</button>
            <button id="btn-earth" onclick="selectPlanet('earth')" class="planet-pill px-3 py-1.5 text-emerald-300 border-emerald-500/50 rounded-xl text-xs font-bold whitespace-nowrap">🌍 Trái Đất ✨</button>
            <button id="btn-mars" onclick="selectPlanet('mars')" class="planet-pill px-3 py-1.5 text-rose-400 rounded-xl text-xs font-bold whitespace-nowrap">🔴 Sao Hỏa</button>
            <button id="btn-jupiter" onclick="selectPlanet('jupiter')" class="planet-pill px-3 py-1.5 text-orange-300 rounded-xl text-xs font-bold whitespace-nowrap">🪐 Sao Mộc</button>
            <button id="btn-saturn" onclick="selectPlanet('saturn')" class="planet-pill px-3 py-1.5 text-yellow-200 rounded-xl text-xs font-bold whitespace-nowrap">🪐 Sao Thổ</button>
            <button id="btn-uranus" onclick="selectPlanet('uranus')" class="planet-pill px-3 py-1.5 text-cyan-300 rounded-xl text-xs font-bold whitespace-nowrap">💎 Thiên Vương</button>
            <button id="btn-neptune" onclick="selectPlanet('neptune')" class="planet-pill px-3 py-1.5 text-indigo-400 rounded-xl text-xs font-bold whitespace-nowrap">🔵 Hải Vương</button>
        </nav>
    </div>

    <!-- =====================================================================
         GIAO DIỆN GAME TRONG GAME: NÂNG CẤP SỰ SỐNG TRÁI ĐẤT (EARTH MODE HUD)
    ===================================================================== -->
    <div id="earthUI" class="hidden">
        <!-- THANH TÀI NGUYÊN & QUAY LẠI VŨ TRỤ -->
        <div class="absolute top-4 left-4 right-4 z-20 flex flex-wrap justify-between items-center gap-3">
            <div class="hud-glass px-4 py-2.5 rounded-2xl flex items-center gap-5">
                <div>
                    <div class="text-[10px] text-slate-400">⚡ NĂNG LƯỢNG</div>
                    <div class="font-orbitron font-bold text-base text-amber-300"><span id="resEnergy">50</span> <span class="text-[10px] text-emerald-400">(+<span id="rateEnergy">2</span>/s)</span></div>
                </div>
                <div class="h-7 w-[1px] bg-slate-700"></div>
                <div>
                    <div class="text-[10px] text-slate-400">🧬 TIẾN HÓA SỰ SỐNG</div>
                    <div class="font-orbitron font-bold text-base text-emerald-300"><span id="resEvo">0</span> <span class="text-[10px] text-cyan-400">(+<span id="rateEvo">1</span>/s)</span></div>
                </div>
                <div class="h-7 w-[1px] bg-slate-700 hidden sm:block"></div>
                <div class="hidden sm:block">
                    <div class="text-[10px] text-slate-400">🌍 CẤP ĐỘ VĂN MINH</div>
                    <div id="civStage" class="font-orbitron font-bold text-sm text-cyan-300">Kỷ Nguyên Khởi Nguyên</div>
                </div>
            </div>

            <div class="flex gap-2">
                <button onclick="harvestManual()" class="hud-glass px-4 py-2.5 rounded-2xl font-orbitron font-bold text-xs text-amber-300 hover:border-amber-400 active:scale-95 transition">
                    ✨ Thu Thập (+<span id="clickPower">5</span> ⚡)
                </button>
                <button onclick="exitEarthMode()" class="hud-glass px-4 py-2.5 rounded-2xl font-orbitron font-bold text-xs text-cyan-300 hover:border-cyan-400 hover:bg-cyan-950/60 transition">
                    🌌 Quay lại Vũ Trụ
                </button>
            </div>
        </div>

        <!-- BẢNG NÂNG CẤP CÔNG TRÌNH BÊN DƯỚI -->
        <div class="absolute bottom-5 left-1/2 -translate-x-1/2 z-20 hud-glass p-3.5 rounded-2xl w-[95vw] max-w-4xl">
            <div class="flex justify-between items-center mb-2 px-1">
                <span class="text-[11px] font-orbitron font-bold text-emerald-300 tracking-wider">🏗️ KIẾN TẠO & NÂNG CẤP HỆ SINH THÁI 3D</span>
                <span class="text-[10px] text-slate-400">Kéo chuột để xoay đảo • Cuộn chuột để thu phóng</span>
            </div>
            <div class="grid grid-cols-2 sm:grid-cols-4 gap-2.5">
                <!-- Nâng cấp 1: Rừng Sinh Quyển -->
                <button id="upg-forest" onclick="buyUpgrade('forest')" class="upgrade-card p-3 rounded-xl text-left flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center">
                            <span class="text-sm font-bold text-emerald-300">🌲 Rừng Sinh Quyển</span>
                            <span id="lvl-forest" class="text-[10px] bg-emerald-950 text-emerald-300 px-1.5 py-0.5 rounded font-bold">Lv.1</span>
                        </div>
                        <p class="text-[10px] text-slate-400 mt-1">Trồng thêm rừng cây 3D, tăng +2 🧬/s & +1 ⚡/s</p>
                    </div>
                    <div class="mt-2 pt-1.5 border-t border-slate-700/60 flex justify-between items-center text-[11px] font-bold text-amber-300">
                        <span>Chi phí:</span>
                        <span id="cost-forest">25 ⚡</span>
                    </div>
                </button>

                <!-- Nâng cấp 2: Đô Thị Tương Lai -->
                <button id="upg-city" onclick="buyUpgrade('city')" class="upgrade-card p-3 rounded-xl text-left flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center">
                            <span class="text-sm font-bold text-sky-300">🏙️ Đô Thị Neon</span>
                            <span id="lvl-city" class="text-[10px] bg-sky-950 text-sky-300 px-1.5 py-0.5 rounded font-bold">Lv.0</span>
                        </div>
                        <p class="text-[10px] text-slate-400 mt-1">Mọc tháp chọc trời, tăng +6 ⚡/s & +3 🧬/s</p>
                    </div>
                    <div class="mt-2 pt-1.5 border-t border-slate-700/60 flex justify-between items-center text-[11px] font-bold text-amber-300">
                        <span>Chi phí:</span>
                        <span id="cost-city">80 ⚡</span>
                    </div>
                </button>

                <!-- Nâng cấp 3: Lò Phản Ứng Nhiệt Hạch -->
                <button id="upg-reactor" onclick="buyUpgrade('reactor')" class="upgrade-card p-3 rounded-xl text-left flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center">
                            <span class="text-sm font-bold text-amber-300">⚛️ Lò Nhiệt Hạch</span>
                            <span id="lvl-reactor" class="text-[10px] bg-amber-950 text-amber-300 px-1.5 py-0.5 rounded font-bold">Lv.0</span>
                        </div>
                        <p class="text-[10px] text-slate-400 mt-1">Xây lõi năng lượng phát sáng, tăng +22 ⚡/s</p>
                    </div>
                    <div class="mt-2 pt-1.5 border-t border-slate-700/60 flex justify-between items-center text-[11px] font-bold text-amber-300">
                        <span>Chi phí:</span>
                        <span id="cost-reactor">250 ⚡</span>
                    </div>
                </button>

                <!-- Nâng cấp 4: Trạm Phóng Vệ Tinh -->
                <button id="upg-orbital" onclick="buyUpgrade('orbital')" class="upgrade-card p-3 rounded-xl text-left flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center">
                            <span class="text-sm font-bold text-purple-300">🛰️ Vòng Quỹ Đạo</span>
                            <span id="lvl-orbital" class="text-[10px] bg-purple-950 text-purple-300 px-1.5 py-0.5 rounded font-bold">Lv.0</span>
                        </div>
                        <p class="text-[10px] text-slate-400 mt-1">Phóng vệ tinh bay quanh đảo, x1.5 toàn sản lượng!</p>
                    </div>
                    <div class="mt-2 pt-1.5 border-t border-slate-700/60 flex justify-between items-center text-[11px] font-bold text-emerald-300">
                        <span>Chi phí:</span>
                        <span id="cost-orbital">120 🧬</span>
                    </div>
                </button>
            </div>
        </div>
    </div>

    <div id="canvasContainer" class="fixed inset-0 z-0"></div>

    <script>
        // =========================================================================
        // BỘ NHẠC ĐIỆN ẢNH NÂNG CAO (SYNTH & ARPEGGIO)
        // =========================================================================
        let audioCtx, masterGain, delayNode, feedbackGain, isMusicPlaying = false, seqInterval;

        function startProceduralStrayMusic() {
            const AudioContext = window.AudioContext || window.webkitAudioContext;
            audioCtx = new AudioContext();

            masterGain = audioCtx.createGain();
            masterGain.gain.value = 0.85;

            delayNode = audioCtx.createDelay();
            delayNode.delayTime.value = 0.5;
            feedbackGain = audioCtx.createGain();
            feedbackGain.gain.value = 0.45;

            const delayFilter = audioCtx.createBiquadFilter();
            delayFilter.type = 'lowpass';
            delayFilter.frequency.value = 1500;

            delayNode.connect(delayFilter);
            delayFilter.connect(feedbackGain);
            feedbackGain.connect(delayNode);
            delayFilter.connect(masterGain);
            masterGain.connect(audioCtx.destination);

            const chords = [
                { bass: 110.00, notes: [220.00, 261.63, 329.63, 440.00] },
                { bass: 87.31,  notes: [174.61, 220.00, 261.63, 349.23] },
                { bass: 130.81, notes: [261.63, 329.63, 392.00, 523.25] },
                { bass: 98.00,  notes: [196.00, 246.94, 293.66, 392.00] }
            ];

            let chordIdx = 0;
            let noteIdx = 0;

            function playNote(freq, isBass, duration) {
                if (!audioCtx || audioCtx.state !== 'running') return;
                const now = audioCtx.currentTime;

                const osc1 = audioCtx.createOscillator();
                const osc2 = audioCtx.createOscillator();
                const env = audioCtx.createGain();
                const filter = audioCtx.createBiquadFilter();

                if (isBass) {
                    osc1.type = 'sine';
                    osc2.type = 'triangle';
                    osc1.frequency.setValueAtTime(freq, now);
                    osc2.frequency.setValueAtTime(freq, now);
                    filter.type = 'lowpass';
                    filter.frequency.setValueAtTime(250, now);

                    env.gain.setValueAtTime(0.001, now);
                    env.gain.linearRampToValueAtTime(0.32, now + 0.2);
                    env.gain.exponentialRampToValueAtTime(0.0001, now + duration * 2.5);
                } else {
                    osc1.type = 'sawtooth';
                    osc2.type = 'sine';
                    osc1.frequency.setValueAtTime(freq, now);
                    osc2.frequency.setValueAtTime(freq * 1.004, now);

                    filter.type = 'lowpass';
                    filter.frequency.setValueAtTime(1400, now);
                    filter.frequency.exponentialRampToValueAtTime(400, now + duration);

                    env.gain.setValueAtTime(0.001, now);
                    env.gain.linearRampToValueAtTime(0.12, now + 0.1);
                    env.gain.exponentialRampToValueAtTime(0.0001, now + duration * 1.8);
                }

                osc1.connect(filter);
                osc2.connect(filter);
                filter.connect(env);
                env.connect(masterGain);
                if (!isBass) env.connect(delayNode);

                osc1.start(now);
                osc2.start(now);
                osc1.stop(now + duration * 2);
                osc2.stop(now + duration * 2);
            }

            function triggerStep() {
                if (!isMusicPlaying) return;
                const currentChord = chords[chordIdx];
                if (noteIdx === 0) playNote(currentChord.bass, true, 3.5);
                const n = currentChord.notes[noteIdx % currentChord.notes.length];
                playNote(n, false, 1.4);

                noteIdx++;
                if (noteIdx >= 8) {
                    noteIdx = 0;
                    chordIdx = (chordIdx + 1) % chords.length;
                }
            }

            isMusicPlaying = true;
            triggerStep();
            seqInterval = setInterval(triggerStep, 450);
        }

        function setVolume(v) {
            const val = parseFloat(v);
            document.getElementById('volVal').innerText = Math.round(val * 100) + '%';
            if (masterGain && audioCtx) masterGain.gain.setTargetAtTime(val, audioCtx.currentTime, 0.05);
        }

        function toggleSpaceAudio() {
            if (!audioCtx) { startProceduralStrayMusic(); return; }
            if (audioCtx.state === 'running') {
                audioCtx.suspend();
                isMusicPlaying = false;
                document.getElementById('audioBtn').innerText = '🔇 Muted';
            } else {
                audioCtx.resume();
                isMusicPlaying = true;
                document.getElementById('audioBtn').innerText = '🔊 Music: ON';
            }
        }

        // =========================================================================
        // HỆ THỐNG GAME TRONG GAME: NÂNG CẤP VĂN MINH TRÁI ĐẤT
        // =========================================================================
        let gameMode = 'SPACE'; // 'SPACE' hoặc 'EARTH'
        let spaceGroup, earthGroup;
        let earthSatellites = [];
        let earthClouds = [];

        const civState = {
            energy: 50,
            evo: 0,
            upgrades: {
                forest: { level: 1, baseCost: 25, costType: 'energy', scale: 1.55 },
                city: { level: 0, baseCost: 80, costType: 'energy', scale: 1.65 },
                reactor: { level: 0, baseCost: 250, costType: 'energy', scale: 1.75 },
                orbital: { level: 0, baseCost: 120, costType: 'evo', scale: 1.9 }
            }
        };

        function getUpgradeCost(key) {
            const u = civState.upgrades[key];
            return Math.floor(u.baseCost * Math.pow(u.scale, u.level));
        }

        function getProductionRates() {
            const f = civState.upgrades.forest.level;
            const c = civState.upgrades.city.level;
            const r = civState.upgrades.reactor.level;
            const o = civState.upgrades.orbital.level;

            const mult = Math.pow(1.5, o);
            const energyRate = Math.round((2 + f * 1 + c * 6 + r * 22) * mult);
            const evoRate = Math.round((1 + f * 2 + c * 3) * mult);
            const clickPower = Math.round((5 + f * 2 + c * 5 + r * 10) * mult);
            return { energyRate, evoRate, clickPower };
        }

        function updateCivUI() {
            const rates = getProductionRates();
            document.getElementById('resEnergy').innerText = Math.floor(civState.energy).toLocaleString();
            document.getElementById('resEvo').innerText = Math.floor(civState.evo).toLocaleString();
            document.getElementById('rateEnergy').innerText = rates.energyRate.toLocaleString();
            document.getElementById('rateEvo').innerText = rates.evoRate.toLocaleString();
            document.getElementById('clickPower').innerText = rates.clickPower.toLocaleString();

            // Cập nhật tên Kỷ nguyên
            const totalLvl = civState.upgrades.forest.level + civState.upgrades.city.level + civState.upgrades.reactor.level + civState.upgrades.orbital.level;
            let stage = "Kỷ Nguyên Khởi Nguyên";
            if (totalLvl >= 20) stage = "Văn Minh Liên Sao (Type II)";
            else if (totalLvl >= 12) stage = "Kỷ Nguyên Lượng Tử & Quỹ Đạo";
            else if (totalLvl >= 6) stage = "Đô Thị Tương Lai Sinh Thái";
            else if (totalLvl >= 3) stage = "Kỷ Nguyên Công Nghiệp Xanh";
            document.getElementById('civStage').innerText = stage;

            // Cập nhật từng thẻ nâng cấp
            ['forest', 'city', 'reactor', 'orbital'].forEach(key => {
                const u = civState.upgrades[key];
                const cost = getUpgradeCost(key);
                document.getElementById(`lvl-${key}`).innerText = `Lv.${u.level}`;
                document.getElementById(`cost-${key}`).innerText = `${cost.toLocaleString()} ${u.costType === 'energy' ? '⚡' : '🧬'}`;

                const btn = document.getElementById(`upg-${key}`);
                const canAfford = u.costType === 'energy' ? civState.energy >= cost : civState.evo >= cost;
                btn.disabled = !canAfford;
            });
        }

        function harvestManual() {
            const rates = getProductionRates();
            civState.energy += rates.clickPower;
            civState.evo += Math.max(1, Math.floor(rates.clickPower / 3));
            updateCivUI();
        }

        function buyUpgrade(key) {
            const u = civState.upgrades[key];
            const cost = getUpgradeCost(key);
            if (u.costType === 'energy' && civState.energy >= cost) {
                civState.energy -= cost;
                u.level++;
                spawn3DBuilding(key, u.level);
            } else if (u.costType === 'evo' && civState.evo >= cost) {
                civState.evo -= cost;
                u.level++;
                spawn3DBuilding(key, u.level);
            }
            updateCivUI();
        }

        // Vòng lặp cộng tài nguyên mỗi giây (chạy ngầm kể cả khi đang ở ngoài vũ trụ!)
        setInterval(() => {
            const rates = getProductionRates();
            civState.energy += rates.energyRate;
            civState.evo += rates.evoRate;
            if (gameMode === 'EARTH') updateCivUI();
        }, 1000);

        // =========================================================================
        // KHỞI TẠO THẾ GIỚI 3D TRÊN MẶT ĐẤT (ISLAND BIOME 3D)
        // =========================================================================
        function buildEarthBiome3D() {
            earthGroup = new THREE.Group();
            earthGroup.visible = false;
            scene.add(earthGroup);

            // Ánh sáng riêng cho đảo sự sống
            const hemiLight = new THREE.HemisphereLight(0x7dd3fc, 0x064e3b, 1.3);
            earthGroup.add(hemiLight);

            const dirLight = new THREE.DirectionalLight(0xfef08a, 1.8);
            dirLight.position.set(25, 40, 25);
            earthGroup.add(dirLight);

            // 1. Đế đá nổi khổng lồ
            const baseGeo = new THREE.ConeGeometry(16, 18, 7);
            baseGeo.rotateX(Math.PI);
            const baseMat = new THREE.MeshStandardMaterial({ color: 0x334155, roughness: 0.9, flatShading: true });
            const baseMesh = new THREE.Mesh(baseGeo, baseMat);
            baseMesh.position.y = -9.2;
            earthGroup.add(baseMesh);

            // 2. Mặt cỏ xanh tươi & Hồ nước phát sáng
            const grassGeo = new THREE.CylinderGeometry(16.2, 16, 0.8, 32);
            const grassMat = new THREE.MeshStandardMaterial({ color: 0x10b981, roughness: 0.7 });
            const grassMesh = new THREE.Mesh(grassGeo, grassMat);
            grassMesh.position.y = -0.2;
            earthGroup.add(grassMesh);

            const lakeGeo = new THREE.CylinderGeometry(5.5, 5.5, 0.85, 24);
            const lakeMat = new THREE.MeshStandardMaterial({ color: 0x0ea5e9, emissive: 0x0284c7, emissiveIntensity: 0.4, roughness: 0.2 });
            const lakeMesh = new THREE.Mesh(lakeGeo, lakeMat);
            lakeMesh.position.set(-4, -0.15, 3);
            earthGroup.add(lakeMesh);

            // 3. Mây trắng bay lơ lửng phía trên đảo
            for (let i = 0; i < 6; i++) {
                const cloud = new THREE.Mesh(
                    new THREE.DodecahedronGeometry(1.6 + Math.random(), 1),
                    new THREE.MeshStandardMaterial({ color: 0xffffff, transparent: true, opacity: 0.75, flatShading: true })
                );
                cloud.scale.set(2.2, 0.6, 1.3);
                cloud.position.set((Math.random() - 0.5) * 26, 9 + Math.random() * 4, (Math.random() - 0.5) * 26);
                earthGroup.add(cloud);
                earthClouds.push(cloud);
            }

            // Cây khởi đầu Level 1
            spawn3DBuilding('forest', 1);
        }

        function spawn3DBuilding(type, level) {
            if (!earthGroup) return;

            if (type === 'forest') {
                for (let i = 0; i < 4; i++) {
                    const angle = Math.random() * Math.PI * 2;
                    const rad = 6.5 + Math.random() * 8;
                    const x = Math.cos(angle) * rad;
                    const z = Math.sin(angle) * rad;

                    const tree = new THREE.Group();
                    const trunk = new THREE.Mesh(
                        new THREE.CylinderGeometry(0.18, 0.28, 1.2, 6),
                        new THREE.MeshStandardMaterial({ color: 0x78350f })
                    );
                    trunk.position.y = 0.6;

                    const leaves = new THREE.Mesh(
                        new THREE.ConeGeometry(1.1, 2.6, 7),
                        new THREE.MeshStandardMaterial({ color: level % 2 === 0 ? 0x34d399 : 0x059669, flatShading: true })
                    );
                    leaves.position.y = 2.1;

                    tree.add(trunk, leaves);
                    tree.position.set(x, 0.2, z);
                    const s = 0.8 + Math.random() * 0.6;
                    tree.scale.set(s, s, s);
                    earthGroup.add(tree);
                }
            } else if (type === 'city') {
                for (let i = 0; i < 2; i++) {
                    const angle = Math.random() * Math.PI * 2;
                    const rad = 3 + Math.random() * 9;
                    const h = 3 + Math.random() * 5 + level * 0.4;
                    const tower = new THREE.Mesh(
                        new THREE.BoxGeometry(1.3, h, 1.3),
                        new THREE.MeshStandardMaterial({
                            color: 0x0f172a,
                            emissive: 0x38bdf8,
                            emissiveIntensity: 0.45,
                            roughness: 0.2,
                            metalness: 0.8
                        })
                    );
                    tower.position.set(Math.cos(angle) * rad, h / 2 + 0.1, Math.sin(angle) * rad);
                    earthGroup.add(tower);
                }
            } else if (type === 'reactor') {
                const angle = Math.random() * Math.PI * 2;
                const rad = 4 + Math.random() * 6;
                const coreGroup = new THREE.Group();

                const ring = new THREE.Mesh(
                    new THREE.TorusGeometry(1.2, 0.18, 12, 24),
                    new THREE.MeshStandardMaterial({ color: 0xf59e0b, emissive: 0xf59e0b, emissiveIntensity: 0.8 })
                );
                ring.rotation.x = Math.PI / 2;
                ring.position.y = 1.5;

                const sphere = new THREE.Mesh(
                    new THREE.OctahedronGeometry(0.75, 1),
                    new THREE.MeshStandardMaterial({ color: 0xfef08a, emissive: 0xf97316, emissiveIntensity: 1.2 })
                );
                sphere.position.y = 1.5;

                coreGroup.add(ring, sphere);
                coreGroup.position.set(Math.cos(angle) * rad, 0.2, Math.sin(angle) * rad);
                earthGroup.add(coreGroup);
            } else if (type === 'orbital') {
                const satPivot = new THREE.Group();
                const sat = new THREE.Mesh(
                    new THREE.OctahedronGeometry(0.6, 0),
                    new THREE.MeshStandardMaterial({ color: 0xc084fc, emissive: 0xa855f7, emissiveIntensity: 0.9 })
                );
                sat.position.set(14 + level * 1.2, 7 + (level % 3) * 1.5, 0);
                satPivot.add(sat);
                earthGroup.add(satPivot);
                earthSatellites.push(satPivot);
            }
        }

        function enterEarthMode() {
            gameMode = 'EARTH';
            spaceGroup.visible = false;
            earthGroup.visible = true;

            document.getElementById('spaceUI').classList.add('hidden');
            document.getElementById('earthUI').classList.remove('hidden');

            // Đặt lại góc nhìn camera cho đảo 3D
            currentLookAt.set(0, 1.5, 0);
            targetCamDist = 32;
            camDist = 32;
            camAngleX = 0.45;
            camAngleY = 0.6;
            updateCivUI();
        }

        function exitEarthMode() {
            gameMode = 'SPACE';
            earthGroup.visible = false;
            spaceGroup.visible = true;

            document.getElementById('earthUI').classList.add('hidden');
            document.getElementById('spaceUI').classList.remove('hidden');

            selectPlanet('earth');
        }

        // =========================================================================
        // ENGINE 3D VŨ TRỤ THREE.JS
        // =========================================================================
        let scene, camera, renderer, composer, bloomPass;
        let planets = {};
        let sunUniforms, milkyWayGroup, asteroidBelt, spacetimeGrid, cosmicDust;
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, prevMouse = { x: 0, y: 0 };
        let camDist = 72, targetCamDist = 72;
        let camAngleX = 0.35, camAngleY = 0.25;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0;
        let isFreeFly = false, freeFlyAngle = 0;

        const planetDB = {
            sun: { name: "MẶT TRỜI (SOL)", desc: "Khối cầu plasma sôi sục. Bấm chọn Trái Đất để đổ bộ xuống nền văn minh!", dist: "0 AU", temp: "5,500 °C", zoom: 72 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Bề mặt đá bazan xám bạc lồi lõm hàng triệu hố thiên thạch.", dist: "0.39 AU", temp: "430 °C", zoom: 4.5 },
            venus: { name: "SAO KIM (VENUS)", desc: "Khí quyển siêu đặc cuộn xoáy mây axit sulfuric ánh vàng rực.", dist: "0.72 AU", temp: "462 °C", zoom: 6 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Hành tinh duy nhất có sự sống! Bấm nút bên dưới để đổ bộ & nâng cấp văn minh.", dist: "1.00 AU", temp: "15 °C", zoom: 6.5 },
            mars: { name: "SAO HỎA (MARS)", desc: "Sa mạc oxit sắt đỏ rực với chỏm băng cực và hẻm núi sâu.", dist: "1.52 AU", temp: "-63 °C", zoom: 5.5 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Khí quyển dải băng tầng khí động học & cơn bão Vết Đỏ Lớn.", dist: "5.20 AU", temp: "-110 °C", zoom: 13 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Hệ thống vành đai băng đa tầng phân tách bởi khe Cassini sắc nét.", dist: "9.58 AU", temp: "-140 °C", zoom: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng khí metan xanh ngọc bích với vành đai dọc mờ ảo.", dist: "19.22 AU", temp: "-195 °C", zoom: 10 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Thế giới băng giá xanh thẫm với những luồng gió siêu thanh.", dist: "30.05 AU", temp: "-200 °C", zoom: 10 }
        };

        const sunVertexShader = `
            varying vec3 vNormal;
            varying vec3 vPosition;
            void main() {
                vNormal = normalize(normalMatrix * normal);
                vPosition = position;
                gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
            }
        `;

        const sunFragmentShader = `
            uniform float uTime;
            varying vec3 vNormal;
            varying vec3 vPosition;

            float hash(vec3 p) {
                p = fract(p * 0.3183099 + 0.1);
                p *= 17.0;
                return fract(p.x * p.y * p.z * (p.x + p.y + p.z));
            }

            float noise(vec3 x) {
                vec3 i = floor(x);
                vec3 f = fract(x);
                f = f * f * (3.0 - 2.0 * f);
                return mix(mix(mix(hash(i + vec3(0,0,0)), hash(i + vec3(1,0,0)), f.x),
                               mix(hash(i + vec3(0,1,0)), hash(i + vec3(1,1,0)), f.x), f.y),
                           mix(mix(hash(i + vec3(0,0,1)), hash(i + vec3(1,0,1)), f.x),
                               mix(hash(i + vec3(0,1,1)), hash(i + vec3(1,1,1)), f.x), f.y), f.z);
            }

            float fbm(vec3 p) {
                float v = 0.0;
                float a = 0.5;
                for (int i = 0; i < 5; i++) {
                    v += a * noise(p);
                    p = p * 2.05 + vec3(0.15);
                    a *= 0.5;
                }
                return v;
            }

            void main() {
                vec3 pos = vPosition * 1.4;
                float n1 = fbm(pos + vec3(uTime * 0.25, -uTime * 0.15, uTime * 0.2));
                float n2 = fbm(pos * 2.2 - vec3(uTime * 0.3));
                float plasma = smoothstep(0.25, 0.85, n1 * 0.65 + n2 * 0.45);

                vec3 darkSpot = vec3(0.55, 0.08, 0.01);
                vec3 midOrange = vec3(1.0, 0.38, 0.02);
                vec3 hotYellow = vec3(1.45, 1.2, 0.6);

                vec3 col = mix(darkSpot, midOrange, smoothstep(0.1, 0.55, plasma));
                col = mix(col, hotYellow, smoothstep(0.5, 0.9, plasma));

                float fresnel = pow(1.0 - max(dot(vNormal, vec3(0.0, 0.0, 1.0)), 0.0), 2.5);
                col += vec3(1.2, 0.5, 0.1) * fresnel * 0.8;

                gl_FragColor = vec4(col, 1.0);
            }
        `;

        function createAtmosphereMaterial(colorHex, intensity = 1.1, power = 3.5) {
            return new THREE.ShaderMaterial({
                uniforms: {
                    uColor: { value: new THREE.Color(colorHex) },
                    uIntensity: { value: intensity },
                    uPower: { value: power }
                },
                vertexShader: `
                    varying vec3 vNormal;
                    void main() {
                        vNormal = normalize(normalMatrix * normal);
                        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
                    }
                `,
                fragmentShader: `
                    uniform vec3 uColor;
                    uniform float uIntensity;
                    uniform float uPower;
                    varying vec3 vNormal;
                    void main() {
                        float glow = pow(0.68 - dot(vNormal, vec3(0.0, 0.0, 1.0)), uPower);
                        gl_FragColor = vec4(uColor * uIntensity, max(0.0, glow));
                    }
                `,
                side: THREE.BackSide,
                blending: THREE.AdditiveBlending,
                transparent: true,
                depthWrite: false
            });
        }

        function createFlareTextures() {
            const streakCanvas = document.createElement('canvas');
            streakCanvas.width = 512; streakCanvas.height = 64;
            const sCtx = streakCanvas.getContext('2d');
            const sGrad = sCtx.createLinearGradient(0, 32, 512, 32);
            sGrad.addColorStop(0.0, 'rgba(56, 189, 248, 0)');
            sGrad.addColorStop(0.35, 'rgba(56, 189, 248, 0.45)');
            sGrad.addColorStop(0.5, 'rgba(255, 251, 235, 0.95)');
            sGrad.addColorStop(0.65, 'rgba(56, 189, 248, 0.45)');
            sGrad.addColorStop(1.0, 'rgba(56, 189, 248, 0)');
            sCtx.fillStyle = sGrad;
            sCtx.fillRect(0, 26, 512, 12);

            const orbCanvas = document.createElement('canvas');
            orbCanvas.width = 128; orbCanvas.height = 128;
            const oCtx = orbCanvas.getContext('2d');
            const oGrad = oCtx.createRadialGradient(64, 64, 4, 64, 64, 60);
            oGrad.addColorStop(0, 'rgba(255, 255, 255, 0.9)');
            oGrad.addColorStop(0.4, 'rgba(56, 189, 248, 0.35)');
            oGrad.addColorStop(1, 'rgba(0, 0, 0, 0)');
            oCtx.fillStyle = oGrad;
            oCtx.fillRect(0, 0, 128, 128);

            return {
                streak: new THREE.CanvasTexture(streakCanvas),
                orb: new THREE.CanvasTexture(orbCanvas)
            };
        }

        function generatePlanetMaps(type) {
            const w = 2048, h = 1024;
            const diffCanvas = document.createElement('canvas');
            const bumpCanvas = document.createElement('canvas');
            diffCanvas.width = bumpCanvas.width = w;
            diffCanvas.height = bumpCanvas.height = h;
            const ctx = diffCanvas.getContext('2d');
            const bCtx = bumpCanvas.getContext('2d');

            bCtx.fillStyle = '#808080';
            bCtx.fillRect(0, 0, w, h);

            if (type === 'mercury') {
                ctx.fillStyle = '#64748b'; ctx.fillRect(0, 0, w, h);
                for (let i = 0; i < 1200; i++) {
                    const x = Math.random() * w, y = Math.random() * h, r = Math.random() * 22 + 2;
                    ctx.fillStyle = i % 2 === 0 ? 'rgba(30,41,59,0.45)' : 'rgba(148,163,184,0.35)';
                    ctx.beginPath(); ctx.arc(x, y, r, 0, Math.PI * 2); ctx.fill();
                    bCtx.fillStyle = i % 2 === 0 ? 'rgba(20,20,20,0.6)' : 'rgba(230,230,230,0.5)';
                    bCtx.beginPath(); bCtx.arc(x, y, r, 0, Math.PI * 2); bCtx.fill();
                }
            } else if (type === 'venus') {
                const grad = ctx.createLinearGradient(0, 0, 0, h);
                grad.addColorStop(0, '#92400e'); grad.addColorStop(0.5, '#f59e0b'); grad.addColorStop(1, '#78350f');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, w, h);
                for (let i = 0; i < 200; i++) {
                    ctx.fillStyle = `rgba(254, 243, 199, ${Math.random() * 0.18})`;
                    ctx.beginPath();
                    ctx.ellipse(Math.random() * w, Math.random() * h, Math.random() * 350 + 80, Math.random() * 25 + 6, 0, 0, Math.PI * 2);
                    ctx.fill();
                }
            } else if (type === 'earth') {
                const seaGrad = ctx.createLinearGradient(0, 0, 0, h);
                seaGrad.addColorStop(0, '#e0f2fe');
                seaGrad.addColorStop(0.12, '#0369a1');
                seaGrad.addColorStop(0.5, '#0284c7');
                seaGrad.addColorStop(0.88, '#0369a1');
                seaGrad.addColorStop(1, '#e0f2fe');
                ctx.fillStyle = seaGrad; ctx.fillRect(0, 0, w, h);

                for (let i = 0; i < 65; i++) {
                    const cx = Math.random() * w, cy = 150 + Math.random() * (h - 300);
                    const rx = Math.random() * 190 + 50, ry = Math.random() * 110 + 30;
                    const rot = Math.random() * Math.PI;
                    ctx.fillStyle = i % 3 === 0 ? '#a16207' : '#15803d';
                    ctx.beginPath(); ctx.ellipse(cx, cy, rx, ry, rot, 0, Math.PI * 2); ctx.fill();

                    ctx.fillStyle = '#fef08a';
                    for (let k = 0; k < 14; k++) {
                        ctx.fillRect(cx + (Math.random() - 0.5) * rx, cy + (Math.random() - 0.5) * ry, 2.5, 2.5);
                    }

                    bCtx.fillStyle = 'rgba(240, 240, 240, 0.75)';
                    bCtx.beginPath(); bCtx.ellipse(cx, cy, rx * 0.85, ry * 0.85, rot, 0, Math.PI * 2); bCtx.fill();
                }
            } else if (type === 'clouds') {
                ctx.clearRect(0, 0, w, h);
                for (let i = 0; i < 240; i++) {
                    ctx.fillStyle = `rgba(255, 255, 255, ${Math.random() * 0.45 + 0.15})`;
                    ctx.beginPath();
                    ctx.ellipse(Math.random() * w, Math.random() * h, Math.random() * 220 + 40, Math.random() * 35 + 8, (Math.random() - 0.5) * 0.4, 0, Math.PI * 2);
                    ctx.fill();
                }
            } else if (type === 'mars') {
                ctx.fillStyle = '#9a3412'; ctx.fillRect(0, 0, w, h);
                for (let i = 0; i < 500; i++) {
                    const x = Math.random() * w, y = Math.random() * h, r = Math.random() * 45 + 5;
                    ctx.fillStyle = i % 2 === 0 ? 'rgba(67, 20, 7, 0.4)' : 'rgba(234, 88, 12, 0.35)';
                    ctx.beginPath(); ctx.arc(x, y, r, 0, Math.PI * 2); ctx.fill();
                    bCtx.fillStyle = i % 2 === 0 ? 'rgba(40,40,40,0.5)' : 'rgba(210,210,210,0.5)';
                    bCtx.beginPath(); bCtx.arc(x, y, r, 0, Math.PI * 2); bCtx.fill();
                }
                ctx.fillStyle = '#f8fafc';
                ctx.fillRect(0, 0, w, 45); ctx.fillRect(0, h - 45, w, 45);
            } else if (type === 'jupiter') {
                const palette = ['#451a03', '#92400e', '#d97706', '#fef3c7', '#b45309', '#78350f', '#fde68a'];
                let y = 0;
                while (y < h) {
                    const bandH = Math.random() * 28 + 8;
                    ctx.fillStyle = palette[Math.floor(Math.random() * palette.length)];
                    ctx.fillRect(0, y, w, bandH);
                    for (let x = 0; x < w; x += 24) {
                        ctx.beginPath();
                        ctx.arc(x, y, Math.random() * 12 + 4, 0, Math.PI * 2);
                        ctx.fill();
                    }
                    y += bandH;
                }
                const grsGrad = ctx.createRadialGradient(1350, 640, 10, 1350, 640, 125);
                grsGrad.addColorStop(0, '#7f1d1d'); grsGrad.addColorStop(0.6, '#dc2626'); grsGrad.addColorStop(1, 'rgba(217,119,6,0)');
                ctx.fillStyle = grsGrad;
                ctx.beginPath(); ctx.ellipse(1350, 640, 130, 80, 0, 0, Math.PI * 2); ctx.fill();
            } else if (type === 'saturn') {
                const palette = ['#713f12', '#a16207', '#eab308', '#fef08a', '#ca8a04', '#fde047'];
                let y = 0;
                while (y < h) {
                    const bandH = Math.random() * 22 + 6;
                    ctx.fillStyle = palette[Math.floor(Math.random() * palette.length)];
                    ctx.fillRect(0, y, w, bandH);
                    y += bandH;
                }
            } else if (type === 'uranus') {
                const grad = ctx.createLinearGradient(0, 0, 0, h);
                grad.addColorStop(0, '#cffafe'); grad.addColorStop(0.5, '#06b6d4'); grad.addColorStop(1, '#cffafe');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, w, h);
            } else if (type === 'neptune') {
                const grad = ctx.createLinearGradient(0, 0, 0, h);
                grad.addColorStop(0, '#1e1b4b'); grad.addColorStop(0.5, '#1d4ed8'); grad.addColorStop(1, '#1e1b4b');
                ctx.fillStyle = grad; ctx.fillRect(0, 0, w, h);
                ctx.fillStyle = 'rgba(147, 197, 253, 0.35)';
                for (let i = 0; i < 40; i++) ctx.fillRect(Math.random() * w, Math.random() * h, Math.random() * 260 + 40, 6);
            }

            return {
                map: new THREE.CanvasTexture(diffCanvas),
                bumpMap: new THREE.CanvasTexture(bumpCanvas)
            };
        }

        function createSaturnRingTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 1024; canvas.height = 64;
            const ctx = canvas.getContext('2d');
            const grad = ctx.createLinearGradient(0, 0, 1024, 0);
            grad.addColorStop(0.0, 'rgba(254, 240, 138, 0.0)');
            grad.addColorStop(0.15, 'rgba(180, 150, 90, 0.35)');
            grad.addColorStop(0.35, 'rgba(254, 243, 199, 0.92)');
            grad.addColorStop(0.58, 'rgba(234, 179, 8, 0.85)');
            grad.addColorStop(0.63, 'rgba(0, 0, 0, 0.05)');
            grad.addColorStop(0.68, 'rgba(253, 224, 71, 0.8)');
            grad.addColorStop(0.92, 'rgba(202, 138, 4, 0.45)');
            grad.addColorStop(1.0, 'rgba(254, 240, 138, 0.0)');
            ctx.fillStyle = grad;
            ctx.fillRect(0, 0, 1024, 64);
            return new THREE.CanvasTexture(canvas);
        }

        function createMilkyWayAndGalaxies() {
            milkyWayGroup = new THREE.Group();
            milkyWayGroup.rotation.z = Math.PI / 6;

            const starCount = 60000;
            const starGeo = new THREE.BufferGeometry();
            const starPos = new Float32Array(starCount * 3);
            const starCol = new Float32Array(starCount * 3);

            const starColors = [
                new THREE.Color(0x38bdf8),
                new THREE.Color(0xa855f7),
                new THREE.Color(0xfde047),
                new THREE.Color(0xf472b6),
                new THREE.Color(0xffffff),
                new THREE.Color(0x60a5fa)
            ];

            for (let i = 0; i < starCount; i++) {
                const angle = Math.random() * Math.PI * 2;
                const radius = 800 + Math.random() * 2200;
                const spread = (Math.random() - 0.5) * (Math.random() - 0.5) * 850;

                starPos[i * 3] = Math.cos(angle) * radius;
                starPos[i * 3 + 1] = spread;
                starPos[i * 3 + 2] = Math.sin(angle) * radius;

                const c = starColors[Math.floor(Math.random() * starColors.length)];
                const brightness = 0.5 + Math.random() * 0.5;
                starCol[i * 3] = c.r * brightness;
                starCol[i * 3 + 1] = c.g * brightness;
                starCol[i * 3 + 2] = c.b * brightness;
            }

            starGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            starGeo.setAttribute('color', new THREE.BufferAttribute(starCol, 3));
            
            const starMat = new THREE.PointsMaterial({
                size: 2.8,
                vertexColors: true,
                transparent: true,
                opacity: 0.9,
                blending: THREE.AdditiveBlending,
                depthWrite: false
            });
            milkyWayGroup.add(new THREE.Points(starGeo, starMat));

            const dustCount = 8000;
            const dustGeo = new THREE.BufferGeometry();
            const dustPos = new Float32Array(dustCount * 3);
            for (let i = 0; i < dustCount; i++) {
                dustPos[i * 3] = (Math.random() - 0.5) * 700;
                dustPos[i * 3 + 1] = (Math.random() - 0.5) * 300;
                dustPos[i * 3 + 2] = (Math.random() - 0.5) * 700;
            }
            dustGeo.setAttribute('position', new THREE.BufferAttribute(dustPos, 3));
            cosmicDust = new THREE.Points(dustGeo, new THREE.PointsMaterial({ color: 0xbae6fd, size: 1.2, transparent: true, opacity: 0.8, blending: THREE.AdditiveBlending }));
            spaceGroup.add(cosmicDust);

            spaceGroup.add(milkyWayGroup);
        }

        function createAsteroidBelt() {
            const count = 2200;
            const geo = new THREE.DodecahedronGeometry(0.11, 0);
            const mat = new THREE.MeshStandardMaterial({ color: 0x94a3b8, roughness: 0.85, metalness: 0.2 });
            asteroidBelt = new THREE.InstancedMesh(geo, mat, count);

            const dummy = new THREE.Object3D();
            for (let i = 0; i < count; i++) {
                const angle = Math.random() * Math.PI * 2;
                const radius = 30.2 + Math.random() * 3.6;
                const y = (Math.random() - 0.5) * 1.1;

                dummy.position.set(Math.cos(angle) * radius, y, Math.sin(angle) * radius);
                dummy.rotation.set(Math.random() * Math.PI, Math.random() * Math.PI, Math.random() * Math.PI);
                const scale = 0.4 + Math.random() * 1.4;
                dummy.scale.set(scale, scale * 0.7, scale);
                dummy.updateMatrix();
                asteroidBelt.setMatrixAt(i, dummy.matrix);
            }
            spaceGroup.add(asteroidBelt);
        }

        function createSpacetimeGrid() {
            const size = 160;
            const divisions = 48;
            const geo = new THREE.PlaneGeometry(size, size, divisions, divisions);
            geo.rotateX(-Math.PI / 2);

            const pos = geo.attributes.position;
            for (let i = 0; i < pos.count; i++) {
                const x = pos.getX(i);
                const z = pos.getZ(i);
                const dist = Math.sqrt(x * x + z * z);
                const dip = -18.0 / (Math.sqrt(dist * 0.35 + 1.2));
                pos.setY(i, dip - 3.5);
            }
            geo.computeVertexNormals();

            const mat = new THREE.MeshBasicMaterial({
                color: 0x0ea5e9,
                wireframe: true,
                transparent: true,
                opacity: 0.16
            });
            spacetimeGrid = new THREE.Mesh(geo, mat);
            spacetimeGrid.visible = false;
            spaceGroup.add(spacetimeGrid);
        }

        function init3D() {
            const container = document.getElementById('canvasContainer');
            scene = new THREE.Scene();

            spaceGroup = new THREE.Group();
            scene.add(spaceGroup);

            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 10000);

            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.toneMapping = THREE.ReinhardToneMapping;
            renderer.toneMappingExposure = 1.25;
            container.appendChild(renderer.domElement);

            const renderScene = new THREE.RenderPass(scene, camera);
            bloomPass = new THREE.UnrealBloomPass(new THREE.Vector2(window.innerWidth, window.innerHeight), 1.4, 0.75, 0.25);
            composer = new THREE.EffectComposer(renderer);
            composer.addPass(renderScene);
            composer.addPass(bloomPass);

            const sunLight = new THREE.PointLight(0xfff5e4, 3.8, 2000, 0.25);
            spaceGroup.add(sunLight);
            spaceGroup.add(new THREE.AmbientLight(0x0f172a, 0.45));

            const flareTex = createFlareTextures();
            const lensflare = new THREE.Lensflare();
            lensflare.addElement(new THREE.LensflareElement(flareTex.streak, 720, 0, new THREE.Color(0x7dd3fc)));
            lensflare.addElement(new THREE.LensflareElement(flareTex.orb, 150, 0, new THREE.Color(0xfef08a)));
            lensflare.addElement(new THREE.LensflareElement(flareTex.orb, 65, 0.35, new THREE.Color(0x38bdf8)));
            lensflare.addElement(new THREE.LensflareElement(flareTex.orb, 95, 0.6, new THREE.Color(0x818cf8)));
            lensflare.addElement(new THREE.LensflareElement(flareTex.orb, 50, 0.85, new THREE.Color(0xf472b6)));
            sunLight.add(lensflare);

            createMilkyWayAndGalaxies();
            createAsteroidBelt();
            createSpacetimeGrid();
            buildEarthBiome3D();

            sunUniforms = { uTime: { value: 0 } };
            const sunMat = new THREE.ShaderMaterial({
                uniforms: sunUniforms,
                vertexShader: sunVertexShader,
                fragmentShader: sunFragmentShader
            });
            const sun = new THREE.Mesh(new THREE.SphereGeometry(5.2, 64, 64), sunMat);
            const corona = new THREE.Mesh(new THREE.SphereGeometry(6.8, 64, 64), createAtmosphereMaterial(0xff8c00, 1.4, 3.0));
            sun.add(corona);
            spaceGroup.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0 };

            const planetConfigs = [
                { id: 'mercury', size: 0.55, dist: 10, speed: 0.022, tilt: 0.03, atmos: null },
                { id: 'venus', size: 0.95, dist: 14.5, speed: 0.016, tilt: 3.09, atmos: 0xfbbf24 },
                { id: 'earth', size: 1.18, dist: 20, speed: 0.012, tilt: 0.41, atmos: 0x38bdf8, clouds: true, moon: true },
                { id: 'mars', size: 0.78, dist: 26, speed: 0.009, tilt: 0.44, atmos: 0xf97316 },
                { id: 'jupiter', size: 2.8, dist: 38, speed: 0.006, tilt: 0.05, atmos: 0xfdba74 },
                { id: 'saturn', size: 2.3, dist: 50, speed: 0.004, tilt: 0.47, atmos: 0xfef08a, rings: true },
                { id: 'uranus', size: 1.55, dist: 60, speed: 0.003, tilt: 1.71, atmos: 0x22d3ee },
                { id: 'neptune', size: 1.5, dist: 69, speed: 0.002, tilt: 0.49, atmos: 0x3b82f6 }
            ];

            planetConfigs.forEach(cfg => {
                const orbitGeo = new THREE.RingGeometry(cfg.dist - 0.06, cfg.dist + 0.06, 220);
                const orbitMat = new THREE.MeshBasicMaterial({ color: cfg.atmos || 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.22, blending: THREE.AdditiveBlending });
                const orbitRing = new THREE.Mesh(orbitGeo, orbitMat);
                orbitRing.rotation.x = Math.PI / 2;
                spaceGroup.add(orbitRing);

                const maps = generatePlanetMaps(cfg.id);
                const pMat = new THREE.MeshStandardMaterial({
                    map: maps.map,
                    bumpMap: maps.bumpMap,
                    bumpScale: 0.045,
                    roughness: 0.72,
                    metalness: 0.08
                });

                const mesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size, 64, 64), pMat);
                mesh.rotation.z = cfg.tilt;
                spaceGroup.add(mesh);

                if (cfg.atmos) {
                    const atmosMesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size * 1.12, 48, 48), createAtmosphereMaterial(cfg.atmos, 1.2, 3.6));
                    mesh.add(atmosMesh);
                }

                if (cfg.clouds) {
                    const cloudMaps = generatePlanetMaps('clouds');
                    const cloudMat = new THREE.MeshStandardMaterial({
                        map: cloudMaps.map,
                        transparent: true,
                        opacity: 0.88,
                        depthWrite: false
                    });
                    const cloudMesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size * 1.022, 64, 64), cloudMat);
                    mesh.add(cloudMesh);
                    mesh.userData.cloudMesh = cloudMesh;
                }

                if (cfg.moon) {
                    const moonPivot = new THREE.Group();
                    const moonMesh = new THREE.Mesh(
                        new THREE.SphereGeometry(0.28, 32, 32),
                        new THREE.MeshStandardMaterial({ map: generatePlanetMaps('mercury').map, roughness: 0.9 })
                    );
                    moonMesh.position.x = 2.3;
                    moonPivot.add(moonMesh);
                    mesh.add(moonPivot);
                    mesh.userData.moonPivot = moonPivot;
                }

                if (cfg.rings) {
                    const rGeo = new THREE.RingGeometry(cfg.size * 1.35, cfg.size * 2.45, 128);
                    const posAttr = rGeo.attributes.position;
                    const uvAttr = rGeo.attributes.uv;
                    const v3 = new THREE.Vector3();
                    for (let i = 0; i < posAttr.count; i++) {
                        v3.fromBufferAttribute(posAttr, i);
                        const u = (v3.length() - cfg.size * 1.35) / (cfg.size * 1.1);
                        uvAttr.setXY(i, u, 0.5);
                    }
                    const rMat = new THREE.MeshStandardMaterial({
                        map: createSaturnRingTexture(),
                        side: THREE.DoubleSide,
                        transparent: true,
                        opacity: 0.95,
                        roughness: 0.5
                    });
                    const ringMesh = new THREE.Mesh(rGeo, rMat);
                    ringMesh.rotation.x = Math.PI / 2;
                    mesh.add(ringMesh);
                }

                planets[cfg.id] = { mesh, distance: cfg.dist, speed: cfg.speed, angle: Math.random() * Math.PI * 2 };
            });

            window.addEventListener('mousedown', e => { 
                if (e.target.tagName === 'CANVAS') {
                    isDragging = true; 
                    prevMouse = { x: e.clientX, y: e.clientY }; 
                }
            });
            window.addEventListener('mousemove', e => {
                if (!isDragging) return;
                camAngleY -= (e.clientX - prevMouse.x) * 0.005;
                camAngleX = Math.max(-1.45, Math.min(1.45, camAngleX - (e.clientY - prevMouse.y) * 0.005));
                prevMouse = { x: e.clientX, y: e.clientY };
            });
            window.addEventListener('mouseup', () => isDragging = false);
            window.addEventListener('wheel', e => {
                targetCamDist = Math.max(2.5, Math.min(450, targetCamDist + e.deltaY * 0.06));
            });
            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
                composer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function startExperience() {
            document.getElementById('startOverlay').style.display = 'none';
            init3D();
            startProceduralStrayMusic();
        }

        function toggleFreeFly() {
            isFreeFly = !isFreeFly;
            const btn = document.getElementById('flyBtn');
            btn.innerText = isFreeFly ? '🛸 Free-Fly: ON' : '🛸 Free-Fly: OFF';
            btn.className = isFreeFly
                ? 'py-1.5 px-2 bg-cyan-950/80 border border-cyan-500/60 text-cyan-200 rounded-xl font-bold transition'
                : 'py-1.5 px-2 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-xl font-bold hover:border-cyan-400 transition';
        }

        function toggleSpacetimeGrid() {
            if (!spacetimeGrid) return;
            spacetimeGrid.visible = !spacetimeGrid.visible;
            const btn = document.getElementById('gridBtn');
            btn.innerText = spacetimeGrid.visible ? '🕸️ Lưới KG: ON' : '🕸️ Lưới KG: OFF';
            btn.className = spacetimeGrid.visible
                ? 'py-1.5 px-2 bg-cyan-950/80 border border-cyan-500/60 text-cyan-200 rounded-xl font-bold transition'
                : 'py-1.5 px-2 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-xl font-bold hover:border-cyan-400 transition';
        }

        function setSpeed(v) {
            timeSpeed = parseFloat(v);
            document.getElementById('speedVal').innerText = timeSpeed.toFixed(1) + 'x';
        }

        function toggleHud() {
            isHudVisible = !isHudVisible;
            const body = document.getElementById('hudBody');
            body.style.maxHeight = isHudVisible ? '380px' : '0px';
            body.style.opacity = isHudVisible ? '1' : '0';
            document.getElementById('hudToggleBtn').innerText = isHudVisible ? '👁️ Ẩn' : '👁️ Hiện';
        }

        function selectPlanet(id) {
            focusedObject = id;
            isFreeFly = false;
            document.getElementById('flyBtn').innerText = '🛸 Free-Fly: OFF';
            document.getElementById('flyBtn').className = 'py-1.5 px-2 bg-slate-900/90 border border-slate-700 text-slate-300 rounded-xl font-bold hover:border-cyan-400 transition';

            document.querySelectorAll('.planet-pill').forEach(b => b.classList.remove('active'));
            const btn = document.getElementById(`btn-${id}`);
            if (btn) btn.classList.add('active');

            const info = planetDB[id];
            if (info) {
                document.getElementById('planetTitle').innerText = info.name;
                document.getElementById('planetDesc').innerText = info.desc;
                document.getElementById('planetDist').innerText = info.dist;
                document.getElementById('planetTemp').innerText = info.temp;
                targetCamDist = info.zoom;
            }

            // Hiện nút Đổ bộ nếu chọn Trái Đất
            const landBtn = document.getElementById('landEarthBtn');
            if (id === 'earth') {
                landBtn.classList.remove('hidden');
            } else {
                landBtn.classList.add('hidden');
            }
        }

        function animate() {
            requestAnimationFrame(animate);

            if (gameMode === 'EARTH') {
                // Xoay nhẹ vệ tinh và mây trên đảo Trái Đất
                earthSatellites.forEach((sat, idx) => {
                    sat.rotation.y += 0.015 + idx * 0.004;
                });
                earthClouds.forEach((c, idx) => {
                    c.position.x += Math.sin(Date.now() * 0.0005 + idx) * 0.01;
                });

                camDist += (targetCamDist - camDist) * 0.08;
                camera.position.x = currentLookAt.x + camDist * Math.sin(camAngleY) * Math.cos(camAngleX);
                camera.position.y = currentLookAt.y + camDist * Math.sin(camAngleX);
                camera.position.z = currentLookAt.z + camDist * Math.cos(camAngleY) * Math.cos(camAngleX);
                camera.lookAt(currentLookAt);

                composer.render();
                return;
            }

            // CHẾ ĐỘ VŨ TRỤ (SPACE MODE)
            if (sunUniforms) sunUniforms.uTime.value += 0.015 * timeSpeed;
            if (milkyWayGroup) milkyWayGroup.rotation.y += 0.00015 * timeSpeed;
            if (cosmicDust) cosmicDust.rotation.y += 0.00035 * timeSpeed;
            if (asteroidBelt && asteroidBelt.visible) asteroidBelt.rotation.y += 0.0018 * timeSpeed;

            if (isFreeFly) {
                freeFlyAngle += 0.003 * timeSpeed;
                currentLookAt.set(0, 0, 0);
                camDist = 85 + Math.sin(freeFlyAngle * 0.5) * 25;
                camera.position.x = Math.cos(freeFlyAngle) * camDist;
                camera.position.y = 25 + Math.sin(freeFlyAngle) * 15;
                camera.position.z = Math.sin(freeFlyAngle) * camDist;
                camera.lookAt(currentLookAt);
                composer.render();
                return;
            }

            Object.keys(planets).forEach(id => {
                const p = planets[id];
                if (p.speed > 0) {
                    p.angle += p.speed * timeSpeed;
                    p.mesh.position.x = Math.cos(p.angle) * p.distance;
                    p.mesh.position.z = Math.sin(p.angle) * p.distance;
                    p.mesh.rotation.y += 0.012 * timeSpeed;

                    if (p.mesh.userData.cloudMesh) p.mesh.userData.cloudMesh.rotation.y += 0.006 * timeSpeed;
                    if (p.mesh.userData.moonPivot) p.mesh.userData.moonPivot.rotation.y += 0.03 * timeSpeed;
                }
            });

            const targetPos = new THREE.Vector3();
            if (planets[focusedObject]) planets[focusedObject].mesh.getWorldPosition(targetPos);
            currentLookAt.lerp(targetPos, 0.08);
            camDist += (targetCamDist - camDist) * 0.08;

            camera.position.x = currentLookAt.x + camDist * Math.sin(camAngleY) * Math.cos(camAngleX);
            camera.position.y = currentLookAt.y + camDist * Math.sin(camAngleX);
            camera.position.z = currentLookAt.z + camDist * Math.cos(camAngleY) * Math.cos(camAngleX);
            camera.lookAt(currentLookAt);

            composer.render();
        }
    </script>
</body>
</html>
