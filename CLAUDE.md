# biz-tracker 프로젝트 가이드

## 프로젝트 개요

- **앱명**: 🏢 사업자 휴·폐업 조회
- **파일**: `index.html` (단일 파일 — 모든 CSS·HTML·JS 포함, 약 1000줄), `version-history.json`
- **현재 버전**: v2.0.0
- **저장소**: `junpal5/biz-tracker` (GitHub Pages로 배포)
- **배포 URL**: `https://junpal5.github.io/biz-tracker/`
- **사용자**: 비개발자 — 기술 용어 없이 한국어로 안내할 것

---

## 앱 기능 구조

| 단계 | 설명 |
|------|------|
| Step 1 | 공공데이터포털 API 인증키 입력 (password 타입, 보기/숨기기 토글) |
| Step 2 | XLSX 파일 업로드 → SheetJS로 파싱 → 사업자등록번호 열 선택 드롭다운 (A~G열, 최대 7개만 표시) |
| Step 2 (파일 삭제) | 업로드된 파일명 pill 우측 ✕ 버튼으로 파일·열 선택 상태 전체 초기화 |
| Step 2 (헤더 감지) | `XLSX.utils.sheet_to_json(ws, { header: 1 })`으로 raw 배열 읽기 → 상단 빈 행 건너뛰고 비어있지 않은 셀이 2개 이상인 첫 행을 헤더로 자동 인식. 빈 헤더 셀은 `열N`으로 대체 |
| Step 2-1 | 열 선택 시 하이픈 제거 후 숫자 이외 문자가 있는 행 번호를 자동 감지해 경고 표시 |
| Step 3 | 조회 버튼 클릭 → 첫 번째 데이터 1건으로 API KEY·시스템 사전 검증 → 실패 시 중단 및 오류 표시 |
| Step 3-1 | 검증 통과 시 100건 단위 배치 API 호출 → 프로그레스바 → 결과 테이블 표시 |
| 결과 | 원본 데이터에 `사업자상태` / `상태코드` / `세금종류` 열 추가 후 XLSX 다운로드 |

---

## 사용 API

- **엔드포인트**: `https://api.odcloud.kr/api/nts-businessman/v1/status`
- **메서드**: POST
- **인증**: `serviceKey` 쿼리 파라미터 (공공데이터포털 일반 인증키 Decoding 값)
- **요청 body**: `{ "b_no": ["0000000000", "1111111111", ...] }` ← 문자열 배열 (객체 배열 아님)
- **주의**: `/v1/validate`(진위확인) API는 `businesses` 객체 배열 형식이 다름 — 혼동 금지
- **배치 한도**: 1회 최대 100건
- **상태 코드**: `01` 계속사업자 / `02` 휴업자 / `03` 폐업자

### CORS 주의사항
브라우저에서 `api.odcloud.kr`을 직접 호출하면 CORS 오류가 발생할 수 있다.
오류 발생 시 앱 내에 "Netlify Functions 프록시 필요" 안내 메시지가 표시된다.
프록시가 필요한 경우 Netlify Functions 추가를 별도로 검토한다.

---

## 디자인 시스템 (MiniMax 기반)

v2.0.0부터 MiniMax 디자인 시스템을 적용한다. 새 UI 요소 추가 시 아래 토큰을 준수할 것.

### 폰트
- **기본 폰트**: `DM Sans` (Google Fonts 로드) → fallback: `Inter`, `-apple-system`
- 단일 폰트 전략 — 두 번째 폰트 패밀리 혼용 금지

### 컬러 토큰 (CSS 변수)

