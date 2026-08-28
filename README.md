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

        /* PAINEL LARANJA PEDIDO */
        .panel {
            background-color: #ea580c;
            border-radius: 12px;
            padding: 15px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
            width: 450px;
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

        /* ÁREAS DE SIMULAÇÃO REFEITAS USANDO DIVS SÓLIDAS */
        .display-box {
            background-color: #020617;
            border-radius: 8px;
            width: 450px;
            height: 400px;
            position: relative;
            overflow: hidden;
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

        /* BOTÕES PRETO E VERDE PEDIDOS */
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

        /* ELEMENTOS DA CÂMARA DE XENÔNIO */
        .atom {
            width: 24px;
            height: 24px;
            background-color: #0284c7;
            border: 2px solid #38bdf8;
            border-radius: 50%;
            position: absolute;
            transition: transform 0.1s linear;
        }

        .particle {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            position: absolute;
        }

        /* ANIMAÇÃO DE IMPACTO CINTILAÇÃO (S1) */
        @keyframes pulse-effect {
            0% { box-shadow: 0 0 0 0 rgba(56, 189, 248, 0.7); }
            100% { box-shadow: 0 0 0 20px rgba(56, 189, 248, 0); }
        }
        .pulse {
            animation: pulse-effect 0.4s ease-out;
        }

        /* LINHA DIVISÓRIA DO GRÁFICO */
        .chart-line {
            position: absolute;
            left: 50px;
            top: 180px;
            width: 370px;
            height: 2px;
            background-color: #10b981;
            z-index: 2;
        }

        /* TEXTOS INTERNOS DO GRÁFICO */
        .chart-label {
            position: absolute;
            font-size: 11px;
            color: #64748b;
        }

        .dot {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            position: absolute;
            border: 1px solid #ffffff;
            transform: translate(-50%, -50%);
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
            <div id="detectorView" class="display-box"></div>
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
            <div id="chartView" class="display-box">
                <!-- Zonas de Fundo -->
                <div style="position:absolute; left:50px; top:40px; width:370px; height:140px; background-color:rgba(59,130,246,0.08);"></div>
                <div style="position:absolute; left:50px; top:180px; width:370px; height:160px; background-color:rgba(239,68,68,0.05);"></div>
                
                <div class="chart-line"></div>
                
                <!-- Textos das Zonas -->
                <div class="chart-label" style="left:280px; top:60px;">Zona de Ruído (ER)</div>
                <div class="chart-label" style="left:230px; top:280px; color:#f87171;">Zona de Matéria Escura (NR)</div>
                <div class="chart-label" style="left:170px; top:360px; color:#94a3b8;">Energia de Recuo (Subindo &rarr;)</div>
            </div>
            <div class="stats">
                <span>Ruídos Filtrados: <strong id="count-noise" style="color:#000000">0</strong></span>
                <span>WIMPs Confirmados: <strong id="count-wimp" style="color:#ffffff">0</strong></span>
            </div>
        </div>
    </div>

    <script>
        const detectorView = document.getElementById('detectorView');
        const chartView = document.getElementById('chartView');
        const massSlider = document.getElementById('mass-slider');
        const massVal = document.getElementById('mass-val');

        let atoms = [];
        let counters = { noise: 0, wimp: 0 };
        let currentWimpMass = 100;

        massSlider.addEventListener('input', function(e) {
            currentWimpMass = parseInt(e.target.value);
            massVal.innerText = currentWimpMass + " GeV";
        });

        // Criar os Átomos de Xenônio visíveis fisicamente na tela
        for (let i = 0; i < 15; i++) {
            let el = document.createElement('div');
            el.className = 'atom';
            detectorView.appendChild(el);

            atoms.push({
                element: el,
                x: Math.random() * 380 + 20,
                y: Math.random() * 340 + 20,
                vx: (Math.random() - 0.5) * 2,
                vy: (Math.random() - 0.5) * 2
            });
        }

        // Evento Injetar Ruído (Partícula Amarela)
        document.getElementById('trigger-noise').addEventListener('click', function() {
            createParticle('#eab308', 'noise');
        });

        // Evento Disparar Matéria Escura (Partícula Vermelha)
        document.getElementById('trigger-wimp').addEventListener('click', function() {
            createParticle('#ef4444', 'wimp');
        });

        function createParticle(color, type) {
            let pEl = document.createElement('div');
            pEl.className = 'particle';
            pEl.style.backgroundColor = color;
            pEl.style.left = '0px';
            
            let targetY = Math.random() * 340 + 30;
            pEl.style.top = targetY + 'px';
            detectorView.appendChild(pEl);

            let posX = 0;
            let interval = setInterval(function() {
                posX += 8;
                pEl.style.left = posX + 'px';

                // Checar proximidade com os átomos
                atoms.forEach(function(atom) {
                    let dx = posX - atom.x;
                    let dy = targetY - atom.y;
                    let dist = Math.sqrt(dx * dx + dy * dy);

                    if (dist < 22) { // Houve Colisão
                        clearInterval(interval);
                        pEl.remove();
                        
                        // Efeito visual de cintilação (S1) instantâneo
                        atom.element.classList.add('pulse');
                        setTimeout(() => atom.element.classList.remove('pulse'), 400);

                        // Agita o átomo (transfere energia)
                        atom.vx += type === 'wimp' ? (currentWimpMass / 50) : 0.5;
                        atom.vy += (Math.random() - 0.5) * 2;

                        plotData(type);
                    }
                });

                if (posX > 450) {
