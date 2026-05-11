<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RecordCPR V1 - UI Update</title>
    <style>
        :root {
            --bg-color: #0b1120;
            --card-bg: #1e293b;
            --text-main: #ffffff;
            --accent-red: #ef4444;
            --accent-blue: #3b82f6;
            --accent-green: #10b981;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-main);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }

        /* ปรับ Container ให้กว้างขึ้นแต่ไม่เต็มจอ */
        .app-container {
            width: 100%;
            max-width: 650px; /* ขยายจากเดิม เพื่อให้ดูโปร่ง */
            background: #111827;
            border-radius: 16px;
            padding: 24px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.5);
        }

        header {
            text-align: center;
            margin-bottom: 20px;
            border-bottom: 1px solid #374151;
            padding-bottom: 10px;
        }

        .timer-section {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 20px;
        }

        .card {
            background: var(--card-bg);
            padding: 15px;
            border-radius: 12px;
            text-align: center;
        }

        .timer-display {
            font-size: 2.5rem;
            font-weight: bold;
            font-family: monospace;
        }

        .btn-group-main {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            margin-bottom: 20px;
        }

        /* แผงปุ่มบันทึกเหตุการณ์ - ปรับให้ดูเป็นระเบียบขึ้น */
        .action-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 12px;
            background: #f8fafc; /* พื้นหลังสว่างตามภาพต้นฉบับ */
            padding: 20px;
            border-radius: 12px;
            margin-bottom: 20px;
        }

        button {
            border: none;
            border-radius: 8px;
            padding: 12px 5px;
            font-weight: bold;
            cursor: pointer;
            transition: opacity 0.2s;
            text-transform: uppercase;
            font-size: 0.85rem;
        }

        button:active { opacity: 0.7; }

        /* สีปุ่มต่างๆ */
        .btn-green { background-color: var(--accent-green); color: white; }
        .btn-red { background-color: var(--accent-red); color: white; }
        .btn-blue { background-color: var(--accent-blue); color: white; }
        .btn-outline { background: white; border: 2px solid #ddd; color: #333; }
        .btn-purple { border: 2px solid #8b5cf6; color: #8b5cf6; background: white; }
        .btn-orange { border: 2px solid #f59e0b; color: #f59e0b; background: white; }

        /* ตารางบันทึก */
        .log-container {
            background: white;
            color: #333;
            border-radius: 8px;
            max-height: 250px;
            overflow-y: auto;
        }

        table {
            width: 100%;
            border-collapse: collapse;
        }

        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #eee;
        }

        .footer-controls {
            margin-top: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .rosc-btn {
            background: var(--accent-green);
            color: white;
            padding: 15px 30px;
            font-size: 1.2rem;
            flex-grow: 1;
            margin-right: 15px;
        }

        .stats-badge {
            display: flex;
            gap: 10px;
        }

        .badge {
            padding: 10px;
            border-radius: 50px;
            min-width: 40px;
            text-align: center;
            color: white;
            font-size: 0.8rem;
        }
    </style>
</head>
<body>

<div class="app-container">
    <header>
        <h2 style="color: var(--accent-red); margin: 0;">ACLS 2025 <span style="color: var(--accent-green); margin-left: 10px;">00:09</span></h2>
    </header>

    <div class="timer-section">
        <div class="card" style="border: 2px solid var(--accent-blue);">
            <div style="font-size: 0.7rem; color: var(--accent-blue);">RHYTHM CHECK</div>
            <div class="timer-display">01:58</div>
        </div>
        <div class="card">
            <div style="font-size: 0.7rem; color: #f59e0b;">GUIDANCE</div>
            <div style="margin-top: 10px; font-weight: bold;">
                <span style="color: var(--accent-red);">SHOCK 200J</span> -> CPR
            </div>
        </div>
    </div>

    <div class="btn-group-main">
        <button class="btn-green">Resume</button>
        <button class="btn-red">Shockable</button>
        <button class="btn-blue">Non-Shock</button>
    </div>

    <div class="action-grid">
        <button class="btn-outline" style="color: var(--accent-green);">Start CPR</button>
        <button class="btn-outline" style="color: var(--accent-blue);">IV/IO</button>
        <button class="btn-outline" style="color: var(--accent-blue);">Fluid</button>
        <button class="btn-outline" style="color: var(--accent-red); border-color: var(--accent-red);">Defib 200J</button>
        <button class="btn-blue">Epinephrine</button>
        <button class="btn-purple">Amio 300</button>
        <button class="btn-outline">Amio 150</button>
        <button class="btn-orange">Adv. Airway</button>
    </div>

    <div class="log-container">
        <table>
            <thead>
                <tr style="background: #f1f5f9;">
                    <th>TIME</th>
                    <th>ACTION RECORDED</th>
                </tr>
            </thead>
            <tbody>
                <tr><td>00:07</td><td>Rhythm: Shockable</td></tr>
                <tr><td>00:07</td><td>Rhythm: Non-Shockable</td></tr>
                <tr><td>00:06</td><td>Access IV/IO</td></tr>
            </tbody>
        </table>
    </div>

    <div class="footer-controls">
        <button class="rosc-btn">ROSC ACHIEVED</button>
        <div class="stats-badge">
            <div class="badge" style="background: var(--accent-red);">SHOCK<br>2</div>
            <div class="badge" style="background: var(--accent-blue);">EPI<br>1</div>
        </div>
        <button style="background: #334155; color: white; margin-left: 10px;">Close Case</button>
    </div>
</div>

</body>
</html>
