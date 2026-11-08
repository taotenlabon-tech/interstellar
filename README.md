<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cosmic Conqueror 3D - Galactic Overlord</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@700;900&family=Inter:wght@400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; background: #030712; color: #f8fafc; overflow-x: hidden; }
        .font-orbitron { font-family: 'Orbitron', sans-serif; }
        .shake { animation: shake 0.3s cubic-bezier(.36,.07,.19,.97) both; }
        @keyframes shake {
            10%, 90% { transform: translate3d(-4px, 0, 0); }
            20%, 80% { transform: translate3d(8px, 0, 0); }
            30%, 50%, 70% { transform: translate3d(-12px, 0, 0); }
            40%, 60% { transform: translate3d(12px, 0, 0); }
        }
        .damage-pop {
            position: absolute; animation: floatUp 0.85s cubic-bezier(0.25, 1, 0.5, 1) forwards;
            pointer-events: none; font-weight: 900; text-shadow: 0 0 15px rgba(56, 189, 248, 0.9); z-index: 50;
        }
        @keyframes floatUp {
            0% { opacity: 1; transform: translateY(0) scale(0.6); }
            40% { opacity: 1; transform: translateY(-45px) scale(1.4); }
            100% { opacity: 0; transform: translateY(-90px) scale(1); }
        }
        .tab-btn-active {
            background: linear-gradient(135deg, #0284c7 0%, #0369a1 100%);
            border-color: #38bdf8; box-shadow: 0 0 25px rgba(56, 189, 248, 0.6); color: #ffffff;
        }
        .glass-panel {
            background: linear-gradient(135deg, rgba(15, 23, 42, 0.95) 0%, rgba(3, 7, 18, 0.98) 100%);
            backdrop-filter: blur(16px); border: 1px solid rgba(56, 189, 248, 0.4); box-shadow: 0 10px 30px rgba(0, 0, 0, 0.8);
        }
        .space-bg { background: radial-gradient(circle at center, #0b1329 0%, #020617 80%); }
        .sci-fi-border {
            border: 2px solid rgba(56, 189, 248, 0.7);
            box-shadow: 0 0 35px rgba(56, 189, 248, 0.3), inset 0 0 20px rgba(56, 189, 248, 0.2);
        }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center justify-between p-2 sm:p-6 bg-[#030712]">

    <!-- MÀN HÌNH BẮT ĐẦU VŨ TRỤ -->
    <div id="startMenu" class="fixed inset-0 z-50 space-bg flex flex-col items-center justify-center p-6 transition-opacity duration-700">
        <div class="absolute inset-0 bg-[radial-gradient(ellipse_at_top,_var(--tw-gradient-stops))] from-cyan-500/20 via-transparent to-slate-950 pointer-events-none"></div>
        <div class="z-10 text-center max-w-xl mx-auto space-y-6 sci-fi-border p-8 rounded-3xl bg-slate-950/95 backdrop-blur-xl">
            <div class="inline-block p-4 bg-cyan-950/60 border border-cyan-400 rounded-3xl shadow-[0_0_35px_rgba(56,189,248,0.5)] mb-2 animate-bounce">
                <span class="text-5xl">🚀</span>
            </div>
            <h1 class="font-orbitron text-3xl sm:text-5xl font-black text-transparent bg-clip-text bg-gradient-to-r from-cyan-300 via-sky-400 to-indigo-500 tracking-wider">
                COSMIC CONQUEROR
            </h1>
            <p class="text-slate-300 text-xs sm:text-sm max-w-md mx-auto italic">
                Chinh phục các Vì sao - Điều khiển Phi thuyền 3D & Tiêu diệt Boss Thiên Hà!
            </p>
            <div class="pt-4 space-y-3 w-full max-w-sm mx-auto">
                <button onclick="startAction('battle')" class="w-full py-3.5 bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 font-orbitron font-bold text-base rounded-2xl text-white shadow-[0_0_20px_rgba(56,189,248,0.6)] active:scale-95 transition tracking-wider border border-cyan-400 flex items-center justify-center gap-3">
                    <span>🚀</span> ĐẤU TRƯỜNG VŨ TRỤ 3D
                </button>
                <button onclick="startAction('stats')" class="w-full py-3.5 bg-gradient-to-r from-purple-600 to-indigo-600 hover:from-purple-500 hover:to-indigo-500 font-orbitron font-bold text-base rounded-2xl text-white shadow-[0_0_20px_rgba(168,85,247,0.6)] active:scale-95 transition tracking-wider border border-purple-400 flex items-center justify-center gap-3">
                    <span>⚡</span> NÂNG CẤP PHI THUYỀN
                </button>
                <button onclick="startAction('gather')" class="w-full py-3.5 bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-500 hover:to-teal-500 font-orbitron font-bold text-base rounded-2xl text-white shadow-[0_0_20px_rgba(16,185,129,0.6)] active:scale-95 transition tracking-wider border border-emerald-400 flex items-center justify-center gap-3">
                    <span>💎</span> MỎ TINH THỂ TINH VÂN
                </button>
                <div class="pt-2">
                    <button onclick="resetData()" class="w-full py-2 bg-slate-900 hover:bg-slate-800 font-semibold text-xs rounded-xl text-slate-300 border border-slate-700 transition">
                        🔄 Xóa Dữ Liệu Lưu Cũ
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- HEADER -->
    <header class="w-full max-w-5xl glass-panel rounded-2xl p-4 mb-4 shadow-2xl flex flex-wrap justify-between items-center gap-4">
        <div class="flex items-center gap-3">
            <div class="w-12 h-12 bg-cyan-950 border border-cyan-500 rounded-xl flex items-center justify-center text-2xl shadow-[0_0_15px_rgba(56,189,248,0.4)] cursor-pointer hover:bg-cyan-900 transition" onclick="returnToMenu()" title="Về Menu Chính">
                🏠
            </div>
            <div>
                <h1 class="font-orbitron text-xl font-bold text-cyan-400 tracking-wider">COSMIC CONQUEROR 3D</h1>
                <p class="text-xs text-slate-300">Dạng Boss Thiên Hà: <span id="bossEvoLvl" class="text-sky-300 font-bold">Tier 1 (Gốc)</span></p>
            </div>
        </div>
        <div class="flex flex-wrap gap-2.5 text-xs sm:text-sm">
            <div class="bg-slate-950/90 px-3 py-1.5 rounded-xl border border-slate-800 flex items-center gap-1.5 shadow-inner">
                <span>🪙 Năng Lượng:</span> <span id="resGold" class="text-yellow-300 font-bold">0</span>
            </div>
            <div class="bg-slate-950/90 px-3 py-1.5 rounded-xl border border-slate-800 flex items-center gap-1.5 shadow-inner">
                <span>💎 Tinh Thể:</span> <span id="resCrystal" class="text-cyan-300 font-bold">0</span>
            </div>
            <div class="bg-slate-950/90 px-3 py-1.5 rounded-xl border border-slate-800 flex items-center gap-1.5 shadow-inner">
                <span>🔮 Vật Chất Tối:</span> <span id="resDarkMatter" class="text-purple-300 font-bold">0</span>
            </div>
        </div>
    </header>

    <!-- THANH NAV TAB -->
    <nav class="w-full max-w-5xl flex gap-2 mb-4">
        <button id="tabBattleBtn" onclick="switchTab('battle')" class="flex-1 py-3 px-2 rounded-xl font-orbitron font-bold text-xs sm:text-sm border border-slate-700 bg-slate-950 transition flex items-center justify-center gap-1 text-slate-300 shadow-lg">
            <span>🚀</span> ĐẤU TRƯỜNG VŨ TRỤ
        </button>
        <button id="tabStatsBtn" onclick="switchTab('stats')" class="flex-1 py-3 px-2 rounded-xl font-orbitron font-bold text-xs sm:text-sm border border-slate-700 bg-slate-950 transition flex items-center justify-center gap-1 text-slate-300 shadow-lg">
            <span>⚡</span> NÂNG CẤP
        </button>
        <button id="tabGatherBtn" onclick="switchTab('gather')" class="flex-1 py-3 px-2 rounded-xl font-orbitron font-bold text-xs sm:text-sm border border-slate-700 bg-slate-950 transition flex items-center justify-center gap-1 text-slate-300 shadow-lg">
            <span>💎</span> MỎ TINH THỂ
        </button>
    </nav>

    <main class="w-full max-w-5xl flex-1 flex flex-col">
        
        <!-- MÀN HÌNH 1: ĐẤU TRƯỜNG VŨ TRỤ 3D -->
        <section id="pageBattle" class="flex-1 flex-col gap-4">
            <div class="glass-panel rounded-2xl p-3 flex flex-wrap items-center justify-between gap-2 shadow-lg">
                <span class="text-xs text-slate-300 font-bold">CHỌN TỌA ĐỘ BOSS THIÊN HÀ:</span>
                <div class="flex gap-1.5 flex-wrap" id="dungeonButtons">
                    <button onclick="selectDungeon(0)" class="btn-dungeon border border-emerald-500 bg-emerald-950/70 text-emerald-200 text-xs px-2.5 py-1 rounded-xl font-semibold hover:bg-emerald-900 transition">Màn 1: Trạm Trinh Thám</button>
                    <button onclick="selectDungeon(1)" class="btn-dungeon border border-slate-400 bg-slate-800/70 text-slate-200 text-xs px-2.5 py-1 rounded-xl font-semibold hover:bg-slate-700 transition">Màn 2: Titan Cổ Đại</button>
                    <button onclick="selectDungeon(2)" class="btn-dungeon border border-cyan-500 bg-cyan-950/70 text-cyan-200 text-xs px-2.5 py-1 rounded-xl font-semibold hover:bg-cyan-900 transition">Màn 3: Leviathan Vũ Trụ 🌌</button>
                    <button onclick="selectDungeon(3)" class="btn-dungeon border border-red-500 bg-red-950/70 text-red-200 text-xs px-2.5 py-1 rounded-xl font-semibold hover:bg-red-900 transition">Màn 4: Hỏa Long Tinh Vân</button>
                    <button onclick="selectDungeon(5)" class="btn-dungeon border border-purple-500 bg-purple-950/70 text-purple-200 text-xs px-2.5 py-1 rounded-xl font-semibold hover:bg-purple-900 transition">Màn 5: Tàu Mẹ Ma Vương</button>
                </div>
            </div>

            <div id="battleArena" class="relative glass-panel rounded-2xl p-4 flex-1 flex flex-col items-center justify-between min-h-[460px] overflow-hidden shadow-2xl">
                <div class="text-center z-10 flex items-center justify-center gap-4">
                    <div>
                        <span id="dungeonLevelBadge" class="text-xs font-bold uppercase tracking-widest text-cyan-400 bg-cyan-950/80 px-2.5 py-1 rounded-full border border-cyan-600/50">MÀN 1</span>
                        <h3 id="monsterName" class="text-xl font-bold font-orbitron text-white mt-1">Trạm Trinh Thám Không Gian</h3>
                    </div>
                </div>

                <div class="relative my-2 w-full max-w-xl h-72 flex flex-col items-center justify-center">
                    <div id="battle3DCanvas" class="w-full h-full rounded-2xl border border-slate-700 shadow-[inset_0_0_40px_rgba(0,0,0,0.9)] bg-black"></div>
                    
                    <div class="w-full bg-slate-950 rounded-full h-4 p-0.5 border border-slate-700 overflow-hidden shadow-inner mt-2">
                        <div id="monsterHpBar" class="bg-gradient-to-r from-cyan-500 to-blue-600 h-full rounded-full transition-all duration-200" style="width: 100%;"></div>
                    </div>
                    <div id="monsterHpText" class="text-xs text-cyan-300 font-bold mt-1">300 / 300 HP</div>
                </div>

                <!-- ĐIỀU KHIỂN & KỸ NĂNG KHÔNG GIAN -->
                <div class="w-full max-w-xl flex flex-col gap-2 z-10">
                    <div class="grid grid-cols-3 gap-2">
                        <button id="btnSkill1" onclick="useSkill(1)" class="py-2 bg-gradient-to-r from-orange-600 to-red-600 hover:from-orange-500 hover:to-red-500 font-bold rounded-xl text-white text-xs border border-orange-400 transition shadow">
                            🔥 Bão Plasma (CD: <span id="cdSkill1">0</span>s)
                        </button>
                        <button id="btnSkill2" onclick="useSkill(2)" class="py-2 bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 font-bold rounded-xl text-white text-xs border border-cyan-400 transition shadow">
                            ⚡ Khiên Quang Học (CD: <span id="cdSkill2">0</span>s)
                        </button>
                        <button id="btnSkill3" onclick="useSkill(3)" class="py-2 bg-gradient-to-r from-purple-600 to-pink-600 hover:from-purple-500 hover:to-pink-500 font-bold rounded-xl text-white text-xs border border-purple-400 transition shadow">
                            💀 Pháo Laser Vực Thẳm (CD: <span id="cdSkill3">0</span>s)
                        </button>
                    </div>

                    <div class="flex gap-2">
                        <button onclick="shipAttack()" class="flex-1 py-3 bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 font-orbitron font-bold rounded-xl text-white shadow-lg shadow-cyan-600/40 active:scale-95 transition flex items-center justify-center gap-2 text-sm">
                            <span>🚀</span> BẮN PHÁO LASER (Space)
                        </button>
                        <button id="btnAutoAttack" onclick="toggleAutoAttack()" class="py-3 px-4 bg-slate-900 border border-slate-700 hover:bg-slate-800 font-semibold rounded-xl text-slate-200 transition text-xs">
                            🤖 Tự Động: OFF
                        </button>
                    </div>
                </div>
            </div>
        </section>

        <!-- MÀN HÌNH 2: NÂNG CẤP PHI THUYỀN VŨ TRỤ -->
        <section id="pageStats" class="hidden grid-cols-1 md:grid-cols-2 gap-5 flex-1">
            <div class="glass-panel rounded-2xl p-5 flex flex-col gap-4 shadow-2xl">
                <h2 class="text-lg font-bold text-cyan-400 border-b border-slate-800 pb-2.5 flex items-center gap-2">
                    <span>⚡ Nâng Cấp Hệ Thống Phi Thuyền (Năng Lượng)</span>
                </h2>
                <div class="space-y-3 flex-1 overflow-y-auto pr-1">
                    <div class="p-3 bg-slate-950/80 rounded-xl border border-slate-800 flex items-center justify-between">
                        <div>
                            <div class="font-bold text-sm text-slate-200">🚀 Sát Thương Pháo Laser (ATK)</div>
                            <div class="text-xs text-slate-400">Tăng +10 Sát Thương Gốc (Cấp <span id="statAtkLvl">0</span>)</div>
                        </div>
                        <button onclick="upgradeStat('atk')" class="py-2 px-3 bg-cyan-600 hover:bg-cyan-500 font-bold text-xs rounded-lg text-white transition border border-cyan-400">
                            +10 ATK (<span id="costStatAtk">200</span> 🪙)
                        </button>
                    </div>
                    <div class="p-3 bg-slate-950/80 rounded-xl border border-slate-800 flex items-center justify-between">
                        <div>
                            <div class="font-bold text-sm text-slate-200">🎯 Tỉ Lệ Bạo Kích Laser (CRIT %)</div>
                            <div class="text-xs text-slate-400">Tăng +2% Tỉ lệ bạo kích (Cấp <span id="statCritLvl">0</span>)</div>
                        </div>
                        <button onclick="upgradeStat('crit')" class="py-2 px-3 bg-cyan-600 hover:bg-cyan-500 font-bold text-xs rounded-lg text-white transition border border-cyan-400">
                            +2% CRIT (<span id="costStatCrit">400</span> 🪙)
                        </button>
                    </div>
                    <div class="p-3 bg-slate-950/80 rounded-xl border border-slate-800 flex items-center justify-between">
                        <div>
                            <div class="font-bold text-sm text-slate-200">💰 Thu Hoạch Năng Lượng Boss</div>
                            <div class="text-xs text-slate-400">Tăng +15% Năng lượng nhận (Cấp <span id="statGoldLvl">0</span>)</div>
                        </div>
                        <button onclick="upgradeStat('gold')" class="py-2 px-3 bg-cyan-600 hover:bg-cyan-500 font-bold text-xs rounded-lg text-white transition border border-cyan-400">
                            +15% Năng Lượng (<span id="costStatGold">300</span> 🪙)
                        </button>
                    </div>
                </div>
            </div>

            <div class="glass-panel rounded-2xl p-5 flex flex-col gap-4 shadow-2xl">
                <h2 class="text-lg font-bold text-purple-400 border-b border-slate-800 pb-2.5 flex items-center gap-2">
                    <span>🔮 Kỹ Năng Vũ Trụ Tinh Vân (Vật Chất Tối)</span>
                </h2>
                <div class="space-y-3 flex-1 overflow-y-auto pr-1">
                    <div class="p-3 bg-slate-950/80 rounded-xl border border-purple-900/60 flex items-center justify-between">
                        <div>
                            <div class="font-bold text-sm text-orange-400">🔥 Bão Plasma</div>
                            <div class="text-xs text-slate-400">Gây x<span id="skill1Mult">4</span> sát thương plasma (Cấp <span id="skill1Lvl">1</span>)</div>
                        </div>
                        <button onclick="upgradeSkill(1)" class="py-2 px-3 bg-purple-700 hover:bg-purple-600 font-bold text-xs rounded-lg text-white transition border border-purple-400">
                            Nâng Cấp (<span id="costSkill1">10</span> 🔮)
                        </button>
                    </div>
                    <div class="p-3 bg-slate-950/80 rounded-xl border border-purple-900/60 flex items-center justify-between">
                        <div>
                            <div class="font-bold text-sm text-cyan-400">⚡ Khiên Quang Học</div>
                            <div class="text-xs text-slate-400">100% Chí Mạng trong <span id="skill2Dur">5</span> giây (Cấp <span id="skill2Lvl">1</span>)</div>
                        </div>
                        <button onclick="upgradeSkill(2)" class="py-2 px-3 bg-purple-700 hover:bg-purple-600 font-bold text-xs rounded-lg text-white transition border border-purple-400">
                            Nâng Cấp (<span id="costSkill2">15</span> 🔮)
                        </button>
                    </div>
                    <div class="p-3 bg-slate-950/80 rounded-xl border border-purple-900/60 flex items-center justify-between">
                        <div>
                            <div class="font-bold text-sm text-pink-400">💀 Pháo Laser Vực Thẳm</div>
                            <div class="text-xs text-slate-400">Bắn thẳng <span id="skill3Pct">10</span>% Max HP Boss (Cấp <span id="skill3Lvl">1</span>)</div>
                        </div>
                        <button onclick="upgradeSkill(3)" class="py-2 px-3 bg-purple-700 hover:bg-purple-600 font-bold text-xs rounded-lg text-white transition border border-purple-400">
                            Nâng Cấp (<span id="costSkill3">25</span> 🔮)
                        </button>
                    </div>
                </div>
            </div>
        </section>

        <!-- MÀN HÌNH 3: MỎ TINH THỂ VŨ TRỤ 3D -->
        <section id="pageGather" class="hidden flex-1 flex-col items-center justify-center gap-6 glass-panel rounded-2xl p-8 shadow-2xl">
            <div class="text-center space-y-2">
                <span class="text-xs text-cyan-400 font-bold tracking-widest uppercase">MỎ VẬT CHẤT VŨ TRỤ</span>
                <h2 class="text-3xl font-bold font-orbitron text-white">KHU KHAI THÁC TINH THỂ 3D</h2>
                <p class="text-xs text-slate-300 max-w-md mx-auto">Vung tia thu hoạch năng lượng vào khối Tinh thể Tinh vân để thu thập Năng lượng & Vật chất tối!</p>
            </div>
            <div class="relative w-full max-w-md h-64 bg-slate-950/90 rounded-2xl border border-cyan-800/80 flex flex-col items-center justify-center p-6 shadow-inner overflow-hidden cursor-pointer" onclick="performGather()" id="gatherZone">
                <div id="ore3DContainer" class="w-full h-full absolute inset-0"></div>
                <div class="absolute bottom-3 text-xs font-semibold text-cyan-300 bg-slate-950/80 px-3 py-1 rounded-full border border-cyan-500/50 animate-pulse pointer-events-none">
                    CLICK ĐỂ THU HOẠCH TINH THỂ!
                </div>
            </div>
            <button onclick="performGather()" class="w-full max-w-md py-4 bg-gradient-to-r from-cyan-600 to-blue-600 font-orbitron font-bold text-lg rounded-2xl text-white shadow-lg active:scale-95 transition">
                💎 THU HOẠCH TINH THỂ
            </button>
        </section>

    </main>

    <footer class="w-full max-w-5xl mt-4 glass-panel rounded-xl p-3 text-xs text-slate-300 flex items-center justify-between shadow-lg">
        <span class="font-semibold text-cyan-400">📜 Nhật Ký Hàng Hải:</span>
        <span id="gameLog" class="truncate max-w-md text-right text-slate-200">Sẵn sàng xuất kích chinh phục thiên hà!</span>
    </footer>

    <!-- MODAL TIẾN HOÁ VŨ TRỤ NEWGAME+ -->
    <div id="evoModal" class="fixed inset-0 z-50 bg-slate-950/90 flex flex-col items-center justify-center p-6 hidden">
        <div class="max-w-md w-full glass-panel p-6 rounded-3xl text-center space-y-5 border border-cyan-400 shadow-[0_0_40px_rgba(56,189,248,0.5)]">
            <h2 class="text-2xl font-bold font-orbitron text-cyan-300">👑 TIÊU DIỆT SẠCH BOSS THIÊN HÀ!</h2>
            <p class="text-xs text-slate-300 leading-relaxed">
                Bạn đã chinh phục thành công cả 5 Màn chơi cùng Siêu Boss Leviathan Vũ Trụ! Bạn muốn thực hiện bước nhảy không gian nào tiếp theo?
            </p>
            <div class="flex flex-col gap-3 pt-2">
                <button onclick="chooseEvoOption('evolve')" class="w-full py-3 bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 font-bold rounded-xl text-white text-sm border border-cyan-300 shadow-lg">
                    🧬 TIẾN HOÁ VŨ TRỤ (Tier +1, Boss +50% HP & Năng Lượng)
                </button>
                <button onclick="chooseEvoOption('reset')" class="w-full py-3 bg-slate-800 hover:bg-slate-700 font-semibold rounded-xl text-slate-200 text-sm border border-slate-600">
                    🔄 THIẾT LẬP LẠI VỀ TIER 1 (Dạng Gốc)
                </button>
            </div>
        </div>
    </div>

    <script>
        let audioCtx = null;
        function initAudio() {
            if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            if (audioCtx.state === 'suspended') audioCtx.resume();
        }

        function playSound(type) {
            try {
                initAudio();
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.connect(gain); gain.connect(audioCtx.destination);
                if (type === 'hit') {
                    osc.type = 'sawtooth'; osc.frequency.setValueAtTime(400, audioCtx.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(80, audioCtx.currentTime + 0.1);
                    gain.gain.setValueAtTime(0.3, audioCtx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.1);
                    osc.start(); osc.stop(audioCtx.currentTime + 0.1);
                } else if (type === 'crit') {
                    osc.type = 'square'; osc.frequency.setValueAtTime(600, audioCtx.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(1200, audioCtx.currentTime + 0.15);
                    gain.gain.setValueAtTime(0.4, audioCtx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.15);
                    osc.start(); osc.stop(audioCtx.currentTime + 0.15);
                } else if (type === 'skill') {
                    osc.type = 'sine'; osc.frequency.setValueAtTime(300, audioCtx.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(1500, audioCtx.currentTime + 0.35);
                    gain.gain.setValueAtTime(0.5, audioCtx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.35);
                    osc.start(); osc.stop(audioCtx.currentTime + 0.35);
                } else if (type === 'win') {
                    osc.type = 'triangle'; osc.frequency.setValueAtTime(523.25, audioCtx.currentTime);
                    osc.frequency.setValueAtTime(659.25, audioCtx.currentTime + 0.15);
                    gain.gain.setValueAtTime(0.4, audioCtx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.5);
                    osc.start(); osc.stop(audioCtx.currentTime + 0.5);
                }
            } catch(e) {}
        }

        let player = {
            gold: 2000, crystal: 200, darkMatter: 50,
            bossEvoTier: 1,
            stats: { atkLvl: 0, critLvl: 0, goldLvl: 0 },
            skills: {
                s1: { lvl: 1, cd: 0, maxCd: 6 },
                s2: { lvl: 1, cd: 0, maxCd: 10 },
                s3: { lvl: 1, cd: 0, maxCd: 15 }
            }
        };

        let tempBerserkActive = false;

        // 5 Boss Vũ Trụ (Có Leviathan ở Màn 3)
        const baseDungeons = [
            { level: 1, name: "Màn 1: Trạm Trinh Thám", monster: "Trạm Trinh Thám Không Gian", bossType: 'station', maxHp: 300, goldReward: 300, darkMatterDrop: 2 },
            { level: 2, name: "Màn 2: Titan Cổ Đại", monster: "Thạch Anh Vũ Trụ Titan", bossType: 'titan', maxHp: 1000, goldReward: 700, darkMatterDrop: 4 },
            { level: 3, name: "Màn 3: Leviathan Vũ Trụ", monster: "Thủy Tộc Leviathan Vũ Trụ", bossType: 'leviathan', maxHp: 2800, goldReward: 1500, darkMatterDrop: 8 },
            { level: 4, name: "Màn 4: Hỏa Long Tinh Vân", monster: "Hỏa Long Tinh Vân Bão Tố", bossType: 'dragon', maxHp: 7000, goldReward: 3500, darkMatterDrop: 15 },
            { level: 5, name: "Màn 5: Tàu Mẹ Ma Vương", monster: "Tàu Mẹ Hủy Diệt Ma Vương", bossType: 'overlord', maxHp: 25000, goldReward: 12000, darkMatterDrop: 30 }
        ];

        let dungeons = [];

        function recalculateDungeons() {
            const mult = Math.pow(1.5, player.bossEvoTier - 1);
            dungeons = baseDungeons.map(b => ({
                ...b,
                maxHp: Math.floor(b.maxHp * mult),
                goldReward: Math.floor(b.goldReward * mult),
                monster: player.bossEvoTier > 1 ? `${b.monster} (Tier ${player.bossEvoTier})` : b.monster
            }));
        }

        let currentDungeonIdx = 0;
        let currentMonsterHp = 300;
        let autoAttackInterval = null;

        let battleScene, battleCamera, battleRenderer, shipGroup, bossGroup, fxParticlesGroup, starsGroup;
        let oreScene, oreCamera, oreRenderer, oreMeshGroup, pickaxeMesh, isMiningAnim = false, miningframe = 0;
        let is3DInitialized = false;

        let isShipAttacking = false;
        let shipAnimFrame = 0;

        function startAction(action) {
            playSound('skill');
            const menu = document.getElementById("startMenu");
            menu.style.opacity = '0';
            setTimeout(() => {
                menu.style.display = 'none';
                if (!is3DInitialized) {
                    init3DScenes();
                    is3DInitialized = true;
                }
                switchTab(action);
            }, 500);
        }

        function returnToMenu() {
            if (autoAttackInterval) { clearInterval(autoAttackInterval); autoAttackInterval = null; }
            const menu = document.getElementById("startMenu");
            menu.style.display = 'flex';
            setTimeout(() => menu.style.opacity = '1', 10);
        }

        function resetData() {
            if (confirm("Xóa toàn bộ dữ liệu lưu và chơi lại từ đầu?")) {
                localStorage.removeItem("nextGenCosmicSave");
                location.reload();
            }
        }

        function init3DScenes() {
            // Battle Arena 3D Scene
            const battleContainer = document.getElementById("battle3DCanvas");
            battleScene = new THREE.Scene();

            battleCamera = new THREE.PerspectiveCamera(40, battleContainer.clientWidth / battleContainer.clientHeight, 0.1, 1000);
            battleCamera.position.set(0, 1.8, 7.5);
            battleCamera.lookAt(0, 0.4, 0);

            battleRenderer = new THREE.WebGLRenderer({ alpha: true, antialias: true, powerPreference: "high-performance" });
            battleRenderer.setSize(battleContainer.clientWidth, battleContainer.clientHeight);
            battleRenderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            battleContainer.appendChild(battleRenderer.domElement);

            battleScene.add(new THREE.DirectionalLight(0xffffff, 3.2).translateX(4).translateY(6).translateZ(4));
            battleScene.add(new THREE.DirectionalLight(0x38bdf8, 2.5).translateX(-4).translateY(-2).translateZ(-2));
            battleScene.add(new THREE.AmbientLight(0xffffff, 0.9));

            // Vũ trụ ngàn sao
            starsGroup = new THREE.Group();
            for(let i=0; i<200; i++) {
                const starGeo = new THREE.SphereGeometry(0.03 + Math.random()*0.04, 4, 4);
                const starMat = new THREE.MeshBasicMaterial({ color: 0xffffff });
                const star = new THREE.Mesh(starGeo, starMat);
                star.position.set((Math.random()-0.5)*20, (Math.random()-0.5)*15, (Math.random()-0.5)*15);
                starsGroup.add(star);
            }
            battleScene.add(starsGroup);

            // PHI THUYỀN ĐỨNG TRÁI, PHẢN DIỆN ĐỨNG PHẢI
            shipGroup = new THREE.Group();
            shipGroup.position.set(-1.8, -0.4, 0);
            shipGroup.rotation.y = Math.PI / 2;
            battleScene.add(shipGroup);
            createSpaceshipModel();

            bossGroup = new THREE.Group();
            bossGroup.position.set(1.8, -0.4, 0);
            bossGroup.rotation.y = -Math.PI / 2;
            battleScene.add(bossGroup);

            fxParticlesGroup = new THREE.Group();
            battleScene.add(fxParticlesGroup);

            updateBoss3DModel('station');

            // Ore 3D Scene
            const oreContainer = document.getElementById("ore3DContainer");
            oreScene = new THREE.Scene();
            oreCamera = new THREE.PerspectiveCamera(45, oreContainer.clientWidth / oreContainer.clientHeight, 0.1, 1000);
            oreCamera.position.set(0, 0, 5.5);
            oreRenderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
            oreRenderer.setSize(oreContainer.clientWidth, oreContainer.clientHeight);
            oreContainer.appendChild(oreRenderer.domElement);
            oreScene.add(new THREE.DirectionalLight(0xffffff, 3.0).translateX(3).translateY(5).translateZ(4));
            oreScene.add(new THREE.AmbientLight(0xffffff, 0.8));

            oreMeshGroup = new THREE.Group();
            const oreMesh = new THREE.Mesh(new THREE.DodecahedronGeometry(1.5, 1), new THREE.MeshStandardMaterial({ color: 0x0284c7, metalness: 0.9, roughness: 0.1, flatShading: true, emissive: 0x0369a1, emissiveIntensity: 0.8 }));
            oreMeshGroup.add(oreMesh);
            
            const pickGroup = new THREE.Group();
            const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.06, 0.08, 2.6, 8), new THREE.MeshStandardMaterial({ color: 0x38bdf8, emissive: 0x0284c7 }));
            handle.position.y = -0.8; pickGroup.add(handle);
            pickGroup.position.set(1.8, 1.2, 0.5); pickGroup.rotation.z = Math.PI / 4;
            pickaxeMesh = pickGroup; oreMeshGroup.add(pickaxeMesh);
            oreScene.add(oreMeshGroup);

            animate3D();

            window.addEventListener('keydown', (e) => {
                if (e.code === 'Space' && document.getElementById('pageBattle').classList.contains('flex')) {
                    e.preventDefault();
                    shipAttack();
                }
            });

            setInterval(updateCooldowns, 1000);
        }

        // TẠO MÔ HÌNH PHI THUYỀN KHÔNG GIANG 3D
        function createSpaceshipModel() {
            while(shipGroup.children.length > 0) shipGroup.remove(shipGroup.children[0]);

            const hullMat = new THREE.MeshStandardMaterial({ color: 0x0284c7, metalness: 0.95, roughness: 0.1 });
            const wingMat = new THREE.MeshStandardMaterial({ color: 0x38bdf8, metalness: 0.9, roughness: 0.2, emissive: 0x0369a1, emissiveIntensity: 0.5 });
            const glassMat = new THREE.MeshStandardMaterial({ color: 0x38bdf8, metalness: 0.9, transparent: true, opacity: 0.8 });

            // Thân chính phi thuyền
            const body = new THREE.Mesh(new THREE.ConeGeometry(0.5, 2.2, 8), hullMat);
            body.rotation.x = Math.PI / 2; shipGroup.add(body);

            // Buồng lái kính
            const cockpit = new THREE.Mesh(new THREE.SphereGeometry(0.3, 16, 16), glassMat);
            cockpit.position.set(0, 0.25, 0.3); shipGroup.add(cockpit);

            // Cánh phi thuyền
            const wingLeft = new THREE.Mesh(new THREE.BoxGeometry(1.6, 0.08, 0.6), wingMat);
            wingLeft.position.set(0, 0, -0.4); shipGroup.add(wingLeft);
        }

        // ================= XÂY DỰNG MÔ HÌNH 5 BOSS VŨ TRỤ 3D =================
        function createBossMesh(type) {
            const group = new THREE.Group();

            if (type === 'station') {
                // Trạm vũ trụ xoay
                const metalMat = new THREE.MeshStandardMaterial({ color: 0x475569, metalness: 0.9, roughness: 0.2 });
                const coreMat = new THREE.MeshStandardMaterial({ color: 0x38bdf8, emissive: 0x0284c7, emissiveIntensity: 1.0 });

                const ring = new THREE.Mesh(new THREE.TorusGeometry(1.4, 0.18, 16, 32), metalMat);
                group.add(ring);

                const core = new THREE.Mesh(new THREE.SphereGeometry(0.7, 16, 16), coreMat);
                group.add(core);
            } 
            else if (type === 'titan') {
                // Thạch Anh Titan
                const rockMat = new THREE.MeshStandardMaterial({ color: 0x334155, roughness: 0.9, metalness: 0.5, flatShading: true });
                const crystalMat = new THREE.MeshStandardMaterial({ color: 0x38bdf8, emissive: 0x0284c7, emissiveIntensity: 0.9 });

                const body = new THREE.Mesh(new THREE.DodecahedronGeometry(1.4, 0), rockMat);
                group.add(body);

                for(let i=0; i<5; i++) {
                    const c = new THREE.Mesh(new THREE.ConeGeometry(0.3, 1.2, 4), crystalMat);
                    c.position.set((Math.random()-0.5)*1.8, (Math.random()-0.5)*1.8, (Math.random()-0.5)*1.8);
                    group.add(c);
                }
            } 
            else if (type === 'leviathan') {
                // HẢI VƯƠNG LEVIATHAN VŨ TRỤ
                const seaMat = new THREE.MeshStandardMaterial({ color: 0x06b6d4, metalness: 0.85, roughness: 0.15, emissive: 0x0891b2, emissiveIntensity: 0.7 });
                const finMat = new THREE.MeshStandardMaterial({ color: 0x38bdf8, metalness: 0.9 });

                const body = new THREE.Mesh(new THREE.CylinderGeometry(0.9, 1.3, 3.2, 12), seaMat);
                body.rotation.z = Math.PI / 6; group.add(body);

                const head = new THREE.Mesh(new THREE.ConeGeometry(1.1, 2.2, 6), seaMat);
                head.rotation.x = Math.PI / 2.3; head.position.set(0.6, 1.4, 0.3); group.add(head);

                for(let i=0; i<4; i++) {
                    const fin = new THREE.Mesh(new THREE.ConeGeometry(0.3, 1.2, 4), finMat);
                    fin.position.set(-0.2 - (i*0.4), 0.5 + (i*0.4), -0.6);
                    fin.rotation.z = -Math.PI / 4; group.add(fin);
                }
            }
            else if (type === 'dragon') {
                const scaleMat = new THREE.MeshStandardMaterial({ color: 0x991b1b, metalness: 0.8, emissive: 0x7f1d1d, emissiveIntensity: 0.6 });
                const head = new THREE.Mesh(new THREE.ConeGeometry(1.3, 3.2, 6), scaleMat);
                head.rotation.x = Math.PI / 2.1; group.add(head);
            } 
            else if (type === 'overlord') {
                const darkMat = new THREE.MeshStandardMaterial({ color: 0x09090b, roughness: 0.2 });
                const redMat = new THREE.MeshStandardMaterial({ color: 0xef4444, emissive: 0xb91c1c, emissiveIntensity: 0.9 });

                const head = new THREE.Mesh(new THREE.BoxGeometry(2.2, 2.2, 1.8), darkMat);
                group.add(head);

                for(let i = -1; i <= 1; i += 2) {
                    const horn = new THREE.Mesh(new THREE.ConeGeometry(0.4, 2.6, 6), redMat);
                    horn.position.set(i * 1.1, 1.8, -0.2); group.add(horn);
                }
            }
            return group;
        }

        function updateBoss3DModel(type) {
            while(bossGroup.children.length > 0) bossGroup.remove(bossGroup.children[0]);
            bossGroup.add(createBossMesh(type));
        }

        function resize3DCanvases() {
            setTimeout(() => {
                const battleContainer = document.getElementById("battle3DCanvas");
                if (battleContainer && battleRenderer && battleCamera) {
                    battleCamera.aspect = battleContainer.clientWidth / battleContainer.clientHeight;
                    battleCamera.updateProjectionMatrix(); battleRenderer.setSize(battleContainer.clientWidth, battleContainer.clientHeight);
                }
                const oreContainer = document.getElementById("ore3DContainer");
                if (oreContainer && oreRenderer && oreCamera) {
                    oreCamera.aspect = oreContainer.clientWidth / oreContainer.clientHeight;
                    oreCamera.updateProjectionMatrix(); oreRenderer.setSize(oreContainer.clientWidth, oreContainer.clientHeight);
                }
            }, 50);
        }

        let time3D = 0;
        function animate3D() {
            requestAnimationFrame(animate3D);
            time3D += 0.03;

            if (starsGroup) starsGroup.rotation.y += 0.001;

            if (bossGroup) {
                bossGroup.rotation.y = -Math.PI / 2 + Math.sin(time3D * 0.5) * 0.2;
                bossGroup.position.y = -0.4 + Math.sin(time3D * 2) * 0.1;
            }

            // Animation phi thuyền bắn lao tới
            if (isShipAttacking && shipGroup) {
                shipAnimFrame += 0.35;
                shipGroup.position.x = -1.8 + Math.sin(shipAnimFrame) * 1.6;
                
                if (shipAnimFrame > Math.PI) {
                    isShipAttacking = false;
                    shipAnimFrame = 0;
                    shipGroup.position.x = -1.8;
                }
            }

            // Cập nhật particle laser không gian 3D
            if (fxParticlesGroup) {
                for (let i = fxParticlesGroup.children.length - 1; i >= 0; i--) {
                    const p = fxParticlesGroup.children[i];
                    p.position.x += p.userData.vx;
                    p.position.y += p.userData.vy;
                    p.position.z += p.userData.vz;
                    p.scale.multiplyScalar(0.92);
                    if (p.scale.x < 0.05) fxParticlesGroup.remove(p);
                }
            }

            if (oreMeshGroup) {
                oreMeshGroup.rotation.y += 0.015;
            }

            if (battleRenderer && battleScene && battleCamera) battleRenderer.render(battleScene, battleCamera);
            if (oreRenderer && oreScene && oreCamera) oreRenderer.render(oreScene, oreCamera);
        }

        function switchTab(tab) {
            const pageBattle = document.getElementById("pageBattle");
            const pageStats = document.getElementById("pageStats");
            const pageGather = document.getElementById("pageGather");

            pageBattle.classList.add("hidden"); pageBattle.classList.remove("flex");
            pageStats.classList.add("hidden"); pageStats.classList.remove("grid");
            pageGather.classList.add("hidden"); pageGather.classList.remove("flex");

            if (tab === 'battle') {
                pageBattle.classList.remove("hidden"); pageBattle.classList.add("flex");
                resize3DCanvases();
            } else if (tab === 'stats') {
                pageStats.classList.remove("hidden"); pageStats.classList.add("grid");
            } else if (tab === 'gather') {
                pageGather.classList.remove("hidden"); pageGather.classList.add("flex");
                resize3DCanvases();
            }
        }

        function getPlayerStats() {
            let baseAtk = 20 + (player.stats.atkLvl * 10);
            let baseCrit = 5 + (player.stats.critLvl * 2);
            if (tempBerserkActive) baseCrit = 100;
            return { atk: baseAtk, crit: baseCrit };
        }

        function shipAttack() {
            const d = dungeons[currentDungeonIdx];
            if (currentMonsterHp <= 0) return;

            isShipAttacking = true; shipAnimFrame = 0;
            const stats = getPlayerStats();
            const isCrit = Math.random() * 100 < stats.crit;
            let dmg = stats.atk; if (isCrit) dmg = Math.floor(dmg * 2.0);

            currentMonsterHp -= dmg;
            const arena = document.getElementById("battleArena");
            arena.classList.remove("shake"); void arena.offsetWidth; arena.classList.add("shake");

            playSound(isCrit ? 'crit' : 'hit');
            spawnDamagePop(dmg, isCrit);
            spawn3DSparkEffect(0x38bdf8, 12);

            if (currentMonsterHp <= 0) { currentMonsterHp = 0; monsterKilled(); }
            updateBattleDisplay();
        }

        function spawn3DSparkEffect(hexColor, count) {
            for(let i=0; i<count; i++) {
                const pGeo = new THREE.SphereGeometry(0.12, 8, 8);
                const pMat = new THREE.MeshBasicMaterial({ color: hexColor });
                const p = new THREE.Mesh(pGeo, pMat);
                p.position.set(0.8 + (Math.random()-0.5)*0.8, (Math.random()-0.5)*0.8, (Math.random()-0.5)*0.8);
                p.userData = {
                    vx: (Math.random() - 0.5) * 0.18,
                    vy: (Math.random() - 0.2) * 0.18,
                    vz: (Math.random() - 0.5) * 0.18
                };
                fxParticlesGroup.add(p);
            }
        }

        function upgradeStat(type) {
            if (type === 'atk') {
                const cost = (player.stats.atkLvl + 1) * 200;
                if (player.gold >= cost) { player.gold -= cost; player.stats.atkLvl++; log("Nâng cấp ATK Laser thành công!"); }
                else log("Không đủ Năng lượng!");
            } else if (type === 'crit') {
                const cost = (player.stats.critLvl + 1) * 400;
                if (player.gold >= cost) { player.gold -= cost; player.stats.critLvl++; log("Nâng cấp CRIT thành công!"); }
                else log("Không đủ Năng lượng!");
            } else if (type === 'gold') {
                const cost = (player.stats.goldLvl + 1) * 300;
                if (player.gold >= cost) { player.gold -= cost; player.stats.goldLvl++; log("Nâng cấp Thu hoạch thành công!"); }
                else log("Không đủ Năng lượng!");
            }
            updateUI();
        }

        function upgradeSkill(id) {
            if (id === 1) {
                const cost = player.skills.s1.lvl * 10;
                if (player.darkMatter >= cost) { player.darkMatter -= cost; player.skills.s1.lvl++; log("Cường hóa Bão Plasma thành công!"); }
                else log("Không đủ Vật chất tối!");
            } else if (id === 2) {
                const cost = player.skills.s2.lvl * 15;
                if (player.darkMatter >= cost) { player.darkMatter -= cost; player.skills.s2.lvl++; log("Cường hóa Khiên quang học thành công!"); }
                else log("Không đủ Vật chất tối!");
            } else if (id === 3) {
                const cost = player.skills.s3.lvl * 25;
                if (player.darkMatter >= cost) { player.darkMatter -= cost; player.skills.s3.lvl++; log("Cường hóa Pháo Laser Vực thẳm!"); }
                else log("Không đủ Vật chất tối!");
            }
            updateUI();
        }

        function useSkill(id) {
            const d = dungeons[currentDungeonIdx];
            if (currentMonsterHp <= 0) return;

            if (id === 1 && player.skills.s1.cd === 0) {
                const stats = getPlayerStats();
                const mult = 3 + player.skills.s1.lvl;
                const dmg = stats.atk * mult;
                currentMonsterHp -= dmg;
                player.skills.s1.cd = player.skills.s1.maxCd;
                playSound('skill');
                spawnDamagePop(`🔥 BÃO PLASMA -${dmg.toLocaleString()}`, true);
                spawn3DSparkEffect(0xef4444, 30);
                log(`Thi triển Bão Plasma 3D gây ${dmg.toLocaleString()} ST!`);
            } else if (id === 2 && player.skills.s2.cd === 0) {
                tempBerserkActive = true;
                const dur = 4 + player.skills.s2.lvl;
                player.skills.s2.cd = player.skills.s2.maxCd;
                playSound('skill');
                spawn3DSparkEffect(0x38bdf8, 25);
                log(`Kích hoạt Khiên Quang Học: 100% Chí Mạng trong ${dur}s!`);
                setTimeout(() => { tempBerserkActive = false; log("Khiên quang học đã hết."); }, dur * 1000);
            } else if (id === 3 && player.skills.s3.cd === 0) {
                const pct = 8 + (player.skills.s3.lvl * 4);
                const dmg = Math.floor(d.maxHp * (pct / 100));
                currentMonsterHp -= dmg;
                player.skills.s3.cd = player.skills.s3.maxCd;
                playSound('skill');
                spawnDamagePop(`💀 PHÁO LASER VỰC THẲM -${dmg.toLocaleString()}`, true);
                spawn3DSparkEffect(0xa855f7, 35);
                log(`Thi triển Pháo Laser Vực Thẳm trừ ${pct}% Max HP Boss!`);
            }

            if (currentMonsterHp <= 0) { currentMonsterHp = 0; monsterKilled(); }
            updateBattleDisplay();
            updateSkillButtons();
        }

        function updateCooldowns() {
            if (player.skills.s1.cd > 0) player.skills.s1.cd--;
            if (player.skills.s2.cd > 0) player.skills.s2.cd--;
            if (player.skills.s3.cd > 0) player.skills.s3.cd--;
            updateSkillButtons();
        }

        function updateSkillButtons() {
            document.getElementById("cdSkill1").innerText = player.skills.s1.cd;
            document.getElementById("cdSkill2").innerText = player.skills.s2.cd;
            document.getElementById("cdSkill3").innerText = player.skills.s3.cd;
            document.getElementById("btnSkill1").disabled = player.skills.s1.cd > 0;
            document.getElementById("btnSkill2").disabled = player.skills.s2.cd > 0;
            document.getElementById("btnSkill3").disabled = player.skills.s3.cd > 0;
        }

        function performGather() {
            playSound('hit');
            const addCrystal = Math.floor(Math.random() * 15) + 5;
            const addDarkMatter = Math.random() > 0.5 ? Math.floor(Math.random() * 3) + 1 : 0;
            const addGold = Math.floor(Math.random() * 60) + 30;

            player.crystal += addCrystal; player.darkMatter += addDarkMatter; player.gold += addGold;
            const zone = document.getElementById("gatherZone");
            zone.classList.remove("shake"); void zone.offsetWidth; zone.classList.add("shake");
            spawnGatherPop(`+${addCrystal} Tinh Thể ${addDarkMatter > 0 ? `, +${addDarkMatter} Vật Chất Tối` : ''}`);
            log(`💎 Khai thác thành công: +${addCrystal} Tinh Thể, +${addGold} Năng Lượng!`);
            updateUI();
        }

        function spawnGatherPop(text) {
            const zone = document.getElementById("gatherZone");
            const pop = document.createElement("div");
            pop.className = "damage-pop text-cyan-300 text-sm font-bold";
            pop.innerText = text; pop.style.left = (Math.random() * 40 + 30) + "%"; pop.style.top = "30%";
            zone.appendChild(pop); setTimeout(() => pop.remove(), 850);
        }

        function selectDungeon(idx) {
            currentDungeonIdx = idx;
            currentMonsterHp = dungeons[idx].maxHp;
            updateBoss3DModel(dungeons[idx].bossType);
            updateBattleDisplay();
        }

        function updateBattleDisplay() {
            const d = dungeons[currentDungeonIdx];
            document.getElementById("dungeonLevelBadge").innerText = `MÀN ${d.level}`;
            document.getElementById("monsterName").innerText = d.monster;
            const pct = Math.max(0, (currentMonsterHp / d.maxHp) * 100);
            document.getElementById("monsterHpBar").style.width = pct + "%";
            document.getElementById("monsterHpText").innerText = `${Math.max(0, currentMonsterHp).toLocaleString()} / ${d.maxHp.toLocaleString()} HP`;
        }

        function spawnDamagePop(dmg, isCrit) {
            const arena = document.getElementById("battleArena");
            const pop = document.createElement("div");
            pop.className = `damage-pop ${isCrit ? 'text-cyan-400 text-2xl' : 'text-sky-300 text-lg'}`;
            pop.innerText = typeof dmg === 'string' ? dmg : ((isCrit ? "💥 CRIT " : "") + "-" + dmg.toLocaleString());
            pop.style.left = (Math.random() * 40 + 50) + "%"; pop.style.top = "40%";
            arena.appendChild(pop); setTimeout(() => pop.remove(), 850);
        }

        function monsterKilled() {
            const d = dungeons[currentDungeonIdx];
            playSound('win');
            const bonusGoldMult = 1 + (player.stats.goldLvl * 0.15);
            const totalGold = Math.floor(d.goldReward * bonusGoldMult);

            player.gold += totalGold; player.darkMatter += d.darkMatterDrop;
            log(`🏆 Đã hạ gục ${d.monster}! Nhận +${totalGold.toLocaleString()} Năng Lượng.`);

            if (currentDungeonIdx < dungeons.length - 1) {
                setTimeout(() => {
                    currentDungeonIdx++; currentMonsterHp = dungeons[currentDungeonIdx].maxHp;
                    updateBoss3DModel(dungeons[currentDungeonIdx].bossType); updateBattleDisplay();
                    log(`🚀 Đã tiến tới tọa độ ${dungeons[currentDungeonIdx].name}!`);
                }, 1500);
            } else {
                setTimeout(() => {
                    document.getElementById("evoModal").classList.remove("hidden");
                }, 600);
            }
            updateUI();
        }

        function chooseEvoOption(opt) {
            document.getElementById("evoModal").classList.add("hidden");
            if (opt === 'evolve') {
                player.bossEvoTier++;
                log(`🧬 BOSS THIÊN HÀ TIẾN HOÁ TIER ${player.bossEvoTier}!`);
            } else {
                player.bossEvoTier = 1;
                log(`🔄 Đã làm mới các Boss về Tier 1 Gốc!`);
            }
            recalculateDungeons();
            currentDungeonIdx = 0;
            currentMonsterHp = dungeons[0].maxHp;
            updateBoss3DModel(dungeons[0].bossType);
            updateUI();
        }

        function toggleAutoAttack() {
            const btn = document.getElementById("btnAutoAttack");
            if (autoAttackInterval) {
                clearInterval(autoAttackInterval); autoAttackInterval = null;
                btn.innerText = "🤖 Tự Động: OFF"; btn.className = "py-3 px-4 bg-slate-900 border border-slate-700 hover:bg-slate-800 font-semibold rounded-xl text-slate-200 transition text-xs";
            } else {
                autoAttackInterval = setInterval(() => { shipAttack(); }, 800);
                btn.innerText = "🤖 Tự Động: ON"; btn.className = "py-3 px-4 bg-cyan-700/60 border border-cyan-400 hover:bg-cyan-700 font-bold rounded-xl text-cyan-200 transition text-xs shadow-lg";
            }
        }

        function log(msg) { document.getElementById("gameLog").innerText = msg; }

        function updateUI() {
            recalculateDungeons();

            document.getElementById("bossEvoLvl").innerText = `Tier ${player.bossEvoTier} (${player.bossEvoTier > 1 ? '+' + ((player.bossEvoTier-1)*50) + '% Stats' : 'Gốc'})`;
            document.getElementById("resGold").innerText = player.gold.toLocaleString();
            document.getElementById("resCrystal").innerText = player.crystal.toLocaleString();
            document.getElementById("resDarkMatter").innerText = player.darkMatter.toLocaleString();

            document.getElementById("statAtkLvl").innerText = player.stats.atkLvl;
            document.getElementById("statCritLvl").innerText = player.stats.critLvl;
            document.getElementById("statGoldLvl").innerText = player.stats.goldLvl;
            document.getElementById("costStatAtk").innerText = ((player.stats.atkLvl + 1) * 200).toLocaleString();
            document.getElementById("costStatCrit").innerText = ((player.stats.critLvl + 1) * 400).toLocaleString();
            document.getElementById("costStatGold").innerText = ((player.stats.goldLvl + 1) * 300).toLocaleString();

            document.getElementById("skill1Lvl").innerText = player.skills.s1.lvl;
            document.getElementById("skill1Mult").innerText = 3 + player.skills.s1.lvl;
            document.getElementById("costSkill1").innerText = player.skills.s1.lvl * 10;

            document.getElementById("skill2Lvl").innerText = player.skills.s2.lvl;
            document.getElementById("skill2Dur").innerText = 4 + player.skills.s2.lvl;
            document.getElementById("costSkill2").innerText = player.skills.s2.lvl * 15;

            document.getElementById("skill3Lvl").innerText = player.skills.s3.lvl;
            document.getElementById("skill3Pct").innerText = 8 + (player.skills.s3.lvl * 4);
            document.getElementById("costSkill3").innerText = player.skills.s3.lvl * 25;

            updateBattleDisplay(); saveGame();
        }

        function saveGame() { localStorage.setItem("nextGenCosmicSave", JSON.stringify(player)); }
        function loadGame() {
            const data = localStorage.getItem("nextGenCosmicSave");
            if (data) { try { player = Object.assign(player, JSON.parse(data)); } catch(e) {} }
            recalculateDungeons();
            updateUI();
        }

        window.onload = () => { loadGame(); window.addEventListener('resize', resize3DCanvases); };
    </script>
</body>
</html>
