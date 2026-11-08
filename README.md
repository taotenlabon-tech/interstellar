<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cosmic Defender: Starfighter 3D</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@700;900&family=Inter:wght@400;600&display=swap');
        body { font-family: 'Inter', sans-serif; background: #020617; color: #f8fafc; overflow: hidden; margin: 0; }
        .font-orbitron { font-family: 'Orbitron', sans-serif; }
        .glass-hud { background: rgba(15, 23, 42, 0.85); backdrop-filter: blur(12px); border: 1px solid rgba(56, 189, 248, 0.3); }
    </style>
</head>
<body class="w-screen h-screen flex flex-col items-center justify-between select-none">

    <!-- MÀN HÌNH BẮT ĐẦU -->
    <div id="startMenu" class="fixed inset-0 z-50 bg-[radial-gradient(ellipse_at_center,_#0f172a_0%,_#020617_100%)] flex flex-col items-center justify-center p-6 text-center">
        <div class="max-w-md w-full glass-hud p-8 rounded-3xl border border-cyan-500/50 shadow-[0_0_50px_rgba(56,189,248,0.3)] space-y-6">
            <div class="text-6xl animate-pulse">🚀</div>
            <h1 class="font-orbitron text-3xl font-black text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 to-sky-500">
                COSMIC DEFENDER
            </h1>
            <p class="text-xs text-slate-300 leading-relaxed">
                Lái phi thuyền né đạn, tiêu diệt quái vật vũ trụ và đánh bại <span class="text-cyan-400 font-bold">Leviathan Không Gian</span>!
            </p>
            <div class="text-xs text-slate-400 bg-slate-900/80 p-3 rounded-xl border border-slate-800 space-y-1">
                <div>🎮 **Điều khiển:** Chuột (hoặc phím W, A, S, D) để di chuyển.</div>
                <div>💥 **Tự động bắn:** Phi thuyền liên tục xả đạn Laser.</div>
            </div>
            <button onclick="startGame()" class="w-full py-4 bg-gradient-to-r from-cyan-600 to-blue-600 hover:from-cyan-500 hover:to-blue-500 font-orbitron font-bold rounded-2xl text-white shadow-lg shadow-cyan-500/40 active:scale-95 transition">
                ⚡ XUẤT KÍCH NGAY
            </button>
        </div>
    </div>

    <!-- HUD TRONG TRẬN ĐẤU -->
    <div id="gameHud" class="absolute top-4 left-4 right-4 z-20 flex justify-between items-center pointer-events-none hidden max-w-5xl mx-auto">
        <div class="glass-hud px-4 py-2 rounded-xl text-xs flex gap-4">
            <div>❤️ Mạng: <span id="hudLives" class="text-rose-400 font-bold">3</span></div>
            <div>🏆 Điểm: <span id="hudScore" class="text-yellow-400 font-bold">0</span></div>
        </div>
        <div class="glass-hud px-4 py-2 rounded-xl text-xs">
            🌌 Mục tiêu: Tiêu diệt <span id="hudBossName" class="text-cyan-300 font-bold">Quái vật Thiên Hà</span>
        </div>
    </div>

    <!-- CANVAS 3D KHÔNG GIAN -->
    <div id="canvasContainer" class="absolute inset-0 z-0"></div>

    <!-- MÀN HÌNH GAME OVER / CHIẾN THẮNG -->
    <div id="endModal" class="fixed inset-0 z-50 bg-slate-950/90 flex flex-col items-center justify-center p-6 hidden">
        <div class="max-w-md w-full glass-hud p-8 rounded-3xl text-center space-y-5 border border-cyan-400">
            <h2 id="endTitle" class="text-2xl font-bold font-orbitron text-cyan-300">THẮNG LỢI VŨ TRỤ!</h2>
            <p id="endDesc" class="text-xs text-slate-300">Bạn đã dọn sạch không gian thiên hà khỏi thế lực hắc ám.</p>
            <button onclick="location.reload()" class="w-full py-3.5 bg-gradient-to-r from-cyan-600 to-blue-600 font-bold rounded-xl text-white text-sm shadow-lg">
                🔄 CHƠI LẠI
            </button>
        </div>
    </div>

    <script>
        let scene, camera, renderer;
        let ship, stars = [], bullets = [], enemies = [], enemyBullets = [], particles = [];
        let score = 0, lives = 3, gameActive = false;
        let mouse = { x: 0, y: 0 };
        let keys = { w: false, a: false, s: false, d: false };
        let shootTimer = 0;
        let bossSpawned = false, boss = null, bossHp = 100, maxBossHp = 100;

        function startGame() {
            document.getElementById('startMenu').style.display = 'none';
            document.getElementById('gameHud').style.display = 'flex';
            init3D();
            gameActive = true;
        }

        function init3D() {
            const container = document.getElementById('canvasContainer');
            scene = new THREE.Scene();
            scene.fog = new THREE.FogExp2(0x020617, 0.015);

            camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 10, 12);
            camera.lookAt(0, 0, -3);

            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            container.appendChild(renderer.domElement);

            // Ánh sáng
            scene.add(new THREE.DirectionalLight(0xffffff, 2.5).translateX(5).translateY(10).translateZ(5));
            scene.add(new THREE.AmbientLight(0xffffff, 0.8));

            // Tạo các ngôi sao trôi nền
            for (let i = 0; i < 400; i++) {
                const sGeo = new THREE.SphereGeometry(0.05 + Math.random() * 0.08, 4, 4);
                const sMat = new THREE.MeshBasicMaterial({ color: Math.random() > 0.2 ? 0xffffff : 0x38bdf8 });
                const star = new THREE.Mesh(sGeo, sMat);
                star.position.set((Math.random() - 0.5) * 50, (Math.random() - 0.5) * 30, (Math.random() - 0.5) * 80);
                scene.add(star);
                stars.push(star);
            }

            // Tạo Phi thuyền người chơi (Hero Ship)
            const shipGroup = new THREE.Group();
            const hullMat = new THREE.MeshStandardMaterial({ color: 0x0284c7, metalness: 0.9, roughness: 0.2 });
            const wingMat = new THREE.MeshStandardMaterial({ color: 0x38bdf8, metalness: 0.8 });
            
            const body = new THREE.Mesh(new THREE.ConeGeometry(0.6, 2.2, 6), hullMat);
            body.rotation.x = Math.PI / 2; shipGroup.add(body);

            const wingL = new THREE.Mesh(new THREE.BoxGeometry(1.6, 0.05, 0.5), wingMat);
            wingL.position.set(0, 0, -0.3); shipGroup.add(wingL);

            ship = shipGroup;
            ship.position.set(0, 0, 4);
            scene.add(ship);

            // Lắng nghe sự kiện di chuyển
            window.addEventListener('mousemove', (e) => {
                mouse.x = (e.clientX / window.innerWidth) * 2 - 1;
                mouse.y = -(e.clientY / window.innerHeight) * 2 + 1;
            });
            window.addEventListener('keydown', (e) => handleKey(e, true));
            window.addEventListener('keyup', (e) => handleKey(e, false));
            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            });

            animate();
        }

        function handleKey(e, isDown) {
            if (e.key === 'w' || e.key === 'ArrowUp') keys.w = isDown;
            if (e.key === 'a' || e.key === 'ArrowLeft') keys.a = isDown;
            if (e.key === 's' || e.key === 'ArrowDown') keys.s = isDown;
            if (e.key === 'd' || e.key === 'ArrowRight') keys.d = isDown;
        }

        function spawnEnemy() {
            if (!gameActive || bossSpawned) return;
            if (Math.random() < 0.03 && enemies.length < 5) {
                const eGroup = new THREE.Group();
                const eMat = new THREE.MeshStandardMaterial({ color: 0xef4444, metalness: 0.8, emissive: 0xb91c1c, emissiveIntensity: 0.5 });
                const eMesh = new THREE.Mesh(new THREE.DodecahedronGeometry(0.8, 0), eMat);
                eGroup.add(eMesh);
                eGroup.position.set((Math.random() - 0.5) * 16, 0, -25);
                scene.add(eGroup);
                enemies.push(eGroup);
            }
        }

        function spawnBoss() {
            bossSpawned = true;
            document.getElementById('hudBossName').innerText = "Thủy Tộc Leviathan Vũ Trụ 🌌";
            
            const bGroup = new THREE.Group();
            const bMat = new THREE.MeshStandardMaterial({ color: 0x06b6d4, metalness: 0.9, emissive: 0x0891b2, emissiveIntensity: 0.8 });
            
            const head = new THREE.Mesh(new THREE.ConeGeometry(2.0, 4.0, 8), bMat);
            head.rotation.x = Math.PI / 2; bGroup.add(head);

            bGroup.position.set(0, 0, -22);
            scene.add(bGroup);
            boss = bGroup;
        }

        function shootBullet() {
            const bGeo = new THREE.CylinderGeometry(0.08, 0.08, 0.8, 6);
            const bMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8 });
            const bullet = new THREE.Mesh(bGeo, bMat);
            bullet.rotation.x = Math.PI / 2;
            bullet.position.copy(ship.position);
            scene.add(bullet);
            bullets.push(bullet);
        }

        function animate() {
            if (!gameActive) return;
            requestAnimationFrame(animate);

            // Di chuyển phi thuyền theo phím hoặc chuột
            let targetX = mouse.x * 8;
            let targetZ = 4 + (-mouse.y * 5);
            if (keys.a) ship.position.x -= 0.2;
            if (keys.d) ship.position.x += 0.2;
            if (keys.w) ship.position.z -= 0.2;
            if (keys.s) ship.position.z += 0.2;
            
            ship.position.x += (targetX - ship.position.x) * 0.1;
            ship.position.z += (targetZ - ship.position.z) * 0.1;
            ship.position.x = Math.max(-9, Math.min(9, ship.position.x));
            ship.position.z = Math.max(-2, Math.min(10, ship.position.z));
            ship.rotation.z = (ship.position.x - targetX) * 0.05;

            // Tự động bắn đạn
            shootTimer++;
            if (shootTimer % 12 === 0) shootBullet();

            // Cập nhật sao trôi
            stars.forEach(s => {
                s.position.z += 0.4;
                if (s.position.z > 15) s.position.z = -70;
            });

            // Cập nhật đạn người chơi
            for (let i = bullets.length - 1; i >= 0; i--) {
                bullets[i].position.z -= 0.8;
                if (bullets[i].position.z < -40) {
                    scene.remove(bullets[i]);
                    bullets.splice(i, 1);
                }
            }

            // Sinh quái vật thường
            spawnEnemy();
            if (score >= 100 && !bossSpawned) spawnBoss();

            // Cập nhật quái vật nhỏ
            for (let i = enemies.length - 1; i >= 0; i--) {
                const en = enemies[i];
                en.position.z += 0.25;
                en.rotation.y += 0.03;

                // Va chạm đạn với quái thường
                for (let j = bullets.length - 1; j >= 0; j--) {
                    if (en.position.distanceTo(bullets[j].position) < 1.2) {
                        scene.remove(en); enemies.splice(i, 1);
                        scene.remove(bullets[j]); bullets.splice(j, 1);
                        score += 10;
                        document.getElementById('hudScore').innerText = score;
                        break;
                    }
                }

                // Va chạm quái với phi thuyền
                if (en && en.position.distanceTo(ship.position) < 1.5) {
                    scene.remove(en); enemies.splice(i, 1);
                    lives--;
                    document.getElementById('hudLives').innerText = lives;
                    if (lives <= 0) gameOver(false);
                }

                if (en && en.position.z > 10) {
                    scene.remove(en); enemies.splice(i, 1);
                }
            }

            // Cập nhật Boss Leviathan
            if (bossSpawned && boss) {
                boss.position.x = Math.sin(Date.now() * 0.002) * 5;
                boss.rotation.y += 0.02;

                // Boss bắn đạn phản công
                if (Math.random() < 0.04) {
                    const ebGeo = new THREE.SphereGeometry(0.25, 8, 8);
                    const ebMat = new THREE.MeshBasicMaterial({ color: 0xef4444 });
                    const eb = new THREE.Mesh(ebGeo, ebMat);
                    eb.position.copy(boss.position);
                    scene.add(eb);
                    enemyBullets.push(eb);
                }

                // Đạn trúng Boss
                for (let j = bullets.length - 1; j >= 0; j--) {
                    if (boss.position.distanceTo(bullets[j].position) < 2.5) {
                        scene.remove(bullets[j]); bullets.splice(j, 1);
                        bossHp -= 5;
                        if (bossHp <= 0) {
                            scene.remove(boss);
                            boss = null;
                            gameOver(true);
                        }
                        break;
                    }
                }
            }

            // Cập nhật đạn Boss
            for (let i = enemyBullets.length - 1; i >= 0; i--) {
                const eb = enemyBullets[i];
                eb.position.z += 0.35;
                if (eb.position.distanceTo(ship.position) < 1.2) {
                    scene.remove(eb); enemyBullets.splice(i, 1);
                    lives--;
                    document.getElementById('hudLives').innerText = lives;
                    if (lives <= 0) gameOver(false);
                }
                if (eb.position.z > 15) {
                    scene.remove(eb); enemyBullets.splice(i, 1);
                }
            }

            renderer.render(scene, camera);
        }

        function gameOver(isWin) {
            gameActive = false;
            document.getElementById('gameHud').style.display = 'none';
            document.getElementById('endModal').classList.remove('hidden');
            if (isWin) {
                document.getElementById('endTitle').innerText = "🌌 CHIẾN THẮNG VŨ TRỤ!";
                document.getElementById('endDesc').innerText = `Bạn đã tiêu diệt Leviathan Không Gian với tổng số điểm: ${score}!`;
            } else {
                document.getElementById('endTitle').innerText = "💥 PHI THUYỀN BỊ PHÁ HUỶ!";
                document.getElementById('endDesc').innerText = `Điểm số đạt được: ${score}. Hãy thử lại nhé!`;
            }
        }
    </script>
</body>
</html>
