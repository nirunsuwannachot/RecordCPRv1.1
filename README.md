# RecordCPRv1.1
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS Pro 2025 - Fixed Layout</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        :root { --app-height: 100dvh; }
        body {
            height: var(--app-height);
            margin: 0;
            overflow: hidden; /* ล็อคหน้าจอหลักไม่ให้เลื่อน */
            display: flex;
            flex-direction: column;
            background-color: #0f172a;
            font-family: sans-serif;
        }

        /* ส่วนหัวและปุ่มกด: ล็อคขนาดห้ามยืดหด */
        .fixed-panel { flex-shrink: 0; }

        /* ส่วน Action Log: กินพื้นที่ที่เหลือและเลื่อนได้ภายในตัว */
        .log-container {
            flex-grow: 1;
            overflow-y: auto;
            -webkit-overflow-scrolling: touch;
            background-color: #f1f5f9;
            /* ล็อคความสูงขั้นต่ำเพื่อให้ UI นิ่ง */
            min-height: 200px; 
        }

        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            height: 48px; font-size: 11px; font-weight: 800;
            border-radius: 10px; transition: all 0.1s;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
        }
        .drug-btn:active { transform: scale(0.95); opacity: 0.8; }

        .no-scrollbar::-webkit-scrollbar { display: none; }
        
        /* ตกแต่งแถวของ Log ให้ความสูงคงที่ */
        #log-body tr { height: 45px; }
    </style>
