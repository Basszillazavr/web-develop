````md
# Решение практического занятия “TaskHub”
Ниже я даю **референс-решение** для всех треков:
- **React (JS)** — работает в 2 режимах: `localStorage` и `API`
- **Flask (Python)** — REST API `/api/tasks` + CORS
- **Full-stack** — React ↔ Flask

> Можно копировать как есть. Если у вас уже другой шаблон — переносите логику и endpoints 1:1.

---

# ✅ Трек A — React (JavaScript) решение (Vite)

## 1) Создание проекта
```bash
npm create vite@latest taskhub-react -- --template react
cd taskhub-react
npm i
npm run dev
````

## 2) Структура

```
src/
  api/
    tasksApi.js
  components/
    TaskForm.jsx
    TaskFilters.jsx
    TaskItem.jsx
    TaskList.jsx
  utils/
    storage.js
  App.jsx
  main.jsx
  styles.css
```

---

## 3) Настройка режима данных

### Вариант A: localStorage (по умолчанию)

Ничего дополнительно не нужно.

### Вариант B: API (Flask)

Создайте файл `.env` в корне `taskhub-react`:

```env
VITE_DATA_MODE=api
VITE_API_BASE=http://127.0.0.1:5000
```

---

## 4) Код решения

### `src/utils/storage.js`

```js
const KEY = "taskhub.tasks.v1";

export function loadTasks() {
  try {
    const raw = localStorage.getItem(KEY);
    return raw ? JSON.parse(raw) : [];
  } catch {
    return [];
  }
}

export function saveTasks(tasks) {
  localStorage.setItem(KEY, JSON.stringify(tasks));
}
```

### `src/api/tasksApi.js`

```js
const MODE = import.meta.env.VITE_DATA_MODE || "local";
const API_BASE = import.meta.env.VITE_API_BASE || "";

function nowIso() {
  return new Date().toISOString();
}

function uuid() {
  return crypto.randomUUID ? crypto.randomUUID() : String(Date.now()) + Math.random();
}

// ------- LOCAL (localStorage) -------
import { loadTasks, saveTasks } from "../utils/storage";

async function localList() {
  return loadTasks();
}
async function localCreate(title) {
  const tasks = loadTasks();
  const task = { id: uuid(), title, status: "todo", createdAt: nowIso() };
  const next = [task, ...tasks];
  saveTasks(next);
  return task;
}
async function localUpdate(id, patch) {
  const tasks = loadTasks();
  const idx = tasks.findIndex(t => t.id === id);
  if (idx === -1) throw new Error("NOT_FOUND");
  const updated = { ...tasks[idx], ...patch };
  const next = tasks.slice();
  next[idx] = updated;
  saveTasks(next);
  return updated;
}
async function localRemove(id) {
  const tasks = loadTasks();
  const next = tasks.filter(t => t.id !== id);
  saveTasks(next);
  return true;
}

// ------- API (Flask) -------
async function apiFetch(path, options) {
  const res = await fetch(`${API_BASE}${path}`, {
    headers: { "Content-Type": "application/json", ...(options?.headers || {}) },
    ...options,
  });

  const isJson = (res.headers.get("content-type") || "").includes("application/json");
  const data = isJson ? await res.json() : null;

  if (!res.ok) {
    const code = data?.error || res.statusText || "REQUEST_FAILED";
    const err = new Error(code);
    err.status = res.status;
    throw err;
  }
  return data;
}

async function apiList() {
  const data = await apiFetch(`/api/tasks`, { method: "GET" });
  return data.data;
}
async function apiCreate(title) {
  const data = await apiFetch(`/api/tasks`, {
    method: "POST",
    body: JSON.stringify({ title }),
  });
  return data.data;
}
async function apiUpdate(id, patch) {
  const data = await apiFetch(`/api/tasks/${id}`, {
    method: "PATCH",
    body: JSON.stringify(patch),
  });
  return data.data;
}
async function apiRemove(id) {
  await apiFetch(`/api/tasks/${id}`, { method: "DELETE" });
  return true;
}

// ------- PUBLIC API -------
export const tasksApi = {
  mode: MODE,
  list: () => (MODE === "api" ? apiList() : localList()),
  create: (title) => (MODE === "api" ? apiCreate(title) : localCreate(title)),
  update: (id, patch) => (MODE === "api" ? apiUpdate(id, patch) : localUpdate(id, patch)),
  remove: (id) => (MODE === "api" ? apiRemove(id) : localRemove(id)),
};
```

### `src/components/TaskForm.jsx`

```jsx
import React, { useState } from "react";

export default function TaskForm({ onAdd, loading }) {
  const [title, setTitle] = useState("");
  const [error, setError] = useState("");

  const submit = async (e) => {
    e.preventDefault();
    const v = title.trim();
    if (!v) {
      setError("Название задачи обязательно");
      return;
    }
    setError("");
    await onAdd(v);
    setTitle("");
  };

  return (
    <form onSubmit={submit} className="card">
      <div className="row">
        <input
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          placeholder="Новая задача..."
          disabled={loading}
        />
        <button disabled={loading}>{loading ? "..." : "Добавить"}</button>
      </div>
      {error ? <div className="error">{error}</div> : null}
    </form>
  );
}
```

### `src/components/TaskFilters.jsx`

```jsx
import React from "react";

