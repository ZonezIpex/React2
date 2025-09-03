<h1> 202130413 신민수

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
