<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Laboratório Virtual: Matéria Escura</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            font-family: sans-serif;
            background-color: #0f172a;
            color: #f8fafc;
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

        /* PAINEL ALTERADO PARA LARANJA */
        .panel {
            background-color: #ea580c;
            border-radius: 12px;
            padding: 15px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
        }

        .panel-title {
            font-size: 14px;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 0.05em;
            margin-bottom: 10px;
            color: #ffffff;
            border-bottom: 1px solid #ff7a33;
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

        /* BOTÕES ALTERADOS PARA PRETO E VERDE */
        button {
            background-color: #000000;
            color: #10b981;
            border: 2px solid #10b981;
            padding: 12px 20px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
            font-size: 13px;
            transition: all 0.2s;
            flex: 1;
        }

        button:hover {
            background-color: #10b981;
            color: #000000;
        }

        .slider-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
            font-size: 12px;
            color: #ffffff;
            font-weight: bold;
        }

        .slider-row {
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 10px;
        }

        input[type="range"] {
            flex: 1;
            accent-color: #000000;
        }

        .stats {
            margin-top: 10px;
            font-size: 13px;
            color: #ffffff;
            font-weight: bold;
            display: flex;
            justify-content: space-between;
        }
    </style>
</head>
<body>

    <h1>Simulador de Laboratório: Criação e Detecção de Matéria Escura</h1>
    <p class="subtitle">Interações de WIMPs vs. Ruído de Radiação em Câmaras de Xenônio Líquido</p>

    <div class="container">
        <!-- Painel Esquerdo -->
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

        <!-- Painel Direito -->
        <div class="panel">
            <div class="panel-title">📊 Análise de Dados: log10(S2/S1) vs Energia</div>
            <canvas id="chartCanvas" width="450" height="400"></canvas>
            <div class="stats">
                <span>Ruídos Filtrados: <strong id="count-noise" style="color:#000000">0</strong></span>
                <span>WIMPs Confirmados: <strong id="count-wimp" style="color:#ffffff">0</strong></span>
            </div>
        </div>
    </div>

    <script>
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

        massSlider.addEventListener('input', function(e) {
            currentWimpMass = parseInt(e.target.value);
            massVal.innerText = currentWimpMass + " GeV";
        });

        // Criar átomos estáveis na tela
        for (let i = 0; i < 20; i++) {
            atoms.push({
                x: Math.random() * 390 + 30,
                y: Math.random() * 340 + 30,
                vx: (Math.random() - 0.5) * 1.2,
                vy: (Math.random() - 0.5) * 1.2,
                radius: 12,
                flash: 0
            });
        }

        document.getElementById('trigger-noise').addEventListener('click', function() {
            particles.push({
                x: 10, y: Math.random() * 340 + 30,
                vx: 5, vy: (Math.random() - 0.5) * 2,
                type: 'noise', radius: 5, color: '#eab308', mass: 1
            });
        });

        document.getElementById('trigger-wimp').addEventListener('click', function() {
            particles.push({
                x: 10, y: Math.random() * 340 + 30,
                vx: 6, vy: 0,
                type: 'wimp', radius: 6, color: '#ef4444', mass: currentWimpMass
            });
        });

        function registerDetection(type, mass) {
            let energia, ratio;
            if (type === 'noise') {
                energia = Math.random() * 12 + 4; 
                ratio = 2.2 + (Math.random() * 0.4); 
                counters.noise++;
                document.getElementById('count-noise').innerText = counters.noise;
            } else {
                energia = (mass / 200) * 35 + Math.random() * 8; 
                ratio = 1.0 + (Math.random() * 0.4); 
                counters.wimp++;
                document.getElementById('count-wimp').innerText = counters.wimp;
            }

            let px = 60 + (energia / 50) * 340;
            let py = 340 - ((ratio - 0.5) / 2.5) * 290;
            plotPoints.push({ x: px, y: py, type: type });
        }

        function update() {
            // Desenhar Detector (Painel Esquerdo)
            ctxD.fillStyle = '#020617';
            ctxD.fillRect(0, 0, 450, 400);

            atoms.forEach(function(atom) {
                atom.x += atom.vx;
                atom.y += atom.vy;

                if (atom.x < atom.radius || atom.x > 450 - atom.radius) atom.vx *= -1;
                if (atom.y < atom.radius || atom.y > 400 - atom.radius) atom.vy *= -1;

                if (atom.flash > 0) {
                    ctxD.beginPath();
                    ctxD.arc(atom.x, atom.y, atom.radius + atom.flash, 0, Math.PI * 2);
                    ctxD.fillStyle = 'rgba(56, 189, 248, 0.3)';
                    ctxD.fill();
                    atom.flash += 2;
                    if (atom.flash > 20) atom.flash = 0;
                }

                ctxD.beginPath();
                ctxD.arc(atom.x, atom.y, atom.radius, 0, Math.PI * 2);
                ctxD.fillStyle = '#0284c7';
                ctxD.fill();
                ctxD.strokeStyle = '#38bdf8';
                ctxD.lineWidth = 2;
                ctxD.stroke();
            });

            for (let i = particles.length - 1; i >= 0; i--) {
                let p = particles[i];
                p.x += p.vx;
                p.y += p.vy;

                ctxD.beginPath();
                ctxD.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
                ctxD.fillStyle = p.color;
                ctxD.fill();

                atoms.forEach(function(atom) {
                    let dx = p.x - atom.x;
                    let dy = p.y - atom.y;
                    let dist = Math.sqrt(dx * dx + dy * dy);

                    if (dist < atom.radius + p.radius) {
                        atom.flash = 1;
                        atom.vx += p.vx * (p.mass / 200);
                        registerDetection(p.type, p.mass);
                        particles.splice(i, 1);
                    }
                });

                if (p.x > 450) particles.splice(i, 1);
            }

            // Desenhar Gráfico (Painel Direito)
            ctxC.fillStyle = '#020617';
            ctxC.fillRect(0, 0, 450, 400);

            // Zonas de cor de fundo estáticas no gráfico
            ctxC.fillStyle = 'rgba(59, 130, 246, 0.08)';
            ctxC.fillRect(60, 40, 360, 140);
            ctxC.fillStyle = 'rgba(239, 68, 68, 0.05)';
            ctxC.fillRect(60, 180, 360, 160);

            // Linha divisória verde
            ctxC.beginPath();
            ctxC.moveTo(60, 180);
            ctxC.lineTo(420, 180);
            ctxC.strokeStyle = '#10b981';
            ctxC.lineWidth = 2;
            ctxC.stroke();

            // Eixos cartesianos
            ctxC.beginPath();
            ctxC.moveTo(60, 30);
            ctxC.lineTo(60, 340);
            ctxC.lineTo(430, 340);
