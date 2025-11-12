<h1> 202130413 신민수
<h2>2025년 11월 12일 (11주차)</h2>  
<h2>수업내용: Next.js 스트리밍 (Streaming) — loading.tsx / Suspense 활용</h2>

---

<h2>스트리밍이란? (Streaming)</h2>

> ⚠️ **경고 (Warning)**  
> 아래 내용은 애플리케이션에서 `cacheComponents config` 옵션이 활성화되어 있다고 **가정**합니다.  
> 이 기능은 **Next.js 15 Canary** 버전에서 도입되었습니다.  
>
> - `latest` : 현재 가장 안정적인 최신 버전  
> - `canary` : 안정화 직전의 최신 개발 버전  

---

### ✅ 서버 컴포넌트에서 async/await 사용 시

- 서버 컴포넌트에서 `async/await`를 사용하면 **Next.js는 동적 렌더링(Dynamic Rendering)** 을 수행합니다.  
- 모든 요청 시 서버가 데이터를 가져오며, 느린 요청일 경우 전체 렌더링이 지연됩니다.  
- 이를 개선하기 위해 HTML을 작은 블록 단위로 분할하고, **점진적으로 스트리밍(Streaming)** 하여 전송합니다.  

---

<h2>3-1. loading.tsx를 사용하는 방법</h2>

> Next.js에서 스트리밍을 구현하는 방법은 두 가지가 있습니다.  
> 1️⃣ `loading.tsx` 파일로 페이지 감싸기  
> 2️⃣ `<Suspense>`로 특정 컴포넌트 감싸기  

---

<h2> 2025년 11월 05일 11주차 </h2>
<h2> 수업내용: Next.js 데이터 가져오기 (Fetching Data) — 서버 & 클라이언트 컴포넌트 </h2>

---

## 🧩 1. Fetching Data (데이터 가져오기)

Next.js에서 데이터를 가져오는 방법은  
**서버 컴포넌트(Server Component)** 와 **클라이언트 컴포넌트(Client Component)** 로 나뉩니다.  
각 방식은 동작 구조와 최적화 포인트가 다르므로 목적에 따라 선택해야 합니다.

---

## 🧠 1-1. 서버 컴포넌트 (Server Component)

### ✅ 데이터 가져오는 두 가지 방법
1. **fetch API 사용**  
2. **ORM 또는 데이터베이스 직접 접근**

---

### 📘 ① fetch API 사용

서버 컴포넌트에서 데이터를 가져오려면 `fetch()` 함수를 비동기식으로 호출하고,  
호출 결과를 기다린 뒤 렌더링합니다.

```tsx
// app/blog/page.tsx
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

🧩 **설명**
- `await fetch()` 호출을 통해 서버에서 데이터를 불러오며,  
  렌더링 전에 데이터를 모두 확보한 후 HTML을 생성합니다.  
- 즉, **SSR(Server-Side Rendering)** 방식으로 초기 페이지 로드 시 완성된 HTML을 전달합니다.

---

## 💡 알아두면 좋은 정보

- `fetch()`의 응답은 **기본적으로 캐싱되지 않습니다.**  
- 그러나 **Next.js는 라우팅된 페이지를 미리 렌더링(Pre-rendering)** 하여,  
  성능 향상을 위해 **출력된 결과를 자동으로 캐싱**합니다.
- 만약 **동적 렌더링**을 사용하려면 아래처럼 명시합니다:
  ```tsx
  fetch(url, { cache: 'no-store' })
  ```
- 개발 중에는 디버깅을 위해 `fetch` 요청을 기록할 수 있습니다.  
  관련 문서: **fetch API 참고 / logging API 참고**

---

### ⚙️ 추가 설명
Next.js **15.1 이후 버전**부터는  
서버 컴포넌트 내부에서 `await` 없이도 `fetch()` 사용이 가능해졌습니다.  
즉, 자동으로 **비동기 처리와 스트리밍 렌더링이 통합**되었습니다.

---

## 🧩 1-2. 클라이언트 컴포넌트 (Client Component)

### ✅ 데이터 fetch 방법
1. **React의 use Hook (예: useEffect, useState)**  
2. **SWR / React Query** 같은 클라이언트 통신 라이브러리 사용

---

### 📘 use Hook을 사용한 스트리밍 데이터

React의 `use` Hook을 이용하면  
서버에서 데이터를 가져온 후 **클라이언트에서 스트리밍 형태로 표시**할 수 있습니다.

서버 컴포넌트는 async/await를 사용할 수 있지만,  
클라이언트 컴포넌트는 렌더링 도중 `async`가 허용되지 않습니다.  
그래서 서버에서 미리 `fetch()` 결과를 **Promise 형태로 전달**하고,  
클라이언트에서는 `use(promise)`를 통해 데이터를 처리합니다.

---

## ⚙️ React의 use Hook을 사용한 실습

React의 `use` Hook을 사용하여 서버에서 클라이언트로 데이터를 스트리밍하는 예제입니다.

```tsx
// app/blog/page.tsx
import Posts from '@/ui/posts'
import { Suspense } from 'react'

export default function Page() {
  // Don't await the data fetching function.
  const posts = fetch('https://jsonplaceholder.typicode.com/posts')
    .then((res) => res.json())

  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Posts posts={posts} />
    </Suspense>
  )
}
```

📍 **핵심 요약**
- `await` 없이 `fetch()`를 호출해 **Promise 상태**를 유지합니다.  
- React는 이 Promise를 감지하고, 데이터가 도착할 때까지 자동으로 **Suspense fallback**을 표시합니다.  
- 데이터가 완성되면 `Posts` 컴포넌트로 전달되어 화면에 렌더링됩니다.  

---

## 🧾 요약

| 구분 | 설명 |
|------|------|
| Server Component | 비동기 fetch() 사용 가능, SSR로 초기 렌더링 최적화 |
| Client Component | Hook(`useEffect`, `use`) 또는 SWR 등 라이브러리 사용 |
| 캐싱 | Next.js는 기본적으로 fetch 결과를 캐시함 (성능 향상) |
| 동적 렌더링 | `{ cache: 'no-store' }` 옵션 사용 |
| Next.js 15.1+ | await 없이 fetch 가능 (자동 비동기 처리) |

---

> **정리 한 줄 요약**  
> 서버 컴포넌트는 SSR을 통해 **초기 렌더링 최적화**,  
> 클라이언트 컴포넌트는 **스트리밍 기반의 인터랙션 데이터 처리**에 적합합니다.


---

## 🔍 Fetch의 동작 원리 이해하기

`fetch()`는 JavaScript의 내장 API로, 네트워크 요청을 보내고 응답을 받는 데 사용됩니다.  
하지만 Next.js 환경에서는 **서버와 클라이언트 모두에서 fetch가 동작**할 수 있기 때문에  
Promise의 처리 방식과 에러 핸들링 구조를 정확히 이해하는 것이 중요합니다.

---

### ⚙️ 기본 동작 방식

- `fetch()`는 **Promise 객체**를 반환합니다.  
- 하지만 **HTTP 상태 코드가 4xx / 5xx**여도 `reject`되지 않습니다.  
- 네트워크 연결 자체가 실패하지 않는 한, **항상 resolve 상태로 응답**을 반환합니다.

```tsx
function getPosts() {
  return fetch('https://jsonplaceholder.typicode.com/posts')
    .then((res) => res.json())
}
```

🧩 **즉**,  
HTTP 오류(예: 404, 500)는 네트워크 에러가 아니므로  
`try-catch` 또는 `res.ok`를 통해 직접 검사해야 합니다.

---

### 🧠 예외 처리 방식 예시

```tsx
async function getPostsSafe() {
  const res = await fetch('https://jsonplaceholder.typicode.com/posts')

  if (!res.ok) {
    throw new Error(`HTTP 오류 발생: ${res.status}`)
  }

  return res.json()
}
```

💡 **요약**
- `fetch()`는 네트워크 단절이 아니면 오류로 간주하지 않는다.  
- 따라서 **응답 코드 검사**가 필수적이다.

---

## 🧩 Promise의 이해

`fetch()`의 동작을 완전히 이해하려면 Promise의 구조를 알아야 합니다.  
Promise는 **비동기 작업의 성공(resolve)** 또는 **실패(reject)** 를 나타내는 객체입니다.

---

### 📘 Promise 기본 구조

```tsx
const promise = new Promise((resolve, reject) => {
  // 비동기 작업 수행
  const success = true

  if (success) {
    resolve('성공 결과')
  } else {
    reject('에러 메시지')
  }
})
```

### 🔹 Promise의 2가지 상태
| 상태 | 설명 |
|------|------|
| `resolve` | 작업이 성공했을 때 실행되는 콜백 |
| `reject` | 작업이 실패했을 때 실행되는 콜백 |

---

### 📘 Promise 결과 사용하기

```tsx
promise
  .then((result) => {
    console.log('성공:', result)
  })
  .catch((error) => {
    console.error('실패:', error)
  })
