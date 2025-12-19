<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gacoan Timer - Multi Branch</title>
    
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap" rel="stylesheet">
    
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-auth.js"></script>
    
    <style>
        :root {
            --color-white: #ffffff;
            --color-bg: #f4f7f6;
            --color-primary: #4A90E2;
            --color-accent: #FF69B4;
            --color-text-main: #212529; /* Warna font gelap konsisten */
            --color-text-muted: #6c757d;
            --color-danger: #dc3545;
            --color-warning: #ffc107;
        }
        
        body {
            font-family: 'Poppins', sans-serif;
            background-color: var(--color-bg);
            color: var(--color-text-main);
            margin: 0; 
            padding: 10px;
            display: flex; justify-content: center;
        }

        .main-container {
            background: var(--color-white);
            padding: 25px;
            border-radius: 15px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 800px;
            margin-top: 20px;
        }

        /* Form Login */
        #auth-container { max-width: 400px; margin: 0 auto; text-align: left; }
        .auth-group { margin-bottom: 15px; }
        .auth-group label { display: block; font-weight: 600; margin-bottom: 5px; color: var(--color-text-main); }
        .auth-group input { width: 100%; padding: 12px; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; }
        
        #login-btn { width: 100%; padding: 12px; background: var(--color-primary); color: white; border: none; border-radius: 8px; font-weight: 700; cursor: pointer; }

        /* Header */
        .header-controls { display: flex; justify-content: space-between; align-items: flex-start; border-bottom: 2px solid #eee; margin-bottom: 20px; padding-bottom: 10px; }
        .header-controls h1 { margin: 0; color: var(--color-primary); font-size: 1.4em; }
        .header-controls p { margin: 0; font-size: 0.85em; color: var(--color-text-muted); }
        .logout-btn { background: var(--color-danger); color: white; border: none; padding: 8px 12px; border-radius: 6px; cursor: pointer; font-size: 0.8em; font-weight: 600; }

        /* Timer Cards */
        .timer-list { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 15px; }
        .timer-card { border: 2px solid #eee; padding: 20px; border-radius: 12px; position: relative; transition: 0.3s; background: #fff; }
        
        .timer-card h2 { margin: 0; color: var(--color-text-main); font-size: 1.2em; font-weight: 700; }
        .countdown-display { font-size: 2.8em; font-weight: 700; color: var(--color-primary); margin: 10px 0; }
        .end-time-display { font-size: 0.9em; color: var(--color-text-muted); font-weight: 400; }
        
        /* State Running */
        .timer-card.running { border-color: var(--color-primary); background: #f0f7ff; }
        
        /* State Alert/Finish */
        .timer-card.alert { border-color: var(--color-danger); background: #fff5f5; }
        .timer-card.alert .countdown-display { color: var(--color-danger); }

        /* Controls */
        .timer-controls { display: flex; gap: 8px; margin-top: 15px; align-items: center; }
        .timer-controls input { width: 70px; padding: 8px; text-align: center; border: 1px solid #ccc; border-radius: 6px; font-weight: 700; color: var(--color-text-main); }
        .btn { flex: 1; padding: 10px; border: none; border-radius: 6px; cursor: pointer; font-weight: 700; }
        .btn-start { background: var(--color-primary); color: white; }
        .btn-reset { background: var(--color-text-muted); color: white; }
        .btn-stop { background: var(--color-danger); color: white; animation: pulse 0.8s infinite alternate; }

        @keyframes pulse { from { transform: scale(1); } to { transform: scale(1.03); } }
        .flash-red { background-color: #ff4d4d !important; }

        @media (max-width: 600px) { .timer-list { grid-template-columns: 1fr; } }
    </style>
</head>
<body>
    <div class="main-container">
        <div id="auth-container">
            <h2 style="text-align: center; color: var(--color-primary);">Login Cabang</h2>
            <div class="auth-group">
                <label>Email Cabang</label>
                <input type="email" id="email" placeholder="contoh: malang@gacoan.com">
            </div>
            <div class="auth-group">
                <label>Password</label>
                <input type="password" id="password" placeholder="******">
            </div>
            <button id="login-btn">MASUK KE SISTEM</button>
            <p id="error-msg" style="color:red; font-size:0.8em; display:none; text-align:center;"></p>
        </div>

        <div id="timer-content" style="display: none;">
            <div class="header-controls">
                <div>
                    <h1 id="branch-title">Gacoan Timer</h1>
                    <p id="version-tag">V.1.5 - Sync Stable</p>
                </div>
                <button id="logout-btn" class="logout-btn">LOGOUT</button>
            </div>
            <div id="timer-list" class="timer-list"></div>
        </div>
    </div>

    <script>
        // 1. CONFIG
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
        const urlParams = new URLSearchParams(window.location.search);
        const branchName = urlParams.get('branch') || 'pusat';
        const dbRef = firebase.database().ref('branches/' + branchName);

        const THAWING_ITEMS = [
            { id: 1, name: "ADONAN", time: 40 },
            { id: 2, name: "ACIN", time: 60 },
            { id: 3, name: "MIE", time: 120 },
            { id: 4, name: "PENTOL", time: 120 },
            { id: 5, name: "SURAI NAGA", time: 120 },
            { id: 6, name: "KRUPUK MIE", time: 120 },
            { id: 7, name: "KULIT PANGSIT", time: 120 }
        ];

        let activeIntervals = {};

        // 2. AUTH HANDLER
        document.getElementById('login-btn').onclick = () => {
            const email = document.getElementById('email').value;
            const pass = document.getElementById('password').value;
            auth.signInWithEmailAndPassword(email, pass).catch(e => {
                const err = document.getElementById('error-msg');
                err.innerText = "Gagal: " + e.message;
                err.style.display = 'block';
            });
        };

        document.getElementById('logout-btn').onclick = () => auth.signOut();

        auth.onAuthStateChanged(user => {
            if (user) {
                document.getElementById('auth-container').style.display = 'none';
                document.getElementById('timer-content').style.display = 'block';
                document.getElementById('branch-title').innerText = `Gacoan (${branchName.toUpperCase()})`;
                initApp();
            } else {
                document.getElementById('auth-container').style.display = 'block';
                document.getElementById('timer-content').style.display = 'none';
                stopGlobalAlarm();
            }
        });

        // 3. CORE LOGIC
        function initApp() {
            const container = document.getElementById('timer-list');
            container.innerHTML = '';
            
            THAWING_ITEMS.forEach(item => {
                const div = document.createElement('div');
                div.className = 'timer-card';
                div.id = `card-${item.id}`;
                div.innerHTML = `
                    <h2>${item.name}</h2>
                    <div class="countdown-display" id="display-${item.id}">--:--:--</div>
                    <div class="end-time-display" id="end-${item.id}">Standby</div>
                    <div class="timer-controls">
                        <input type="number" id="input-${item.id}" value="${item.time}">
                        <button class="btn btn-start" id="btn-start-${item.id}">START</button>
                        <button class="btn btn-reset" id="btn-reset-${item.id}" style="display:none">RESET</button>
                    </div>
                `;
                container.appendChild(div);

                document.getElementById(`btn-start-${item.id}`).onclick = () => {
                    const m = parseInt(document.getElementById(`input-${item.id}`).value);
                    const end = Date.now() + (m * 60 * 1000);
                    dbRef.child(item.id).set({ end, m });
                };

                document.getElementById(`btn-reset-${item.id}`).onclick = () => {
                    dbRef.child(item.id).remove();
                    stopGlobalAlarm();
                };
            });

            // Sinkronisasi Realtime
            dbRef.on('value', snap => {
                const data = snap.val() || {};
                THAWING_ITEMS.forEach(item => {
                    const state = data[item.id];
                    if (state) {
                        updateTimer(item.id, state.end, state.m);
                    } else {
                        clearLocalTimer(item.id);
                    }
                });
            });
        }

        function updateTimer(id, end, m) {
            clearInterval(activeIntervals[id]);
            const display = document.getElementById(`display-${id}`);
            const card = document.getElementById(`card-${id}`);
            const input = document.getElementById(`input-${id}`);
            const btnS = document.getElementById(`btn-start-${id}`);
            const btnR = document.getElementById(`btn-reset-${id}`);
            const endTxt = document.getElementById(`end-${id}`);

            input.readOnly = true;
            btnS.style.display = 'none';
            btnR.style.display = 'block';
            card.classList.add('running');
            endTxt.innerText = "Selesai jam: " + new Date(end).toLocaleTimeString('id-ID',{hour:'2-digit', minute:'2-digit'});

            activeIntervals[id] = setInterval(() => {
                const now = Date.now();
                const diff = Math.floor((end - now) / 1000);

                if (diff <= 0) {
                    clearInterval(activeIntervals[id]);
                    display.innerText = "WAKTU HABIS!";
                    card.classList.add('alert');
                    btnR.innerText = "STOP & AMBIL";
                    btnR.className = "btn btn-stop";
                    triggerAlarm(id);
                } else {
                    const hrs = Math.floor(diff / 3600);
                    const mins = Math.floor((diff % 3600) / 60);
                    const secs = diff % 60;
                    display.innerText = `${String(hrs).padStart(2,'0')}:${String(mins).padStart(2,'0')}:${String(secs).padStart(2,'0')}`;
                }
            }, 1000);
        }

        function clearLocalTimer(id) {
            clearInterval(activeIntervals[id]);
            const item = THAWING_ITEMS.find(i => i.id === id);
            const card = document.getElementById(`card-${id}`);
            if (!card) return;

            card.className = 'timer-card';
            document.getElementById(`display-${id}`).innerText = "00:00:00";
            document.getElementById(`input-${id}`).readOnly = false;
            document.getElementById(`input-${id}`).value = item.time;
            document.getElementById(`btn-start-${id}`).style.display = 'block';
            const rb = document.getElementById(`btn-reset-${id}`);
            rb.style.display = 'none';
            rb.innerText = "RESET";
            rb.className = "btn btn-reset";
            document.getElementById(`end-${id}`).innerText = "Standby";
        }

        // 4. ALARM SYSTEM
        let globalAlarmId = null;
        function triggerAlarm(id) {
            if (globalAlarmId) return;
            const item = THAWING_ITEMS.find(i => i.id === id);
            globalAlarmId = setInterval(() => {
                document.body.classList.toggle('flash-red');
                if ('speechSynthesis' in window && !speechSynthesis.speaking) {
                    const utter = new SpeechSynthesisUtterance(`${item.name} HABIS`);
                    utter.lang = 'id-ID';
                    speechSynthesis.speak(utter);
                }
            }, 1000);
        }

        function stopGlobalAlarm() {
            clearInterval(globalAlarmId);
            globalAlarmId = null;
            document.body.classList.remove('flash-red');
            if ('speechSynthesis' in window) speechSynthesis.cancel();
        }
    </script>
</body>
</html>
