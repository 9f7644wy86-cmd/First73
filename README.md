<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"/>
  <title>Генератор мемов — Mini App</title>
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <style>
    body {
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      background: var(--tg-theme-bg-color, #000);
      color: var(--tg-theme-text-color, #fff);
      min-height: 100vh;
      overflow-x: hidden;
    }
    .container {
      padding: 16px;
      max-width: 600px;
      margin: 0 auto;
    }
    h1 {
      text-align: center;
      margin: 12px 0;
      font-size: 24px;
    }
    canvas {
      width: 100%;
      border: 2px solid var(--tg-theme-button-color, #0088cc);
      border-radius: 12px;
      background: #222;
      display: block;
      margin: 16px auto;
    }
    .controls {
      display: grid;
      gap: 12px;
    }
    input, select, button {
      padding: 12px;
      font-size: 16px;
      border-radius: 10px;
      border: 1px solid #444;
      background: var(--tg-theme-bg-color, #111);
      color: var(--tg-theme-text-color, #fff);
    }
    button {
      background: var(--tg-theme-button-color, #0088cc);
      color: white;
      border: none;
      font-weight: bold;
      cursor: pointer;
    }
    button:active {
      opacity: 0.8;
    }
    .row {
      display: flex;
      gap: 12px;
    }
    .row > * { flex: 1; }
    .templates {
      display: flex;
      overflow-x: auto;
      gap: 12px;
      padding: 12px 0;
    }
    .template-thumb {
      width: 100px;
      height: 100px;
      object-fit: cover;
      border-radius: 8px;
      border: 2px solid transparent;
      cursor: pointer;
    }
    .template-thumb.active {
      border-color: #00ff88;
    }
    #download {
      margin-top: 16px;
      width: 100%;
    }
  </style>
</head>
<body>

<div class="container">
  <h1>Генератор мемов</h1>

  <div class="templates">
    <img class="template-thumb active" src="https://i.imgflip.com/1bij.jpg" data-src="https://i.imgflip.com/1bij.jpg" alt="Drake">
    <img class="template-thumb" src="https://i.imgflip.com/30b1gy.jpg" data-src="https://i.imgflip.com/30b1gy.jpg" alt="Distracted Boyfriend">
    <img class="template-thumb" src="https://i.imgflip.com/1ur9b0.jpg" data-src="https://i.imgflip.com/1ur9b0.jpg" alt="Expanding Brain">
    <img class="template-thumb" src="https://i.imgflip.com/1g8my4.jpg" data-src="https://i.imgflip.com/1g8my4.jpg" alt="Two Buttons">
  </div>

  <canvas id="memeCanvas" width="500" height="500"></canvas>

  <div class="controls">
    <input type="file" id="imageUpload" accept="image/*">

    <div class="row">
      <input type="text" id="topText" placeholder="Верхний текст" maxlength="60">
      <input type="text" id="bottomText" placeholder="Нижний текст" maxlength="60">
    </div>

    <div class="row">
      <select id="fontSize">
        <option value="36">Размер 36</option>
        <option value="48" selected>Размер 48</option>
        <option value="60">Размер 60</option>
        <option value="72">Размер 72</option>
      </select>
      <select id="fontColor">
        <option value="white">Белый</option>
        <option value="black">Чёрный</option>
        <option value="yellow">Жёлтый</option>
        <option value="#ff0000">Красный</option>
      </select>
    </div>

    <button id="generate">Обновить мем</button>
    <button id="download">Скачать мем</button>
  </div>
</div>

<script>
const tg = window.Telegram.WebApp;
tg.ready();
tg.expand();
tg.MainButton.setText("Поделиться мемом").hide();

const canvas = document.getElementById('memeCanvas');
const ctx = canvas.getContext('2d');
let currentImage = new Image();

const defaultImg = 'https://i.imgflip.com/1bij.jpg';
currentImage.crossOrigin = "anonymous";
currentImage.src = defaultImg;

function drawMeme() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Рисуем изображение (масштабируем под canvas)
  const ratio = Math.min(canvas.width / currentImage.width, canvas.height / currentImage.height);
  const w = currentImage.width * ratio;
  const h = currentImage.height * ratio;
  const x = (canvas.width - w) / 2;
  const y = (canvas.height - h) / 2;
  ctx.drawImage(currentImage, x, y, w, h);

  // Текст настройки
  const top = document.getElementById('topText').value.toUpperCase();
  const bottom = document.getElementById('bottomText').value.toUpperCase();
  const fontSize = parseInt(document.getElementById('fontSize').value);
  const color = document.getElementById('fontColor').value;

  ctx.font = `bold ${fontSize}px Impact, Arial, sans-serif`;
  ctx.fillStyle = color;
  ctx.strokeStyle = 'black';
  ctx.lineWidth = fontSize / 15;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';

  // Верхний текст
  if (top) {
    ctx.strokeText(top, canvas.width/2, fontSize * 1.2);
    ctx.fillText(top, canvas.width/2, fontSize * 1.2);
  }

  // Нижний текст
  if (bottom) {
    ctx.strokeText(bottom, canvas.width/2, canvas.height - fontSize * 0.8);
    ctx.fillText(bottom, canvas.width/2, canvas.height - fontSize * 0.8);
  }
}

// Загрузка своей картинки
document.getElementById('imageUpload').addEventListener('change', e => {
  const file = e.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = ev => {
      currentImage.src = ev.target.result;
      currentImage.onload = drawMeme;
    };
    reader.readAsDataURL(file);
  }
});

// Шаблоны
document.querySelectorAll('.template-thumb').forEach(img => {
  img.addEventListener('click', () => {
    document.querySelectorAll('.template-thumb').forEach(i => i.classList.remove('active'));
    img.classList.add('active');
    currentImage.src = img.dataset.src;
    currentImage.onload = drawMeme;
  });
});

// Обновление при вводе
['topText', 'bottomText', 'fontSize', 'fontColor'].forEach(id => {
  document.getElementById(id).addEventListener('input', drawMeme);
});

document.getElementById('generate').addEventListener('click', drawMeme);

// Скачивание
document.getElementById('download').addEventListener('click', () => {
  const link = document.createElement('a');
  link.download = 'мой_мем.png';
  link.href = canvas.toDataURL('image/png');
  link.click();
  tg.HapticFeedback.notificationOccurred('success');
});

// Первая отрисовка
currentImage.onload = drawMeme;

// Telegram интеграция
tg.MainButton.onClick(() => {
  canvas.toBlob(blob => {
    const file = new File([blob], "мем.png", { type: "image/png" });
    tg.sendData(JSON.stringify({ action: "share_meme" })); // можно передать данные боту
    tg.close(); // или оставить открытым
  }, 'image/png');
});

// Автоматически показываем кнопку поделиться после генерации
document.getElementById('generate').addEventListener('click', () => {
  tg.MainButton.show();
});
</script>
</body>
</html>