```

🧩 **설명**
- `.then()` → resolve가 호출되면 실행  
- `.catch()` → reject가 호출되면 실행  

---

## 🧠 실전에서의 Fetch + Promise 처리 흐름

```tsx
async function loadData() {
  try {
    const res = await fetch('https://api.vercel.app/posts')

    if (!res.ok) throw new Error('데이터를 불러올 수 없습니다.')

    const data = await res.json()
    console.log('불러온 데이터:', data)
  } catch (error) {
    console.error('에러 발생:', error)
  }
}
```

📍 **핵심 요약**
- `fetch()` → 항상 Promise 반환  
- `.ok` 검사로 HTTP 상태 확인  
- `try/catch`로 네트워크 예외 처리  
- Promise는 **비동기 흐름을 제어하는 핵심 도구**

---

<h3>🧾 전체 정리</h3>

| 구분 | 설명 |
|------|------|
| fetch 기본 동작 | 항상 Promise resolve (네트워크 에러 제외) |
| 에러 처리 | `res.ok` 또는 `try/catch`로 직접 검사 |
| Promise resolve/reject | 성공/실패 상태를 나타냄 |
| then / catch | 각각 성공/실패 시 동작 지정 |
| Next.js에서의 의미 | 서버/클라이언트 모두 fetch 가능하나, 데이터 검증이 중요 |

---

> **정리 한 줄 요약**  
> Next.js에서의 `fetch()`는 Promise 기반의 비동기 동작으로,  
> **성공/실패 흐름을 명확히 제어해야 안정적인 데이터 로딩이 가능**합니다.
  
<h2> 2025년 10월 29일 10주차 </h2>
<h2> 수업내용: Next.js `"use client"` 완벽 정리 및 Server-Client 경계 이해 </h2>

---

## 🧭 개요

Next.js의 **Server Components / Client Components** 개념에서  
`"use client"` 지시문은 **클라이언트 전용 컴포넌트임을 명시**하는 역할을 합니다.  

이 문서는 `"use client"`의 **사용 예시**, **데이터 전달 방식**, **성능 최적화 전략**,  
그리고 **Server ↔ Client 컴포넌트 통신 구조**를 단계별로 정리합니다.

---

## 🧩 Client Component 생성

Client Component를 만들려면 파일 맨 위에 `"use client"`를 추가해야 합니다.

```tsx
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>{count} likes</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  )
}
```

📌 **핵심 요약**
- `"use client"`는 **React Hook (`useState`, `useEffect`, 등)** 을 사용하는 컴포넌트에서 필수입니다.  
- Server Component에서는 Hook을 사용할 수 없기 때문입니다.  
- 해당 지시문이 선언된 파일은 번들 시 클라이언트 사이드로 분리되어 전송됩니다.

---

## ⚡ JS 번들 크기 줄이기

> `"use client"`는 **필요한 최소한의 영역에만 적용**하는 것이 중요합니다.  
> 상위 레이아웃 전체에 `"use client"`를 선언하면 불필요한 자바스크립트가 클라이언트로 전송되어  
> **페이지 로딩 속도와 성능이 저하**됩니다.

### ✅ 올바른 예시

```tsx
// app/layout.tsx
import Search from './search'
import Logo from './logo'

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Logo />     {/* 서버에서 렌더링 가능 */}
        <Search />   {/* 클라이언트 전용 */}
      </nav>
      <main>{children}</main>
    </>
  )
}
```

```tsx
// app/ui/search.tsx
'use client'

export default function Search() {
  // ...
}
```

🧠 **설명**
- `Logo`는 정적 콘텐츠이므로 **Server Component**로 유지.  
- `Search`는 입력 이벤트와 상태 관리가 필요하므로 **Client Component**로 처리.  
- 이 구조는 **JS 번들 크기를 최소화**하고 **TTV (Time To View)** 를 단축시킵니다.

---

## 🔗 서버 → 클라이언트 데이터 전달

Server Component에서 데이터를 가져오고,  
Client Component로 **props를 통해 전달**할 수 있습니다.

```tsx
// app/[id]/page.tsx
import LikeButton from '@/app/ui/like-button'
import { getPost } from '@/lib/data'

export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  const post = await getPost(id)

  return <LikeButton likes={post.likes} />
}
```

```tsx
// app/ui/like-button.tsx
'use client'

export default function LikeButton({ likes }: { likes: number }) {
  // ...
}
```

⚠️ **주의사항**
- Client Component로 전달되는 `props`는 반드시 **직렬화 가능 (Serializable)** 해야 합니다.  
- 즉, 함수, 클래스 인스턴스, Symbol 등은 props로 전달 불가합니다.  
- 단순 JSON 형태(`number`, `string`, `object`, `array`)만 가능.

---

## 🧱 Server ↔ Client 컴포넌트 섞기 (Interleaving)

Next.js에서는 **Server Component를 Client Component의 children으로 넘기는 구조**가 가능합니다.

### 📘 예시

```tsx
// app/ui/modal.tsx
'use client'

export default function Modal({ children }: { children: React.ReactNode }) {
  return <div className="modal">{children}</div>
}
```

```tsx
// app/page.tsx
import Modal from './ui/modal'
import Cart from './ui/cart'

export default function Page() {
  return (
    <Modal>
      <Cart /> {/* 서버에서 렌더링된 컴포넌트 */}
    </Modal>
  )
}
```

🧩 **설명**
- `Modal`은 클라이언트에서 열고 닫는 토글 상태를 제어합니다.  
- `Cart`는 서버에서 데이터를 불러오며, `Modal` 내부에 주입(interleave)됩니다.  
- 즉, **서버 렌더링된 UI**를 **클라이언트 인터랙션을 가진 컴포넌트 안에 자연스럽게 포함**할 수 있습니다.  
- 이를 **인터리빙(Interleaving)** 이라 부릅니다.

---

## 🎛 React Context 사용하기

> ⚠️ **주의:**  
> React의 Context API는 **Server Component에서 직접 지원되지 않습니다.**  
> 따라서 반드시 **Client Provider**를 만들어 감싸야 합니다.

### ✅ 예시

```tsx
// app/theme-provider.tsx
'use client'

import { createContext } from 'react'

export const ThemeContext = createContext({})

export default function ThemeProvider({ children }: { children: React.ReactNode }) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
}
```

```tsx
// app/layout.tsx
import ThemeProvider from './theme-provider'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  )
}
```

💡 **팁**
- Provider는 **트리의 가능한 한 깊은 곳**에 배치해야 합니다.  
- Next.js는 위로 갈수록 정적 렌더링 최적화를 수행하므로  
  Context Provider를 상위에 둘수록 성능이 떨어질 수 있습니다.

---

## 🧩 써드파티(Third-party) 컴포넌트 사용

`useState`, `useEffect`와 같은 **클라이언트 전용 기능**을 사용하는  
서드파티 라이브러리는 반드시 **Client Component** 안에서만 사용할 수 있습니다.

### 📘 예시

```tsx
// app/gallery.tsx
'use client'

import { useState } from 'react'
import { Carousel } from 'acme-carousel'

export default function Gallery() {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>View pictures</button>
      {isOpen && <Carousel />}
    </div>
  )
}
```

🚫 **문제 상황**
- `Server Component` 내부에서 직접 `Carousel`을 호출하면 오류 발생.  

✅ **해결 방법**
- **Client Wrapper**를 만들어 해당 컴포넌트를 감싸서 사용.

```tsx
// app/carousel.tsx
'use client'

import { Carousel } from 'acme-carousel'
export default Carousel
```

이제 Server Component에서도 안전하게 사용 가능 👇

```tsx
// app/page.tsx
import Carousel from './carousel'

export default function Page() {
  return (
    <div>
      <p>View pictures</p>
      <Carousel />
    </div>
  )
}
```

---

## 🧠 라이브러리 제작자를 위한 조언

- 라이브러리에서 **클라이언트 전용 기능**(`useState`, `useEffect`)을 사용하는 엔트리 포인트에는  
  반드시 `"use client"`를 추가해야 합니다.
- 이렇게 하면 사용자가 별도의 Wrapper를 만들지 않고도 바로 import 가능.
- 단, 일부 번들러(`esbuild`, `rollup`)는 `"use client"` 지시문을 제거할 수 있으므로  
  **빌드 설정에서 보존되도록 명시**해야 합니다.

---

## 🧾 요약 정리

| 구분 | 설명 |
|------|------|
| `"use client"` | 해당 파일을 클라이언트 컴포넌트로 선언 |
| 데이터 전달 | Server → Client는 `props`를 통해 전달 (직렬화 필요) |
| 번들 최적화 | 불필요한 `"use client"` 최소화 |
| Context | Client Provider로 감싸서 사용 |
| 써드파티 | Client에서만 사용하거나 Wrapper로 감싸기 |
| 라이브러리 | `"use client"`를 엔트리 포인트에 명시해야 함 |

---

## ✨ 결론: `"use client"` 한 줄 요약

> `"use client"`는 **Next.js의 서버-클라이언트 경계를 명시하는 핵심 도구**입니다.  
> 정적인 부분은 **서버에서 렌더링**,  
> 상호작용이 필요한 부분은 **클라이언트에서 실행**함으로써  
> **성능, 보안, 유지보수성** 모두를 향상시킬 수 있습니다.


---

<h2> 2025년 10월 22일 9주차 </h2>
<h2> 수업내용: Next.js Server 및 Client Component 인터리빙 (Interleaving) </h2>

---

## 3-4. Server 및 Client Component 인터리빙 (Interleaving)

### 🧩 인터리빙(Interleaving)이란?
- 일반적으로 **여러 데이터 블록이나 비트를 섞어서 전송하거나 처리**하여,  
  오류 발생 시 영향을 최소화하는 기술을 의미한다.  
- 주로 데이터 통신에서 **버스트 오류(연속적인 오류)**를 줄이고  
  오류 정정 코드를 효과적으로 활용하기 위해 사용된다.

---

### 💻 프로그래밍에서의 의미
- **Server Component**와 **Client Component**가 **섞여(interleaved)**  
  동시에 작동하는 방식을 의미한다.
- 즉, **서버에서 렌더링된 UI와 클라이언트 상호작용 UI가 하나의 트리 안에서 공존**하는 구조다.

---

### ⚙️ 작동 방식 요약
1. **Server Component → Client Component로 props 전달 가능**
   - 서버에서 처리한 데이터를 클라이언트 컴포넌트로 넘길 수 있다.
2. **Client Component 내부에서 Server UI를 시각적으로 중첩 가능**
   - 예를 들어 서버에서 렌더링된 콘텐츠를 클라이언트 상호작용 영역에 삽입할 수 있다.
3. `<ClientComponent>`에 **slot(children)**을 두어  
   **Server Component의 결과물을 자식으로 전달**하는 방식이 일반적이다.

---

### 📘 예시 코드
```tsx
// app/ui/modal.tsx
'use client';