</head>
<body>

    <div class="fixed-panel p-4 bg-slate-900 text-white shadow-xl">
        <div class="flex justify-between items-center mb-3">
            <div>
                <h1 class="text-xl font-black text-red-500 italic leading-none">ACLS 2025</h1>
                <p class="text-[9px] text-slate-500 font-bold mt-1 uppercase">NIRUN SUWANNACHOT</p>
            </div>
            <div class="text-right">
                <p class="text-[9px] text-slate-500 font-bold uppercase">Total Time</p>
                <p id="total-timer" class="text-2xl font-mono font-bold text-emerald-400 leading-none">00:00</p>
            </div>
        </div>
        <div class="grid grid-cols-2 gap-2 mb-3">
            <div class="bg-slate-800 p-2 rounded-xl border-2 border-blue-500 text-center">
                <p class="text-[9px] text-blue-400 font-bold">RHYTHM CHECK IN</p>
                <p id="cpr-timer" class="text-3xl font-mono font-black">02:00</p>
            </div>
            <div class="bg-slate-800 p-2 rounded-xl border border-slate-700 text-[10px] flex flex-col justify-center px-3">
                <p class="text-yellow-500 font-black uppercase">Algorithm Guidance</p>
                <div id="guidance-text" class="text-slate-400 italic leading-tight mt-1">Waiting...</div>
            </div>
        </div>
        <div class="grid grid-cols-3 gap-2">
            <button onclick="toggleStart()" id="btn-start" class="bg-emerald-600 py-3 rounded-lg font-black text-xs uppercase shadow-lg">Start Case</button>
            <button onclick="setRhythm('Shockable')" class="bg-rose-700 py-3 rounded-lg font-black text-xs uppercase shadow-lg">Shockable</button>
            <button onclick="setRhythm('Non-Shockable')" class="bg-blue-700 py-3 rounded-lg font-black text-[9px] uppercase shadow-lg">Non-Shockable</button>
        </div>
    </div>

    <div class="fixed-panel p-3 bg-white border-b border-slate-200">
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordAction('💓 CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700">START CPR</button>
            <button onclick="recordAction('💉 Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700">IV/IO</button>
            <button onclick="recordAction('💧 IV Fluid Given')" class="drug-btn border-2 border-blue-400 text-blue-700">FLUID</button>
        </div>
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordDefib()" class="drug-btn border-2 border-rose-500 text-rose-600 bg-rose-50 italic">DEFIB 200J</button>
            <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md">EPINEPHRINE</button>
            <button onclick="recordAction('💊 Amio 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600 bg-white">AMIO 300</button>
        </div>
        <div class="grid grid-cols-2 gap-2">
            <button onclick="recordAction('💊 Amio 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700">AMIO 150</button>
            <button onclick="recordAction('🌬️ Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-300 text-orange-700 italic uppercase">Adv. Airway</button>
        </div>
    </div>

    <div class="log-container no-scrollbar">
        <table class="w-full text-left border-separate border-spacing-y-1 px-3">
            <thead class="sticky top-0 bg-slate-50 z-10 shadow-sm">
                <tr class="text-[9px] text-slate-400 font-black uppercase">
                    <th class="py-2 w-16 text-center">Time</th>
                    <th class="py-2 px-2">Action Recorded</th>
                </tr>
            </thead>
            <tbody id="log-body" class="text-[11px] font-bold text-slate-700">
                </tbody>
        </table>
    </div>

    <div class="fixed-panel bg-white border-t p-3 flex items-center gap-3">
        <button onclick="recordAction('🏁 ROSC Achieved')" class="flex-[2] bg-emerald-500 text-white py-4 rounded-xl font-black text-xs uppercase shadow-lg active:scale-95 transition-transform">
            ROSC Achieved
        </button>

        <div class="flex gap-1.5 flex-1">
            <div class="bg-rose-600 text-white flex-1 py-1 rounded-xl shadow-md text-center">
                <p class="text-[8px] font-black uppercase leading-none">Shock</p>
                <p id="dash-shock" class="text-xl font-black leading-none">0</p>
            </div>
            <div class="bg-blue-600 text-white flex-1 py-1 rounded-xl shadow-md text-center">
                <p class="text-[8px] font-black uppercase leading-none">Epi</p>
                <p id="dash-epi" class="text-xl font-black leading-none">0</p>
            </div>
        </div>

        <button onclick="closeCase()" class="flex-1 bg-slate-800 text-white py-4 rounded-xl font-black text-[10px] uppercase active:scale-95 transition-transform">
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
                isRunning = true; 
                btn.innerText = 'PAUSE'; 
                btn.classList.replace('bg-emerald-600', 'bg-amber-600');
                if (totalSec === 0) recordAction('🚀 Code Blue Activated');
                
                mainInterval = setInterval(() => {
                    totalSec++; 
                    cprSec--;
                    document.getElementById('total-timer').innerText = formatTime(totalSec);
                    document.getElementById('cpr-timer').innerText = formatTime(cprSec);
                    
                    if (cprSec <= 0) { 
                        cprSec = 120; 
                        recordAction('⚠️ Check Rhythm & Pulse!');
                        // Optional: Add alert sound here
                    }
                }, 1000);
            } else {
                isRunning = false; 
                btn.innerText = 'RESUME'; 
                btn.classList.replace('bg-amber-600', 'bg-emerald-600');
                clearInterval(mainInterval);
            }
        }

        function setRhythm(type) {
            const guidance = document.getElementById('guidance-text');
            if(type === 'Shockable') {
                guidance.innerHTML = '<b class="text-rose-500">⚡ SHOCKABLE:</b> Give Shock 200J -> Start CPR Immediately';
            } else {
                guidance.innerHTML = '<b class="text-blue-500">💉 NON-SHOCKABLE:</b> Give Epi ASAP -> Start CPR Immediately';
            }
            recordAction(`🔍 Rhythm: ${type}`);
            cprSec = 120; // Reset CPR timer on rhythm check
        }

        function recordDefib() {
            defibCount++; 
            document.getElementById('dash-shock').innerText = defibCount;
            recordAction(`⚡ Defibrillation #${defibCount} (200J)`);
            cprSec = 120;
        }

        function recordEpi() {
            epiCount++; 
            document.getElementById('dash-epi').innerText = epiCount;
            recordAction(`💉 Epinephrine #${epiCount} (1 mg)`);
        }

        function recordAction(msg) {
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            
            // เพิ่มสีขอบตามประเภทข้อความ
            let borderColor = 'border-slate-300';
            if (msg.includes('⚡')) borderColor = 'border-rose-500 bg-rose-50';
            if (msg.includes('💉')) borderColor = 'border-blue-500 bg-blue-50';
            if (msg.includes('🏁')) borderColor = 'border-emerald-500 bg-emerald-50';

            row.className = `bg-white border-l-4 ${borderColor} shadow-sm mb-1`;
            row.innerHTML = `
                <td class="py-2 text-center bg-slate-50 font-mono text-[10px] font-black border-r border-slate-100">${formatTime(totalSec)}</td>
                <td class="py-2 px-3">${msg}</td>
            `;
            
            // แทรกรายการใหม่ไว้บนสุด
            body.insertBefore(row, body.firstChild);
            
            // เลื่อน Log container ไปที่บนสุดเสมอเพื่อดูรายการล่าสุด
            document.querySelector('.log-container').scrollTop = 0;
        }

        function closeCase() {
            if(confirm("ต้องการจบ Case และสรุปข้อมูลใช่หรือไม่?")) {
                isRunning = false; 
                clearInterval(mainInterval);
                alert(`Case Summary:\nTotal Time: ${formatTime(totalSec)}\nTotal Shocks: ${defibCount}\nTotal Epi: ${epiCount}`);
            }
        }
    </script>
</body>
</html>
