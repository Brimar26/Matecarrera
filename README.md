
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

        #main-title {
            color: white;
            font-size: 3rem;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
            margin-top: 20px;
        }

        #score-counter {
            color: #333;
            font-size: 1.5rem;
            font-weight: bold;
            margin-top: 10px;
        }

        #game-area {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .cloud {
            position: absolute;
            background: rgba(255, 255, 255, 0.8);
            border-radius: 50px;
            animation: moveClouds linear infinite;
        }
        @keyframes moveClouds {
            from { left: 110%; } to { left: -20%; }
        }

        #ground {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 40vh;
            background-color: var(--grass-green);
            background-image: linear-gradient(var(--grass-green) 50%, #2e7d2e 50%);
            background-size: 100% 20px;
        }

        #track {
            position: absolute;
            bottom: 0;
            left: 50%;
            transform: translateX(-50%);
            width: 80%;
            height: 100%;
            background: repeating-linear-gradient(90deg, #fff 0, #fff 10px, transparent 10px, transparent 40px), var(--road-gray);
            clip-path: polygon(20% 0%, 80% 0%, 100% 100%, 0% 100%);
        }

        #finish-line {
            position: absolute;
            top: 10px;
            left: 50%;
            transform: translateX(-50%);
            width: 60%;
            height: 30px;
            background-image: linear-gradient(45deg, #fff 25%, transparent 25%), linear-gradient(-45deg, #fff 25%, transparent 25%), linear-gradient(45deg, transparent 75%, #fff 75%), linear-gradient(-45deg, transparent 75%, #fff 75%);
            background-size: 20px 20px;
            background-color: #333;
        }

        #robot-driver {
            position: absolute;
            bottom: 5vh;
            left: 50%;
            transform: translateX(-50%);
            font-size: 120px;
            transition: bottom 0.5s ease-out;
            z-index: 10;
        }

        #math-box {
            position: absolute;
            top: 35vh;
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
        }

        #submit-btn {
            font-size: 1.5rem;
            padding: 12px 24px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
        }

        #results-overlay {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.9);
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

        @keyframes shake {
            0%, 100% { transform: translate(-50%, -50%) translateX(0); }
            25% { transform: translate(-50%, -50%) translateX(-10px); }
            75% { transform: translate(-50%, -50%) translateX(10px); }
        }
    </style>
</head>
<body>

    <div id="game-area">
        <h1 id="main-title">🏎️ Matecarrera</h1>
        <p id="score-counter">Aciertos: <span id="hits">0</span> / 10</p>

        <div id="ground">
            <div id="track">
                <div id="finish-line"></div>
            </div>
        </div>

        <div id="robot-driver">🏎️</div>

        <div id="math-box">
            <div id="operation-text">¡Dale a Responder!</div>
            <input type="number" id="answer-input" placeholder="?">
            <button id="submit-btn" onclick="checkAnswer()">Responder</button>
        </div>
    </div>

    <div id="results-overlay">
        <h2 id="final-message">Resultado</h2>
        <p id="percentage-text">0%</p>
        <div>
            <button class="game-btn btn-restart" onclick="restartGame()">Intentar de nuevo</button>
        </div>
    </div>

    <script>
        let score = 0;
        let currentLevel = 0;
        const totalLevels = 10;
        let currentCorrectAnswer = 0;
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

        // --- SISTEMA DE SONIDOS ---
        function playTone(freq, type, duration, vol = 0.2) {
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.type = type;
            osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
            gain.gain.setValueAtTime(vol, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duration);
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            osc.start();
            osc.stop(audioCtx.currentTime + duration);
        }

        function playCoinSound() {
            playTone(987, 'sine', 0.1);
            setTimeout(() => playTone(1318, 'sine', 0.2), 50);
        }

        function playErrorSound() {
            playTone(150, 'square', 0.3, 0.1);
        }

        function playWinFanfare() {
            [523, 659, 783, 1046].forEach((f, i) => {
                setTimeout(() => playTone(f, 'triangle', 0.4), i * 150);
            });
        }

        function playLoseSound() {
            [300, 250, 200].forEach((f, i) => {
                setTimeout(() => playTone(f, 'sawtooth', 0.5), i * 200);
            });
        }

        // --- LÓGICA DEL JUEGO ---
        function generateOperation() {
            if (currentLevel >= totalLevels) {
                showResults();
                return;
            }
            const ops = ['+', '-', '*', '/'];
            const op = ops[Math.floor(Math.random() * ops.length)];
            let a, b;
            if (op === '+') { a = rand(10, 250); b = rand(10, 250); currentCorrectAnswer = a + b; }
            else if (op === '-') { a = rand(100, 400); b = rand(10, 90); currentCorrectAnswer = a - b; }
            else if (op === '*') { a = rand(2, 12); b = rand(2, 12); currentCorrectAnswer = a * b; }
            else { b = rand(2, 10); a = b * rand(1, 10); currentCorrectAnswer = a / b; }

            document.getElementById('operation-text').innerText = `¿Cuánto es ${a} ${op === '*' ? 'x' : op === '/' ? '÷' : op} ${b}?`;
            document.getElementById('answer-input').value = '';
            document.getElementById('answer-input').focus();
        }

        function rand(min, max) { return Math.floor(Math.random() * (max - min + 1)) + min; }

        function checkAnswer() {
            if (audioCtx.state === 'suspended') audioCtx.resume();
            const userAnswer = parseInt(document.getElementById('answer-input').value);

            if (userAnswer === currentCorrectAnswer) {
                score++;
                playCoinSound();
                document.getElementById('robot-driver').style.bottom = `${5 + (score * 3)}vh`;
            } else {
                playErrorSound();
                document.getElementById('math-box').style.animation = 'shake 0.3s';
                setTimeout(() => document.getElementById('math-box').style.animation = '', 300);
            }

            currentLevel++;
            document.getElementById('hits').innerText = score;
            generateOperation();
        }

        function showResults() {
            const percentage = (score / totalLevels) * 100;
            const overlay = document.getElementById('results-overlay');
            const msg = document.getElementById('final-message');
            
            overlay.style.display = 'flex';
            document.getElementById('percentage-text').innerText = `${percentage}%`;

            if (percentage >= 70) {
                msg.innerText = "¡Eres un ganador! 🏆";
                msg.style.color = "#4CAF50";
                playWinFanfare();
            } else {
                msg.innerText = "Necesitas mejorar 🚩";
                msg.style.color = "#FF5252";
                playLoseSound();
            }
        }

        function restartGame() {
            score = 0;
            currentLevel = 0;
            document.getElementById('hits').innerText = "0";
            document.getElementById('results-overlay').style.display = 'none';
            document.getElementById('robot-driver').style.bottom = '5vh';
            generateOperation();
        }

        document.getElementById('answer-input').addEventListener('keypress', (e) => { if (e.key === 'Enter') checkAnswer(); });
        generateOperation();
    </script>
</body>
</html>
