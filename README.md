<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS Pro 2025 - Expert Mode</title>
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
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }

        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; color: #f43f5e; transform: scale(1.05); }
        }
        .blink-warning { animation: blink 0.8s infinite; }

        .log-container {
            flex-grow: 1;
            overflow-y: auto;
            background-color: #f8fafc;
            border-top: 2px solid #e2e8f0;
        }
        
        .drug-btn {
            display: flex; align-items: center; justify-content: center;
            height: 44px; font-size: 10px; font-weight: 800;
            border-radius: 12px; transition: all 0.1s;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .drug-btn:active { transform: scale(0.95); filter: brightness(0.8); }

        #summary-screen {
            display: none; position: fixed; inset: 0;
            background: white; z-index: 100; overflow-y: auto; padding: 20px;
        }
    </style>
</head>
<body>

    <div id="main-app" class="flex flex-col h-full">
        <!-- Header & Timers -->
        <div class="p-3 bg-slate-900 text-white shadow-xl z-10">
            <div class="flex justify-between items-center mb-3">
                <div>
                    <h1 class="text-2xl font-black text-rose-500 italic leading-none">ACLS PRO</h1>
                    <span id="status-badge" class="text-[9px] bg-slate-800 text-slate-400 px-2 py-0.5 rounded-full font-bold">READY</span>
                </div>
                <div class="text-right">
                    <p class="text-[8px] text-slate-500 font-bold uppercase">Total Time</p>
                    <p id="total-timer" class="text-3xl font-mono font-black text-emerald-400 leading-none">00:00</p>
                </div>
            </div>
            
            <div class="grid grid-cols-2 gap-3 mb-3">
                <div class="bg-slate-800 p-3 rounded-2xl border-2 border-blue-600/50">
                    <p class="text-[9px] text-blue-400 font-bold uppercase mb-1 text-center">Next Rhythm Check</p>
                    <p id="cpr-timer" class="text-4xl font-mono font-black text-center">02:00</p>
                </div>
                <div class="bg-slate-800 p-3 rounded-2xl border-2 border-amber-600/50">
                    <p class="text-[9px] text-amber-500 font-bold uppercase mb-1 text-center">Next Epi In</p>
                    <p id="epi-timer" class="text-4xl font-mono font-black text-center text-slate-500">--:--</p>
                </div>
            </div>

            <!-- Start / Pause Button -->
            <button onclick="toggleStart()" id="btn-start" class="w-full bg-emerald-600 py-3 mb-3 rounded-xl font-black text-sm uppercase shadow-lg border-b-4 border-emerald-800 active:border-b-0 transition-all">Start Code Blue</button>

            <!-- Rhythm Selection (4 Buttons) -->
            <div class="grid grid-cols-2 gap-2">
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="setRhythm('VF')" class="bg-rose-700 py-3 rounded-xl font-black text-xs uppercase shadow-lg border-b-4 border-rose-950 active:border-b-0">VF</button>
                    <button onclick="setRhythm('pVT')" class="bg-rose-500 py-3 rounded-xl font-black text-xs uppercase shadow-lg border-b-4 border-rose-800 active:border-b-0">pVT</button>
                </div>
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="setRhythm('PEA')" class="bg-blue-700 py-3 rounded-xl font-black text-xs uppercase shadow-lg border-b-4 border-blue-950 active:border-b-0">PEA</button>
                    <button onclick="setRhythm('Asystole')" class="bg-sky-600 py-3 rounded-xl font-black text-xs uppercase shadow-lg border-b-4 border-sky-800 active:border-b-0">Asystole</button>
                </div>
            </div>
        </div>

        <!-- Drug Actions -->
        <div class="p-3 bg-white grid grid-cols-3 gap-2 shadow-inner">
            <button onclick="recordAction('CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700 bg-emerald-50">CPR START</button>
            <button onclick="recordAction('IV/IO Access')" class="drug-btn border-2 border-sky-500 text-sky-700 bg-sky-50">IV/IO</button>
            <button onclick="recordAction('Fluid Bolus')" class="drug-btn border-2 border-blue-400 text-blue-700 bg-blue-50">FLUID</button>
            
            <button onclick="recordDefib()" class="drug-btn bg-rose-600 text-white col-span-1 italic font-black">SHOCK 200J</button>
            <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white font-black">EPINEPHRINE</button>
            <button onclick="recordAction('Amiodarone 300mg')" class="drug-btn border-2 border-purple-500 text-purple-700 bg-purple-50">AMIO 300</button>
            
            <button onclick="recordAction('Amiodarone 150mg')" class="drug-btn bg-slate-200 text-slate-700">AMIO 150</button>
            <button onclick="recordAction('Advanced Airway')" class="drug-btn bg-orange-100 text-orange-700 border border-orange-200 col-span-2 uppercase">Advanced Airway</button>
        </div>

        <!-- Log Container -->
        <div class="log-container">
            <div class="sticky top-0 bg-slate-100 px-4 py-1 text-[10px] font-black text-slate-400 uppercase flex justify-between">
                <span>Timeline</span>
                <span>Actions Recorded</span>
            </div>
            <table class="w-full text-left border-separate border-spacing-y-1 px-3">
                <tbody id="log-body" class="text-[11px] font-bold text-slate-700"></tbody>
            </table>
        </div>

        <!-- ROSC / DEAD -->
        <div class="p-3 bg-white border-t flex items-center gap-3">
            <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white py-4 rounded-2xl font-black text-sm uppercase shadow-lg">ROSC</button>
            <button onclick="showSummary('DEAD')" class="flex-1 bg-slate-800 text-white py-4 rounded-2xl font-black text-[10px] uppercase">DEAD</button>
        </div>
    </div>

    <!-- Summary Screen -->
    <div id="summary-screen">
        <div class="flex justify-between items-center border-b-4 border-slate-900 pb-4 mb-6">
            <h2 class="text-3xl font-black italic text-slate-900 uppercase">Summary</h2>
            <button onclick="location.reload()" class="bg-rose-500 text-white px-6 py-2 rounded-full font-black text-xs uppercase shadow-lg">Reset</button>
        </div>
        <div class="grid grid-cols-2 gap-4 mb-8" id="summary-stats"></div>
        <div id="sum-timeline" class="space-y-3 mb-10"></div>
        <button onclick="document.getElementById('summary-screen').style.display='none'" class="w-full py-5 bg-slate-900 text-white rounded-2xl font-black uppercase tracking-widest">Back</button>
    </div>

    <script>
        let totalSec = 0, cprSec = 120, epiSec = 0, isRunning = false, mainInterval = null;
        let defibCount = 0, epiCount = 0, timelineData = [];

        function formatTime(s) {
            const m = Math.floor(Math.abs(s) / 60).toString().padStart(2, '0');
            const sec = (Math.abs(s) % 60).toString().padStart(2, '0');
            return `${m}:${sec}`;
        }

        function toggleStart() {
            const btn = document.getElementById('btn-start');
            const status = document.getElementById('status-badge');
            
            if (!isRunning) {
                isRunning = true;
                btn.innerText = 'PAUSE';
                btn.className = 'w-full bg-amber-600 py-3 mb-3 rounded-xl font-black text-sm uppercase shadow-lg border-b-4 border-amber-800 transition-all';
                status.innerText = 'ACTIVE';
                status.className = 'text-[10px] bg-rose-500 text-white px-2 py-0.5 rounded-full font-bold animate-pulse';
                
                if (totalSec === 0) recordAction('Code Blue Activated');
                
                mainInterval = setInterval(() => {
                    totalSec++; 
                    cprSec--;
                    if(epiSec > 0) epiSec--;
                    updateDisplay();
                    if (cprSec <= 0) triggerRhythmAlert();
                }, 1000);
            } else {
                isRunning = false;
                btn.innerText = 'RESUME';
                btn.className = 'w-full bg-emerald-600 py-3 mb-3 rounded-xl font-black text-sm uppercase shadow-lg border-b-4 border-emerald-800 transition-all';
                status.innerText = 'PAUSED';
                status.className = 'text-[10px] bg-slate-700 text-slate-300 px-2 py-0.5 rounded-full font-bold';
                clearInterval(mainInterval);
            }
        }

        function updateDisplay() {
            document.getElementById('total-timer').innerText = formatTime(totalSec);
            const cprDisp = document.getElementById('cpr-timer');
            cprDisp.innerText = formatTime(cprSec);
            
            if (cprSec <= 15) cprDisp.classList.add('blink-warning');
            else cprDisp.classList.remove('blink-warning');

            const epiDisp = document.getElementById('epi-timer');
            if (epiSec > 0) {
                epiDisp.innerText = formatTime(epiSec);
                epiDisp.className = epiSec <= 30 ? 'text-4xl font-mono font-black text-center text-rose-500 animate-pulse' : 'text-4xl font-mono font-black text-center text-amber-500';
            } else if (epiCount > 0) {
                epiDisp.innerText = "READY";
                epiDisp.className = "text-2xl font-black text-center text-emerald-500 animate-bounce";
            }
        }

        function triggerRhythmAlert() {
            cprSec = 120;
            if (window.navigator.vibrate) window.navigator.vibrate([200, 100, 200]);
            const msg = new SpeechSynthesisUtterance("Time is up. Check Rhythm and Pulse.");
            window.speechSynthesis.speak(msg);
            
            Swal.fire({
                title: 'RHYTHM CHECK!',
                text: 'Check Pulse & EKG. Consider switching compressor.',
                icon: 'warning',
                confirmButtonText: 'CPR RESUMED',
                confirmButtonColor: '#0f172a',
                timer: 15000,
                timerProgressBar: true
            });
            recordAction('⏰ RHYTHM CHECK DUE');
        }

        // --- NEW RHYTHM LOGIC ---
        function setRhythm(rhythm) {
            let type = (rhythm === 'VF' || rhythm === 'pVT') ? 'Shockable' : 'Non-Shockable';
            let colorClass = type === 'Shockable' ? 'text-rose-600' : 'text-blue-600';
            
            recordAction(`Rhythm Check: <span class="${colorClass}">${rhythm} (${type})</span>`);
            
            // Auto-Guidance via Toast
            Swal.fire({
                toast: true,
                position: 'top',
                title: `Rhythm: ${rhythm}`,
                html: type === 'Shockable' ? '<b>SHOCK DELIVERED?</b> then CPR' : '<b>GIVE EPI?</b> then CPR',
                showConfirmButton: false,
                timer: 3000,
                background: type === 'Shockable' ? '#fff1f2' : '#eff6ff'
            });

            cprSec = 120; // รีเซ็ต 2 นาทีทันทีที่เลือก Rhythm
            if (!isRunning) toggleStart();
        }

        function recordDefib() {
            defibCount++;
            document.getElementById('dash-shock').innerText = defibCount;
            recordAction(`⚡ SHOCK #${defibCount} (200J)`);
            cprSec = 120;
        }

        function recordEpi() {
            epiCount++;
            epiSec = 240; // 4 mins cycle
            document.getElementById('dash-epi').innerText = epiCount;
            recordAction(`💊 Epinephrine #${epiCount}`);
        }

        function recordAction(msg) {
            const timeStr = formatTime(totalSec);
            timelineData.push({ time: timeStr, action: msg });
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-b border-slate-100";
            row.innerHTML = `<td class="py-3 w-16 font-mono text-slate-400 border-r text-center">${timeStr}</td><td class="px-4 text-slate-900">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
        }

        function showSummary(outcome) {
            if (isRunning) toggleStart();
            const screen = document.getElementById('summary-screen');
            screen.style.display = 'block';
            document.getElementById('summary-stats').innerHTML = `
                <div class="bg-slate-900 text-white p-4 rounded-3xl"><p class="text-[10px] text-slate-400 font-bold uppercase">Duration</p><p class="text-2xl font-mono font-black text-emerald-400">${formatTime(totalSec)}</p></div>
                <div class="bg-slate-100 p-4 rounded-3xl border-2 border-slate-200"><p class="text-slate-500 text-[10px] font-bold uppercase">Outcome</p><p class="text-xl font-black">${outcome}</p></div>
                <div class="bg-rose-50 p-4 rounded-3xl border-2 border-rose-200"><p class="text-rose-600 text-[10px] font-bold uppercase font-black text-center">Total Shocks: ${defibCount}</p></div>
                <div class="bg-blue-50 p-4 rounded-3xl border-2 border-blue-200"><p class="text-blue-600 text-[10px] font-bold uppercase font-black text-center">Total Epi: ${epiCount}</p></div>
            `;
            const timelineBox = document.getElementById('sum-timeline');
            timelineBox.innerHTML = '';
            timelineData.forEach(item => {
                const div = document.createElement('div');
                div.className = "flex gap-4 items-center bg-slate-50 p-3 rounded-xl border-l-4 border-slate-300";
                div.innerHTML = `<span class="font-mono text-slate-400 text-sm font-bold">${item.time}</span><span class="text-slate-800 font-bold">${item.action}</span>`;
                timelineBox.appendChild(div);
            });
        }
    </script>
</body>
</html>
