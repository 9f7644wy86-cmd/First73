<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"/>
  <title>Волейбол Mini App</title>
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <style>
    body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      background: #87CEEB;
      font-family: Arial, sans-serif;
      color: white;
      touch-action: none; /* для лучшего тача на мобиле */
    }
    #game-container {
      position: relative;
      width: 100vw;
      height: 100vh;
    }
    canvas {
      display: block;
      width: 100%;
      height: 100%;
      image-rendering: pixelated;
    }
    #ui {
      position: absolute;
      top: 10px;
      left: 0;
      right: 0;
      text-align: center;
      pointer-events: none;
      z-index: 10;
    }
    #score {
      font-size: 32px;
      margin: 10px;
      text-shadow: 2px 2px 4px #000;
    }
    #message {
      font-size: 24px;
      margin: 10px;
      text-shadow: 2px 2px 4px #000;
    }
    #controls {
      position: absolute;
      bottom: 20px;
      left: 0;
      right: 0;
      display: flex;
      justify-content: space-between;
      padding: 0 30px;
      pointer-events: auto;
      z-index: 10;
    }
    .btn {
      width: 80px;
      height: 80px;
      background: rgba(255,255,255,0.3);
      border: 3px solid white;
      border-radius: 50%;
      font-size: 40px;
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      user-select: none;
      touch-action: manipulation;
    }
    .btn:active {
      background: rgba(255,255,255,0.5);
    }
  </style>
</head>
<body>

<div id="game-container">
  <canvas id="canvas"></canvas>
  <div id="ui">
    <div id="score">0 : 0</div>
    <div id="message">Коснись экрана чтобы начать</div>
  </div>
  <div id="controls">
    <div class="btn" id="left">←</div>
    <div class="btn" id="jump">↑</div>
    <div class="btn" id="right">→</div>
  </div>
</div>

<script>
// Telegram WebApp
const tg = window.Telegram.WebApp;
tg.ready();
tg.expand();

// Canvas setup
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const scoreEl = document.getElementById('score');
const messageEl = document.getElementById('message');

// Адаптация под экран
function resize() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}
window.addEventListener('resize', resize);
resize();

// Константы
const GRAVITY = 0.38;
const NET_X = canvas.width / 2;
const NET_WIDTH = 12;
const NET_HEIGHT = canvas.height * 0.6;
const PLAYER_WIDTH = 60;
const PLAYER_HEIGHT = 100;
const BALL_RADIUS = 18;
const PLAYER_SPEED = 5.5;
const JUMP_POWER = -12;

// Объекты
let ball = { x: canvas.width/2, y: 200, vx: 0, vy: 0, lastHit: 'none' };
let player = { x: canvas.width*0.25, y: canvas.height-PLAYER_HEIGHT-20, vx: 0, jumping: false, score: 0 };
let computer = { x: canvas.width*0.75, y: canvas.height-PLAYER_HEIGHT-20, score: 0 };
let keys = {};
let gameRunning = false;
let serve = 'player'; // кто подаёт первым

// Управление (клавиатура + тач)
const leftBtn = document.getElementById('left');
const rightBtn = document.getElementById('right');
const jumpBtn = document.getElementById('jump');

function addControl(el, key) {
  el.addEventListener('touchstart', e => { e.preventDefault(); keys[key] = true; });
  el.addEventListener('touchend',   e => { e.preventDefault(); keys[key] = false; });
}
addControl(leftBtn,  'a');
addControl(rightBtn, 'd');
addControl(jumpBtn,  ' ');

// Клавиатура (для теста на ПК)
window.addEventListener('keydown', e => { keys[e.key.toLowerCase()] = true; });
window.addEventListener('keyup',   e => { keys[e.key.toLowerCase()] = false; });

// Старт игры по любому касанию/клику
canvas.addEventListener('touchstart', startGame, {once: true});
canvas.addEventListener('click', startGame, {once: true});

function startGame() {
  if (!gameRunning) {
    gameRunning = true;
    messageEl.textContent = '';
    resetBall();
    loop();
  }
}

