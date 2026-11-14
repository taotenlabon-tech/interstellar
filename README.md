
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA Eyes 3D - Cosmic & Interplanetary Tycoon Edition</title>
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
                Hệ Mặt Trời 3D & Đảo Siêu To Khổng Lồ!
            </p>
        </div>
        <button onclick="startExperience()" class="px-9 py-4 bg-gradient-to-r from-cyan-500 via-emerald-500 to-indigo-500 hover:brightness-110 font-orbitron font-bold text-xs sm:text-sm rounded-full text-white shadow-[0_0_60px_rgba(56,189,248,0.8)] transition active:scale-95">
            ⚡ KHỞI ĐỘNG TRẢI NGHIỆM
        </button>
    </div>

    <!-- GIAO DIỆN VŨ TRỤ -->
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
                        Chọn hành tinh để quan sát hoặc chọn Trái Đất để đổ bộ!
                    </p>
                    <div class="grid grid-cols-2 gap-1 pt-1 text-[10px] text-slate-400">
                        <div>Khoảng cách: <span id="planetDist" class="text-slate-200">0 AU</span></div>
                        <div>Nhiệt độ: <span id="planetTemp" class="text-amber-300">5,500 °C</span></div>
                    </div>

                    <button id="landEarthBtn" onclick="enterEarthMode()" class="hidden w-full mt-2 py-2.5 px-3 bg-gradient-to-r from-emerald-500 to-cyan-500 hover:brightness-110 text-slate-950 font-orbitron font-black text-xs rounded-xl shadow-[0_0_25px_rgba(16,185,129,0.7)] transition animate-bounce">
                        🌍 ĐỔ BỘ XUỐNG TRÁI ĐẤT
                    </button>
                </div>
            </div>
        </header>

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

    <!-- GIAO DIỆN TRÁI ĐẤT -->
    <div id="earthUI" class="hidden">
        <div class="absolute top-4 left-4 right-4 z-20 flex flex-wrap justify-between items-center gap-3">
            <div class="hud-glass px-4 py-2.5 rounded-2xl flex items-center gap-4 flex-wrap">
                <div>
                    <div class="text-[10px] text-slate-400">⚡ NĂNG LƯỢNG</div>
                    <div class="font-orbitron font-bold text-sm text-amber-300"><span id="resEnergy">50</span> <span class="text-[9px] text-emerald-400">(+<span id="rateEnergy">2</span>/s)</span></div>
                </div>
                <div class="h-7 w-[1px] bg-slate-700"></div>
                <div>
                    <div class="text-[10px] text-slate-400">🧬 TIẾN HÓA</div>
                    <div class="font-orbitron font-bold text-sm text-emerald-300"><span id="resEvo">0</span> <span class="text-[9px] text-cyan-400">(+<span id="rateEvo">1</span>/s)</span></div>
                </div>
                <div class="h-7 w-[1px] bg-slate-700"></div>
                <div>
                    <div class="text-[10px] text-slate-400">🪙 HELIUM-3</div>
                    <div class="font-orbitron font-bold text-sm text-purple-300"><span id="resMoon">0</span></div>
                </div>
                <div class="h-7 w-[1px] bg-slate-700 hidden sm:block"></div>
                <div class="hidden sm:block">
                    <div class="text-[10px] text-slate-400">🌍 VĂN MINH</div>
                    <div id="civStage" class="font-orbitron font-bold text-xs text-cyan-300">Khởi Nguyên</div>
                </div>
            </div>

            <div class="flex gap-2">
                <button onclick="harvestManual()" class="hud-glass px-3.5 py-2.5 rounded-2xl font-orbitron font-bold text-xs text-amber-300 hover:border-amber-400 active:scale-95 transition">
                    ✨ Thu Thập (+<span id="clickPower">5</span>)
                </button>
                <button id="launchMoonBtn" onclick="enterMoonMode()" class="hidden hud-glass px-3.5 py-2.5 rounded-2xl font-orbitron font-bold text-xs text-purple-300 border-purple-500/60 hover:bg-purple-950/50 active:scale-95 transition animate-pulse">
                    🚀 Thám Hiểm Mặt Trăng
                </button>
                <button onclick="exitEarthMode()" class="hud-glass px-3.5 py-2.5 rounded-2xl font-orbitron font-bold text-xs text-cyan-300 hover:border-cyan-400 transition">
                    🌌 Về Vũ Trụ
                </button>
            </div>
        </div>

        <div class="absolute bottom-5 left-1/2 -translate-x-1/2 z-20 hud-glass p-3.5 rounded-2xl w-[95vw] max-w-4xl">
            <div class="flex justify-between items-center mb-2 px-1">
                <span class="text-[11px] font-orbitron font-bold text-emerald-300 tracking-wider">🏗️ HỆ SINH THÁI TRÁI ĐẤT KHỔNG LỒ</span>
            </div>
            <div class="grid grid-cols-2 sm:grid-cols-4 gap-2.5">
                <button id="upg-forest" onclick="buyUpgrade('forest')" class="upgrade-card p-3 rounded-xl text-left flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center"><span class="text-xs font-bold text-emerald-300">🌲 Rừng Xanh</span><span id="lvl-forest" class="text-[10px] bg-emerald-950 text-emerald-300 px-1.5 py-0.5 rounded font-bold">Lv.1</span></div>
                        <p class="text-[10px] text-slate-400 mt-1">Trồng rừng lớn, +2 🧬/s</p>
                    </div>
                    <div class="mt-2 pt-1.5 border-t border-slate-700/60 flex justify-between items-center text-[11px] font-bold text-amber-300"><span>Chi phí:</span><span id="cost-forest">25 ⚡</span></div>
                </button>

                <button id="upg-city" onclick="buyUpgrade('city')" class="upgrade-card p-3 rounded-xl text-left flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center"><span class="text-xs font-bold text-sky-300">🏙️ Đô Thị Neon</span><span id="lvl-city" class="text-[10px] bg-sky-950 text-sky-300 px-1.5 py-0.5 rounded font-bold">Lv.0</span></div>
                        <p class="text-[10px] text-slate-400 mt-1">Tháp chọc trời, +6 ⚡/s</p>
                    </div>
                    <div class="mt-2 pt-1.5 border-t border-slate-700/60 flex justify-between items-center text-[11px] font-bold text-amber-300"><span>Chi phí:</span><span id="cost-city">80 ⚡</span></div>
                </button>

                <button id="upg-reactor" onclick="buyUpgrade('reactor')" class="upgrade-card p-3 rounded-xl text-left flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center"><span class="text-xs font-bold text-amber-300">⚛️ Lò Nhiệt Hạch</span><span id="lvl-reactor" class="text-[10px] bg-amber-950 text-amber-300 px-1.5 py-0.5 rounded font-bold">Lv.0</span></div>
                        <p class="text-[10px] text-slate-400 mt-1">Lõi năng lượng, +22 ⚡/s</p>
                    </div>
                    <div class="mt-2 pt-1.5 border-t border-slate-700/60 flex justify-between items-center text-[11px] font-bold text-amber-300"><span>Chi phí:</span><span id="cost-reactor">250 ⚡</span></div>
                </button>

                <button id="upg-orbital" onclick="buyUpgrade('orbital')" class="upgrade-card p-3 rounded-xl text-left flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center"><span class="text-xs font-bold text-purple-300">🛰️ Trạm Quỹ Đạo</span><span id="lvl-orbital" class="text-[10px] bg-purple-950 text-purple-300 px-1.5 py-0.5 rounded font-bold">Lv.0</span></div>
                        <p class="text-[10px] text-slate-400 mt-1">Vệ tinh, x1.5 toàn sản lượng!</p>
                    </div>
                    <div class="mt-2 pt-1.5 border-t border-slate-700/60 flex justify-between items-center text-[11px] font-bold text-emerald-300"><span>Chi phí:</span><span id="cost-orbital">120 🧬</span></div>
                </button>
            </div>
        </div>
    </div>

    <!-- GIAO DIỆN MẶT TRĂNG -->
    <div id="moonUI" class="hidden">
        <div class="absolute top-4 left-4 right-4 z-20 flex justify-between items-center gap-3">
            <div class="hud-glass px-4 py-2.5 rounded-2xl flex items-center gap-4">
                <div>
                    <div class="text-[10px] text-slate-400">🪙 HELIUM-3</div>
                    <div class="font-orbitron font-bold text-sm text-purple-300"><span id="moonRes">0</span> <span class="text-[9px] text-amber-300">(+<span id="moonRate">1</span>/s)</span></div>
                </div>
            </div>
            <button onclick="exitMoonMode()" class="hud-glass px-4 py-2.5 rounded-2xl font-orbitron font-bold text-xs text-emerald-300 hover:border-emerald-400 transition">
                🌍 Quay lại Trái Đất
            </button>
        </div>

        <div class="absolute bottom-5 left-1/2 -translate-x-1/2 z-20 hud-glass p-3.5 rounded-2xl w-[95vw] max-w-2xl">
            <div class="flex justify-between items-center mb-2 px-1">
                <span class="text-[11px] font-orbitron font-bold text-purple-300 tracking-wider">🚀 CĂN CỨ MẶT TRĂNG KHỔNG LỒ</span>
            </div>
            <div class="grid grid-cols-2 gap-3">
                <button onclick="buyMoonUpgrade('dome')" class="upgrade-card p-3 rounded-xl text-left flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center"><span class="text-xs font-bold text-cyan-300">🌕 Trạm Vòm Lunar</span><span id="lvl-dome" class="text-[10px] bg-cyan-950 text-cyan-300 px-1.5 py-0.5 rounded font-bold">Lv.0</span></div>
                        <p class="text-[10px] text-slate-400 mt-1">Vòm sinh học lớn, +2 He-3/s</p>
                    </div>
                    <div class="mt-2 pt-1.5 border-t border-slate-700/60 flex justify-between items-center text-[11px] font-bold text-purple-300"><span>Chi phí:</span><span id="cost-dome">50 🪙</span></div>
                </button>

                <button onclick="buyMoonUpgrade('laser')" class="upgrade-card p-3 rounded-xl text-left flex flex-col justify-between">
                    <div>
                        <div class="flex justify-between items-center"><span class="text-xs font-bold text-rose-300">⚡ Máy Khoan Laser</span><span id="lvl-laser" class="text-[10px] bg-rose-950 text-rose-300 px-1.5 py-0.5 rounded font-bold">Lv.0</span></div>
                        <p class="text-[10px] text-slate-400 mt-1">Khoan khổng lồ, +8 He-3/s</p>
                    </div>
                    <div class="mt-2 pt-1.5 border-t border-slate-700/60 flex justify-between items-center text-[11px] font-bold text-purple-300"><span>Chi phí:</span><span id="cost-laser">200 🪙</span></div>
                </button>
            </div>
        </div>
    </div>

    <div id="canvasContainer" class="fixed inset-0 z-0"></div>

    <script>
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

            let chordIdx = 0, noteIdx = 0;

            function playNote(freq, isBass, duration) {
                if (!audioCtx || audioCtx.state !== 'running') return;
                const now = audioCtx.currentTime;
                const osc1 = audioCtx.createOscillator();
                const osc2 = audioCtx.createOscillator();
                const env = audioCtx.createGain();
                const filter = audioCtx.createBiquadFilter();

                if (isBass) {
                    osc1.type = 'sine'; osc2.type = 'triangle';
                    osc1.frequency.setValueAtTime(freq, now);
                    osc2.frequency.setValueAtTime(freq, now);
                    filter.type = 'lowpass'; filter.frequency.setValueAtTime(250, now);
                    env.gain.setValueAtTime(0.001, now);
                    env.gain.linearRampToValueAtTime(0.32, now + 0.2);
                    env.gain.exponentialRampToValueAtTime(0.0001, now + duration * 2.5);
                } else {
                    osc1.type = 'sawtooth'; osc2.type = 'sine';
                    osc1.frequency.setValueAtTime(freq, now);
                    osc2.frequency.setValueAtTime(freq * 1.004, now);
                    filter.type = 'lowpass'; filter.frequency.setValueAtTime(1400, now);
                    filter.frequency.exponentialRampToValueAtTime(400, now + duration);
                    env.gain.setValueAtTime(0.001, now);
                    env.gain.linearRampToValueAtTime(0.12, now + 0.1);
                    env.gain.exponentialRampToValueAtTime(0.0001, now + duration * 1.8);
                }

                osc1.connect(filter); osc2.connect(filter); filter.connect(env);
                env.connect(masterGain);
                if (!isBass) env.connect(delayNode);

                osc1.start(now); osc2.start(now);
                osc1.stop(now + duration * 2); osc2.stop(now + duration * 2);
            }

            function triggerStep() {
                if (!isMusicPlaying) return;
                const currentChord = chords[chordIdx];
                if (noteIdx === 0) playNote(currentChord.bass, true, 3.5);
                const n = currentChord.notes[noteIdx % currentChord.notes.length];
                playNote(n, false, 1.4);
                noteIdx++;
                if (noteIdx >= 8) { noteIdx = 0; chordIdx = (chordIdx + 1) % chords.length; }
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
                audioCtx.suspend(); isMusicPlaying = false;
                document.getElementById('audioBtn').innerText = '🔇 Muted';
            } else {
                audioCtx.resume(); isMusicPlaying = true;
                document.getElementById('audioBtn').innerText = '🔊 Music: ON';
            }
        }

        // =========================================================================
        // HỆ THỐNG GAME TRONG GAME (TRÁI ĐẤT & MẶT TRĂNG KHỔNG LỒ)
        // =========================================================================
        let gameMode = 'SPACE'; // 'SPACE', 'EARTH', 'MOON'
        let spaceGroup, earthGroup, moonGroup;
        let earthSatellites = [], earthClouds = [];

        const civState = {
            energy: 50,
            evo: 0,
            moonRes: 0,
            upgrades: {
                forest: { level: 1, baseCost: 25, costType: 'energy', scale: 1.55 },
                city: { level: 0, baseCost: 80, costType: 'energy', scale: 1.65 },
                reactor: { level: 0, baseCost: 250, costType: 'energy', scale: 1.75 },
                orbital: { level: 0, baseCost: 120, costType: 'evo', scale: 1.9 }
            },
            moonUpgrades: {
                dome: { level: 0, baseCost: 50, scale: 1.6 },
                laser: { level: 0, baseCost: 200, scale: 1.8 }
            }
        };

        function getUpgradeCost(key) {
            const u = civState.upgrades[key];
            return Math.floor(u.baseCost * Math.pow(u.scale, u.level));
        }

        function getMoonCost(key) {
            const u = civState.moonUpgrades[key];
            return Math.floor(u.baseCost * Math.pow(u.scale, u.level));
        }

        function getTotalLvl() {
            return civState.upgrades.forest.level + civState.upgrades.city.level + civState.upgrades.reactor.level + civState.upgrades.orbital.level;
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

        function getMoonRates() {
            const d = civState.moonUpgrades.dome.level;
            const l = civState.moonUpgrades.laser.level;
            return { moonRate: Math.round(d * 2 + l * 8) };
        }

        function updateCivUI() {
            const rates = getProductionRates();
            const moonRates = getMoonRates();
            document.getElementById('resEnergy').innerText = Math.floor(civState.energy).toLocaleString();
            document.getElementById('resEvo').innerText = Math.floor(civState.evo).toLocaleString();
            document.getElementById('resMoon').innerText = Math.floor(civState.moonRes).toLocaleString();
            document.getElementById('rateEnergy').innerText = rates.energyRate.toLocaleString();
            document.getElementById('rateEvo').innerText = rates.evoRate.toLocaleString();
            document.getElementById('clickPower').innerText = rates.clickPower.toLocaleString();

            document.getElementById('moonRes').innerText = Math.floor(civState.moonRes).toLocaleString();
            document.getElementById('moonRate').innerText = moonRates.moonRate.toLocaleString();

            const totalLvl = getTotalLvl();
            let stage = "Khởi Nguyên";
            if (totalLvl >= 20) stage = "Liên Sao (Type II)";
            else if (totalLvl >= 12) stage = "Lượng Tử & Quỹ Đạo";
            else if (totalLvl >= 6) stage = "Đô Thị Tương Lai";
            else if (totalLvl >= 3) stage = "Công Nghiệp Xanh";
            document.getElementById('civStage').innerText = stage;

            const launchBtn = document.getElementById('launchMoonBtn');
            if (totalLvl >= 6) {
                launchBtn.classList.remove('hidden');
            } else {
                launchBtn.classList.add('hidden');
            }

            ['forest', 'city', 'reactor', 'orbital'].forEach(key => {
                const u = civState.upgrades[key];
                const cost = getUpgradeCost(key);
                document.getElementById(`lvl-${key}`).innerText = `Lv.${u.level}`;
                document.getElementById(`cost-${key}`).innerText = `${cost.toLocaleString()} ${u.costType === 'energy' ? '⚡' : '🧬'}`;
                document.getElementById(`upg-${key}`).disabled = u.costType === 'energy' ? civState.energy < cost : civState.evo < cost;
            });

            ['dome', 'laser'].forEach(key => {
                const u = civState.moonUpgrades[key];
                const cost = getMoonCost(key);
                document.getElementById(`lvl-${key}`).innerText = `Lv.${u.level}`;
                document.getElementById(`cost-${key}`).innerText = `${cost.toLocaleString()} 🪙`;
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
                civState.energy -= cost; u.level++; spawn3DBuilding(key, u.level);
            } else if (u.costType === 'evo' && civState.evo >= cost) {
                civState.evo -= cost; u.level++; spawn3DBuilding(key, u.level);
            }
            updateCivUI();
        }

        function buyMoonUpgrade(key) {
            const u = civState.moonUpgrades[key];
            const cost = getMoonCost(key);
            if (civState.moonRes >= cost) {
                civState.moonRes -= cost;
                u.level++;
                spawnMoonStructure(key, u.level);
            }
            updateCivUI();
        }

        setInterval(() => {
            const rates = getProductionRates();
            const moonRates = getMoonRates();
            civState.energy += rates.energyRate;
            civState.evo += rates.evoRate;
            civState.moonRes += moonRates.moonRate;
            if (gameMode !== 'SPACE') updateCivUI();
        }, 1000);

        // =========================================================================
        // KHỞI TẠO ĐẢO TRÁI ĐẤT & MẶT TRĂNG (KÍCH THƯỚC ĐÃ PHÓNG TO GẤP ĐÔI)
        // =========================================================================
        function buildEarthBiome3D() {
            earthGroup = new THREE.Group();
            earthGroup.visible = false;
            scene.add(earthGroup);

            const hemiLight = new THREE.HemisphereLight(0x7dd3fc, 0x064e3b, 0.8);
            earthGroup.add(hemiLight);
            const dirLight = new THREE.DirectionalLight(0xfef3c7, 1.2);
            dirLight.position.set(40, 60, 40);
            earthGroup.add(dirLight);

            // Phóng to kích thước đảo Trái Đất (Bán kính từ 16 lên 32)
            const baseMesh = new THREE.Mesh(
                new THREE.ConeGeometry(32, 35, 8).rotateX(Math.PI),
                new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.9, flatShading: true })
            );
            baseMesh.position.y = -17.5;
            earthGroup.add(baseMesh);

            const grassMesh = new THREE.Mesh(
                new THREE.CylinderGeometry(32.4, 32, 1.5, 36),
                new THREE.MeshStandardMaterial({ color: 0x059669, roughness: 0.8 })
            );
            grassMesh.position.y = -0.4;
            earthGroup.add(grassMesh);

            const lakeMesh = new THREE.Mesh(
                new THREE.CylinderGeometry(11, 11, 1.6, 28),
                new THREE.MeshStandardMaterial({ color: 0x0284c7, emissive: 0x0369a1, emissiveIntensity: 0.2, roughness: 0.3 })
            );
            lakeMesh.position.set(-8, -0.3, 6);
            earthGroup.add(lakeMesh);

            for (let i = 0; i < 10; i++) {
                const cloud = new THREE.Mesh(
                    new THREE.DodecahedronGeometry(3 + Math.random() * 2, 1),
                    new THREE.MeshStandardMaterial({ color: 0xe2e8f0, transparent: true, opacity: 0.55, flatShading: true })
                );
                cloud.scale.set(2.5, 0.7, 1.5);
                cloud.position.set((Math.random() - 0.5) * 45, 16 + Math.random() * 6, (Math.random() - 0.5) * 45);
                earthGroup.add(cloud);
                earthClouds.push(cloud);
            }
            spawn3DBuilding('forest', 1);
        }

        function buildMoonBiome3D() {
            moonGroup = new THREE.Group();
            moonGroup.visible = false;
            scene.add(moonGroup);

            const moonHemi = new THREE.HemisphereLight(0x94a3b8, 0x1e293b, 1.0);
            moonGroup.add(moonHemi);
            const moonSun = new THREE.DirectionalLight(0xffffff, 2.2);
            moonSun.position.set(40, 60, 40);
            moonGroup.add(moonSun);

            // Phóng to kích thước đảo Mặt Trăng (Bán kính từ 17 lên 34)
            const moonBase = new THREE.Mesh(
                new THREE.ConeGeometry(34, 37, 8).rotateX(Math.PI),
                new THREE.MeshStandardMaterial({ color: 0x475569, roughness: 0.95, metalness: 0.2, flatShading: true })
            );
            moonBase.position.y = -18.5;
            moonGroup.add(moonBase);

            const moonSurface = new THREE.Mesh(
                new THREE.CylinderGeometry(33, 32.5, 1.6, 36),
                new THREE.MeshStandardMaterial({ color: 0x64748b, roughness: 0.85, metalness: 0.1 })
            );
            moonSurface.position.y = -0.4;
            moonGroup.add(moonSurface);

            for (let i = 0; i < 14; i++) {
                const crater = new THREE.Mesh(
                    new THREE.RingGeometry(1, 3.5, 16),
                    new THREE.MeshStandardMaterial({ color: 0x334155, side: THREE.DoubleSide, roughness: 0.9 })
                );
                crater.rotation.x = Math.PI / 2;
                const angle = Math.random() * Math.PI * 2;
                const r = 8 + Math.random() * 20;
                crater.position.set(Math.cos(angle) * r, 0.45, Math.sin(angle) * r);
                moonGroup.add(crater);
            }
        }

        function spawn3DBuilding(type, level) {
            if (!earthGroup) return;
            if (type === 'forest') {
                for (let i = 0; i < 6; i++) {
                    const angle = Math.random() * Math.PI * 2;
                    const rad = 12 + Math.random() * 16;
                    const tree = new THREE.Group();
                    const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.35, 0.55, 2.4, 6), new THREE.MeshStandardMaterial({ color: 0x78350f }));
                    trunk.position.y = 1.2;
                    const leaves = new THREE.Mesh(new THREE.ConeGeometry(2.2, 5, 7), new THREE.MeshStandardMaterial({ color: 0x059669, flatShading: true }));
                    leaves.position.y = 4.2;
                    tree.add(trunk, leaves);
                    tree.position.set(Math.cos(angle) * rad, 0.4, Math.sin(angle) * rad);
                    earthGroup.add(tree);
                }
            } else if (type === 'city') {
                const angle = Math.random() * Math.PI * 2;
                const rad = 6 + Math.random() * 18;
                const h = 8 + level * 1.2;
                const tower = new THREE.Mesh(
                    new THREE.BoxGeometry(2.5, h, 2.5),
                    new THREE.MeshStandardMaterial({ color: 0x0f172a, emissive: 0x38bdf8, emissiveIntensity: 0.4, roughness: 0.2, metalness: 0.9 })
                );
                tower.position.set(Math.cos(angle) * rad, h / 2 + 0.4, Math.sin(angle) * rad);
                earthGroup.add(tower);
            } else if (type === 'reactor') {
                const angle = Math.random() * Math.PI * 2;
                const rad = 10;
                const ring = new THREE.Mesh(new THREE.TorusGeometry(2.5, 0.35, 12, 24), new THREE.MeshStandardMaterial({ color: 0xf59e0b, emissive: 0xd97706, emissiveIntensity: 0.7 }));
                ring.rotation.x = Math.PI / 2; ring.position.set(Math.cos(angle) * rad, 3.5, Math.sin(angle) * rad);
                earthGroup.add(ring);
            } else if (type === 'orbital') {
                const satPivot = new THREE.Group();
                const sat = new THREE.Mesh(new THREE.OctahedronGeometry(1.2, 0), new THREE.MeshStandardMaterial({ color: 0xc084fc, emissive: 0x9333ea, emissiveIntensity: 0.8 }));
                sat.position.set(28, 14, 0); satPivot.add(sat);
                earthGroup.add(satPivot);
                earthSatellites.push(satPivot);
            }
        }

        function spawnMoonStructure(type, level) {
            if (!moonGroup) return;
            const angle = Math.random() * Math.PI * 2;
            const rad = 8 + Math.random() * 18;
            if (type === 'dome') {
                const dome = new THREE.Mesh(
                    new THREE.SphereGeometry(3, 16, 16, 0, Math.PI * 2, 0, Math.PI / 2),
                    new THREE.MeshStandardMaterial({ color: 0x38bdf8, emissive: 0x0284c7, emissiveIntensity: 0.5, transparent: true, opacity: 0.75, roughness: 0.1 })
                );
                dome.position.set(Math.cos(angle) * rad, 0.5, Math.sin(angle) * rad);
                moonGroup.add(dome);
            } else if (type === 'laser') {
                const laserTower = new THREE.Group();
                const base = new THREE.Mesh(new THREE.CylinderGeometry(1.6, 2.2, 3, 8), new THREE.MeshStandardMaterial({ color: 0x334155, metalness: 0.8 }));
                base.position.y = 1.5;
                const beam = new THREE.Mesh(new THREE.CylinderGeometry(0.3, 0.3, 8, 8), new THREE.MeshStandardMaterial({ color: 0xf43f5e, emissive: 0xe11d48, emissiveIntensity: 1.5 }));
                beam.position.y = 6;
                laserTower.add(base, beam);
                laserTower.position.set(Math.cos(angle) * rad, 0.5, Math.sin(angle) * rad);
                moonGroup.add(laserTower);
            }
        }

        function enterEarthMode() {
            gameMode = 'EARTH';
            spaceGroup.visible = false; moonGroup.visible = false; earthGroup.visible = true;
            if (bloomPass) bloomPass.strength = 0.28;
            if (renderer) renderer.toneMappingExposure = 0.85;

            document.getElementById('spaceUI').classList.add('hidden');
            document.getElementById('moonUI').classList.add('hidden');
            document.getElementById('earthUI').classList.remove('hidden');

            currentLookAt.set(0, 2.5, 0); targetCamDist = 65; camDist = 65; camAngleX = 0.45; camAngleY = 0.6;
            updateCivUI();
        }

        function enterMoonMode() {
            gameMode = 'MOON';
            spaceGroup.visible = false; earthGroup.visible = false; moonGroup.visible = true;
            if (bloomPass) bloomPass.strength = 0.35;
            if (renderer) renderer.toneMappingExposure = 0.9;

            document.getElementById('spaceUI').classList.add('hidden');
            document.getElementById('earthUI').classList.add('hidden');
            document.getElementById('moonUI').classList.remove('hidden');

            currentLookAt.set(0, 2.5, 0); targetCamDist = 65; camDist = 65; camAngleX = 0.45; camAngleY = 0.6;
            updateCivUI();
        }

        function exitEarthMode() {
            gameMode = 'SPACE';
            earthGroup.visible = false; moonGroup.visible = false; spaceGroup.visible = true;
            if (bloomPass) bloomPass.strength = 1.4;
            if (renderer) renderer.toneMappingExposure = 1.25;

            document.getElementById('earthUI').classList.add('hidden');
            document.getElementById('moonUI').classList.add('hidden');
            document.getElementById('spaceUI').classList.remove('hidden');
            selectPlanet('earth');
        }

        function exitMoonMode() {
            enterEarthMode();
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
            sun: { name: "MẶT TRỜI (SOL)", desc: "Khối cầu plasma sôi sục.", dist: "0 AU", temp: "5,500 °C", zoom: 72 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Bề mặt đá bazan xám bạc.", dist: "0.39 AU", temp: "430 °C", zoom: 4.5 },
            venus: { name: "SAO KIM (VENUS)", desc: "Khí quyển mây axit sulfuric.", dist: "0.72 AU", temp: "462 °C", zoom: 6 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Hành tinh sự sống. Đạt cấp độ 6 để mở khóa thám hiểm Mặt Trăng!", dist: "1.00 AU", temp: "15 °C", zoom: 6.5 },
            mars: { name: "SAO HỎA (MARS)", desc: "Sa mạc oxit sắt đỏ rực.", dist: "1.52 AU", temp: "-63 °C", zoom: 5.5 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Khí quyển dải băng tầng & Vết Đỏ Lớn.", dist: "5.20 AU", temp: "-110 °C", zoom: 13 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Hệ thống vành đai băng đa tầng.", dist: "9.58 AU", temp: "-140 °C", zoom: 16 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng khí metan xanh ngọc.", dist: "19.22 AU", temp: "-195 °C", zoom: 10 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Thế giới băng giá xanh thẫm.", dist: "30.05 AU", temp: "-200 °C", zoom: 10 }
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
            float hash(vec3 p) { p = fract(p * 0.3183099 + 0.1); p *= 17.0; return fract(p.x * p.y * p.z * (p.x + p.y + p.z)); }
            float noise(vec3 x) {
                vec3 i = floor(x); vec3 f = fract(x); f = f * f * (3.0 - 2.0 * f);
                return mix(mix(mix(hash(i + vec3(0,0,0)), hash(i + vec3(1,0,0)), f.x),
                               mix(hash(i + vec3(0,1,0)), hash(i + vec3(1,1,0)), f.x), f.y),
                           mix(mix(hash(i + vec3(0,0,1)), hash(i + vec3(1,0,1)), f.x),
                               mix(hash(i + vec3(0,1,1)), hash(i + vec3(1,1,1)), f.x), f.y), f.z);
            }
            float fbm(vec3 p) { float v = 0.0; float a = 0.5; for (int i = 0; i < 5; i++) { v += a * noise(p); p = p * 2.05 + vec3(0.15); a *= 0.5; } return v; }
            void main() {
                vec3 pos = vPosition * 1.4;
                float n1 = fbm(pos + vec3(uTime * 0.25, -uTime * 0.15, uTime * 0.2));
                float n2 = fbm(pos * 2.2 - vec3(uTime * 0.3));
                float plasma = smoothstep(0.25, 0.85, n1 * 0.65 + n2 * 0.45);
                vec3 col = mix(vec3(0.55, 0.08, 0.01), vec3(1.0, 0.38, 0.02), smoothstep(0.1, 0.55, plasma));
                col = mix(col, vec3(1.45, 1.2, 0.6), smoothstep(0.5, 0.9, plasma));
                float fresnel = pow(1.0 - max(dot(vNormal, vec3(0.0, 0.0, 1.0)), 0.0), 2.5);
                col += vec3(1.2, 0.5, 0.1) * fresnel * 0.8;
                gl_FragColor = vec4(col, 1.0);
            }
        `;

        function createAtmosphereMaterial(colorHex, intensity = 1.1, power = 3.5) {
            return new THREE.ShaderMaterial({
                uniforms: { uColor: { value: new THREE.Color(colorHex) }, uIntensity: { value: intensity }, uPower: { value: power } },
                vertexShader: `varying vec3 vNormal; void main() { vNormal = normalize(normalMatrix * normal); gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0); }`,
                fragmentShader: `uniform vec3 uColor; uniform float uIntensity; uniform float uPower; varying vec3 vNormal; void main() { float glow = pow(0.68 - dot(vNormal, vec3(0.0, 0.0, 1.0)), uPower); gl_FragColor = vec4(uColor * uIntensity, max(0.0, glow)); }`,
                side: THREE.BackSide, blending: THREE.AdditiveBlending, transparent: true, depthWrite: false
            });
        }

        function createFlareTextures() {
            const streakCanvas = document.createElement('canvas'); streakCanvas.width = 512; streakCanvas.height = 64;
            const sCtx = streakCanvas.getContext('2d');
            const sGrad = sCtx.createLinearGradient(0, 32, 512, 32);
            sGrad.addColorStop(0.0, 'rgba(56, 189, 248, 0)'); sGrad.addColorStop(0.5, 'rgba(255, 251, 235, 0.95)'); sGrad.addColorStop(1.0, 'rgba(56, 189, 248, 0)');
            sCtx.fillStyle = sGrad; sCtx.fillRect(0, 26, 512, 12);

            const orbCanvas = document.createElement('canvas'); orbCanvas.width = 128; orbCanvas.height = 128;
            const oCtx = orbCanvas.getContext('2d');
            const oGrad = oCtx.createRadialGradient(64, 64, 4, 64, 64, 60);
            oGrad.addColorStop(0, 'rgba(255, 255, 255, 0.9)'); oGrad.addColorStop(1, 'rgba(0, 0, 0, 0)');
            oCtx.fillStyle = oGrad; oCtx.fillRect(0, 0, 128, 128);

            return { streak: new THREE.CanvasTexture(streakCanvas), orb: new THREE.CanvasTexture(orbCanvas) };
        }

        function generatePlanetMaps(type) {
            const w = 2048, h = 1024;
            const diffCanvas = document.createElement('canvas'); const bumpCanvas = document.createElement('canvas');
            diffCanvas.width = bumpCanvas.width = w; diffCanvas.height = bumpCanvas.height = h;
            const ctx = diffCanvas.getContext('2d'); const bCtx = bumpCanvas.getContext('2d');
            bCtx.fillStyle = '#808080'; bCtx.fillRect(0, 0, w, h);

            if (type === 'mercury') { ctx.fillStyle = '#64748b'; ctx.fillRect(0, 0, w, h); }
            else if (type === 'venus') { ctx.fillStyle = '#f59e0b'; ctx.fillRect(0, 0, w, h); }
            else if (type === 'earth') {
                ctx.fillStyle = '#0284c7'; ctx.fillRect(0, 0, w, h);
                ctx.fillStyle = '#15803d'; ctx.beginPath(); ctx.ellipse(1000, 500, 300, 180, 0, 0, Math.PI * 2); ctx.fill();
            }
            else if (type === 'clouds') {
                ctx.clearRect(0, 0, w, h);
                ctx.fillStyle = 'rgba(255,255,255,0.4)';
                for (let i = 0; i < 150; i++) ctx.fillRect(Math.random() * w, Math.random() * h, 180, 25);
            }
            else if (type === 'mars') { ctx.fillStyle = '#9a3412'; ctx.fillRect(0, 0, w, h); }
            else if (type === 'jupiter') { ctx.fillStyle = '#d97706'; ctx.fillRect(0, 0, w, h); }
            else if (type === 'saturn') { ctx.fillStyle = '#ca8a04'; ctx.fillRect(0, 0, w, h); }
            else if (type === 'uranus') { ctx.fillStyle = '#06b6d4'; ctx.fillRect(0, 0, w, h); }
            else if (type === 'neptune') { ctx.fillStyle = '#1d4ed8'; ctx.fillRect(0, 0, w, h); }
            return { map: new THREE.CanvasTexture(diffCanvas), bumpMap: new THREE.CanvasTexture(bumpCanvas) };
        }

        function createMilkyWayAndGalaxies() {
            milkyWayGroup = new THREE.Group();
            milkyWayGroup.rotation.z = Math.PI / 6;
            const starCount = 60000;
            const starGeo = new THREE.BufferGeometry();
            const starPos = new Float32Array(starCount * 3);
            const starCol = new Float32Array(starCount * 3);
            const colors = [new THREE.Color(0x38bdf8), new THREE.Color(0xa855f7), new THREE.Color(0xfde047), new THREE.Color(0xffffff)];

            for (let i = 0; i < starCount; i++) {
                const angle = Math.random() * Math.PI * 2;
                const radius = 800 + Math.random() * 2200;
                const spread = (Math.random() - 0.5) * (Math.random() - 0.5) * 850;
                starPos[i * 3] = Math.cos(angle) * radius; starPos[i * 3 + 1] = spread; starPos[i * 3 + 2] = Math.sin(angle) * radius;
                const c = colors[Math.floor(Math.random() * colors.length)];
                starCol[i * 3] = c.r; starCol[i * 3 + 1] = c.g; starCol[i * 3 + 2] = c.b;
            }
            starGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            starGeo.setAttribute('color', new THREE.BufferAttribute(starCol, 3));
            milkyWayGroup.add(new THREE.Points(starGeo, new THREE.PointsMaterial({ size: 2.8, vertexColors: true, transparent: true, opacity: 0.9, blending: THREE.AdditiveBlending })));

            cosmicDust = new THREE.Points(
                new THREE.BufferGeometry().setAttribute('position', new THREE.BufferAttribute(new Float32Array(24000), 3)),
                new THREE.PointsMaterial({ color: 0xbae6fd, size: 1.2, transparent: true, opacity: 0.8, blending: THREE.AdditiveBlending })
            );
            spaceGroup.add(cosmicDust, milkyWayGroup);
        }

        function createAsteroidBelt() {
            asteroidBelt = new THREE.InstancedMesh(new THREE.DodecahedronGeometry(0.11, 0), new THREE.MeshStandardMaterial({ color: 0x94a3b8 }), 2200);
            const dummy = new THREE.Object3D();
            for (let i = 0; i < 2200; i++) {
                const angle = Math.random() * Math.PI * 2; const radius = 30.2 + Math.random() * 3.6;
                dummy.position.set(Math.cos(angle) * radius, (Math.random() - 0.5) * 1.1, Math.sin(angle) * radius);
                dummy.updateMatrix(); asteroidBelt.setMatrixAt(i, dummy.matrix);
            }
            spaceGroup.add(asteroidBelt);
        }

        function createSpacetimeGrid() {
            spacetimeGrid = new THREE.Mesh(new THREE.PlaneGeometry(160, 160, 48, 48).rotateX(-Math.PI / 2), new THREE.MeshBasicMaterial({ color: 0x0ea5e9, wireframe: true, transparent: true, opacity: 0.16 }));
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
            composer.addPass(renderScene); composer.addPass(bloomPass);

            const sunLight = new THREE.PointLight(0xfff5e4, 3.8, 2000, 0.25);
            spaceGroup.add(sunLight, new THREE.AmbientLight(0x0f172a, 0.45));

            const flareTex = createFlareTextures();
            const lensflare = new THREE.Lensflare();
            lensflare.addElement(new THREE.LensflareElement(flareTex.streak, 720, 0, new THREE.Color(0x7dd3fc)));
            lensflare.addElement(new THREE.LensflareElement(flareTex.orb, 150, 0, new THREE.Color(0xfef08a)));
            sunLight.add(lensflare);

            createMilkyWayAndGalaxies();
            createAsteroidBelt();
            createSpacetimeGrid();
            buildEarthBiome3D();
            buildMoonBiome3D();

            sunUniforms = { uTime: { value: 0 } };
            const sun = new THREE.Mesh(new THREE.SphereGeometry(5.2, 64, 64), new THREE.ShaderMaterial({ uniforms: sunUniforms, vertexShader: sunVertexShader, fragmentShader: sunFragmentShader }));
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(6.8, 64, 64), createAtmosphereMaterial(0xff8c00, 1.4, 3.0)));
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
                const maps = generatePlanetMaps(cfg.id);
                const mesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size, 64, 64), new THREE.MeshStandardMaterial({ map: maps.map, bumpMap: maps.bumpMap, roughness: 0.72 }));
                spaceGroup.add(mesh);
                planets[cfg.id] = { mesh, distance: cfg.dist, speed: cfg.speed, angle: Math.random() * Math.PI * 2 };
            });

            window.addEventListener('mousedown', e => { if (e.target.tagName === 'CANVAS') { isDragging = true; prevMouse = { x: e.clientX, y: e.clientY }; } });
            window.addEventListener('mousemove', e => {
                if (!isDragging) return;
                camAngleY -= (e.clientX - prevMouse.x) * 0.005;
                camAngleX = Math.max(-1.45, Math.min(1.45, camAngleX - (e.clientY - prevMouse.y) * 0.005));
                prevMouse = { x: e.clientX, y: e.clientY };
            });
            window.addEventListener('mouseup', () => isDragging = false);
            window.addEventListener('wheel', e => { targetCamDist = Math.max(2.5, Math.min(450, targetCamDist + e.deltaY * 0.06)); });
            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight); composer.setSize(window.innerWidth, window.innerHeight);
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
            document.getElementById('flyBtn').innerText = isFreeFly ? '🛸 Free-Fly: ON' : '🛸 Free-Fly: OFF';
        }

        function toggleSpacetimeGrid() {
            if (!spacetimeGrid) return;
            spacetimeGrid.visible = !spacetimeGrid.visible;
            document.getElementById('gridBtn').innerText = spacetimeGrid.visible ? '🕸️ Lưới KG: ON' : '🕸️ Lưới KG: OFF';
        }

        function setSpeed(v) { timeSpeed = parseFloat(v); document.getElementById('speedVal').innerText = timeSpeed.toFixed(1) + 'x'; }

        function toggleHud() {
            isHudVisible = !isHudVisible;
            document.getElementById('hudBody').style.maxHeight = isHudVisible ? '380px' : '0px';
        }

        function selectPlanet(id) {
            focusedObject = id; isFreeFly = false;
            document.querySelectorAll('.planet-pill').forEach(b => b.classList.remove('active'));
            document.getElementById(`btn-${id}`).classList.add('active');
            const info = planetDB[id];
            if (info) {
                document.getElementById('planetTitle').innerText = info.name;
                document.getElementById('planetDesc').innerText = info.desc;
                document.getElementById('planetDist').innerText = info.dist;
                document.getElementById('planetTemp').innerText = info.temp;
                targetCamDist = info.zoom;
            }
            document.getElementById('landEarthBtn').classList.toggle('hidden', id !== 'earth');
        }

        function animate() {
            requestAnimationFrame(animate);

            if (gameMode === 'EARTH') {
                earthSatellites.forEach((sat, i) => sat.rotation.y += 0.015 + i * 0.004);
            } else if (gameMode === 'MOON') {
                // Xoay nhẹ Mặt Trăng
            } else {
                if (sunUniforms) sunUniforms.uTime.value += 0.015 * timeSpeed;
                if (milkyWayGroup) milkyWayGroup.rotation.y += 0.00015 * timeSpeed;
                Object.keys(planets).forEach(id => {
                    const p = planets[id];
                    if (p.speed > 0) {
                        p.angle += p.speed * timeSpeed;
                        p.mesh.position.x = Math.cos(p.angle) * p.distance;
                        p.mesh.position.z = Math.sin(p.angle) * p.distance;
                        p.mesh.rotation.y += 0.012 * timeSpeed;
                    }
                });
            }

            if (gameMode === 'SPACE') {
                const targetPos = new THREE.Vector3();
                if (planets[focusedObject]) planets[focusedObject].mesh.getWorldPosition(targetPos);
                currentLookAt.lerp(targetPos, 0.08);
            }

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
