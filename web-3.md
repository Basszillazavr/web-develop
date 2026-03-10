# Урок 3. Routing и страницы приложения (React + Ant Design)

## Цель урока

На прошлом занятии мы сделали **Main Layout** приложения:

- Header
- Sidebar
- Footer
- Menu

Теперь приложение должно **открывать разные страницы**.

Сегодня мы добавим **маршрутизацию (routing)** и создадим несколько страниц.

---

# Что такое Routing

Routing — это механизм, который показывает разные страницы в зависимости от URL.

Пример:

```

/ - главная
/students - студенты
/courses - курсы
/contacts - контакты

```

В React это работает **без перезагрузки страницы**.

Для этого используется библиотека:

```

react-router-dom

```

---

# Установка библиотеки

Установите пакет:

```

npm install react-router-dom

```

или

```

yarn add react-router-dom

```

---

# Структура проекта

Создайте структуру:

```

src
components
AppHeader.jsx
AppFooter.jsx
AppSidebar.jsx

layout
MainLayout.jsx

pages
HomePage.jsx
StudentsPage.jsx
CoursesPage.jsx
ContactsPage.jsx

App.jsx
main.jsx

````

---

# Создание страниц

## HomePage.jsx

```jsx
const HomePage = () => {
  return (
    <div>
      <h1>Главная страница</h1>
      <p>Добро пожаловать в систему.</p>
    </div>
  )
}

export default HomePage
````

---

## StudentsPage.jsx

```jsx
const StudentsPage = () => {
  return (
    <div>
      <h1>Студенты</h1>
      <p>Здесь будет список студентов.</p>
    </div>
  )
}

export default StudentsPage
```

---

## CoursesPage.jsx

```jsx
const CoursesPage = () => {
  return (
    <div>
      <h1>Курсы</h1>
      <p>Здесь будет список курсов.</p>
    </div>
  )
}

export default CoursesPage
```

---

## ContactsPage.jsx

```jsx
const ContactsPage = () => {
  return (
    <div>
      <h1>Контакты</h1>
      <p>Email: example@mail.com</p>
    </div>
  )
}

export default ContactsPage
```

---

# Подключение Router

## main.jsx

```jsx
import React from "react"
import ReactDOM from "react-dom/client"
import { BrowserRouter } from "react-router-dom"
import App from "./App"

ReactDOM.createRoot(document.getElementById("root")).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
)
```

---

# Настройка маршрутов

## App.jsx

```jsx
import { Routes, Route } from "react-router-dom"
import MainLayout from "./layout/MainLayout"

import HomePage from "./pages/HomePage"
import StudentsPage from "./pages/StudentsPage"
import CoursesPage from "./pages/CoursesPage"
import ContactsPage from "./pages/ContactsPage"

function App() {
  return (
    <MainLayout>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/students" element={<StudentsPage />} />
        <Route path="/courses" element={<CoursesPage />} />
        <Route path="/contacts" element={<ContactsPage />} />
      </Routes>
    </MainLayout>
  )
}

export default App
```

---

# Обновление Sidebar Menu

Меню должно открывать страницы.

## AppSidebar.jsx

```jsx
import { Layout, Menu } from "antd"
import { Link, useLocation } from "react-router-dom"

const { Sider } = Layout

const AppSidebar = () => {

  const location = useLocation()

  const items = [
    {
      key: "/",
      label: <Link to="/">Главная</Link>
    },
    {
      key: "/students",
      label: <Link to="/students">Студенты</Link>
    },
    {
      key: "/courses",
      label: <Link to="/courses">Курсы</Link>
    },
    {
      key: "/contacts",
      label: <Link to="/contacts">Контакты</Link>
    }
  ]

  return (
    <Sider width={220}>
      <Menu
        theme="dark"
        mode="inline"
        selectedKeys={[location.pathname]}
        items={items}
      />
    </Sider>
  )
}

export default AppSidebar
```

---

# Как это работает

Когда пользователь открывает:

```

/ -> HomePage
/students -> StudentsPage
/courses -> CoursesPage
/contacts -> ContactsPage

```

React Router показывает нужный компонент.

---

# Почему используем Link

Неправильно:

```html
<a href="/students">Студенты</a>
```

Это вызывает **перезагрузку страницы**.

Правильно:

```jsx
<Link to="/students">Студенты</Link>
```

React меняет страницу **без перезагрузки**.

---

# Частые ошибки

### 1. Забыли BrowserRouter

Routing работать не будет.

---

### 2. Используют `<a>` вместо `Link`

Это ломает SPA.

---

### 3. key меню не совпадает с URL

```
key="/students"
```

должен совпадать с путем страницы.

---

# Практическое задание

Создайте страницы:

* Главная
* Преподаватели
* Группы
* Расписание

Требования:

1. Подключить `react-router-dom`
2. Сделать переход через Sidebar Menu
3. Подсветить активный пункт меню
4. На каждой странице добавить заголовок

---

# Дополнительное задание

Сделать страницу **404**.

```
pages/NotFoundPage.jsx
```

```jsx
const NotFoundPage = () => {
  return <h1>404 Страница не найдена</h1>
}

export default NotFoundPage
```

Подключить:

```jsx
<Route path="*" element={<NotFoundPage />} />
```

---

# Итог урока

Сегодня мы:

* разобрали routing
* установили react-router-dom
* создали страницы
* подключили маршруты
* связали Sidebar Menu с переходами

Теперь наше приложение стало **многостраничным SPA**.

```
