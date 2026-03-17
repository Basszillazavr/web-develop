````md
# 📘 Урок 4. Формы и передача данных

Сегодня продолжаем практику по разработке веб-приложений.  
Тема занятия — формы и передача данных.

Сделаем 2 отдельных варианта:

1. React + JSON Server  
2. Flask + SQLite / запись в файл

---

# Часть 1. React + JSON Server

## Цель

Научиться:
- создавать форму в React;
- хранить значения полей через `useState`;
- отправлять данные через `fetch`;
- сохранять данные в `json-server`.

---

## Что будем делать

Сделаем форму обратной связи с полями:
- имя
- email
- сообщение

После отправки данные будут сохраняться в `db.json`.

---

## Шаг 1. Установка проекта

Если проект React уже создан, пропускаем.

Если нет:

```bash
npx create-react-app lesson-forms
cd lesson-forms
npm install json-server
````

---

## Шаг 2. Создать файл базы

В корне проекта создать файл:

```bash
db.json
```

Содержимое:

```json
{
  "messages": []
}
```

---

## Шаг 3. Запуск JSON Server

```bash
npx json-server --watch db.json --port 5000
```

Теперь API будет доступен по адресу:

```bash
http://localhost:5000/messages
```

---

## Шаг 4. Создать страницу с формой

Создать файл:

```bash
src/FormPage.jsx
```

Код:

```jsx
import React, { useState } from "react";

export default function FormPage() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [message, setMessage] = useState("");
  const [status, setStatus] = useState("");

  const handleSubmit = async (e) => {
    e.preventDefault();

    if (!name || !email || !message) {
      setStatus("Заполните все поля");
      return;
    }

    try {
      const response = await fetch("http://localhost:5000/messages", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          name,
          email,
          message,
          createdAt: new Date().toISOString()
        })
      });

      if (!response.ok) {
        throw new Error("Ошибка при сохранении");
      }

      setStatus("Данные успешно отправлены");
      setName("");
      setEmail("");
      setMessage("");
    } catch (error) {
      setStatus("Ошибка сервера");
    }
  };

  return (
    <div style={{ maxWidth: "500px", margin: "40px auto" }}>
      <h2>Форма обратной связи</h2>

      <form onSubmit={handleSubmit}>
        <div>
          <label>Имя</label>
          <br />
          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
            placeholder="Введите имя"
          />
        </div>

        <div style={{ marginTop: "12px" }}>
          <label>Email</label>
          <br />
          <input
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            placeholder="Введите email"
          />
        </div>

        <div style={{ marginTop: "12px" }}>
          <label>Сообщение</label>
          <br />
          <textarea
            value={message}
            onChange={(e) => setMessage(e.target.value)}
            placeholder="Введите сообщение"
            rows="5"
          />
        </div>

        <button type="submit" style={{ marginTop: "16px" }}>
          Отправить
        </button>
      </form>

      {status && <p style={{ marginTop: "16px" }}>{status}</p>}
    </div>
  );
}
```

---

## Шаг 5. Подключить страницу

В файле:

```bash
src/App.js
```

Написать:

```jsx
import React from "react";
import FormPage from "./FormPage";

function App() {
  return <FormPage />;
}

export default App;
```

---

## Что важно понять

### `useState`

Используется для хранения значения поля.

Пример:

```jsx
const [name, setName] = useState("");
```

---

### `onChange`

Срабатывает при вводе текста.

```jsx
onChange={(e) => setName(e.target.value)}
```

---

### `fetch`

Используется для отправки данных на сервер.

```jsx
await fetch("http://localhost:5000/messages", {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({...})
})
```

---

## Практика

### Задание 1

Добавить поле:

* телефон

### Задание 2

Добавить проверку:

* email должен содержать `@`

### Задание 3

После отправки выводить список всех сообщений

Подсказка:

```jsx
useEffect(() => {
  fetch("http://localhost:5000/messages")
    .then((res) => res.json())
    .then((data) => console.log(data));
}, []);
```

### Задание 4

Добавить кнопку удаления сообщения

---

## Домашнее задание

Сделать форму:

* имя
* email
* курс
* комментарий

Сохранять данные в `json-server`.

---

# Часть 2. Flask + SQLite + запись в файл

## Цель

Научиться:

* создавать форму на Flask;
* принимать данные из формы;
* сохранять данные в базу SQLite;
* дополнительно записывать данные в текстовый файл.

---

## Что будем делать

Сделаем страницу с HTML-формой.
После отправки:

* данные сохраняются в SQLite;
* данные записываются в файл `messages.txt`.

---

## Шаг 1. Установка Flask

```bash
pip install flask
```

---

## Шаг 2. Создать файл проекта

Создать файл:

```bash
app.py
```

---

## Шаг 3. Полный код Flask

```python
from flask import Flask, request, render_template_string, redirect
import sqlite3

