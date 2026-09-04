<!DOCTYPE html>
<html lang="hu">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cimbi / Undergrund System</title>
    <style>
        :root {
            --bg-color: #0f111a;
            --card-bg: #1e2230;
            --accent: #00ffcc;
            --text: #e0e6ed;
            --warn: #ff4757;
        }
        body {
            background-color: var(--bg-color);
            color: var(--text);
            font-family: monospace;
            margin: 0;
            padding: 15px;
        }
        h1, h2 { color: var(--accent); }
        .card {
            background: var(--card-bg);
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 15px;
            border: 1px solid #2a3142;
        }
        input, button, textarea {
            width: 100%;
            padding: 10px;
            margin-top: 5px;
            background: #0b0d13;
            border: 1px solid var(--accent);
            color: var(--text);
            border-radius: 4px;
            box-sizing: border-box;
        }
        button {
            background: var(--accent);
            color: #000;
            font-weight: bold;
            cursor: pointer;
            margin-top: 10px;
        }
        .grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            text-align: center;
        }
        .grid div {
            background: #141722;
            padding: 10px;
            border-radius: 4px;
        }
    </style>
</head>
<body>

    <h1>CIMBI // UNDERGRUND SYSTEM v1.0</h1>

    <!-- API BEÁLLÍTÁS -->
    <div class="card">
        <h2>1. Rendszer Csatlakozás</h2>
        <label>Google AI Studio API Kulcs:</label>
        <input type="password" id="apiKey" placeholder="AI Studio API Key beillesztése">
        <button onclick="saveKey()">Kulcs Mentése</button>
    </div>

    <!-- PÉNZÜGYI AUTOMATA (60 / 25 / 15) -->
    <div class="card">
        <h2>2. Revolut Büdzsé Kalkulátor</h2>
        <label>Bejövő Összeg (€):</label>
        <input type="number" id="incomeInput" placeholder="Pl. 50">
        <button onclick="calculateBudget()">Kiszámolás & Osztás</button>
        
        <div style="margin-top: 15px;" class="grid">
            <div>
                <small>Kiszabadulás (60%)</small>
                <h3 id="p60">0 €</h3>
            </div>
            <div>
                <small>Himiway (25%)</small>
                <h3 id="p25">0 €</h3>
            </div>
            <div>
                <small>Projekt/Cimbi (15%)</small>
                <h3 id="p15">0 €</h3>
            </div>
        </div>
    </div>

    <!-- TERMINÁL / KOMMUNIKÁCIÓ -->
    <div class="card">
        <h2>3. Direkt Parancssor & Prompt</h2>
        <textarea id="promptInput" rows="4" placeholder="Írd be a feladatot vagy a műszaki kérdést..."></textarea>
        <button onclick="sendToCimbi()">Küldés Cimbinek</button>
        
        <h3>Válasz / Státusz:</h3>
        <div id="output" style="white-space: pre-wrap; background: #0b0d13; padding: 10px; border-radius: 4px; min-height: 80px;">Rendszer készenlétben...</div>
    </div>

    <script>
        function saveKey() {
            const key = document.getElementById('apiKey').value;
            localStorage.setItem('cimbi_api_key', key);
            alert('API kulcs elmentve a helyi tárolóba!');
        }

        function calculateBudget() {
            const val = parseFloat(document.getElementById('incomeInput').value) || 0;
            document.getElementById('p60').innerText = (val * 0.60).toFixed(2) + ' €';
            document.getElementById('p25').innerText = (val * 0.25).toFixed(2) + ' €';
            document.getElementById('p15').innerText = (val * 0.15).toFixed(2) + ' €';
        }

        async function sendToCimbi() {
            const key = localStorage.getItem('cimbi_api_key') || document.getElementById('apiKey').value;
            const prompt = document.getElementById('promptInput').value;
            const output = document.getElementById('output');

            if (!key) {
                output.innerText = 'HIBA: Hiányzik az API kulcs! Regisztrálj egyet az aistudio.google.com-on.';
                return;
            }

            output.innerText = 'Gondolkodás és adatfeldolgozás...';

            try {
                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${key}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: "Te vagy Cimbi, egy közvetlen, műszaki és logisztikai partner. A válaszod legyen lényegretörő: " + prompt }] }]
                    })
                });

                const data = await response.json();
                if (data.candidates && data.candidates[0].content.parts[0].text) {
                    output.innerText = data.candidates[0].content.parts[0].text;
                } else {
                    output.innerText = 'Rendszerhiba a válasz feldolgozásakor.';
                }
            } catch (err) {
                output.innerText = 'Hálózati hiány vagy API kapcsolódási hiba: ' + err.message;
            }
        }
    </script>
</body>
</html>