function resetBall() {
  ball.x = serve === 'player' ? canvas.width*0.3 : canvas.width*0.7;
  ball.y = 150;
  ball.vx = serve === 'player' ? 3 : -3;
  ball.vy = -8;
  ball.lastHit = 'none';
}

function resetPositions() {
  player.x = canvas.width*0.25;
  computer.x = canvas.width*0.75;
}

// Главный цикл
function loop() {
  if (!gameRunning) return;

  update();
  draw();

  requestAnimationFrame(loop);
}

function update() {
  // Игрок
  player.vx = 0;
  if (keys['a'] || keys['arrowleft'])  player.vx = -PLAYER_SPEED;
  if (keys['d'] || keys['arrowright']) player.vx =  PLAYER_SPEED;

  player.x += player.vx;
  player.x = Math.max(0, Math.min(NET_X - PLAYER_WIDTH - 10, player.x));

  // Прыжок
  if ((keys[' '] || keys['w'] || keys['arrowup']) && !player.jumping) {
    player.vy = JUMP_POWER;
    player.jumping = true;
    tg.HapticFeedback.impactOccurred('medium');
  }

  if (player.jumping) {
    player.y += player.vy;
    player.vy += GRAVITY;
    if (player.y >= canvas.height - PLAYER_HEIGHT - 20) {
      player.y = canvas.height - PLAYER_HEIGHT - 20;
      player.jumping = false;
      player.vy = 0;
    }
  }

  // Компьютер (простой AI)
  let targetX = ball.x + ball.vx * 12; // предсказание
  if (ball.vx > 0) { // только когда мяч летит к нему
    if (computer.x + PLAYER_WIDTH/2 < targetX - 30) {
      computer.x += PLAYER_SPEED * 0.9;
    } else if (computer.x + PLAYER_WIDTH/2 > targetX + 30) {
      computer.x -= PLAYER_SPEED * 0.9;
    }
    computer.x = Math.max(NET_X + 10, Math.min(canvas.width - PLAYER_WIDTH, computer.x));
  }

  // Прыжок компьютера
  if (ball.y < computer.y + PLAYER_HEIGHT/2 && ball.vy > 0 && Math.abs(ball.x - computer.x - PLAYER_WIDTH/2) < 80) {
    if (!computer.jumping) {
      computer.vy = JUMP_POWER - 1.5; // чуть слабее игрока
      computer.jumping = true;
    }
  }
  if (computer.jumping) {
    computer.y += computer.vy;
    computer.vy += GRAVITY;
    if (computer.y >= canvas.height - PLAYER_HEIGHT - 20) {
      computer.y = canvas.height - PLAYER_HEIGHT - 20;
      computer.jumping = false;
      computer.vy = 0;
    }
  }

  // Мяч
  ball.x += ball.vx;
  ball.y += ball.vy;
  ball.vy += GRAVITY;

  // Удар игрока
  if (Math.abs(ball.x - (player.x + PLAYER_WIDTH/2)) < BALL_RADIUS + PLAYER_WIDTH/2 &&
      Math.abs(ball.y - (player.y + PLAYER_HEIGHT/2)) < BALL_RADIUS + PLAYER_HEIGHT/2) {
    ball.lastHit = 'player';
    ball.vx = (ball.x - (player.x + PLAYER_WIDTH/2)) * 0.25 + player.vx * 0.4;
    ball.vy = -Math.abs(ball.vy) * 0.8 - 4; // вверх
    tg.HapticFeedback.impactOccurred('light');
  }

  // Удар компьютера
  if (Math.abs(ball.x - (computer.x + PLAYER_WIDTH/2)) < BALL_RADIUS + PLAYER_WIDTH/2 &&
      Math.abs(ball.y - (computer.y + PLAYER_HEIGHT/2)) < BALL_RADIUS + PLAYER_HEIGHT/2) {
    ball.lastHit = 'computer';
    ball.vx = (ball.x - (computer.x + PLAYER_WIDTH/2)) * 0.22 + (Math.random()-0.5)*2;
    ball.vy = -Math.abs(ball.vy) * 0.75 - 3;
  }

  // Отскок от стен
  if (ball.x < BALL_RADIUS || ball.x > canvas.width - BALL_RADIUS) {
    ball.vx *= -0.8;
    ball.x = ball.x < BALL_RADIUS ? BALL_RADIUS : canvas.width - BALL_RADIUS;
  }

  // Пропуск мяча → очко
  if (ball.y > canvas.height + BALL_RADIUS*2) {
    if (ball.lastHit === 'player' || (ball.lastHit === 'none' && serve === 'computer')) {
      player.score++;
    } else {
      computer.score++;
    }

    scoreEl.textContent = `${player.score} : ${computer.score}`;

    if (player.score >= 5 || computer.score >= 5) {
      messageEl.textContent = player.score >= 5 ? 'Ты победил! 🎉' : 'Компьютер победил 😔';
      tg.HapticFeedback.notificationOccurred('success');
      gameRunning = false;
      setTimeout(() => {
        player.score = computer.score = 0;
        serve = serve === 'player' ? 'computer' : 'player';
        messageEl.textContent = 'Коснись экрана чтобы начать';
        canvas.addEventListener('touchstart', startGame, {once: true});
      }, 3000);
      return;
    }

    serve = ball.lastHit === 'player' ? 'computer' : 'player';
    resetBall();
    resetPositions();
  }

  // Сетка (простая коллизия)
  if (Math.abs(ball.x - NET_X) < BALL_RADIUS + NET_WIDTH/2 &&
      ball.y > canvas.height - NET_HEIGHT) {
    ball.vx *= -0.85;
    ball.x += ball.vx * 2;
  }
}