export default function Modal({ children }: { children: React.ReactNode }) {
  return <div>{children}</div>;
}
```

- `Modal`은 클라이언트 컴포넌트이지만,  
  내부의 `children`에는 서버에서 렌더링된 요소를 전달할 수 있다.
- 이런 식으로 **서버에서 생성된 콘텐츠와 클라이언트 인터랙션이 함께 작동**하도록 구성하는 것이  
  Next.js의 **인터리빙(Interleaving)** 개념이다.

---

## 🧩 3-4. Server 및 Client Component 인터리빙 (Interleaving)

### ✅ 개념 정리
- **인터리빙(Interleaving)**:  
  서버 컴포넌트와 클라이언트 컴포넌트가 **섞여서 작동하는 구조**.  
  데이터를 효율적으로 주고받고, 렌더링을 분리·조합할 수 있게 한다.
- 예시:  
  클라이언트의 `state`로 표시 여부를 제어하는 `<Modal>` 컴포넌트 안에,  
  서버에서 데이터를 가져오는 `<Cart>` 컴포넌트를 포함할 수 있다.

---

### ⚙️ 예시 코드
```tsx
// app/page.tsx
import Modal from "./ui/modal";
import Cart from "./ui/cart";

export default function Page() {
  return (
    <Modal>
      <Cart />
    </Modal>
  );
}
```

- `Page`는 **server component**,  
  `Modal`은 **client component**로 동작.  
- `<Cart>`는 서버에서 데이터를 미리 렌더링하여  
  `<Modal>`의 자식(children)으로 전달된다.  
- 즉, 서버에서 만들어진 HTML이 클라이언트 렌더링 트리에 **중첩(interleaved)** 되어 있는 구조이다.

---

### 🧠 작동 방식 요약
1. Next.js는 먼저 `ServerContent`를 서버에서 렌더링하여 HTML로 변환한다.  
2. 이 HTML을 `ClientLayout`의 `{children}`에 “끼워 넣는다.”  
3. 이후 클라이언트 측에서는 `ClientLayout`만 **Hydration (이벤트 연결)** 수행.  
4. 서버 데이터는 이미 포함되어 있고, 버튼 등 상호작용은 클라이언트 측에서 처리된다.

➡️ 즉, **서버와 클라이언트가 섞여(interleaved)** 작동하는 구조이다.

---

## 🧭 3-5. Context란 무엇인가?

### 🧩 기본 개념
- React 및 Next.js에서 **Context**는  
  **컴포넌트 간 데이터 공유를 위한 메커니즘**이다.
- 부모 컴포넌트가 자식에게 직접 `props`를 전달하지 않아도,  
  하위 트리 내에서 데이터를 **공통적으로 접근 및 사용**할 수 있도록 돕는다.

---

### 💡 동작 개요
- `React.createContext()`로 Context 객체를 생성한다.
- `MyContext.Provider`를 통해 하위 컴포넌트에 값을 전달한다.
- 하위 컴포넌트에서는 `useContext(MyContext)`를 사용하여 그 값을 받아온다.

---

### 📘 코드 예시
```tsx
// Context 생성
const MyContext = React.createContext();

function MyComponent() {
  const value = useContext(MyContext);
  return <div>{value}</div>;
}

function App() {
  return (
    <MyContext.Provider value="Hello from Context">
      <MyComponent />
    </MyContext.Provider>
  );
}
```

### 🔍 작동 원리 요약
1. `App`은 `MyContext.Provider`를 통해 데이터를 하위 트리로 전달.  
2. `MyComponent`는 `useContext(MyContext)`를 통해 해당 값("Hello from Context")을 읽는다.  
3. 따라서 **props 없이도 데이터 공유**가 가능하다.

---

### 📎 정리 포인트
| 구분 | 설명 |
|------|------|
| **Provider** | Context 값을 하위 컴포넌트에 전달 |
| **Consumer (useContext)** | Context 값을 읽고 사용 |
| **장점** | props drilling 없이 전역 상태 공유 |
| **활용 예시** | 로그인 사용자 정보, 테마 색상, 다국어 설정 등 |

> ✅ Context는 “React의 전역 데이터 전달 시스템”으로,  
> Next.js에서도 상태나 설정을 컴포넌트 트리 전반에 효율적으로 전달할 때 사용된다.

---

<h2> 2025년 10월 17일 8주차 </h2>
<h2> 수업내용: Next.js Server 및 Client Component 사용 시점 </h2>

---

## 1. server 및 client component를 언제 사용해야 하나?
- **Client 환경과 Server 환경은 서로 다른 기능**을 수행한다.
- `server`와 `client` 컴포넌트를 상황에 따라 적절히 사용하면,  
  각 환경에서 필요한 로직을 분리하고 효율적으로 실행할 수 있다.

---

### ✅ 다음과 같은 항목이 필요한 경우 → **Client Component**를 사용
- **state 및 event handler**
  - 예: `onClick`, `onChange` 등 사용자 이벤트 처리
- **Lifecycle logic**
  - 예: `useEffect` 등 컴포넌트 마운트/업데이트 시 동작하는 로직
- **브라우저 전용 API 사용**
  - 예: `localStorage`, `window`, `Navigator.geolocation`
- **사용자 정의 Hook**
  - 클라이언트 상태나 상호작용을 다루는 훅(Custom Hook)

---

### ✅ 다음과 같은 항목이 필요한 경우 → **Server Component**를 사용
- **서버 DB나 API에서 데이터를 가져오는 경우**
  - 서버 사이드에서 `fetch`, `query` 등을 통해 직접 데이터 처리
- **API key, 보안 데이터 보호**
  - 민감한 정보를 클라이언트로 노출하지 않기 위해 서버에서만 처리
- **전송되는 JavaScript 용량 감소**
  - 브라우저에서 실행되는 JS 양을 줄여 성능 향상
- **콘텐츠가 포함된 첫 번째 페인트(First Contentful Paint, FCP) 개선**
  - 초기 렌더링 속도를 높이기 위해, 콘텐츠를 **Server → Client**로 점진적 스트리밍

---

> 💡 **요약**  
> - `Client Component`: 상호작용 중심 (이벤트, 상태, 브라우저 API)  
> - `Server Component`: 데이터 중심 (DB, API, 보안, 초기 렌더링)

---

## 1. server 및 client component를 언제 사용해야 하나? (실습 예제)
- 예를 들어 `<Page>` 컴포넌트는 **게시물 데이터**를 서버에서 가져와서,  
  클라이언트 측 상호작용을 처리하는 `<LikeButton>`에 props로 전달하는 **Server Component**이다.
- 반면 `/ui/like-button`은 **Client Component**이므로 `use client`를 선언해야 한다.

```tsx
// app/[id]/page.tsx
import LikeButton from "@/app/ui/like-button";
import { getPost } from "@/lib/data";

export default async function Page({ params }: { params: { id: string } }) {
  const post = await getPost(params.id);

  return (
    <main>
      <h1>{post.title}</h1>
      <LikeButton likes={post.likes} />
    </main>
  );
}
```

```tsx
// app/ui/like-button.tsx
"use client";

import { useState } from "react";

export default function LikeButton({ likes }: { likes: number }) {
  const [count, setCount] = useState(likes);
  return (
    <button onClick={() => setCount(count + 1)}>
      ❤️ {count} likes
    </button>
  );
}
```

---

## 문서의 코드를 완성해 봅시다 — slug page
- 완성된 코드는 다음과 같다.  
- `getPost` 함수를 별도로 구현하지 않고, **slug에 포함된 id**를 직접 비교하여 데이터를 가져온다.

```tsx
// app/[id]/page.tsx
import LikeButton from "@/app/ui/like-button";
import { posts } from "@/lib/data";
import { notFound } from "next/navigation";

