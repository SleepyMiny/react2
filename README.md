<h1>202130123 이민영</h1>
2주차 3주차 병결

<h1> **2025-09-17 / 4번째 수업**</h1>

### Git: `switch` vs `checkout`
- **핵심 차이**
  - `switch` : 브랜치 **이동 전용** → 안전성 높음
  - `checkout` : 브랜치 이동 + 파일 복구/커밋 해시 이동 등 **범용** → 실수 시 위험(덮어쓰기, Detached HEAD)
- **도입 시기**
  - `checkout` : 초기부터
  - `switch` : Git 2.23(2019)
- **왜 `checkout`이 여전히 존재?**  
  브랜치 전환 외에도 커밋 해시로 이동, 특정 파일 복구 등 **다기능** 지원 때문.

#### 자주 쓰는 명령 요약
```bash
# 새 브랜치 생성 + 즉시 이동 (권장)
git switch -c <branch>

# (구버전 호환) 같은 동작
git checkout -b <branch>

# 기존 브랜치로 이동
git switch <branch>
# 또는
git checkout <branch>

# 브랜치 생성만(이동 없음)
git branch <branch>
```
> 원칙: 특별한 이유 없으면 **`git switch`** 사용. `git branch`는 생성/삭제/조회 전용.

---

### Next.js: 페이지 & 레이아웃 기본

#### 최소 페이지
```tsx
// app/page.tsx  →  /
export default function Home() {
  return <h1>Hello Next.js!</h1>;
}
```

#### 다른 경로 페이지
```tsx
// app/about/page.tsx  →  /about
export default function About() {
  return <h1>소개</h1>;
}
```

#### 루트 레이아웃(공유 UI)
```tsx
// app/layout.tsx
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = { title: "My App" };

export default function RootLayout({ children }: { children: React.ReactNode }) {
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

#### 부분 레이아웃(섹션 전용)
```tsx
// app/dashboard/layout.tsx → /dashboard 이하 적용
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <section>
      <aside>대시보드 메뉴</aside>
      <div>{children}</div>
    </section>
  );
}
```
> `layout.tsx`는 네비게이션 간 **상태/DOM 유지**, `template.tsx`는 **매 방문 새 인스턴스**.

---

### 중첩 라우트 & 동적 세그먼트 실습

#### 폴더 구조
```
app/
 ├─ page.tsx                → /
 └─ blog/
    ├─ page.tsx             → /blog
    └─ [slug]/
       └─ page.tsx          → /blog/:slug
```

#### 더미 데이터
```tsx
// app/blog/posts.ts
export type Post = { slug: string; title: string; content: string };

export const posts: Post[] = [
  { slug: "nextjs",  title: "Next.js 한눈에 보기", content: "React 기반 풀스택 프레임워크입니다." },
  { slug: "routing", title: "App Router 익히기",   content: "폴더=세그먼트, page=UI." },
];
```

#### 블로그 인덱스
```tsx
// app/blog/page.tsx → /blog
import Link from "next/link";
import { posts } from "./posts";

