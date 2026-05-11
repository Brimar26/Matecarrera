
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Robot Mate: Multidispositivo</title>
    <style>
        :root {
            --grass: #2ecc71;
            --road: #34495e;
            --sky: #87CEEB;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: var(--sky);
            overflow: hidden;
            display: flex;
            flex-direction: column;
            height: 100vh;
        }

        /* PAISAJE RESPONSIVO */
        #game-world {
            position: relative;
            flex: 1;
            background: linear-gradient(to bottom, var(--sky) 0%, var(--sky) 70%, var(--grass) 70%, var(--grass) 100%);
            display: flex;
            align-items: flex-end;
            justify-content: flex-start;
            padding-bottom: 20px;
        }

        /* Nubes */
        .cloud {
            position: absolute;
            background: white;
            border-radius: 50px;
            height: 40px;
            width: 100px;
            opacity: 0.8;
            animation: move linear infinite;
        }
        @keyframes move { from { left: -150px; } to { left: 110vw; } }

        /* ROBOT MATE */
        #robot {
            font-size: clamp(40px, 10vw, 80px); /* Tamaño fluido */
            position: relative;
            left: 5%;
            transition: left 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            z-index: 10;
            filter: drop-shadow(2px 5px 2px rgba(0,0,0,0.2));
        }

        /* INTERFAZ DE USUARIO */
        #ui-layer {
            position: absolute;
            top: 0;
            width: 100%;
            padding: 15px;
            display: flex;
            justify-content: space-between;
            color: white;
            font-weight: bold;
            font-size: clamp(16px, 4vw, 24px);
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        }

        /* TARJETA DE OPERACIÓN */
        #card-container {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 90%;
            max-width: 400px;
            background: rgba(255, 255, 255, 0.95);
            padding: 20px;
            border-radius: 20px;
            text-align: center;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
            border: 5px solid #f1c40f;
        }

        #math-text { font-size: 2rem; margin-bottom: 15px; color: #333; }
        
        input {
            width: 100%;
            padding: 15px;
            font-size: 1.5rem;
            border-radius: 10px;
            border: 2px solid #ddd;
            margin-bottom: 10px;
            text-align: center;
            outline: none;
        }

        button {
            width: 100%;
            padding: 15px;
            background: #e67e22;
            color: white;
            border: none;
            border-radius: 10px;
            font-size: 1.2rem;
            cursor: pointer;
            font-weight: bold;
            transition: transform 0.1s;
        }
        button:active { transform: scale(0.95); }

        /* MODAL FINAL */
        #modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.8);
            z-index: 100;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            padding: 20px;
        }

        .btn-share { background: #3498db; margin-top: 10px; }
    </style>
</head>
<body>

<div id="game-world">
    <div id="ui-layer">
        <div>Robot Mate 🏎️</div>
        <div>Meta: <span id="progress">0</span> / 10</div>
    </div>

    <!-- Nubes -->
    <div class="cloud" style="top: 10%; animation-duration: 25s;"></div>
    <div class="cloud" style="top: 25%; animation-duration: 18s; width: 140px;"></div>

    <div id="robot">🤖</div>

    <div id="card-container">
        <div id="math-text">Cargando...</div>
        <input type="number" id="user-input" pattern="\d*" inputmode="numeric" placeholder="?">
        <button onclick="check()">¡ENVIAR!</button>
    </div>
</div>

<div id="modal">
    <h1 id="result-title">¡ERES UN GANADOR!</h1>
    <h2 id="result-percent">100%</h2>
    <p style="margin: 20px 0;">¡Has cruzado la meta!</p>
    <button onclick="restart()">Jugar de nuevo</button>
    <button class="btn-share" onclick="share()">Compartir con amigos 📤</button>
</div>

<script>
    let score = 0;
    let total = 0;
    let target = 10;
    let currentAns = 0;

    function playCoin() {
        const ctx = new (window.AudioContext || window.webkitAudioContext)();
        const osc = ctx.createOscillator();
        const g = ctx.createGain();
        osc.frequency.setValueAtTime(900, ctx.currentTime);
        osc.frequency.exponentialRampToValueAtTime(1200, ctx.currentTime + 0.1);
        g.gain.setValueAtTime(0.1, ctx.currentTime);
        g.gain.linearRampToValueAtTime(0, ctx.currentTime + 0.4);
        osc.connect(g); g.connect(ctx.destination);
        osc.start(); osc.stop(ctx.currentTime + 0.4);
    }

    function generate() {
        if (total >= target) { finish(); return; }
        
        const types = ['+', '-', '*', '/'];
        const type = types[Math.floor(Math.random() * types.length)];
        let a, b;

        if(type === '+') { a = r(10,99); b = r(10,99); currentAns = a+b; }
        else if(type === '-') { a = r(50,100); b = r(1,49); currentAns = a-b; }
        else if(type === '*') { a = r(2,9); b = r(2,9); currentAns = a*b; }
        else { b = r(2,10); a = b * r(1,10); currentAns = a/b; }

        document.getElementById('math-text').innerText = `${a} ${type === '*' ? '×' : type === '/' ? '÷' : type} ${b} = ?`;
        document.getElementById('user-input').value = '';
        document.getElementById('user-input').focus();
    }

    function r(min, max) { return Math.floor(Math.random() * (max - min + 1)) + min; }

    function check() {
        const val = parseInt(document.getElementById('user-input').value);
        if (val === currentAns) {
            score++;
            playCoin();
            // Mover robot proporcionalmente
            document.getElementById('robot').style.left = (5 + (score * 8.5)) + "%";
        }
        total++;
        document.getElementById('progress').innerText = score;
        generate();
    }

    // Soporte para tecla Enter
    document.getElementById('user-input').addEventListener('keypress', (e) => {
        if (e.key === 'Enter') check();
    });

    function finish() {
        const p = (score / target) * 100;
        document.getElementById('modal').style.display = 'flex';
        document.getElementById('result-percent').innerText = p + "% Logrado";
        document.getElementById('result-title').innerText = p >= 60 ? "¡ERES UN GANADOR!" : "¡SIGUE PRACTICANDO!";
    }

    function restart() {
        score = 0; total = 0;
        document.getElementById('progress').innerText = "0";
        document.getElementById('robot').style.left = "5%";
        document.getElementById('modal').style.display = 'none';
        generate();
    }

    function share() {
        const msg = `¡Saqué ${score}/10 en Robot Mate! ¿Puedes superarme?`;
        if (navigator.share) {
            navigator.share({ title: 'Robot Mate', text: msg, url: window.location.href });
        } else {
            alert(msg);
        }
    }

    generate();
</script>
</body>
</html>
