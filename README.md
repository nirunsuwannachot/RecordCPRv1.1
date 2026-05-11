<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS Pro 2025</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* ล็อคความสูงให้เต็มหน้าจอจริง (Dynamic Viewport Height) */
        :root { --app-height: 100dvh; }
        
        body {
            height: var(--app-height);
            margin: 0;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            background-color: #0f172a;
            /* เพิ่มพื้นที่ปลอดภัยสำหรับ iPad/iPhone */
            padding-top: env(safe-area-inset-top);
            padding-bottom: env(safe-area-inset-bottom);
            padding-left: env(safe-area-inset-left);
            padding-right: env(safe-area-inset-right);
        }

        /* ส่วนหัวและปุ่ม ให้ขยายกว้างเต็ม 100% */
        .fixed-panel { 
            flex-shrink: 0; 
            width: 100%;
        }

        /* ส่วน Log ให้ยืดขยายเต็มพื้นที่ที่เหลือ */
        .log-container {
            flex-grow: 1;
            width: 100%;
            overflow-y: auto;
            -webkit-overflow-scrolling: touch;
            background-color: #f1f5f9;
        }

        /* ปรับปุ่มให้ขนาดใหญ่ขึ้นเพื่อให้กดง่ายในโหมดเต็มหน้าจอ */
        .drug-btn {
            display: flex; align-items: center; justify-content: center;
            height: 55px; /* เพิ่มความสูงปุ่ม */
            font-size: 12px; font-weight: 800;
            border-radius: 12px;
            width: 100%;
        }

        #summary-screen {
            display: none;
            position: fixed;
            inset: 0;
            background: white;
            z-index: 100;
            padding: 20px;
            padding-top: env(safe-area-inset-top);
            overflow-y: auto;
        }
        
        .no-scrollbar::-webkit-scrollbar { display: none; }
    </style>
