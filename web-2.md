# Урок 2. Main Layout: Header + Footer + Sidebar + Menu (HTML/CSS + Ant Design)

> Цель урока: собрать **общий каркас** веб‑приложения (main layout), чтобы дальше в него добавлять страницы.  
> Стек: **React + Ant Design (antd)**. Верстку и структуру делаем максимально “по‑взрослому”.

---

## 0) Что должно быть готово
- У вас уже создан проект React (например, Vite) и он запускается.
- Node.js установлен.
- В проекте есть Git‑репозиторий.

---

## 1) Установка Ant Design
В корне React‑проекта:
```bash
npm install antd
```

> Если у вас pnpm:
```bash
pnpm add antd
```

---

## 2) Структура папок (рекомендуемая)
Создайте:
```
src/
  app/
    App.tsx (или App.jsx)
  layouts/
    MainLayout.tsx
  pages/
    Dashboard.tsx
    Settings.tsx
  styles/
    layout.css
```

---

## 3) База: страницы (пока заглушки)

### 3.1 `src/pages/Dashboard.tsx`
```tsx
export default function Dashboard() {
  return <div>Dashboard page</div>
}
```

### 3.2 `src/pages/Settings.tsx`
```tsx
export default function Settings() {
  return <div>Settings page</div>
}
```

---

## 4) MainLayout на Ant Design

### 4.1 `src/layouts/MainLayout.tsx`
> Важно: используем готовый Layout от antd: `Layout`, `Sider`, `Header`, `Content`, `Footer`.

```tsx
import { Layout, Menu, Button } from "antd"
import type { MenuProps } from "antd"
import { useMemo, useState } from "react"

import Dashboard from "../pages/Dashboard"
import Settings from "../pages/Settings"
import "../styles/layout.css"

const { Header, Sider, Content, Footer } = Layout

type PageKey = "dashboard" | "settings"

export default function MainLayout() {
  const [collapsed, setCollapsed] = useState(false)
  const [page, setPage] = useState<PageKey>("dashboard")

  const items: MenuProps["items"] = useMemo(
    () => [
      { key: "dashboard", label: "Dashboard" },
      { key: "settings", label: "Settings" },
    ],
    []
  )

  const title = page === "dashboard" ? "Dashboard" : "Settings"
  const content = page === "dashboard" ? <Dashboard /> : <Settings />

  return (
    <Layout className="app-root">
      <Sider collapsible collapsed={collapsed} onCollapse={setCollapsed}>
        <div className="logo">My App</div>

        <Menu
          theme="dark"
          mode="inline"
          selectedKeys={[page]}
          items={items}
          onClick={(e) => setPage(e.key as PageKey)}
        />
      </Sider>

      <Layout>
        <Header className="app-header">
          <div className="header-left">{title}</div>
          <div className="header-right">
            <Button type="primary" onClick={() => alert("Logout (demo)")}>
              Logout
            </Button>
          </div>
        </Header>

        <Content className="app-content">
          <div className="content-card">{content}</div>
        </Content>

        <Footer className="app-footer">© {new Date().getFullYear()} My App</Footer>
      </Layout>
    </Layout>
  )
}
```

---

## 5) Подключаем MainLayout в App

### 5.1 `src/app/App.tsx` (или `src/App.jsx`)
```tsx
import MainLayout from "../layouts/MainLayout"

export default function App() {
  return <MainLayout />
}
```

> Если у вас Vite по умолчанию создал `src/App.jsx`, можно оставить там, просто импортируйте MainLayout.

---

## 6) CSS: аккуратная верстка

### 6.1 `src/styles/layout.css`
```css
/* Растягиваем на весь экран */
.app-root {
  min-height: 100vh;
}

/* Простая “заглушка‑логотип” */
.logo {
  height: 48px;
  margin: 16px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: 600;
  background: rgba(255, 255, 255, 0.15);
}

/* Header */
.app-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

/* Content */
.app-content {
  padding: 16px;
}

.content-card {
  background: white;
  border-radius: 12px;
  padding: 16px;
  min-height: 240px;
  max-width: 1100px;
  margin: 0 auto;
}

/* Footer */
.app-footer {
  text-align: center;
}
```

---

## 7) Практика на уроке (обязательные задания)

### Задание A (10–15 мин)
Сделайте в Sidebar 3 пункта меню:
- Dashboard
- Settings
- Profile (можно заглушку)

### Задание B (10–15 мин)
В Header добавьте:
- слева название текущей страницы (Dashboard/Settings/Profile)
- справа кнопку “Logout” (пока просто кнопка)

### Задание C (10–15 мин)
Стили:
- Сделайте, чтобы “content-card” был шириной максимум 1100px и по центру.
- Добавьте отступы в sidebar logo (уже есть) и сделайте его кликабельным (можно просто `onClick` с `alert`).

---

## 8) Типовые проблемы и решения

### Проблема 1: “Cannot find module 'antd'”
**Решение:**
```bash
npm install
npm install antd
```
И перезапустить dev‑сервер:
```bash
npm run dev
```

### Проблема 2: Меню не выделяет активный пункт
Проверьте:
- `selectedKeys={[page]}`
- `key` пунктов меню совпадает со значениями `page`

### Проблема 3: Стили “поехали”
Проверьте импорт:
- `import "../styles/layout.css"` внутри `MainLayout.tsx`

---

## 9) Чек‑лист сдачи
- ✅ Есть Header / Footer / Sidebar  
- ✅ Меню переключает контент  
- ✅ CSS аккуратный, контент в “карточке”  
- ✅ Коммит: `feat: add main layout with antd`
