<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gacoan Thawing Timer - Sync</title>
    
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com; style-src 'self' 'unsafe-inline'; media-src *; connect-src 'self' https://*.firebaseio.com wss://*.firebaseio.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.gstatic.com https://*.firebaseio.com https://www.googleapis.com; font-src 'self' data:;">
    
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
    
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-auth.js"></script>
    
    <style>
        /* ==================== 
           CSS: STYLING 
           ==================== */
        :root {
            --color-white: #ffffff;
            --color-light-bg: #f7f9ff;
            --color-primary-blue: #4A90E2;
            --color-accent-pink: #FF69B4;
            --color-text-dark: #333333;
            --color-warning: #FFC0CB;
            --color-alert: #DC3545;
        }
        
        body {
            font-family: 'Poppins', sans-serif;
            background-color: var(--color-light-bg);
            min-height: 100vh;
            margin: 0; padding: 20px 10px;
            display: flex; justify-content: center; align-items: flex-start;
            transition: background-color 0.2s; color: var(--color-text-dark);
        }

        .main-container {
            background-color: var(--color-white);
            padding: 30px; border-radius: 16px;
            box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
            text-align: center; width: 100%; max-width: 850px;
        }

        /* --- LOGIN --- */
        #auth-container { max-width: 350px; margin: 0 auto; text-align: left; }
        .auth-group { margin-bottom: 15px; }
        .auth-group label { display: block; font-weight: 600; margin-bottom: 5px; }
        .auth-group input, #branch-select {
            width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box;
        }
        #login-btn {
            width: 100%; padding: 12px; border: none; border-radius: 6px;
            background: var(--color-primary-blue); color: white; font-weight: 700; cursor: pointer;
        }

        /* --- TIMER --- */
        .header-controls { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
        .logout-btn-small { padding: 8px 15px; background: #f44336; color: white; border: none; border-radius: 6px; cursor: pointer; }
        
        .timer-list { display: grid; grid-template-columns: repeat(2, 1fr); gap: 20px; }
        .timer-card { border: 1px solid #e0e0e0; padding: 20px; border-radius: 12px; transition: all 0.3s; }
        .timer-card.running-mode { border-left: 5px solid var(--color-primary-blue); background: #f0f7ff; }
        .timer-card h2 { color: var(--color-accent-pink); margin: 0; font-size: 1.2em; }
        
        .countdown-display { font-size: 2.5em; font-weight: 700; color: var(--color-primary-blue); margin: 10px 0; }
        .end-time-display { font-size: 0.85em; color: #999; }
        .alarm-message { color: var(--color-alert); font-weight: 600; font-size: 0.9em; margin-top: 5px; }

        .timer-controls { display: flex; gap: 8px; margin-top: 15px; justify-content: center; align-items: center; }
        .timer-controls input { width: 60px; text-align: center; padding: 8px; border-radius: 6px; border: 1px solid #ddd; }
        .start-btn { background: var(--color-primary-blue); color: white; border: none; padding: 10px 15px; border-radius: 6px; cursor: pointer; flex-grow: 1; }
        .reset-btn { background: #6c757d; color: white; border: none; padding: 10px 15px; border-radius: 6px; cursor: pointer; }
        
        .stop-alarm-btn { background: var(--color-alert) !important; animation: pulse-stop 0.5s infinite alternate; }
        .flash-alarm-red { background-color: #ff4d6d !important; }

        @keyframes pulse-stop { from { opacity: 1; } to { opacity: 0.7; } }
        @media (max-width: 600px) { .timer-list { grid-template-columns: 1fr; } }
    </style>
</head>
<body>
    <div class="main-container">
        <div id="auth-container">
            <h2 style="text-align: center; color: var(--color-primary-blue);">Login Thawing Timer</h2>
            <div class="auth-group">
                <label>Email</label>
                <input type="email" id="email" placeholder="Email Cabang">
            </div>
            <div class="auth-group">
                <label>Password</label>
                <input type="password" id="password" placeholder="Password">
            </div>
            <div class="auth-group">
                <label>Pilih Cabang</label>
                <select id="branch-select">
                    <option value="Cabang_1">Cabang 1 (MLGJAK)</option>
                    <option value="Cabang_2">Cabang 2 (MLGMON)</option>
                </select>
            </div>
            <button id="login-btn">MASUK</button>
        </div>

        <div id="timer-content" style="display: none;">
            <div class="header-controls">
                <div style="text-align: left;">
                    <h1 id="branch-title" style="margin:0; font-size: 1.5em;">Gacoan Timer</h1>
                    <p style="margin:0;">V.1.4 (Multi-Branch & Auto-Voice)</p>
                </div>
                <button id="logout-btn" class="logout-btn-small">LOGOUT</button>
            </div>
            
            <div class="timer-list" id="timer-list"></div>
        </div>
    </div>

    <script>
        // --- KONFIGURASI FIREBASE ---
        const firebaseConfig = {
            apiKey: "AIzaSyBtUlghTw806GuGuwOXGNgoqN6Rkcg0IMM",
            authDomain: "thawing-ec583.firebaseapp.com",
            databaseURL: "https://thawing-ec583-default-rtdb.asia-southeast1.firebasedatabase.app",
            projectId: "thawing-ec583",
            storageBucket: "thawing-ec583.firebasestorage.app",
            messagingSenderId: "1043079332713",
            appId: "1:1043079332713:web:6d289ad2b7c13a222bb3f8"
        };

        if (!firebase.apps.length) firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        let currentDbRef = null;
        let activeIntervals = {};
        let isSpeaking = false;
        let speechQueue = [];

        // --- AUTH LOGIC ---
        document.getElementById('login-btn').addEventListener('click', () => {
            const email = document.getElementById('email').value;
            const password = document.getElementById('password').value;
            auth.signInWithEmailAndPassword(email, password).catch(err => alert("Gagal: " + err.message));
        });

        document.getElementById('logout-btn').addEventListener('click', () => auth.signOut());

        auth.onAuthStateChanged((user) => {
            if (user) {
                const branch = document.getElementById('branch-select').value;
                currentDbRef = firebase.database().ref(`thawingTimers/${branch}`);
                document.getElementById('auth-container').style.display = 'none';
                document.getElementById('timer-content').style.display = 'block';
                document.getElementById('branch-title').innerText = `Gacoan - ${branch.replace('_', ' ')}`;
                initApp();
            } else {
                document.getElementById('auth-container').style.display = 'block';
                document.getElementById('timer-content').style.display = 'none';
                cleanup();
            }
        });

        // --- VOICE LOGIC ---
        function processQueue() {
            if (isSpeaking || speechQueue.length === 0) return;
            isSpeaking = true;
            const msg = speechQueue.shift();
            window.speechSynthesis.cancel();
            const ut = new SpeechSynthesisUtterance(msg);
            ut.lang = 'id-ID';
            ut.onend = () => { isSpeaking = false; processQueue(); };
            ut.onerror = () => { isSpeaking = false; processQueue(); };
            window.speechSynthesis.speak(ut);
        }

        function speak(msg) {
            if (!speechQueue.includes(msg)) speechQueue.push(msg);
            processQueue();
        }

        // --- CORE APP ---
        function initApp() {
            const ITEMS = [
                { id: 1, name: "ADONAN", time: 40 },
                { id: 2, name: "ACIN", time: 120 },
                { id: 3, name: "MIE", time: 120 },
                { id: 4, name: "PENTOL", time: 120 },
                { id: 5, name: "SURAI NAGA", time: 120 },
                { id: 6, name: "KRUPUK MIE", time: 120 },
                { id: 7, name: "KULIT PANGSIT", time: 120 }
            ];

            const list = document.getElementById('timer-list');
            list.innerHTML = '';

            ITEMS.forEach(item => {
                const card = document.createElement('div');
                card.className = 'timer-card';
                card.id = `card-${item.id}`;
                card.innerHTML = `
                    <h2>${item.name}</h2>
                    <div id="display-${item.id}" class="countdown-display">00:00:00</div>
                    <div id="end-time-${item.id}" class="end-time-display">Ready</div>
                    <div id="msg-${item.id}" class="alarm-message" style="display:none"></div>
                    <div class="timer-controls">
                        <input type="number" id="input-${item.id}" value="${item.time}">
                        <button class="start-btn" onclick="triggerStart(${item.id})">START</button>
                        <button class="reset-btn" id="res-${item.id}" style="display:none" onclick="triggerReset(${item.id})">RESET</button>
                    </div>`;
                list.appendChild(card);
            });

            currentDbRef.on('value', snap => {
                const data = snap.val() || {};
                ITEMS.forEach(item => {
                    if (data[item.id]) {
                        runTick(item.id, data[item.id].endTime, item.name);
                    } else {
                        stopTick(item.id, item.time);
                    }
                });
            });
        }

        function runTick(id, end, name) {
            clearTimeout(activeIntervals[id]);
            const card = document.getElementById(`card-${id}`);
            const disp = document.getElementById(`display-${id}`);
            const msg = document.getElementById(`msg-${id}`);
            const btnRes = document.getElementById(`res-${id}`);
            const input = document.getElementById(`input-${id}`);
            const endTimeDisp = document.getElementById(`end-time-${id}`);

            const duration = Math.floor((end - Date.now()) / 1000);
            const endStr = new Date(end).toLocaleTimeString('id-ID', {hour:'2-digit', minute:'2-digit'});

            card.classList.add('running-mode');
            input.readOnly = true;
            card.querySelector('.start-btn').style.display = 'none';
            btnRes.style.display = 'block';
            endTimeDisp.textContent = `Selesai: ${endStr}`;

            if (duration > 0) {
                const h = String(Math.floor(duration/3600)).padStart(2,'0');
                const m = String(Math.floor((duration%3600)/60)).padStart(2,'0');
                const s = String(duration%60).padStart(2,'0');
                disp.textContent = `${h}:${m}:${s}`;

                if (duration <= 900) { // Fase Warning 15 menit
                    card.classList.add('warning');
                    msg.style.display = 'block';
                    msg.textContent = `🔔 Sisa ${Math.ceil(duration/60)} mnt!`;
                    if (duration % 60 === 0) speak(`Bahan ${name} segera siap.`);
                }
                activeIntervals[id] = setTimeout(() => runTick(id, end, name), 1000);
            } else {
                disp.textContent = "HABIS!";
                card.classList.replace('warning', 'alert');
                btnRes.classList.add('stop-alarm-btn');
                btnRes.textContent = "STOP & AMBIL";
                document.body.classList.add('flash-alarm-red');
                speak(`Waktu thawing ${name} habis! Segera ambil bahan!`);
                activeIntervals[id] = setTimeout(() => runTick(id, end, name), 5000);
            }
        }

        function stopTick(id, defaultTime) {
            clearTimeout(activeIntervals[id]);
            const card = document.getElementById(`card-${id}`);
            if (!card) return;
            card.classList.remove('running-mode', 'warning', 'alert');
            document.getElementById(`display-${id}`).textContent = "00:00:00";
            document.getElementById(`input-${id}`).readOnly = false;
            card.querySelector('.start-btn').style.display = 'block';
            document.getElementById(`res-${id}`).style.display = 'none';
            document.getElementById(`res-${id}`).classList.remove('stop-alarm-btn');
            document.getElementById(`msg-${id}`).style.display = 'none';
            document.getElementById(`end-time-${id}`).textContent = 'Ready';
        }

        window.triggerStart = (id) => {
            window.speechSynthesis.speak(new SpeechSynthesisUtterance("")); // Unblock Audio
            const mins = document.getElementById(`input-${id}`).value;
            currentDbRef.child(id).set({ endTime: Date.now() + (mins * 60 * 1000) });
        };

        window.triggerReset = (id) => {
            document.body.classList.remove('flash-alarm-red');
            window.speechSynthesis.cancel();
            speechQueue = [];
            isSpeaking = false;
            currentDbRef.child(id).remove();
        };

        function cleanup() {
            if (currentDbRef) currentDbRef.off();
            Object.values(activeIntervals).forEach(clearTimeout);
            activeIntervals = {};
            window.speechSynthesis.cancel();
            document.body.classList.remove('flash-alarm-red');
        }
    </script>
</body>
</html>