export default async function PostPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const post = posts.find((p) => p.id === id);

  if (!post) return notFound();

  return (
    <main>
      <h1>{post.title}</h1>
      <LikeButton likes={post.likes} />
    </main>
  );
}
```

> 💡 `notFound()`는 Next.js 내장 함수로, 존재하지 않는 페이지 접근 시 404를 렌더링한다.

---

## 1. Optimistic Update (낙관적 업데이트)
- 사용자의 이벤트(예: 좋아요 버튼 클릭)가 발생하면,  
  **서버 응답을 기다리지 않고 클라이언트(UI)를 즉시 업데이트**한다.
- 서버 요청의 성공을 ‘낙관적(optimistic)’으로 가정하고,  
  화면에 먼저 변화를 보여준다.
- 서버 응답이 실패하면 → **UI를 원래 상태로 되돌림(rollback)**.
- 네트워크 지연 중에도 빠른 반응성을 제공하는 것이 핵심 목적.

### ✅ 장점
- 서버 속도와 관계없이 **즉각적인 피드백** 제공 → 사용자 경험 향상.  
- 네트워크 상태가 나빠도 체감 속도가 빠름.

### ⚠️ 단점
- 서버 오류 발생 시 잠시 잘못된 정보가 표시될 수 있음.  
- 오류 복구(rollback) 로직이 반드시 필요.

---

## 2. Pessimistic Update (비관적 업데이트)
- 이벤트가 발생하면 **서버 요청 → 응답 후 UI 업데이트**를 수행.  
- 서버로부터의 성공 응답을 기반으로 UI를 변경하기 때문에  
  데이터의 일관성이 높다.

### ✅ 장점
- **데이터의 정확성과 신뢰성** 보장.  
- 잘못된 데이터 표시 가능성이 낮음.

### ⚠️ 단점
- 서버 응답을 기다려야 하므로 **체감 속도가 느릴 수 있음.**  
- 네트워크 지연 시 UX 저하.

| 구분 | 낙관적 업데이트 | 비관적 업데이트 |
|------|----------------|----------------|
| UI 업데이트 시점 | 요청 직후 즉시 | 서버 응답 받은 후 |
| 응답 실패 시 | 원래 상태로 롤백 | UI 변화 없음 |
| 장점 | 빠른 반응성, 즉각 피드백 | 일관성, 정확성 보장 |
| 대표 예시 | 좋아요, 버튼 클릭 | 데이터 수정, 폼 제출 |

---

## 3. like-button.tsx 구성
- `/ui/like-button.tsx`에서 **state를 2개** 사용한다.

```tsx
const [count, setCount] = useState<number>(likes ?? 0);
const [isLiking, setIsLiking] = useState(false);
```

### count
- 좋아요 클릭 횟수를 저장하는 state  
- 초기값은 data의 `likes` 필드

### isLiking
- “서버 요청이 진행 중인지”를 나타내는 state  
- 초기값: `false`

#### 💡 isLiking의 주요 역할
1. **중복 클릭 방지** — `isLiking === true`일 때 버튼을 `disabled` 처리.  
2. **UI 피드백 제공** — 로딩 상태 표시(스피너 등)에 사용 가능.  
3. **상태 안정성 유지** — 요청 완료 전 추가 변경을 방지해 일관된 상태 보장.

---

## 4. Null 병합 연산자 (Nullish Coalescing Operator)
```tsx
const [count, setCount] = useState<number>(likes ?? 0);
```

### 설명
- `??` 연산자는 **왼쪽 피연산자가 null 또는 undefined일 경우** 오른쪽 값을 반환.
- 즉, `likes`가 `null` 또는 `undefined`면 `0`을 반환하고,  
  값이 있으면 그대로 사용.

### OR(`||`) 연산자와 차이
- `||`은 `false`, `0`, `""`, `null`, `undefined` 등 **falsy 값 전체**를 오른쪽으로 대체.
- 반면 `??`는 `null`과 `undefined`만 대체하므로  
  숫자 `0` 같은 유효한 값은 그대로 유지할 수 있다.

> ✅ 결론: `likes ?? 0`은 **좋아요 수가 null일 때만 0으로 초기화**되며,  
> 정상적인 0값은 그대로 반영된다.

---

## 2. Next.js에서 server와 client component는 어떻게 작동합니까?

---

### 2-2. client component의 작동 (첫 번째 load)

1. **HTML 렌더링**
   - 사용자가 페이지를 요청하면, **라우팅된 페이지의 비대화형 미리보기(HTML)**를 즉시 표시한다.  
   - 즉, 초기 로드 시 서버가 생성한 HTML을 먼저 보여줌으로써 **빠른 첫 화면(FCP)**을 제공한다.

2. **RSC 페이지 로드**
   - 이후 **RSC(Rendering Server Component)** 가  
     `client component`와 `server component` 트리를 조정한다.

3. **JavaScript Hydration**
   - `client component`는 **hydration** 과정을 통해  
     정적 HTML을 **인터랙티브(대화형)**한 React 애플리케이션으로 변환한다.

---

### 💧 Hydration이란 무엇인가?
- **Hydration**은  
  서버에서 전달된 **정적 HTML**에  
  **이벤트 핸들러(onClick, onChange 등)** 를 연결하여  
  DOM을 **React가 제어 가능한 상태로 만드는 과정**이다.
- 즉, 사용자가 상호작용할 수 있도록  
  HTML을 React의 완전한 애플리케이션으로 “활성화”하는 React의 핵심 프로세스다.



<h1> 2025년 10월 01일 6주차 </h1>
<h1> 수업내용: Next.js Client-side transitions, 네비게이션 작동 방식 실습 </h1>

---

## 1-4. Client-side transitions (클라이언트 측 전환)
- 일반적으로 서버 렌더링 페이지로 이동하면 전체 페이지가 **로드**된다.  
  → 이로 인해 **state가 초기화**, **스크롤 위치 재설정**, **상호작용 차단** 발생.
- Next.js는 `<Link>` 컴포넌트를 사용한 **클라이언트 측 전환**으로 이를 방지한다.  
  페이지를 다시 로딩하는 대신, **콘텐츠를 동적으로 업데이트**한다.
- 특징:
  - **공유 레이아웃과 UI 유지**
  - **Prefetching**(미리 가져오기)으로 다음 페이지 로딩 상태를 빠르게 전환
- 효과:
  - 서버 렌더링된 앱을 **클라이언트에서 렌더링된 앱처럼** 느끼게 한다.
  - 프리패칭 + 스트리밍과 함께 사용하면 **동적 경로 전환도 빠르게 가능**.

---

## 1절 네비게이션 작동 방식 실습
- 앞에서 배운 내용을 다시 확인.
- 디렉토리 구조는 다음과 같다.  
  (blog 이름은 다른 이름으로 바꿔도 무방)
  ```
  app/
   ├─ page.tsx       // Root Page
   ├─ layout.tsx     // RootLayout
   └─ blog/
       ├─ page.tsx   // 블로그 목록
       └─ loading.tsx // 로딩 스켈톤
  ```
- 실습 순서:
  1. **Root Page 작성**
  2. `blog` 디렉토리에 `page`와 **로딩 스켈톤**(`loading.tsx`) 추가
  3. `RootLayout`에서 `<Link>`를 이용해 블로그 네비게이션 구현
- 참고:
  - 로딩 스켈톤 동작을 확인하기 위해 `blog/page`에 **time delay** 추가
  - 문서에는 `RootLayout`에 `<a>` 태그를 사용한 예시가 있으나, 이는 Next.js에서 권장하지 않음.

---

## 1절 네비게이션 작동 방식 실습 (오류 예시)
- `RootLayout`에서 `<a>` 태그를 사용하면 다음과 같은 오류 발생:
  ```txt
  Do not use an `<a>` element to navigate to `/blog/`. 
  Use `<Link />` from `next/link` instead.
  ```
- 따라서 내부 이동 시:
  - `<a>` 태그 ❌  
  - `<Link>` 컴포넌트 ✅
- 외부 링크 사용 시:
  - `<a>` 태그에 `target` 속성 추가 가능
  - 내부 라우팅은 반드시 `<Link>`를 사용해야 함

---

## 2. 전환을 느리게 만드는 요인은 무엇일까요?
- Next.js는 최적화를 통해 **네비게이션 속도가 빠르고 반응성**이 뛰어남.
- 하지만 특정 조건에서는 여전히 전환 속도가 느려질 수 있다.
- 다음은 일반적인 원인과 개선 방법이다.

---

## 2-1. 동적 경로 없는 loading.tsx
- 동적 경로 이동 시, 클라이언트는 서버 응답을 기다려야 결과를 표시 가능.  
  → 사용자는 앱이 **응답하지 않는 것처럼** 느낄 수 있음.
- 개선 방법:
  - **부분 프리패칭 활성화**
  - 네비게이션 트리거 시, 경로 렌더링 동안 **로딩 UI** 표시
  - 동적 경로에 `loading.tsx` 추가 권장

```tsx
// app/blog/[slug]/loading.tsx
export default function Loading() {
  return <LoadingSkeleton />;
}
```

---

### 개발 모드에서 확인
- Next.js **개발자 도구(Devtools)**를 사용하면 경로가 정적/동적인지 확인 가능.  
  관련 옵션: **devIndicators**
- Next.js 15.2.0부터 `position` 옵션 추가.  
  (기존 `appIsrStatus`, `buildActivity`, `buildActivityPosition`은 더 이상 사용 X)

```ts
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  devIndicators: { position: "bottom-left" },
};

export default nextConfig;
```

- 보통 화면 좌측 하단에 N자 아이콘 표시됨.  
- 인디케이터 위치는 설정에서 변경 가능.  
- 현재는 라우팅 결과 정도만 제공.

---

## 2-2. 동적 세그먼트 없는 generateStaticParams
- 동적 세그먼트는 **사전 렌더링 가능**.  
- 하지만 `generateStaticParams`가 누락되면 → 해당 경로는 **동적 렌더링**으로 대체됨.
- 해결: `generateStaticParams`를 추가하여 **빌드 시점에 정적 경로 생성**.

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch("../posts").then((res) => res.json());

  return posts.map((post) => ({
    slug: post.slug,
  }));
}

export default async function Page({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;
  // ...
}
```

---

## 2-2. 동적 세그먼트 없는 generateStaticParams 실습
- `generateStaticParams`를 사용하면 **빌드 시점에 정적 HTML을 미리 생성**한다.
- 사용하지 않으면 → 요청할 때마다 **서버에서 동적으로 처리**됨.
- 자주 변하지 않는 페이지는 `generateStaticParams` 사용 권장  
  → 정적 사이트처럼 빠르게 제공 가능.
- 반면, 사용자 입력/DB 조회 등 **실시간 데이터가 필요한 경우**는  
  `generateStaticParams` 없이 런타임 처리하는 것이 적절.

---

