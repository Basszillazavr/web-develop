# 📘 Урок 7. JWT Авторизация + Роли (React + Flask)

## 🎯 Цель урока

Научиться:

* работать с JWT токенами
* защищать API
* добавлять роли (user / admin)
* делать авторизацию как в проде

---

# 🐍 Часть 1. Flask (JWT)

## 🔧 Установка

```bash
pip install flask flask-cors pyjwt bcrypt
```

---

## 🧠 Обновляем backend

```python
from flask import Flask, request, jsonify
from flask_cors import CORS
import jwt
import datetime
import bcrypt

app = Flask(__name__)
CORS(app)

SECRET_KEY = "secret123"

users = []

# REGISTER
@app.route('/register', methods=['POST'])
def register():
    data = request.json

    hashed_password = bcrypt.hashpw(
        data["password"].encode('utf-8'),
        bcrypt.gensalt()
    )

    user = {
        "id": len(users) + 1,
        "email": data["email"],
        "password": hashed_password,
        "role": "user"
    }

    users.append(user)
    return jsonify({"message": "User created"})

# LOGIN
@app.route('/login', methods=['POST'])
def login():
    data = request.json

    for user in users:
        if user["email"] == data["email"]:
            if bcrypt.checkpw(data["password"].encode('utf-8'), user["password"]):

                token = jwt.encode({
                    "user_id": user["id"],
                    "role": user["role"],
                    "exp": datetime.datetime.utcnow() + datetime.timedelta(hours=2)
                }, SECRET_KEY, algorithm="HS256")

                return jsonify({
                    "token": token
                })

    return jsonify({"message": "Invalid credentials"}), 401
```

---

# 🔐 Часть 2. Middleware (защита маршрутов)

```python
from functools import wraps

def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get("Authorization")

        if not token:
            return jsonify({"message": "Token missing"}), 401

        try:
            data = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        except:
            return jsonify({"message": "Token invalid"}), 401

        return f(data, *args, **kwargs)

    return decorated
```

---

## 🔒 Пример защищенного route

```python
@app.route('/profile', methods=['GET'])
@token_required
def profile(user_data):
    return jsonify({
        "message": "Welcome",
        "user": user_data
    })
```

---

# 👑 Часть 3. Роли (admin / user)

```python
def admin_required(f):
    @wraps(f)
    def decorated(user_data, *args, **kwargs):
        if user_data["role"] != "admin":
            return jsonify({"message": "Admin only"}), 403
        return f(user_data, *args, **kwargs)

    return decorated
```

---

## 👑 Пример admin route

```python
@app.route('/admin', methods=['GET'])
@token_required
@admin_required
def admin(user_data):
    return jsonify({"message": "Hello Admin"})
```

---

# ⚛️ Часть 4. React (JWT)

## 🔑 Логин

```jsx
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
    localStorage.setItem("token", data.token);
  }
};
```

---

## 📡 Запрос с токеном

```js
const token = localStorage.getItem("token");

await fetch("http://127.0.0.1:5000/profile", {
  headers: {
    Authorization: token
  }
});
```

---

# 🔐 Часть 5. Protected Route (React)

```jsx
const token = localStorage.getItem("token");

if (!token) {
  return <div>Нет доступа</div>;
}
```

---

# 💡 Важные моменты

* JWT хранит:

  * user_id
  * role
* token имеет срок жизни (exp)
* нельзя хранить секрет в фронте ❌

---

# ⚠️ Частые ошибки

* забыли передать Authorization
* токен протух
* не проверили role
* нет CORS

---
