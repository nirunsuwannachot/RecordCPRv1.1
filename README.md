# RecordCPRv1.1
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS Pro 2025 - Fixed Layout</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* ล็อคความสูงหน้าจอไม่ให้ Scroll ทั้งหน้า */
        :root { --app-height: 100dvh; }
        body {
            height: var(--app-height);
            margin: 0;
            overflow: hidden; 
            display: flex;
            flex-direction: column; /* เรียงจากบนลงล่าง */
            background-color: #0f172a;
        }

        /* ส่วนที่เป็นปุ่มและหัวข้อ ให้คงขนาดเดิมไว้ ไม่ให้โดนเบียด */
        .fixed-panel { flex-shrink: 0; }

        /* หัวใจสำคัญ: ส่วน Log ให้ยืดเท่าพื้นที่ที่เหลือ และถ้าเกินให้ Scroll ในตัวเอง */
        .log-container {
            flex-grow: 1; 
            overflow-y: auto;
            -webkit-overflow-scrolling: touch;
            background-color: #f1f5f9;
        }

        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            height: 44px; font-size: 10px; font-weight: 800;
            border-radius: 8px; transition: all 0.1s;
        }
        .drug-btn:active { transform: scale(0.96); opacity: 0.8; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
    </style>
</head>
<body>

    <div class="fixed-panel p-3 bg-slate-900 text-white shadow-xl">
        <div class="flex justify-between items-center mb-2">
            <div>
                <h1 class="text-xl font-black text-red-500 italic leading-none">ACLS 2025</h1>
                <p class="text-[9px] text-slate-500 font-bold mt-1 uppercase">NIRUN SUWANNACHOT</p>
            </div>
            <div class="text-right">
                <p class="text-[8px] text-slate-500 font-bold uppercase">Total Time</p>
                <p id="total-timer" class="text-2xl font-mono font-bold text-emerald-400">00:00</p>
            </div>
        </div>
        
        <div class="grid grid-cols-2 gap-2 mb-2">
            <div class="bg-slate-800 p-2 rounded-xl border-2 border-blue-500 text-center">
                <p class="text-[8px] text-blue-400 font-bold">RHYTHM CHECK IN</p>
                <p id="cpr-timer" class="text-3xl font-mono font-black">02:00</p>
            </div>
            <div class="bg-slate-800 p-2 rounded-xl border border-slate-700 flex flex-col justify-center">
                <p class="text-yellow-500 text-[9px] font-black uppercase px-2">Algorithm</p>
                <div id="guidance-text" class="text-slate-400 text-[10px] italic leading-tight px-2">Waiting...</div>
            </div>
        </div>

        <div class="grid grid-cols-3 gap-2">
            <button onclick="toggleStart()" id="btn-start" class="bg-emerald-600 py-2.5 rounded-lg font-black text-[10px] uppercase shadow-lg">Start</button>
            <button onclick="setRhythm('Shockable')" class="bg-rose-700 py-2.5 rounded-lg font-black text-[10px] uppercase shadow-lg">Shockable</button>
            <button onclick="setRhythm('Non-Shockable')" class="bg-blue-700 py-2.5 rounded-lg font-black text-[10px] uppercase shadow-lg">Non-Shock</button>
        </div>
    </div>

    <div class="fixed-panel p-2.5 bg-white border-b border-slate-200">
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordAction('💓 CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700">START CPR</button>
            <button onclick="recordAction('💉 Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700">IV/IO</button>
            <button onclick="recordAction('💧 IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700">FLUID</button>
        </div>
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordDefib()" class="drug-btn border-2 border-rose-500 text-rose-600 bg-rose-50 italic">DEFIB 200J</button>
            <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md">EPINEPHRINE</button>
            <button onclick="recordAction('💊 Amio 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600 bg-white">AMIO 300</button>
        </div>
        <div class="grid grid-cols-2 gap-2">
            <button onclick="recordAction('💊 Amio 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700 uppercase">Amio 150</button>
            <button onclick="recordAction('🌬️ Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-200 text-orange-700 italic uppercase">Adv. Airway</button>
        </div>
    </div>

    <div class="log-container no-scrollbar">
        <table class="w-full text-left border-separate border-spacing-y-1 px-3 py-2">
            <thead class="sticky top-0 bg-slate-50 z-10 shadow-sm">
                <tr class="text-[9px] text-slate-400 font-black uppercase">
                    <th class="py-2 w-16 text-center">Time</th>
                    <th class="py-2 px-2">Action Recorded</th>
                </tr>
            </thead>
            <tbody id="log-body" class="text-[11px] font-bold text-slate-700"></tbody>
        </table>
    </div>

    <div class="fixed-panel bg-white border-t p-3 flex items-center gap-3">
        <button onclick="recordAction('🏁 ROSC Achieved')" class="flex-[2] bg-emerald-500 text-white py-4 rounded-xl font-black text-xs uppercase shadow-lg">
            ROSC Achieved
        </button>

        <div class="flex gap-1.5 flex-1">
            <div class="bg-rose-600 text-white flex-1 py-1 rounded-xl text-center shadow-md">
                <p class="text-[8px] font-black uppercase leading-none mt-1">Shock</p>
                <p id="dash-shock" class="text-xl font-black leading-none">0</p>
            </div>
            <div class="bg-blue-600 text-white flex-1 py-1 rounded-xl text-center shadow-md">
                <p class="text-[8px] font-black uppercase leading-none mt-1">Epi</p>
                <p id="dash-epi" class="text-xl font-black leading-none">0</p>
            </div>
        </div>

        <button onclick="closeCase()" class="flex-1 bg-slate-800 text-white py-4 rounded-xl font-black text-[10px] uppercase">
            Close
        </button>
    </div>

    <script>
        let totalSec = 0, cprSec = 120, isRunning = false, mainInterval = null;
        let defibCount = 0, epiCount = 0;

        function formatTime(s) {
            const m = Math.floor(s / 60).toString().padStart(2, '0');
            const sec = (s % 60).toString().padStart(2, '0');
            return `${m}:${sec}`;
        }

        function toggleStart() {
            const btn = document.getElementById('btn-start');
            if (!isRunning) {
                isRunning = true; btn.innerText = 'PAUSE'; btn.classList.replace('bg-emerald-600', 'bg-amber-600');
                if (totalSec === 0) recordAction('🚀 Code Blue Activated');
                mainInterval = setInterval(() => {
                    totalSec++; cprSec--;
                    document.getElementById('total-timer').innerText = formatTime(totalSec);
                    document.getElementById('cpr-timer').innerText = formatTime(cprSec);
                    if (cprSec <= 0) { cprSec = 120; recordAction('⚠️ Rhythm Check Due!'); }
                }, 1000);
            } else {
                isRunning = false; btn.innerText = 'RESUME'; btn.classList.replace('bg-amber-600', 'bg-emerald-600');
                clearInterval(mainInterval);
            }
        }

        function setRhythm(type) {
            document.getElementById('guidance-text').innerHTML = type === 'Shockable' ? '<b class="text-rose-500">Shock 200J</b> -> CPR' : '<b class="text-blue-500">Give Epi</b> -> CPR';
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
            recordAction(`💉 Epinephrine #${epiCount}`);
        }

        function recordAction(msg) {
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-4 border-slate-300 shadow-sm transition-all";
            row.innerHTML = `<td class="py-2.5 text-center bg-slate-50 font-mono text-[10px] font-black">${formatTime(totalSec)}</td><td class="py-2.5 px-3">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
            // สั่งให้เลื่อนขึ้นไปดูรายการล่าสุดที่บนสุดของช่อง Log
            document.querySelector('.log-container').scrollTop = 0;
        }

        function closeCase() {
            if(confirm("ต้องการปิดเคสใช่หรือไม่?")) {
                isRunning = false; clearInterval(mainInterval);
                alert(`Case Summary:\nTotal Time: ${formatTime(totalSec)}\nShocks: ${defibCount}\nEpi: ${epiCount}`);
            }
        }
    </script>
</body>
</html>