## 코드 분석 - generateStaticParams가 없는 경우
```tsx
// app/blog2/[slug]/page.tsx
// blog2의 동적 라우트 각 포스트에 대응하는 페이지를 렌더링.
// generateStaticParams를 사용하지 않아 런타임에서 params를 Promise로 전달받음.
// 안전하게 사용하려면 await로 해제 필요.

import { posts } from "../posts";

export default async function PostPage({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  // 런타임에서 전달되는 params는 Promise 타입일 수 있음.
  const { slug } = await params; // 런타임에서 slug 해석
  const post = posts.find((p) => p.slug === slug);

  if (!post) {
    return <h1>포스트를 찾을 수 없습니다.</h1>;
    // 실제 프로젝트에서는 notFound() 호출 권장
  }

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

- 특징:
  - `params`는 런타임에만 해석 가능 → 정적 빌드 시점에는 알 수 없음.
  - `generateStaticParams`를 쓰지 않으면 → 페이지 접근 시마다 서버에서 동적 처리.
  - 실습에서는 `notFound()` 같은 Next.js 내장 핸들러를 쓰는 것이 더 적절.

---

## await이 없어도 async를 붙여 두는 이유
- Next.js 13+ App Router의 `page.tsx` 같은 **Server Component**는 비동기 렌더링을 기본으로 한다.
- 따라서 `page.tsx` 안에서 데이터를 `fetch`하는 경우가 많아 `async`를 붙여도 전혀 문제 없음.

---

### 1. 일관성 유지
- 프로젝트 내에서 어떤 페이지는 `async`, 어떤 페이지는 일반 function이면 혼란스럽다.  
- 따라서 **Next.js 공식 문서도 대부분 `async function`으로 예시 제공.**

---

### 2. 확장성
- 현재는 단순히 `posts.find(...)` 같은 더미 데이터를 사용.  
- 하지만 나중에 **DB / API에서 데이터 fetch**가 추가될 수 있다.  
- 미리 `async`를 붙여두면 → 나중에 `await fetch(...)`를 추가할 때 **수정 불필요.**

---

### 3. React Server Component 호환성
- Server Component는 **Promise를 반환**할 수 있어야 한다.  
- Next.js는 내부적으로 `async` 함수 패턴에 맞게 최적화된 렌더링 파이프라인을 제공.  
- 따라서 `async`가 붙어 있어도 **불필요한 오버헤드가 거의 없음.**

---

<h1> 2025년 09월 24일 5주차 </h1>
<h1> 수업내용: Next.js searchParams, [slug] 비동기 params 정리, Link 컴포넌트/전역 메뉴 실습 </h1>

## searchParams란?
- URL의 **쿼리 문자열(Query String)** 을 읽는 방법.
- 예) `/products?category=shoes&page=2`  
  - `category=shoes`, `page=2`가 **search parameters**.

### App Router에서 사용 (동기 형태)
```tsx
// app/products/page.tsx
export default function ProductsPage({
  searchParams,
}: {
  searchParams: { [key: string]: string | string[] | undefined };
}) {
  const category = searchParams.category ?? "all";
  const page = Number(searchParams.page ?? 1);
  return <p>카테고리: {category} / 페이지: {page}</p>;
}
```

### App Router에서 사용 (비동기 형태를 명시)
> 일부 버전/설정(14.2+/15.x)에서는 `searchParams`가 **Promise**로 전달될 수 있음.
```tsx
// app/products/page.tsx  (비동기 props 대응)
export default async function ProductsPage({
  searchParams,
}: {
  searchParams: Promise<{ [k: string]: string | string[] | undefined }>;
}) {
  const sp = await searchParams;
  const category = sp.category ?? "all";
  const page = Number(sp.page ?? 1);
  return <p>카테고리: {category} / 페이지: {page}</p>;
}
```

---

## `[slug]` 심화: 성능/타이핑 메모
- 데이터가 커지면 `.find`(O(n)) 대신 **DB 쿼리/인덱스**로 대체.
- `params`가 동기처럼 보일 때도 **비동기일 수 있음을 타입으로 명시**하면 가독성과 안전성↑
- 실수로 `await`을 빼먹었을 때 **TypeScript가 잡아줌**.

간단 예시(지난 실습 보강):
```tsx
// app/blog/[slug]/page.tsx
import { notFound } from "next/navigation";
import { posts } from "../posts";

type Params = { slug: string };

export default async function PostPage({
  params,
}: { params: Promise<Params> }) {
  const { slug } = await params;           // 비동기 해제
  const post = posts.find((p) => p.slug === slug);
  if (!post) return notFound();
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

---

## Link 컴포넌트 실습
### 1) 블로그 목록에 링크 추가 (이미 적용되어 있으면 스킵)
```tsx
// app/blog/page.tsx
import Link from "next/link";
import { posts } from "./posts";

export default function BlogPage() {
  return (
    <>
      <h2>블로그 목록</h2>
      <ul>
        {posts.map((p) => (
          <li key={p.slug}>
            <Link href={`/blog/${p.slug}`}>{p.title}</Link>
          </li>
        ))}
      </ul>
    </>
  );
}
```

### 2) 모든 페이지에서 보이는 전역 메뉴 만들기
```tsx
// app/layout.tsx  (전역 네비게이션 추가)
import type { Metadata } from "next";
import Link from "next/link";
import "./globals.css";

export const metadata: Metadata = { title: "My App" };

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        <header style={{ padding: 12, borderBottom: "1px solid #ddd" }}>
          <nav style={{ display: "flex", gap: 12 }}>
            <Link href="/">Home</Link>
            <Link href="/blog">Blog</Link>
            <Link href="/products?category=shoes&page=1">Products</Link>
          </nav>
        </header>
        <main style={{ padding: 16 }}>{children}</main>
        <footer style={{ padding: 12, borderTop: "1px solid #ddd" }}>© 2025</footer>
      </body>
    </html>
  );
}
```

> 팁  
> - **내부 이동**은 `Link` 사용(클라이언트 네비게이션, 프리페치).  
> - 쿼리 문자열 조합 시 `Link`에 객체 형태도 가능:  
>   `href={{ pathname: "/products", query: { category: "shoes", page: 2 } }}`

---

## 1) Route 방식 비교: React vs Next.js
- **React(기본)**  
  - 라우팅: 외부 라이브러리 설치 필요(예: `react-router-dom`)  
  - 정의: 코드에서 `<Route>`로 직접 경로 매핑  
  - 예시:
  ```tsx
  // React Router
  import { Routes, Route } from "react-router-dom";
  export default function App() {
    return (
      <Routes>
        <Route path="/about" element={<About />} />
      </Routes>
    );
  }
  ```
- **Next.js**  
  - 라우팅: **파일/폴더 기반 자동 매핑**(내장)  
  - 정의: 파일 경로로 자동 라우트 생성  
  - 예시:
  ```txt
  pages/about.js     → /about
  app/about/page.tsx → /about
  ```

## 2) Next.js 라우팅: Pages Router vs App Router
- **Pages Router**
  - 도입: 초기 버전(Next 1~12)
  - 디렉토리: `pages/`
  - 특징: 단순·익숙함(기존 React 방식과 유사)
  - 대표: 동적 라우트, `getStaticProps`, `getServerSideProps`
  - 상태: 유지보수 중(신규에 권장 X)
- **App Router**
  - 도입: Next 13+
  - 디렉토리: `app/`
  - 특징: **레이아웃 중첩**, 서버 컴포넌트, 로딩 UI, 병렬/인터셉트 라우트 등 강력
  - 권장: Next 14+ 기본 권장
- Pages Router 예시
  ```txt
  pages/
  ├─ index.tsx       → /
  ├─ about.tsx       → /about
  └─ blog/[slug].tsx → /blog/:slug
  ```
- App Router 예시
  ```txt
  app/
  ├─ layout.tsx
  ├─ page.tsx          → /
  └─ about/
     └─ page.tsx       → /about
  ```

## 3) Introduction: Next.js 네비게이션 개요
- Next.js의 경로는 **기본적으로 서버에서 렌더링**됨.
- 성능을 위해 **prefetching, streaming, client-side transitions**를 제공 → 빠르고 부드러운 전환.

## 4) How navigation works(작동 방식)
- 익혀둘 핵심 개념
  - **Server Rendering(서버 렌더링)**
  - **Prefetching(프리패칭)**
  - **Streaming(스트리밍)**
  - **Client-side transitions(클라이언트 측 전환)**

### 4-1) Server Rendering(서버 렌더링)
- 레이아웃/페이지는 기본적으로 **서버 컴포넌트**로 동작.
- 서버 렌더링 유형
  - **정적 렌더링**(빌드/재검증 시점, 결과는 캐시됨)
  - **동적 렌더링**(요청 시점 생성)
- 단점: 새 경로 표시 전 **서버 응답 대기**가 필요.
- Next.js 보완
  - **Prefetching**: 방문 가능성이 높은 경로 데이터를 **미리 가져오기**  
  - **Client-side transitions**: 클라이언트 전환으로 체감 지연 감소
- 메모: 최초 방문을 위해 **HTML이 서버에서 생성**됨.

<h1> 2025년 09월 17일 4주차 </h1>
<h1> 수업내용: Git 브랜치 전환(checkout vs switch)와 Next.js 페이지 생성 </h1>

## Git: checkout vs switch 차이
- **checkout**: 브랜치 전환 + 파일 복구/수정까지 가능 → 실수 위험 높음(파일 덮어쓰기, Detached HEAD 등)
- **switch**: 브랜치 전환 전용 → 비교적 안전함
- 도입 시기: checkout(초기부터), switch(Git 2.23, 2019)

### 왜 checkout은 남아있나?
- 커밋 해시로 특정 커밋으로 이동, 파일 복구 등 **브랜치 전환 외의 작업**을 지원하기 때문

### 브랜치 생성/이동 명령 요약
```bash
# 새 브랜치 생성 + 이동(권장)
git switch -c <branch>

# checkout 방식(구버전 호환)
git checkout -b <branch>

# 브랜치 이동
git switch <branch>
git checkout <branch>

# 브랜치 생성만(이동 없음)
git branch <branch>
```

- 특별한 이유가 없다면 **switch 사용 권장**
- `git branch`는 생성/삭제/조회 전용(이동 불가)

---

## Next.js: Page 만들기
- Next.js는 **파일 시스템 기반 라우팅**을 사용
- 특정 경로의 화면은 `app/` 폴더 내부의 `page.tsx`로 정의
- 컴포넌트는 **default export** 해야 함

### 기본 예시
```tsx
// app/page.tsx  →  경로: /
export default function Page() {
  return <h1>Hello Next.js!</h1>;
}
```

- 다른 경로 예: `app/about/page.tsx` → `/about`
- 2장에서 다뤘던 내용이지만, 직접 다시 작성해 보며 흐름 확인

---

## Next.js: Layout 만들기
- 여러 페이지에서 공유되는 UI를 정의함
- 네비게이션 중에도 상태/상호작용 유지, 불필요한 재렌더링 없음
- `layout.tsx` 파일에서 `default export`로 컴포넌트를 내보내고, `children`을 반드시 허용해야 함

### 기본 예시
```tsx
// app/layout.tsx
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = { title: "My App" };

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ko">
      <body>
        <header>공통 헤더</header>
        <main>{children}</main>
        <footer>공통 푸터</footer>
      </body>
    </html>
  );
}
```

### 중첩 레이아웃 예시
```tsx
// app/dashboard/layout.tsx  → /dashboard 이하에서만 적용
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <section>
      <aside>대시보드 메뉴</aside>
      <div>{children}</div>
    </section>
  );
}
```
- `children`은 해당 레이아웃 내부에 렌더링될 `page` 또는 다른 `layout`을 의미

---

## Next.js: 중첩 라우트 만들기 (Nested routes)
- 라우트는 다중 URL 세그먼트로 구성됨  
  예: `/blog/[slug]` → `/`(root) / `blog` / `[slug]`(leaf)
- **폴더 = URL 세그먼트**, **파일(`page.tsx`, `route.ts`) = UI/엔드포인트**

### 폴더 구조 예
```txt
app/
 ├─ page.tsx                → /
 └─ blog/
    ├─ page.tsx             → /blog
    └─ [slug]/
       └─ page.tsx          → /blog/:slug
```

### /blog 인덱스 페이지 예
```tsx
// app/blog/page.tsx
type Post = { id: string; title: string };

async function getPosts(): Promise<Post[]> {
  // 실제로는 DB/파일/외부 API 등에서 가져옴
  return [{ id: "hello", title: "첫 글" }, { id: "next", title: "Next.js 소개" }];
}

export default async function BlogPage() {
  const posts = await getPosts();
  return (
    <ul>
      {posts.map((p) => (
        <li key={p.id}>
          <a href={`/blog/${p.id}`}>{p.title}</a>
        </li>
      ))}
    </ul>
  );
}
```

### 동적 세그먼트 페이지 예
```tsx
// app/blog/[slug]/page.tsx
export default function PostPage({ params }: { params: { slug: string } }) {
  return <h1>포스트: {params.slug}</h1>;
}
```
- 폴더를 중첩할수록 세그먼트가 깊어짐
- 공개 라우트로 접근하려면 해당 세그먼트에 `page.tsx`(또는 API면 `route.ts`)가 있어야 함

---

## 문서 코드 그대로 복사 시 오류 해결
- 원인: 예제에 등장하는 `@/lib/posts`, `@/ui/post` 파일을 만들지 않아 **모듈을 찾지 못함**.
- 해결 방향(강의 흐름용 최소 구현):
  1) `app/blog/posts.ts`에 **더미 데이터**를 만든다.  
  2) `app/blog/page.tsx`에서는 목록만 렌더링한다.  
  3) `app/blog/[slug]/page.tsx`에서는 `params.slug`로 해당 포스트를 찾아 렌더링한다.

