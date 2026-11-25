# Development Guidelines

## 프로젝트 개요

- 프로젝트 이름: **Notion CMS Blog**. 기술 스택: Next.js 15, TypeScript, Tailwind CSS, shadcn/ui.
- Notion DB(`Projects`)를 단일 원본으로 쓰는 포트폴리오 사이트. npm workspaces 모노레포.
- `apps/web`: Next.js 15 App Router, TypeScript, Tailwind v4, shadcn/ui(Base UI), TanStack Query, Zustand (:3000)
- `apps/api`: Express, TypeScript(ESM), zod, helmet, cors, morgan (:4000)
- 데이터 흐름: `Notion → apps/api(캐시 60초) → apps/web → 방문자`. **Notion 토큰은 apps/api에만 둔다.**
- 기준 문서: `docs/PRD.md`(요구사항), `docs/ROADMAP.md`(단계 순서). 작업 전 해당 단계의 완료 기준을 확인한다.
- 현재 코드의 `todos` 라우트/페이지, `counter` 스토어/컴포넌트, `about` 페이지는 스타터킷 예제다. 새 기능의 참고 패턴으로만 쓴다.

## 디렉터리와 수정 위치

| 작업 | 위치 |
|---|---|
| API 환경 변수 추가 | `apps/api/src/config/env.ts`(zod 스키마) + `apps/api/.env.example` |
| API 라우터 추가 | `apps/api/src/routes/<이름>.ts` + `apps/api/src/routes/index.ts`에 `apiRouter.use` 등록 |
| 오류 처리 | `apps/api/src/middlewares/error-handler.ts` (이미 ZodError/HttpError 처리) |
| Notion 클라이언트/매퍼/캐시 | `apps/api/src/` 하위 신규 디렉터리 (예: `services/`, `utils/cache.ts`) |
| 웹 페이지 | `apps/web/src/app/<경로>/page.tsx` |
| 웹 공통 컴포넌트 | `apps/web/src/components/` (레이아웃은 `components/layout/`) |
| 클라이언트 상태 | `apps/web/src/stores/<이름>.ts` (Zustand) |
| 메뉴 | `apps/web/src/components/layout/header.tsx`의 `links` 배열 |
| shadcn 컴포넌트 | `components/ui/` — `cd apps/web && npx shadcn@latest add <이름>`로만 추가 |

## 함께 수정해야 하는 파일

- 환경 변수 추가 시: `env.ts` + `.env.example` 동시 수정. 실제 값은 쓰지 않는다.
- API 라우터 추가 시: 라우터 파일 + `routes/index.ts` 등록.
- 웹 메뉴 추가 시: `header.tsx`의 `links` 한 곳만 수정(데스크톱/모바일 공용).
- API 응답 형태나 `Project` 타입 변경 시: `apps/api` 타입과 `apps/web` 타입을 같이 수정하고 `docs/PRD.md` 6.4와 일치시킨다.
- 단계 작업 완료 시: `docs/ROADMAP.md`의 해당 체크박스를 `[x]`로 갱신한다.
- 프로젝트 규칙 변경 시: `CLAUDE.md`와 이 파일을 함께 갱신한다.

## 코딩 스타일

- TypeScript **strict mode**(`apps/web`, `apps/api` tsconfig 모두 `"strict": true`)를 끄거나 완화하지 않는다.
- ❌ `any`, `@ts-ignore`, `@ts-expect-error`(사유 없이), 불필요한 `as` 단언. 외부 데이터(Notion 응답, `req.*`)는 zod 또는 타입 가드로 좁힌다.
- ESLint 규칙을 준수한다. 웹은 `next/core-web-vitals` + `next/typescript` 설정(`apps/web/eslint.config.mjs`). ❌ `eslint-disable`로 우회, 규칙 완화.
- `apps/api`의 `lint` 스크립트는 `tsc --noEmit`이다. 타입 오류가 곧 린트 오류다.
- 사용하지 않는 import/변수를 남기지 않는다.
- 스타일링은 Tailwind 유틸리티 클래스와 shadcn/ui 컴포넌트만 사용한다. ❌ 별도 CSS 파일/인라인 style 추가, 다른 UI 라이브러리 도입.
- 코드 수정 후 `npm run typecheck`와 `npm run lint`를 통과시킨다(커밋 시 훅이 강제).

## API 규칙 (apps/api)

