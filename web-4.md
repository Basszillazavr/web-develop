````md
# 📘 Урок 4. Формы и передача данных (React + Flask)

## 🎯 Цель занятия

Научиться:
- создавать формы в React;
- работать с useState;
- отправлять данные на сервер;
- принимать данные во Flask;
- выводить ответ на страницу.

---

## 🧱 Часть 1. Frontend (React)

### Создать файл:

src/pages/FormPage.jsx

---

## Код:

```jsx
import React, { useState } from "react"

export const FormPage = () => {
  const [name, setName] = useState("")
  const [email, setEmail] = useState("")
  const [message, setMessage] = useState("")
  const [response, setResponse] = useState("")

  const handleSubmit = async (e) => {
    e.preventDefault()

    try {
      const res = await fetch("http://localhost:5000/api/form", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          name,
          email,
          message,
        }),
      })

      const data = await res.json()
      setResponse(data.message)
    } catch (error) {
      setResponse("Ошибка отправки")
    }
  }

  return (
    <div style={{ maxWidth: 500, margin: "50px auto" }}>
      <h2>Форма обратной связи</h2>

      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Имя"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />

        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />

        <textarea
          placeholder="Сообщение"
          value={message}
          onChange={(e) => setMessage(e.target.value)}
        />

        <button type="submit">Отправить</button>
      </form>

      {response && <p>{response}</p>}
    </div>
  )
}
````

---

## 📌 Важно

* useState — хранит данные
* onChange — обновляет состояние
* fetch — отправляет данные
* preventDefault — убирает перезагрузку

---

## 🔥 Часть 2. Backend (Flask)

### Установка:

```bash
pip install flask flask-cors
```

---

## Создать файл:

app.py

---

## Код:

```python
from flask import Flask, request, jsonify
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

@app.route('/api/form', methods=['POST'])
def handle_form():
    data = request.get_json()

    name = data.get('name')
    email = data.get('email')
    message = data.get('message')

    return jsonify({
        "status": "success",
        "message": f"Спасибо, {name}! Мы получили сообщение"
    })

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 🔗 Запуск

Frontend:

```bash
npm run dev
```

Backend:

```bash
python app.py
```

---

## 🧪 Практика

### Задание 1

Добавить поле:

* телефон

---

### Задание 2

Добавить проверки:

* имя не пустое
* email содержит @

---

### Задание 3

Если ошибка:

```js
setResponse("Заполните все поля")
```

---

### Задание 4

Очистить форму после отправки:

```js
setName("")
setEmail("")
setMessage("")
```

---

## 🚀 Продвинутое

Добавить loading:

```js
const [loading, setLoading] = useState(false)
```

Кнопка:

```jsx
<button disabled={loading}>
  {loading ? "Отправка..." : "Отправить"}
</button>
```

---

Если хочешь, следующий сделаем **Урок 5 — CRUD (список + добавление + удаление)**, это уже прям уровень твоего SaaS 💰
```
