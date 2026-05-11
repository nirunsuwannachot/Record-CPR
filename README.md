# Record-CPR
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS Pro Recorder</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* ล็อคความสูงหน้าจอให้พอดีกับ Device จริงๆ (แก้ปัญหาแถบ URL บัง) */
        :root { --app-height: 100dvh; }
        body {
            height: var(--app-height);
            margin: 0;
            overflow: hidden; /* ห้ามเลื่อนทั้งหน้าจอ */
            display: flex;
            flex-direction: column;
            background-color: #0f172a;
            touch-action: manipulation;
        }

        /* ส่วน Action Log: บังคับความสูงคงที่และเลื่อนได้ภายในตัว */
        .log-area {
            flex: 1; /* กินพื้นที่ที่เหลือระหว่างปุ่ม */
            min-height: 200px; /* กะความสูงให้พอดีกับ ~5 แถว */
            overflow-y: auto;
            -webkit-overflow-scrolling: touch;
            background-color: #f8fafc;
        }

        .no-scrollbar::-webkit-scrollbar { display: none; }
        
        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            height: 46px; font-size: 10px; font-weight: 900;
            border-radius: 8px; transition: all 0.1s;
        }
        .drug-btn:active { transform: scale(0.96); opacity: 0.8; }

        /* ป้องกันส่วนหัวและท้ายถูกบีบ */
        .header-panel, .control-panel, .footer-panel { flex-shrink: 0; }
    </style>
