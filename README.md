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

        /* PAINEL LARANJA */
        .panel {
            background-color: #ea580c;
            border-radius: 12px;
            padding: 20px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
            width: 450px;
            box-sizing: border-box;
        }

        .panel-title {
            font-size: 14px;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 0.05em;
            margin-bottom: 15px;
            color: #ffffff;
            border-bottom: 1px solid #ff7a33;
            padding-bottom: 5px;
        }

        /* ÁREAS DE CONTEÚDO CORRIGIDAS */
        .display-box {
            background-color: #020617;
            border-radius: 8px;
            height: 350px;
            position: relative;
            overflow: hidden;
            border: 1px solid #334155;
        }

        /* GRID FORÇADO PARA OS ÁTOMOS APARECEREM SEMPRE */
        #detectorView {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 15px;
            padding: 25px;
            align-content: center;
            justify-items: center;
            box-sizing: border-box;
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

        /* BOTÕES PRETO E VERDE */
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

        /* ÁTOMOS DE XENÔNIO VISÍVEIS */
        .atom {
            width: 40px;
            height: 40px;
            background-color: #0284c7;
            border: 2px solid #38bdf8;
            border-radius: 50%;
            transition: all 0.2s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 10px;
            font-weight: bold;
            color: #ffffff;
        }

        /* EFEITO DE IMPACTO PISCANDE */
        .flash-active {
            background-color: #38bdf8 !important;
            box-shadow: 0 0 20px #38bdf8;
            transform: scale(1.2);
        }

        /* ELEMENTOS DO GRÁFICO DIREITO */
        .chart-zone-top {
            position: absolute;
            left: 50px;
            top: 30px;
            width: 330px;
            height: 130px;
            background-color: rgba(59, 130, 246, 0.15);
            border-left: 2px solid #64748b;
        }

        .chart-zone-bottom {
            position: absolute;
            left: 50px;
            top: 162px;
            width: 330px;
            height: 130px;
            background-color: rgba(239, 68, 68, 0.1);
            border-left: 2px solid #64748b;
            border-bottom: 2px solid #64748b;
        }

        .chart-line {
            position: absolute;
            left: 50px;
            top: 160px;
            width: 330px;
            height: 2px;
            background-color: #10b981;
        }

        .chart-label {
            position: absolute;
            font-size: 11px;
            font-weight: bold;
        }

        .dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            position: absolute;
            border: 2px solid #ffffff;
            transform: translate(-50%, -50%);
            animation: pop 0.3s ease-out;
        }

        @keyframes pop {
            0% { transform: translate(-50%, -50%) scale(0); }
            100% { transform: translate(-50%, -50%) scale(1); }
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
            <div id="detectorView" class="display-box">
                <!-- Os átomos serão gerados aqui via script de forma visível e fixa -->
            </div>
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
                <!-- Estruturas Estáticas Visuais do Gráfico -->
                <div class="chart-zone-top"></div>
                <div class="chart-zone-bottom"></div>
                <div class="chart-line"></div>
                
                <div class="chart-label" style="left: 230px; top: 40px; color: #94a3b8;">Zona de Ruído (ER)</div>
                <div class="chart-label" style="left: 190px; top: 250px; color: #f87171;">Zona de Matéria Escura (NR)</div>
                <div class="chart-label" style="left: 150px; top: 310px; color: #64748b;">Energia de Recuo (keV) &rarr;</div>
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

        let counters = { noise: 0, wimp: 0 };
        let currentWimpMass = 100;

        massSlider.addEventListener('input', function(e) {
            currentWimpMass = parseInt(e.target.value);
            massVal.innerText = currentWimpMass + " GeV";
        });

        // 1. Gera 12 átomos de Xenônio fixos e perfeitamente visíveis em Grid (Garante o fim da tela preta)
        const totalAtoms = 12;
        for (let i = 0; i < totalAtoms; i++) {
            let atomEl = document.createElement('div');
            atomEl.className = 'atom';
            atomEl.innerText = "Xe";
            detectorView.appendChild(atomEl);
        }

        // Captura a lista de átomos criados para podermos animá-los nos cliques
        const atomList = document.querySelectorAll('.atom');

        // 2. Interações dos Botões Corrigidas para Atualizar os Dois Painéis Simultaneamente
        document.getElementById('trigger-noise').addEventListener('click', function() {
            executeCollision('noise');
        });

        document.getElementById('trigger-wimp').addEventListener('click', function() {
            executeCollision('wimp');
        });

        function executeCollision(type) {
            // Seleciona um átomo aleatório para simular a colisão direta
            let randomIndex = Math.floor(Math.random() * atomList.length);
            let targetAtom = atomList[randomIndex];

            // Ativa o flash visual de cintilação (S1) no painel esquerdo
            targetAtom.classList.add('flash-active');
            setTimeout(function() {
                targetAtom.classList.remove('flash-active');
            }, 250);

            // Processa as pontuações e gera os pontos no gráfico correspondente
            let dot = document.createElement('div');
            dot.className = 'dot';

