<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Timer Cabang - Multi Sync</title>
    
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>

    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">

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

        /* ANIMASI BACKGROUND MERAH SAAT WAKTU HABIS */
        .alarm-active {
            background-color: #ff4d6d !important;
            animation: pulse-bg 1s infinite alternate;
        }
        @keyframes pulse-bg {
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
            border: 2px solid #e0e0e0; padding: 20px;
            border-radius: 12px; background: white; transition: all 0.3s;
        }

        .timer-card.running { border-color: var(--color-primary-blue); background-color: #f0f7ff; }
        
        /* CARD JADI MERAH SAAT HABIS */
        .timer-card.expired { border-color: var(--color-alert); background-color: #fff0f1; }

        .countdown-display {
            font-size: 3em; font-weight: 700; color: var(--color-primary-blue);
        }
        .expired .countdown-display { color: var(--color-alert); }

        .btn {
            padding: 10px 20px; border: none; border-radius: 8px;
            cursor: pointer; font-weight: 600; margin-top: 10px;
        }
        .btn-start { background: var(--color-primary-blue); color: white; }
        .btn-reset { background: #6c757d; color: white; }
    </style>
</head>
<body>

<div class="main-container">
    <h1>Kitchen Timer Cabang</h1>
    <p>Sinkronisasi otomatis antar perangkat</p>
    
    <div id="timerContainer" class="timer-list">
        </div>
</div>

<script>
    // 1. MASUKKAN KONFIGURASI FIREBASE ANDA DI SINI
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
    const db = firebase.database();
    const dbRef = db.ref('timers'); // Folder utama di Firebase

    let audioCtx = null;
    let alarmInterval = null;

    // 2. FUNGSI SUARA BEEP
    function playBeep() {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = "sine";
        osc.frequency.value = 1000;
        gain.gain.value = 0.1;
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + 0.2);
    }

    // 3. LISTEN DATA DARI FIREBASE
    dbRef.on('value', (snapshot) => {
        const data = snapshot.val();
        const container = document.getElementById('timerContainer');
        container.innerHTML = ''; 
        
        let isAnyAlarm = false;

        for (let key in data) {
            const timer = data[key];
            const isExpired = timer.timeLeft <= 0 && timer.status === "RUNNING";
            if (isExpired) isAnyAlarm = true;

            const card = document.createElement('div');
            card.className = `timer-card ${timer.status === 'RUNNING' ? 'running' : ''} ${isExpired ? 'expired' : ''}`;
            card.innerHTML = `
                <h2>Timer ${key}</h2>
                <div class="countdown-display">${timer.timeLeft}s</div>
                <button class="btn btn-start" onclick="startTimer('${key}', 60)">Start 1m</button>
                <button class="btn btn-reset" onclick="resetTimer('${key}')">Reset</button>
            `;
            container.appendChild(card);
        }

        // KONTROL ALARM GLOBAL (BACKGROUND & SUARA)
        if (isAnyAlarm) {
            document.body.classList.add('alarm-active');
            if (!alarmInterval) alarmInterval = setInterval(playBeep, 1000);
        } else {
            document.body.classList.remove('alarm-active');
            clearInterval(alarmInterval);
            alarmInterval = null;
        }
    });

    // 4. LOGIKA KONTROL (HANYA MENGUBAH DATA DI FIREBASE)
    function startTimer(id, seconds) {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)(); // Unblock audio
        dbRef.child(id).update({
            timeLeft: seconds,
            status: "RUNNING"
        });

        // Jalankan hitung mundur lokal hanya untuk update Firebase
        const interval = setInterval(() => {
            dbRef.child(id).transaction((current) => {
                if (current && current.timeLeft > 0 && current.status === "RUNNING") {
                    return { ...current, timeLeft: current.timeLeft - 1 };
                }
                clearInterval(interval);
                return current;
            });
        }, 1000);
    }

    function resetTimer(id) {
        dbRef.child(id).update({
            timeLeft: 0,
            status: "IDLE"
        });
    }

    // Pastikan suara aktif setelah klik pertama (Aturan Browser)
    document.body.addEventListener('click', () => {
        if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
    }, { once: true });

</script>

</body>
</html>