| 변수 | 용도 |
|------|------|
| `--color-primary` (#0A0A0A) | 버튼·헤더·step badge 배경 |
| `--color-on-primary` (#FFFFFF) | primary 위 텍스트 |
| `--color-canvas` (#FFFFFF) | 카드·입력 필드 배경 |
| `--color-surface` (#F5F5F5) | 카드 헤더·호버 배경 |
| `--color-surface-soft` (#FAFAFA) | 페이지 배경 |
| `--color-hairline` (#E5E5E5) | 카드·입력 테두리 |
| `--color-hairline-soft` (#F0F0F0) | 테이블 행 구분선 |
| `--color-ink` (#1A1A1A) | 주요 텍스트 |
| `--color-charcoal` (#404040) | 본문 텍스트 |
| `--color-steel` (#888888) | 보조 텍스트·테이블 헤더 |
| `--color-stone` (#A0A0A0) | 비활성 텍스트 |
| `--color-brand-coral` (#FF4B2B) | 강조 액센트 (프로그레스 바, NEW 뱃지, 버전 칩 dot) |
| `--color-brand-blue` (#2563EB) | 포커스 링·링크 |

### 컴포넌트 규칙
- **버튼**: 반드시 `border-radius: var(--rounded-full)` (pill shape) — 직각 버튼 금지
- **카드**: `border-radius: var(--rounded-xl)` (16px) + `border: 1px solid var(--color-hairline)`
- **뱃지/pill**: `border-radius: var(--rounded-full)`
- **그림자**: 카드는 `rgba(0,0,0,0.04) 0px 1px 2px 0px` (flat 기본), 모달은 `rgba(36,36,36,0.12) 0px 12px 24px -4px`
- **브랜드 코랄 사용 범위**: 프로그레스 바, NEW 뱃지, 버전 칩 dot에만 한정 — 일반 버튼·배경에 사용 금지

---

## 작업 워크플로우 (모든 작업 요청 시 반드시 준수)

### 1단계 — 계획 안내 (작업 시작 전 필수)
작업을 세부 단계로 나누어 **먼저 사용자에게 안내**한다. 작업을 시작하기 전에 항상 이 단계를 수행한다.

### 2단계 — 단계별 작업 수행
계획한 순서대로 작업을 진행한다.
`index.html`은 단일 파일이므로 편집 시 Read 도구로 전체를 읽지 말고, grep/offset으로 필요한 부분만 읽는다.

### 3단계 — 변경 내용 요약 (한국어)
작업 완료 후 변경된 내용을 **한국어**로 간결하게 요약한다.
`version-history.json`의 `changes` 배열에 들어갈 항목 형태로 작성한다.

### 4단계 — 버전 선택지 제공
요약 후 아래 선택지를 사용자에게 제시한다. 현재 버전은 `version-history.json`의 `currentVersion`을 참조한다.

| 선택 | 버전 변화 | 적합한 경우 |
|------|-----------|-------------|
| 패치 | x.x.**+1** | 오탈자 수정, 사소한 버그 수정 |
| 마이너 | x.**+1**.0 | 새 기능 추가, UI 개선, 기존 기능 변경 |
| 메이저 | **+1**.0.0 | 전체 구조 변경, 대규모 리디자인 |
| 버전 유지 | 변경 없음 | 임시 수정 또는 테스트 |

### 5단계 — 자동 Push
사용자가 버전을 선택하면:
1. `version-history.json` 업데이트 (`currentVersion` 갱신 + `history` 배열 **맨 앞**에 새 항목 추가)
2. 날짜는 아래 명령으로 실제 수정 시각을 사용한다 (`T00:00:00.000Z` 고정 금지)
   ```bash
   date -u +"%Y-%m-%dT%H:%M:%S.000Z"
   ```
3. 변경된 파일 전체 commit 후 `main` 브랜치에 직접 push (PR 없이 바로 반영)

```bash
git add index.html version-history.json
git commit -m "biz-tracker: <작업 요약>"
git push origin main
```

> Push 실패 시 (충돌): `git pull origin main --rebase` 후 재시도한다.
> Push 실패 시 (403 인증 오류): 아래 Push 인증 설정 섹션 참고.

---

## 파일별 주의사항

### index.html
- 약 1000줄의 단일 파일. CSS·JS 모두 인라인 포함.
- 편집 시 Read 도구로 전체 파일을 읽지 말고, grep/offset으로 필요한 부분만 읽는다.
- **버전 칩**: `<button class="ver-chip">` — 화면 좌측 하단 fixed 위치에 항상 표시. 클릭 시 `.ver-modal-overlay`를 `.open` 클래스로 활성화해 모달 표시.
- **버전 모달**: JS 내 `VERSION_HISTORY` 배열을 `renderVerModal()`이 동적으로 렌더링. 버전 추가 시 해당 배열 맨 앞에 항목을 추가하면 된다.

### version-history.json
- `currentVersion`: 현재 버전 문자열
- `history`: 최신 버전이 배열 **맨 앞**에 위치
- 날짜 형식: ISO 8601 UTC — 반드시 실제 수정 시각을 사용할 것

---

## Push 인증 설정

### 세션 시작 시 필수 절차

Claude Code는 세션이 초기화될 때마다 인증 정보가 사라진다.
따라서 **새 세션에서 biz-tracker 작업 시작 시, 사용자가 아래 형식으로 PAT를 전달해야 한다:**

> "biz-tracker 작업할게. PAT: `ghp_xxxx`"

PAT를 받으면 즉시 아래 명령으로 git remote에 설정한 뒤 작업을 시작한다:

```bash
git remote set-url origin https://junpal5:<PAT>@github.com/junpal5/biz-tracker.git
```

### PAT 발급 및 관리 안내 (사용자용)

- **발급 위치**: GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
- **필요 권한**: `repo` 전체 선택
- **만료 기간**: `No expiration` (무제한)으로 설정하면 자주 갱신하지 않아도 됨
- **보안 주의**: PAT는 채팅창에 입력 후 별도 보관하지 말 것. GitHub에서 언제든 삭제/재발급 가능

---

## 금지 사항

- PR(Pull Request) 생성 금지 — main에 직접 push한다.
- 불필요한 브랜치 생성 금지.
- 사용자에게 git 명령어를 직접 실행하도록 요청하지 말 것 (Claude가 대신 실행).
- 영어 기술 용어를 설명 없이 사용하지 말 것.
