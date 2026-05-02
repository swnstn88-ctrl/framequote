# FrameQuote — 작업 인수인계 문서

> 이 문서는 새 Claude Code 세션이 저장소만 열어도 즉시 작업을 이어갈 수 있도록 만든 컨텍스트 파일입니다.
> 마지막 업데이트: 2026-05-02 / 작성: Claude (이전 세션 인수)

## 1. 프로젝트 개요

- **이름**: FrameQuote — Global Production Estimator
- **소유자**: swnstn88-ctrl (STONFILM)
- **저장소**: `swnstn88-ctrl/framequote`
- **배포**: GitHub Pages → `frame-quote.com` (`CNAME` 파일로 도메인 설정됨)
- **성격**: 영상 제작 견적 산출 웹툴. 단일 HTML 파일로 동작하는 정적 사이트.

## 2. 파일 구조

```
framequote/
├── CNAME        # frame-quote.com (커스텀 도메인)
├── index.html   # 전체 앱 (HTML + 인라인 CSS + 인라인 JS, 약 145줄)
└── CLAUDE.md    # 이 문서
```

## 3. 기술 스택

- 빌드 도구 없음. 순수 HTML/CSS/JS.
- 다국어 i18n: `ko`, `en`, `ja` (인라인 `I18N` 객체)
- 통화 환율: `exchangerate.host` API (실패 시 캐시/하드코딩 폴백)
- 인증/DB: Supabase JS (CDN 로드, 이메일 OTP 6자리)
- 결제: Stripe Checkout (현재는 스텁/미연동)

## 4. 현재 상태 — 알려진 미완성 항목

### 🔴 시크릿 플레이스홀더 (배포 전 반드시 교체)
`index.html` 20–21줄:
```js
const SUPABASE_URL = 'https://YOUR-PROJECT.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR-ANON-PUBLIC-KEY';
```
→ 실제 Supabase 프로젝트의 값으로 교체 필요.
→ anon key는 공개 가능하지만 RLS(Row Level Security) 정책이 켜져 있어야 함.

### 🟡 결제(Stripe) 미연동
`index.html` 131–137줄: `subscribeBtn` 클릭 시 토스트만 띄우고 끝.
→ 실제 Stripe Checkout URL을 넣어야 구독 동작.

### 🟡 로그인 모달의 비밀번호 로그인 미연동
`index.html` 139–140줄 주석 참고. 현재 `btnLogin` → 가짜 `afterLogin()` 호출만 함.
→ `supabase.auth.signInWithPassword({email, password})` 연결 필요.

### 🟡 Supabase `profiles` 테이블 의존
다음 컬럼이 있어야 함: `user_id` (PK, auth.users 참조), `email`, `role`, `phone`, `pro`.
→ Supabase 콘솔에서 테이블 생성 + RLS 정책 설정 필요.

## 5. 주요 기능 영역 (코드 위치)

| 기능 | 위치 |
|------|------|
| i18n 사전 | `index.html` 1줄 `I18N` 객체 (한 줄에 압축됨) |
| 프리셋 (shorts/MV/CF/롱폼 등) | `presetFill()` 함수 |
| 환율 API | `refreshRates()` 함수 |
| 견적 계산 (소계 → 복잡도 → 마진 → VAT) | `calc()` 함수 |
| PDF 미리보기 영역 | `#printArea` (인쇄용 라이트 테마) |
| Supabase 인증 (OTP 발송/검증) | 18–144줄 별도 `<script>` 블록 |

## 6. 브랜치 정책

- 메인 브랜치: `main`
- **현재 작업 브랜치**: `claude/continue-previous-session-IwKQr`
  - 이 브랜치는 이전 PC Claude Code 세션(`da9b6316-...`)에서 만들어졌으나
    **실제 커밋된 변경은 없음** (`main`과 동일한 HEAD).
- 모든 새 작업은 이 브랜치에 커밋 후 push.
- push 후에는 PR을 draft로 생성.

## 7. 로컬에서 미리보기

빌드 없음. 그냥 브라우저로 `index.html`을 열거나:
```bash
python3 -m http.server 8000
# → http://localhost:8000
```

## 8. 새 세션이 가장 먼저 할 일

1. `git status` 와 `git log --oneline -5` 확인
2. 사용자에게 "어떤 항목을 진행할지" 묻기 — 위 §4의 미완성 항목 중 선택
3. 작업 시작

## 9. 컨텍스트 제약 (중요)

- 본 환경은 격리된 클라우드 컨테이너. PC 로컬의 `~/.claude/projects/` 세션 파일에 접근 불가.
- 따라서 세션 ID로 이전 대화를 직접 복원할 수 없음.
- 인수인계는 **이 문서 + git 히스토리**로만 이루어짐.
- GitHub MCP 도구는 `swnstn88-ctrl/framequote` 저장소로만 제한됨.