### 1) 더미 데이터
```ts
// app/blog/posts.ts
export type Post = {
  slug: string;
  title: string;
  content: string;
};

export const posts: Post[] = [
  { slug: "nextjs",        title: "Next.js 소개",          content: "Next.js는 React 기반의 풀스택 프레임워크." },
  { slug: "routing",       title: "App Router 알아보기",   content: "Next.js 13부터 App Router가 도입됨." },
  { slug: "ssr-ssg",       title: "SSR vs SSG",            content: "서버 사이드 렌더링과 정적 사이트 생성 비교." },
  { slug: "dynamic-routes",title: "동적 라우팅",           content: "폴더명을 [slug]로 두면 동적 세그먼트가 생성됨." }
];
```

### 2) 목록 페이지(블로그 루트)
```tsx
// app/blog/page.tsx   → /blog
import Link from "next/link";
import { posts } from "./posts";

export default function BlogPage() {
  return (
    <ul>
      {posts.map((p) => (
        <li key={p.slug}>
          <Link href={`/blog/${p.slug}`}>{p.title}</Link>
        </li>
      ))}
    </ul>
  );
}
```

### 3) 상세 페이지(동적 세그먼트)
```tsx
// app/blog/[slug]/page.tsx   → /blog/:slug
import { notFound } from "next/navigation";
import { posts } from "../posts";

type Props = { params: { slug: string } };

export function generateStaticParams() {
  return posts.map((p) => ({ slug: p.slug }));
}

export default function PostPage({ params }: Props) {
  const post = posts.find((p) => p.slug === params.slug);
  if (!post) return notFound();
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

---

## 중첩 라우트 만들기(복습)
- 폴더를 중첩하여 **세그먼트**를 만든다. 파일(`page.tsx`,`route.ts`)이 실제 UI/엔드포인트를 담당.
- 예시 구조:
```txt
app/
 └─ blog/
    ├─ page.tsx          → /blog
    ├─ posts.ts          → 데이터(예시)
    └─ [slug]/
       └─ page.tsx       → /blog/:slug
```
- 폴더명을 대괄호로 감싼 `[slug]`는 **동적 경로 세그먼트**를 의미.

---

## `[slug]` 이해
- `slug`는 특정 페이지를 **사람이 읽기 쉬운 식별자**로 구분하기 위한 URL 조각.
- 데이터에서도 **동일 키**(여기선 `slug`)가 반드시 존재해야 라우트와 매칭 가능.
  - 예: `/blog/nextjs` → 데이터 객체의 `{ slug: "nextjs", ... }`와 매칭.
- 이름은 반드시 `slug`일 필요는 없지만, 폴더명과 데이터 키가 **일치**해야 한다.

---

## `[slug]`의 이해 (디렉토리/파일 연결)
- 디렉토리 구조 예
```txt
app/
 └─ blog/
    ├─ page.tsx      → 블로그 메인(목록)
    └─ [slug]/
       └─ page.tsx   → 블로그 상세
```

- 기본 상세 페이지(동기 props 가정) 예
```tsx
// app/blog/[slug]/page.tsx
import { posts } from "../posts";

