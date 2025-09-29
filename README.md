<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Jogo da Cobrinha — 800x250</title>
  <style>
    body {
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      margin: 0;
      font-family: Arial, Helvetica, sans-serif;
      background: #f3f3f3;
    }
    .container {
      text-align: center;
    }
    canvas {
      background: #ffffff; /* fundo branco */
      display: block;
      border: 1px solid #ddd;
    }
    .info {
      margin-top: 8px;
      color: #222;
      font-size: 14px;
    }
    .small {
      font-size: 12px;
      color: #666;
    }
    button {
      margin-left: 8px;
      padding: 6px 10px;
      border-radius: 6px;
      border: 1px solid #ccc;
      cursor: pointer;
      background: white;
    }
  </style>
</head>
<body>
  <div class="container">
    <canvas id="game" width="800" height="250"></canvas>
    <div class="info">
      <span id="status">Pressione ← → ↑ ↓ ou WASD para jogar</span>
      <span class="small" id="timer">Tempo: 0:00</span>
      <button id="restartBtn">Reiniciar agora</button>
    </div>
    <div class="small" style="margin-top:6px;">
      Durante os primeiros <strong>5 minutos</strong> a cobrinha não morrerá (wrap + sem colisão própria). Após 5 minutos, colisões voltam ao normal.
    </div>
  </div>

  <script>
    /* Configurações */
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const WIDTH = canvas.width;   // 800
    const HEIGHT = canvas.height; // 250

    const CELL = 10; // tamanho do bloco (em pixels)
    const COLS = WIDTH / CELL;  // 800 / 10 = 80
    const ROWS = HEIGHT / CELL; // 250 / 10 = 25

    const FPS = 10; // "velocidade normal" (10 updates por segundo)
    const UPDATE_INTERVAL = 1000 / FPS;

    const GRACE_MS = 300000; // 5 minutos = 300000 ms

    /* Estado do jogo */
    let snake;
    let dir;
    let nextDir;
    let food;
    let lastUpdate = 0;
    let startTime = Date.now();
    let running = true;
    let score = 0;

    const statusEl = document.getElementById('status');
    const timerEl = document.getElementById('timer');
    const restartBtn = document.getElementById('restartBtn');

    /* Inicializa jogo */
    function resetGame() {
      startTime = Date.now();
      score = 0;
      // cobra começa no meio, comprimento inicial 5
      const midX = Math.floor(COLS / 2);
      const midY = Math.floor(ROWS / 2);
      snake = [];
      const initialLength = 5;
      for (let i = initialLength - 1; i >= 0; i--) {
        snake.push({ x: midX - i, y: midY });
      }
      dir = { x: 1, y: 0 }; // indo para a direita
      nextDir = { x: 1, y: 0 };
      placeFood();
      running = true;
      statusEl.textContent = 'Jogue! (Setas ou WASD)';
    }

    function placeFood() {
      // sorteia posição que não esteja ocupada pela cobra
      let tries = 0;
      while (true) {
        tries++;
        const fx = Math.floor(Math.random() * COLS);
        const fy = Math.floor(Math.random() * ROWS);
        let collision = false;
        for (let s of snake) {
          if (s.x === fx && s.y === fy) { collision = true; break; }
        }
        if (!collision) {
          food = { x: fx, y: fy };
          return;
        }
        // safety net
        if (tries > 1000) {
          // se não achar, coloque no (0,0)
          food = { x: 0, y: 0 };
          return;
        }
      }
    }

    /* Lógica de atualização */
    function update(delta) {
      const now = Date.now();
      const elapsedFromStart = now - startTime;
      // controla direção (não permitir 180º instantâneo)
      if ((nextDir.x !== -dir.x || nextDir.y !== -dir.y) || snake.length === 1) {
        dir = nextDir;
      }

      // nova cabeça
      const head = { x: snake[snake.length - 1].x + dir.x, y: snake[snake.length - 1].y + dir.y };

      const inGrace = elapsedFromStart < GRACE_MS;

      // durante o período de graça: faz wrap nas bordas
      if (inGrace) {
        if (head.x < 0) head.x = COLS - 1;
        if (head.x >= COLS) head.x = 0;
        if (head.y < 0) head.y = ROWS - 1;
        if (head.y >= ROWS) head.y = 0;
      }

      // checa colisão com paredes (após GRACE, se bater na borda -> game over)
      if (!inGrace) {
        if (head.x < 0 || head.x >= COLS || head.y < 0 || head.y >= ROWS) {
          gameOver('Bateu na parede!');
          return;
        }
      }

      // checa colisão com corpo (após GRACE, se colidir -> game over)
      let hitSelf = false;
      for (let s of snake) {
        if (s.x === head.x && s.y === head.y) {
          hitSelf = true;
          break;
        }
      }
      if (!inGrace && hitSelf) {
        gameOver('Colidiu com o próprio corpo!');
        return;
      }

      // adiciona cabeça
      snake.push(head);

      // comeu?
      if (head.x === food.x && head.y === food.y) {
        score++;
        placeFood();
        // não remove a cauda => cresce
      } else {
        // movimento normal: remove a cauda
        snake.shift();
      }
    }

    function gameOver(reason) {
      running = false;
      statusEl.textContent = `Game Over — ${reason} (reiniciando...)`;
      // pequeno atraso visual antes de reset
      setTimeout(() => {
        resetGame();
      }, 800);
    }

    /* Render */
    function draw() {
      // fundo branco (já pedido)
      ctx.fillStyle = '#ffffff';
      ctx.fillRect(0, 0, WIDTH, HEIGHT);

      // desenha comida (pino preto)
      ctx.fillStyle = '#000000';
      ctx.fillRect(food.x * CELL, food.y * CELL, CELL, CELL);

      // desenha cobrinha (preto)
      for (let i = 0; i < snake.length; i++) {
        const s = snake[i];
        // se quiser estilos diferentes, poderia variar aqui
        ctx.fillRect(s.x * CELL, s.y * CELL, CELL - 0.5, CELL - 0.5);
      }

      // desenha pontuação e tamanho no canto (sutil)
      ctx.font = '12px Arial';
      ctx.fillStyle = '#222';
      ctx.fillText(`Tamanho: ${snake.length}`, 6, 14);
      ctx.fillText(`Pontos: ${score}`, 6, 28);
    }

    /* Loop principal controlando FPS */
    function loop(timestamp) {
      if (!lastUpdate) lastUpdate = timestamp;
      const delta = timestamp - lastUpdate;
      if (delta >= UPDATE_INTERVAL) {
        if (running) update(delta);
        draw();
        lastUpdate = timestamp;
      }

      // atualiza cronômetro visível
      const elapsed = Math.floor((Date.now() - startTime) / 1000); // segundos completos
      const minutes = Math.floor(elapsed / 60);
      const seconds = elapsed % 60;
      timerEl.textContent = `Tempo: ${minutes}:${seconds.toString().padStart(2,'0')}`;

      requestAnimationFrame(loop);
    }

    /* Controles */
    window.addEventListener('keydown', (e) => {
      const key = e.key;
      if (key === 'ArrowLeft' || key === 'a' || key === 'A') {
        nextDir = { x: -1, y: 0 };
      } else if (key === 'ArrowRight' || key === 'd' || key === 'D') {
        nextDir = { x: 1, y: 0 };
      } else if (key === 'ArrowUp' || key === 'w' || key === 'W') {
        nextDir = { x: 0, y: -1 };
      } else if (key === 'ArrowDown' || key === 's' || key === 'S') {
        nextDir = { x: 0, y: 1 };
      }
    });

    restartBtn.addEventListener('click', () => {
      resetGame();
    });

    /* Previne comportamento padrão de algumas teclas (rolagem) */
    window.addEventListener('keydown', function(e) {
      if (["ArrowUp","ArrowDown","ArrowLeft","ArrowRight"," "].indexOf(e.key) > -1) {
        e.preventDefault();
      }
    }, false);

    /* Inicializa e começa o loop */
    resetGame();
    requestAnimationFrame(loop);

    /* Observação sobre o comportamento pedido:
       - Durante os primeiros 300000 ms (5 minutos) a cobrinha não reinicia nem perde tamanho:
         -> elle faz wrap nas bordas e ignora colisões consigo mesma.
       - Após esse período, as colisões com paredes e consigo mesma voltam ao normal,
         e então um choque reiniciará o jogo (retornando tamanho inicial).
    */
  </script>
</body>
</html>
