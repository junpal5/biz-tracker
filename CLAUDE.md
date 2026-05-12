# biz-tracker 프로젝트 가이드

## 개요
- **앱**: 사업자 휴·폐업 조회 (단일 파일 `index.html`, 약 1000줄)
- **현재 버전**: `version-history.json`의 `currentVersion` 참조 (현재 v2.0.1)
- **배포**: GitHub Pages — `https://junpal5.github.io/biz-tracker/`
- **대상 사용자**: 비개발자 — 한국어로 안내

---

## 매 작업 시 필수 워크플로우 (순서 엄수)

```
1. 계획 안내  →  2. 작업 수행  →  3. 버전 업데이트 제안  →  4. Push
```

### 3단계: 버전 업데이트 제안 (절대 생략 금지)
작업 완료 후 **반드시** 아래 선택지를 사용자에게 제시한다.

| 선택 | 버전 변화 | 적합한 경우 |
|------|-----------|-------------|
| 패치 | x.x.**+1** | 오탈자, 사소한 버그 수정 |
| 마이너 | x.**+1**.0 | 새 기능 추가, UI 개선 |
| 메이저 | **+1**.0.0 | 전체 구조 변경, 대규모 리디자인 |
| 버전 유지 | 변경 없음 | 임시 수정, 테스트 |

### 4단계: Push
사용자가 버전을 선택하면 즉시 실행:
1. `version-history.json` 업데이트 — `currentVersion` 갱신 + `history` 배열 **맨 앞**에 추가
2. 날짜: `date -u +"%Y-%m-%dT%H:%M:%S.000Z"` 실행값 사용 (`T00:00:00.000Z` 고정 금지)
3. **main에 직접 push** (브랜치·PR 생성 금지)

```bash
git add index.html version-history.json
git commit --no-gpg-sign -m "biz-tracker: <작업 요약>"
git push origin main
# 실패 시: git pull origin main --rebase 후 재시도
```

---

## 파일 구조 & 주의사항

### index.html
- CSS·JS 모두 인라인. 편집 시 전체 Read 금지 — offset/grep으로 필요한 부분만 읽는다.
- **버전 칩**: `<button class="ver-chip">` — 좌측 하단 fixed. 클릭 시 `.ver-modal-overlay.open`으로 모달 표시.
- **버전 모달**: JS 내 `VERSION_HISTORY` 배열을 수정. 버전 추가 시 배열 맨 앞에 항목 추가.

### version-history.json
- `currentVersion`: 현재 버전 문자열
- `history`: 최신 버전이 배열 **맨 앞**

---

## 앱 기능 요약

| 단계 | 설명 |
|------|------|
| Step 1 | API 인증키 입력 (password 타입, 보기/숨기기 토글) |
| Step 2 | XLSX 업로드 → 헤더 자동 감지 (빈 행 건너뛰기) → 열 선택 드롭다운 (A~G, 최대 7개) |
| Step 2 | 파일명 pill 우측 ✕ 버튼으로 업로드 취소 |
| Step 2 | 열 선택 시 비숫자 문자 포함 행 자동 감지·경고 |
| Step 3 | 1건 사전 검증 → 100건 배치 API 호출 → 프로그레스바 → 결과 테이블 |
| 결과 | 원본 데이터 + `사업자상태`/`상태코드`/`세금종류` 열 → XLSX 다운로드 |

### 헤더 자동 감지 로직
`XLSX.utils.sheet_to_json(ws, { header: 1 })`으로 raw 배열 읽기 →
비어있지 않은 셀이 2개 이상인 첫 행을 헤더로 사용. 빈 셀은 `열N`으로 대체.

---

## API

- **URL**: `POST https://api.odcloud.kr/api/nts-businessman/v1/status?serviceKey=...`
- **Body**: `{ "b_no": ["0000000000", ...] }` ← 문자열 배열 (객체 배열 아님)
- **배치**: 1회 최대 100건 / **상태코드**: `01` 계속사업자 · `02` 휴업자 · `03` 폐업자
- **CORS**: 브라우저 직접 호출 시 오류 가능 → Netlify Functions 프록시 검토

---

## 디자인 시스템 (MiniMax)

- **폰트**: DM Sans (Google Fonts) — 단일 폰트 전략, 혼용 금지
- **버튼**: 반드시 `border-radius: var(--rounded-full)` (pill)
- **카드**: `border-radius: var(--rounded-xl)` + `border: 1px solid var(--color-hairline)`
- **코랄 (`--color-brand-coral`, #FF4B2B)**: 프로그레스 바·NEW 뱃지·버전 칩 dot에만 사용

주요 컬러 토큰: `--color-primary` #0A0A0A · `--color-canvas` #FFFFFF · `--color-surface` #F5F5F5 · `--color-hairline` #E5E5E5 · `--color-ink` #1A1A1A · `--color-charcoal` #404040 · `--color-steel` #888888 · `--color-brand-blue` #2563EB

---

## Push 인증 (새 세션 시작 시)

새 세션에서 작업 시작 전 PAT를 전달받아 즉시 설정:
```bash
cd /tmp && git clone https://<PAT>@github.com/junpal5/biz-tracker.git biz-tracker
# 또는 기존 클론이 있으면:
git remote set-url origin https://junpal5:<PAT>@github.com/junpal5/biz-tracker.git
```
- PAT 발급: GitHub → Settings → Developer settings → Personal access tokens (classic), `repo` 권한

---

## 금지 사항
- 브랜치 생성 · PR 생성 금지 — **main 직접 push**
- 버전 업데이트 제안 생략 금지
- 사용자에게 git 명령어 직접 실행 요청 금지
