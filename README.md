<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Laboratório Virtual: Simulação de Matéria Escura</title>
    <style>
        :root {
            --bg-color: #0f172a;
            --panel-color: #1e293b;
            --accent-wimp: #ef4444;
            --accent-xenon: #38bdf8;
            --text-color: #f8fafc;
        }

        body {
            margin: 0;
            padding: 20px;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        h1 {
            margin-bottom: 5px;
            font-size: 24px;
            text-align: center;
        }

        p.subtitle {
            color: #94a3b8;
            margin-top: 0;
            margin-bottom: 20px;
            font-size: 14px;
        }

        .container {
            display: flex;
            gap: 20px;
            max-width: 1100px;
            width: 100%;
            flex-wrap: wrap;
            justify-content: center;
        }

        .panel {
            background-color: var(--panel-color);
            border-radius: 12px;
            padding: 15px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }

        .panel-title {
            font-size: 14px;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 0.05em;
            margin-bottom: 10px;
            color: #cbd5e1;
            border-bottom: 1px solid #334155;
            padding-bottom: 5px;
        }

        canvas {
            background-color: #020617;
            border-radius: 8px;
            display: block;
        }

        .controls {
            margin-top: 15px;
            display: flex;
            flex-direction: column;
            gap: 12px;
            width: 100%;
        }

        .btn-group {
            display: flex;
            gap: 10px;
        }

        button {
            background-color: #2563eb;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: 600;
            transition: background 0.2s;
            flex: 1;
        }

        button:hover {
            background-color: #1d4ed8;
        }

        button#trigger-wimp {
            background-color: var(--accent-wimp);
        }

        button#trigger-wimp:hover {
            background-color: #dc2626;
        }

        .slider-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
            font-size: 12px;
            color: #94a3b8;
        }

        .slider-row {
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 10px;
        }

        input[type="range"] {
            flex: 1;
            accent-color: var(--accent-xenon);
        }

        .stats {
            margin-top: 10px;
            font-size: 13px;
            color: #94a3b8;
            display: flex;
            justify-content: space-between;
        }
    </style>