</head>
<body class="font-sans antialiased">

    <div class="header-panel p-4 bg-slate-900 text-white shadow-xl">
        <div class="flex justify-between items-center mb-3">
            <div>
                <h1 class="text-xl font-black text-red-500 italic leading-none">ACLS 2025</h1>
                <p class="text-[9px] text-slate-500 font-bold uppercase mt-1">Nirun Suwannachot</p>
            </div>
            <div class="text-right">
                <p class="text-[9px] text-slate-500 font-bold uppercase">Total Time</p>
                <p id="total-timer" class="text-2xl font-mono font-bold text-emerald-400">00:00</p>
            </div>
        </div>
        <div class="grid grid-cols-2 gap-2">
            <div id="cpr-card" class="bg-slate-800 p-2 rounded-xl border-2 border-blue-500 text-center">
                <p class="text-[9px] text-blue-400 font-bold uppercase">Rhythm Check In</p>
                <p id="cpr-timer" class="text-3xl font-mono font-black">02:00</p>
            </div>
            <div class="bg-slate-800 p-2 rounded-xl border border-slate-700 text-[10px]">
                <p class="text-yellow-500 font-bold uppercase mb-0.5">Algorithm</p>
                <div id="guidance-text" class="text-slate-400 italic leading-tight">Waiting for Rhythm...</div>
            </div>
        </div>
        <div class="grid grid-cols-3 gap-2 mt-3">
            <button onclick="toggleStart()" id="btn-start" class="bg-emerald-600 py-3 rounded-lg font-black text-xs uppercase shadow-md text-white">Start</button>
            <button onclick="setRhythm('Shockable')" class="bg-rose-700 py-3 rounded-lg font-black text-xs uppercase text-white">Shockable</button>
            <button onclick="setRhythm('Non-Shockable')" class="bg-blue-700 py-3 rounded-lg font-black text-[9px] uppercase text-white">Non-Shock</button>
        </div>
    </div>

    <div class="control-panel p-3 bg-white border-b border-slate-200">
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordAction('💓 START CPR')" class="drug-btn border-2 border-emerald-500 text-emerald-700 bg-emerald-50">START CPR</button>
            <button onclick="recordAction('💉 Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700 bg-sky-50 uppercase">IV/IO</button>
            <button onclick="recordAction('💧 IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700 bg-blue-50 uppercase">Fluid</button>
        </div>
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordDefib()" class="drug-btn border-2 border-rose-500 text-rose-600 bg-rose-50 italic uppercase">Defib 200J</button>
            <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md uppercase">Epinephrine</button>
            <button onclick="recordAmio()" class="drug-btn border-2 border-purple-500 text-purple-600 bg-white uppercase">Amio 300</button>
        </div>
        <div class="grid grid-cols-2 gap-2">
            <button onclick="recordAction('💊 Amio 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700 uppercase">Amio 150</button>
            <button onclick="recordAction('🌬️ Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-300 text-orange-700 italic uppercase">Adv. Airway</button>
        </div>
    </div>

    <div class="log-area no-scrollbar relative">
        <table class="w-full text-left border-separate border-spacing-y-1 p-2">
            <thead class="sticky top-0 bg-slate-50 shadow-sm z-10">
                <tr class="text-[9px] text-slate-400 uppercase font-black">
                    <th class="py-1.5 w-16 text-center">Time</th>
                    <th class="py-1.5 px-2">Action</th>
                </tr>
            </thead>
            <tbody id="log-body" class="text-xs font-bold text-slate-700"></tbody>
        </table>
        <div class="sticky bottom-2 right-2 flex justify-end gap-2 pr-2 pb-2 pointer-events-none">
            <div class="bg-rose-600 text-white px-2 py-1 rounded-lg border border-white shadow-lg text-[9px] font-black">SHOCK: <span id="dash-shock">0</span></div>
            <div class="bg-blue-600 text-white px-2 py-1 rounded-lg border border-white shadow-lg text-[9px] font-black">EPI: <span id="dash-epi">0</span></div>
        </div>
    </div>

    <div class="footer-panel p-3 bg-white border-t flex gap-3">
        <button onclick="recordAction('🏁 ROSC Achieved')" class="flex-1 bg-emerald-500 text-white py-3.5 rounded-xl font-black text-xs uppercase shadow-lg">ROSC</button>
        <button id="btn-close-case" onclick="closeCase()" class="flex-1 bg-slate-800 text-white py-3.5 rounded-xl font-black text-xs uppercase">Close Case</button>
    </div>

    <div id="summary-modal" class="fixed inset-0 z-50 hidden bg-black/90 backdrop-blur-sm p-4 flex items-center justify-center">
        <div class="bg-white w-full max-w-sm rounded-3xl p-6 shadow-2xl flex flex-col max-h-[85vh]">
            <h2 class="text-2xl font-black mb-4 uppercase border-b pb-2 text-slate-800">Case Summary</h2>
            <div class="overflow-y-auto mb-4 no-scrollbar">
                <div class="grid grid-cols-2 gap-2 mb-4 text-center">
                    <div class="bg-slate-100 p-2 rounded-xl"><p class="text-[9px] font-bold text-slate-500">TOTAL TIME</p><p id="sum-time" class="text-xl font-black">00:00</p></div>
                    <div class="bg-rose-100 p-2 rounded-xl"><p class="text-[9px] font-bold text-rose-500">SHOCK</p><p id="sum-shock" class="text-xl font-black text-rose-600">0</p></div>
                </div>
                <h3 class="text-xs font-black text-slate-400 uppercase mb-2">Rhythm Timeline</h3>
                <div id="rhythm-timeline" class="space-y-1 text-xs"></div>
            </div>
            <button onclick="document.getElementById('summary-modal').style.display='none'" class="w-full bg-slate-800 text-white py-4 rounded-2xl font-black shadow-lg">DONE</button>
        </div>
    </div>

    <script>
        let totalSec = 0, cprSec = 120, isRunning = false, mainInterval = null;
        let defibCount = 0, epiCount = 0, rhythmHistory = [];

        function formatTime(s) {
            const m = Math.floor(s / 60).toString().padStart(2, '0');
            const sec = (s % 60).toString().padStart(2, '0');
            return `${m}:${sec}`;
        }

        function toggleStart() {
            const btn = document.getElementById('btn-start');
            if (!isRunning) {
                isRunning = true;
                btn.innerText = 'PAUSE'; btn.classList.replace('bg-emerald-600', 'bg-amber-600');
                if (totalSec === 0) recordAction('🚀 Code Blue Activated');
                mainInterval = setInterval(() => {
                    totalSec++; cprSec--;
                    document.getElementById('total-timer').innerText = formatTime(totalSec);
                    document.getElementById('cpr-timer').innerText = formatTime(cprSec);
                    if (cprSec <= 0) { cprSec = 120; recordAction('⚠️ 2-Min Rhythm Check!'); }
                }, 1000);
            } else {
                isRunning = false;
                btn.innerText = 'RESUME'; btn.classList.replace('bg-amber-600', 'bg-emerald-600');
                clearInterval(mainInterval);
            }
        }

        function setRhythm(type) {
            const guide = document.getElementById('guidance-text');
            guide.innerHTML = type === 'Shockable' ? '<b class="text-rose-400">⚡ VF/pVT:</b> Shock 200J -> CPR' : '<b class="text-blue-400">💉 PEA/Asy:</b> Epi ASAP -> CPR';
            rhythmHistory.push({ time: formatTime(totalSec), type });
            recordAction(`🔍 Rhythm: ${type}`);
            cprSec = 120;
        }

        function recordDefib() {
            defibCount++; document.getElementById('dash-shock').innerText = defibCount;
            recordAction(`⚡ Defibrillation #${defibCount} (200J)`);
            cprSec = 120;
        }

        function recordEpi() {
            epiCount++; document.getElementById('dash-epi').innerText = epiCount;
            recordAction(`💉 Epinephrine #${epiCount} (1 mg)`);
        }

        function recordAction(msg) {
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-4 border-slate-300 shadow-sm";
            row.innerHTML = `<td class="py-2 text-center bg-slate-100 font-mono text-[10px] font-black">${formatTime(totalSec)}</td><td class="py-2 px-3">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
            document.querySelector('.log-area').scrollTop = 0;
        }

        function closeCase() {
            isRunning = false; clearInterval(mainInterval);
            document.getElementById('sum-time').innerText = formatTime(totalSec);
            document.getElementById('sum-shock').innerText = defibCount;
            const timeline = document.getElementById('rhythm-timeline');
            timeline.innerHTML = rhythmHistory.map(h => `<div class="p-2 bg-slate-50 rounded border-l-2 border-slate-300 flex justify-between"><span>${h.type}</span><span>${h.time}</span></div>`).join('');
            document.getElementById('summary-modal').style.display = 'flex';
        }
    </script>
</body>
</html>
