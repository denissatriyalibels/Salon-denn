<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Salon Glamour - Game Simulasi Salon</title>
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
        }

        body {
            background: linear-gradient(145deg, #f9d6e0 0%, #f8c1cc 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Poppins', 'Segoe UI', system-ui, sans-serif;
            margin: 0;
            padding: 20px;
        }

        /* Container utama */
        .game-wrapper {
            background: #ffeff4;
            border-radius: 64px;
            box-shadow: 0 25px 40px rgba(0, 0, 0, 0.2), inset 0 1px 3px rgba(255,255,255,0.8);
            padding: 20px;
        }

        .salon-container {
            max-width: 1100px;
            margin: 0 auto;
            background: #ffe6ed;
            border-radius: 48px;
            padding: 20px;
            box-shadow: inset 0 0 0 2px #fff5f8, 0 10px 20px rgba(0,0,0,0.1);
        }

        /* Header panel */
        .stats-panel {
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 12px;
            background: #ffffffcc;
            backdrop-filter: blur(8px);
            padding: 15px 25px;
            border-radius: 60px;
            margin-bottom: 24px;
            box-shadow: 0 5px 12px rgba(0,0,0,0.05);
        }

        .stat {
            background: #f8d7e2;
            padding: 8px 20px;
            border-radius: 40px;
            display: flex;
            align-items: center;
            gap: 12px;
            font-weight: 600;
            font-size: 1.2rem;
            color: #a13e5c;
            box-shadow: inset 0 1px 2px #fff8, 0 2px 5px rgba(0,0,0,0.05);
        }

        .stat span:first-child {
            font-size: 1.7rem;
        }

        .stat-value {
            font-size: 1.5rem;
            font-weight: 800;
            color: #c95a7a;
        }

        /* area salon (kursi dan ruang) */
        .salon-area {
            display: flex;
            flex-wrap: wrap;
            gap: 24px;
            justify-content: center;
        }

        /* kursi / meja rias */
        .chairs-grid {
            flex: 2;
            min-width: 280px;
            background: #fcecf2;
            border-radius: 48px;
            padding: 20px;
            box-shadow: inset 0 0 0 2px #fff, 0 8px 18px rgba(0,0,0,0.05);
        }

        .chairs-title {
            text-align: center;
            font-weight: bold;
            color: #b14b6e;
            margin-bottom: 15px;
            font-size: 1.3rem;
            background: #ffeef4;
            display: inline-block;
            width: 100%;
            border-radius: 40px;
            padding: 6px;
        }

        .chair-list {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .chair-card {
            background: white;
            border-radius: 48px;
            padding: 12px 20px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 12px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.05);
            transition: all 0.2s;
            border: 1px solid #ffcfdf;
        }

        .chair-info {
            display: flex;
            align-items: center;
            gap: 12px;
            font-weight: bold;
        }

        .chair-icon {
            font-size: 2rem;
        }

        .customer-name {
            background: #f8e2eb;
            padding: 4px 12px;
            border-radius: 40px;
            font-size: 0.8rem;
            font-weight: 600;
            color: #b34166;
        }

        .service-buttons {
            display: flex;
            gap: 8px;
        }

        .btn-service {
            background: #ffc0d0;
            border: none;
            font-size: 1.2rem;
            padding: 6px 12px;
            border-radius: 40px;
            font-weight: bold;
            cursor: pointer;
            transition: 0.1s;
            color: #962d51;
            font-family: monospace;
            font-weight: 800;
        }

        .btn-service:active {
            transform: scale(0.96);
        }

        .empty-chair {
            color: #bc8f9c;
            font-style: italic;
            background: #fff0f5;
            padding: 8px 15px;
            border-radius: 40px;
            width: 100%;
            text-align: center;
        }

        /* antrian pelanggan */
        .queue-area {
            flex: 1.2;
            min-width: 240px;
            background: #fcecf2;
            border-radius: 48px;
            padding: 20px;
            box-shadow: inset 0 0 0 2px #fff, 0 8px 18px rgba(0,0,0,0.05);
        }

        .queue-header {
            text-align: center;
            font-weight: bold;
            font-size: 1.3rem;
            color: #b14b6e;
        }

        .queue-list {
            margin-top: 15px;
            max-height: 350px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .queue-item {
            background: white;
            padding: 10px 15px;
            border-radius: 60px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-left: 8px solid #f3a0b9;
        }

        .queue-name {
            font-weight: bold;
        }

        .queue-want {
            font-size: 0.7rem;
            background: #ffdae6;
            padding: 2px 8px;
            border-radius: 20px;
        }

        .btn-call {
            background: #f5a3bc;
            border: none;
            border-radius: 30px;
            padding: 4px 14px;
            font-weight: bold;
            color: white;
            cursor: pointer;
            font-size: 0.75rem;
        }

        /* toko / beli peralatan */
        .shop-area {
            margin-top: 25px;
            background: #fff4f8;
            border-radius: 48px;
            padding: 16px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 12px;
        }

        .shop-item {
            display: flex;
            align-items: center;
            gap: 12px;
            background: white;
            padding: 5px 20px;
            border-radius: 60px;
        }

        .btn-buy {
            background: #b0d9b1;
            border: none;
            border-radius: 40px;
            padding: 6px 15px;
            font-weight: bold;
            cursor: pointer;
        }

        .reset-btn {
            background: #e7b2c4;
            border: none;
            border-radius: 60px;
            padding: 8px 22px;
            font-weight: bold;
            font-size: 1rem;
            cursor: pointer;
            color: #4e2c38;
        }

        .message-area {
            margin-top: 15px;
            text-align: center;
            background: #fff0e7;
            border-radius: 60px;
            padding: 8px;
            font-size: 0.8rem;
            color: #ae6c81;
        }

        @media (max-width: 700px) {
            .stat { font-size: 0.9rem; padding: 4px 14px; }
            .btn-service { font-size: 0.9rem; padding: 4px 10px; }
        }
    </style>
</head>
<body>
<div class="game-wrapper">
<div class="salon-container">
    <!-- Panel Statistik -->
    <div class="stats-panel">
        <div class="stat"><span>💰</span> Uang: <span class="stat-value" id="moneyAmount">500</span></div>
        <div class="stat"><span>⭐</span> Reputasi: <span class="stat-value" id="reputationAmount">50</span></div>
        <div class="stat"><span>💇</span> Pelanggan puas: <span class="stat-value" id="servedCount">0</span></div>
        <button class="reset-btn" id="resetGameBtn">🔄 Buka Salon Baru</button>
    </div>

    <div class="salon-area">
        <!-- KIRI: Kursi Layanan -->
        <div class="chairs-grid">
            <div class="chairs-title">💺 Kursi Rias & Layanan</div>
            <div class="chair-list" id="chairList"></div>
        </div>

        <!-- KANAN: Antrian Pelanggan -->
        <div class="queue-area">
            <div class="queue-header">🚪 Antrian Pelanggan</div>
            <div class="queue-list" id="queueList"></div>
            <div style="margin-top: 12px; text-align:center; font-size:0.7rem;">⬅️ Klik "Panggil" untuk menempatkan ke kursi kosong</div>
        </div>
    </div>

    <!-- Toko perlengkapan -->
    <div class="shop-area">
        <div class="shop-item">✨ <strong>Gunting Super</strong> (cepat +10 reputasi) <button class="btn-buy" id="buyScissors">Beli 250💰</button></div>
        <div class="shop-item">💄 <strong>Kursi Mewah</strong> (+1 kursi) <button class="btn-buy" id="buyChair">Beli 400💰</button></div>
        <div class="shop-item">🌟 <strong>Iklan Salon</strong> (+15 reputasi) <button class="btn-buy" id="buyAd">Beli 180💰</button></div>
    </div>
    <div class="message-area" id="gameMessage">✨ Selamat datang di Salon Glamour! Panggil pelanggan & layani mereka ✨</div>
</div>
</div>

<script>
    // ======================== DATA GAME SALON ========================
    // Kursi: setiap kursi punya { id, customer: null atau object customer, serviceProgress, serviceMax, serviceName }
    // Untuk sederhananya, layanan langsung selesai dalam 1 klik? tapi lebih asyik dengan progres? 
    // Biar seru tapi tidak terlalu kompleks, kita buat tiap layanan butuh beberapa klik sesuai "lama" (tingkat kesulitan)
    // dan ada efek reputasi & uang.
    
    let chairs = [];
    let nextChairId = 1;
    
    // Antrian pelanggan (array of customer objects)
    let queue = [];
    
    // Statistik
    let money = 500;
    let reputation = 50;      // makin tinggi, pelanggan datang lebih cepat / hadiah lebih
    let totalServed = 0;
    
    // Upgrade efek
    let speedMultiplier = 1;   // dari gunting super (mengurangi jumlah langkah layanan)
    let baseMaxProgress = 3;    // default butuh 3 klik
    
    // Helper: generate nama pelanggan & permintaan
    const firstNames = ["Mila", "Sari", "Dewi", "Luna", "Cinta", "Rara", "Tia", "Vina", "Naya", "Bella"];
    const servicesList = ["Potong Rambut", "Creambath", "Cat Rambut", "Hair Spa", "Smoothing"];
    const serviceIcons = ["✂️", "🧴", "🎨", "💆", "🌀"];
    
    function getRandomService() {
        let idx = Math.floor(Math.random() * servicesList.length);
        return { name: servicesList[idx], icon: serviceIcons[idx], baseReward: 40 + idx*5, repGain: 5 + idx*2 };
    }
    
    function generateCustomer() {
        let name = firstNames[Math.floor(Math.random() * firstNames.length)] + Math.floor(Math.random() * 90);
        let service = getRandomService();
        // progress yang diperlukan = baseMaxProgress / speedMultiplier (min 1)
        let neededClicks = Math.max(1, Math.floor(baseMaxProgress / speedMultiplier));
        return {
            id: Date.now() + Math.random(),
            name: name,
            serviceName: service.name,
            serviceIcon: service.icon,
            baseReward: service.baseReward,
            repGain: service.repGain,
            requiredClicks: neededClicks,
            currentClick: 0,
            status: "waiting" // waiting, serving
        };
    }
    
    // Fungsi untuk menambah antrian secara periodik
    let spawnInterval;
    
    function spawnCustomer() {
        if (queue.length < 8) { // batas antrian
            let newCust = generateCustomer();
            queue.push(newCust);
            renderQueue();
            showMessage("✨ " + newCust.name + " datang, ingin " + newCust.serviceName + "!", "#b14b6e");
        } else {
            // antrian penuh, sedikit kurangi reputasi jika terlalu lama?
            if (Math.random() < 0.3) {
                reputation = Math.max(0, reputation - 2);
                updateStatsUI();
                showMessage("⚠️ Antrian penuh! Beberapa pelanggan pergi, reputasi -2", "#c95a7a");
            }
        }
        renderQueue();
    }
    
    // Mulai interval spawn (semakin tinggi reputasi, makin cepat spawn)
    function restartSpawnInterval() {
        if (spawnInterval) clearInterval(spawnInterval);
        let delay = Math.max(2500, 6000 - Math.floor(reputation / 3) * 30);
        delay = Math.min(delay, 5000);
        spawnInterval = setInterval(() => {
            if (document.hidden) return;
            spawnCustomer();
        }, delay);
    }
    
    // Memanggil pelanggan dari antrian ke kursi kosong
    function callCustomerToChair(customerIndex, chairId) {
        if (customerIndex < 0 || customerIndex >= queue.length) return false;
        let chair = chairs.find(c => c.id === chairId);
        if (!chair || chair.customer !== null) return false;
        let customer = queue[customerIndex];
        if (!customer || customer.status !== "waiting") return false;
        
        // pindahkan ke kursi
        chair.customer = { ...customer, status: "serving", currentClick: 0 };
        // hapus dari antrian
        queue.splice(customerIndex, 1);
        renderAll();
        showMessage(`📞 ${chair.customer.name} duduk di kursi ${chair.id}. Mulai layanan!`, "#6f9e6f");
        return true;
    }
    
    // Melayani pelanggan di kursi (klik tombol layanan)
    function serveProgress(chairId) {
        let chair = chairs.find(c => c.id === chairId);
        if (!chair || !chair.customer) {
            showMessage("Kursi kosong, tidak ada yang dilayani!", "#c95a7a");
            return false;
        }
        let cust = chair.customer;
        if (cust.currentClick + 1 >= cust.requiredClicks) {
            // Selesai layanan!
            // Hitung pendapatan + reputasi
            let earn = Math.floor(cust.baseReward + (reputation/30) * 5);
            let repUp = cust.repGain + Math.floor(Math.random() * 4);
            money += earn;
            reputation += repUp;
            totalServed++;
            updateStatsUI();
            showMessage(`✅ Pelayanan ${cust.name} selesai! +${earn}💰, +${repUp}⭐`, "#6da56d");
            // kosongkan kursi
            chair.customer = null;
            renderAll();
            // jika reputasi berubah, update interval spawn
            restartSpawnInterval();
            return true;
        } else {
            // tambah progres
            cust.currentClick++;
            showMessage(`✂️ Melayani ${cust.name} (${cust.currentClick}/${cust.requiredClicks})`, "#b3809b");
            renderChairs();
            return true;
        }
    }
    
    // Beli item toko
    function buyScissors() {
        if (money >= 250) {
            money -= 250;
            speedMultiplier += 0.5;
            reputation += 5;
            updateStatsUI();
            showMessage("✂️ Gunting super diperoleh! Kecepatan layanan +50% (lebih sedikit klik)", "#a56b7e");
            // update required clicks untuk pelanggan yang sedang dilayani? biarkan existing, tapi customer baru terpengaruh
            // kita biarkan untuk customer baru nanti.
            restartSpawnInterval();
            renderAll(); // refresh progres di UI (tapi pelanggan existing progress tidak berubah, tapi yg baru lebih cepat)
        } else {
            showMessage("💰 Uang tidak cukup! Butuh 250💰", "#dd7766");
        }
    }
    
    function buyChair() {
        if (money >= 400) {
            money -= 400;
            addNewChair();
            updateStatsUI();
            showMessage("💺 Kursi baru ditambahkan! Kapasitas salon bertambah.", "#8faa7a");
            renderAll();
        } else {
            showMessage("💰 Uang tidak cukup untuk kursi mewah (400💰)", "#dd7766");
        }
    }
    
    function addNewChair() {
        let newId = nextChairId++;
        chairs.push({
            id: newId,
            customer: null,
        });
    }
    
    function buyAd() {
        if (money >= 180) {
            money -= 180;
            reputation += 15;
            updateStatsUI();
            showMessage("📢 Iklan salon dipasang! Reputasi +15⭐", "#f3b0c2");
            restartSpawnInterval();
            renderAll();
        } else {
            showMessage("💰 Uang kurang untuk iklan (180💰)", "#dd7766");
        }
    }
    
    // Reset Game
    function resetGame() {
        money = 500;
        reputation = 50;
        totalServed = 0;
        speedMultiplier = 1;
        baseMaxProgress = 3;
        
        // reset chairs
        chairs = [];
        for (let i = 1; i <= 3; i++) {
            chairs.push({ id: i, customer: null });
        }
        nextChairId = 4;
        queue = [];
        // tambah beberapa pelanggan awal agar tidak sepi
        for (let i = 0; i < 2; i++) {
            queue.push(generateCustomer());
        }
        if (spawnInterval) clearInterval(spawnInterval);
        restartSpawnInterval();
        updateStatsUI();
        renderAll();
        showMessage("🔄 Salon direset! Mulai lagi dengan 3 kursi dan 500💰", "#ad7a8f");
    }
    
    // ========== UI RENDER ==========
    function renderChairs() {
        const chairContainer = document.getElementById("chairList");
        if (!chairContainer) return;
        chairContainer.innerHTML = "";
        chairs.forEach(chair => {
            const chairDiv = document.createElement("div");
            chairDiv.className = "chair-card";
            const isOccupied = chair.customer !== null;
            const cust = chair.customer;
            let leftHtml = `
                <div class="chair-info">
                    <div class="chair-icon">💺</div>
                    <div><strong>Kursi #${chair.id}</strong><br>
            `;
            if (isOccupied) {
                let progressPercent = (cust.currentClick / cust.requiredClicks) * 100;
                leftHtml += `<span class="customer-name">${cust.name} (${cust.serviceIcon} ${cust.serviceName})</span>
                            <div style="font-size:0.7rem; background:#ffe2ec; border-radius:20px; width:100px; margin-top:5px;">
                                <div style="width:${progressPercent}%; background:#c78aa3; height:5px; border-radius:20px;"></div>
                            </div>
                            <div>${cust.currentClick}/${cust.requiredClicks} klik</div>`;
            } else {
                leftHtml += `<span class="empty-chair">✨ Kursi kosong ✨</span>`;
            }
            leftHtml += `</div></div>`;
            
            const rightDiv = document.createElement("div");
            rightDiv.className = "service-buttons";
            if (isOccupied) {
                const serviceBtn = document.createElement("button");
                serviceBtn.className = "btn-service";
                serviceBtn.innerHTML = `✂️ Layani (${cust.currentClick}/${cust.requiredClicks})`;
                serviceBtn.addEventListener("click", (e) => {
                    e.stopPropagation();
                    serveProgress(chair.id);
                });
                rightDiv.appendChild(serviceBtn);
            } else {
                // tombol panggil dari antrian? lebih praktis: tombol panggil dari kursi bisa memanggil antrian pertama
                const callBtn = document.createElement("button");
                callBtn.className = "btn-service";
                callBtn.style.background = "#b8d9aa";
                callBtn.innerHTML = "📞 Panggil Antrian";
                callBtn.addEventListener("click", (e) => {
                    e.stopPropagation();
                    if (queue.length > 0) {
                        callCustomerToChair(0, chair.id);
                    } else {
                        showMessage("Tidak ada pelanggan dalam antrian. Tunggu sebentar..", "#b56783");
                    }
                });
                rightDiv.appendChild(callBtn);
            }
            chairDiv.appendChild(leftHtml);
            chairDiv.appendChild(rightDiv);
            chairContainer.appendChild(chairDiv);
        });
    }
    
    function renderQueue() {
        const queueContainer = document.getElementById("queueList");
        if (!queueContainer) return;
        queueContainer.innerHTML = "";
        if (queue.length === 0) {
            queueContainer.innerHTML = "<div style='text-align:center; padding:20px;'>🚫 Antrian kosong<br>Pelanggan akan segera datang</div>";
            return;
        }
        queue.forEach((cust, idx) => {
            const qItem = document.createElement("div");
            qItem.className = "queue-item";
            qItem.innerHTML = `
                <div>
                    <div class="queue-name">${cust.name}</div>
                    <div class="queue-want">${cust.serviceIcon} ${cust.serviceName}</div>
                </div>
                <button class="btn-call" data-idx="${idx}">Panggil</button>
            `;
            const btn = qItem.querySelector(".btn-call");
            btn.addEventListener("click", (e) => {
                e.stopPropagation();
                // cari kursi kosong pertama
                const emptyChair = chairs.find(c => c.customer === null);
                if (emptyChair) {
                    callCustomerToChair(idx, emptyChair.id);
                } else {
                    showMessage("🚫 Semua kursi terisi, selesaikan layanan terlebih dahulu!", "#dd7766");
                }
            });
            queueContainer.appendChild(qItem);
        });
    }
    
    function updateStatsUI() {
        document.getElementById("moneyAmount").innerText = Math.floor(money);
        document.getElementById("reputationAmount").innerText = Math.floor(reputation);
        document.getElementById("servedCount").innerText = totalServed;
        // batasi reputasi tidak negatif & maks
        if (reputation < 0) reputation = 0;
        if (reputation > 300) reputation = 300;
    }
    
    function showMessage(msg, colorHint = "#b14b6e") {
        const msgDiv = document.getElementById("gameMessage");
        if (msgDiv) {
            msgDiv.innerHTML = `💬 ${msg}`;
            msgDiv.style.transition = "0.2s";
            msgDiv.style.backgroundColor = "#fff4ed";
            setTimeout(() => {
                if (msgDiv.innerHTML === `💬 ${msg}`) {
                    // biarkan agak lama, tidak perlu revert otomatis
                }
            }, 2000);
        }
    }
    
    function renderAll() {
        renderChairs();
        renderQueue();
        updateStatsUI();
    }
    
    // inisialisasi awal
    function init() {
        // buat 3 kursi
        chairs = [];
        for (let i = 1; i <= 3; i++) {
            chairs.push({ id: i, customer: null });
        }
        nextChairId = 4;
        queue = [];
        // spawn 2 pelanggan awal
        for (let i = 0; i < 2; i++) queue.push(generateCustomer());
        money = 500;
        reputation = 50;
        totalServed = 0;
        speedMultiplier = 1;
        baseMaxProgress = 3;
        if (spawnInterval) clearInterval(spawnInterval);
        restartSpawnInterval();
        updateStatsUI();
        renderAll();
        showMessage("💖 Selamat datang di Salon Glamour! Panggil pelanggan dari antrian & layani!", "#b85c7c");
        
        // event listener tombol toko
        document.getElementById("buyScissors").onclick = () => buyScissors();
        document.getElementById("buyChair").onclick = () => buyChair();
        document.getElementById("buyAd").onclick = () => buyAd();
        document.getElementById("resetGameBtn").onclick = () => resetGame();
    }
    
    init();
</script>
</body>
</html>
