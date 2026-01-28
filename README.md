
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Моя первая Mini App</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacOSystemFont, 'Segoe UI', Roboto, sans-serif;
      margin: 0;
      padding: 20px;
      background: #0f0f0f;
      color: #ffffff;
      text-align: center;
    }
    button {
      background: #0088cc;
      color: white;
      border: none;
      padding: 16px 32px;
      font-size: 18px;
      border-radius: 12px;
      margin: 12px;
      cursor: pointer;
    }
    button:active { transform: scale(0.96); }
    #status { font-size: 1.3em; margin: 20px 0; }
  </style>
</head>
<body>

  <h2>Привет из Mini App!</h2>
  <div id="status">Загрузка...</div>
  
  <button id="btn">Кликни меня!</button>
  <div>Кликов: <span id="counter">0</span></div>

  <script>
    // Официальный скрипт Telegram
    const script = document.createElement('script');
    script.src = 'https://telegram.org/js/telegram-web-app.js';
    document.head.appendChild(script);

    script.onload = () => {
      const tg = window.Telegram.WebApp;
      tg.ready();
      tg.expand();           // на весь экран

      const user = tg.initDataUnsafe?.user;
      const status = document.getElementById('status');
      
      if (user) {
        status.innerHTML = `Привет, ${user.first_name || 'путешественник'}! 👋<br>
                           ID: ${user.id}`;
      } else {
        status.textContent = 'Не удалось получить данные пользователя 😔';
      }

      // Счётчик кликов
      let count = 0;
      document.getElementById('btn').onclick = () => {
        count++;
        document.getElementById('counter').textContent = count;
        tg.MainButton.setText('Кликнул ' + count + ' раз!').show();
      };

      // Главная кнопка внизу (как в нативных приложениях)
      tg.MainButton.setText('Пока просто тест').show();
      tg.MainButton.onClick(() => {
        tg.sendData('Кто-то нажал главную кнопку!');
        alert('Данные отправлены боту!');
      });
    };
  </script>

</body>
</html>