</head>
<body>

    <h1>Simulador de Laboratório: Criação e Detecção de Matéria Escura</h1>
    <p class="subtitle">Interações de WIMPs vs. Ruído de Radiação em Câmaras de Xenônio Líquido</p>

    <div class="container">
        <!-- Painel Esquerdo: O Detector Físico -->
        <div class="panel">
            <div class="panel-title">🔬 Câmara de Xenônio Líquido (Tempo Real)</div>
            <canvas id="detectorCanvas" width="450" height="400"></canvas>
            <div class="controls">
                <div class="btn-group">
                    <button id="trigger-noise">Injetar Ruído</button>
                    <button id="trigger-wimp">Disparar Matéria Escura</button>
                </div>
                <div class="slider-group">
                    <div class="slider-row">
                        <label>Massa do WIMP:</label>
                        <input type="range" id="mass-slider" min="10" max="200" value="100">
                        <span id="mass-val">100 GeV</span>
                    </div>
                </div>
            </div>
        </div>

        <!-- Painel Direito: O Gráfico de Análise Estatística -->
        <div class="panel">
            <div class="panel-title">📊 Análise de Dados: log10(S2/S1) vs Energia</div>
            <canvas id="chartCanvas" width="450" height="400"></canvas>
            <div class="stats">
                <span>Ruídos Filtrados: <strong id="count-noise" style="color:#94a3b8">0</strong></span>
                <span>WIMPs Confirmados: <strong id="count-wimp" style="color:var(--accent-wimp)">0</strong></span>
            </div>
        </div>
    </div>

    <script>
        // --- CONFIGURAÇÕES DO SISTEMA ---
        const dCanvas = document.getElementById('detectorCanvas');
        const ctxD = dCanvas.getContext('2d');
        const cCanvas = document.getElementById('chartCanvas');
        const ctxC = cCanvas.getContext('2d');

        const massSlider = document.getElementById('mass-slider');
        const massVal = document.getElementById('mass-val');

        let atoms = [];
        let particles = [];
        let plotPoints = [];
        let counters = { noise: 0, wimp: 0 };
        let currentWimpMass = 100;

        // Atualizar valor da massa no painel
        massSlider.addEventListener('input', (e) => {
            currentWimpMass = parseInt(e.target.value);
            massVal.innerText = `${currentWimpMass} GeV`;
        });

        // Inicializar átomos de Xenônio na câmara
        for (let i = 0; i < 25; i++) {
            atoms.push({
                x: Math.random() * (dCanvas.width - 40) + 20,
                y: Math.random() * (dCanvas.height - 40) + 20,
                vx: (Math.random() - 0.5) * 0.8,
                vy: (Math.random() - 0.5) * 0.8,
                radius: 10,
                pulse: 0
            });
        }

        // --- GATILHOS DE EVENTOS ---
        document.getElementById('trigger-noise').addEventListener('click', () => {
            particles.push({
                x: 0, y: Math.random() * (dCanvas.height - 40) + 20,
                vx: Math.random() * 3 + 3, vy: (Math.random() - 0.5) * 1,
                type: 'noise', radius: 4, color: '#f59e0b', mass: 1
            });
        });

        document.getElementById('trigger-wimp').addEventListener('click', () => {
            particles.push({
                x: 0, y: Math.random() * (dCanvas.height - 40) + 20,
                vx: Math.random() * 4 + 4, vy: (Math.random() - 0.5) * 0.5,
                type: 'wimp', radius: 5, color: 'rgba(239, 68, 68, 0.2)', mass: currentWimpMass
            });
        });

        // --- DETECÇÃO E PLOTAGEM ---
        function registerDetection(type, mass) {
            let energia;
            let ratio;
            
            if (type === 'noise') {
                energia = Math.random() * 15 + 5; // Ruído gera baixa energia de recuo nuclear
                ratio = 2.3 + (Math.random() * 0.5); // Sinal S2 alto (Eletrônico)
                counters.noise++;
                document.getElementById('count-noise').innerText = counters.noise;
            } else {
                // Física real: massa maior do WIMP transfere mais energia cinética (keV) ao núcleo
                energia = (mass / 200) * 35 + Math.random() * 10; 
                ratio = 1.1 + (Math.random() * 0.5); // Sinal S2 baixo (Nuclear)
                counters.wimp++;
                document.getElementById('count-wimp').innerText = counters.wimp;
            }

            // Limitar energia no gráfico
            if (energia > 50) energia = 50;

            // Mapeia dados para pixels
            let px = 50 + (energia / 50) * (cCanvas.width - 80);
            let py = (cCanvas.height - 50) - ((ratio - 0.5) / 2.5) * (cCanvas.height - 80);
            
            plotPoints.push({ x: px, y: py, type: type });
        }

        // --- LOOP PRINCIPAL ---
        function update() {
            // 1. Atualizar Fundo do Detector
            ctxD.clearRect(0, 0, dCanvas.width, dCanvas.height);

            // Desenhar Átomos de Xenônio
            atoms.forEach(atom => {
                atom.x += atom.vx;
                atom.y += atom.vy;

                if (atom.x < atom.radius || atom.x > dCanvas.width - atom.radius) atom.vx *= -1;
                if (atom.y < atom.radius || atom.y > dCanvas.height - atom.radius) atom.vy *= -1;

                // Animação de Cintilação (S1)
                if (atom.pulse > 0) {
                    ctxD.beginPath();
                    ctxD.arc(atom.x, atom.y, atom.radius + atom.pulse, 0, Math.PI * 2);
                    ctxD.fillStyle = `rgba(56, 189, 248, ${0.4 - atom.pulse/30})`;
                    ctxD.fill();
                    atom.pulse += 1.5;
                    if (atom.pulse > 25) atom.pulse = 0;
                }

                ctxD.beginPath();
                ctxD.arc(atom.x, atom.y, atom.radius, 0, Math.PI * 2);
                ctxD.fillStyle = '#0284c7';
                ctxD.fill();
                ctxD.strokeStyle = '#38bdf8';
                ctxD.stroke();
            });

            // Gerenciar Partículas
            for (let i = particles.length - 1; i >= 0; i--) {
                let p = particles[i];
                p.x += p.vx;
                p.y += p.vy;

                ctxD.beginPath();
                ctxD.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
                ctxD.fillStyle = p.color;
                ctxD.fill();

                // Checar colisões
                atoms.forEach(atom => {
                    let dx = p.x - atom.x;
                    let dy = p.y - atom.y;
