
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NASA Eyes 3D - Earth Doom Narrative Edition</title>
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
            background: rgba(3, 7, 18, 0.88);
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
        .choice-card {
            background: rgba(15, 23, 42, 0.9);
            border: 1px solid rgba(56, 189, 248, 0.35);
            transition: all 0.25s ease;
        }
        .choice-card:hover {
            border-color: #f43f5e;
            background: rgba(159, 18, 57, 0.25);
            transform: translateY(-2px);
            box-shadow: 0 0 25px rgba(244, 63, 94, 0.4);
        }
        .custom-scrollbar::-webkit-scrollbar { height: 4px; width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: rgba(56, 189, 248, 0.6); border-radius: 99px; }
        #warpOverlay {
            position: fixed; inset: 0; background: radial-gradient(circle, rgba(244,63,94,0.8) 0%, rgba(15,23,42,1) 100%);
            z-index: 100; opacity: 0; pointer-events: none; transition: opacity 0.6s ease;
        }
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
                TRÁI ĐẤT TẬN THẾ <span class="text-transparent bg-clip-text bg-gradient-to-r from-rose-400 via-amber-400 to-cyan-300">NGÀY CUỐI CÙNG</span>
            </h1>
            <p class="text-xs sm:text-sm text-slate-400 leading-relaxed">
                Vòm Vạn Sao 360° • Quỹ Đạo Sao Thủy Rõ Ràng • Đưa Ra Quyết Định Trước Khi Trái Đất Bị Hủy Diệt!
            </p>
        </div>
        <button onclick="startExperience()" class="px-9 py-4 bg-gradient-to-r from-rose-600 via-amber-500 to-cyan-500 hover:brightness-110 font-orbitron font-bold text-xs sm:text-sm rounded-full text-white shadow-[0_0_60px_rgba(244,63,94,0.8)] transition active:scale-95">
            ⚡ KHỞI ĐỘNG HỆ MẶT TRỜI & SINH TỒN
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
                        Chọn hành tinh để quan sát hoặc chọn Trái Đất để bước vào kịch bản Ngày Tận Thế!
                    </p>
                    <div class="grid grid-cols-2 gap-1 pt-1 text-[10px] text-slate-400">
                        <div>Khoảng cách: <span id="planetDist" class="text-slate-200">0 AU</span></div>
                        <div>Nhiệt độ: <span id="planetTemp" class="text-amber-300">5,500 °C</span></div>
                    </div>

                    <button id="landEarthBtn" onclick="enterEarthMode()" class="hidden w-full mt-2 py-2.5 px-3 bg-gradient-to-r from-rose-600 to-amber-500 hover:brightness-110 text-white font-orbitron font-black text-xs rounded-xl shadow-[0_0_25px_rgba(244,63,94,0.7)] transition animate-pulse">
                        ⚠️ TRUNG TÂM KHƠI XƯỚNG NGÀY TẬN THẾ
                    </button>
                </div>
            </div>
        </header>

        <nav class="absolute bottom-5 left-1/2 -translate-x-1/2 z-20 hud-glass px-3 py-2.5 rounded-2xl flex gap-2 overflow-x-auto max-w-[95vw] custom-scrollbar">
            <button id="btn-sun" onclick="selectPlanet('sun')" class="planet-pill active px-3 py-1.5 text-amber-300 rounded-xl text-xs font-bold whitespace-nowrap">☀️ Mặt Trời</button>
            <button id="btn-mercury" onclick="selectPlanet('mercury')" class="planet-pill px-3 py-1.5 text-slate-300 rounded-xl text-xs font-bold whitespace-nowrap">🪨 Sao Thủy</button>
            <button id="btn-venus" onclick="selectPlanet('venus')" class="planet-pill px-3 py-1.5 text-amber-200 rounded-xl text-xs font-bold whitespace-nowrap">🌕 Sao Kim</button>
            <button id="btn-earth" onclick="selectPlanet('earth')" class="planet-pill px-3 py-1.5 text-rose-400 border-rose-500/50 rounded-xl text-xs font-bold whitespace-nowrap">🌍 Trái Đất ⚠️</button>
            <button id="btn-mars" onclick="selectPlanet('mars')" class="planet-pill px-3 py-1.5 text-rose-400 rounded-xl text-xs font-bold whitespace-nowrap">🔴 Sao Hỏa</button>
            <button id="btn-jupiter" onclick="selectPlanet('jupiter')" class="planet-pill px-3 py-1.5 text-orange-300 rounded-xl text-xs font-bold whitespace-nowrap">🪐 Sao Mộc</button>
            <button id="btn-saturn" onclick="selectPlanet('saturn')" class="planet-pill px-3 py-1.5 text-yellow-200 rounded-xl text-xs font-bold whitespace-nowrap">🪐 Sao Thổ</button>
            <button id="btn-uranus" onclick="selectPlanet('uranus')" class="planet-pill px-3 py-1.5 text-cyan-300 rounded-xl text-xs font-bold whitespace-nowrap">💎 Thiên Vương</button>
            <button id="btn-neptune" onclick="selectPlanet('neptune')" class="planet-pill px-3 py-1.5 text-indigo-400 rounded-xl text-xs font-bold whitespace-nowrap">🔵 Hải Vương</button>
        </nav>
    </div>

    <!-- GIAO DIỆN KỊCH BẢN NGÀY TẬN THẾ TRÁI ĐẤT -->
    <div id="earthUI" class="hidden">
        <div class="absolute top-4 left-4 right-4 z-20 flex flex-wrap justify-between items-center gap-3">
            <div class="hud-glass px-4 py-2.5 rounded-2xl flex items-center gap-4 flex-wrap">
                <div>
                    <div class="text-[10px] text-rose-400">⏳ THỜI GIAN CÒN LẠI</div>
                    <div class="font-orbitron font-bold text-sm text-amber-300"><span id="doomTimer">72:00:00</span></div>
                </div>
                <div class="h-7 w-[1px] bg-slate-700"></div>
                <div>
                    <div class="text-[10px] text-cyan-400">🛡️ TỶ LỆ SỐNG SÓT NHÂN LOẠI</div>
                    <div class="font-orbitron font-bold text-sm text-cyan-300"><span id="survivalRate">15</span>%</div>
                </div>
                <div class="h-7 w-[1px] bg-slate-700"></div>
                <div>
                    <div class="text-[10px] text-purple-400">🚀 TIẾN ĐỘ TÀU DI TẢN</div>
                    <div class="font-orbitron font-bold text-sm text-purple-300"><span id="arkProgress">5</span>%</div>
                </div>
            </div>

            <div class="flex gap-2">
                <button onclick="exitEarthMode()" class="hud-glass px-4 py-2.5 rounded-2xl font-orbitron font-bold text-xs text-cyan-300 hover:border-cyan-400 transition">
                    🌌 Trở Lại Quỹ Đạo Vũ Trụ
                </button>
            </div>
        </div>

        <div class="absolute bottom-5 left-1/2 -translate-x-1/2 z-20 hud-glass p-5 rounded-2xl w-[95vw] max-w-2xl space-y-4">
            <div class="flex justify-between items-center border-b border-slate-800 pb-2">
                <span id="scenarioTitle" class="text-xs font-orbitron font-bold text-rose-400 tracking-wider">⚠️ TÌNH HUỐNG KHẨN CẤP #1</span>
                <span id="scenarioPhase" class="text-[10px] bg-rose-950 text-rose-300 px-2 py-0.5 rounded font-bold">Giai đoạn: Thảm họa bắt đầu</span>
            </div>
            
            <p id="scenarioText" class="text-xs sm:text-sm text-slate-200 leading-relaxed">
                Mặt Trời bất ngờ bùng nổ siêu bão plasma Coronal Mass Ejection (CME) hướng trực diện vào Trái Đất. Từ trường hành tinh bắt đầu suy yếu, bầu khí quyển sôi sục. Bạn sẽ ra lệnh cho nhân loại làm gì trước tiên?
            </p>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 pt-2">
                <button id="choiceA" onclick="makeChoice(1)" class="choice-card p-3.5 rounded-xl text-left flex flex-col justify-between">
                    <span id="choiceAText" class="text-xs font-bold text-cyan-300">1. Kích hoạt toàn bộ mạng lưới khiên từ trường nhân tạo và sơ tán dân cư ngầm dưới lòng đất.</span>
                </button>
                <button id="choiceB" onclick="makeChoice(2)" class="choice-card p-3.5 rounded-xl text-left flex flex-col justify-between">
                    <span id="choiceBText" class="text-xs font-bold text-amber-300">2. Dồn toàn bộ nguồn lực và ngân sách thế giới để đẩy nhanh việc chế tạo Hạm đội Tàu Không Gian di tản liên sao.</span>
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
        // HỆ THỐNG KỊCH BẢN TẬN THẾ TRÁI ĐẤT
        // =========================================================================
        let gameMode = 'SPACE'; // 'SPACE', 'EARTH'
        let spaceGroup, earthDoomGroup;
        let earthSatellites = [], earthClouds = [];
        let currentScenario = 1;
        let survivalRate = 15;
        let arkProgress = 5;

        const scenarios = {
            1: {
                title: "⚠️ TÌNH HUỐNG KHẨN CẤP #1: SIÊU BÃO MẶT TRỜI",
                phase: "Giai đoạn: 72 giờ trước va chạm",
                text: "Mặt Trời bất ngờ bùng nổ siêu bão plasma Coronal Mass Ejection (CME) hướng trực diện vào Trái Đất. Từ trường hành tinh bắt đầu suy yếu, bầu khí quyển sôi sục. Bạn sẽ ra lệnh cho nhân loại làm gì trước tiên?",
                choiceA: "1. Kích hoạt toàn bộ mạng lưới khiên từ trường nhân tạo và sơ tán dân cư ngầm dưới lòng đất.",
                choiceB: "2. Dồn toàn bộ nguồn lực và ngân sách thế giới để đẩy nhanh việc chế tạo Hạm đội Tàu Không Gian di tản liên sao.",
                effectA: { survival: +15, ark: +5, next: 2 },
                effectB: { survival: +5, ark: +25, next: 2 }
            },
            2: {
                title: "⚠️ TÌNH HUỐNG KHẨN CẤP #2: SỤP ĐỔ HỆ THỐNG CÔNG NGHỆ",
                phase: "Giai đoạn: 48 giờ trước va chạm",
                text: "Làn sóng bức xạ đầu tiên đã quét qua quỹ đạo, khiến toàn bộ mạng vệ tinh GPS, thông tin liên lạc và lưới điện toàn cầu tê liệt. Bạo loạn bùng phát ở các siêu đô thị. Phản ứng của bạn là gì?",
                choiceA: "1. Ban bố thiết quân luật toàn cầu, huy động quân đội duy trì trật tự và bảo vệ các kho lương thực khẩn cấp.",
                choiceB: "2. Chuyển hướng toàn bộ điện năng còn lại cho các phòng thí nghiệm nghiên cứu động cơ cong vênh (Warp Drive).",
                effectA: { survival: +20, ark: +5, next: 3 },
                effectB: { survival: -5, ark: +30, next: 3 }
            },
            3: {
                title: "⚠️ TÌNH HUỐNG KHẨN CẤP #3: ĐÊM TRƯỚC NGÀY TẬN THẾ",
                phase: "Giai đoạn: 12 giờ trước va chạm",
                text: "Nhiệt độ bề mặt Trái Đất đang tăng vọt vượt mức sinh tồn. Đại dương bắt đầu bốc hơi dữ dội. Thời khắc quyết định định mệnh nhân loại đã đến, bạn chọn phương án nào để lưu giữ giống nòi?",
                choiceA: "1. Tải toàn bộ tri thức, ký ức và DNA nhân loại vào các trạm tự hành phóng vào không gian sâu thẳm.",
                choiceB: "2. Đưa những nhà khoa học và tinh hoa ưu tú nhất bước lên chuyến tàu cuối cùng rời khỏi hành tinh.",
                effectA: { survival: +25, ark: +35, next: 4 },
                effectB: { survival: +10, ark: +40, next: 4 }
            },
            4: {
                title: "🌟 KẾT CỤC: VẬN MỆNH NHÂN LOẠI",
                phase: "Giai đoạn: Hậu Tận Thế",
                text: "Trái Đất đã chính thức chìm trong biển lửa và trở thành một hành tinh hoang vắng. Tuy nhiên, nhờ vào quyết định của bạn, những tia lửa hy vọng của nền văn minh nhân loại đã vươn tới các vì sao xa xôi...",
                choiceA: "🔄 Chơi lại từ đầu kịch bản tận thế",
                choiceB: "🌌 Khám phá lại Hệ Mặt Trời 3D",
                effectA: { reset: true },
                effectB: { orbit: true }
            }
        };

        function makeChoice(option) {
            const sc = scenarios[currentScenario];
            const eff = option === 1 ? sc.effectA : sc.effectB;

            if (eff.reset) {
                currentScenario = 1;
                survivalRate = 15;
                arkProgress = 5;
                loadScenario(1);
                return;
            }
            if (eff.orbit) {
                exitEarthMode();
                return;
            }

            survivalRate = Math.min(100, survivalRate + eff.survival);
            arkProgress = Math.min(100, arkProgress + eff.ark);
            currentScenario = eff.next;

            document.getElementById('survivalRate').innerText = survivalRate;
            document.getElementById('arkProgress').innerText = arkProgress;

            loadScenario(currentScenario);
        }

        function loadScenario(id) {
            const sc = scenarios[id];
            document.getElementById('scenarioTitle').innerText = sc.title;
            document.getElementById('scenarioPhase').innerText = sc.phase;
            document.getElementById('scenarioText').innerText = sc.text;
            document.getElementById('choiceAText').innerText = sc.choiceA;
            document.getElementById('choiceBText').innerText = sc.choiceB;

            if (id === 4) {
                document.getElementById('choiceAText').innerText = "🔄 Thử lại kịch bản khác";
                document.getElementById('choiceBText').innerText = "🌌 Thoát ra quan sát vũ trụ";
            }
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
            triggerWarpEffect(() => {
                gameMode = 'EARTH';
                spaceGroup.visible = false;
                if (!earthDoomGroup) buildEarthDoomBiome3D();
                earthDoomGroup.visible = true;
                if (bloomPass) bloomPass.strength = 0.35;
                if (renderer) renderer.toneMappingExposure = 0.9;

                document.getElementById('spaceUI').classList.add('hidden');
                document.getElementById('earthUI').classList.remove('hidden');

                currentLookAt.set(0, 2.5, 0); targetCamDist = 65; camDist = 65; camAngleX = 0.45; camAngleY = 0.6;
                loadScenario(currentScenario);
            });
        }

        function exitEarthMode() {
            triggerWarpEffect(() => {
                gameMode = 'SPACE';
                if (earthDoomGroup) earthDoomGroup.visible = false;
                spaceGroup.visible = true;
                if (bloomPass) bloomPass.strength = 1.4;
                if (renderer) renderer.toneMappingExposure = 1.25;

                document.getElementById('earthUI').classList.add('hidden');
                document.getElementById('spaceUI').classList.remove('hidden');
                selectPlanet('earth');
            });
        }

        function buildEarthDoomBiome3D() {
            earthDoomGroup = new THREE.Group();
            scene.add(earthDoomGroup);

            const hemiLight = new THREE.HemisphereLight(0xf43f5e, 0x450a0a, 0.8);
            earthDoomGroup.add(hemiLight);
            const dirLight = new THREE.DirectionalLight(0xf59e0b, 1.5);
            dirLight.position.set(30, 45, 30);
            earthDoomGroup.add(dirLight);

            const baseMesh = new THREE.Mesh(
                new THREE.ConeGeometry(32, 35, 7).rotateX(Math.PI),
                new THREE.MeshStandardMaterial({ color: 0x18181b, roughness: 0.9, flatShading: true })
            );
            baseMesh.position.y = -17.5;
            earthDoomGroup.add(baseMesh);

            const scorchedEarth = new THREE.Mesh(
                new THREE.CylinderGeometry(32.4, 32, 1.5, 32),
                new THREE.MeshStandardMaterial({ color: 0x7f1d1d, roughness: 0.8, emissive: 0x450a0a, emissiveIntensity: 0.3 })
            );
            scorchedEarth.position.y = -0.4;
            earthDoomGroup.add(scorchedEarth);

            // Cột khói dung nham ngày tận thế
            for (let i = 0; i < 8; i++) {
                const angle = Math.random() * Math.PI * 2;
                const r = 8 + Math.random() * 18;
                const smoke = new THREE.Mesh(
                    new THREE.ConeGeometry(1.5, 8, 6),
                    new THREE.MeshStandardMaterial({ color: 0xf43f5e, emissive: 0xe11d48, emissiveIntensity: 0.8, transparent: true, opacity: 0.7 })
                );
                smoke.position.set(Math.cos(angle) * r, 3, Math.sin(angle) * r);
                earthDoomGroup.add(smoke);
            }
        }

        // =========================================================================
        // ENGINE 3D VŨ TRỤ THREE.JS (VÒM VẠN SAO + QUỸ ĐẠO THUỶ RÕ RÀNG)
        // =========================================================================
        let scene, camera, renderer, composer, bloomPass;
        let planets = {};
        let sunUniforms, starDomeGroup, asteroidBelt, spacetimeGrid, cosmicDust;
        let focusedObject = 'sun';
        let isHudVisible = true, isDragging = false, prevMouse = { x: 0, y: 0 };
        let camDist = 95, targetCamDist = 95;
        let camAngleX = 0.35, camAngleY = 0.25;
        let currentLookAt = new THREE.Vector3(0, 0, 0);
        let timeSpeed = 1.0;
        let isFreeFly = false, freeFlyAngle = 0;

        const planetDB = {
            sun: { name: "MẶT TRỜI (SOL)", desc: "Khối cầu plasma sôi sục khổng lồ. Bấm chọn Trái Đất để mở bảng điều khiển Tận Thế!", dist: "0 AU", temp: "5,500 °C", zoom: 90 },
            mercury: { name: "SAO THỦY (MERCURY)", desc: "Bề mặt đá bazan xám bạc lồi lõm hàng triệu hố thiên thạch.", dist: "0.39 AU", temp: "430 °C", zoom: 20 },
            venus: { name: "SAO KIM (VENUS)", desc: "Khí quyển siêu đặc cuộn xoáy mây axit sulfuric ánh vàng rực.", dist: "0.72 AU", temp: "462 °C", zoom: 22 },
            earth: { name: "TRÁI ĐẤT (EARTH)", desc: "Hành tinh đang lâm nguy! Khởi xướng kịch bản sinh tồn ngày tận thế.", dist: "1.00 AU", temp: "45 °C+", zoom: 22 },
            mars: { name: "SAO HỎA (MARS)", desc: "Sa mạc oxit sắt đỏ rực với chỏm băng cực và hẻm núi sâu.", dist: "1.52 AU", temp: "-63 °C", zoom: 20 },
            jupiter: { name: "SAO MỘC (JUPITER)", desc: "Khí quyển dải băng tầng khí động học & cơn bão Vết Đỏ Lớn.", dist: "5.20 AU", temp: "-110 °C", zoom: 32 },
            saturn: { name: "SAO THỔ (SATURN)", desc: "Hệ thống vành đai băng đa tầng phân tách bởi khe Cassini sắc nét.", dist: "9.58 AU", temp: "-140 °C", zoom: 38 },
            uranus: { name: "SAO THIÊN VƯƠNG (URANUS)", desc: "Hành tinh băng khí metan xanh ngọc bích với vành đai dọc mờ ảo.", dist: "19.22 AU", temp: "-195 °C", zoom: 28 },
            neptune: { name: "SAO HẢI VƯƠNG (NEPTUNE)", desc: "Thế giới băng giá xanh thẫm với những luồng gió siêu thanh.", dist: "30.05 AU", temp: "-200 °C", zoom: 28 }
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
                ctx.fillStyle = '#94a3b8'; ctx.fillRect(0, 0, w, h);
            } else if (type === 'venus') {
                ctx.fillStyle = '#f59e0b'; ctx.fillRect(0, 0, w, h);
            } else if (type === 'earth') {
                ctx.fillStyle = '#0284c7'; ctx.fillRect(0, 0, w, h);
                ctx.fillStyle = '#15803d'; ctx.beginPath(); ctx.ellipse(1000, 500, 300, 180, 0, 0, Math.PI * 2); ctx.fill();
            } else if (type === 'clouds') {
                ctx.clearRect(0, 0, w, h);
                ctx.fillStyle = 'rgba(255,255,255,0.4)';
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
                const angle = Math.random() * Math.PI * 2; const radius = 48 + Math.random() * 8;
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
            renderer.toneMappingExposure = 1.25;
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
                { id: 'mercury', size: 1.6, dist: 17, speed: 0.022, tilt: 0.03, atmos: null },
                { id: 'venus', size: 2.2, dist: 25, speed: 0.016, tilt: 3.09, atmos: 0xfbbf24 },
                { id: 'earth', size: 2.6, dist: 34, speed: 0.012, tilt: 0.41, atmos: 0x38bdf8, clouds: true },
                { id: 'mars', size: 1.8, dist: 44, speed: 0.009, tilt: 0.44, atmos: 0xf97316 },
                { id: 'jupiter', size: 5.8, dist: 62, speed: 0.006, tilt: 0.05, atmos: 0xfdba74 },
                { id: 'saturn', size: 4.8, dist: 80, speed: 0.004, tilt: 0.47, atmos: 0xfef08a, rings: true },
                { id: 'uranus', size: 3.2, dist: 98, speed: 0.003, tilt: 1.71, atmos: 0x22d3ee },
                { id: 'neptune', size: 3.1, dist: 114, speed: 0.002, tilt: 0.49, atmos: 0x3b82f6 }
            ];

            planetConfigs.forEach(cfg => {
                const orbitGeo = new THREE.RingGeometry(cfg.dist - 0.12, cfg.dist + 0.12, 220);
                const orbitRing = new THREE.Mesh(orbitGeo, new THREE.MeshBasicMaterial({ color: cfg.atmos || 0x38bdf8, side: THREE.DoubleSide, transparent: true, opacity: 0.45, blending: THREE.AdditiveBlending }));
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
                // Earth doom animations if needed
            } else {
                if (sunUniforms) sunUniforms.uTime.value += 0.015 * timeSpeed;
                if (starDomeGroup) starDomeGroup.rotation.y += 0.0001 * timeSpeed;
                if (cosmicDust) cosmicDust.rotation.y += 0.00025 * timeSpeed;
                if (asteroidBelt && asteroidBelt.visible) asteroidBelt.rotation.y += 0.0018 * timeSpeed;

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
