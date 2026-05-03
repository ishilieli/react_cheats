# Шпаргалка по React (для тех, кто уже знает JS/TS)

Краткий справочник по основам клиентского React с примерами на TypeScript.
Подходит как стартовая шпаргалка для студентов, знакомых с JS, TS, HTML/CSS и ООП.

> Только классический клиентский React (без Server Components, без SSR-фреймворков).
> Все примеры — функциональные компоненты + хуки. Классовые компоненты не рассматриваем — они актуальны только для поддержки старого кода.

## Оглавление

1. [Что такое React](#1-что-такое-react)
2. [Старт проекта (Vite + TS)](#2-старт-проекта-vite--ts)
3. [JSX](#3-jsx)
4. [Компоненты](#4-компоненты)
5. [Пропсы (props)](#5-пропсы-props)
6. [Состояние: useState](#6-состояние-usestate)
7. [Обработка событий](#7-обработка-событий)
8. [Условный рендеринг](#8-условный-рендеринг)
9. [Списки и ключи](#9-списки-и-ключи)
10. [Формы (controlled inputs)](#10-формы-controlled-inputs)
11. [useEffect — сайд-эффекты](#11-useeffect--сайд-эффекты)
12. [useRef — ссылка на DOM и мутабельное значение](#12-useref--ссылка-на-dom-и-мутабельное-значение)
13. [useContext — общий контекст](#13-usecontext--общий-контекст)
14. [useReducer — сложное состояние](#14-usereducer--сложное-состояние)
15. [useMemo и useCallback — мемоизация](#15-usememo-и-usecallback--мемоизация)
16. [Кастомные хуки](#16-кастомные-хуки)
17. [Поднятие состояния (lifting state up)](#17-поднятие-состояния-lifting-state-up)
18. [Композиция через children](#18-композиция-через-children)
19. [Работа с API](#19-работа-с-api)
20. [Типичные ошибки новичков](#20-типичные-ошибки-новичков)

---

## 1. Что такое React

React — библиотека для построения UI из переиспользуемых компонентов.

Ключевые идеи:

- **Декларативность.** Описываете, *что* нужно отрисовать в текущем состоянии — React сам решает, *как* обновить DOM.
- **Компоненты.** UI собирается из независимых функций, возвращающих разметку.
- **Однонаправленный поток данных.** Данные текут сверху вниз через пропсы.
- **Виртуальный DOM + reconciliation.** React сравнивает новое дерево элементов со старым и точечно обновляет реальный DOM.

---

## 2. Старт проекта (Vite + TS)

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
npm run dev
```

Минимальная точка входа:

```tsx
// src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```tsx
// src/App.tsx
export default function App() {
  return <h1>Привет, React!</h1>;
}
```

`StrictMode` в dev-режиме намеренно вызывает компоненты и эффекты дважды — это помогает поймать побочные эффекты в местах, где их быть не должно. На продакшен-сборку не влияет.

---

## 3. JSX

JSX — синтаксический сахар над `React.createElement`. Выглядит как HTML, но это всё ещё JavaScript.

```tsx
const name = 'Аня';
const element = <h1 className="title">Привет, {name}!</h1>;
```

Отличия от HTML:

| HTML       | JSX             |
|------------|-----------------|
| `class`    | `className`     |
| `for`      | `htmlFor`       |
| `tabindex` | `tabIndex`      |
| `onclick`  | `onClick`       |
| `style="..."` (строка) | `style={{ color: 'red' }}` (объект) |

Правила:

- Возвращать можно только **один корневой элемент**. Если нужно несколько — используйте фрагмент `<>...</>`.
- Все теги обязаны быть закрыты: `<br />`, `<img />`.
- Внутри `{}` — любое JS-выражение (но не оператор: `if`, `for` нельзя).
- Комментарии внутри JSX: `{/* комментарий */}`.

```tsx
function Card() {
  const isAdmin = true;
  return (
    <>
      <h2>Заголовок</h2>
      <p style={{ color: isAdmin ? 'green' : 'gray' }}>
        Роль: {isAdmin ? 'админ' : 'гость'}
      </p>
    </>
  );
}
```

---

## 4. Компоненты

Компонент — функция, возвращающая JSX. Имя обязательно с **большой буквы** (с маленькой React сочтёт это HTML-тегом).

```tsx
function Greeting() {
  return <p>Привет!</p>;
}

// Использование:
<Greeting />
```

Компоненты можно вкладывать друг в друга:

```tsx
function Header() {
  return <h1>Мой сайт</h1>;
}

function Page() {
  return (
    <div>
      <Header />
      <main>Контент</main>
    </div>
  );
}
```

---

## 5. Пропсы (props)

Пропсы — входные параметры компонента. Передаются как атрибуты в JSX, читаются как поля одного объекта-аргумента.

```tsx
type ButtonProps = {
  label: string;
  disabled?: boolean;          // необязательный
  onClick: () => void;
};

function Button({ label, disabled = false, onClick }: ButtonProps) {
  return (
    <button disabled={disabled} onClick={onClick}>
      {label}
    </button>
  );
}

// Использование
<Button label="Сохранить" onClick={() => console.log('save')} />
```

Важные правила:

- **Пропсы иммутабельны.** Внутри компонента их менять нельзя.
- Поток данных однонаправленный: родитель → ребёнок. Чтобы ребёнок «сообщил» родителю об изменении — родитель передаёт callback-функцию через пропс.
- Пропс `children` — это всё, что вложено между открывающим и закрывающим тегом компонента.

```tsx
type CardProps = { children: React.ReactNode };

function Card({ children }: CardProps) {
  return <div className="card">{children}</div>;
}

<Card>
  <h2>Заголовок</h2>
  <p>Текст</p>
</Card>
```

---

## 6. Состояние: useState

`useState` хранит данные между ре-рендерами компонента и инициирует ре-рендер при изменении.

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Нажато: {count}
    </button>
  );
}
```

Что важно понимать:

- `setCount` — **асинхронный** в смысле «не меняет переменную сразу». В пределах текущего рендера `count` остаётся прежним.
- Если новое значение зависит от предыдущего — используйте функциональную форму:

  ```tsx
  setCount(prev => prev + 1);
  ```

- React сравнивает значения по ссылке (`Object.is`). Если передать **тот же** объект/массив — ре-рендера не будет. Поэтому состояние нужно обновлять **иммутабельно**: создавать новый объект/массив.

  ```tsx
  // ❌ так — мутация, React не заметит изменения
  user.name = 'Аня';
  setUser(user);

  // ✅ так — новый объект
  setUser({ ...user, name: 'Аня' });

  // ✅ массивы
  setItems([...items, newItem]);                       // добавить
  setItems(items.filter(i => i.id !== id));            // удалить
  setItems(items.map(i => i.id === id ? { ...i, done: true } : i)); // изменить
  ```

- Несколько `setState` подряд React батчит в один ре-рендер.

Типизация:

```tsx
const [count, setCount] = useState<number>(0);          // явно
const [user, setUser] = useState<User | null>(null);    // объединение
```

---

## 7. Обработка событий

События в JSX называются в camelCase, в обработчик передаётся **синтетическое событие** React (обёртка над нативным).

```tsx
function Form() {
  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    console.log('submit');
  }

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    console.log(e.target.value);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleChange} />
      <button type="submit">Ок</button>
    </form>
  );
}
```

Частые типы событий:

| Событие             | Тип                                        |
|---------------------|--------------------------------------------|
| Клик                | `React.MouseEvent<HTMLButtonElement>`      |
| Изменение input     | `React.ChangeEvent<HTMLInputElement>`      |
| Submit формы        | `React.FormEvent<HTMLFormElement>`         |
| Нажатие клавиши     | `React.KeyboardEvent<HTMLInputElement>`    |
| Фокус               | `React.FocusEvent<HTMLInputElement>`       |

Передача аргументов в обработчик — через стрелочную функцию:

```tsx
<button onClick={() => deleteItem(item.id)}>Удалить</button>
```

Не вызывайте функцию сразу:

```tsx
// ❌ выполнится при рендере, а не при клике
<button onClick={deleteItem(item.id)} />

// ✅
<button onClick={() => deleteItem(item.id)} />
```

---

## 8. Условный рендеринг

```tsx
function Status({ isOnline }: { isOnline: boolean }) {
  // 1) Тернарный оператор
  return <span>{isOnline ? 'онлайн' : 'офлайн'}</span>;
}

function Notification({ count }: { count: number }) {
  // 2) Логическое И — рендерит только если условие истинно
  return <div>{count > 0 && <span>Уведомлений: {count}</span>}</div>;
}

function Page({ user }: { user: User | null }) {
  // 3) Ранний возврат — удобно, когда ветка большая
  if (!user) return <p>Войдите</p>;
  return <Dashboard user={user} />;
}
```

⚠️ **Ловушка с `&&`:** если левая часть — число `0`, оно отрендерится в DOM как `0`.

```tsx
// ❌ при count === 0 на экране появится "0"
{count && <Badge count={count} />}

// ✅ явно приводите к булеву
{count > 0 && <Badge count={count} />}
```

---

## 9. Списки и ключи

Массивы JSX-элементов рендерятся как есть.

```tsx
type Todo = { id: number; text: string };

function TodoList({ todos }: { todos: Todo[] }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

Правила про `key`:

- `key` нужен, чтобы React сопоставлял элементы между рендерами и не пере-создавал DOM зря.
- `key` должен быть **уникальным среди соседей** и **стабильным** (не менялся для одного и того же логического элемента).
- ❌ Не используйте `index` массива как key, если список можно сортировать, фильтровать или вставлять элементы в середину — это приведёт к багам с состоянием инпутов и анимациями.
- ✅ Используйте `id` из данных. Если `id` нет — генерируйте при создании элемента (например, `crypto.randomUUID()`), а не на каждом рендере.

---

## 10. Формы (controlled inputs)

В **управляемом** инпуте значение хранится в state, а `value` и `onChange` синхронизируют его с DOM.

```tsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    console.log({ email, password });
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
      />
      <input
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
      />
      <button type="submit">Войти</button>
    </form>
  );
}
```

Если полей много, удобнее хранить их в одном объекте:

```tsx
const [form, setForm] = useState({ email: '', password: '' });

function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
  const { name, value } = e.target;
  setForm(prev => ({ ...prev, [name]: value }));
}

<input name="email" value={form.email} onChange={handleChange} />
<input name="password" value={form.password} onChange={handleChange} />
```

Чекбоксы — через `checked`, не `value`:

```tsx
<input
  type="checkbox"
  checked={agreed}
  onChange={e => setAgreed(e.target.checked)}
/>
```

---

## 11. useEffect — сайд-эффекты

`useEffect` запускает код **после** того, как React отрисовал компонент. Нужен для синхронизации с внешним миром: подписки, таймеры, запросы, прямые манипуляции с DOM.

```tsx
import { useEffect, useState } from 'react';

function Clock() {
  const [now, setNow] = useState(new Date());

  useEffect(() => {
    const id = setInterval(() => setNow(new Date()), 1000);
    return () => clearInterval(id); // cleanup
  }, []);

  return <p>{now.toLocaleTimeString()}</p>;
}
```

Массив зависимостей определяет, **когда** эффект перезапустится:

| Зависимости       | Когда срабатывает эффект               |
|-------------------|----------------------------------------|
| отсутствуют       | после **каждого** рендера              |
| `[]` (пустой)     | один раз после первого рендера         |
| `[a, b]`          | при изменении `a` или `b`              |

Функция, которую возвращает эффект, — это **cleanup**. Она вызывается:

- перед следующим запуском эффекта (когда зависимости изменились);
- при размонтировании компонента.

Без cleanup появляются утечки памяти: подписки висят, таймеры стреляют после удаления компонента и пытаются обновить уже отсутствующее состояние.

⚠️ **В StrictMode эффект в dev-режиме запускается дважды** (mount → unmount → mount). Это специально, чтобы вы заметили отсутствующий cleanup.

Когда useEffect **не** нужен:

- Преобразование данных для рендера — считайте прямо в теле компонента или через `useMemo`.
- Обработка пользовательских событий — кладите логику в обработчик клика, не в эффект.

```tsx
// ❌ лишний эффект и лишний рендер
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);

// ✅ просто посчитайте
const fullName = `${firstName} ${lastName}`;
```

---

## 12. useRef — ссылка на DOM и мутабельное значение

`useRef` возвращает объект `{ current: ... }`, который **сохраняется между рендерами** и **не вызывает ре-рендер при изменении**.

Два сценария:

**1. Ссылка на DOM-элемент:**

```tsx
import { useEffect, useRef } from 'react';

function AutoFocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} />;
}
```

**2. Хранение мутабельных значений между рендерами** (например, id таймера):

```tsx
function Timer() {
  const intervalId = useRef<number | null>(null);

  function start() {
    intervalId.current = window.setInterval(() => console.log('tick'), 1000);
  }

  function stop() {
    if (intervalId.current !== null) clearInterval(intervalId.current);
  }

  return (
    <>
      <button onClick={start}>Старт</button>
      <button onClick={stop}>Стоп</button>
    </>
  );
}
```

Чем `useRef` отличается от `useState`:

|                          | useState        | useRef           |
|--------------------------|-----------------|------------------|
| Изменение → ре-рендер    | да              | нет              |
| Подходит для UI-данных   | да              | нет              |
| Подходит для DOM-нод     | нет             | да               |
| Подходит для таймеров/id | нет (избыточно) | да               |

---

## 13. useContext — общий контекст

Контекст — способ передать данные «сквозь» дерево компонентов без проброса через каждый промежуточный пропс.

```tsx
import { createContext, useContext, useState } from 'react';

type Theme = 'light' | 'dark';
const ThemeContext = createContext<Theme>('light');

function App() {
  const [theme, setTheme] = useState<Theme>('light');
  return (
    <ThemeContext.Provider value={theme}>
      <Toolbar />
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Сменить тему
      </button>
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  return <ThemedButton />;
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={`btn-${theme}`}>Кнопка</button>;
}
```

Когда уместен:

- Тема, локаль, текущий пользователь, язык — то, что нужно «везде».
- Не используйте контекст для часто меняющихся данных всему приложению — каждый компонент-потребитель будет ре-рендериться при каждом изменении.

---

## 14. useReducer — сложное состояние

`useReducer` — альтернатива `useState`, когда:

- состояние — это объект с несколькими связанными полями;
- следующее состояние зависит от типа действия;
- логика обновления повторяется в нескольких местах.

```tsx
import { useReducer } from 'react';

type State = { count: number };
type Action = { type: 'inc' } | { type: 'dec' } | { type: 'set'; value: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'inc': return { count: state.count + 1 };
    case 'dec': return { count: state.count - 1 };
    case 'set': return { count: action.value };
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: 'inc' })}>+</button>
      <button onClick={() => dispatch({ type: 'dec' })}>−</button>
      <button onClick={() => dispatch({ type: 'set', value: 0 })}>сброс</button>
    </>
  );
}
```

Reducer **должен быть чистой функцией**: только считает новое состояние из старого и action, никаких запросов, мутаций, `console.log`'ов с побочными эффектами.

---

## 15. useMemo и useCallback — мемоизация

Оба хука кэшируют результат между рендерами, пока зависимости не изменились. Применяются для **оптимизации**, а не для корректности.

`useMemo` — кэширует **значение**:

```tsx
const sorted = useMemo(
  () => bigList.slice().sort((a, b) => a.value - b.value),
  [bigList]
);
```

`useCallback` — кэширует **функцию** (тождественен `useMemo(() => fn, deps)`):

```tsx
const handleClick = useCallback(
  (id: number) => deleteItem(id),
  [deleteItem]
);
```

Зачем стабильная ссылка:

- Если функция передаётся в `React.memo`-обёрнутый дочерний компонент — без `useCallback` он будет ре-рендериться каждый раз, потому что пропс — «новая» функция.
- Если функция/значение в зависимостях другого хука (`useEffect`, `useMemo`) — без мемоизации хук будет срабатывать каждый рендер.

⚠️ **Не оборачивайте всё подряд.** Мемоизация сама по себе не бесплатная: лишний код, проверки зависимостей, удержание ссылок. Применяйте, когда есть измеримая проблема: тяжёлое вычисление или дочерний компонент, который реально страдает от ре-рендеров.

---

## 16. Кастомные хуки

Кастомный хук — обычная функция, имя которой начинается с `use` и которая использует другие хуки. Это способ вынести и переиспользовать логику между компонентами.

```tsx
// useLocalStorage.ts
import { useEffect, useState } from 'react';

export function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    const raw = localStorage.getItem(key);
    return raw ? (JSON.parse(raw) as T) : initial;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}
```

Использование:

```tsx
function Settings() {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Тема: {theme}
    </button>
  );
}
```

Правила хуков (включая кастомных):

- Вызывать **только на верхнем уровне** функции компонента или другого хука. Никаких `if`, `for`, callback'ов.
- Вызывать **только из React-функций**: компонентов или других хуков.
- Имя обязательно начинается с `use` — по нему линтер понимает, что это хук, и проверяет правила.

Зачем это надо: React определяет, какому состоянию принадлежит вызов, по **порядку** вызовов. Если порядок поменяется между рендерами — состояние смешается.

---

## 17. Поднятие состояния (lifting state up)

Если двум соседним компонентам нужно одно и то же состояние — переносите его в их **общего родителя** и пробрасывайте вниз через пропсы.

```tsx
function App() {
  const [query, setQuery] = useState('');
  return (
    <>
      <SearchInput value={query} onChange={setQuery} />
      <ResultsList query={query} />
    </>
  );
}

function SearchInput({ value, onChange }: {
  value: string;
  onChange: (v: string) => void;
}) {
  return <input value={value} onChange={e => onChange(e.target.value)} />;
}

function ResultsList({ query }: { query: string }) {
  return <p>Ищем: {query}</p>;
}
```

Правило: храните состояние **на минимально возможном уровне**, общем для всех, кому оно нужно. Не выше, не ниже.

---

## 18. Композиция через children

Вместо того чтобы зашивать жёсткую структуру внутрь компонента, передавайте контент через `children` — это даёт гибкость без наследования.

```tsx
type ModalProps = {
  open: boolean;
  onClose: () => void;
  children: React.ReactNode;
};

function Modal({ open, onClose, children }: ModalProps) {
  if (!open) return null;
  return (
    <div className="overlay" onClick={onClose}>
      <div className="modal" onClick={e => e.stopPropagation()}>
        {children}
      </div>
    </div>
  );
}

// Использование — внутрь модалки можно положить что угодно
<Modal open={isOpen} onClose={() => setIsOpen(false)}>
  <h2>Подтвердите удаление</h2>
  <button onClick={handleDelete}>Удалить</button>
</Modal>
```

Можно принимать несколько «слотов» через именованные пропсы:

```tsx
type LayoutProps = {
  header: React.ReactNode;
  sidebar: React.ReactNode;
  children: React.ReactNode;
};

function Layout({ header, sidebar, children }: LayoutProps) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
    </div>
  );
}
```

В React предпочитают **композицию вместо наследования**: компоненты не наследуют друг друга, а собираются как Lego.

---

## 19. Работа с API

Базовый шаблон с `fetch`, `useEffect` и состояниями загрузки/ошибки.

```tsx
import { useEffect, useState } from 'react';

type User = { id: number; name: string };

function UsersList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      try {
        setLoading(true);
        setError(null);
        const res = await fetch('/api/users', { signal: controller.signal });
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const data: User[] = await res.json();
        setUsers(data);
      } catch (e) {
        if ((e as Error).name === 'AbortError') return;
        setError((e as Error).message);
      } finally {
        setLoading(false);
      }
    }

    load();
    return () => controller.abort();
  }, []);

  if (loading) return <p>Загрузка…</p>;
  if (error) return <p>Ошибка: {error}</p>;
  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name}</li>)}
    </ul>
  );
}
```

Что здесь важно:

- **`AbortController`** в cleanup отменяет запрос, если компонент размонтировали или зависимости изменились — иначе `setUsers` сработает уже после анмаунта.
- Три состояния: `loading`, `error`, `data`. Не забывайте про каждое.
- `fetch` **не бросает исключение** на HTTP-ошибках (404, 500). Проверяйте `res.ok` вручную.

В реальных проектах вместо ручного `useEffect` обычно берут готовое решение: **TanStack Query** (`@tanstack/react-query`), **SWR**, **RTK Query**. Они дают кэш, дедупликацию запросов, рефетч при фокусе окна, состояния — почти бесплатно.

---

## 20. Типичные ошибки новичков

**Мутация состояния напрямую**

```tsx
// ❌
items.push(newItem);
setItems(items);

