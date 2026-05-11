<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS Pro 2025 - Smart Timer</title>
    <!-- Tailwind CSS & SweetAlert2 -->
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

        /* Animation สำหรับการกระพริบเตือน */
        @keyframes blink {
            0% { opacity: 1; }
            50% { opacity: 0.3; color: #f43f5e; transform: scale(1.05); }
            100% { opacity: 1; }
        }
        .blink-warning {
            animation: blink 0.8s infinite;
        }

        .fixed-panel { flex-shrink: 0; }
        .log-container {
            flex-grow: 1;
            overflow-y: auto;
            background-color: #f8fafc;
            max-height: 240px; 
            min-height: 240px;
            border-top: 2px solid #e2e8f0;
        }
        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            height: 42px; font-size: 10px; font-weight: 800;
            border-radius: 8px; transition: all 0.1s;
        }
        .drug-btn:active { transform: scale(0.96); }

        #summary-screen {
            display: none; position: fixed; inset: 0;
            background: white; z-index: 50; overflow-y: auto; padding: 20px;
        }
    </style>
</head>
<body>

    <div id="main-app" class="flex flex-col h-full">
        <!-- Header & Timers -->
        <div class="fixed-panel p-3 bg-slate-900 text-white">
            <div class="flex justify-between items-center mb-2">
                <div>
                    <h1 class="text-xl font-black text-red-500 italic leading-none">ACLS 2025</h1>
                    <p class="text-[9px] text-slate-500 font-bold mt-1 uppercase">Advanced Life Support</p>
                </div>
                <div class="text-right">
                    <p class="text-[8px] text-slate-500 font-bold uppercase">Total Time</p>
                    <p id="total-timer" class="text-2xl font-mono font-bold text-emerald-400">00:00</p>
                </div>
            </div>
            
            <div class="grid grid-cols-2 gap-2 mb-2">
                <div class="bg-slate-800 p-2 rounded-xl border-2 border-blue-500 text-center">
                    <p class="text-[8px] text-blue-400 font-bold uppercase">Next Rhythm Check</p>
                    <p id="cpr-timer" class="text-3xl font-mono font-black transition-all">02:00</p>
                </div>
                <div class="bg-slate-800 p-2 rounded-xl border border-slate-700 flex flex-col justify-center px-2">
                    <p class="text-yellow-500 text-[9px] font-bold uppercase">Guidance</p>
                    <div id="guidance-text" class="text-slate-400 text-[10px] italic leading-tight">Waiting for Start...</div>
                </div>
            </div>

            <div class="grid grid-cols-3 gap-2">
                <button onclick="toggleStart()" id="btn-start" class="bg-emerald-600 py-2.5 rounded-lg font-black text-[10px] uppercase shadow-lg">Start Code</button>
                <button onclick="setRhythm('Shockable')" class="bg-rose-700 py-2.5 rounded-lg font-black text-[10px] uppercase shadow-lg">Shockable</button>
                <button onclick="setRhythm('Non-Shockable')" class="bg-blue-700 py-2.5 rounded-lg font-black text-[10px] uppercase shadow-lg">Non-Shock</button>
            </div>
        </div>

        <!-- Action Buttons -->
        <div class="fixed-panel p-2.5 bg-white border-b border-slate-200">
            <div class="grid grid-cols-3 gap-1.5 mb-1.5">
                <button onclick="recordAction(' CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700">START CPR</button>
                <button onclick="recordAction(' Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700">IV/IO</button>
                <button onclick="recordAction(' IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700">FLUID</button>
            </div>
            <div class="grid grid-cols-3 gap-1.5 mb-1.5">
                <button onclick="recordDefib()" class="drug-btn border-2 border-rose-500 text-rose-600 bg-rose-50 italic">DEFIB 200J</button>
                <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md">EPINEPHRINE</button>
                <button onclick="recordAction(' Amiodarone 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600">AMIODARONE 300</button>
            </div>
            <div class="grid grid-cols-2 gap-1.5">
                <button onclick="recordAction(' Amiodarone 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700 uppercase">AMIODARONE 150</button>
                <button onclick="recordAction(' Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-200 text-orange-700 italic uppercase tracking-tighter">Adv. Airway</button>
            </div>
        </div>

        <!-- Log Table -->
        <div class="log-container no-scrollbar">
            <table class="w-full text-left border-separate border-spacing-y-1 px-3 py-1">
                <thead class="sticky top-0 bg-slate-50 z-10 shadow-sm">
                    <tr class="text-[9px] text-slate-400 font-black uppercase">
                        <th class="py-1 w-14 text-center border-r">Time</th>
                        <th class="py-1 px-2">Action Recorded</th>
                    </tr>
                </thead>
                <tbody id="log-body" class="text-[10px] font-bold text-slate-700"></tbody>
            </table>
        </div>

        <!-- Footer Dashboard -->
        <div class="fixed-panel bg-white border-t p-3 flex items-center gap-3">
            <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white py-4 rounded-xl font-black text-xs uppercase shadow-lg">ROSC</button>
            <div class="flex gap-1.5 flex-1">
                <div class="bg-rose-600 text-white flex-1 py-1 rounded-xl text-center"><p class="text-[7px] uppercase leading-none mt-1">Defib</p><p id="dash-shock" class="text-xl font-black">0</p></div>
                <div class="bg-blue-600 text-white flex-1 py-1 rounded-xl text-center"><p class="text-[7px] uppercase leading-none mt-1">Epinephrine</p><p id="dash-epi" class="text-xl font-black">0</p></div>
            </div>
            <button onclick="showSummary('DEAD')" class="flex-1 bg-slate-800 text-white py-4 rounded-xl font-black text-[9px] uppercase">DEAD</button>
        </div>
    </div>

    <!-- Summary Screen -->
    <div id="summary-screen">
        <div class="flex justify-between items-center border-b-2 border-slate-900 pb-3 mb-4">
            <h2 class="text-2xl font-black italic text-slate-900">CASE SUMMARY</h2>
            <button onclick="location.reload()" class="bg-rose-500 text-white px-4 py-1 rounded-full font-bold text-xs uppercase">Reset</button>
        </div>
        <div class="grid grid-cols-2 gap-4 mb-6" id="summary-stats"></div>
        <h3 class="font-black text-slate-900 uppercase text-xs mb-2">Timeline</h3>
        <div id="sum-timeline" class="space-y-2 mb-8"></div>
        <button onclick="document.getElementById('summary-screen').style.display='none'" class="w-full py-4 border-2 border-slate-900 rounded-xl font-black uppercase">Back</button>
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
                isRunning = true; 
                btn.innerText = 'PAUSE'; 
                btn.className = 'bg-amber-600 py-2.5 rounded-lg font-black text-[10px] uppercase shadow-lg';
                
                if (totalSec === 0) recordAction(' Code Blue Activated');
                
                mainInterval = setInterval(() => {
                    totalSec++; 
                    cprSec--;
                    
                    document.getElementById('total-timer').innerText = formatTime(totalSec);
                    const cprDisplay = document.getElementById('cpr-timer');
                    cprDisplay.innerText = formatTime(cprSec);

                    // 1. กระพริบเตือนเมื่อเหลือ 15 วินาที
                    if (cprSec <= 15 && cprSec > 0) {
                        cprDisplay.classList.add('blink-warning');
                    } else {
                        cprDisplay.classList.remove('blink-warning');
                    }

                    // 2. ครบ 2 นาที (0 วินาที)
                    if (cprSec <= 0) {
                        cprSec = 120;
                        triggerRhythmCheck();
                    }
                }, 1000);
            } else {
                isRunning = false; 
                btn.innerText = 'RESUME'; 
                btn.className = 'bg-emerald-600 py-2.5 rounded-lg font-black text-[10px] uppercase shadow-lg';
                clearInterval(mainInterval);
            }
        }

        function triggerRhythmCheck() {
            // เสียงเตือน
            const msg = new SpeechSynthesisUtterance("Time is up. Check Pulse. Check EKG.");
            window.speechSynthesis.speak(msg);

            // บันทึก Log
            recordAction('⏰ RHYTHM CHECK DUE');

            // Pop-up เตือน
            Swal.fire({
                title: 'TIME IS UP!',
                html: '<b style="font-size: 20px; color: #e11d48;">Check Pulse & Check EKG</b><br>พิจารณาเปลี่ยนคนปั๊ม',
                icon: 'warning',
                confirmButtonText: 'OK, DONE',
                confirmButtonColor: '#0f172a',
                timer: 10000,
                timerProgressBar: true
            });
        }

        function setRhythm(type) {
            const g = document.getElementById('guidance-text');
            g.innerHTML = type === 'Shockable' ? '<b class="text-rose-500 uppercase">Shock 200J</b> -> Resume CPR' : '<b class="text-blue-500 uppercase">Give Epi</b> -> Resume CPR';
            recordAction(`Rhythm: ${type}`);
            cprSec = 120; // รีเซ็ตเวลาทุกครั้งที่เช็ค Rhythm
        }

        function recordDefib() {
            defibCount++; 
            document.getElementById('dash-shock').innerText = defibCount;
            recordAction(` Defib #${defibCount} (200J)`);
            cprSec = 120; // AHA: หลัง shock เริ่ม 2 นาทีใหม่
        }

        function recordEpi() {
            epiCount++; 
            document.getElementById('dash-epi').innerText = epiCount;
            recordAction(` Epinephrine #${epiCount}`);
        }

        function recordAction(msg) {
            const timeStr = formatTime(totalSec);
            timelineData.push({ time: timeStr, action: msg });
            
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-4 border-slate-300 shadow-sm";
            row.innerHTML = `<td class="text-center font-mono py-2 bg-slate-50 border-r">${timeStr}</td><td class="px-3">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
        }

        function showSummary(outcome) {
            if (isRunning) toggleStart();
            const screen = document.getElementById('summary-screen');
            screen.style.display = 'block';

            document.getElementById('summary-stats').innerHTML = `
                <div class="bg-slate-100 p-4 rounded-2xl">
                    <p class="text-slate-500 text-[10px] font-bold uppercase">Duration</p>
                    <p class="text-2xl font-mono font-black">${formatTime(totalSec)}</p>
                </div>
                <div class="bg-slate-100 p-4 rounded-2xl">
                    <p class="text-slate-500 text-[10px] font-bold uppercase">Outcome</p>
                    <p class="text-xl font-black ${outcome==='ROSC'?'text-emerald-600':'text-slate-600'}">${outcome}</p>
                </div>
            `;

            const timelineBox = document.getElementById('sum-timeline');
            timelineBox.innerHTML = '';
            timelineData.forEach(item => {
                const div = document.createElement('div');
                div.className = "flex gap-3 border-b border-slate-100 pb-2";
                div.innerHTML = `<span class="font-mono text-slate-400 text-xs w-12">${item.time}</span><span class="text-sm font-bold text-slate-700">${item.action}</span>`;
                timelineBox.appendChild(div);
            });
        }
    </script>
</body>
</html>
