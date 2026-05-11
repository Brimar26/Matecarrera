
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Matecarrera - Mejorado</title>
    <style>
        :root {
            --sky-blue: #70cfff;
            --grass-green: #3a9b3a;
            --road-gray: #555;
            --gold: #ffcc00;
        }

        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--sky-blue);
            text-align: center;
        }

        /* Título Principal */
        #main-title {
            color: white;
            font-size: 3rem;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
            margin-top: 20px;
        }

        /* Contador de aciertos */
        #score-counter {
            color: #333;
            font-size: 1.5rem;
            font-weight: bold;
            margin-top: 10px;
        }

        /* El Paisaje */
        #game-area {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        /* Nubes (generadas por CSS) */
        .cloud {
            position: absolute;
            background: rgba(255, 255, 255, 0.8);
            border-radius: 50px;
            animation: moveClouds linear infinite;
        }
        .cloud::before, .cloud::after {
            content: ''; position: absolute; background: inherit; border-radius: 50%;
        }
        @keyframes moveClouds {
            from { left: 110%; } to { left: -20%; }
        }

        /* Área de césped y pista */
        #ground {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 40vh;
            background-color: var(--grass-green);
            background-image: linear-gradient(var(--grass-green) 50%, #2e7d2e 50%);
            background-size: 100% 20px;
        }

        /* Pista de carreras (perspectiva) */
        #track {
            position: absolute;
            bottom: 0;
            left: 50%;
            transform: translateX(-50%);
            width: 80%;
            height: 100%;
            background: 
                repeating-linear-gradient(90deg, #fff 0, #fff 10px, transparent 10px, transparent 40px),
                var(--road-gray);
            clip-path: polygon(20% 0%, 80% 0%, 100% 100%, 0% 100%); /* Efecto perspectiva */
        }

        /* Línea de meta */
        #finish-line {
            position: absolute;
            top: 10px;
            left: 50%;
            transform: translateX(-50%);
            width: 60%;
            height: 30px;
            background-image: 
                linear-gradient(45deg, #fff 25%, transparent 25%), 
                linear-gradient(-45deg, #fff 25%, transparent 25%),
                linear-gradient(45deg, transparent 75%, #fff 75%),
                linear-gradient(-45deg, transparent 75%, #fff 75%);
            background-size: 20px 20px;
            background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
            background-color: #333;
        }

        /* ROBOT Y CARRO GRANDES */
        #robot-driver {
            position: absolute;
            bottom: 5vh; /* Empieza al fondo de la pista */
            left: 50%;
            transform: translateX(-50%);
            font-size: 120px; /* ¡SUPER GRANDE! */
            transition: bottom 0.5s ease-out;
            z-index: 10;
        }

        /* CAJA DE MATEMÁTICAS (CENTRAL) */
        #math-box {
            position: absolute;
            top: 35vh; /* Centrado vertical */
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(255, 255, 255, 0.9);
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 8px 20px rgba(0,0,0,0.2);
            border: 4px solid #444;
            min-width: 300px;
            z-index: 20;
        }

        #operation-text { font-size: 2rem; color: #333; margin-bottom: 20px; font-weight: bold;}
        
        #answer-input {
            width: 100px;
            font-size: 1.8rem;
            padding: 10px;
            text-align: center;
            border: 3px solid #ccc;
            border-radius: 8px;
            margin-right: 10px;
        }

        #submit-btn {
            font-size: 1.5rem;
            padding: 12px 24px;
            background-color: #4CAF50; /* Verde */
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: background 0.2s;
        }
        #submit-btn:hover { background-color: #3d8b40; }

        /* PANTALLA DE RESULTADOS */
        #results-overlay {
            display: none; /* Oculto al inicio */
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.8);
            z-index: 100;
            color: white;
            justify-content: center;
            align-items: center;
            flex-direction: column;
        }

        #final-message { font-size: 3rem; margin-bottom: 10px; }
        #percentage-text { font-size: 5rem; color: var(--gold); font-weight: bold; margin-bottom: 30px; }

        .game-btn {
            font-size: 1.2rem;
            padding: 15px 30px;
            margin: 10px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
        }
        .btn-restart { background-color: #2196F3; color: white; }
        .btn-share { background-color: #ff9800; color: white; }

    </style>
</head>
<body>

    <div id="game-area">
        <h1 id="main-title">Matecarrera</h1>
        <p id="score-counter">Aciertos: <span id="hits">0</span> / 10</p>

        <div class="cloud" style="width: 150px; height: 60px; top: 15%; animation-duration: 25s;"></div>
        <div class="cloud" style="width: 100px; height: 40px; top: 25%; animation-duration: 35s; animation-delay: -10s;"></div>
        <div class="cloud" style="width: 180px; height: 70px; top: 10%; animation-duration: 30s; animation-delay: -5s;"></div>

        <div id="ground">
            <div id="track">
                <div id="finish-line"></div>
            </div>
        </div>

        <div id="robot-driver">🤖🏎️</div>

        <div id="math-box">
            <div id="operation-text">Cargando...</div>
            <input type="number" id="answer-input" placeholder="?">
            <button id="submit-btn" onclick="checkAnswer()">Responder</button>
        </div>
    </div>

    <div id="results-overlay">
        <h2 id="final-message">¡Eres un Ganador!</h2>
        <p id="percentage-text">100%</p>
        <div>
            <button class="game-btn btn-restart" onclick="restartGame()">Jugar de nuevo</button>
            <button class="game-btn btn-share" onclick="shareGame()">Compartir 📤</button>
        </div>
    </div>

    <script>
        let score = 0;
        let currentLevel = 0;
        const totalLevels = 10;
        let currentCorrectAnswer = 0;

        // --- Generación de Operaciones de 4to Grado ---
        function generateOperation() {
            if (currentLevel >= totalLevels) {
                showResults();
                return;
            }

            const ops = ['+', '-', '*', '/'];
            const op = ops[Math.floor(Math.random() * ops.length)];
            let a, b;

            switch (op) {
                case '+': // Sumas hasta 500
                    a = Math.floor(Math.random() * 250) + 10;
                    b = Math.floor(Math.random() * 250) + 10;
                    currentCorrectAnswer = a + b;
                    break;
                case '-': // Restas sin negativos
                    a = Math.floor(Math.random() * 400) + 100;
                    b = Math.floor(Math.random() * (a - 10)) + 10;
                    currentCorrectAnswer = a - b;
                    break;
                case '*': // Tablas hasta el 12
                    a = Math.floor(Math.random() * 11) + 2;
                    b = Math.floor(Math.random() * 11) + 2;
                    currentCorrectAnswer = a * b;
                    break;
                case '/': // Divisiones exactas sencillas
                    b = Math.floor(Math.random() * 9) + 2;
                    a = b * (Math.floor(Math.random() * 10) + 1); // Asegura división exacta
                    currentCorrectAnswer = a / b;
                    break;
            }

            document.getElementById('operation-text').innerText = `¿Cuánto es ${a} ${op === '*' ? 'x' : op === '/' ? '÷' : op} ${b}?`;
            document.getElementById('answer-input').value = '';
            document.getElementById('answer-input').focus();
        }

        // --- Sonido de Moneda (Usando Web Audio API) ---
        function playCoinSound() {
            try {
                const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();

                osc.type = 'sine';
                osc.frequency.setValueAtTime(987.77, audioCtx.currentTime); // Si (B5)
                osc.frequency.exponentialRampToValueAtTime(1318.51, audioCtx.currentTime + 0.1); // Mi (E6)

                gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.3);

                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.3);
            } catch(e) {
                console.log("El navegador bloqueó el audio inicial.");
            }
        }

        // --- Mecánica del Juego ---
        function checkAnswer() {
            const userAnswer = parseInt(document.getElementById('answer-input').value);

            if (userAnswer === currentCorrectAnswer) {
                score++;
                playCoinSound();
                updateRobotPosition();
            } else {
                // Pequeña vibración visual si falla (opcional)
                const box = document.getElementById('math-box');
                box.style.animation = 'shake 0.3s';
                setTimeout(() => box.style.animation = '', 300);
            }

            currentLevel++;
            document.getElementById('hits').innerText = score;
            generateOperation();
        }

        function updateRobotPosition() {
            const robot = document.getElementById('robot-driver');
            // Mueve el robot de abajo hacia arriba (más cerca de la meta)
            // 5vh es el inicio, 30vh es casi la meta
            const newBottom = 5 + (score * 2.5); // Avanza un 2.5% del alto por acierto
            robot.style.bottom = `${newBottom}vh`;
            
            // Efecto de hacerse ligeramente más pequeño por perspectiva
            const scale = 1 - (score * 0.02);
            robot.style.transform = `translateX(-50%) scale(${scale})`;
        }

        // --- Final del Juego ---
        function showResults() {
            const percentage = Math.round((score / totalLevels) * 100);
            document.getElementById('math-box').style.display = 'none';
            document.getElementById('robot-driver').style.bottom = '30vh'; // Robot llega a la meta visualmente
            document.getElementById('results-overlay').style.display = 'flex';
            document.getElementById('percentage-text').innerText = `${percentage}%`;
        }

        function restartGame() {
            score = 0;
            currentLevel = 0;
            document.getElementById('hits').innerText = "0";
            document.getElementById('math-box').style.display = 'block';
            document.getElementById('results-overlay').style.display = 'none';
            document.getElementById('robot-driver').style.bottom = '5vh';
            document.getElementById('robot-driver').style.transform = `translateX(-50%) scale(1)`;
            generateOperation();
        }

        function shareGame() {
            const msg = `¡Completé el desafío de Matecarrera con un ${document.getElementById('percentage-text').innerText}! 🤖🏎️`;
            if (navigator.share) {
                navigator.share({
                    title: 'Matecarrera - Mejorado',
                    text: msg,
                    url: window.location.href
                });
            } else {
                alert(`Copiado al portapapeles:\n${msg}`);
            }
        }

        // Soporte para tecla Enter
        document.getElementById('answer-input').addEventListener('keypress', function (e) {
            if (e.key === 'Enter') checkAnswer();
        });

        // Iniciar
        generateOperation();

    </script>
    <style>
        /* Animación de error */
        @keyframes shake {
            0%, 100% { transform: translate(-50%, -50%) translateX(0); }
            20%, 60% { transform: translate(-50%, -50%) translateX(-5px); }
            40%, 80% { transform: translate(-50%, -50%) translateX(5px); }
        }
    </style>
</body>
</html>
