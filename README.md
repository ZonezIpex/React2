<h1> 202130413 신민수

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

