<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com; style-src 'self' 'unsafe-inline'; media-src *; connect-src 'self' https://*.firebaseio.com wss://*.firebaseio.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.gstatic.com https://*.firebaseio.com; font-src 'self' data:;">
    <script type="text/javascript" src="cordova.js"></script>

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
            transition: background-color 0.3s; 
            color: var(--color-text-dark);
        }

        /* EFEK VISUAL BACKGROUND BERKEDIP SAAT ALARM */
        .flash-alarm-active {
            animation: pulse-bg-red 0.5s infinite alternate;
        }
        @keyframes pulse-bg-red {
            from { background-color: #ff4d6d; }
            to { background-color: #ffffff; }
        }

        .main-container {
            background-color: var(--color-white);
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
            text-align: center;
            width: 100%;
            max-width: 850px;
        }

        .timer-list {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); 
            gap: 20px;
            margin-top: 20px;
        }

        .timer-card {
            border: 1px solid #e0e0e0;
            padding: 20px;
            border-radius: 12px;
            background-color: var(--color-white);
            transition: all 0.3s;
        }

        .timer-card.running-mode {
            border-left: 5px solid var(--color-primary-blue); 
            background-color: #f0f7ff; 
        }

        .countdown-display {
            font-size: 2.5em;
            margin: 15px 0; 
            font-weight: 700;
            color: var(--color-primary-blue);
        }

        /* WARNA SAAT ALARM AKTIF PADA KARTU */
        .timer-card.alert {
            border-color: var(--color-alert);
            background-color: #ffeff3;
        }

        .timer-controls button {
            padding: 10px 15px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 600;
            margin: 5px;
        }

        .start-btn { background-color: var(--color-primary-blue); color: white; }
        .reset-btn { background-color: #6c757d; color: white; }
    </style>
</head>
<body>

<div class="main-container">
    <h1>Kitchen Timer Cabang</h1>
    <p>Waktu habis akan memicu suara beep dan layar merah</p>
    <div id="timerContainer" class="timer-list"></div>
</div>

<script>
    // KONFIGURASI FIREBASE ANDA
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
    const dbRef = firebase.database().ref('timers');

    let audioCtx = null;
    let alarmInterval = null;

    // FUNGSI SUARA BEEP
    function playBeep() {
        try {
            if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.type = "sine";
            osc.frequency.value = 880; 
            gain.gain.value = 0.1;
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            osc.start();
            osc.stop(audioCtx.currentTime + 0.2);
        } catch (e) { console.error("Audio error:", e); }
    }

    // LISTENER DATA FIREBASE
    dbRef.on('value', (snapshot) => {
        const data = snapshot.val();
        const container = document.getElementById('timerContainer');
        container.innerHTML = '';
        
        let shouldAlarm = false;

        for (let id in data) {
            const timer = data[id];
            const isExpired = timer.timeLeft <= 0 && timer.status === "RUNNING";
            if (isExpired) shouldAlarm = true;

            const card = document.createElement('div');
            card.className = `timer-card ${timer.status === 'RUNNING' ? 'running-mode' : ''} ${isExpired ? 'alert' : ''}`;
            card.innerHTML = `
                <h2>Timer ${id}</h2>
                <div class="countdown-display">${timer.timeLeft}s</div>
                <div class="timer-controls">
                    <button class="start-btn" onclick="startTimer('${id}', 60)">Start 1 Menit</button>
                    <button class="reset-btn" onclick="resetTimer('${id}')">Stop / Reset</button>
                </div>
            `;
            container.appendChild(card);
        }

        // LOGIKA ALARM GLOBAL
        if (shouldAlarm) {
            document.body.classList.add('flash-alarm-active');
            if (!alarmInterval) {
                playBeep();
                alarmInterval = setInterval(playBeep, 1000);
            }
        } else {
            document.body.classList.remove('flash-alarm-active');
            if (alarmInterval) {
                clearInterval(alarmInterval);
                alarmInterval = null;
            }
        }
    });

    // FUNGSI START & RESET
    function startTimer(id, seconds) {
        // Aktifkan AudioContext pada interaksi user
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        
        dbRef.child(id).set({ timeLeft: seconds, status: "RUNNING" });

        // Hitung mundur otomatis di Firebase
        const countdown = setInterval(() => {
            dbRef.child(id).transaction((current) => {
                if (current && current.timeLeft > 0 && current.status === "RUNNING") {
                    return { ...current, timeLeft: current.timeLeft - 1 };
                }
                clearInterval(countdown);
                return current;
            });
        }, 1000);
    }

    function resetTimer(id) {
        dbRef.child(id).update({ timeLeft: 0, status: "IDLE" });
    }

    // Unblock audio untuk browser
    document.addEventListener('click', () => {
        if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
    }, { once: true });

</script>
</body>
</html>
