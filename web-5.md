# 📘 Урок 5-6. API + Авторизация (React + Flask)

## 🎯 Цель урока

Научиться:

* работать с API (GET / POST)
* соединять frontend и backend
* реализовать регистрацию и логин
* хранить пользователя в системе (простая auth)

---

# ⚛️ Часть 1. React + API

## 🔧 1. Базовая структура

```bash
src/
  App.jsx
  api.js
  pages/
    Login.jsx
    Register.jsx
    Home.jsx
```

---

## 🌐 2. Работа с API (api.js)

```js
const API_URL = "http://127.0.0.1:5000";

export const getPosts = async () => {
  const res = await fetch(`${API_URL}/posts`);
  return res.json();
};

export const addPost = async (title) => {
  return fetch(`${API_URL}/posts`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({ title })
  });
};
```

---

## 🏠 3. Главная страница (Home.jsx)

```jsx
import { useEffect, useState } from "react";
import { getPosts, addPost } from "../api";

export default function Home() {
  const [posts, setPosts] = useState([]);
  const [title, setTitle] = useState("");

  const fetchData = async () => {
    const data = await getPosts();
    setPosts(data);
  };

  useEffect(() => {
    fetchData();
  }, []);

  const handleAdd = async () => {
    await addPost(title);
    setTitle("");
    fetchData();
  };

  return (
    <div>
      <h1>Посты</h1>

      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
      />
      <button onClick={handleAdd}>Добавить</button>

      {posts.map((p) => (
        <div key={p.id}>{p.title}</div>
      ))}
    </div>
  );
}
```

---

# 🐍 Часть 2. Flask Backend

## 🔧 Установка

```bash
pip install flask flask-cors
```

---

## 🧠 Сервер (app.py)

```python
from flask import Flask, request, jsonify
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

posts = []
users = []

# POSTS
@app.route('/posts', methods=['GET'])
def get_posts():
    return jsonify(posts)

@app.route('/posts', methods=['POST'])
def add_post():
    data = request.json
    post = {
        "id": len(posts) + 1,
        "title": data["title"]
    }
    posts.append(post)
    return jsonify(post)

# AUTH
@app.route('/register', methods=['POST'])
def register():
    data = request.json

    user = {
        "id": len(users) + 1,
        "email": data["email"],
        "password": data["password"]
    }

    users.append(user)
    return jsonify({"message": "User created"})

@app.route('/login', methods=['POST'])
def login():
    data = request.json

    for user in users:
        if user["email"] == data["email"] and user["password"] == data["password"]:
            return jsonify({
                "message": "Success",
                "user": user
            })

    return jsonify({"message": "Invalid credentials"}), 401

if __name__ == "__main__":
    app.run(debug=True)
```

---

# 🔐 Часть 3. Авторизация (React)

## 📝 1. Регистрация (Register.jsx)

```jsx
import { useState } from "react";

export default function Register() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const handleSubmit = async () => {
    await fetch("http://127.0.0.1:5000/register", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ email, password })
    });

    alert("Регистрация успешна");
  };

  return (
    <div>
      <h1>Регистрация</h1>
      <input onChange={(e) => setEmail(e.target.value)} placeholder="email" />
      <input onChange={(e) => setPassword(e.target.value)} placeholder="password" />
      <button onClick={handleSubmit}>Создать</button>
    </div>
  );
}
```

---

## 🔑 2. Логин (Login.jsx)

```jsx
import { useState } from "react";

export default function Login() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const handleLogin = async () => {
    const res = await fetch("http://127.0.0.1:5000/login", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ email, password })
    });

    const data = await res.json();

    if (res.ok) {
      localStorage.setItem("user", JSON.stringify(data.user));
      alert("Успешный вход");
    } else {
      alert("Ошибка");
    }
  };

  return (
    <div>
      <h1>Логин</h1>
      <input onChange={(e) => setEmail(e.target.value)} placeholder="email" />
      <input onChange={(e) => setPassword(e.target.value)} placeholder="password" />
      <button onClick={handleLogin}>Войти</button>
    </div>
  );
}
```

---

## 👤 3. Проверка пользователя

```js
const user = JSON.parse(localStorage.getItem("user"));

if (user) {
  console.log("Пользователь вошел:", user.email);
}
```

---

# 🔒 Часть 4. Защита страниц

```jsx
const user = localStorage.getItem("user");

if (!user) {
  return <div>Нет доступа</div>;
}
```

---

# 💡 Важные моменты

* сейчас пароль хранится ❌ небезопасно (для обучения норм)
* в реальности:

  * хеширование (bcrypt)
  * JWT токены
  * refresh tokens

---