</head>
<body>

    <div id="main-app" class="flex flex-col h-full w-full">
        
        <div class="fixed-panel p-4 bg-slate-900 text-white shadow-2xl">
            <div class="flex justify-between items-center mb-4">
                <div>
                    <h1 class="text-2xl font-black text-red-500 italic leading-none">ACLS 2025</h1>
                    <p class="text-[10px] text-slate-500 font-bold mt-1 uppercase tracking-widest">NIRUN SUWANNACHOT</p>
                </div>
                <div class="text-right">
                    <p class="text-[10px] text-slate-500 font-bold uppercase">Total Time</p>
                    <p id="total-timer" class="text-3xl font-mono font-black text-emerald-400">00:00</p>
                </div>
            </div>
            
            <div class="grid grid-cols-2 gap-4 mb-4">
                <div class="bg-slate-800 p-4 rounded-2xl border-2 border-blue-500 text-center shadow-inner">
                    <p class="text-xs text-blue-400 font-bold uppercase mb-1">Rhythm Check</p>
                    <p id="cpr-timer" class="text-5xl font-mono font-black">02:00</p>
                </div>
                <div class="bg-slate-800 p-4 rounded-2xl border border-slate-700 flex flex-col justify-center px-4">
                    <p class="text-yellow-500 text-xs font-bold uppercase mb-1">Guidance</p>
                    <div id="guidance-text" class="text-slate-300 text-sm italic font-medium leading-tight">Waiting...</div>
                </div>
            </div>

            <div class="grid grid-cols-3 gap-3">
                <button onclick="toggleStart()" id="btn-start" class="bg-emerald-600 py-4 rounded-xl font-black text-sm uppercase shadow-lg active:scale-95 transition-transform">Start</button>
                <button onclick="setRhythm('Shockable')" class="bg-rose-700 py-4 rounded-xl font-black text-sm uppercase shadow-lg active:scale-95 transition-transform">Shockable</button>
                <button onclick="setRhythm('Non-Shockable')" class="bg-blue-700 py-4 rounded-xl font-black text-sm uppercase shadow-lg active:scale-95 transition-transform">Non-Shock</button>
            </div>
        </div>

        <div class="fixed-panel p-4 bg-white border-b border-slate-200">
            <div class="grid grid-cols-3 gap-3 mb-3">
                <button onclick="recordAction('💓 CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700 font-black">START CPR</button>
                <button onclick="recordAction('💉 Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700 font-black">IV/IO</button>
                <button onclick="recordAction('💧 IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700 font-black">FLUID</button>
            </div>
            <div class="grid grid-cols-3 gap-3 mb-3">
                <button onclick="recordDefib()" class="drug-btn border-2 border-rose-500 text-rose-600 bg-rose-50 italic font-black">DEFIB 200J</button>
                <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md font-black">EPINEPHRINE</button>
                <button onclick="recordAction('💊 Amio 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600 bg-white font-black">AMIO 300</button>
            </div>
            <div class="grid grid-cols-2 gap-3">
                <button onclick="recordAction('💊 Amio 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700 uppercase font-black">Amio 150</button>
                <button onclick="recordAction('🌬️ Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-200 text-orange-700 italic uppercase font-black">Adv. Airway</button>
            </div>
        </div>

        <div class="log-container no-scrollbar">
            <table class="w-full text-left border-separate border-spacing-y-2 px-4 py-2">
                <thead class="sticky top-0 bg-slate-50 z-10">
                    <tr class="text-xs text-slate-400 font-black uppercase">
                        <th class="py-2 w-20 text-center border-r">Time</th>
                        <th class="py-2 px-4">Action Recorded</th>
                    </tr>
                </thead>
                <tbody id="log-body" class="text-sm font-bold text-slate-700"></tbody>
            </table>
        </div>

        <div class="fixed-panel bg-white border-t p-4 flex items-center gap-4">
            <button onclick="showSummary('ROSC')" class="flex-[2.5] bg-emerald-500 text-white py-6 rounded-2xl font-black text-lg uppercase shadow-xl active:scale-95 transition-transform">
                ROSC Achieved
            </button>
            <div class="flex gap-2 flex-1">
                <div class="bg-rose-600 text-white flex-1 py-2 rounded-xl text-center shadow-lg">
                    <p class="text-[10px] font-black uppercase leading-none mt-1">Shock</p>
                    <p id="dash-shock" class="text-2xl font-black">0</p>
                </div>
                <div class="bg-blue-600 text-white flex-1 py-2 rounded-xl text-center shadow-lg">
                    <p class="text-[10px] font-black uppercase leading-none mt-1">Epi</p>
                    <p id="dash-epi" class="text-2xl font-black">0</p>
                </div>
            </div>
            <button onclick="showSummary('CLOSE')" class="flex-1 bg-slate-800 text-white py-6 rounded-2xl font-black text-xs uppercase shadow-lg active:scale-95 transition-transform">
                Close
            </button>
        </div>
    </div>

    <div id="summary-screen">
        <div class="flex justify-between items-center border-b-4 border-slate-900 pb-4 mb-6">
            <h2 class="text-3xl font-black italic text-slate-900">CASE SUMMARY</h2>
            <button onclick="location.reload()" class="bg-rose-500 text-white px-6 py-2 rounded-full font-black text-sm uppercase">Reset</button>
        </div>
        <div class="grid grid-cols-2 gap-4 mb-8" id="summary-stats">
            </div>
        <div id="sum-timeline" class="space-y-3"></div>
        <button onclick="document.getElementById('summary-screen').style.display='none'" class="w-full mt-8 py-5 border-4 border-slate-900 rounded-2xl font-black text-slate-900 uppercase">Back to Records</button>
    </div>

    <script>
        // ... (Script ส่วนเดิมทั้งหมด แต่ปรับปรุงตัวแปรและการแสดงผลใน showSummary)
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
            const g = document.getElementById('guidance-text');
            g.innerHTML = type === 'Shockable' ? '<b class="text-rose-500 uppercase">Shock 200J</b> -> Resume CPR' : '<b class="text-blue-500 uppercase">Give Epi ASAP</b> -> Resume CPR';
            recordAction(`🔍 Rhythm: ${type}`);
            cprSec = 120;
        }

        function recordDefib() {
            defibCount++; document.getElementById('dash-shock').innerText = defibCount;
            recordAction(`⚡ Defib #${defibCount} (200J)`);
            cprSec = 120;
        }

        function recordEpi() {
            epiCount++; document.getElementById('dash-epi').innerText = epiCount;
            recordAction(`💉 Epinephrine #${epiCount}`);
        }

        function recordAction(msg) {
            const timeStr = formatTime(totalSec);
            timelineData.push({ time: timeStr, action: msg });
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-8 border-slate-300 shadow-sm rounded-lg overflow-hidden";
            row.innerHTML = `<td class="text-center font-mono py-4 bg-slate-50 border-r-2">${timeStr}</td><td class="px-6 font-bold">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
            document.querySelector('.log-container').scrollTop = 0;
        }

        function showSummary(outcome) {
            if (isRunning) toggleStart();
            const screen = document.getElementById('summary-screen');
            screen.style.display = 'block';

            const statsHtml = `
                <div class="bg-slate-100 p-6 rounded-3xl"><p class="text-slate-500 text-xs font-black uppercase">Duration</p><p class="text-3xl font-mono font-black">${formatTime(totalSec)}</p></div>
                <div class="bg-emerald-100 p-6 rounded-3xl"><p class="text-emerald-600 text-xs font-black uppercase">Outcome</p><p class="text-2xl font-black">${outcome}</p></div>
                <div class="bg-rose-100 p-6 rounded-3xl"><p class="text-rose-600 text-xs font-black uppercase">Total Shock</p><p class="text-4xl font-black">${defibCount}</p></div>
                <div class="bg-blue-100 p-6 rounded-3xl"><p class="text-blue-600 text-xs font-black uppercase">Total Epi</p><p class="text-4xl font-black">${epiCount}</p></div>
            `;
            document.getElementById('summary-stats').innerHTML = statsHtml;

            const timelineBox = document.getElementById('sum-timeline');
            timelineBox.innerHTML = '<h3 class="text-lg font-black mb-4 uppercase">Timeline Log</h3>';
            timelineData.forEach(item => {
                const div = document.createElement('div');
                div.className = "flex gap-4 items-center border-b border-slate-100 py-3";
                div.innerHTML = `<span class="font-mono text-slate-400 font-bold w-16">${item.time}</span><span class="text-slate-800 font-bold">${item.action}</span>`;
                timelineBox.appendChild(div);
            });
        }
    </script>
</body>
</html>
