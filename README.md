<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Thawing Sync Reminder</title>
    
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; media-src *; connect-src 'self' https://*.firebaseio.com wss://*.firebaseio.com https://script.google.com https://script.googleusercontent.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://www.gstatic.com https://*.firebaseio.com; font-src 'self' data: https://fonts.gstatic.com;">
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

        h1 { color: var(--color-primary-blue); font-weight: 700; margin-bottom: 5px; }
        p { color: #666; margin-bottom: 25px; font-size: 0.9em; }

        .timer-list {
            display: grid;
            grid-template-columns: repeat(2, 1fr); 
            gap: 20px;
            margin-top: 20px;
        }

        .timer-card {
            border: 1px solid #e0e0e0;
            padding: 20px;
            border-radius: 12px;
            background-color: var(--color-white);
            transition: all 0.3s;
            text-align: center;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
        }

        .timer-card.running-mode {
            border-left: 5px solid var(--color-primary-blue); 
            background-color: #f0f7ff; 
        }

        .timer-card.running-mode h2 { color: var(--color-primary-blue); }
        .timer-card.running-mode .countdown-display { color: var(--color-accent-pink); }

        .timer-card h2 { color: var(--color-accent-pink); font-size: 1.3em; font-weight: 600; }

        .countdown-display {
            font-size: 2.5em;
            margin: 15px 0 5px 0; 
            font-weight: 700;
            color: var(--color-primary-blue);
        }
        
        .end-time-display { font-size: 0.85em; color: #999; margin-bottom: 10px; }
        .alarm-message { color: var(--color-alert); font-weight: 600; margin-top: 10px; font-size: 0.95em; }

        .timer-controls {
            display: flex;
            gap: 10px;
            margin-top: 15px;
            align-items: center;
            justify-content: center;
        }

        .timer-controls input {
            padding: 10px 5px;
            max-width: 60px; 
            text-align: center; 
            border: 1px solid #ddd;
            border-radius: 8px;
        }

        .timer-controls button {
            padding: 10px 15px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 600;
            flex-grow: 1; 
        }

        .start-btn { background-color: var(--color-primary-blue); color: white; }
        .reset-btn { background-color: #6c757d; color: white; }
        .stop-alarm-btn { background-color: var(--color-alert) !important; color: white; animation: pulse-stop 0.5s infinite alternate; }

        .timer-card.warning { border-color: var(--color-warning); background-color: #fffafa; }
        .timer-card.alert { border-color: var(--color-alert); background-color: #ffeff3; }

        .flash-alarm-red { background-color: #ff4d6d !important; }

        @keyframes pulse-stop {
            from { opacity: 1; box-shadow: 0 0 10px rgba(220, 53, 69, 0.7); }
            to { opacity: 0.8; box-shadow: none; }
        }

        @media (max-width: 650px) {
            .timer-list { grid-template-columns: 1fr; }
            .timer-controls { flex-wrap: wrap; }
            .timer-controls button { flex-basis: calc(50% - 5px); }
        }
    </style>
</head>
<body>
    <div class="main-container">
        <h1>Gacoan Timer Thawing 🧊</h1>
        <p>Malang Jakarta V.1.4 (Rekap Spreadsheet)</p>
        <div class="timer-list" id="timer-list"></div>
    </div>

    <script>
        // ===================================
        // FIREBASE CONFIGURATION
        // ===================================
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

        if (!firebase.apps.length) {
            firebase.initializeApp(firebaseConfig);
        }
        
        const dbRef = firebase.database().ref('thawingTimers');
        const SPREADSHEET_URL = "https://script.google.com/macros/s/AKfycbwFXw1vNzCzsMIm4BDysaC1qcJzcbByyQvTrPrNilnv5Z7sJT8u_IzNgV_uoaIWvI9F/exec";

        // --- KONFIGURASI ITEM ---
        const THAWING_ITEMS = [
            { id: 1, name: "ADONAN", defaultTimeMinutes: 40 },
            { id: 2, name: "ACIN", defaultTimeMinutes: 120 },
            { id: 3, name: "MIE", defaultTimeMinutes: 120 },
            { id: 4, name: "PENTOL", defaultTimeMinutes: 120 },
            { id: 5, name: "SURAI NAGA", defaultTimeMinutes: 120 },
            { id: 6, name: "KRUPUK MIE", defaultTimeMinutes: 120 },
            { id: 7, name: "KULIT PANGSIT", defaultTimeMinutes: 120 },
        ];
        
        const WARNING_TIME_SECONDS = 15 * 60; 
        let activeIntervals = {}; 
        let isSpeaking = false; 
        const speechQueue = []; 
        let flashInterval = null;
        let titleInterval = null;
        const originalTitle = document.title;

        // --- UTILITY FUNCTIONS ---
        function formatTime(totalSeconds) {
            totalSeconds = Math.max(0, totalSeconds); 
            const h = String(Math.floor(totalSeconds / 3600)).padStart(2, '0');
            const m = String(Math.floor((totalSeconds % 3600) / 60)).padStart(2, '0');
            const s = String(totalSeconds % 60).padStart(2, '0');
            return `${h}:${m}:${s}`;
        }

        function speakMessage(message) {
            if ('speechSynthesis' in window) {
                if (!speechQueue.includes(message)) speechQueue.push(message);
                processQueue();
            }
        }

        function processQueue() {
            if (isSpeaking || speechQueue.length === 0) return;
            isSpeaking = true;
            const msg = speechQueue.shift();
            const utterance = new SpeechSynthesisUtterance(msg);
            utterance.lang = 'id-ID';
            utterance.onend = () => { isSpeaking = false; processQueue(); };
            utterance.onerror = () => { isSpeaking = false; processQueue(); };
            window.speechSynthesis.speak(utterance);
        }

        function sendToSpreadsheet(itemName, duration, endTimeMs) {
            const endTimeString = new Date(endTimeMs).toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' });
            fetch(SPREADSHEET_URL, {
                method: "POST",
                mode: "no-cors",
                body: JSON.stringify({ itemName, duration, endTimeString })
            }).catch(e => console.error("Spreadsheet Error:", e));
        }

        // --- CORE LOGIC ---
        function tick(itemId, endTimeMs, inputMinutes) {
            const timerCard = document.getElementById(`card-${itemId}`);
            if (!timerCard) return;

            const display = document.getElementById(`display-${itemId}`);
            const alarmMsg = document.getElementById(`msg-${itemId}`);
            const endTimeDisplay = document.getElementById(`end-time-${itemId}`);
            const item = THAWING_ITEMS.find(i => i.id === itemId);
            
            clearTimeout(activeIntervals[itemId]);

            const now = Date.now();
            let duration = Math.floor((endTimeMs - now) / 1000);

            // UI Mode Running
            timerCard.classList.add('running-mode');
            document.getElementById(`time-input-${itemId}`).readOnly = true;
            document.getElementById(`start-btn-${itemId}`).style.display = 'none';
            const resetBtn = document.getElementById(`reset-btn-${itemId}`);
            resetBtn.style.display = 'block';

            endTimeDisplay.textContent = `Selesai: ${new Date(endTimeMs).toLocaleTimeString('id-ID', {hour:'2-digit', minute:'2-digit'})}`;

            if (duration > 0) {
                display.textContent = formatTime(duration);
                if (duration <= WARNING_TIME_SECONDS) {
                    timerCard.classList.add('warning');
                    alarmMsg.style.display = 'block';
                    alarmMsg.textContent = `🔔 Tersisa ${Math.ceil(duration/60)} menit!`;
                    if (duration % 60 === 0) speakMessage(`Waktu ${item.name} tinggal ${Math.ceil(duration/60)} menit.`);
                }
                activeIntervals[itemId] = setTimeout(() => tick(itemId, endTimeMs, inputMinutes), 1000);
            } else {
                // Time Up
                display.textContent = "WAKTU HABIS!";
                timerCard.classList.replace('warning', 'alert');
                alarmMsg.textContent = `✅ SELESAI! Segera ambil ${item.name}.`;
                resetBtn.textContent = "STOP & AMBIL";
                resetBtn.classList.add('stop-alarm-btn');
                
                startGlobalAlert(item.name);
            }
        }

        function startGlobalAlert(name) {
            if (!flashInterval) {
                flashInterval = setInterval(() => document.body.classList.toggle('flash-alarm-red'), 300);
            }
            speakMessage(`Peringatan! Waktu thawing ${name} telah habis!`);
        }

        function stopGlobalAlert() {
            clearInterval(flashInterval);
            flashInterval = null;
            document.body.classList.remove('flash-alarm-red');
            window.speechSynthesis.cancel();
            speechQueue.length = 0;
            isSpeaking = false;
        }

        function startCountdown(itemId) {
            const input = document.getElementById(`time-input-${itemId}`);
            const durationMinutes = parseInt(input.value);
            if (isNaN(durationMinutes) || durationMinutes <= 0) return alert("Masukkan waktu valid!");

            const endTimeMs = Date.now() + (durationMinutes * 60 * 1000);
            const item = THAWING_ITEMS.find(i => i.id === itemId);

            dbRef.child(itemId).set({ endTime: endTimeMs, inputMinutes: durationMinutes })
                .then(() => {
                    sendToSpreadsheet(item.name, durationMinutes, endTimeMs);
                });
        }

        function resetTimer(itemId) {
            stopGlobalAlert();
            dbRef.child(itemId).remove();
        }

        function localResetUI(itemId) {
            const item = THAWING_ITEMS.find(i => i.id === itemId);
            const card = document.getElementById(`card-${itemId}`);
            if (!card) return;

            clearTimeout(activeIntervals[itemId]);
            card.className = 'timer-card';
            document.getElementById(`time-input-${itemId}`).readOnly = false;
            document.getElementById(`time-input-${itemId}`).value = item.defaultTimeMinutes;
            document.getElementById(`display-${itemId}`).textContent = formatTime(item.defaultTimeMinutes * 60);
            document.getElementById(`start-btn-${itemId}`).style.display = 'block';
            const rb = document.getElementById(`reset-btn-${itemId}`);
            rb.style.display = 'none';
            rb.className = 'reset-btn';
            rb.textContent = 'RESET';
            document.getElementById(`msg-${itemId}`).style.display = 'none';
            document.getElementById(`end-time-${itemId}`).textContent = 'Durasi default';
        }

        // --- INIT ---
        document.addEventListener('DOMContentLoaded', () => {
            const container = document.getElementById('timer-list');
            THAWING_ITEMS.forEach(item => {
                const card = document.createElement('div');
                card.className = 'timer-card';
                card.id = `card-${item.id}`;
                card.innerHTML = `
                    <h2>${item.name}</h2>
                    <div id="display-${item.id}" class="countdown-display">${formatTime(item.defaultTimeMinutes*60)}</div>
                    <div id="end-time-${item.id}" class="end-time-display">Durasi default</div>
                    <div id="msg-${item.id}" class="alarm-message" style="display:none"></div>
                    <div class="timer-controls">
                        <input type="number" id="time-input-${item.id}" value="${item.defaultTimeMinutes}">
                        <button onclick="startCountdown(${item.id})" class="start-btn" id="start-btn-${item.id}">START</button>
                        <button onclick="resetTimer(${item.id})" class="reset-btn" id="reset-btn-${item.id}" style="display:none">RESET</button>
                    </div>
                `;
                container.appendChild(card);
            });

            dbRef.on('value', snapshot => {
                const data = snapshot.val() || {};
                THAWING_ITEMS.forEach(item => {
                    if (data[item.id]) {
                        tick(item.id, data[item.id].endTime, data[item.id].inputMinutes);
                    } else {
                        localResetUI(item.id);
                    }
                });
            });
        });
    </script>
</body>
</html>
