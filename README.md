<h1>202130123 이민영</h1>
2주차 3주차 병결

<h1> **2025-09-24 / 5번째 수업**</h1>

> **주요 주제**: Next.js `searchParams`(동기/비동기), 동적 라우트 `[slug]` 비동기 `params`, `Link` 컴포넌트, 전역 네비게이션, 라우팅 방식 비교 및 네비게이션 동작 원리

---

## 개요
- `searchParams`: URL 쿼리 문자열을 읽는 표준 방식  
- `[slug]`: 동적 라우팅에서 매개변수 처리(비동기 타입 표기로 안전성 향상)  
- `Link`: 내부 이동 최적화(프리페치 + 클라이언트 전환)  
- 전역 메뉴: `app/layout.tsx`에서 모든 페이지 공통 네비게이션 구성  
- 라우팅 비교: React Router vs Next.js(파일 기반) / Pages Router vs App Router  
- 네비게이션 동작: Server Rendering, Prefetching, Streaming, Client-side transitions

---

## 1) `searchParams`란?
- URL의 **쿼리 문자열(Query String)** 을 읽는 방법  
  예: `/products?category=shoes&page=2` → `category=shoes`, `page=2`
- **App Router**에서 페이지 컴포넌트 props로 전달되어 사용

### 1-1. 동기 형태로 사용
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

### 1-2. 비동기 props(명시적) 대응
- 일부 버전/설정(예: 14.2+/15.x)에서는 `searchParams`가 **Promise** 로 전달될 수 있음
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

> **Tip**: 비동기 타입을 명시해두면 `await` 누락 같은 실수를 TypeScript가 잡아줍니다.

---

## 2) 동적 라우트 `[slug]` 심화 (성능/타이핑 메모)
- 데이터가 커지면 `Array.find`(O(n)) 대신 **DB 쿼리 + 인덱스**로 대체 권장
- `params` 가 동기처럼 보여도 **비동기일 수 있음** → 타입에서 **Promise** 명시로 가독성·안전성↑

### 예시(실습 보강)
```tsx
// app/blog/[slug]/page.tsx
import { notFound } from "next/navigation";
import { posts } from "../posts";

type Params = { slug: string };

export default async function PostPage({
  params,
}: { params: Promise<Params> }) {
  const { slug } = await params; // 비동기 해제
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

## 3) `Link` 컴포넌트 실습

### 3-1. 블로그 목록에 링크 추가
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

### 3-2. 쿼리 문자열을 객체 형태로 조합
```tsx
// 객체형 href 예시
<Link
  href={{
    pathname: "/products",
    query: { category: "shoes", page: 2 },
  }}
>
  신발 2페이지
</Link>
```

> **Tip**  
> - **내부 이동**은 `Link` 사용(클라이언트 전환 + 프리페치)  
> - 외부 링크는 `<a href="...">` 사용

---

## 4) 전역 메뉴(레이아웃) 추가
```tsx
// app/layout.tsx
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

---

## 5) 라우팅 방식 비교: React vs Next.js

### React(기본)
- 라우팅: 외부 라이브러리 필요(예: `react-router-dom`)
- 코드에서 `<Route>` 로 경로 매핑
```tsx
// React Router 예시
import { Routes, Route } from "react-router-dom";
export default function App() {
  return (
    <Routes>
      <Route path="/about" element={<About />} />
    </Routes>
  );
}
```

### Next.js
- 파일/폴더 기반 **자동 매핑(내장)**
- 파일 경로만 맞추면 자동으로 라우트 생성  
  - `pages/about.js` → `/about`  
  - `app/about/page.tsx` → `/about`

---

## 6) Next.js 라우팅: Pages Router vs App Router

### Pages Router
- 디렉토리: `pages/` (초기 버전~12)
- 특징: 단순·익숙, `getStaticProps`, `getServerSideProps` 사용
- 상태: 유지보수 중(신규 프로젝트에는 비권장)

**예시**
```
pages/
├─ index.tsx       → /
├─ about.tsx       → /about
└─ blog/[slug].tsx → /blog/:slug
```

### App Router
- 도입: Next 13+
- 디렉토리: `app/`
- 특징: **레이아웃 중첩**, **서버 컴포넌트**, **로딩 UI**, **병렬/인터셉트 라우트** 등 강력
- 권장: Next 14+ 기본 권장

**예시**
```
app/
├─ layout.tsx
├─ page.tsx          → /
└─ about/
   └─ page.tsx       → /about
```

---

## 7) Introduction: Next.js 네비게이션 개요
- 기본적으로 서버에서 렌더링(SSR/SSG)
- 빠른 경험을 위해 **prefetching**, **streaming**, **client-side transitions** 제공

---

## 8) How navigation works (작동 방식)

### 8-1. Server Rendering(서버 렌더링)
- 레이아웃/페이지는 기본 **서버 컴포넌트**
- 유형
  - **정적 렌더링**: 빌드/재검증 시점 생성, 캐시
  - **동적 렌더링**: 요청 시점 생성
- 단점: 새 경로 표시 전 서버 응답 대기

### 8-2. Next.js 보완 기법
- **Prefetching**: 방문 가능성이 높은 경로 데이터를 미리 가져오기
- **Client-side transitions**: 클라이언트 전환으로 체감 지연 감소
- **Streaming**: 서버에서 데이터를 준비되는 대로 점진적으로 전송

> **메모**: 최초 방문을 위해 HTML은 서버에서 생성되며, 이후 전환은 클라이언트에서 더 부드럽게 진행됩니다.

---

## 체크리스트
- [ ] `searchParams` 사용 시 버전에 따라 **Promise 여부** 확인  
- [ ] 동적 라우트의 `params`도 **비동기 타입**으로 명시해 `await` 누락 방지  
- [ ] 데이터 검색은 규모 커질수록 **DB 쿼리/인덱스**로 전환  
- [ ] 내부 이동은 반드시 **`Link`** 사용(프리페치 + 클라이언트 전환)  
- [ ] 전역 네비는 `app/layout.tsx`에서 구성해 일관된 UX 제공

---

## 참고 코드 모음

### `searchParams` 동기/비동기 타입 선언
```ts
type SearchParamsSync = { [key: string]: string | string[] | undefined };
type SearchParamsAsync = Promise<SearchParamsSync>;
```

### `[slug]` 페이지 스캐폴드
```tsx
// app/blog/[slug]/page.tsx
import { notFound } from "next/navigation";

type Params = { slug: string };

async function getPostBySlug(slug: string) {
  // 실제 서비스에서는 DB 쿼리(인덱스)로 대체 권장
  const all = await import("../posts").then(m => m.posts);
  return all.find(p => p.slug === slug) ?? null;
}

export default async function PostPage({ params }: { params: Promise<Params> }) {
  const { slug } = await params;
  const post = await getPostBySlug(slug);
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

## 요약
- `searchParams`는 URL 쿼리 읽기 표준이며, 버전에 따라 **동기/비동기**로 전달 가능  
- `[slug]` 라우트에서 `params`를 **Promise** 로 다루면 타입 안정성↑  
- `Link`는 내부 네비게이션 최적화의 중심(프리페치/클라이언트 전환)  
- **App Router**는 레이아웃/서버 컴포넌트/스트리밍 등 최신 기능 제공  
- 성능은 **DB 인덱스/쿼리**로 해결하고, 타입은 **명시적으로 비동기**를 반영

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