export default function BlogIndex() {
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

#### 블로그 상세(동기 params 버전)
```tsx
// app/blog/[slug]/page.tsx
import { notFound } from "next/navigation";
import { posts } from "../posts";

export default function BlogPost({ params }: { params: { slug: string } }) {
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

### Next 14.2+/15에서의 `params` 경고 대응(비동기 props)

일부 환경에서 `params`가 **Promise**로 전달되어 경고가 발생할 수 있음.  
해결: 컴포넌트를 `async`로 선언하고 `await params` 후 사용.

```tsx
// app/blog/[slug]/page.tsx — Promise 기반 params 대응
import { notFound } from "next/navigation";
import { posts } from "../posts";

type Params = { slug: string };

export default async function BlogPost({
  params,
}: {
  params: Promise<Params>;
}) {
  const { slug } = await params;  // ✅ await로 안전하게 해제
  const post = posts.find((p) => p.slug === slug);
  if (!post) return notFound();

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}

export function generateStaticParams() {
  return posts.map((p) => ({ slug: p.slug }));
}
```
> 파일을 비동기로 바꾼 뒤에는 **개발 서버 재시작**이 필요한 경우가 있음.

---

## 체크리스트(공통)
- [ ] 세그먼트 폴더에 `page.tsx`/`route.ts` 누락 → 공개 접근 불가
- [ ] `@/lib/...` 등 경로 import 실패 → `tsconfig`의 `baseUrl`/`paths` 확인
- [ ] 브랜치 이동 중 파일 덮어쓰기 → 브랜치 전환은 **`git switch`** 중심
- [ ] Next 14.2+/15 경고 → **`async` 컴포넌트 + `await params`**

<h1> 2025-09-03/10 / 2주차/3주차 </h1>

### 개발 환경 메모
- **VS Code TypeScript 워크스페이스 버전 사용**  
  `Ctrl/⌘+Shift+P → "TypeScript: Select TypeScript Version" → "Use Workspace Version"`
- **ESLint 실행 스크립트**
  ```json
  {
    "scripts": { "lint": "next lint" }
  }
  ```
- **ESLint 구성 방식**
  - `.eslintrc.json`: 정적(간단)
  - `eslint.config.mjs`: ESM 기반 동적 구성(최신 권장)

### 프로젝트 생성 & 구조
- `npx create-next-app@latest` 실행 후 옵션 선택(TypeScript, ESLint, Tailwind, App Router, src/ 등)
- **src 디렉토리 사용 추천**: 설정/환경파일과 애플리케이션 소스 분리
  ```
  my-project/
  ├─ public/
  ├─ src/
  │  ├─ app/
  │  ├─ components/
  │  ├─ styles/
  │  └─ lib/
  ├─ next.config.js
  ├─ eslint.config.mjs
  └─ package.json
  ```

### 경로 별칭 설정(예시)
```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "src/",
    "paths": {
      "@/components/*": ["components/*"],
      "@/styles/*": ["styles/*"],
      "@/lib/*": ["lib/*"]
    }
  }
}
```

### 패키지 매니저(PNPM) 간단 명령
```bash
pnpm install        # 의존성 설치
pnpm add <pkg>      # 패키지 추가
pnpm remove <pkg>   # 패키지 제거
pnpm dev            # 개발 서버
pnpm build          # 빌드
pnpm start          # 프로덕션 서버
```
> PNPM은 하드링크 기반 저장소로 설치 속도/디스크 효율이 좋음.

---

## 2025-09-10 · 3주차 — 라우팅 용어·규칙 정리

### 용어 정리
- **route(라우트)**: URL 경로 자체  
- **routing(라우팅)**: 경로를 정해 매칭하는 과정  
- **segment(세그먼트)**: 경로를 이루는 각 폴더 조각(예: `/blog/[slug]`의 `blog`, `[slug]`)

### App Router 파일 규칙(요지)
| 이름 | 의미 |
|---|---|
| `layout.(js/tsx)` | 세그먼트 공유 레이아웃(상태/DOM 유지) |
| `page.(js/tsx)` | 해당 경로의 페이지 UI |
| `loading.(js/tsx)` | Suspense 로딩 경계 UI |
| `error.(js/tsx)` | 세그먼트 범위 오류 UI(React 에러 경계) |
| `not-found.(js/tsx)` | 404 UI |
| `route.(js/ts)` | API 엔드포인트 |

### 라우팅 패턴
- **중첩 라우트**: `folder/folder` → 중첩 세그먼트
- **동적 라우트**:
  - `[id]` : 단일 파라미터
  - `[...all]` : 포괄(catch-all)
  - `[[...opt]]` : 선택적 포괄
- **Route Group**: `(group)` — URL에는 노출되지 않지만 구조/레이아웃 구분 가능
- **Private 폴더**: `_components` 처럼 언더스코어로 라우팅에서 제외(관용)

### 메타데이터/OGP 스니펫
```html
<meta property="og:type" content="website" />
<meta property="og:url" content="https://example.com/" />
<meta property="og:title" content="페이지 제목" />
<meta property="og:description" content="페이지 설명" />
<meta property="og:image" content="https://example.com/og.jpg" />
<meta property="og:site_name" content="사이트 이름" />
<meta property="og:locale" content="ko_KR" />
```

### 라우트별 로딩 UI
```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <div className="skeleton">로딩 중…</div>;
}
```
