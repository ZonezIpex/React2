<h1> 202130413 신민수

<h1> 2025년 09월03일 2주차 </h1>
<h1> 수업내용: Next.js 설치, TypeScript/ESLint 설정, 경로 별칭 구성 </h1>

## 📝 IDE 플러그인
- Next.js에는 **사용자 정의 TypeScript 플러그인과 유형 검사기**가 포함되어 있음.
- VS Code 등에서 **고급 타입 검사/자동완성** 가능.
- 작업 전 TypeScript reference 참고 후 `next.config.js` 먼저 작성.
- VS Code에서 플러그인 활성화:
  1. 명령 팔레트 `(Ctrl/⌘ + Shift + P)`
  2. **"TypeScript: TypeScript 버전 선택"** 검색
  3. **"Use Workspace Version"** 선택

---

## 📝 ESLint 설정
- Next.js에는 **ESLint 내장**.
- `create-next-app`으로 새 프로젝트 생성 시 필요한 패키지 자동 설치 및 설정 구성.
- 기존 프로젝트에 수동 추가 시 `package.json`에 스크립트 추가:

```json
{
  "scripts": {
    "lint": "next lint"
  }
}
```

- 이후 `npm run lint` 실행하면 설치/구성 안내 확인 가능.

---

## 🧩 ESLint 구성 옵션
- 설치 후 메시지:
  - **Strict**: Next.js 기본 ESLint + Core Web Vitals 규칙(권장)
  - **Base**: Next.js 기본 ESLint 구성
  - **Cancel**: 구성 건너뜀
- Strict/Base 선택 시 `eslint`와 `eslint-config-next`가 **애플리케이션 의존성**으로 설치됨.
- 선택한 설정은 `.eslintrc.json`이 **프로젝트 루트**에 생성됨.
- 이제 `next lint`를 실행할 때마다 ESLint가 오류를 탐지.

---

## 🧭 import 및 절대 경로 별칭 설정
- Next.js는 `tsconfig.json` 또는 `jsconfig.json`의 `"paths"`, `"baseUrl"` 옵션을 지원.
- 이 옵션으로 **절대 경로 별칭**을 사용해 모듈을 더 깔끔하게 관리 가능.

```tsx
// Before
import { Button } from '../../../components/button'

// After
import { Button } from '@/components/button'
```

- 설정 방법(`tsconfig.json` 또는 `jsconfig.json`):

```json
{
  "compilerOptions": {
    "baseUrl": "src/"
  }
}
```

---

## 🧭 baseUrl & paths 설정
- `baseUrl` 외에도 `"paths"` 옵션을 사용해 모듈 경로를 **별칭(alias)** 으로 지정 가능.  
- 예: `@/components/*` → `components/*` 경로로 매핑.  

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

- `"paths"` 항목은 `baseUrl` 경로를 기준으로 상대적으로 해석됨.  

---

## 📝 자동 생성되는 항목
강의에서는 프로젝트를 자동 생성해서 사용.  
프로젝트 생성 시 자동으로 생성되는 항목은 다음과 같음:

- `package.json`: scripts 자동 추가, public 디렉토리  
- TypeScript (선택): `tsconfig.json` 파일 생성  
- ESLint 설정 (선택): `.eslintrc.json` 대신 `eslint.config.mjs` 파일 생성  
- Tailwind CSS (선택)  
- `src` 디렉토리 사용 (선택)  
- App Router (선택): `app/layout.tsx`, `app/page.tsx` 파일 생성  
- Turbopack (선택)  
- import alias (선택): `tsconfig.json`에 `"paths"` 자동 생성  

👉 수동으로 생성 시 추가적으로 해야 할 작업을 자동으로 처리해줌.  

---