// ✅
setItems([...items, newItem]);
```

**Использование индекса как `key` в изменяемом списке** — приводит к потере фокуса в инпутах и неправильному поведению при сортировке/фильтрации.

**Бесконечный цикл из-за объекта в зависимостях `useEffect`**

```tsx
// ❌ объект создаётся каждый рендер → эффект срабатывает бесконечно
useEffect(() => { /* ... */ }, [{ id: 1 }]);

// ✅ зависимости — примитивы или стабильные ссылки
useEffect(() => { /* ... */ }, [id]);
```

**Чтение `state` сразу после `setState`**

```tsx
// ❌ count здесь ещё старое
setCount(count + 1);
console.log(count);

// ✅ если нужно отреагировать на новое значение — useEffect или функциональная форма
setCount(prev => {
  const next = prev + 1;
  console.log(next);
  return next;
});
```

**Условный вызов хука**

```tsx
// ❌ нарушает правило хуков
if (props.user) {
  const [name, setName] = useState('');
}

// ✅ хуки всегда на верхнем уровне, условие — внутри
const [name, setName] = useState('');
if (!props.user) return null;
```

**`onClick={fn(arg)}` вместо `onClick={() => fn(arg)}`** — в первом варианте функция вызовется на рендере, а не на клике.

**`async` прямо на `useEffect`**

```tsx
// ❌ useEffect должен возвращать функцию-cleanup или undefined, не Promise
useEffect(async () => { /* ... */ }, []);

// ✅ async-функция внутри
useEffect(() => {
  async function load() { /* ... */ }
  load();
}, []);
```

**Забытый cleanup в подписках/таймерах** — компонент размонтировался, а интервал продолжает дёргать `setState` → ворнинги и утечки.

**Слишком много `useState`** — если 5 значений всегда меняются вместе, объедините их в объект или возьмите `useReducer`.

---

## Что почитать дальше

- Официальная документация: [react.dev](https://react.dev) (есть русская версия — [ru.react.dev](https://ru.react.dev))
- Вопросы для самопроверки: попробуйте без подсказок объяснить — что делает `useEffect`, чем `useRef` отличается от `useState`, зачем нужен `key`, почему пропсы иммутабельны.
- Следующие темы за рамками шпаргалки: `React.memo`, `forwardRef`, порталы, error boundaries, роутинг (`react-router`), управление серверным состоянием (TanStack Query), формы (`react-hook-form`), стейт-менеджеры (Zustand, Redux Toolkit).