app = Flask(__name__)

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Форма обратной связи</title>
</head>
<body>
    <h2>Форма обратной связи</h2>

    <form method="POST" action="/submit">
        <div>
            <label>Имя:</label><br>
            <input type="text" name="name" required>
        </div>

        <div style="margin-top: 10px;">
            <label>Email:</label><br>
            <input type="email" name="email" required>
        </div>

        <div style="margin-top: 10px;">
            <label>Сообщение:</label><br>
            <textarea name="message" rows="5" required></textarea>
        </div>

        <button type="submit" style="margin-top: 15px;">Отправить</button>
    </form>

    {% if success %}
        <p style="color: green; margin-top: 20px;">Данные успешно сохранены</p>
    {% endif %}
</body>
</html>
"""

def init_db():
    conn = sqlite3.connect("messages.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS messages (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT NOT NULL,
            message TEXT NOT NULL
        )
    """)
    conn.commit()
    conn.close()

@app.route("/")
def index():
    return render_template_string(HTML_TEMPLATE, success=False)

@app.route("/submit", methods=["POST"])
def submit():
    name = request.form.get("name")
    email = request.form.get("email")
    message = request.form.get("message")

    conn = sqlite3.connect("messages.db")
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO messages (name, email, message) VALUES (?, ?, ?)",
        (name, email, message)
    )
    conn.commit()
    conn.close()

    with open("messages.txt", "a", encoding="utf-8") as file:
        file.write(f"Имя: {name}\n")
        file.write(f"Email: {email}\n")
        file.write(f"Сообщение: {message}\n")
        file.write("-" * 30 + "\n")

    return render_template_string(HTML_TEMPLATE, success=True)

if __name__ == "__main__":
    init_db()
    app.run(debug=True)
```

---

## Как это работает

### `render_template_string`

Показывает HTML прямо из Python-кода.

### `request.form.get(...)`

Получает значения из формы.

Пример:

```python
name = request.form.get("name")
```

### SQLite

Используем встроенную базу Python.
Это один из самых простых вариантов для первых проектов.

### Запись в файл

Дополнительно все заявки сохраняются в обычный текстовый файл:

```python
with open("messages.txt", "a", encoding="utf-8") as file:
```

---

## Шаг 4. Запуск проекта

```bash
python app.py
```

Открыть в браузере:

```bash
http://127.0.0.1:5000
```

---

## Что создастся после запуска

### База:

```bash
messages.db
```

### Текстовый файл:

```bash
messages.txt
```

---

## Практика

### Задание 1

Добавить поле:

* телефон

Для этого:

* добавить `<input>`
* добавить колонку в базу
* сохранять телефон в файл

### Задание 2

Сделать страницу со списком всех заявок

Подсказка:

```python
@app.route("/messages")
def messages():
    conn = sqlite3.connect("messages.db")
    cursor = conn.cursor()
    cursor.execute("SELECT id, name, email, message FROM messages")
    rows = cursor.fetchall()
    conn.close()
    return str(rows)
```

### Задание 3

Сделать кнопку удаления записи по `id`

### Задание 4

Разделить HTML и Python:

* HTML вынести в папку `templates`
* Flask оставить только для логики

# Итог урока

## React + JSON Server

* создавать управляемую форму;
* отправлять POST-запрос;
* сохранять данные в `db.json`.

## Flask + SQLite

* создавать HTML-форму;
* принимать POST-данные;
* сохранять информацию в базу;
* записывать данные в файл.

---

# Следующий урок

Практика 5:

* React: список данных, удаление, редактирование
* Flask: вывод записей из базы, просмотр и удаление заявок

```

Могу сразу сделать и **Практику 5** в таком же `.md` формате.
```