- 상대 import에 **반드시 `.js` 확장자**를 붙인다. ✅ `from "../utils/http-error.js"` ❌ `from "../utils/http-error"`
- 성공: `res.json({ data })`, 실패: `{ error: { message, details? } }`. 직접 오류 JSON을 만들지 말고 `HttpError`를 던진다.
- 비동기 핸들러는 `asyncHandler`로 감싼다. 오류는 `throw new HttpError(status, "한국어 메시지")`.
- 요청 입력(`req.body`, `req.query`, `req.params`)은 zod `.parse()`로 검증한다.
- `process.env`를 `env.ts` 밖에서 직접 읽지 않는다. `env`를 import한다.
- 오류 메시지는 한국어.
- Notion 규칙: `공개` 체크 필터는 **서버 쿼리 조건으로 강제**한다. 슬러그 누락 행은 매퍼에서 제외한다. 비공개/없는 슬러그는 `HttpError(404)`.
- Notion 호출은 반드시 메모리 캐시(TTL 60초)를 거친다. Notion 장애 시 마지막 캐시를 반환하고, 캐시도 없으면 오류를 던진다.
- 응답과 로그에 `NOTION_TOKEN`을 포함하지 않는다.
- 예정된 엔드포인트: `GET /api/projects`(쿼리 `tag`, `featured`), `GET /api/projects/:slug`, `GET /api/tags`.

## 웹 규칙 (apps/web)

- **모든 UI 문구는 한국어.**
- shadcn/ui는 Base UI 기반: ❌ `asChild` 사용 금지. ✅ 트리거는 `render` prop, 링크 버튼은 `className={buttonVariants({ ... })}`를 적용한 `<Link>`. (`header.tsx` 참고)
- 서버 데이터는 TanStack Query(`useQuery`/`useMutation`), 클라이언트 상태는 Zustand. 서버 데이터를 Zustand에 복사하지 않는다.
- API 호출은 `@/lib/api`의 `api<T>()`만 사용한다. ❌ 컴포넌트에서 `fetch` 직접 호출. 응답은 `{ data: T }` 형태로 받는다.
- import 경로는 `@/` 별칭을 사용한다.
- `useQuery`, `useState`, 이벤트 핸들러를 쓰는 파일은 첫 줄에 `"use client";`. 페이지는 서버 컴포넌트로 두고 상호작용 부분만 별도 파일(예: `todos/todo-list.tsx`)로 분리한다.
- 페이지에는 `export const metadata: Metadata = { title: "..." }`를 둔다(루트 템플릿 `%s | 스타터킷`).
- 데이터 화면에는 로딩(스켈레톤), 오류, 빈 상태를 모두 구현한다. 빈 상태 문구: "프로젝트가 없습니다".
- 카드 그리드는 모바일 우선 1/2/3열.
- 이미지는 `alt`를 반드시 넣는다.
- 예정 경로: `/`, `/projects`, `/projects/[slug]`.

## 프로젝트 전용 워크플로

1. 해당 `docs/ROADMAP.md` 단계의 "할 일"과 "완료 기준"을 읽는다.
2. 하위 계층부터 구현한다: 공통 모듈 → API → 웹. API 응답 형식이 확정되기 전에 웹 화면을 만들지 않는다.
3. 변경 후 `npm run typecheck`, `npm run lint`, `npm run build`를 실행한다 (또는 `/check`).
4. 새 페이지/라우트는 `/add-page <경로>`, `/add-route <이름>` 명령의 절차를 따른다.

## AI 결정 기준

- 본문 렌더링 방식(`react-notion-x` vs 직접 구현)은 미결이다. 3단계 상세 페이지 착수 시 결정하고, 새 의존성 추가 전 사용자에게 확인한다.
- 호스팅 방식은 5단계에서 결정한다. 그 전에 배포 설정 파일을 만들지 않는다.
- PRD와 코드가 다르면 PRD를 우선하되, 충돌 내용을 사용자에게 보고한다.
- 지원 블록은 문단, 제목, 목록, 이미지, 코드, 인용, 구분선뿐이다. 그 외 블록은 건너뛴다.
- P1(정렬, 검색, 다크 모드 등) 기능은 MVP(P0) 완료 전에 구현하지 않는다. 다크 모드는 `next-themes`가 이미 `providers.tsx`에 설정되어 있다.

## 금지 사항

- ❌ `node.exe`/node 프로세스 전체 종료(`taskkill /IM node.exe`, `pkill node`, `Stop-Process -Name node`). 포트를 점유한 PID만 종료한다. (훅이 차단함)
- ❌ `.env`, `.env.local` 수정·커밋. 변수 목록은 `.env.example`에만 추가한다. (훅이 차단함)
- ❌ `package-lock.json` 직접 수정. `npm install`로만 갱신한다. (훅이 차단함)
- ❌ `git push --force`(필요 시 `--force-with-lease`), `sudo`, 루트/홈에 대한 `rm -rf`.
- ❌ 토큰, 개인 정보, `.shrimp_data/` 커밋. 단, `git commit` 전에는 훅이 typecheck + lint를 실행하며 실패 시 커밋이 차단된다.
- ❌ 브라우저 번들(`NEXT_PUBLIC_*` 포함)에 Notion 토큰/DB ID 노출.
- ❌ 비공개 프로젝트가 목록, 상세, 태그 어디에도 나오게 하는 코드.
- ❌ `components/ui/` 파일을 CLI 없이 새로 작성하거나 `asChild` 패턴의 shadcn 코드를 그대로 붙여넣기.
- ❌ DB 연결 추가(저장소는 Notion 또는 인메모리).
