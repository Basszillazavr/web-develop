# Курс: Создание ИИ-приложения на Google Apps Script и Gemini

## Урок 1: Подключение ИИ к экосистеме Google
**Цель:** Создать мост между вашим Google-аккаунтом и нейросетью Gemini.

### 1. Регистрация в Google AI Studio
1. Зайдите на [Google AI Studio](https://aistudio.google.com/).
2. Нажмите **"Get API key"**, а затем **"Create API key in new project"**.
3. **Важно:** Сохраните этот ключ, он дает доступ к модели.

### 2. Настройка скрипта в Google Таблицах
Google Sheets будет выступать в роли нашей базы данных и панели управления.
1. Создайте новую таблицу.
2. Откройте **Расширения** -> **Apps Script**.
3. Вставьте код:

```javascript
const API_KEY = "ВАШ_КЛЮЧ";
const API_URL = "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent";

function askGemini(prompt) {
  const payload = {
    "contents": [{ "parts": [{ "text": prompt }] }]
  };
  
  const options = {
    "method": "post",
    "contentType": "application/json",
    "payload": JSON.stringify(payload)
  };

  const response = UrlFetchApp.fetch(`${API_URL}?key=${API_KEY}`, options);
  const result = JSON.parse(response.getContentText());
  return result.candidates[0].content.parts[0].text;
}
```

---

## Урок 2: Создание веб-интерфейса (Frontend)
**Цель:** Сделать так, чтобы ИИ работал не в таблице, а на отдельной веб-странице.

### 1. Создание HTML-файла
В редакторе Apps Script нажмите **+** -> **HTML** и назовите файл `index`.

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 20px; background: #f0f2f5; }
    .container { max-width: 600px; margin: auto; background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
    textarea { width: 100%; height: 100px; border: 1px solid #ddd; border-radius: 8px; padding: 10px; box-sizing: border-box; }
    button { background: #1a73e8; color: white; border: none; padding: 10px 20px; border-radius: 5px; cursor: pointer; margin-top: 10px; }
    #response { margin-top: 20px; padding: 15px; border-left: 4px solid #1a73e8; background: #f9f9f9; min-height: 50px; }
  </style>
</head>
<body>
  <div class="container">
    <h2>AI Assistant</h2>
    <textarea id="userInput" placeholder="Напишите ваш запрос здесь..."></textarea>
    <button onclick="callAi()">Отправить</button>
    <div id="response">Здесь будет ответ...</div>
  </div>

  <script>
    function callAi() {
      const prompt = document.getElementById('userInput').value;
      const display = document.getElementById('response');
      display.innerText = "Загрузка...";
      
      google.script.run
        .withSuccessHandler(res => display.innerText = res)
        .askGemini(prompt);
    }
  </script>
</body>
</html>
```

---

## Урок 3: Деплой и интеграция с сервисами
**Цель:** Опубликовать сайт в сети и научить его взаимодействовать с Google Drive или Gmail.

### 1. Публикация
1. В Apps Script нажмите **Начать развертывание** -> **Новое развертывание**.
2. Выберите тип: **Веб-приложение**.
3. Установите доступ: **Все (Anyone)**.
4. Нажмите "Развернуть" и скопируйте ссылку. Это и есть ваш готовый микро-сайт.

### 2. Пример интеграции (Запись в Таблицу)
Добавьте этот код в `Code.gs`, чтобы ИИ автоматически сохранял логи запросов:

```javascript
function askGemini(prompt) {
  // ... код запроса из 1-го урока ...
  const answer = result.candidates[0].content.parts[0].text;
  
  // Дополнительно: логируем в таблицу
  SpreadsheetApp.getActiveSpreadsheet().appendRow([new Date(), prompt, answer]);
  
  return answer;
}
```

---

## Для чего это нужно и как поможет в будущем?



### 1. Бесплатный бэкенд и хостинг
Обычно для сайта нужен сервер (Node.js, Python и т.д.) и хостинг. Google Apps Script дает это **бесплатно**. Это идеальный полигон для тестирования идей.

### 2. Глубокая интеграция с Google Services
Если твоя цель — сайт, работающий с Google-сервисами, то Apps Script — это «родная» среда. Из этого приложения ты можешь одной строчкой кода:
* Создавать документы в **Google Docs**.
* Отправлять письма через **Gmail**.
* Составлять расписание в **Google Calendar**.
* Читать и записывать данные в **Google Drive**.

### 3. Подготовка к «большому» сайту
Когда ты решишь создать профессиональный сайт на React/Next.js (в чем ты уже разбираешься):
* **Логика API останется прежней:** Ты уже будешь знать, как структурировать запросы к Gemini.
* **Apps Script как API:** Твой будущий сайт на React сможет обращаться к твоему текущему скрипту как к API (через `ContentService`), используя Google-таблицы как базу данных.

**Итог:** Сейчас ты учишься управлять «мозгами» ИИ внутри экосистемы, где находятся все данные пользователя (почта, таблицы, файлы). В будущем это позволит тебе создавать приложения, которые, например, автоматически анализируют входящие письма и составляют отчеты в таблицах.
