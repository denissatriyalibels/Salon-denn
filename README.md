<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Ninja Blade: Fruit Slash | Game Iris Buah</title>
    <style>
        * {
            user-select: none;
            -webkit-tap-highlight-color: transparent;
            box-sizing: border-box;
        }

        body {
            margin: 0;
            min-height: 100vh;
            background: radial-gradient(circle at 20% 30%, #0a2f2a, #051f1b);
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Segoe UI', 'Poppins', 'Righteous', cursive, system-ui;
            padding: 16px;
        }

        .game-container {
            background: #1e3b32;
            border-radius: 56px;
            padding: 16px 20px 20px;
            box-shadow: 0 30px 35px rgba(0,0,0,0.5), inset 0 1px 4px rgba(255,255,200,0.2);
        }

        canvas {
            display: block;
            margin: 0 auto;
            border-radius: 32px;
            box-shadow: 0 10px 20px rgba(0,0,0,0.4);
            cursor: crosshair;
            background: #87CEEB;
            background-image: radial-gradient(circle at 10% 20%, #5fa8b3 2%, transparent 2.5%);
            background-size: 28px 28px;
        }

        .info-panel {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-top: 20px;
            gap: 12px;
            flex-wrap: wrap;
            background: #2c4a3bdd;
            backdrop-filter: blur(8px);
            padding: 8px 20px;
            border-radius: 80px;
            border: 1px solid #ffd966aa;
        }

        .score-box, .combo-box, .health-box {
            background: #231f1b;
            padding: 6px 20px;
            border-radius: 60px;
            color: #ffe6b3;
            font-weight: bold;
            font-size: 1.3rem;
            display: flex;
            align-items: center;
            gap: 10px;
            box-shadow: inset 0 1px 2px #fffaec, 0 5px 10px rgba(0,0,0,0.2);
        }

        .score-box span:first-child, .combo-box span:first-child, .health-box span:first-child {
            font-size: 1.5rem;
        }

        .value {
            font-size: 1.7rem;
            font-weight: 800;
            color: #ffbb66;
            text-shadow: 0 2px 0 #5a2e1a;
        }

        .reset-btn {
            background: #e68a2e;
            border: none;
            font-size: 1.2rem;
            font-weight: bold;
            padding: 8px 28px;
            border-radius: 40px;
            font-family: inherit;
            cursor: pointer;
            color: #261c0f;
            transition: 0.07s linear;
            box-shadow: 0 6px 0 #8b4c1a;
        }

        .reset-btn:active {
            transform: translateY(3px);
            box-shadow: 0 2px 0 #8b4c1a;
        }

        @media (max-width: 600px) {
            .score-box, .combo-box, .health-box { font-size: 1rem; padding: 4px 14px; }
            .value { font-size: 1.3rem; }
            .reset-btn { padding: 6px 20px; font-size: 1rem; }
            .info-panel { justify-content: center; }
        }
    </style>
</head>
<body>
<div class="game-container">
    <canvas id="gameCanvas" width="900" height="600"></canvas>
    <div class="info-panel">
        <div class="score-box"><span>🍉</span> SKOR <span class="value" id="scoreValue">0</span></div>
        <div class="combo-box"><span>⚡</span> COMBO <span class="value" id="comboValue">0x</span></div>
        <div class="health-box"><span>❤️</span> NYAWA <span class="value" id="healthValue">3</span></div>
        <button class="reset-btn" id="resetGameBtn">🗡️ Potong Baru</button>
    </div>
</div>

<script>
    (function(){
        // ---------- KONFIGURASI CANVAS ----------
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        
        // Ukuran canvas tetap 900x600
        const W = 900, H = 600;
        canvas.width = W;
        canvas.height = H;
        
        // ---------- PARAMETER GAME ----------
        let score = 0;
        let lives = 3;           // nyawa
        let combo = 0;           // combo saat ini
        let maxCombo = 0;        // untuk semangat
        let gameRunning = true;
        let frameRequest;
        let lastTimestamp = 0;
        
        // Daftar objek buah / bom
        let fruits = [];
        let explosions = [];      // efek visual
        
        // kontrol swipe / mouse trail (ninja slice)
        let mousePositions = [];   // simpan posisi terakhir untuk garis putus-putus
        let isPointerDown = false;
        let lastSliceTime = 0;
        
        // spawn rate control
        let lastSpawnTime = 0;
        let spawnInterval = 450;   // ms antar spawn (bisa berubah sesuai kesulitan)
        
        // --- Koleksi buah dan bom ---
        const fruitTypes = [
            { name: 'apel', color: '#E34234', emoji: '🍎', points: 10, radius: 26 },
            { name: 'jeruk', color: '#FFA500', emoji: '🍊', points: 12, radius: 25 },
            { name: 'semangka', color: '#3B9E3B', emoji: '🍉', points: 20, radius: 28 },
            { name: 'nanas', color: '#FFD966', emoji: '🍍', points: 15, radius: 27 },
            { name: 'strawberry', color: '#FC4A6C', emoji: '🍓', points: 15, radius: 24 },
            { name: 'pisang', color: '#FFE45C', emoji: '🍌', points: 12, radius: 26 }
        ];
        
        const bombType = { name: 'bom', color: '#2c2c2c', emoji: '💣', points: -30, radius: 28 };
        
        // ----- Kelas entity -----
        class FruitEntity {
            constructor(x, y, vx, vy, typeData, isBomb = false) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.type = typeData;
                this.isBomb = isBomb;
                this.radius = typeData.radius;
                this.gravity = 0.38;
                this.life = true;
                this.sliced = false;
                // rotasi kecil
                this.rot = Math.random() * Math.PI * 2;
                this.rotSpeed = (Math.random() - 0.5) * 0.1;
            }
            
            update() {
                this.vx *= 0.998;
                this.vy += this.gravity;
                this.x += this.vx;
                this.y += this.vy;
                this.rot += this.rotSpeed;
                // batas: jika y > H + radius atau terlalu ke samping, matikan
                if (this.y - this.radius > H || this.y + this.radius < -50 || this.x + this.radius < -50 || this.x - this.radius > W + 50) {
                    this.life = false;
                    // Jika buah keluar tanpa dipotong dan bukan bom -> mengurangi combo & bisa mengurangi nyawa? hanya bom tidak mempengaruhi
                    if (!this.sliced && !this.isBomb && gameRunning) {
                        // buah luput tidak mengurangi nyawa, tapi combo reset (karena miss)
                        if (combo > 0) {
                            combo = 0;
                            updateUI();
                        }
                    }
                    // jika bom meledak tanpa terkena slice? tidak mengurangi nyawa (hanya jika kena slice)
                }
            }
            
            draw(ctx) {
                if (this.sliced) return;
                ctx.save();
                ctx.shadowBlur = 8;
                ctx.shadowColor = "rgba(0,0,0,0.4)";
                ctx.translate(this.x, this.y);
                ctx.rotate(this.rot);
                ctx.font = `${this.radius * 1.3}px "Segoe UI Emoji", "Apple Color Emoji", system-ui, sans-serif`;
                ctx.textAlign = "center";
                ctx.textBaseline = "middle";
                if (!this.isBomb) {
                    // buah
                    ctx.shadowColor = "rgba(0,0,0,0.3)";
                    ctx.fillStyle = this.type.color;
                    ctx.beginPath();
                    ctx.arc(0, 0, this.radius-2, 0, Math.PI*2);
                    ctx.fill();
                    ctx.fillStyle = "#ffffffcc";
                    ctx.beginPath();
                    ctx.arc(-5, -5, 6, 0, Math.PI*2);
                    ctx.fill();
                    ctx.fillStyle = "#00000066";
                    ctx.beginPath();
                    ctx.arc(-3, -3, 2, 0, Math.PI*2);
                    ctx.fill();
                    ctx.fillStyle = "#2c2c2c";
                    ctx.font = `${this.radius * 1.1}px "Segoe UI Emoji"`;
                    ctx.fillText(this.type.emoji, 0, 0);
                } else {
                    // bom
                    ctx.fillStyle = "#2f2a24";
                    ctx.beginPath();
                    ctx.arc(0, 0, this.radius-2, 0, Math.PI*2);
                    ctx.fill();
                    ctx.fillStyle = "#ff4d4d";
                    ctx.font = `${this.radius * 1.1}px "Segoe UI Emoji"`;
                    ctx.fillText("💣", 0, 0);
                    ctx.fillStyle = "#f5a623";
                    ctx.font = `${this.radius * 0.7}px monospace`;
                    ctx.fillText("!", 10, -12);
                }
                ctx.restore();
            }
        }
        
        // Efek potongan (percikan)
        class SlashEffect {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.life = 1.0;
                this.particles = [];
                for(let i=0;i<6;i++){
                    this.particles.push({
                        x: x, y: y, vx: (Math.random()-0.5)*7, vy: (Math.random()-0.5)*7 - 2,
                        life: 0.8
                    });
                }
            }
            update() {
                this.life -= 0.08;
                for(let p of this.particles){
                    p.x += p.vx;
                    p.y += p.vy;
                    p.vy += 0.2;
                    p.life -= 0.03;
                }
                return this.life > 0;
            }
            draw(ctx) {
                for(let p of this.particles){
                    ctx.globalAlpha = p.life * 0.8;
                    ctx.beginPath();
                    ctx.arc(p.x, p.y, 4, 0, Math.PI*2);
                    ctx.fillStyle = "#ffd966";
                    ctx.fill();
                    ctx.beginPath();
                    ctx.arc(p.x-1, p.y-1, 2, 0, Math.PI*2);
                    ctx.fillStyle = "#ffaa33";
                    ctx.fill();
                }
                ctx.globalAlpha = 1;
            }
        }
        
        // ---- Fungsi utama slicing (deteksi lingkaran & garis) ----
        function sliceAtLine(x1, y1, x2, y2) {
            if (!gameRunning) return;
            let anySlice = false;
            // iterasi dari belakang agar aman saat splice
            for (let i = fruits.length-1; i >= 0; i--) {
                const fruit = fruits[i];
                if (fruit.sliced) continue;
                // jarak titik terdekat dari segmen garis ke pusat fruit
                const closest = closestPointOnSegment(x1, y1, x2, y2, fruit.x, fruit.y);
                const dx = closest.x - fruit.x;
                const dy = closest.y - fruit.y;
                const dist = Math.hypot(dx, dy);
                if (dist < fruit.radius + 5) {
                    // kena slice
                    anySlice = true;
                    fruit.sliced = true;
                    fruit.life = false;
                    
                    if (fruit.isBomb) {
                        // BOM: kurangi nyawa, reset combo, efek ledakan
                        lives = Math.max(0, lives - 1);
                        combo = 0;
                        updateUI();
                        addExplosionEffect(fruit.x, fruit.y, true);
                        if (lives <= 0) {
                            gameRunning = false;
                            updateUI();
                        }
                        // hapus bom
                        fruits.splice(i,1);
                        continue;
                    } 
                    else {
                        // Buah: tambah skor sesuai combo
                        let addPoints = fruit.type.points;
                        // Bonus combo: setiap multiplier combo (1x = 0%, 2x = +20%, 3x +50%, 4x +100%)
                        let comboBonus = 1 + Math.min(0.8, (combo * 0.18));
                        let finalPoints = Math.floor(addPoints * comboBonus);
                        score += finalPoints;
                        combo++;
                        if(combo > maxCombo) maxCombo = combo;
                        updateUI();
                        addExplosionEffect(fruit.x, fruit.y, false);
                        // efek juice
                        fruits.splice(i,1);
                    }
                }
            }
            if(anySlice){
                // tambah efek garis slice setiap slice
                addSlashTrailEffect(x1,y1,x2,y2);
            }
        }
        
        // helper: titik terdekat pada segmen garis
        function closestPointOnSegment(ax, ay, bx, by, px, py) {
            const abx = bx - ax;
            const aby = by - ay;
            const t = ((px - ax) * abx + (py - ay) * aby) / (abx * abx + aby * aby || 1);
            const clamped = Math.max(0, Math.min(1, t));
            return { x: ax + abx * clamped, y: ay + aby * clamped };
        }
        
        // menambah efek visual
        function addExplosionEffect(x, y, isBomb) {
            for(let i=0;i<12;i++){
                explosions.push(new SlashEffect(x, y));
            }
        }
        
        function addSlashTrailEffect(x1,y1,x2,y2){
            // efek kilat
            for(let i=0;i<3;i++){
                let mx = (x1+x2)/2 + (Math.random()-0.5)*12;
                let my = (y1+y2)/2 + (Math.random()-0.5)*12;
                explosions.push(new SlashEffect(mx, my));
            }
        }
        
        // Spawn buah / bom
        function spawnRandomItem(now) {
            if (!gameRunning) return;
            // probabilitas bom 18%
            const isBomb = Math.random() < 0.18;
            let typeData;
            if(isBomb){
                typeData = bombType;
            } else {
                typeData = fruitTypes[Math.floor(Math.random() * fruitTypes.length)];
            }
            // posisi spawn: dari bawah tengah atau samping? agar melambung: x random antara 80 sampai W-80
            const startX = 50 + Math.random() * (W - 100);
            const startY = H - 40 + Math.random() * 30;
            // kecepatan arah melambung ke atas + sedikit acak horizontal
            const vx = (Math.random() - 0.5) * 2.6;
            const vy = -9 - Math.random() * 4.5;
            const fruit = new FruitEntity(startX, startY, vx, vy, typeData, isBomb);
            fruits.push(fruit);
        }
        
        // update posisi semua buah, hapus yang mati
        function updateEntities() {
            for(let i=0;i<fruits.length;i++){
                fruits[i].update();
                if(!fruits[i].life){
                    fruits.splice(i,1);
                    i--;
                }
            }
            for(let i=0;i<explosions.length;i++){
                let alive = explosions[i].update();
                if(!alive){
                    explosions.splice(i,1);
                    i--;
                }
            }
        }
        
        // ---- DRAW SEMUA ----
        function drawBackground() {
            // langit gradien
            const grad = ctx.createLinearGradient(0,0,0,H);
            grad.addColorStop(0, "#b1e1e6");
            grad.addColorStop(1, "#6fc3cf");
            ctx.fillStyle = grad;
            ctx.fillRect(0,0,W,H);
            // awan
            ctx.fillStyle = "#ffffffaa";
            ctx.shadowBlur = 0;
            ctx.beginPath();
            ctx.ellipse(150, 70, 50, 40, 0, 0, Math.PI*2);
            ctx.ellipse(200, 55, 45, 38, 0, 0, Math.PI*2);
            ctx.ellipse(100, 60, 40, 35, 0, 0, Math.PI*2);
            ctx.fill();
            ctx.beginPath();
            ctx.ellipse(700, 90, 65, 45, 0, 0, Math.PI*2);
            ctx.ellipse(760, 75, 50, 40, 0, 0, Math.PI*2);
            ctx.fill();
        }
        
        function drawSliceTrail() {
            if (mousePositions.length < 2) return;
            ctx.beginPath();
            ctx.lineWidth = 8;
            ctx.lineCap = 'round';
            ctx.lineJoin = 'round';
            ctx.shadowBlur = 12;
            ctx.shadowColor = "white";
            for (let i = 0; i < mousePositions.length - 1; i++) {
                const p1 = mousePositions[i];
                const p2 = mousePositions[i+1];
                ctx.beginPath();
                ctx.moveTo(p1.x, p1.y);
                ctx.lineTo(p2.x, p2.y);
                const gradient = ctx.createLinearGradient(p1.x, p1.y, p2.x, p2.y);
                gradient.addColorStop(0, '#fde16d');
                gradient.addColorStop(1, '#ffa047');
                ctx.strokeStyle = gradient;
                ctx.stroke();
            }
            // efek kilap
            ctx.beginPath();
            ctx.lineWidth = 3;
            for (let i = 0; i < mousePositions.length - 1; i++) {
                ctx.moveTo(mousePositions[i].x, mousePositions[i].y);
                ctx.lineTo(mousePositions[i+1].x, mousePositions[i+1].y);
                ctx.strokeStyle = "#fff9cf";
                ctx.stroke();
            }
            ctx.shadowBlur = 0;
        }
        
        function drawUItext() {
            if (!gameRunning) {
                ctx.font = "bold 44px 'Righteous', cursive";
                ctx.fillStyle = "#fff6cf";
                ctx.shadowBlur = 0;
                ctx.fillText("GAME OVER", W/2-130, H/2-50);
                ctx.font = "24px monospace";
                ctx.fillStyle = "#ffdd99";
                ctx.fillText(`Skor Akhir: ${score}  |  Combo tertinggi: ${maxCombo}x`, W/2-200, H/2+30);
                ctx.font = "20px system-ui";
                ctx.fillStyle = "#ffecb3";
                ctx.fillText("Tekan tombol 'Potong Baru' untuk memulai ulang", W/2-220, H/2+100);
            }
        }
        
        function drawAll() {
            if (!ctx) return;
            drawBackground();
            // buah & bom
            for(let f of fruits) f.draw(ctx);
            for(let ex of explosions) ex.draw(ctx);
            drawSliceTrail();
            drawUItext();
            // instruksi tambahan
            ctx.font = "12px 'Segoe UI'";
            ctx.fillStyle = "#222222aa";
            ctx.fillText("Geser / seret mouse untuk mengiris!", 20, 30);
        }
        
        // ---------- LOOP GAME ----------
        let lastSpawnFrame = 0;
        function gameLoop(now) {
            if (!gameRunning && frameRequest) {
                drawAll();
                frameRequest = requestAnimationFrame(gameLoop);
                return;
            }
            // update physics
            updateEntities();
            // spawning berdasarkan waktu real
            const currentTime = performance.now();
            if (gameRunning && (currentTime - lastSpawnTime) > spawnInterval) {
                spawnRandomItem(currentTime);
                lastSpawnTime = currentTime;
                // sesuaikan spawn interval makin tinggi skor makin padat (tapi max 220ms)
                let dynamicDelay = Math.max(280, 550 - Math.floor(score / 280) * 10);
                spawnInterval = Math.min(dynamicDelay, 500);
            }
            drawAll();
            frameRequest = requestAnimationFrame(gameLoop);
        }
        
        // ---- POINTER / MOUSE EVENT UNTUK SLICING ----
        function getCanvasCoords(e) {
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;
            let clientX, clientY;
            if (e.touches) {
                if (e.touches.length === 0) return null;
                clientX = e.touches[0].clientX;
                clientY = e.touches[0].clientY;
            } else {
                clientX = e.clientX;
                clientY = e.clientY;
            }
            let canvasX = (clientX - rect.left) * scaleX;
            let canvasY = (clientY - rect.top) * scaleY;
            canvasX = Math.min(Math.max(0, canvasX), W);
            canvasY = Math.min(Math.max(0, canvasY), H);
            return { x: canvasX, y: canvasY };
        }
        
        function handlePointerStart(e) {
            e.preventDefault();
            if (!gameRunning) return;
            isPointerDown = true;
            const coord = getCanvasCoords(e);
            if (coord) {
                mousePositions = [coord];
            }
        }
        
        function handlePointerMove(e) {
            e.preventDefault();
            if (!isPointerDown || !gameRunning) return;
            const coord = getCanvasCoords(e);
            if (!coord) return;
            if (mousePositions.length > 0) {
                const last = mousePositions[mousePositions.length-1];
                const dist = Math.hypot(last.x - coord.x, last.y - coord.y);
                if (dist > 7) {
                    // lakukan slicing antara last dan coord
                    sliceAtLine(last.x, last.y, coord.x, coord.y);
                    mousePositions.push(coord);
                    if (mousePositions.length > 12) mousePositions.shift();
                }
            } else {
                mousePositions.push(coord);
            }
        }
        
        function handlePointerEnd(e) {
            e.preventDefault();
            isPointerDown = false;
            mousePositions = [];
            // jika game over, tidak apa
        }
        
        // reset fungsi
        function resetGame() {
            gameRunning = true;
            score = 0;
            lives = 3;
            combo = 0;
            maxCombo = 0;
            fruits = [];
            explosions = [];
            mousePositions = [];
            isPointerDown = false;
            lastSpawnTime = performance.now();
            spawnInterval = 450;
            updateUI();
            // spawn awal beberapa buah agar seru
            for(let i=0;i<4;i++){
                setTimeout(() => { if(gameRunning) spawnRandomItem(performance.now()); }, i*70);
            }
        }
        
        function updateUI() {
            document.getElementById('scoreValue').innerText = Math.floor(score);
            document.getElementById('comboValue').innerText = combo + "x";
            document.getElementById('healthValue').innerText = lives;
            if (lives <= 0) {
                gameRunning = false;
                document.getElementById('healthValue').innerText = "0";
            }
        }
        
        // ----- EVENT LISTENER -----
        function attachEvents() {
            canvas.addEventListener('mousedown', handlePointerStart);
            window.addEventListener('mousemove', (e) => {
                if (isPointerDown) handlePointerMove(e);
            });
            window.addEventListener('mouseup', handlePointerEnd);
            
            canvas.addEventListener('touchstart', handlePointerStart, {passive: false});
            canvas.addEventListener('touchmove', handlePointerMove, {passive: false});
            canvas.addEventListener('touchend', handlePointerEnd);
            canvas.addEventListener('touchcancel', handlePointerEnd);
            
            // hindari context menu
            canvas.addEventListener('contextmenu', (e) => e.preventDefault());
        }
        
        // inisialisasi game
        function init() {
            attachEvents();
            resetGame();
            lastSpawnTime = performance.now();
            frameRequest = requestAnimationFrame(gameLoop);
            document.getElementById('resetGameBtn').addEventListener('click', () => {
                resetGame();
                updateUI();
            });
        }
        
        init();
    })();
</script>
</body>
</html>
