# Урок 1. Установка окружения и запуск первого веб‑приложения (Flask **или** React) на Windows

> Формат: практика + чек‑листы.  
> Цель урока: у каждого студента **в конце урока** должен быть запущен **минимальный веб‑проект** (на выбор: Flask или React) и создан Git‑репозиторий.

---

## 0) Что понадобится (перед началом)

### Вариант A — Flask (Python)
- **Python 3.11+** (лучше 3.11/3.12)
- **pip** (идёт с Python)
- **Git**
- Редактор: VS Code / PyCharm

### Вариант B — React (Node.js)
- **Node.js 18+ LTS** (или 20 LTS)
- **npm** (идёт с Node) или **pnpm/yarn** (по желанию)
- **Git**
- Редактор: VS Code / WebStorm

---

## 1) Установка на Windows: Python / Node / Git

### 1.1 Установка Git
1) Установите Git for Windows (официальный установщик).  
2) В конце установки включите:
- “Git from the command line and also from 3rd‑party software”
- “Use OpenSSH”
3) Проверьте:
```bash
git --version
```

### 1.2 Установка Python (для Flask)
1) Установите Python 3.11+.
2) В установщике **обязательно** поставьте галочку:
- ✅ **Add python.exe to PATH**
3) Проверьте в **PowerShell**:
```bash
python --version
pip --version
```

> Если `python` не найден — см. раздел “Типовые проблемы”.

### 1.3 Установка Node.js (для React)
1) Установите Node.js **LTS**.
2) Проверьте:
```bash
node -v
npm -v
```

---

## 2) Быстрый старт проекта (на выбор)

## Вариант A — Flask: “Hello, Web!”
### 2.1 Создаём папку проекта
```bash
mkdir webapp-flask
cd webapp-flask
```

### 2.2 Создаём виртуальное окружение (venv)
```bash
python -m venv .venv
```

Активируем:

**PowerShell:**
```powershell
.\.venv\Scripts\Activate.ps1
```

**CMD:**
```cmd
.\.venv\Scripts\activate.bat
```

Проверьте, что `( .venv )` появился в начале строки терминала.

### 2.3 Устанавливаем Flask
```bash
pip install flask
pip freeze > requirements.txt
```

### 2.4 Мини‑приложение
Создайте файл `app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.get("/")
def home():
    return "Hello from Flask!"

if __name__ == "__main__":
    app.run(debug=True)
```

### 2.5 Запуск
```bash
python app.py
```

Откройте:
- http://127.0.0.1:5000

---

## Вариант B — React: “Hello, React!”
### 2.1 Создаём проект через Vite
```bash
mkdir webapp-react
cd webapp-react
npm create vite@latest .
```

Выберите:
- Framework: **React**
- Variant: **JavaScript** (или TypeScript — если хотите)

### 2.2 Установка зависимостей и запуск
```bash
npm install
npm run dev
```

Откройте ссылку из терминала (обычно):
- http://localhost:5173

---

## 3) Git: первый коммит

### 3.1 Инициализация репозитория
В корне проекта:
```bash
git init
```

### 3.2 .gitignore
**Для Flask** добавьте `.gitignore`:
```
.venv/
__pycache__/
*.pyc
.env
```

**Для React (Vite)**:
```
node_modules/
dist/
.env
```

### 3.3 Коммит
```bash
git add .
git commit -m "init: first web app"
```

---

## 4) Типовые проблемы и решения (Windows)

### Проблема 1: `python` не найден / запускается Microsoft Store
**Симптом:**
- `python` открывает Microsoft Store
- или “python is not recognized…”

**Решение:**
1) Проверьте, что Python установлен.
2) Отключите “App execution aliases”:
   - Settings → Apps → Advanced app settings → App execution aliases → отключить `python.exe`, `python3.exe`
3) Переоткройте терминал и проверьте:
```bash
where python
python --version
```

---

### Проблема 2: PowerShell не даёт активировать venv (ExecutionPolicy)
**Симптом:**
- “running scripts is disabled on this system”

**Решение (для текущего пользователя):**
Откройте PowerShell:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
Потом снова:
```powershell
.\.venv\Scripts\Activate.ps1
```

---

### Проблема 3: Порт занят (Flask 5000 / Vite 5173)
**Решение:**
- Закройте приложение, которое занимает порт, или используйте другой.

**Flask:**
```python
app.run(debug=True, port=5001)
```

**Vite:**
```bash
npm run dev -- --port 5174
```

---

### Проблема 4: `pip` ставит пакеты “не туда”
**Симптом:**
- Flask установлен, но `ModuleNotFoundError: flask`

**Решение:**
1) Убедитесь, что venv активирован.
2) Используйте `python -m pip`:
```bash
python -m pip install flask
```

---

### Проблема 5: Длинные пути / ошибки сборки node‑модулей
**Решение:**
- Не создавайте проект в глубокой папке типа:
  `C:\Users\...\Desktop\...\...\...`
- Создайте, например:
  `C:\dev\webapp-react`

---

## 6) Чек‑лист сдачи
- ✅ Проект запускается на ПК студента  
- ✅ Репозиторий инициализирован (`git log` показывает коммиты)  
- ✅ Есть `.gitignore`  
- ✅ Скрин: браузер + терминал со стартом проекта