export default function TaskFilters({ query, setQuery, status, setStatus, loading }) {
  return (
    <div className="card">
      <div className="row">
        <input
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Поиск..."
          disabled={loading}
        />
        <select value={status} onChange={(e) => setStatus(e.target.value)} disabled={loading}>
          <option value="all">Все</option>
          <option value="todo">Todo</option>
          <option value="doing">Doing</option>
          <option value="done">Done</option>
        </select>
      </div>
    </div>
  );
}
```

### `src/components/TaskItem.jsx`

```jsx
import React from "react";

const STATUSES = ["todo", "doing", "done"];

export default function TaskItem({ task, onChangeStatus, onDelete, loading }) {
  const nextStatus = () => {
    const i = STATUSES.indexOf(task.status);
    return STATUSES[(i + 1) % STATUSES.length];
  };

  return (
    <div className="task">
      <div className="task-main">
        <div className="task-title">{task.title}</div>
        <div className="task-meta">
          <span className={`badge ${task.status}`}>{task.status}</span>
          <span className="muted">{new Date(task.createdAt).toLocaleString()}</span>
        </div>
      </div>

      <div className="task-actions">
        <button
          className="secondary"
          disabled={loading}
          onClick={() => onChangeStatus(task.id, nextStatus())}
        >
          Следующий статус
        </button>
        <button className="danger" disabled={loading} onClick={() => onDelete(task.id)}>
          Удалить
        </button>
      </div>
    </div>
  );
}
```

### `src/components/TaskList.jsx`

```jsx
import React from "react";
import TaskItem from "./TaskItem";

export default function TaskList({ tasks, onChangeStatus, onDelete, loading }) {
  if (loading) return <div className="card">Загрузка...</div>;
  if (!tasks.length) return <div className="card">Список пуст</div>;

  return (
    <div className="card">
      {tasks.map((t) => (
        <TaskItem
          key={t.id}
          task={t}
          onChangeStatus={onChangeStatus}
          onDelete={onDelete}
          loading={loading}
        />
      ))}
    </div>
  );
}
```

### `src/App.jsx`

```jsx
import React, { useEffect, useMemo, useState } from "react";
import { tasksApi } from "./api/tasksApi";
import TaskForm from "./components/TaskForm";
import TaskFilters from "./components/TaskFilters";
import TaskList from "./components/TaskList";
import "./styles.css";

