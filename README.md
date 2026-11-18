
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA Eyes 3D - Constellation Sword & Fixed Retry</title>
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
            background: rgba(3, 7, 18, 0.92);
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
        .custom-scrollbar::-webkit-scrollbar { height: 4px; width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: rgba(56, 189, 248, 0.6); border-radius: 99px; }
        #warpOverlay {
            position: fixed; inset: 0; background: radial-gradient(circle, rgba(244,63,94,0.9) 0%, rgba(15,5,10,1) 100%);
            z-index: 100; opacity: 0; pointer-events: none; transition: opacity 0.6s ease;
        }
        @keyframes pulseAlert {
            0%, 100% { opacity: 1; transform: scale(1); }
            50% { opacity: 0.35; transform: scale(1.02); }
        }
        .alert-pulse { animation: pulseAlert 0.4s infinite ease-in-out; }
    </style>
</head>
<body class="w-screen h-screen select-none">

    <div id="warpOverlay"></div>

    <!-- MÀN HÌNH CHỜ KHỞI ĐỘNG -->
    <div id="startOverlay" class="fixed inset-0 z-50 bg-[#010206] flex flex-col items-center justify-center p-6 text-center space-y-6">
        <div class="relative w-32 h-32 flex items-center justify-center">
            <div class="absolute inset-0 rounded-full border-2 border-rose-500/50 animate-ping"></div>
            <div class="w-24 h-24 rounded-full bg-gradient-to-tr from-rose-600 via-orange-500 to-cyan-300 shadow-[0_0_100px_rgba(244,63,94,0.8)]"></div>
        </div>
        <div class="space-y-2 max-w-lg">
            <h1 class="font-orbitron font-black text-2xl sm:text-4xl text-white tracking-widest">
                NASA EYES 3D <span class="text-transparent bg-clip-text bg-gradient-to-r from-rose-400 via-amber-400 to-cyan-300">& SWORD CONSTELLATION</span>
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 leading-relaxed">
                Nhiệm Vụ 1 Phút 30 Giây • Chòm Sao Đường Kiếm Laser • Sửa Lỗi Nút Thử Lại Tuyệt Đối!
            </p>
        </div>
        <button onclick="startExperience()" class="px-9 py-4 bg-gradient-to-r from-rose-600 via-amber-500 to-cyan-500 hover:brightness-110 font-orbitron font-bold text-xs sm:text-sm rounded-full text-white shadow-[0_0_60px_rgba(244,63,94,0.8)] transition active:scale-95">
            ⚡ KHỞI ĐỘNG VŨ TRỤ & BẮT ĐẦU
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
                    <button onclick="toggleSwordConstellation()" id="swordBtn" class="px-2 py-1 bg-slate-900/90 border border-slate-700 text-cyan-300 rounded-lg text-[10px] font-bold hover:border-cyan-400 transition">
                        ⚔️ Kiếm Sao: OFF
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
                        <div class="flex justify-between text-slate-400 mb-1"><span>☀️ Ánh sáng</span><span id="brightVal" class="text-amber-300 font-bold">125%</span></div>
                        <input type="range" min="0.3" max="2.5" step="0.05" value="1.25" oninput="setBrightness(this.value)" class="w-full accent-amber-400 h-1 cursor-pointer">
                    </div>
                </div>

                <div class="border-t border-slate-800/80 pt-2.5 space-y-1.5">
                    <div id="planetTitle" class="text-sm font-orbitron font-bold text-amber-300 tracking-wide">MẶT TRỜI (SOL)</div>
                    <p id="planetDesc" class="text-[11px] text-slate-300 leading-relaxed">
                        Chọn hành tinh để quan sát hoặc chọn Trái Đất để đổ bộ thực hiện nhiệm vụ sinh tồn 1 phút 30 giây!
                    </p>
                    <div class="grid grid-cols-2 gap-1 pt-1 text-[10px] text-slate-400">
                        <div>Khoảng cách: <span id="planetDist" class="text-slate-200">0 AU</span></div>
                        <div>Nhiệt độ: <span id="planetTemp" class="text-amber-300">5,500 °C</span></div>
                    </div>

                    <button id="landEarthBtn" onclick="enterEarthMode()" class="hidden w-full mt-2 py-2.5 px-3 bg-gradient-to-r from-rose-600 to-amber-500 hover:brightness-110 text-white font-orbitron font-black text-xs rounded-xl shadow-[0_0_25px_rgba(244,63,94,0.7)] transition animate-bounce">
                        🔥 ĐỔ BỘ TRÁI ĐẤT - 1M30S MISSION
                    </button>
                </div>
            </div>
        </header>

        <nav class="absolute bottom-5 left-1/2 -translate-x-1/2 z-20 hud-glass px-3 py-2.5 rounded-2xl flex gap-2 overflow-x-auto max-w-[95vw] custom-scrollbar">
            <button id="btn-sun" onclick="selectPlanet('sun')" class="planet-pill active px-3 py-1.5 text-amber-300 rounded-xl text-xs font-bold whitespace-nowrap">☀️ Mặt Trời</button>
            <button id="btn-mercury" onclick="selectPlanet('mercury')" class="planet-pill px-3 py-1.5 text-slate-200 rounded-xl text-xs font-bold whitespace-nowrap">🌑 Sao Thủy</button>
            <button id="btn-venus" onclick="selectPlanet('venus')" class="planet-pill px-3 py-1.5 text-amber-200 rounded-xl text-xs font-bold whitespace-nowrap">🌕 Sao Kim</button>
            <button id="btn-earth" onclick="selectPlanet('earth')" class="planet-pill px-3 py-1.5 text-rose-400 border-rose-500/50 rounded-xl text-xs font-bold whitespace-nowrap">🌍 Trái Đất 🔥</button>
            <button id="btn-mars" onclick="selectPlanet('mars')" class="planet-pill px-3 py-1.5 text-rose-400 rounded-xl text-xs font-bold whitespace-nowrap">🔴 Sao Hỏa</button>
            <button id="btn-jupiter" onclick="selectPlanet('jupiter')" class="planet-pill px-3 py-1.5 text-orange-300 rounded-xl text-xs font-bold whitespace-nowrap">🪐 Sao Mộc</button>
            <button id="btn-saturn" onclick="selectPlanet('saturn')" class="planet-pill px-3 py-1.5 text-yellow-200 rounded-xl text-xs font-bold whitespace-nowrap">🪐 Sao Thổ</button>
            <button id="btn-uranus" onclick="selectPlanet('uranus')" class="planet-pill px-3 py-1.5 text-cyan-300 rounded-xl text-xs font-bold whitespace-nowrap">💎 Thiên Vương</button>
            <button id="btn-neptune" onclick="selectPlanet('neptune')" class="planet-pill px-3 py-1.5 text-indigo-400 rounded-xl text-xs font-bold whitespace-nowrap">🔵 Hải Vương</button>
        </nav>
    </div>

    <!-- GIAO DIỆN GAME 2D TRÊN TRÁI ĐẤT -->
    <div id="earthUI" class="hidden absolute inset-0 z-30 bg-slate-950 flex flex-col items-center justify-center p-4 transition-colors duration-500">
        <div id="hudTopBar" class="absolute top-4 left-4 right-4 flex justify-between items-center hud-glass px-5 py-3 rounded-2xl max-w-4xl mx-auto w-full z-10 transition-colors duration-500">
            <div class="flex items-center gap-6">
                <div>
                    <div id="timerLabel" class="text-[10px] text-rose-400">⏳ THỜI GIAN CÒN LẠI (1M30S)</div>
                    <div id="missionTimer" class="font-orbitron font-bold text-sm text-amber-300">01:30</div>
                </div>
                <div class="h-6 w-[1px] bg-slate-700"></div>
                <div>
                    <div class="text-[10px] text-cyan-400">🎮 NHIỆM VỤ (<span id="diffLabel" class="text-amber-300">DỄ</span>)</div>
                    <div id="missionText" class="font-orbitron font-bold text-xs text-cyan-300">Chọn cấp độ khó để bắt đầu!</div>
                </div>
            </div>
            <button onclick="exitEarthMode()" class="px-4 py-2 bg-slate-900 border border-slate-700 hover:border-cyan-400 rounded-xl text-xs font-bold text-cyan-300 transition">
                🌌 Về Quỹ Đạo Vũ Trụ
            </button>
        </div>

        <div id="gameBoxContainer" class="relative w-full max-w-4xl h-[74vh] bg-[#030712] border border-cyan-500/40 rounded-3xl overflow-hidden shadow-[0_0_50px_rgba(56,189,248,0.2)] mt-12 transition-colors duration-500">
            <canvas id="game2DCanvas" class="w-full h-full block"></canvas>

            <!-- BẢNG CẢNH BÁO 30S & 10S CUỐI -->
            <div id="alertBanner30s" class="hidden absolute top-4 left-1/2 -translate-x-1/2 z-20 px-6 py-2.5 bg-rose-600 border-2 border-white text-white font-orbitron font-black text-xs sm:text-sm rounded-xl shadow-[0_0_40px_rgba(244,63,94,1)] alert-pulse pointer-events-none text-center">
                🚨 CẢNH BÁO KHẨN CẤP: NGUY CƠ TRÁI ĐẤT NỔ TUNG TRONG 30 GIÂY! 🚨
            </div>

            <!-- MÀN HÌNH CHỌN CẤP ĐỘ KHÓ -->
            <div id="difficultyOverlay" class="absolute inset-0 bg-slate-950/90 backdrop-blur-md flex flex-col items-center justify-center p-6 text-center space-y-6 z-20">
                <div class="space-y-2">
                    <h2 class="font-orbitron font-black text-2xl text-white tracking-wider">CHỌN CẤP ĐỘ SINH TỒN (01:30)</h2>
                    <p class="text-xs text-slate-400">Dưới 30s còi báo động khẩn cấp sẽ hú liên hồi. Hết giờ Trái Đất sẽ phát nổ BÙM!</p>
                </div>
                <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 w-full max-w-2xl">
                    <button onclick="startMission('easy')" class="p-4 bg-emerald-950/60 border border-emerald-500/50 hover:bg-emerald-900/60 hover:scale-105 rounded-2xl transition flex flex-col items-center space-y-2">
                        <span class="text-2xl">🟢</span>
                        <span class="font-orbitron font-bold text-emerald-300 text-sm">DỄ</span>
                        <span class="text-[11px] text-slate-300">Không có thiên thạch rơi. Dễ thở hơn.</span>
                    </button>
                    <button onclick="startMission('normal')" class="p-4 bg-amber-950/60 border border-amber-500/50 hover:bg-amber-900/60 hover:scale-105 rounded-2xl transition flex flex-col items-center space-y-2">
                        <span class="text-2xl">🟡</span>
                        <span class="font-orbitron font-bold text-amber-300 text-sm">BÌNH THƯỜNG</span>
                        <span class="text-[11px] text-slate-300">Mưa thiên thạch vừa phải. Né tránh cẩn thận.</span>
                    </button>
                    <button onclick="startMission('hard')" class="p-4 bg-rose-950/60 border border-rose-500/50 hover:bg-rose-900/60 hover:scale-105 rounded-2xl transition flex flex-col items-center space-y-2">
                        <span class="text-2xl">🔴</span>
                        <span class="font-orbitron font-bold text-rose-400 text-sm">SIÊU KHÓ</span>
                        <span class="text-[11px] text-slate-300">Mưa thiên thạch dày đặc tốc độ cao! Thử thách cực đại.</span>
                    </button>
                </div>
            </div>

            <!-- BẢNG THẤT BẠI KHI HẾT GIỜ (00:00) -->
            <div id="gameOverModal" class="hidden absolute inset-0 bg-rose-950/95 backdrop-blur-md flex flex-col items-center justify-center p-6 text-center space-y-5 z-40">
                <div class="hud-glass p-6 rounded-3xl max-w-md w-full border-rose-500 space-y-4 shadow-[0_0_80px_rgba(244,63,94,0.9)]">
                    <div class="text-5xl animate-bounce">💥🌍❌</div>
                    <h2 class="font-orbitron font-black text-2xl text-rose-500 tracking-widest">BẠN ĐÃ THẤT BẠI!</h2>
                    <p class="text-xs sm:text-sm text-slate-200 leading-relaxed">
                        Hết 1 phút 30 giây! Trái Đất đã bùng nổ hoàn toàn do không hoàn thành nhiệm vụ kịp thời.
                    </p>
                    <div class="flex gap-3 pt-2">
                        <button onclick="resetToDifficulty()" class="flex-1 py-3 bg-rose-600 hover:bg-rose-500 text-white font-orbitron font-bold text-xs rounded-xl transition shadow-xl cursor-pointer">
                            🔄 Thử Lại
                        </button>
                        <button onclick="exitEarthMode()" class="flex-1 py-3 bg-slate-800 hover:bg-slate-700 text-cyan-300 font-orbitron font-bold text-xs rounded-xl transition cursor-pointer">
                            🌌 Về Vũ Trụ
                        </button>
                    </div>
                </div>
            </div>

            <div id="gameMsg" class="hidden absolute inset-x-0 bottom-4 text-center pointer-events-none z-10">
                <span class="px-4 py-1.5 bg-slate-900/90 border border-cyan-500/60 rounded-xl text-[11px] font-bold text-amber-300 shadow-lg">
                    ⌨️ Dùng phím MŨI TÊN hoặc W, A, S, D để di chuyển! Tránh thiên thạch đỏ!
                </span>
            </div>
        </div>
    </div>

    <!-- GIAO DIỆN HOẠT ẢNH 3D NỔ TUNG TRÁI ĐẤT & TRỐN THOÁT -->
    <div id="moonEscapeUI" class="hidden fixed inset-0 z-40 pointer-events-none flex flex-col justify-between p-6">
        <div class="hud-glass px-5 py-3 rounded-2xl max-w-md mx-auto text-center border-amber-400/50">
            <div class="text-[10px] text-amber-300 tracking-widest font-bold">💥 BÙNG NỔ TRÁI ĐẤT & THOÁT HIỂM 3D</div>
            <div id="launchStatusText" class="font-orbitron font-bold text-sm text-white mt-0.5">Trái Đất đang phát nổ... Phóng tên lửa lên Mặt Trăng!</div>
        </div>

        <!-- BẢNG CHÚC MỪNG -->
        <div id="victoryModal3D" class="hidden pointer-events-auto m-auto hud-glass p-6 sm:p-8 rounded-3xl max-w-md w-full border-amber-400/70 text-center space-y-4 shadow-[0_0_80px_rgba(251,191,36,0.5)]">
            <div class="text-5xl animate-bounce">🌕🚀🎉</div>
            <h2 class="font-orbitron font-black text-xl sm:text-2xl text-transparent bg-clip-text bg-gradient-to-r from-amber-300 via-yellow-200 to-cyan-300">
                CHIẾN THẮNG TUYỆT ĐỐI!
            </h2>
            <p class="text-xs sm:text-sm text-slate-200 leading-relaxed">
                Bạn đã trốn thoát thành công trước khi hết giờ! Quan sát toàn cảnh ngoài vũ trụ để thấy tàn tích Trái Đất vừa phát nổ phía sau.
            </p>
            <div class="bg-slate-900/90 p-3 rounded-xl border border-slate-700 text-xs space-y-1 text-slate-300">
                <div>Cấp độ hoàn thành: <span id="winDiff3D" class="text-amber-300 font-bold">BÌNH THƯỜNG</span></div>
                <div>Thời gian còn lại: <span id="winTime3D" class="text-cyan-300 font-bold">00:45</span></div>
            </div>
            <div class="flex gap-3 pt-2">
                <button onclick="replayFrom3D()" class="flex-1 py-2.5 bg-amber-500 hover:bg-amber-400 text-slate-950 font-orbitron font-bold text-xs rounded-xl transition cursor-pointer">
                    🔄 Chơi Lại
                </button>
                <button onclick="finishToUniverse()" class="flex-1 py-2.5 bg-cyan-500 hover:bg-cyan-400 text-slate-950 font-orbitron font-bold text-xs rounded-xl transition cursor-pointer">
                    🌌 Ngắm Toàn Cảnh Vũ Trụ
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
                if (!audioCtx || audioCtx.state !== 'running' || !isMusicPlaying) return;
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

        function playEmergencySiren() {
            if (!audioCtx) return;
            try {
                const now = audioCtx.currentTime;
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(550, now);
                osc.frequency.linearRampToValueAtTime(1100, now + 0.35);
                osc.frequency.linearRampToValueAtTime(550, now + 0.7);

                gain.gain.setValueAtTime(0.2, now);
                gain.gain.exponentialRampToValueAtTime(0.001, now + 0.75);

                osc.connect(gain);
                gain.connect(masterGain);
                osc.start(now);
                osc.stop(now + 0.8);
            } catch (e) {}
        }

        function playExplosionBoom() {
            if (!audioCtx) return;
            try {
                const now = audioCtx.currentTime;
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(160, now);
                osc.frequency.exponentialRampToValueAtTime(25, now + 1.2);

                gain.gain.setValueAtTime(0.7, now);
                gain.gain.exponentialRampToValueAtTime(0.0001, now + 1.4);

                osc.connect(gain);
                gain.connect(masterGain);
                osc.start(now);
                osc.stop(now + 1.5);

                const bufferSize = audioCtx.sampleRate * 1.2;
                const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
                const output = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    output[i] = Math.random() * 2 - 1;
                }

                const whiteNoise = audioCtx.createBufferSource();
                whiteNoise.buffer = buffer;

                const filter = audioCtx.createBiquadFilter();
                filter.type = 'lowpass';
                filter.frequency.setValueAtTime(800, now);
                filter.frequency.exponentialRampToValueAtTime(60, now + 1.2);

                const noiseGain = audioCtx.createGain();
                noiseGain.gain.setValueAtTime(0.6, now);
                noiseGain.gain.exponentialRampToValueAtTime(0.001, now + 1.3);

                whiteNoise.connect(filter);
                filter.connect(noiseGain);
                noiseGain.connect(masterGain);

                whiteNoise.start(now);
                whiteNoise.stop(now + 1.4);
            } catch (e) {}
        }

        function setVolume(v) {
            const val = parseFloat(v);
            document.getElementById('volVal').innerText = Math.round(val * 100) + '%';
            if (masterGain && audioCtx) masterGain.gain.setTargetAtTime(val, audioCtx.currentTime, 0.05);
        }

        function setBrightness(v) {
            userBrightness = parseFloat(v);
            document.getElementById('brightVal').innerText = Math.round(userBrightness * 100) + '%';
            if (renderer) renderer.toneMappingExposure = userBrightness;
        }

        function toggleSpaceAudio() {
            if (!audioCtx) { startProceduralStrayMusic(); return; }
            if (isMusicPlaying) {
                isMusicPlaying = false;
                document.getElementById('audioBtn').innerText = '🔇 Muted';
            } else {
                isMusicPlaying = true;
                if (audioCtx.state === 'suspended') audioCtx.resume();
                document.getElementById('audioBtn').innerText = '🔊 Music: ON';
            }
        }

        // =========================================================================
        // HỆ THỐNG CHÒM SAO ĐƯỜNG KIẾM (CONSTELLATION SWORD)
        // =========================================================================
        let swordConstellationGroup = null;
        let isSwordVisible = false;

        function buildSwordConstellation() {
            if (swordConstellationGroup) {
                swordConstellationGroup.visible = isSwordVisible;
                return;
            }

            swordConstellationGroup = new THREE.Group();
            
            // Tọa độ các điểm chính tạo hình một thanh kiếm laser (Chuôi kiếm, chắn kiếm, lưỡi kiếm, đỉnh kiếm)
            const swordPoints = [
                new THREE.Vector3(0, -60, 0),   // Đuôi chuôi kiếm
                new THREE.Vector3(0, -35, 0),   // Cán chuôi kiếm
                new THREE.Vector3(0, -25, 0),   // Tâm chắn kiếm (Guard center)
                new THREE.Vector3(-18, -25, 0), // Cánh chắn trái
                new THREE.Vector3(18, -25, 0),  // Cánh chắn phải
                new THREE.Vector3(0, -25, 0),   // Về tâm chắn
                new THREE.Vector3(0, 60, 0)     // Mũi nhọn đỉnh kiếm
            ];

            const lineGeo = new THREE.BufferGeometry().setFromPoints(swordPoints);
            const lineMat = new THREE.LineBasicMaterial({ color: 0x38bdf8, linewidth: 3, transparent: true, opacity: 0.85 });
            const swordLines = new THREE.Line(lineGeo, lineMat);
            swordConstellationGroup.add(swordLines);

            // Đặt các ngôi sao sáng chói tại các điểm mốc của đường kiếm
            swordPoints.forEach((pt, idx) => {
                const starGeo = new THREE.SphereGeometry(idx === 6 ? 1.8 : 1.2, 16, 16);
                const starMat = new THREE.MeshBasicMaterial({ color: idx === 6 ? 0xffffff : 0x7dd3fc });
                const starMesh = new THREE.Mesh(starGeo, starMat);
                starMesh.position.copy(pt);
                swordConstellationGroup.add(starMesh);
            });

            // Đặt chòm sao lơ lửng phía xa trên quỹ đạo không gian
            swordConstellationGroup.position.set(110, 45, -130);
            swordConstellationGroup.rotation.z = Math.PI / 5;
            spaceGroup.add(swordConstellationGroup);
        }

        function toggleSwordConstellation() {
            isSwordVisible = !isSwordVisible;
            const btn = document.getElementById('swordBtn');
            btn.innerText = isSwordVisible ? '⚔️ Kiếm Sao: ON' : '⚔️ Kiếm Sao: OFF';
            btn.style.borderColor = isSwordVisible ? '#38bdf8' : '';

            buildSwordConstellation();
            if (swordConstellationGroup) swordConstellationGroup.visible = isSwordVisible;
        }

        // =========================================================================
        // HỆ THỐNG GAME 2D TRÊN TRÁI ĐẤT & SỬA LỖI NÚT THỦ LẠI
        // =========================================================================
        let gameMode = 'SPACE';
        let spaceGroup, moonEscapeGroup;
        let rocket3D, rocketFlame3D, escapeCurve3D, escapeProgress = 0;
        let explodingEarthMesh, earthDebrisList = [];
        let isEarthExploding = false, explosionProgress = 0;
        let canvas2D, ctx2D, gameInterval, missionTimerInterval;
        let missionState = 'SELECT';
        let currentDifficulty = 'easy';
        let missionStage = 1;
        let progressCount = 0;
        let timeLeft = 90; // 90 giây (1 phút 30 giây)

        let player2D = { x: 80, y: 80, radius: 13, speed: 4.5, color: '#38bdf8' };
        let keys2D = {};
        let items2D = [];
        let meteors2D = [];

        window.addEventListener('keydown', e => keys2D[e.key] = true);
        window.addEventListener('keyup', e => keys2D[e.key] = false);

        function setup2DCanvas() {
            canvas2D = document.getElementById('game2DCanvas');
            canvas2D.width = canvas2D.parentElement.clientWidth;
            canvas2D.height = canvas2D.parentElement.clientHeight;
            ctx2D = canvas2D.getContext('2d');
        }

        function resetToDifficulty() {
            missionState = 'SELECT';
            document.getElementById('difficultyOverlay').classList.remove('hidden');
            document.getElementById('gameOverModal').classList.add('hidden');
            document.getElementById('gameMsg').classList.add('hidden');
            document.getElementById('alertBanner30s').classList.add('hidden');
            document.getElementById('earthUI').style.backgroundColor = '#030712';
            document.getElementById('hudTopBar').style.backgroundColor = '';
            document.getElementById('missionTimer').innerText = "01:30";
            document.getElementById('missionText').innerText = "Chọn cấp độ khó để bắt đầu!";
        }

        function startMission(diff) {
            currentDifficulty = diff;
            missionState = 'PLAYING';
            missionStage = 1;
            progressCount = 0;
            timeLeft = 90;

            const diffNames = { easy: 'DỄ', normal: 'BÌNH THƯỜNG', hard: 'SIÊU KHÓ' };
            document.getElementById('diffLabel').innerText = diffNames[diff];
            document.getElementById('difficultyOverlay').classList.add('hidden');
            document.getElementById('gameOverModal').classList.add('hidden');
            document.getElementById('gameMsg').classList.remove('hidden');
            document.getElementById('alertBanner30s').classList.add('hidden');
            document.getElementById('earthUI').style.backgroundColor = '#030712';

            player2D.x = 70;
            player2D.y = canvas2D.height / 2;

            spawnMissionItems();
            spawnMeteors();
            updateMissionHUD();

            if (gameInterval) clearInterval(gameInterval);
            gameInterval = setInterval(gameLoop2D, 1000 / 60);

            if (missionTimerInterval) clearInterval(missionTimerInterval);
            missionTimerInterval = setInterval(() => {
                if (missionState === 'PLAYING' && gameMode === 'EARTH') {
                    timeLeft--;
                    let m = String(Math.floor(timeLeft / 60)).padStart(2, '0');
                    let s = String(timeLeft % 60).padStart(2, '0');
                    document.getElementById('missionTimer').innerText = `${m}:${s}`;

                    if (timeLeft <= 30 && timeLeft > 10) {
                        const banner = document.getElementById('alertBanner30s');
                        banner.classList.remove('hidden');
                        banner.innerText = `🚨 CẢNH BÁO: CÒN ${timeLeft} GIÂY TRƯỚC KHI TRÁI ĐẤT NỔ! 🚨`;
                        playEmergencySiren();
                    }
                    else if (timeLeft <= 10 && timeLeft > 0) {
                        const banner = document.getElementById('alertBanner30s');
                        banner.classList.remove('hidden');
                        banner.innerText = `⚠️ BÁO ĐỘNG ĐỎ: CÒN ${timeLeft} GIÂY - SẮP BÙNG NỔ! ⚠️`;
                        document.getElementById('earthUI').style.backgroundColor = '#450a0a';
                        document.getElementById('gameBoxContainer').style.borderColor = '#ef4444';
                        playEmergencySiren();
                    }
                    else if (timeLeft <= 0) {
                        triggerGameOverDoom();
                    }
                }
            }, 1000);
        }

        function triggerGameOverDoom() {
            missionState = 'GAMEOVER';
            clearInterval(gameInterval);
            clearInterval(missionTimerInterval);
            document.getElementById('alertBanner30s').classList.add('hidden');

            playExplosionBoom();

            triggerWarpEffect(() => {
                gameMode = 'MOON_ESCAPE_3D';
                document.getElementById('earthUI').classList.add('hidden');
                document.getElementById('moonEscapeUI').classList.remove('hidden');
                document.getElementById('victoryModal3D').classList.add('hidden');
                document.getElementById('launchStatusText').innerText = "💥 BÙM! Trái Đất đã bùng nổ hoàn toàn do hết thời gian...";

                spaceGroup.visible = false;
                if (!moonEscapeGroup) buildMoonEscapeScene3D();
                moonEscapeGroup.visible = true;
                if (starDomeGroup) scene.add(starDomeGroup);

                isEarthExploding = true;
                explosionProgress = 0;
                if (rocket3D) rocket3D.visible = false;

                // Đảm bảo bật hiện bảng Thất Bại có nút Thử Lại ngay sau hiệu ứng nổ
                setTimeout(() => {
                    document.getElementById('gameOverModal').classList.remove('hidden');
                }, 1200);
            });
        }

        function getStageTargetCount() {
            if (missionStage === 4) return 1;
            if (currentDifficulty === 'hard') {
                return missionStage === 1 ? 4 : missionStage === 2 ? 4 : 5;
            }
            return missionStage === 1 ? 3 : missionStage === 2 ? 3 : 3;
        }

        function updateMissionHUD() {
            let target = getStageTargetCount();
            let texts = [
                "",
                `Thu thập Bình Nước & Năng Lượng (${progressCount}/${target})`,
                `Sửa chữa Trạm Khiên Từ Trường (${progressCount}/${target})`,
                `Giải cứu Nhà Khoa Học (${progressCount}/${target})`,
                `Chạy ngay đến Bệ Phóng Tên Lửa! (${progressCount}/${target})`
            ];
            document.getElementById('missionText').innerText = texts[missionStage];
        }

        function spawnMissionItems() {
            items2D = [];
            let count = getStageTargetCount();
            let type = missionStage === 1 ? 'water' : missionStage === 2 ? 'shield' : missionStage === 3 ? 'scientist' : 'rocket';

            for (let i = 0; i < count; i++) {
                items2D.push({
                    x: 140 + Math.random() * (canvas2D.width - 200),
                    y: 60 + Math.random() * (canvas2D.height - 120),
                    radius: missionStage === 4 ? 22 : 15,
                    type: type,
                    collected: false
                });
            }
        }

        function spawnMeteors() {
            meteors2D = [];
            if (currentDifficulty === 'easy') return;
            let count = currentDifficulty === 'normal' ? 5 : 12;
            let speedBase = currentDifficulty === 'normal' ? 2.5 : 4.5;

            for (let i = 0; i < count; i++) {
                meteors2D.push({
                    x: 160 + Math.random() * (canvas2D.width - 200),
                    y: Math.random() * canvas2D.height,
                    radius: 10 + Math.random() * 8,
                    vx: (Math.random() - 0.5) * speedBase,
                    vy: (Math.random() > 0.5 ? 1 : -1) * (speedBase + Math.random() * 1.5)
                });
            }
        }

        function gameLoop2D() {
            if (gameMode !== 'EARTH' || missionState !== 'PLAYING') return;

            if ((keys2D['ArrowUp'] || keys2D['w'] || keys2D['W']) && player2D.y > 20) player2D.y -= player2D.speed;
            if ((keys2D['ArrowDown'] || keys2D['s'] || keys2D['S']) && player2D.y < canvas2D.height - 20) player2D.y += player2D.speed;
            if ((keys2D['ArrowLeft'] || keys2D['a'] || keys2D['A']) && player2D.x > 20) player2D.x -= player2D.speed;
            if ((keys2D['ArrowRight'] || keys2D['d'] || keys2D['D']) && player2D.x < canvas2D.width - 20) player2D.x += player2D.speed;

            meteors2D.forEach(m => {
                m.x += m.vx;
                m.y += m.vy;
                if (m.x < 130 || m.x > canvas2D.width - 20) m.vx *= -1;
                if (m.y < 20 || m.y > canvas2D.height - 20) m.vy *= -1;

                if (Math.hypot(player2D.x - m.x, player2D.y - m.y) < player2D.radius + m.radius) {
                    player2D.x = 70;
                    player2D.y = canvas2D.height / 2;
                }
            });

            items2D.forEach(item => {
                if (!item.collected) {
                    if (Math.hypot(player2D.x - item.x, player2D.y - item.y) < player2D.radius + item.radius) {
                        item.collected = true;
                        progressCount++;
                        checkMissionProgress();
                    }
                }
            });

            render2DPlaying();
        }

        function checkMissionProgress() {
            let target = getStageTargetCount();
            if (progressCount >= target) {
                missionStage++;
                progressCount = 0;
                if (missionStage > 4) {
                    trigger3DMoonEscape();
                } else {
                    updateMissionHUD();
                    spawnMissionItems();
                }
            } else {
                updateMissionHUD();
            }
        }

        function render2DPlaying() {
            ctx2D.fillStyle = timeLeft <= 10 ? '#1a0505' : '#090d16';
            ctx2D.fillRect(0, 0, canvas2D.width, canvas2D.height);

            ctx2D.strokeStyle = timeLeft <= 10 ? 'rgba(239, 68, 68, 0.25)' : 'rgba(244, 63, 94, 0.12)';
            ctx2D.lineWidth = 1;
            for (let x = 0; x < canvas2D.width; x += 40) {
                ctx2D.beginPath(); ctx2D.moveTo(x, 0); ctx2D.lineTo(x, canvas2D.height); ctx2D.stroke();
            }
            for (let y = 0; y < canvas2D.height; y += 40) {
                ctx2D.beginPath(); ctx2D.moveTo(0, y); ctx2D.lineTo(canvas2D.width, y); ctx2D.stroke();
            }

            ctx2D.fillStyle = timeLeft <= 10 ? 'rgba(239, 68, 68, 0.15)' : 'rgba(56, 189, 248, 0.08)';
            ctx2D.fillRect(0, 0, 120, canvas2D.height);

            items2D.forEach(item => {
                if (!item.collected) {
                    ctx2D.save();
                    ctx2D.beginPath();
                    ctx2D.arc(item.x, item.y, item.radius, 0, Math.PI * 2);
                    ctx2D.fillStyle = item.type === 'water' ? '#38bdf8' : item.type === 'shield' ? '#f59e0b' : item.type === 'scientist' ? '#34d399' : '#a855f7';
                    ctx2D.shadowColor = ctx2D.fillStyle;
                    ctx2D.shadowBlur = 15;
                    ctx2D.fill();
                    ctx2D.font = item.type === 'rocket' ? '18px Arial' : '14px Arial';
                    ctx2D.textAlign = 'center';
                    ctx2D.fillText(item.type === 'water' ? '💧' : item.type === 'shield' ? '⚡' : item.type === 'scientist' ? '🧑‍🚀' : '🚀', item.x, item.y + 5);
                    ctx2D.restore();
                }
            });

            meteors2D.forEach(m => {
                ctx2D.save();
                ctx2D.beginPath();
                ctx2D.arc(m.x, m.y, m.radius, 0, Math.PI * 2);
                ctx2D.fillStyle = '#f43f5e';
                ctx2D.shadowColor = '#f43f5e';
                ctx2D.shadowBlur = 15;
                ctx2D.fill();
                ctx2D.restore();
            });

            ctx2D.save();
            ctx2D.beginPath();
            ctx2D.arc(player2D.x, player2D.y, player2D.radius, 0, Math.PI * 2);
            ctx2D.fillStyle = player2D.color;
            ctx2D.shadowColor = '#38bdf8';
            ctx2D.shadowBlur = 20;
            ctx2D.fill();
            ctx2D.lineWidth = 2;
            ctx2D.strokeStyle = '#ffffff';
            ctx2D.stroke();
            ctx2D.restore();
        }

        // =========================================================================
        // HOẠT ẢNH 3D NỔ TUNG TRÁI ĐẤT & TRỐN THOÁT
        // =========================================================================
        function buildMoonEscapeScene3D() {
            moonEscapeGroup = new THREE.Group();
            scene.add(moonEscapeGroup);

            const dirLight = new THREE.DirectionalLight(0xffffff, 2.5);
            dirLight.position.set(40, 50, 40);
            moonEscapeGroup.add(dirLight, new THREE.AmbientLight(0x1e293b, 0.9));

            const earthMaps = generatePlanetMaps('earth');
            explodingEarthMesh = new THREE.Mesh(
                new THREE.SphereGeometry(12, 48, 48),
                new THREE.MeshStandardMaterial({ map: earthMaps.map, roughness: 0.6 })
            );
            explodingEarthMesh.position.set(-28, -12, 0);
            moonEscapeGroup.add(explodingEarthMesh);

            earthDebrisList = [];
            for (let i = 0; i < 45; i++) {
                const debris = new THREE.Mesh(
                    new THREE.DodecahedronGeometry(0.6 + Math.random() * 1.0),
                    new THREE.MeshStandardMaterial({ color: Math.random() > 0.4 ? 0xef4444 : 0xfbbf24, emissive: 0xb91c1c })
                );
                debris.position.set(-28, -12, 0);
                const speed = 0.06 + Math.random() * 0.18;
                const angle1 = Math.random() * Math.PI * 2;
                const angle2 = Math.random() * Math.PI;
                debris.userData = {
                    vx: Math.sin(angle2) * Math.cos(angle1) * speed,
                    vy: Math.cos(angle2) * speed,
                    vz: Math.sin(angle2) * Math.sin(angle1) * speed
                };
                moonEscapeGroup.add(debris);
                earthDebrisList.push(debris);
            }

            const moonMesh = new THREE.Mesh(
                new THREE.SphereGeometry(7, 64, 64),
                new THREE.MeshStandardMaterial({ color: 0xe2e8f0, roughness: 0.85, metalness: 0.1 })
            );
            moonMesh.position.set(28, 16, -5);
            moonEscapeGroup.add(moonMesh);

            escapeCurve3D = new THREE.QuadraticBezierCurve3(
                new THREE.Vector3(-22, -5, 2),
                new THREE.Vector3(0, 18, 8),
                new THREE.Vector3(24, 14, -3)
            );

            rocket3D = new THREE.Group();
            const body = new THREE.Mesh(
                new THREE.CylinderGeometry(0.7, 0.9, 3.8, 24),
                new THREE.MeshStandardMaterial({ color: 0xf8fafc, metalness: 0.4, roughness: 0.3 })
            );
            const nose = new THREE.Mesh(
                new THREE.ConeGeometry(0.7, 1.5, 24),
                new THREE.MeshStandardMaterial({ color: 0xf43f5e, roughness: 0.4 })
            );
            nose.position.y = 2.65;

            const finGeo = new THREE.BoxGeometry(0.15, 1.2, 1.0);
            const finMat = new THREE.MeshStandardMaterial({ color: 0x0284c7 });
            const fin1 = new THREE.Mesh(finGeo, finMat); fin1.position.set(0.85, -1.3, 0);
            const fin2 = new THREE.Mesh(finGeo, finMat); fin2.position.set(-0.85, -1.3, 0);
            const fin3 = new THREE.Mesh(new THREE.BoxGeometry(1.0, 1.2, 0.15), finMat); fin3.position.set(0, -1.3, 0.85);
            const fin4 = new THREE.Mesh(new THREE.BoxGeometry(1.0, 1.2, 0.15), finMat); fin4.position.set(0, -1.3, -0.85);

            rocketFlame3D = new THREE.Mesh(
                new THREE.ConeGeometry(0.65, 2.8, 16).rotateX(Math.PI),
                new THREE.MeshBasicMaterial({ color: 0xf59e0b })
            );
            rocketFlame3D.position.y = -3.2;

            rocket3D.add(body, nose, fin1, fin2, fin3, fin4, rocketFlame3D);
            moonEscapeGroup.add(rocket3D);
        }

        function trigger3DMoonEscape() {
            missionState = 'LAUNCHING_3D';
            playExplosionBoom();

            triggerWarpEffect(() => {
                gameMode = 'MOON_ESCAPE_3D';
                document.getElementById('earthUI').classList.add('hidden');
                document.getElementById('moonEscapeUI').classList.remove('hidden');
                document.getElementById('victoryModal3D').classList.add('hidden');
                document.getElementById('launchStatusText').innerText = "💥 BÙM! Trái Đất bùng nổ! Tên lửa đang phóng lên Mặt Trăng...";

                spaceGroup.visible = false;
                if (!moonEscapeGroup) buildMoonEscapeScene3D();
                moonEscapeGroup.visible = true;
                if (starDomeGroup) scene.add(starDomeGroup);

                escapeProgress = 0;
                isEarthExploding = true;
                explosionProgress = 0;
                if (rocket3D) rocket3D.visible = true;
                if (bloomPass) bloomPass.strength = 1.2;
            });
        }

        function replayFrom3D() {
            if (moonEscapeGroup) moonEscapeGroup.visible = false;
            if (starDomeGroup && spaceGroup) spaceGroup.add(starDomeGroup);
            document.getElementById('moonEscapeUI').classList.add('hidden');
            enterEarthMode();
        }

        function finishToUniverse() {
            triggerWarpEffect(() => {
                if (moonEscapeGroup) moonEscapeGroup.visible = false;
                if (starDomeGroup && spaceGroup) spaceGroup.add(starDomeGroup);
                document.getElementById('moonEscapeUI').classList.add('hidden');
                gameMode = 'SPACE';
                spaceGroup.visible = true;
                if (bloomPass) bloomPass.strength = 1.4;
                if (renderer) renderer.toneMappingExposure = userBrightness;
                document.getElementById('spaceUI').classList.remove('hidden');
                selectPlanet('earth');
                targetCamDist = 160;

                isMusicPlaying = true;
                if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
                document.getElementById('audioBtn').innerText = '🔊 Music: ON';
            });
        }

        function triggerWarpEffect(callback) {
            const overlay = document.getElementById('warpOverlay');
            overlay.style.opacity = '1';
            setTimeout(() => {
                callback();
                setTimeout(() => { overlay.style.opacity = '0'; }, 300);
            }, 400);
        }

        function enterEarthMode() {
            isMusicPlaying = false;
            document.getElementById('audioBtn').innerText = '🔇 Muted';

            triggerWarpEffect(() => {
                gameMode = 'EARTH';
                spaceGroup.visible = false;
                if (moonEscapeGroup) moonEscapeGroup.visible = false;
                if (bloomPass) bloomPass.strength = 0.35;

                document.getElementById('spaceUI').classList.add('hidden');
                document.getElementById('earthUI').classList.remove('hidden');

                setTimeout(() => {
                    setup2DCanvas();
                    resetToDifficulty();
                }, 80);
            });
        }

        function exitEarthMode() {
            triggerWarpEffect(() => {
                gameMode = 'SPACE';
                spaceGroup.visible = true;
                if (moonEscapeGroup) moonEscapeGroup.visible = false;
                if (bloomPass) bloomPass.strength = 1.4;
                if (renderer) renderer.toneMappingExposure = userBrightness;

                document.getElementById('earthUI').classList.add('hidden');
                document.getElementById('spaceUI').classList.remove('hidden');
                selectPlanet('earth');

                isMusicPlaying = true;
                if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
                document.getElementById('audioBtn').innerText = '🔊 Music: ON';
            });
        }

        // =========================================================================
        // ENGINE 3D VŨ TRỤ THREE.JS
        // =========================================================================
        let scene, camera, renderer, composer, bloomPass;
        let planets = {};
        let sunUniforms, starDomeGroup, asteroidBelt, spacetimeGrid, cosmicDust;
        let earthFireFlames = [];
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, prevMouse = { x: 0, y: 0 };
        let camDist = 95, targetCamDist = 95;
        let camAngleX = 0.35, camAngleY = 0.25;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0;
        let userBrightness = 1.25;
        let isFreeFly = false, freeFlyAngle = 0;

        const planetDB = {
            sun: { name: "MẶT TRỜI (SOL)", desc: "Khối cầu plasma sôi sục khổng lồ.", dist: "0 AU", temp: "5,500 °C", zoom: 90 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Hành tinh gần Mặt Trời nhất với bề mặt đá bazan xám bạc.", dist: "0.39 AU", temp: "430 °C", zoom: 20 },
            venus: { name: "SAO KIM (VENUS)", desc: "Khí quyển siêu đặc cuộn xoáy mây axit sulfuric.", dist: "0.72 AU", temp: "462 °C", zoom: 22 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Trái Đất xanh đang bùng phát các đám cháy lớn trên bề mặt! Đổ bộ ngay để cứu lấy hành tinh!", dist: "1.00 AU", temp: "68 °C 🔥", zoom: 22 },
            mars: { name: "SAO HỎA (MARS)", desc: "Sa mạc oxit sắt đỏ rực với chỏm băng cực.", dist: "1.52 AU", temp: "-63 °C", zoom: 20 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Khí quyển dải băng tầng khí động học & Vết Đỏ Lớn.", dist: "5.20 AU", temp: "-110 °C", zoom: 32 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Hệ thống vành đai băng đa tầng sắc nét.", dist: "9.58 AU", temp: "-140 °C", zoom: 38 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng khí metan xanh ngọc bích.", dist: "19.22 AU", temp: "-195 °C", zoom: 28 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Thế giới băng giá xanh thẫm với gió siêu thanh.", dist: "30.05 AU", temp: "-200 °C", zoom: 28 }
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

            if (type === 'mercury') {
                ctx.fillStyle = '#cbd5e1'; ctx.fillRect(0, 0, w, h);
                ctx.fillStyle = '#64748b';
                for (let i = 0; i < 200; i++) {
                    ctx.beginPath();
                    ctx.arc(Math.random() * w, Math.random() * h, 6 + Math.random() * 25, 0, Math.PI * 2);
                    ctx.fill();
                }
            } else if (type === 'venus') {
                ctx.fillStyle = '#f59e0b'; ctx.fillRect(0, 0, w, h);
            } else if (type === 'earth') {
                ctx.fillStyle = '#0284c7'; ctx.fillRect(0, 0, w, h);
                ctx.fillStyle = '#16a34a';
                ctx.beginPath(); ctx.ellipse(500, 400, 260, 150, 0.2, 0, Math.PI * 2); ctx.fill();
                ctx.beginPath(); ctx.ellipse(1200, 520, 340, 200, -0.1, 0, Math.PI * 2); ctx.fill();
                ctx.beginPath(); ctx.ellipse(1750, 350, 220, 140, 0.3, 0, Math.PI * 2); ctx.fill();
            } else if (type === 'clouds') {
                ctx.clearRect(0, 0, w, h);
                ctx.fillStyle = 'rgba(255,255,255,0.45)';
                for (let i = 0; i < 150; i++) ctx.fillRect(Math.random() * w, Math.random() * h, 180, 25);
            } else if (type === 'mars') {
                ctx.fillStyle = '#9a3412'; ctx.fillRect(0, 0, w, h);
            } else if (type === 'jupiter') {
                ctx.fillStyle = '#d97706'; ctx.fillRect(0, 0, w, h);
            } else if (type === 'saturn') {
                ctx.fillStyle = '#ca8a04'; ctx.fillRect(0, 0, w, h);
            } else if (type === 'uranus') {
                ctx.fillStyle = '#06b6d4'; ctx.fillRect(0, 0, w, h);
            } else if (type === 'neptune') {
                ctx.fillStyle = '#1d4ed8'; ctx.fillRect(0, 0, w, h);
            }
            return { map: new THREE.CanvasTexture(diffCanvas), bumpMap: new THREE.CanvasTexture(bumpCanvas) };
        }

        function createSaturnRingTexture() {
            const canvas = document.createElement('canvas'); canvas.width = 1024; canvas.height = 64;
            const ctx = canvas.getContext('2d');
            const grad = ctx.createLinearGradient(0, 0, 1024, 0);
            grad.addColorStop(0, 'rgba(234, 179, 8, 0)'); grad.addColorStop(0.5, 'rgba(234, 179, 8, 0.95)'); grad.addColorStop(1, 'rgba(234, 179, 8, 0)');
            ctx.fillStyle = grad; ctx.fillRect(0, 0, 1024, 64);
            return new THREE.CanvasTexture(canvas);
        }

        function createStarDome() {
            starDomeGroup = new THREE.Group();

            const starCount = 200000;
            const starGeo = new THREE.BufferGeometry();
            const starPos = new Float32Array(starCount * 3);
            const starCol = new Float32Array(starCount * 3);
            const colors = [
                new THREE.Color(0x38bdf8), 
                new THREE.Color(0xa855f7), 
                new THREE.Color(0xfde047), 
                new THREE.Color(0xffffff), 
                new THREE.Color(0xf43f5e),
                new THREE.Color(0x60a5fa),
                new THREE.Color(0x34d399),
                new THREE.Color(0xfb923c)
            ];

            for (let i = 0; i < starCount; i++) {
                const u = Math.random();
                const v = Math.random();
                const theta = u * 2.0 * Math.PI;
                const phi = Math.acos(2.0 * v - 1.0);
                const r = 800 + Math.random() * 2700;

                starPos[i * 3] = r * Math.sin(phi) * Math.cos(theta);
                starPos[i * 3 + 1] = r * Math.sin(phi) * Math.sin(theta);
                starPos[i * 3 + 2] = r * Math.cos(phi);

                const c = colors[Math.floor(Math.random() * colors.length)];
                const bright = 0.75 + Math.random() * 0.45;
                starCol[i * 3] = c.r * bright; 
                starCol[i * 3 + 1] = c.g * bright; 
                starCol[i * 3 + 2] = c.b * bright;
            }
            starGeo.setAttribute('position', new THREE.BufferAttribute(starPos, 3));
            starGeo.setAttribute('color', new THREE.BufferAttribute(starCol, 3));
            
            const starMesh = new THREE.Points(
                starGeo, 
                new THREE.PointsMaterial({ size: 3.5, vertexColors: true, transparent: true, opacity: 1.0, blending: THREE.AdditiveBlending, depthWrite: false })
            );
            starDomeGroup.add(starMesh);

            cosmicDust = new THREE.Points(
                new THREE.BufferGeometry().setAttribute('position', new THREE.BufferAttribute(new Float32Array(70000), 3)),
                new THREE.PointsMaterial({ color: 0x818cf8, size: 2.0, transparent: true, opacity: 0.8, blending: THREE.AdditiveBlending, depthWrite: false })
            );
            
            const dustPos = cosmicDust.geometry.attributes.position.array;
            for (let i = 0; i < 70000; i += 3) {
                const u = Math.random();
                const v = Math.random();
                const theta = u * 2.0 * Math.PI;
                const phi = Math.acos(2.0 * v - 1.0);
                const r = 1000 + Math.random() * 2000;
                dustPos[i] = r * Math.sin(phi) * Math.cos(theta);
                dustPos[i+1] = (Math.random() - 0.5) * 600;
                dustPos[i+2] = r * Math.sin(phi) * Math.sin(theta);
            }
            cosmicDust.geometry.attributes.position.needsUpdate = true;

            spaceGroup.add(cosmicDust, starDomeGroup);
        }

        function createAsteroidBelt() {
            asteroidBelt = new THREE.InstancedMesh(new THREE.DodecahedronGeometry(0.2, 0), new THREE.MeshStandardMaterial({ color: 0x94a3b8 }), 2500);
            const dummy = new THREE.Object3D();
            for (let i = 0; i < 2500; i++) {
                const angle = Math.random() * Math.PI * 2; const radius = 50 + Math.random() * 8;
                dummy.position.set(Math.cos(angle) * radius, (Math.random() - 0.5) * 1.8, Math.sin(angle) * radius);
                dummy.updateMatrix(); asteroidBelt.setMatrixAt(i, dummy.matrix);
            }
            spaceGroup.add(asteroidBelt);
        }

        function createSpacetimeGrid() {
            spacetimeGrid = new THREE.Mesh(new THREE.PlaneGeometry(260, 260, 48, 48).rotateX(-Math.PI / 2), new THREE.MeshBasicMaterial({ color: 0x0ea5e9, wireframe: true, transparent: true, opacity: 0.16 }));
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
            renderer.toneMappingExposure = userBrightness;
            container.appendChild(renderer.domElement);

            const renderScene = new THREE.RenderPass(scene, camera);
            bloomPass = new THREE.UnrealBloomPass(new THREE.Vector2(window.innerWidth, window.innerHeight), 1.5, 0.8, 0.2);
            composer = new THREE.EffectComposer(renderer);
            composer.addPass(renderScene); composer.addPass(bloomPass);

            const sunLight = new THREE.PointLight(0xfff5e4, 4.5, 2500, 0.2);
            spaceGroup.add(sunLight, new THREE.AmbientLight(0x0f172a, 0.45));

            const flareTex = createFlareTextures();
            const lensflare = new THREE.Lensflare();
            lensflare.addElement(new THREE.LensflareElement(flareTex.streak, 850, 0, new THREE.Color(0x7dd3fc)));
            lensflare.addElement(new THREE.LensflareElement(flareTex.orb, 180, 0, new THREE.Color(0xfef08a)));
            sunLight.add(lensflare);

            createStarDome();
            createAsteroidBelt();
            createSpacetimeGrid();

            sunUniforms = { uTime: { value: 0 } };
            const sun = new THREE.Mesh(new THREE.SphereGeometry(9.5, 64, 64), new THREE.ShaderMaterial({ uniforms: sunUniforms, vertexShader: sunVertexShader, fragmentShader: sunFragmentShader }));
            sun.add(new THREE.Mesh(new THREE.SphereGeometry(12.0, 64, 64), createAtmosphereMaterial(0xff8c00, 1.4, 3.0)));
            spaceGroup.add(sun);
            planets['sun'] = { mesh: sun, distance: 0, speed: 0 };

            const planetConfigs = [
                { id: 'mercury', size: 1.9, dist: 20, speed: 0.022, tilt: 0.03, atmos: 0xe2e8f0 },
                { id: 'venus', size: 2.3, dist: 27, speed: 0.016, tilt: 3.09, atmos: 0xfbbf24 },
                { id: 'earth', size: 2.6, dist: 36, speed: 0.012, tilt: 0.41, atmos: 0x38bdf8, clouds: true, hasFire: true },
                { id: 'mars', size: 1.9, dist: 45, speed: 0.009, tilt: 0.44, atmos: 0xf97316 },
                { id: 'jupiter', size: 5.8, dist: 64, speed: 0.006, tilt: 0.05, atmos: 0xfdba74 },
                { id: 'saturn', size: 4.8, dist: 82, speed: 0.004, tilt: 0.47, atmos: 0xfef08a, rings: true },
                { id: 'uranus', size: 3.2, dist: 100, speed: 0.003, tilt: 1.71, atmos: 0x22d3ee },
                { id: 'neptune', size: 3.1, dist: 116, speed: 0.002, tilt: 0.49, atmos: 0x3b82f6 }
            ];

            planetConfigs.forEach(cfg => {
                const orbitGeo = new THREE.RingGeometry(cfg.dist - 0.14, cfg.dist + 0.14, 220);
                const orbitRing = new THREE.Mesh(orbitGeo, new THREE.MeshBasicMaterial({ color: cfg.atmos || 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.55, blending: THREE.AdditiveBlending }));
                orbitRing.rotation.x = Math.PI / 2;
                spaceGroup.add(orbitRing);

                const maps = generatePlanetMaps(cfg.id);
                const pMat = new THREE.MeshStandardMaterial({ map: maps.map, bumpMap: maps.bumpMap, bumpScale: 0.045, roughness: 0.72, metalness: 0.08 });
                const mesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size, 64, 64), pMat);
                mesh.rotation.z = cfg.tilt;
                spaceGroup.add(mesh);

                if (cfg.atmos) {
                    mesh.add(new THREE.Mesh(new THREE.SphereGeometry(cfg.size * 1.12, 48, 48), createAtmosphereMaterial(cfg.atmos, 1.2, 3.6)));
                }

                if (cfg.clouds) {
                    const cloudMaps = generatePlanetMaps('clouds');
                    const cloudMesh = new THREE.Mesh(new THREE.SphereGeometry(cfg.size * 1.022, 64, 64), new THREE.MeshStandardMaterial({ map: cloudMaps.map, transparent: true, opacity: 0.88, depthWrite: false }));
                    mesh.add(cloudMesh);
                    mesh.userData.cloudMesh = cloudMesh;
                }

                if (cfg.hasFire) {
                    earthFireFlames = [];
                    for (let i = 0; i < 36; i++) {
                        const u = Math.random();
                        const v = Math.random();
                        const theta = u * 2.0 * Math.PI;
                        const phi = Math.acos(2.0 * v - 1.0);

                        const normal = new THREE.Vector3(
                            Math.sin(phi) * Math.cos(theta),
                            Math.sin(phi) * Math.sin(theta),
                            Math.cos(phi)
                        ).normalize();

                        const flameHeight = 0.5 + Math.random() * 0.7;
                        const flame = new THREE.Mesh(
                            new THREE.ConeGeometry(0.18 + Math.random() * 0.14, flameHeight, 8),
                            new THREE.MeshBasicMaterial({
                                color: Math.random() > 0.4 ? 0xf97316 : 0xfacc15,
                                transparent: true,
                                opacity: 0.9
                            })
                        );

                        flame.position.copy(normal).multiplyScalar(cfg.size + flameHeight * 0.35);
                        flame.quaternion.setFromUnitVectors(new THREE.Vector3(0, 1, 0), normal);
                        flame.userData = {
                            baseScale: 0.8 + Math.random() * 0.5,
                            phase: Math.random() * Math.PI * 2
                        };

                        mesh.add(flame);
                        earthFireFlames.push(flame);
                    }
                }

                if (cfg.rings) {
                    const rGeo = new THREE.RingGeometry(cfg.size * 1.4, cfg.size * 2.6, 128);
                    const posAttr = rGeo.attributes.position;
                    const uvAttr = rGeo.attributes.uv;
                    const v3 = new THREE.Vector3();
                    for (let i = 0; i < posAttr.count; i++) {
                        v3.fromBufferAttribute(posAttr, i);
                        const u = (v3.length() - cfg.size * 1.4) / (cfg.size * 1.2);
                        uvAttr.setXY(i, u, 0.5);
                    }
                    const ringMesh = new THREE.Mesh(rGeo, new THREE.MeshStandardMaterial({ map: createSaturnRingTexture(), side: THREE.DoubleSide, transparent: true, opacity: 0.95, roughness: 0.5 }));
                    ringMesh.rotation.x = Math.PI / 2;
                    mesh.add(ringMesh);
                }

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
            window.addEventListener('wheel', e => { targetCamDist = Math.max(2.5, Math.min(650, targetCamDist + e.deltaY * 0.08)); });
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
            const nowSec = performance.now() * 0.005;

            if (gameMode === 'MOON_ESCAPE_3D') {
                if (starDomeGroup) starDomeGroup.rotation.y += 0.00015;

                if (isEarthExploding) {
                    explosionProgress += 0.015;
                    if (explodingEarthMesh) explodingEarthMesh.scale.addScalar(0.08);
                    earthDebrisList.forEach(d => {
                        d.position.x += d.userData.vx;
                        d.position.y += d.userData.vy;
                        d.position.z += d.userData.vz;
                        d.rotation.x += 0.05;
                    });
                }

                if (rocket3D && rocket3D.visible && escapeCurve3D) {
                    if (escapeProgress < 1) {
                        escapeProgress += 0.004;
                        if (escapeProgress >= 1) {
                            escapeProgress = 1;
                            rocketFlame3D.visible = false;
                            document.getElementById('launchStatusText').innerText = "✅ Đã hạ cánh an toàn xuống Mặt Trăng!";
                            const diffNames = { easy: 'DỄ', normal: 'BÌNH THƯỜNG', hard: 'SIÊU KHÓ' };
                            document.getElementById('winDiff3D').innerText = diffNames[currentDifficulty];
                            let m = String(Math.floor(timeLeft / 60)).padStart(2, '0');
                            let s = String(timeLeft % 60).padStart(2, '0');
                            document.getElementById('winTime3D').innerText = `${m}:${s}`;
                            document.getElementById('victoryModal3D').classList.remove('hidden');
                        }
                    }

                    const pos = escapeCurve3D.getPointAt(escapeProgress);
                    rocket3D.position.copy(pos);

                    if (escapeProgress < 0.99) {
                        const tangent = escapeCurve3D.getTangentAt(escapeProgress).normalize();
                        rocket3D.quaternion.setFromUnitVectors(new THREE.Vector3(0, 1, 0), tangent);
                        rocketFlame3D.scale.y = 0.8 + Math.random() * 0.6;
                    }

                    camera.position.set(pos.x * 0.4, pos.y * 0.4 + 12, 68);
                    camera.lookAt(pos.x * 0.6, pos.y * 0.6, 0);
                } else if (!rocket3D || !rocket3D.visible) {
                    camera.position.set(-28, -2, 45);
                    camera.lookAt(-28, -12, 0);
                }

            } else if (gameMode === 'SPACE') {
                if (sunUniforms) sunUniforms.uTime.value += 0.015 * timeSpeed;
                if (starDomeGroup) starDomeGroup.rotation.y += 0.0001 * timeSpeed;
                if (cosmicDust) cosmicDust.rotation.y += 0.00025 * timeSpeed;
                if (asteroidBelt && asteroidBelt.visible) asteroidBelt.rotation.y += 0.0018 * timeSpeed;

                earthFireFlames.forEach(f => {
                    const flicker = f.userData.baseScale + Math.sin(nowSec * 3.5 + f.userData.phase) * 0.35;
                    f.scale.set(1, Math.max(0.4, flicker), 1);
                });

                if (isFreeFly) {
                    freeFlyAngle += 0.003 * timeSpeed;
                    currentLookAt.set(0, 0, 0);
                    camDist = 130 + Math.sin(freeFlyAngle * 0.5) * 35;
                    camera.position.x = Math.cos(freeFlyAngle) * camDist;
                    camera.position.y = 40 + Math.sin(freeFlyAngle) * 20;
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
            }

            composer.render();
        }
    </script>
</body>
</html>  
