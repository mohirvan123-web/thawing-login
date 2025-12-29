
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Timer Thawing Pro V.1.6</title>
    
    <link rel="manifest" href="manifest.json">
    <meta name="theme-color" content="#4A90E2">
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
            --color-warning: #ffd166;
            --color-alert: #ef476f;
            --color-success: #06d6a0;
        }

        body {
            font-family: 'Poppins', sans-serif;
            background-color: var(--color-light-bg);
            margin: 0; padding: 20px 10px;
            display: flex; justify-content: center;
            color: var(--color-text-dark);
        }

        .main-container {
            background-color: var(--color-white);
            padding: 25px; border-radius: 16px;
            box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
            width: 100%; max-width: 900px; text-align: center;
        }

        .branch-badge {
            display: inline-block;
            background: var(--color-primary-blue);
            color: white; padding: 5px 18px;
            border-radius: 20px; font-size: 0.85em;
            margin-bottom: 15px; font-weight: 600;
            letter-spacing: 0.5px;
        }

        h1 { color: var(--color-primary-blue); margin: 0; font-size: 1.8em; }
        p.subtitle { color: #666; font-size: 0.9em; margin-bottom: 30px; }

        .timer-list {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
            gap: 15px;
        }

        .timer-card {
            border: 2px solid #edf2f7;
            padding: 20px; border-radius: 15px;
            background-color: var(--color-white);
            transition: all 0.3s ease;
            position: relative;
        }

        .timer-card.running-mode {
            border-color: var(--color-primary-blue);
            background-color: #f0f7ff;
            transform: translateY(-3px);
            box-shadow: 0 4px 12px rgba(74, 144, 226, 0.15);
        }

        .timer-card h2 { color: #444; font-size: 1.1em; margin: 0 0 10px 0; text-transform: uppercase; }
        .running-mode h2 { color: var(--color-primary-blue); }

        .countdown-display {
            font-size: 2.4em; font-weight: 700;
            margin: 10px 0; color: #444;
            font-variant-numeric: tabular-nums;
        }

        .running-mode .countdown-display { color: var(--color-accent-pink); }

        .end-time-display { font-size: 0.8em; color: #999; margin-bottom: 15px; min-height: 1.2em; }

        .timer-controls {
            display: flex; gap: 8px; justify-content: center;
        }

        .timer-controls input {
            width: 60px; padding: 8px; text-align: center;
            border: 2px solid #ddd; border-radius: 8px;
            font-family: inherit; font-weight: 600;
        }

        .btn {
            padding: 10px 15px; border: none; border-radius: 8px;
            font-weight: 700; cursor: pointer; flex-grow: 1;
            transition: opacity 0.2s;
        }

        .btn:active { transform: scale(0.98); }

        .start-btn { background: var(--color-primary-blue); color: white; }
        .reset-btn { background: #cbd5e0; color: #4a5568; }

        /* State Styles */
        .warning { background-color: #fff9eb !important; border-color: var(--color-warning) !important; }
        .alert { 
            background-color: #fff1f2 !important; 
            border-color: var(--color-alert) !important; 
            animation: shake 0.5s infinite;
        }

        .stop-alarm-btn { 
            background: var(--color-alert) !important; 
            color: white !important;
            animation: pulse-stop 0.6s infinite alternate; 
        }

        @keyframes pulse-stop {
            from { transform: scale(1); } to { transform: scale(1.05); }
        }

        @keyframes shake {
            0% { transform: translateX(0); }
            25% { transform: translateX(-2px); }
            75% { transform: translateX(2px); }
            100% { transform: translateX(0); }
        }

        @media (max-width: 600px) {
            .timer-list { grid-template-columns: 1fr; }
            h1 { font-size: 1.4em; }
        }
    </style>
</head>
<body>

<div class="main-container">
    <div id="branch-tag" class="branch-badge">Cabang: Memuat...</div>
    <h1>Gacoan Timer Thawing 🧊</h1>
    <p class="subtitle">Multi-Cabang System V.1.6</p>

    <div class="timer-list" id="timer-list">
        </div>
</div>

<script>
/* =====================
   FIREBASE CONFIG
===================== */
const firebaseConfig = {
    apiKey: "AIzaSyCbk_lVmOP36gqN76rN1Iqd5JRDiwEYA_w",
  authDomain: "gacoantimerthawing.firebaseapp.com",
  databaseURL: "https://gacoantimerthawing-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId: "gacoantimerthawing",
  storageBucket: "gacoantimerthawing.firebasestorage.app",
  messagingSenderId: "587812124504",
  appId: "1:587812124504:web:d78b7cca9eb38b2a5a972d",
};

if (!firebase.apps.length) {
    firebase.initializeApp(firebaseConfig);
}

/* =====================
   BRANCH SYSTEM
===================== */
function getBranchId() {
    const urlParams = new URLSearchParams(window.location.search);
    let branch = urlParams.get("branch");
    
    if (branch) {
        localStorage.setItem("TIMER_BRANCH_ID", branch);
    } else {
        branch = localStorage.getItem("TIMER_BRANCH_ID") || "Pusat";
    }
    return branch;
}

const ACTIVE_BRANCH = getBranchId();
const dbRef = firebase.database().ref(`branches/${ACTIVE_BRANCH}/thawingTimers`);
document.getElementById("branch-tag").innerText = "CABANG: " + ACTIVE_BRANCH.toUpperCase();

/* =====================
   APP LOGIC
===================== */
const THAWING_ITEMS = [
    { id: 1, name: "ADONAN", defaultTime: 40 },
    { id: 2, name: "ACIN", defaultTime: 120 },
    { id: 3, name: "MIE", defaultTime: 120 },
    { id: 4, name: "PENTOL", defaultTime: 120 },
    { id: 5, name: "SURAI NAGA", defaultTime: 120 },
    { id: 6, name: "KRUPUK MIE", defaultTime: 120 },
    { id: 7, name: "KULIT PANGSIT", defaultTime: 120 },
    { id: 8, name: "UDANG KEJU", defaultTime: 120 }
];

let activeIntervals = {};
let speechQueue = [];
let isSpeaking = false;

function formatTime(totalSeconds) {
    const h = Math.floor(totalSeconds / 3600).toString().padStart(2, '0');
    const m = Math.floor((totalSeconds % 3600) / 60).toString().padStart(2, '0');
    const s = (totalSeconds % 60).toString().padStart(2, '0');
    return `${h}:${m}:${s}`;
}

function createTimerCards() {
    const container = document.getElementById('timer-list');
    container.innerHTML = '';
    THAWING_ITEMS.forEach(item => {
        const card = document.createElement('div');
        card.className = 'timer-card';
        card.id = `card-${item.id}`;
        card.innerHTML = `
            <h2>${item.name}</h2>
            <div id="display-${item.id}" class="countdown-display">${formatTime(item.defaultTime * 60)}</div>
            <div id="end-time-${item.id}" class="end-time-display">Siap digunakan</div>
            <div class="timer-controls">
                <input type="number" id="input-${item.id}" value="${item.defaultTime}" min="1">
                <button class="btn start-btn" id="start-${item.id}" onclick="startTimer(${item.id})">START</button>
                <button class="btn reset-btn" id="reset-${item.id}" style="display:none" onclick="resetTimer(${item.id})">RESET</button>
            </div>
        `;
        container.appendChild(card);
    });
}

function startTimer(id) {
    const inputEl = document.getElementById(`input-${id}`);
    const mins = parseInt(inputEl.value);
    
    if (isNaN(mins) || mins <= 0) {
        alert("Masukkan durasi menit yang valid!");
        return;
    }

    const endTime = Date.now() + (mins * 60 * 1000);
    dbRef.child(id).set({ 
        endTime, 
        inputMinutes: mins,
        itemName: THAWING_ITEMS.find(i => i.id === id).name 
    });
}

function resetTimer(id) {
    dbRef.child(id).remove();
    stopAllAudio();
}

function stopAllAudio() {
    window.speechSynthesis.cancel();
    speechQueue = [];
    isSpeaking = false;
}

function speakMessage(message) {
    // Hindari duplikasi pesan yang sama dalam antrean
    if (!speechQueue.includes(message)) {
        speechQueue.push(message);
    }
    if (!isSpeaking) {
        processSpeechQueue();
    }
}

function processSpeechQueue() {
    if (speechQueue.length === 0) {
        isSpeaking = false;
        return;
    }
    
    isSpeaking = true;
    const msg = new SpeechSynthesisUtterance(speechQueue.shift());
    msg.lang = 'id-ID';
    msg.rate = 0.9; // Sedikit lebih lambat agar jelas
    
    msg.onend = () => {
        processSpeechQueue();
    };
    
    msg.onerror = () => {
        isSpeaking = false;
    };

    window.speechSynthesis.speak(msg);
}

function tick(id, endTime) {
    const card = document.getElementById(`card-${id}`);
    const display = document.getElementById(`display-${id}`);
    const endDisplay = document.getElementById(`end-time-${id}`);
    const btnStart = document.getElementById(`start-${id}`);
    const btnReset = document.getElementById(`reset-${id}`);

    clearInterval(activeIntervals[id]);

    activeIntervals[id] = setInterval(() => {
        const now = Date.now();
        const diff = Math.floor((endTime - now) / 1000);
        const itemName = THAWING_ITEMS.find(i => i.id === id).name;
        
        card.classList.add('running-mode');
        btnStart.style.display = 'none';
        btnReset.style.display = 'block';
        
        const endObj = new Date(endTime);
        endDisplay.innerText = "Selesai jam: " + endObj.getHours().toString().padStart(2,'0') + ":" + endObj.getMinutes().toString().padStart(2,'0');

        if (diff <= 0) {
            display.innerText = "HABIS!";
            card.classList.remove('warning');
            card.classList.add('alert');
            btnReset.innerText = "STOP ALARM";
            btnReset.classList.add('stop-alarm-btn');
            
            // Ulangi peringatan setiap 10 detik saat habis
            if (diff % 10 === 0) {
                speakMessage(`Waktu thawing ${itemName} sudah habis.`);
            }
        } else {
            display.innerText = formatTime(diff);
            btnReset.innerText = "RESET";
            btnReset.classList.remove('stop-alarm-btn');
            
            // Warning 15 menit terakhir
            if (diff < 900) {
                card.classList.add('warning');
                // Suara peringatan setiap 5 menit di masa kritis
                if (diff % 300 === 0) {
                    speakMessage(`Perhatian, thawing ${itemName} tersisa ${Math.ceil(diff/60)} menit.`);
                }
            }
        }
    }, 1000);
}

/* =====================
   FIREBASE SYNC
===================== */
document.addEventListener("DOMContentLoaded", () => {
    createTimerCards();
    
    dbRef.on('value', snapshot => {
        const data = snapshot.val() || {};
        
        THAWING_ITEMS.forEach(item => {
            const state = data[item.id];
            const card = document.getElementById(`card-${item.id}`);
            
            if (state) {
                tick(item.id, state.endTime);
            } else {
                clearInterval(activeIntervals[item.id]);
                if(card) {
                    card.classList.remove('running-mode', 'warning', 'alert');
                    document.getElementById(`display-${item.id}`).innerText = formatTime(item.defaultTime * 60);
                    document.getElementById(`start-${item.id}`).style.display = 'block';
                    document.getElementById(`reset-${item.id}`).style.display = 'none';
                    document.getElementById(`end-time-${item.id}`).innerText = "Siap digunakan";
                }
            }
        });
    });
});
</script>
</body>
</html>
