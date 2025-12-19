<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thawing Sync Reminder</title>
    
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com; style-src 'self' 'unsafe-inline'; media-src *; connect-src 'self' https://*.firebaseio.com wss://*.firebaseio.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.gstatic.com https://*.firebaseio.com https://www.googleapis.com; font-src 'self' data:;">
    <script type="text/javascript" src="cordova.js"></script>
    
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
    
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-auth.js"></script>
    
    <style>
        /* ==================== 
           CSS: STYLING (Tetap Sama)
           ==================== */
        :root {
            --color-white: #ffffff;
            --color-light-bg: #f7f9ff;
            --color-primary-blue: #4A90E2;
            --color-accent-pink: #FF69B4;
            --color-text-dark: #333333;
            --color-warning: #FFC0CB;
            --color-alert: #DC3545;
            --color-syncing: #FFA500;
        }
        
        body {
            font-family: 'Poppins', sans-serif;
            background-color: var(--color-light-bg);
            min-height: 100vh;
            margin: 0; 
            padding: 20px 10px;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            transition: background-color 0.2s; 
            color: var(--color-text-dark);
        }

        .main-container {
            background-color: var(--color-white);
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
            text-align: center;
            width: 100%;
            max-width: 850px;
            box-sizing: border-box; 
        }

        #auth-container {
            max-width: 350px;
            margin: 0 auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            background: #fff;
            text-align: left;
        }
        #auth-container h2 { text-align: center; color: var(--color-primary-blue); margin-bottom: 20px; font-weight: 700; }
        .auth-group { margin-bottom: 15px; }
        .auth-group label { display: block; margin-bottom: 5px; font-weight: 600; font-size: 0.9em; }
        .auth-group input { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box; }
        
        #login-btn, #logout-btn {
            width: 100%; padding: 12px; border: none; border-radius: 8px; cursor: pointer; font-weight: 600;
            background-color: var(--color-primary-blue); color: white; transition: background-color 0.2s;
        }

        .header-controls { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
        .header-controls h1 { margin: 0; font-size: 1.5em; }
        .logout-btn-small { padding: 8px 9px; font-size: 0.8em; font-weight: 600; background-color: #f44336; color: white; border: none; border-radius: 6px; cursor: pointer; width: auto !important; }

        .timer-list { display: grid; grid-template-columns: repeat(2, 1fr); gap: 20px; margin-top: 20px; }
        .timer-card { border: 1px solid #e0e0e0; padding: 20px; border-radius: 12px; background-color: var(--color-white); transition: all 0.3s; text-align: center; box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05); }
        .timer-card.running-mode { border-left: 5px solid var(--color-primary-blue); background-color: #f0f7ff; box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1); }
        .countdown-display { font-size: 2.5em; margin: 15px 0 5px 0; font-weight: 700; color: var(--color-primary-blue); }
        .end-time-display { font-size: 0.85em; color: #999; margin-bottom: 10px; }
        .alarm-message { color: var(--color-alert); font-weight: 600; margin-top: 10px; font-size: 0.95em; }
        .timer-controls { display: flex; gap: 10px; margin-top: 15px; align-items: center; justify-content: center; }
        .timer-controls input { padding: 10px 5px; max-width: 60px; text-align: center; border: 1px solid #ddd; border-radius: 8px; }
        .timer-controls button { padding: 10px 15px; border: none; border-radius: 8px; cursor: pointer; font-weight: 600; flex-grow: 1; }
        
        .start-btn { background-color: var(--color-primary-blue); color: white; }
        .reset-btn { background-color: #6c757d; color: white; }
        .stop-alarm-btn { background-color: var(--color-alert) !important; color: white; animation: pulse-stop 0.5s infinite alternate; }
        .flash-alarm-red { background-color: #ff4d6d !important; }

        @keyframes pulse-stop { from { opacity: 1; } to { opacity: 0.8; } }

        @media (max-width: 650px) {
            .timer-list { grid-template-columns: 1fr; }
            .timer-controls { flex-wrap: wrap; }
            .timer-controls button { flex-basis: calc(50% - 5px); }
        }
    </style>
</head>
<body>
    <div class="main-container">
        <div id="auth-container">
            <h2>Login Thawing Timer</h2>
            <div class="auth-group">
                <label>Email</label>
                <input type="email" id="email" placeholder="Masukkan email" required>
            </div>
            <div class="auth-group">
                <label>Password</label>
                <input type="password" id="password" placeholder="Masukkan password" required>
            </div>
            <button id="login-btn">LOGIN</button>
            <p id="error-message" style="display: none; color: red; text-align: center; margin-top: 10px; font-size: 0.8em;"></p>
        </div>

        <div id="timer-content" style="display: none;">
            <div class="header-controls">
                <div>
                    <h1 id="main-title">Gacoan Timer Thawing 🧊</h1>
                    <p id="sub-title">Malang Jakarta V.1.4 (Multi-Branch)</p>
                </div>
                <button id="logout-btn" class="logout-btn-small">LOGOUT</button>
            </div>
            <div class="timer-list" id="timer-list"></div>
        </div>
    </div>

    <script>
        // 1. CONFIGURATION
        const firebaseConfig = {
          apiKey: "AIzaSyBtUlghTw806GuGuwOXGNgoqN6Rkcg0IMM",
          authDomain: "thawing-ec583.firebaseapp.com",
          databaseURL: "https://thawing-ec583-default-rtdb.asia-southeast1.firebasedatabase.app",
          projectId: "thawing-ec583",
          storageBucket: "thawing-ec583.firebasestorage.app", 
          messagingSenderId: "1043079332713",
          appId: "1:1043079332713:web:6d289ad2b7c13a222bb3f8",
          measurementId: "G-WLBFJ6V7QT"            
        };

        // Initialize Firebase
        if (!firebase.apps.length) { firebase.initializeApp(firebaseConfig); }
        const auth = firebase.auth();

        // ==========================================
        // 🚨 LOGIKA MULTI-BRANCH (OPSI 1)
        // ==========================================
        const urlParams = new URLSearchParams(window.location.search);
        const branchName = urlParams.get('branch') || 'pusat'; // Default jika tanpa parameter
        const dbRef = firebase.database().ref('branches/' + branchName);

        // UI Auth Elements
        const authContainer = document.getElementById('auth-container');
        const timerContent = document.getElementById('timer-content');
        const loginBtn = document.getElementById('login-btn');
        const logoutBtn = document.getElementById('logout-btn');
        const emailInput = document.getElementById('email');
        const passwordInput = document.getElementById('password');
        const errorMessage = document.getElementById('error-message');
        const mainTitle = document.getElementById('main-title');

        // Update Title Cabang
        mainTitle.innerText = `Gacoan Timer (${branchName.toUpperCase()}) 🧊`;

        // 2. AUTH LOGIC
        loginBtn.addEventListener('click', () => {
            auth.signInWithEmailAndPassword(emailInput.value, passwordInput.value)
                .catch(err => {
                    errorMessage.innerText = "Login Gagal: " + err.message;
                    errorMessage.style.display = 'block';
                });
        });

        logoutBtn.addEventListener('click', () => auth.signOut());

        auth.onAuthStateChanged(user => {
            if (user) {
                authContainer.style.display = 'none';
                timerContent.style.display = 'block';
                initializeTimerApp();
            } else {
                authContainer.style.display = 'block';
                timerContent.style.display = 'none';
                cleanupTimerApp();
            }
        });

        // 3. TIMER APP LOGIC
        let activeIntervals = {};
        const THAWING_ITEMS = [
            { id: 1, name: "ADONAN", defaultTimeMinutes: 40 },
            { id: 2, name: "ACIN", defaultTimeMinutes: 60 },
            { id: 3, name: "MIE", defaultTimeMinutes: 120 },
            { id: 4, name: "PENTOL", defaultTimeMinutes: 120 },
            { id: 5, name: "SURAI NAGA", defaultTimeMinutes: 120 },
            { id: 6, name: "KRUPUK MIE", defaultTimeMinutes: 120 },
            { id: 7, name: "KULIT PANGSIT", defaultTimeMinutes: 120 },
        ];

        function formatTime(totalSeconds) {
            totalSeconds = Math.max(0, totalSeconds);
            const h = String(Math.floor(totalSeconds / 3600)).padStart(2, '0');
            const m = String(Math.floor((totalSeconds % 3600) / 60)).padStart(2, '0');
            const s = String(totalSeconds % 60).padStart(2, '0');
            return `${h}:${m}:${s}`;
        }

        function initializeTimerApp() {
            const timerListContainer = document.getElementById('timer-list');
            timerListContainer.innerHTML = ''; 

            THAWING_ITEMS.forEach(item => {
                const card = document.createElement('div');
                card.className = 'timer-card';
                card.id = `card-${item.id}`;
                card.innerHTML = `
                    <h2>${item.name}</h2>
                    <div id="display-${item.id}" class="countdown-display">${formatTime(item.defaultTimeMinutes * 60)}</div>
                    <div id="end-time-${item.id}" class="end-time-display">Durasi default</div> 
                    <div id="msg-${item.id}" class="alarm-message" style="display: none;"></div>
                    <div class="timer-controls">
                        <input type="number" id="time-input-${item.id}" value="${item.defaultTimeMinutes}">
                        <button id="start-btn-${item.id}" class="start-btn">START</button>
                        <button id="reset-btn-${item.id}" class="reset-btn" style="display: none;">RESET</button>
                    </div>
                `;
                timerListContainer.appendChild(card);

                document.getElementById(`start-btn-${item.id}`).onclick = () => {
                    const mins = parseInt(document.getElementById(`time-input-${item.id}`).value);
                    const endTime = Date.now() + (mins * 60 * 1000);
                    dbRef.child(item.id).set({ endTime, inputMinutes: mins });
                };

                document.getElementById(`reset-btn-${item.id}`).onclick = () => {
                    stopAggressiveAlarm();
                    dbRef.child(item.id).remove();
                };
            });

            // Sinkronisasi Real-time
            dbRef.on('value', snapshot => {
                const data = snapshot.val() || {};
                THAWING_ITEMS.forEach(item => {
                    if (data[item.id]) {
                        runTick(item.id, data[item.id].endTime, data[item.id].inputMinutes);
                    } else {
                        resetUI(item.id);
                    }
                });
            });
        }

        function runTick(id, endTime, mins) {
            clearInterval(activeIntervals[id]);
            const card = document.getElementById(`card-${id}`);
            const display = document.getElementById(`display-${id}`);
            const input = document.getElementById(`time-input-${id}`);
            const startBtn = document.getElementById(`start-btn-${id}`);
            const resetBtn = document.getElementById(`reset-btn-${id}`);
            const endText = document.getElementById(`end-time-${id}`);

            input.readOnly = true;
            startBtn.style.display = 'none';
            resetBtn.style.display = 'block';
            card.classList.add('running-mode');
            endText.innerText = "Selesai: " + new Date(endTime).toLocaleTimeString('id-ID', {hour:'2-digit', minute:'2-digit'});

            activeIntervals[id] = setInterval(() => {
                const diff = Math.floor((endTime - Date.now()) / 1000);
                if (diff <= 0) {
                    clearInterval(activeIntervals[id]);
                    display.innerText = "WAKTU HABIS!";
                    card.classList.add('alert');
                    resetBtn.innerText = "AMBIL BAHAN";
                    resetBtn.classList.add('stop-alarm-btn');
                    startAggressiveAlarm(id);
                } else {
                    display.innerText = formatTime(diff);
                }
            }, 1000);
        }

        function resetUI(id) {
            clearInterval(activeIntervals[id]);
            const item = THAWING_ITEMS.find(i => i.id === id);
            const card = document.getElementById(`card-${id}`);
            if (!card) return;
            card.classList.remove('running-mode', 'alert', 'warning');
            document.getElementById(`display-${id}`).innerText = formatTime(item.defaultTimeMinutes * 60);
            document.getElementById(`time-input-${id}`).readOnly = false;
            document.getElementById(`time-input-${id}`).value = item.defaultTimeMinutes;
            document.getElementById(`start-btn-${id}`).style.display = 'block';
            const rb = document.getElementById(`reset-btn-${id}`);
            rb.style.display = 'none';
            rb.innerText = "RESET";
            rb.classList.remove('stop-alarm-btn');
            document.getElementById(`end-time-${id}`).innerText = "Durasi default";
        }

        // ALARM HELPERS
        let alarmInterval;
        function startAggressiveAlarm(id) {
            if (alarmInterval) return;
            const item = THAWING_ITEMS.find(i => i.id === id);
            alarmInterval = setInterval(() => {
                document.body.classList.toggle('flash-alarm-red');
                if ('speechSynthesis' in window && !window.speechSynthesis.speaking) {
                    const msg = new SpeechSynthesisUtterance(`Waktu ${item.name} habis`);
                    msg.lang = 'id-ID';
                    window.speechSynthesis.speak(msg);
                }
            }, 1000);
        }

        function stopAggressiveAlarm() {
            clearInterval(alarmInterval);
            alarmInterval = null;
            document.body.classList.remove('flash-alarm-red');
            window.speechSynthesis.cancel();
        }

        function cleanupTimerApp() {
            for (let k in activeIntervals) clearInterval(activeIntervals[k]);
            dbRef.off();
            stopAggressiveAlarm();
        }
    </script>
</body>
</html>
