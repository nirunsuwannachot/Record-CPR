<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS Pro 2026 - Real-time Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    
    <style>
        :root { --app-height: 100dvh; }
        body {
            height: var(--app-height);
            margin: 0;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            background-color: #0f172a;
            font-family: system-ui, -apple-system, sans-serif;
        }

        /* Animation การกระพริบเตือน */
        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; color: #f43f5e; transform: scale(1.05); }
        }
        .blink-warning {
            animation: blink 0.8s infinite;
            color: #f43f5e !important;
        }

        .log-container {
            flex-grow: 1;
            overflow-y: auto;
            background-color: #f1f5f9;
            max-height: 260px; 
            min-height: 260px;
            border-top: 2px solid #e2e8f0;
        }

        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            height: 44px; font-size: 10px; font-weight: 800;
            border-radius: 8px; transition: all 0.1s;
        }
        .drug-btn:active { transform: scale(0.96); }

        #summary-screen {
            display: none; position: fixed; inset: 0;
            background: white; z-index: 100; overflow-y: auto; padding: 20px;
        }
    </style>
</head>
<body>

    <div id="main-app" class="flex flex-col h-full">
        <!-- Top Panel: Timers & Rhythm -->
        <div class="p-3 bg-slate-900 text-white shadow-xl">
            <div class="flex justify-between items-center mb-2">
                <div>
                    <h1 class="text-xl font-black text-rose-500 italic leading-none">ACLS2025</h1>
                </div>
                <div class="text-right">
                    <p class="text-[8px] text-slate-500 font-bold uppercase">Total Time</p>
                    <p id="total-timer" class="text-2xl font-mono font-bold text-emerald-400 leading-none">00:00</p>
                </div>
            </div>
            
            <div class="grid grid-cols-2 gap-2 mb-3">
                <div class="bg-slate-800 p-2 rounded-xl border-2 border-blue-500 text-center">
                    <p class="text-[8px] text-blue-400 font-bold uppercase mb-1">Next Rhythm Check</p>
                    <p id="cpr-timer" class="text-4xl font-mono font-black">02:00</p>
                </div>
                <div class="bg-slate-800 p-2 rounded-xl border border-slate-700 flex flex-col justify-center">
                    <p class="text-amber-500 text-[9px] font-bold uppercase mb-1">Guidance</p>
                    <div id="guidance-text" class="text-slate-400 text-[10px] italic leading-tight">System Ready...</div>
                </div>
            </div>

            <button onclick="toggleStart()" id="btn-start" class="w-full bg-emerald-600 py-3 mb-3 rounded-xl font-black text-sm uppercase shadow-lg border-b-4 border-emerald-800 active:border-b-0 transition-all">Start Code Blue</button>

            <!-- Rhythm Selection: 4 Buttons -->
            <div class="grid grid-cols-2 gap-2">
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="setRhythm('VF')" class="bg-rose-700 py-3 rounded-xl font-black text-xs shadow-lg border-b-4 border-rose-950 active:border-b-0">VF</button>
                    <button onclick="setRhythm('pVT')" class="bg-rose-500 py-3 rounded-xl font-black text-xs shadow-lg border-b-4 border-rose-800 active:border-b-0">pVT</button>
                </div>
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="setRhythm('PEA')" class="bg-blue-700 py-3 rounded-xl font-black text-xs shadow-lg border-b-4 border-blue-950 active:border-b-0">PEA</button>
                    <button onclick="setRhythm('Asystole')" class="bg-sky-600 py-3 rounded-xl font-black text-xs shadow-lg border-b-4 border-sky-800 active:border-b-0">Asystole</button>
                </div>
            </div>
        </div>

        <!-- Action Panel -->
        <div class="p-2.5 bg-white border-b border-slate-200">
            <div class="grid grid-cols-3 gap-1.5 mb-1.5">
                <button onclick="recordAction('CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700">START CPR</button>
                <button onclick="recordAction('Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700">IV/IO</button>
                <button onclick="recordAction('IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700">FLUID</button>
            </div>
            <div class="grid grid-cols-3 gap-1.5 mb-1.5">
                <button onclick="recordDefib()" class="drug-btn border-2 border-rose-500 text-rose-600 bg-rose-50 italic">DEFIB 200J</button>
                <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md">EPINEPHRINE</button>
                <button onclick="recordAction('Amiodarone 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600">AMIO 300</button>
            </div>
            <div class="grid grid-cols-2 gap-1.5">
                <button onclick="recordAction('Amiodarone 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700">AMIO 150</button>
                <button onclick="recordAction('Advanced Airway')" class="drug-btn bg-orange-50 border border-orange-200 text-orange-700 italic">ADV AIRWAY</button>
            </div>
        </div>

        <!-- Log Table with Real-time Timestamp -->
        <div class="log-container no-scrollbar">
            <table class="w-full text-left border-separate border-spacing-y-1 px-3 py-1">
                <thead class="sticky top-0 bg-slate-50 z-10 shadow-sm">
                    <tr class="text-[9px] text-slate-400 font-black uppercase">
                        <th class="py-1 w-20 text-center border-r">Time (Real)</th>
                        <th class="py-1 px-2">Action Recorded</th>
                    </tr>
                </thead>
                <tbody id="log-body" class="text-[10px] font-bold text-slate-700"></tbody>
            </table>
        </div>

        <!-- Bottom Footer -->
        <div class="bg-white border-t p-3 flex items-center gap-3">
            <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white py-4 rounded-xl font-black text-xs uppercase shadow-lg">ROSC</button>
            <div class="flex gap-1.5 flex-1">
                <div class="bg-rose-600 text-white flex-1 py-1 rounded-xl text-center"><p class="text-[7px] uppercase mt-1">Defib</p><p id="dash-shock" class="text-xl font-black">0</p></div>
                <div class="bg-blue-600 text-white flex-1 py-1 rounded-xl text-center"><p class="text-[7px] uppercase mt-1">Epi</p><p id="dash-epi" class="text-xl font-black">0</p></div>
            </div>
            <button onclick="showSummary('STOP CPR')" class="flex-1 bg-slate-800 text-white py-4 rounded-xl font-black text-[9px] uppercase">STOP CPR</button>
        </div>
    </div>

    <!-- Summary Screen -->
    <div id="summary-screen">
        <div class="flex justify-between items-center border-b-2 border-slate-900 pb-3 mb-4">
            <h2 class="text-2xl font-black italic text-slate-900 uppercase">Case Summary</h2>
            <button onclick="location.reload()" class="bg-rose-500 text-white px-4 py-1 rounded-full font-bold text-xs uppercase">New Case</button>
        </div>
        <div id="summary-stats" class="grid grid-cols-2 gap-4 mb-6"></div>
        <h3 class="font-black text-slate-900 uppercase text-xs mb-2">Timeline History</h3>
        <div id="sum-timeline" class="space-y-2 mb-8 border-t pt-2"></div>
        <button onclick="document.getElementById('summary-screen').style.display='none'" class="w-full py-4 border-2 border-slate-900 rounded-xl font-black text-slate-900 uppercase">Close View</button>
    </div>

    <script>
        let totalSec = 0, cprSec = 120, isRunning = false, mainInterval = null;
        let defibCount = 0, epiCount = 0;
        let timelineData = [];

        function formatTime(s) {
            const m = Math.floor(s / 60).toString().padStart(2, '0');
            const sec = (s % 60).toString().padStart(2, '0');
            return `${m}:${sec}`;
        }

        function toggleStart() {
            const btn = document.getElementById('btn-start');
            if (!isRunning) {
                isRunning = true; btn.innerText = 'PAUSE'; btn.className = 'w-full bg-amber-600 py-3 mb-3 rounded-xl font-black text-sm uppercase shadow-lg transition-all';
                if (totalSec === 0) recordAction('Code Blue Activated');
                
                mainInterval = setInterval(() => {
                    totalSec++; cprSec--;
                    document.getElementById('total-timer').innerText = formatTime(totalSec);
                    const cprDisp = document.getElementById('cpr-timer');
                    cprDisp.innerText = formatTime(cprSec);

                    // 15 Sec Blink Warning
                    if (cprSec <= 15 && cprSec > 0) cprDisp.classList.add('blink-warning');
                    else cprDisp.classList.remove('blink-warning');

                    if (cprSec <= 0) {
                        cprSec = 120;
                        triggerAlert();
                    }
                }, 1000);
            } else {
                isRunning = false; btn.innerText = 'RESUME'; btn.className = 'w-full bg-emerald-600 py-3 mb-3 rounded-xl font-black text-sm uppercase shadow-lg transition-all';
                clearInterval(mainInterval);
            }
        }

        function triggerAlert() {
            if ('speechSynthesis' in window) {
                window.speechSynthesis.speak(new SpeechSynthesisUtterance("Time is up. Rhythm Check."));
            }
            recordAction('⏰ Rhythm Check Due!');
            Swal.fire({
                title: 'RHYTHM CHECK!',
                html: '<p class="text-xl font-bold text-rose-600">Check Pulse & EKG</p>Switch Compressor',
                icon: 'warning',
                confirmButtonText: 'CPR RESUMED',
                confirmButtonColor: '#0f172a',
                timer: 15000,
                timerProgressBar: true
            });
        }

        function recordAction(msg) {
            const now = new Date();
            const realTime = now.getHours().toString().padStart(2, '0') + ':' + 
                             now.getMinutes().toString().padStart(2, '0') + ':' + 
                             now.getSeconds().toString().padStart(2, '0');
            const elapsed = formatTime(totalSec);

            timelineData.push({ realTime, elapsed, action: msg });
            
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-4 border-slate-300 shadow-sm";
            row.innerHTML = `
                <td class="text-center font-mono py-2 bg-slate-50 border-r">
                    <div class="text-slate-900 leading-none">${realTime}</div>
                    <div class="text-[8px] text-slate-400 mt-1">T+${elapsed}</div>
                </td>
                <td class="px-3">${msg}</td>
            `;
            body.insertBefore(row, body.firstChild);
        }

        function setRhythm(type) {
            const isShock = (type === 'VF' || type === 'pVT');
            document.getElementById('guidance-text').innerHTML = isShock ? 
                `<b class="text-rose-500 uppercase">Shock 200J</b> -> CPR` : 
                `<b class="text-blue-500 uppercase">Give Epi</b> -> CPR`;
            recordAction(`Rhythm: ${type} (${isShock ? 'Shockable' : 'Non-shock'})`);
            cprSec = 120;
        }

        function recordDefib() {
            defibCount++; 
            document.getElementById('dash-shock').innerText = defibCount;
            recordAction(` Defib #${defibCount} (200J)`);
            cprSec = 120;
        }

        function recordEpi() {
            epiCount++; 
            document.getElementById('dash-epi').innerText = epiCount;
            recordAction(` Epinephrine #${epiCount}`);
        }

        function showSummary(outcome) {
            if (isRunning) toggleStart();
            document.getElementById('summary-screen').style.display = 'block';

            document.getElementById('summary-stats').innerHTML = `
                <div class="bg-slate-100 p-3 rounded-xl"><p class="text-[9px] text-slate-500 uppercase font-bold">Total Time</p><p class="text-xl font-black">${formatTime(totalSec)}</p></div>
                <div class="bg-slate-100 p-3 rounded-xl"><p class="text-[9px] text-slate-500 uppercase font-bold">Outcome</p><p class="text-lg font-black ${outcome==='ROSC'?'text-emerald-600':'text-slate-600'}">${outcome}</p></div>
                <div class="bg-rose-100 p-3 rounded-xl border border-rose-200"><p class="text-rose-600 text-[9px] uppercase font-bold">Total Shock</p><p class="text-2xl font-black text-rose-700">${defibCount}</p></div>
                <div class="bg-blue-100 p-3 rounded-xl border border-blue-200"><p class="text-blue-600 text-[9px] uppercase font-bold">Total Epi</p><p class="text-2xl font-black text-blue-700">${epiCount}</p></div>
            `;

            const timelineBox = document.getElementById('sum-timeline');
            timelineBox.innerHTML = '';
            timelineData.forEach(item => {
                const div = document.createElement('div');
                div.className = "flex gap-3 border-b border-slate-100 pb-1";
                div.innerHTML = `<span class="font-mono text-slate-400 text-[10px] w-24">[${item.realTime}]</span><span class="text-[11px] font-bold text-slate-700">${item.action}</span>`;
                timelineBox.appendChild(div);
            });
        }
    </script>
</body>
</html>