export default function Posts({ params }: { params: { slug: string } }) {
  const post = posts.find((p) => p.slug === params.slug);
  if (!post) return <h1>게시글을 찾을 수 없습니다.</h1>;
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

---

## Next 14.2+/15.x에서 발생하는 `params` 관련 오류
- 증상: 동작은 되지만 아래와 같은 에러 로그가 뜸  
  `Route "/blog/[slug]" used 'params.slug'. 'params' should be awaited before using its properties.`
- 원인: App Router에서 `params`, `searchParams`가 **Promise 기반**으로 전달되는 환경/버전이 있음  
  → 사용 전에 `await` 해야 함

### 수정 코드 (비동기 props 대응)
```tsx
// app/blog/[slug]/page.tsx   (Next 14.2+/15.x 호환)
import { notFound } from "next/navigation";
import { posts } from "../posts";

type Params = { slug: string };

export default async function Posts({
  params,
}: {
  params: Promise<Params>;
}) {
  // 1) 비동기 params 해제
  const { slug } = await params;

  // 2) 데이터 조회
  const post = posts.find((p) => p.slug === slug);
  if (!post) return notFound();

  // 3) 렌더링
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}

// (선택) 정적 생성 시 슬러그 프리패치
export function generateStaticParams() {
  return posts.map((p) => ({ slug: p.slug }));
}
```

> 참고  
> - 파일을 비동기 컴포넌트로 바꾼 뒤에는 **개발 서버를 재시작**해야 경고/오류가 사라지는 경우가 있음.  
> - 만약 현재 환경에서 오류가 없다면, `params`가 동기 객체로 전달되는 설정일 수 있음(이전 예시 코드 사용).

---

## 코드 해설 (수정본 3~5라인 맥락)
- `async function`: 컴포넌트 내부에서 `await` 사용 가능
- `params: Promise<Params>`: 타입으로 **비동기 props**임을 명시
- `const { slug } = await params;`: 실제 사용 전에 `await`로 해제하여 속성 접근 오류를 방지

---



<h1> 2025년 09월 10일 3주차 </h1>  
<h1> 수업내용: Next.js 관련 기본 용어 정리 </h1>  

## 용어 정의
- 원문에서 `route`라는 단어가 자주 등장 → 사전적 의미는 "경로"  
- `route(라우트)` : "경로" 자체를 의미  
- `routing(라우팅)` : "경로를 찾아가는 과정"을 의미  
- `path`도 "경로"라는 뜻이지만, 구분을 위해 주로 `routing(라우팅)`으로 번역  

### directory vs folder
- 둘 사이에 특별한 구분은 없음  
- 최상위 폴더는 directory, 하위 폴더는 folder라고 부르기도 하지만 꼭 그런 건 아님  
- 운영체제에 따라 용어가 다르게 쓰일 뿐, 같은 의미로 이해하면 됨  

### segment
- routing과 관련된 directory의 별칭 정도로 이해하면 됨  

---

## Folder & File Conventions — Top-level folders
- 최상위 폴더는 앱의 코드와 정적 자산을 구성하는 데 사용됨
- `src/`를 쓰면 애플리케이션 소스를 한 곳으로 모아 구조가 깔끔해짐

| 폴더  | 의미(한글) | 의미(영문) |
|------|------------|------------|
| app  | 앱 라우터  | App Router |
| pages| 페이지 라우터 | Pages Router |
| public | 제공될 정적 리소스 | Static assets to be served |
| src  | 선택적 애플리케이션 소스 폴더 | Optional application source folder |

---

## Top-level files
- 프로젝트 생성 시 모두가 자동 생성되는 것은 아님(선택 옵션에 따라 다름)

| 파일 | 용도(한글) | 용도(영문) |
|-----|-----------|-----------|
| `next.config.js` | Next.js 구성 파일 | Configuration for Next.js |
| `package.json` | 프로젝트 종속성/스크립트 | Dependencies and scripts |
| `instrumentation.ts` | OpenTelemetry/계측 | OpenTelemetry & instrumentation |
| `middleware.ts` | 요청 미들웨어 | Request middleware |
| `.env` | 환경 변수 | Environment variables |
| `.env.local` | 로컬 환경 변수 | Local env |
| `.env.production` | 프로덕션 환경 변수 | Production env |
| `.env.development` | 개발 환경 변수 | Development env |
| `.eslintrc.json` 또는 `eslint.config.mjs` | ESLint 설정 | ESLint configuration |
| `.gitignore` | Git에서 무시할 항목 | Git ignore list |
| `next-env.d.ts` | Next.js 타입 선언 | TypeScript declarations for Next.js |
| `tsconfig.json` | TypeScript 설정 | TS configuration |
| `jsconfig.json` | JavaScript 설정 | JS configuration |

---

## Routing Files (App Router)
| 파일명 | 확장자 | 의미(한글) | 의미(영문) |
|-------|-------|-------------|------------|
| `layout` | `js` `jsx` `tsx` | 레이아웃 | Layout |
| `page` | `js` `jsx` `tsx` | 페이지 | Page |
| `loading` | `js` `jsx` `tsx` | 로딩 UI | Loading UI |
| `not-found` | `js` `jsx` `tsx` | 404 UI | Not Found UI |
| `error` | `js` `jsx` `tsx` | 에러 UI(세그먼트 범위) | Error UI |
| `global-error` | `js` `jsx` `tsx` | 전역 에러 UI | Global error UI |
| `route` | `js` `ts` | API 엔드포인트 | API endpoint |
| `template` | `js` `jsx` `tsx` | 다시 렌더되는 레이아웃 | Re-rendered layout |
| `default` | `js` `jsx` `tsx` | 병렬 라우팅 대체 페이지 | Parallel route fallback |

---

## Nested routes(중첩 라우팅)
```txt
folder           → 라우팅 세그먼트
folder/folder    → 중첩된 라우팅 세그먼트
```

---

## Dynamic routes(동적 라우팅)
```txt
[folder]     → 동적 세그먼트  (예: /posts/[id])
[...folder]  → 포괄(catch-all) 세그먼트  (예: /docs/a/b/c)
[[...folder]]→ 선택적 포괄 세그먼트(파라미터 없어도 매칭)
```

---

## Route Groups & Private Folders
- **Route Group**: `(group)` — URL에 포함되지 않고 경로를 그룹화  
  ```txt
  app/(marketing)/page.tsx   → URL: / (group 이름 노출 X)
  ```
- **Private Folder**: `_{name}` — 라우팅에서 제외(파일/컴포넌트 보관용)  
  ```txt
  app/_components/Button.tsx → import 용도로만 사용, URL 매칭 X
  ```

---

## Parallel & Intercepted Routes
- **Parallel routes(병렬 라우팅)**: `@slot` — 명명된 슬롯
  ```txt
  app/dashboard/@analytics/page.tsx
  app/dashboard/@feed/page.tsx
  ```
- **Intercepted routes(차단/가로채기 라우팅)** — 특정 레벨에서 다른 세그먼트 내용을 끌어옴
  ```txt
  (.)segment    → 동일 레벨에서 가로채기
  (..)segment   → 한 레벨 위에서 가로채기
  (...)segment  → 루트에서 가로채기
  ```
---

## Metadata file conventions
### App icons
| 파일명        | 확장자                | 설명(한글)          | 설명(영문)            |
|--------------|----------------------|---------------------|-----------------------|
| favicon      | .ico                 | 파비콘 파일         | Favicon file          |
| icon         | .ico .jpg .jpeg .png .svg | 앱 아이콘 파일     | App icon file         |
| icon         | .js .ts .tsx         | 생성된 앱 아이콘    | Generated app icon    |
| apple-icon   | .jpg .jpeg .png      | Apple 앱 아이콘 파일 | Apple App icon file   |
| apple-icon   | .js .ts .tsx         | 생성된 Apple 아이콘 | Generated Apple icon  |

### Open Graph / Twitter images
| 파일명            | 확장자                | 설명(한글)         | 설명(영문)              |
|------------------|----------------------|--------------------|-------------------------|
| opengraph-image  | .jpg .jpeg .png .gif | Open Graph 이미지  | Open Graph image file   |
| opengraph-image  | .js .ts .tsx         | 생성된 OG 이미지   | Generated OG image      |
| twitter-image    | .jpg .jpeg .png .gif | 트위터 이미지 파일 | Twitter image file      |
| twitter-image    | .js .ts .tsx         | 생성된 트위터 이미지 | Generated Twitter image |

### SEO
| 파일명   | 확장자  | 설명(한글)   | 설명(영문)        |
|----------|---------|-------------|-------------------|
| sitemap  | .xml    | 사이트맵     | Sitemap file      |
| sitemap  | .js .ts | 생성된 사이트맵 | Generated sitemap |
| robots   | .txt    | 로봇 파일   | Robots file       |
| robots   | .js .ts | 생성된 로봇 파일 | Generated robots |

---

## Open Graph
- 링크를 보낼 때 미리보기를 만드는 프로토콜  
- 대표적인 규격: **Open Graph Protocol (OGP)**  
- Facebook 주도, 대부분 SNS 플랫폼에서 활용  
- 모든 플랫폼이 동일한 방식으로 처리하지는 않음  
- 웹페이지 `<head>`의 메타 태그에 선언  

예시:
```html
<meta property="og:type" content="website" />
<meta property="og:url" content="https://example.com/page.html" />
<meta property="og:title" content="페이지 제목" />
<meta property="og:description" content="페이지 설명 요약" />
<meta property="og:image" content="https://example.com/image.jpg" />
<meta property="og:site_name" content="사이트 이름" />
<meta property="og:locale" content="ko_KR" />
```

---

## Component hierarchy
특수 파일에 정의된 컴포넌트는 계층 구조에 따라 렌더링됨  

- layout.js  
- template.js  
- error.js (React 오류 경계)  
- loading.js (서스펜스 로딩 경계)  
- not-found.js (404 UI)  
- page.js (또는 중첩 layout.js)  

렌더링 구조 예시:
```jsx
<Layout>
  <Template>
    <ErrorBoundary fallback={<Error />}>
      <Suspense fallback={<Loading />}>
        <ErrorBoundary fallback={<NotFound />}>
          <Page />
        </ErrorBoundary>
      </Suspense>
    </ErrorBoundary>
  </Template>
</Layout>
```

---

## layout vs template
| 파일         | 특징                       | 상태/DOM 유지   | 사용 사례                        |
|-------------|---------------------------|----------------|----------------------------------|
| layout.tsx  | 경로별 공유 레이아웃       | 유지됨 (정적)  | 네비게이션, 사이드바, 공통 레이아웃 |
| template.tsx| 매번 새 인스턴스 생성      | 초기화됨 (동적) | 페이지별 초기화가 필요한 경우       |

---

## Organizing your project (프로젝트 구성하기)
- **Colocation**: 파일/폴더를 기능 단위로 묶어 구조를 명확히 함  
- `app` 디렉토리에서 중첩 폴더는 라우팅 세그먼트를 정의  
- 각 폴더는 URL 경로 세그먼트에 매핑됨  

주의사항  
- 라우팅 세그먼트 폴더라도 `page.js` 또는 `route.js` 파일이 없으면 공개 접근 불가  
- 따라서 페이지나 API 엔드포인트 파일이 반드시 필요함  

---

## Organizing your project (심화)
- `app` 디렉토리의 파일은 **기본적으로 안전하게 코로케이션** 가능  
- 코로케이션 시 내부 파일을 일관되게 구성 → 관리 용이
- 파일 정리 및 그룹화를 통해 코드 편집기 사용성 향상
- 파일 이름 충돌 방지에 유리

알아두면 좋은 정보
- 프레임워크 규칙은 아니지만, 동일한 패턴으로 비공개 폴더 외부 파일을 `"비공개"` 표시 가능  
- 폴더명 앞에 `%5F`(밑줄 URL 인코딩) 붙이면 밑줄 시작 세그먼트 생성 가능  
- Next.js의 특수 파일 규칙을 알고 있으면 이름 충돌을 방지할 수 있음  

---

## 라우팅 그룹 (Routing Groups)
라우팅 그룹이 유용한 경우:
- 사이트 섹션별/팀별로 라우트 구성 (예: 마케팅, 관리 페이지 등)  
- 동일한 세그먼트 내 중첩 레이아웃 활성화 가능  

예시:
- 공통 세그먼트 내 여러 개의 레이아웃 포함 → 중첩 레이아웃 생성  
- 하위 그룹에도 레이아웃 추가 가능  

---

## 예제 (Examples)
- 일반적인 프로젝트 전략에 대한 개요
- 팀/프로젝트에 맞는 전략을 선택해 전체에 일관되게 적용하는 것이 핵심

알아두면 좋은 점
- `components/`, `lib/` 폴더는 자주 쓰이는 일반적인 플레이스홀더  
- 프레임워크에서 특별한 의미가 있는 이름은 아님  
- 프로젝트 상황에 따라 `UI/`, `utils/`, `hooks/`, `styles/` 등 다른 이름을 자유롭게 사용 가능  

---

## 프로젝트 생성
Next.js 프로젝트 생성 절차
```bash
npx create-next-app@latest
```

선택 항목 (예시는 모두 `yes`)
1. 프로젝트 이름  
2~4. TypeScript, ESLint, Tailwind 사용 여부  
5. `src/` 디렉토리 사용 여부  
6. App Router 사용 여부  
7. import alias 사용 여부  
8. alias 문자 지정 (기본 `@/*`)  

생성 후 실행:
```bash
npm run dev
```

---

## 서버 실행 전후
- 실행 시 `.next` 디렉토리 자동 생성  
- `.next`는 **빌드 아웃풋 및 캐시/중간 산출물 저장용**  
- `next dev`, `next build`, `next start` 실행 시 내부적으로 필요한 작업을 보관하는 폴더

---

## src/ 디렉토리 선택
- 모든 프로젝트 파일을 `src/` 디렉토리 안에 넣어 관리 가능  

### src 디렉토리 사용 (추천)
- 구조: 모든 코드가 `src/` 폴더 안에 들어감  
- 예: `src/app`, `src/components`, `src/lib`  
- 목적: 코드와 설정 파일 분리  
- 추천: 규모가 큰 프로젝트에 적합  

### src 디렉토리 미사용
- 코드가 루트에 바로 위치  
- 예: `app`, `components`, `lib`  
- 전체가 섞여 있어서 단순/작은 프로젝트에 적합  

---

## .eslintrc.json vs eslint.config.mjs
### .eslintrc.json
- JSON 형식 → 단순하지만 조건문/변수 사용 불가  
- 정적인 설정 파일  
- 구버전 ESLint와 호환  
- 간단하고 직관적  

### eslint.config.mjs
- ESM(Module JS) 형식  
- 함수/변수/동적 로직 활용 가능  
- ESLint v9 이상에서 권장  
- 더 유연하고 모듈화 가능  
- Next.js 최신 기본값

---

## 특정 라우트에 스켈레톤 로딩 적용하기

### loading.js / loading.tsx 활용
- 특정 라우트 폴더에 `loading.tsx` 파일을 넣으면 해당 라우트에만 로딩 화면이 적용됨.
- 예: `dashboard/(overview)/loading.tsx` → `dashboard` 라우트에서만 동작.  
- URL 구조에는 영향을 주지 않음 → 단순히 로딩 중일 때 사용자 경험 개선.

### 스켈레톤 로딩 (Loading Skeletons)
- 콘텐츠가 로드되기 전에 자리 표시자(placeholder) 역할.  
- 회색/반투명 박스를 미리 보여주어 **로딩 중임을 시각적으로 안내**.  
- 사용자는 화면 구성을 미리 예측 가능 → 체감 속도 향상.  
- 흔히 카드 리스트, 텍스트 블록, 이미지 영역 등에 사용됨.


<h1> 2025년 09월03일 2주차 </h1>
<h1> 수업내용: Next.js 설치, TypeScript/ESLint 설정, 경로 별칭 구성 </h1>

## IDE 플러그인
- Next.js에는 사용자 정의 TypeScript 플러그인과 타입 검사기가 포함되어 있음  
- VS Code에서 타입 검사와 자동완성 기능 활용 가능  
- 본격적인 작업 전 `next.config.js` 파일을 작성해두는 것이 좋음  

VS Code에서 플러그인 활성화 방법  
1. 명령 팔레트 열기 `(Ctrl/⌘ + Shift + P)`  
2. `TypeScript: Select TypeScript Version` 검색  
3. `Use Workspace Version` 선택  

---

## ESLint 설정
- Next.js에는 기본적으로 ESLint가 내장되어 있음  
- `create-next-app` 실행 시 필요한 패키지가 자동 설치됨  
- 기존 프로젝트라면 `package.json`에 다음과 같이 추가  

```json
{
  "scripts": {
    "lint": "next lint"
  }
}
```

`npm run lint` 실행 시 설치 및 설정 과정 확인 가능  

---

## ESLint 옵션
설치 후 나타나는 선택지  
- Strict: 기본 ESLint + Core Web Vitals 규칙 (추천)  
- Base: 기본 ESLint만 포함  
- Cancel: 설정 건너뜀  

Strict 또는 Base를 선택하면 `eslint`와 `eslint-config-next`가 자동 설치되고,  
프로젝트 루트에 `.eslintrc.json` 파일이 생성됨  
→ `next lint` 실행 시 코드 오류 검사 가능  

---

## 절대 경로(alias) 설정
`tsconfig.json` 또는 `jsconfig.json`에서 `baseUrl`과 `paths` 옵션을 활용해 import 경로를 단순화할 수 있음  

```tsx
// Before
import { Button } from '../../../components/button'

// After
import { Button } from '@/components/button'
```

예시 설정  
```json
{
  "compilerOptions": {
    "baseUrl": "src/"
  }
}
```

추가적으로 paths 설정 가능  
```json
{
  "compilerOptions": {
    "baseUrl": "src/",
    "paths": {
      "@/styles/*": ["styles/*"],
      "@/components/*": ["components/*"]
    }
  }
}
```

---

## 자동 생성되는 항목
Next.js 프로젝트를 생성하면 자동으로 추가되는 항목들  

- `package.json`에 scripts 추가 및 `public` 디렉토리  
- `tsconfig.json` (TypeScript 선택 시)  
- `eslint.config.mjs` (ESLint 선택 시)  
- Tailwind CSS 설정 (선택 시)  
- `src` 디렉토리 (선택 시)  
- `app/layout.tsx`, `app/page.tsx` (App Router 선택 시)  
- Turbopack 설정 (선택 시)  
- import alias (`tsconfig.json`에 paths 자동 생성)  

---

## Core Web Vitals
웹 성능을 평가하는 핵심 지표  

- LCP (Largest Contentful Paint): 뷰포트 내 가장 큰 요소가 표시되는 시간  
- FID (First Input Delay): 사용자가 페이지와 상호작용을 시도한 뒤 반응하기까지 걸리는 시간  
- CLS (Cumulative Layout Shift): 페이지 요소가 갑자기 이동하는 정도 (화면 안정성)  

---

## 실습 프로젝트 생성
공식 문서에서는 기본 패키지 관리자로 pnpm을 사용하지만, 다른 패키지 매니저도 사용 가능  

프로젝트 생성 명령어  
```bash
npx create-next-app@latest
```

실행하면 다음과 같은 선택지가 나옴  
1. 프로젝트 이름  
2. TypeScript 사용 여부  
3. ESLint 사용 여부  
4. TailwindCSS 사용 여부  
5. `src/` 디렉토리 사용 여부  
6. App Router 사용 여부  
7. import alias 사용 여부  
8. alias 기본값 설정 (`@/*`)

---

## src/ 디렉토리 선택
- 모든 프로젝트 파일을 `src/` 디렉토리 안에서 관리하는 방식을 권장함  
- 장점: 코드와 설정 파일이 분리되어 구조가 깔끔해짐  
- 대규모 프로젝트일수록 `src/` 구조가 유리  
- 작은 프로젝트는 `src/` 없이 루트에 바로 두어도 무방  

예시 구조  

```
my-project/
 ├─ node_modules/
 ├─ public/
 ├─ src/
 │   ├─ app/
 │   ├─ components/
 │   ├─ styles/
 │   └─ utils/
 ├─ eslint.config.mjs
 ├─ next.config.js
 └─ package.json
```

---

## .eslintrc.json vs eslint.config.mjs
- JSON은 주석, 변수, 조건문 등을 쓸 수 없어서 복잡한 설정에 제약이 있음  
- `.mjs`는 ECMAScript 모듈 방식으로, 동적 설정(함수, 변수 등) 사용 가능  
- 확장자 `.mjs` = module JavaScript  
- ESLint v9 이상에서 공식 권장하는 방식이며, Next.js에서도 기본값으로 채택됨  
- 다른 설정 파일을 import 해서 재사용 가능 → 유지보수에 유리  

비교 요약  

| 항목              | .eslintrc.json                 | eslint.config.mjs              |
|-------------------|--------------------------------|--------------------------------|
| 포맷              | JSON 형식                      | JavaScript 모듈(ESM)           |
| 실행 방식         | 정적인 설정 파일               | 동적인 설정 가능(함수, 변수 등)|
| 호환성            | 구버전 ESLint와 호환           | ESLint v9+ 공식 권장           |
| 특징              | 단순하고 직관적                | 유연하고 모듈화 가능           |
| 사용 여부         | 여전히 사용 가능               | 최신 Next.js에서 기본값        |

---

## pnpm
- pnpm은 **효율적인(Performant) NPM**의 약자로, 고성능 Node 패키지 매니저  
- npm, yarn과 같은 목적의 패키지 매니저이지만,  
  디스크 공간 낭비, 복잡한 의존성 관리, 느린 설치 속도 문제를 개선하기 위해 개발됨  

대표적인 특징  
1. **하드 링크 기반 저장소**  
   - 패키지를 한 번만 설치해 전역 저장소에 두고,  
     각 프로젝트의 `node_modules`에는 하드 링크(또는 심볼릭 링크) 생성  
   - 디스크 공간 절약  

2. **빠른 설치 속도**  
   - 이미 설치된 패키지는 다시 다운로드하지 않고 재사용  
   - 초기 설치뿐만 아니라 업데이트 시에도 속도가 빠름  

3. **효율적인 종속성 관리**  
   - 의존성 충돌 및 중복 문제를 줄여 관리 효율성을 높임  

4. **기존 패키지 매니저의 비효율 개선**  
   - npm, yarn의 단점을 보완  

---

## pnpm vs npm
주요 명령어 비교표. 기본적으로 명령어 구조는 비슷하지만 세부 키워드에 차이가 있음.  

| 작업            | NPM                              | PNPM                         | Yarn                          |
|-----------------|----------------------------------|------------------------------|-------------------------------|
| 패키지 설치(전체) | `npm install`                   | `pnpm install`               | `yarn`                        |
| 패키지 추가      | `npm install [pkg]`             | `pnpm add [pkg]`             | `yarn add [pkg]`              |
| 패키지 삭제      | `npm uninstall [pkg]`           | `pnpm remove [pkg]`          | `yarn remove [pkg]`           |
| 개발 의존성 추가 | `npm install --save-dev [pkg]`  | `pnpm add -D [pkg]`          | `yarn add --dev [pkg]`        |
| 전역 설치        | `npm install -g [pkg]`          | `pnpm add -g [pkg]`          | `yarn global add [pkg]`       |
| 스크립트 실행    | `npm run [script]`              | `pnpm [script]`              | `yarn [script]`               |
| 캐시 정리        | `npm cache clean`               | `pnpm store prune`           | `yarn cache clean`            |
| 의존성 업데이트  | `npm update`                    | `pnpm update`                | `yarn upgrade`                |
| 잠금 파일 업데이트 | `npm install --force`           | `pnpm install --force`       | `yarn install --force`        |

---

## Hard link vs Symbolic link
pnpm의 특징 중 하나는 하드 링크를 활용해 디스크 공간을 효율적으로 사용한다는 점임.  
탐색기에서 npm과 pnpm 프로젝트의 `node_modules` 용량 차이를 확인해보면 이해 가능.  

### 하드 링크 (Hard link)
파일은 크게 두 부분으로 나뉨  
1. **Directory Entry**: 파일 이름과 해당 inode 번호를 매핑하는 정보  
2. **inode**: 파일/디렉토리에 대한 메타데이터 저장 (권한, 소유자, 크기, 데이터 블록 위치 등)  

즉, 하드 링크는 동일한 inode를 가리키는 또 다른 이름을 만들어내는 방식.  
하나의 실제 데이터 블록을 여러 파일 이름이 공유하게 됨 → 디스크 공간 절약 효과.  

---