// Отрисовка
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Поле (песок)
  ctx.fillStyle = '#F4A460';
  ctx.fillRect(0, canvas.height - 120, canvas.width, 120);

  // Небо градиент
  const grd = ctx.createLinearGradient(0,0,0,canvas.height-120);
  grd.addColorStop(0, '#87CEEB');
  grd.addColorStop(1, '#E0F7FA');
  ctx.fillStyle = grd;
  ctx.fillRect(0, 0, canvas.width, canvas.height-120);

  // Сетка
  ctx.fillStyle = '#8B4513';
  ctx.fillRect(NET_X - NET_WIDTH/2, canvas.height - NET_HEIGHT - 20, NET_WIDTH, NET_HEIGHT);

  // Игрок (человек)
  ctx.fillStyle = '#00BFFF';
  ctx.fillRect(player.x, player.y, PLAYER_WIDTH, PLAYER_HEIGHT);
  ctx.fillStyle = '#FFFFFF';
  ctx.fillRect(player.x + 15, player.y + 20, PLAYER_WIDTH-30, 30); // голова/лицо

  // Компьютер
  ctx.fillStyle = '#FF4500';
  ctx.fillRect(computer.x, computer.y, PLAYER_WIDTH, PLAYER_HEIGHT);
  ctx.fillStyle = '#FFFFFF';
  ctx.fillRect(computer.x + 15, computer.y + 20, PLAYER_WIDTH-30, 30);

  // Мяч
  ctx.fillStyle = '#FFD700';
  ctx.beginPath();
  ctx.arc(ball.x, ball.y, BALL_RADIUS, 0, Math.PI*2);
  ctx.fill();
  ctx.strokeStyle = '#DAA520';
  ctx.lineWidth = 4;
  ctx.stroke();

  // Линия посередине
  ctx.strokeStyle = 'white';
  ctx.setLineDash([10, 15]);
  ctx.beginPath();
  ctx.moveTo(NET_X, 0);
  ctx.lineTo(NET_X, canvas.height);
  ctx.stroke();
  ctx.setLineDash([]);
}

// Запуск
resize();
draw(); // начальный экран
</script>
</body>
</html>