export default function App() {
  const [tasks, setTasks] = useState([]);
  const [query, setQuery] = useState("");
  const [status, setStatus] = useState("all");
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");

  const load = async () => {
    setLoading(true);
    setError("");
    try {
      const data = await tasksApi.list();
      // сортировка: новые сверху
      data.sort((a, b) => (a.createdAt < b.createdAt ? 1 : -1));
      setTasks(data);
    } catch (e) {
      setError(`Ошибка: ${e.message}`);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    load();
  }, []);

  const filtered = useMemo(() => {
    const q = query.trim().toLowerCase();
    return tasks.filter((t) => {
      const okQuery = !q || t.title.toLowerCase().includes(q);
      const okStatus = status === "all" || t.status === status;
      return okQuery && okStatus;
    });
  }, [tasks, query, status]);

  const addTask = async (title) => {
    setLoading(true);
    setError("");
    try {
      const created = await tasksApi.create(title);
      setTasks((prev) => [created, ...prev]);
    } catch (e) {
      setError(`Ошибка: ${e.message}`);
    } finally {
      setLoading(false);
    }
  };

  const changeStatus = async (id, next) => {
    setLoading(true);
    setError("");
    try {
      const updated = await tasksApi.update(id, { status: next });
      setTasks((prev) => prev.map((t) => (t.id === id ? updated : t)));
    } catch (e) {
      setError(`Ошибка: ${e.message}`);
    } finally {
      setLoading(false);
    }
  };

  const removeTask = async (id) => {
    setLoading(true);
    setError("");
    try {
      await tasksApi.remove(id);
      setTasks((prev) => prev.filter((t) => t.id !== id));
    } catch (e) {
      setError(`Ошибка: ${e.message}`);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="wrap">
      <header className="header">
        <h1>TaskHub</h1>
        <div className="muted">Режим: {tasksApi.mode}</div>
      </header>

      {error ? <div className="card error">{error}</div> : null}

      <TaskForm onAdd={addTask} loading={loading} />
      <TaskFilters
        query={query}
        setQuery={setQuery}
        status={status}
        setStatus={setStatus}
        loading={loading}
      />
      <TaskList tasks={filtered} onChangeStatus={changeStatus} onDelete={removeTask} loading={loading} />
    </div>
  );
}
```

### `src/styles.css`

```css
:root { font-family: system-ui, Arial; }
body { margin: 0; background: #f6f7fb; }
.wrap { max-width: 900px; margin: 24px auto; padding: 0 16px; }
.header { display:flex; justify-content:space-between; align-items:baseline; margin-bottom: 16px; }
.card { background:#fff; border:1px solid #e6e8f0; border-radius:12px; padding:12px; margin-bottom:12px; }
.row { display:flex; gap: 10px; }
input, select { flex:1; padding:10px; border:1px solid #d8dbea; border-radius:10px; }
button { padding:10px 12px; border-radius:10px; border:1px solid #2b2f3a; background:#2b2f3a; color:#fff; cursor:pointer; }
button.secondary { background:#fff; color:#2b2f3a; }
button.danger { border-color:#b42318; background:#b42318; }
button:disabled { opacity:.6; cursor:not-allowed; }
.task { display:flex; justify-content:space-between; gap:12px; padding:10px 0; border-top:1px solid #eef0f6; }
.task:first-child { border-top:none; }
.task-title { font-weight: 600; }
.task-meta { display:flex; gap:10px; align-items:center; }
.badge { font-size:12px; padding:2px 8px; border-radius:999px; border:1px solid #ddd; }
.badge.todo { background:#fff; }
.badge.doing { background:#fff7e6; border-color:#ffd591; }
.badge.done { background:#f6ffed; border-color:#b7eb8f; }
.muted { color:#667085; font-size: 13px; }
.error { color:#b42318; }
```

---

## 5) Запуск React

```bash
npm run dev
```

---

# ✅ Трек B — Flask (Python) решение (REST API + CORS)

## 1) Структура

```
taskhub-flask/
  app.py
  requirements.txt
```

## 2) `requirements.txt`

```txt
Flask==3.0.0
flask-cors==4.0.0
```

## 3) `app.py`

```py
from flask import Flask, request, jsonify
from flask_cors import CORS
from datetime import datetime, timezone
import uuid

app = Flask(__name__)
CORS(app)  # для React (localhost)

# In-memory storage: id -> task
TASKS = {}

def now_iso():
    return datetime.now(timezone.utc).isoformat()

def task_to_json(t):
    return {
        "id": t["id"],
        "title": t["title"],
        "status": t["status"],
        "createdAt": t["createdAt"],
    }

@app.get("/api/tasks")
def list_tasks():
    data = list(TASKS.values())
    # сортировка: новые сверху
    data.sort(key=lambda x: x["createdAt"], reverse=True)
    return jsonify({"data": [task_to_json(t) for t in data]})

@app.post("/api/tasks")
def create_task():
    body = request.get_json(silent=True) or {}
    title = (body.get("title") or "").strip()

    if not title:
        return jsonify({"error": "TITLE_REQUIRED"}), 400

    task_id = str(uuid.uuid4())
    task = {
        "id": task_id,
        "title": title,
        "status": "todo",
        "createdAt": now_iso(),
    }
    TASKS[task_id] = task
    return jsonify({"data": task_to_json(task)}), 201

@app.patch("/api/tasks/<task_id>")
def update_task(task_id):
    if task_id not in TASKS:
        return jsonify({"error": "NOT_FOUND"}), 404

    body = request.get_json(silent=True) or {}
    title = body.get("title", None)
    status = body.get("status", None)

    if title is not None:
        title = str(title).strip()
        if not title:
            return jsonify({"error": "TITLE_REQUIRED"}), 400
        TASKS[task_id]["title"] = title

    if status is not None:
        if status not in ("todo", "doing", "done"):
            return jsonify({"error": "BAD_STATUS"}), 400
        TASKS[task_id]["status"] = status

    return jsonify({"data": task_to_json(TASKS[task_id])})

@app.delete("/api/tasks/<task_id>")
def delete_task(task_id):
    if task_id not in TASKS:
        return jsonify({"error": "NOT_FOUND"}), 404
    del TASKS[task_id]
    return "", 204

if __name__ == "__main__":
    app.run(host="127.0.0.1", port=5000, debug=True)
```

## 4) Запуск Flask

```bash
python -m venv .venv
# windows: .venv\Scripts\activate
source .venv/bin/activate

pip install -r requirements.txt
python app.py
```

---

# ✅ Full-stack (React + Flask) запуск вместе

1. Запусти Flask:

```bash
cd taskhub-flask
python app.py
```

2. В React включи API режим (файл `.env`):

```env
VITE_DATA_MODE=api
VITE_API_BASE=http://127.0.0.1:5000
```

3. Запусти React:

```bash
cd taskhub-react
npm run dev
```

---

# 🧪 Проверка endpoints (быстро)

## Создать задачу

```bash
curl -X POST http://127.0.0.1:5000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Buy milk"}'
```

## Получить список

```bash
curl http://127.0.0.1:5000/api/tasks
```

## Обновить статус

(подставь id)

```bash
curl -X PATCH http://127.0.0.1:5000/api/tasks/<id> \
  -H "Content-Type: application/json" \
  -d '{"status":"done"}'
```

## Удалить

```bash
curl -X DELETE http://127.0.0.1:5000/api/tasks/<id>
```

---

# ✅ Что именно “закрыто” этим решением (по требованиям)

* CRUD ✅
* status todo/doing/done ✅
* поиск + фильтр ✅
* валидация title ✅ (React + Flask)
* loading/empty/error ✅
* сортировка новые сверху ✅

---
