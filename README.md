<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gacoan Thawing Sync</title>
    
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com; style-src 'self' 'unsafe-inline'; media-src *; connect-src 'self' https://*.firebaseio.com wss://*.firebaseio.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.gstatic.com https://*.firebaseio.com; font-src 'self' data:;">
    
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    
    <style>
        :root {
            --color-white: #ffffff;
            --color-light-bg: #f7f9ff;
            --color-primary-blue: #4A90E2;
            --color-accent-pink: #FF69B4;
            --color-text-dark: #333333;
            --color-alert: #DC3545;
        }
        
        body {
            font-family: 'Poppins', sans-serif;
            background-color: var(--color-light-bg);
            margin: 0; padding: 20px 10px;
            display: flex; justify-content: center;
            transition: background-color 0.3s;
        }

        /* OVERLAY PEMILIHAN CABANG */
        #branch-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(74, 144, 226, 0.98);
            display: flex; justify-content: center; align-items: center;
            z-index: 9999; color: white;
        }
        .branch-box {
            background: white; padding: 30px; border-radius: 16px;
            color: #333; width: 90%; max-width: 400px; text-align: center;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
        }
        .branch-box select {
            width: 100%; padding: 12px; margin: 20px 0;
            border-radius: 8px; border: 1px solid #ddd; font-size: 16px;
        }
        .branch-box button {
            width: 100%; padding: 12px; background: #4A90E2;
            color: white; border: none; border-radius: 8px; cursor: pointer; font-weight: 600;
        }

        /* ANIMASI ALARM */
        .flash-alarm-red {
            animation: pulse-red 0.5s infinite alternate;
        }
        @keyframes pulse-red {
            from { background-color: #ff4d6d; }
            to { background-color: #ffffff; }
        }

        .main-container {
            background-color: var(--color-white);
            padding: 30px; border-radius: 16px;
            box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
            text-align: center; width: 100%; max-width: 850px;
        }

        .timer-list {
            display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); 
            gap: 20px; margin-top: 20px;
        }

        .timer-card {
            border: 1px solid #e0e0e0; padding: 20px; border-radius: 12px;
            background-color: var(--color-white); transition: all 0.3s;
        }

        .timer-card.running-mode { border-left: 5px solid var(--color-primary-blue); background-color: #f0f7ff; }
        .timer-card.alert { border: 2px solid var(--color-alert); background-color: #ffeff3; }

        .countdown-display { font-size: 2.5em; font-weight: 700; color: var(--color-primary-blue); margin: 10px 0; }
        .alert .countdown-display { color: var(--color-alert); }

        .btn { padding: 10px 15px; border: none; border-radius: 8px; cursor: pointer; font-weight: 600; }
        .start-btn { background: var(--color-primary-blue); color: white; }
        .reset-btn { background: #6c757d; color: white; }
        .stop-alarm-btn { background: var(--color-alert) !important; color: white; animation: pulse-btn 0.5s infinite alternate; }
        
        @keyframes pulse-btn { from { transform: scale(1); } to { transform: scale(1.05); } }
    </style>
</head>
<body>

    <div id="branch-overlay">
        <div class="branch-box">
            <h2 style="color: #4A90E2">Gacoan Sync</h2>
            <p>Pilih Cabang untuk Memulai:</p>
            <select id="branch-select">
                <option value="">-- Pilih Lokasi --</option>
                <option value="mondoroko">Malang Mondoroko</option>
                <option value="batu">Gacoan Batu</option>
                <option value="dinoyo">Gacoan Dinoyo</option>
            </select>
            <button onclick="saveBranch()">MASUK KE SISTEM</button>
        </div>
    </div>

    <div class="main-container">
        <h1 id="branch-title">Gacoan Timer Thawing</h1>
        <p id="sub-title">Memuat data...</p>
        <div class="timer-list" id="timer-list-container"></div>
    </div>

    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyBtUlghTw806GuGuwOXGNgoqN6Rkcg0IMM",
            authDomain: "thawing-ec583.firebaseapp.com",
            databaseURL: "https://thawing-ec583-default-rtdb.asia-southeast1.firebasedatabase.app",
            projectId: "thawing-ec583",
            storageBucket: "thawing-ec583.firebasestorage.app", 
            messagingSenderId: "1043079332713",
            appId: "1:1043079332713:web:6d289ad2b7c13a222bb3f8"
        };

        firebase.initializeApp(firebaseConfig);

        const THAWING_ITEMS = [
            { id: 1, name: "ADONAN", time: 40 },
            { id: 2, name: "ACIN", time: 120 },
            { id: 3, name: "MIE", time: 120 },
            { id: 4, name: "PENTOL", time: 120 },
            { id: 5, name: "SIOMAY", time: 120 },
            { id: 6, name: "KRUPUK", time: 120 }
        ];

        let currentBranch = localStorage.getItem('selectedBranch');
        let dbRef = null;
        let audioCtx = null;
        let alarmInterval = null;

        if (currentBranch) {
            initApp(currentBranch);
        }

        function saveBranch() {
            const b = document.getElementById('branch-select').value;
            if (!b) return alert("Pilih cabang!");
            localStorage.setItem('selectedBranch', b);
            location.reload();
        }

        function initApp(branch) {
            document.getElementById('branch-overlay').style.display = 'none';
            document.getElementById('branch-title').textContent = `Gacoan - ${branch.toUpperCase()}`;
            document.getElementById('sub-title').textContent = `Sinkronisasi Aktif: Folder cabang/${branch}`;
            
            dbRef = firebase.database().ref(`cabang/${branch}/thawingTimers`);
            listenToFirebase();
        }

        function playBeep() {
            if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.type = "sine"; osc.frequency.value = 1000; gain.gain.value = 0.1;
            osc.connect(gain); gain.connect(audioCtx.destination);
            osc.start(); osc.stop(audioCtx.currentTime + 0.2);
        }

        function listenToFirebase() {
            dbRef.on('value', (snapshot) => {
                const data = snapshot.val() || {};
                renderUI(data);

                let anyAlarm = false;
                for (let id in data) {
                    if (data[id].timeLeft <= 0 && data[id].status === "RUNNING") anyAlarm = true;
                }

                if (anyAlarm) {
                    document.body.classList.add('flash-alarm-red');
                    if (!alarmInterval) alarmInterval = setInterval(playBeep, 1000);
                } else {
                    document.body.classList.remove('flash-alarm-red');
                    clearInterval(alarmInterval); alarmInterval = null;
                }
            });
        }

        function renderUI(firebaseData) {
            const container = document.getElementById('timer-list-container');
            container.innerHTML = '';

            THAWING_ITEMS.forEach(item => {
                const state = firebaseData[item.id] || { timeLeft: item.time * 60, status: "IDLE" };
                const isExpired = state.timeLeft <= 0 && state.status === "RUNNING";
                
                const card = document.createElement('div');
                card.className = `timer-card ${state.status === 'RUNNING' ? 'running-mode' : ''} ${isExpired ? 'alert' : ''}`;
                card.innerHTML = `
                    <h3>${item.name}</h3>
                    <div class="countdown-display">${Math.floor(state.timeLeft / 60)}m ${state.timeLeft % 60}s</div>
                    <button class="btn start-btn" onclick="startTimer(${item.id}, ${item.time})" 
                        style="display: ${state.status === 'RUNNING' ? 'none' : 'inline-block'}">START</button>
                    <button class="btn reset-btn ${isExpired ? 'stop-alarm-btn' : ''}" onclick="resetTimer(${item.id})">
                        ${isExpired ? 'STOP ALARM' : 'RESET'}
                    </button>
                `;
                container.appendChild(card);
            });
        }

        function startTimer(id, mins) {
            if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const endTime = Date.now() + (mins * 60 * 1000);
            
            dbRef.child(id).set({ timeLeft: mins * 60, status: "RUNNING" });

            const loop = setInterval(() => {
                dbRef.child(id).transaction(current => {
                    if (current && current.timeLeft > 0 && current.status === "RUNNING") {
                        return { ...current, timeLeft: current.timeLeft - 1 };
                    }
                    clearInterval(loop); return current;
                });
            }, 1000);
        }

        function resetTimer(id) {
            dbRef.child(id).remove();
        }

        document.addEventListener('click', () => {
            if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
        }, { once: true });
    </script>
</body>
</html>
