
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Robot Mate: Carrera de Operaciones</title>
    <style>
        :root { --primary: #2ecc71; --secondary: #e74c3c; --sky: #87CEEB; }
        body { margin: 0; font-family: 'Arial', sans-serif; overflow: hidden; background: var(--sky); text-align: center; }
        
        /* Paisaje */
        #game-container { position: relative; width: 100vw; height: 100vh; background: linear-gradient(to bottom, #87CEEB 60%, #5d4037 60%, #228B22 65%); overflow: hidden; }
        .cloud { position: absolute; background: white; width: 100px; height: 40px; border-radius: 20px; opacity: 0.8; animation: move 20s linear infinite; }
        @keyframes move { from { left: -150px; } to { left: 110%; } }

        /* Robot y Elementos */
        #robot { position: absolute; bottom: 10%; left: 50px; font-size: 50px; transition: 0.3s; z-index: 10; }
        .operation-card { position: absolute; right: -200px; top: 50%; transform: translateY(-50%); background: white; padding: 20px; border-radius: 15px; border: 4px solid #333; box-shadow: 0 10px 0 #bbb; font-size: 24px; }
        
        /* Interfaz */
        #ui { position: absolute; top: 20px; width: 100%; color: white; text-shadow: 2px 2px #000; font-size: 24px; }
        #modal { display: none; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: white; padding: 40px; border-radius: 20px; box-shadow: 0 0 50px rgba(0,0,0,0.5); z-index: 100; }
        button { padding: 10px 20px; font-size: 18px; cursor: pointer; background: var(--primary); color: white; border: none; border-radius: 5px; margin: 5px; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="ui">Aciertos: <span id="score">0</span> / 10</div>
    <div id="robot">🤖🏎️</div>
    
    <div id="question-box" class="operation-card">
        <p id="math-problem">¿2 + 2?</p>
        <input type="number" id="answer-input" placeholder="?">
        <button onclick="checkAnswer()">Responder</button>
    </div>

    <!-- Nubes decorativas -->
    <div class="cloud" style="top:10%; animation-duration: 25s;"></div>
    <div class="cloud" style="top:20%; animation-duration: 30s; width:150px;"></div>
</div>

<div id="modal">
    <h2 id="final-msg">¡Lo lograste!</h2>
    <p id="percentage"></p>
    <button onclick="resetGame()">Jugar de nuevo</button>
    <button onclick="shareGame()" style="background: #3498db;">Compartir Juego</button>
</div>

<script>
    let score = 0;
    let attempts = 0;
    const maxQuestions = 10;
    let currentAnswer = 0;

    // Sonidos simulados con la Web Audio API (Estilo Coin)
    function playCoinSound() {
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        const oscillator = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        oscillator.type = 'sine';
        oscillator.frequency.setValueAtTime(987.77, audioCtx.currentTime); // Si (B5)
        oscillator.frequency.exponentialRampToValueAtTime(1318.51, audioCtx.currentTime + 0.1); // Mi (E6)
        gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3);
        oscillator.connect(gain);
        gain.connect(audioCtx.destination);
        oscillator.start();
        oscillator.stop(audioCtx.currentTime + 0.3);
    }

    function generateOperation() {
        if (attempts >= maxQuestions) {
            endGame();
            return;
        }
        
        const ops = ['+', '-', '*', '/'];
        const op = ops[Math.floor(Math.random() * ops.length)];
        let a, b;

        switch(op) {
            case '+': a = rand(10, 100); b = rand(10, 100); currentAnswer = a + b; break;
            case '-': a = rand(50, 100); b = rand(1, 50); currentAnswer = a - b; break;
            case '*': a = rand(2, 9); b = rand(2, 9); currentAnswer = a * b; break;
            case '/': b = rand(2, 10); a = b * rand(1, 10); currentAnswer = a / b; break;
        }

        document.getElementById('math-problem').innerText = `¿Cuánto es ${a} ${op} ${b}?`;
        document.getElementById('answer-input').value = '';
        document.getElementById('question-box').style.right = '100px';
    }

    function rand(min, max) { return Math.floor(Math.random() * (max - min + 1) + min); }

    function checkAnswer() {
        const userAns = parseInt(document.getElementById('answer-input').value);
        if (userAns === currentAnswer) {
            score++;
            playCoinSound();
            document.getElementById('robot').style.left = (score * 8) + "%"; // El robot avanza
        }
        
        attempts++;
        document.getElementById('score').innerText = score;
        document.getElementById('question-box').style.right = '-300px';
        setTimeout(generateOperation, 1000);
    }

    function endGame() {
        const percent = (score / maxQuestions) * 100;
        document.getElementById('modal').style.display = 'block';
        document.getElementById('percentage').innerText = `Puntaje: ${percent}% - Eres un ganador`;
    }

    function resetGame() {
        score = 0;
        attempts = 0;
        document.getElementById('score').innerText = "0";
        document.getElementById('robot').style.left = "50px";
        document.getElementById('modal').style.display = 'none';
        generateOperation();
    }

    function shareGame() {
        const text = `¡Acabo de completar el desafío de Robot Mate con ${score}/10 aciertos!`;
        if (navigator.share) {
            navigator.share({ title: 'Robot Mate', text: text, url: window.location.href });
        } else {
            alert("Copiado al portapapeles: " + text);
        }
    }

    // Iniciar
    generateOperation();
</script>

</body>
</html>
