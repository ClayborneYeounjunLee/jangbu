# 장부 · Jangbu 📒

> 하루의 수입과 지출을 달력 한 장에 담는, 가볍고 예쁜 가계부 웹앱.
> 캘린더에 일별 순액을 색으로 물들이고, 월급·적금·구독 같은 **고정 반복 항목**을 자동 전개하며, 다중 통화(₩/$/A$/₱)를 실시간 환율로 원화 환산해 월·분기·연간으로 분석합니다.

🔗 **라이브:** https://clayborneyeounjunlee.github.io/jangbu/

`moa` 컬렉션의 형제 앱으로, 단일 HTML 파일(`index.html` 하나)로 완결되는 순수 프론트엔드 앱입니다. 빌드 도구·번들러·백엔드 서버 없이 브라우저만으로 동작하며, 로그인은 선택입니다(게스트 모드 지원).

---

## ✨ 주요 기능

- **📅 달력 중심 가계부** — 월 캘린더의 각 날짜 칸에 그날의 **순액(수입−지출)**을 요약 표시. 순액이 플러스면 초록, 마이너스면 빨강으로 셀 배경을 물들이고(그 달 최대 절대값 기준으로 진하기 조절), 큰 금액은 `1.2만`·`3.4억`(영어는 `K`/`M`)처럼 축약합니다.
- **날짜 상세 시트** — 날짜를 누르면 하단 시트가 올라오며 그날의 모든 항목(일반 거래 + 고정 발생)과 수입/지출/순액 합계를 보여주고, 바로 추가·수정·삭제할 수 있습니다.
- **➕ 빠른 기록 (FAB)** — 달력 화면 우하단의 플로팅 버튼으로 **오늘 날짜** 거래를 즉시 추가.
- **🔁 고정 반복 항목** — 월급·적금·구독처럼 정기적으로 발생하는 수입/지출을 규칙으로 등록. **매월(며칠)/매주(요일)/매년(몇 월 며칠)** 주기를 지원하고, 시작일·종료일로 유효 기간을 제한할 수 있습니다. 등록하면 실제 데이터를 만들지 않고도 캘린더와 분석에 자동 전개됩니다(달력 셀에 🔁 표시).
- **📊 월/분기/연 분석** — 기간별 수입·지출·순액 요약, 순액 **추이 막대그래프**(월간=일별, 분기=월별, 연간=12개월), **카테고리별 지출/수입 비중 바** 표시.
- **💱 다중 통화 + 실시간 환율** — 각 항목을 KRW·USD·AUD·PHP로 입력. 외화는 실시간 환율로 원화 환산되어 합계·분석에 반영되고, 입력 중에도 “≈ ₩…” 미리보기를 보여줍니다.
- **☁️ 클라우드 저장 또는 이 기기에서만** — Google 로그인 시 Firestore에 저장해 여러 기기에서 이어 쓰고, 로그인 없이 **게스트 모드**로 이 기기(localStorage)에만 저장할 수도 있습니다.
- **🌗 다크/라이트 테마 · 🌐 한국어/영어** — 토글로 즉시 전환, 시스템 설정 자동 감지, 선택값은 로컬 저장.
- **⬇ 백업 내보내기** — 전체 상태를 `jangbu-backup-YYYY-MM-DD.json` 파일로 다운로드.
- **📱 PWA 감성** — 홈 화면 추가용 메타(`apple-mobile-web-app-*`), 세이프 에어리어 대응, 모바일 우선 레이아웃(최대 폭 520px).

---

## 🧱 기술 스택 / 언어

| 항목 | 내용 |
|---|---|
| 언어 | **순수 HTML + CSS + JavaScript(ES 모듈)** — 프레임워크 없음 |
| 파일 구성 | **단일 파일 `index.html`** (약 1,195줄, 마크업·스타일·스크립트 인라인) |
| 빌드 도구 | **없음** — 번들러·트랜스파일러·`package.json` 모두 없음. 브라우저가 파일을 그대로 실행 |
| 스크립트 방식 | `<script type="module">` — Firebase SDK를 **동적 `import()`** 로 필요 시 로드 |
| 스타일 | 인라인 `<style>` + **CSS 커스텀 프로퍼티(변수)** 로 라이트/다크 테마 구성, `color-mix()`·`env(safe-area-inset-*)` 사용 |
| 폰트 | 시스템 폰트 스택 — `Pretendard`, `Apple SD Gothic Neo`, `Malgun Gothic`, `Noto Sans KR`, `sans-serif` (웹폰트 CDN 미포함, 로컬 폰트 우선) |
| 아이콘 | 이모지 + 인라인 SVG 파비콘(data URI) |
| 백엔드 | 없음(정적). 데이터는 Firestore 또는 localStorage |

### CDN 라이브러리 / SDK

| 라이브러리 | 버전 | 로드 방식 | 용도 |
|---|---|---|---|
| Firebase App | 12.14.0 | `https://www.gstatic.com/firebasejs/12.14.0/firebase-app.js` (동적 import) | 앱 초기화 |
| Firebase Auth | 12.14.0 | `firebase-auth.js` | Google 로그인(팝업/리다이렉트) |
| Firebase Firestore | 12.14.0 | `firebase-firestore.js` | 클라우드 저장 + 오프라인 영속 캐시 |

> Firebase 버전은 코드 상수 `FB_VER = "12.14.0"` 로 관리되어 세 모듈 모두 같은 버전으로 로드됩니다.

---

## 🏗️ 시스템 구조

### 단일 파일 · 화면 전환 방식

라우터 없이 **DOM 표시/숨김(`.hidden`)** 으로 화면을 전환하는 SPA입니다. 최상위에 세 개의 “스크린”이 있습니다.

```
screen-loading  (스피너)  →  screen-auth  (로그인/게스트)  →  #app  (본 화면)
```

`#app` 안에는 하단 탭바로 오가는 4개의 탭 화면(`.tabpane`)이 있습니다.

| 탭 | 화면 id | 렌더 함수 | 설명 |
|---|---|---|---|
| 📅 달력 | `screen-calendar` | `renderCalendar()` | 월 캘린더 + 하단 월 합계, FAB 표시 |
| 🔁 고정 | `screen-recurring` | `renderRecurring()` | 이번 달 예정 금액 + 고정 수입/지출 목록 |
| 📊 분석 | `screen-analysis` | `renderAnalysis()` | 월/분기/연 세그먼트 + 추이 + 카테고리 바 |
| ⚙️ 설정 | `screen-settings` | `renderProfile()` / `renderRates()` | 프로필·테마·언어·환율·백업·계정 |

### 부팅 흐름 (파일 맨 아래 IIFE)

1. `showLoading()` — 로딩 화면 표시.
2. `loadRatesCache()` — localStorage 환율 캐시 로드, `applyLang(L)` — 언어 적용.
3. `fetchRates()` — 백그라운드로 최신 환율 시도(성공 시 화면 재렌더).
4. `initFirebase()` — Firebase 3개 모듈 동적 import 및 초기화(실패해도 앱은 계속).
5. Firebase 성공 시 `onAuthStateChanged` 구독:
   - 로그인된 사용자 → `startCloud(user)` (Firestore에서 문서 로드)
   - 아니면 이전 모드가 `guest`면 `startGuest()`, 아니면 `showAuth()`
6. Firebase 실패 시(오프라인 등) 게스트/인증 화면으로 폴백.

### 상태 관리

- 전역 `let state` 하나에 모든 데이터가 담깁니다(외부 상태 라이브러리 없음).
- 렌더 함수가 `state`를 읽어 매번 `innerHTML`을 생성하는 **명령형 렌더링**.
- 저장은 **디바운스**: `save()` 호출 → 게스트는 즉시 localStorage, 클라우드는 2초 후 `flushSave()`로 Firestore `setDoc(..., {merge:true})`. 탭 숨김/`pagehide` 시에도 강제 플러시.

### 핵심 함수

| 함수 | 역할 |
|---|---|
| `normalize(s)` | 로드한 원본 데이터를 안전한 스키마로 정규화(누락 필드 보정, 통화 검증) |
| `ruleHitsDate(rule, d)` | 특정 날짜에 고정 항목이 발생하는지 판정(주/월/년 + 시작·종료일 + 말일 보정) |
| `entriesForDate(dk)` | 그날의 일반 거래 + 고정 발생을 합쳐 반환 |
| `aggregate(from, to)` | 기간 집계 — 수입/지출 합, 카테고리별 합, 일별 합 |
| `toKRW(amount, cur)` | 통화 금액을 원화로 환산(환율표 `rateKRW` 사용) |
| `wonShort()` / `fmtCur()` | 금액 축약·통화 기호 포맷(언어별) |

---

## 🗂️ 데이터

모든 데이터는 하나의 `state` 객체에 담깁니다. `freshState()`가 초기값을, `normalize()`가 스키마를 강제합니다.

```js
state = {
  nick:    "나의 장부",     // 표시용 별칭 (로그인 시 Google displayName)
  created: "2026-07-01",    // 생성일 (YYYY-MM-DD)
  tx:        [ …거래… ],    // 일반 거래 배열
  recurring: [ …고정… ],    // 고정 반복 규칙 배열
  cats:      { in:[…], out:[…] }  // 카테고리 정의(기본 + 사용자 추가)
}
```

### 거래 항목 `state.tx[]`

| 필드 | 타입 | 설명 |
|---|---|---|
| `id` | string | 고유 id (`Date.now().toString(36)+random`) |
| `date` | string | 날짜 `YYYY-MM-DD` |
| `type` | `"in"` \| `"out"` | 수입 / 지출 |
| `cur` | `"KRW"`\|`"USD"`\|`"AUD"`\|`"PHP"` | 통화 |
| `amount` | number | 금액(KRW는 정수, 외화는 소수 2자리) |
| `cat` | string | 카테고리 id (예: `food`, `salary`) |
| `memo` | string | 메모(선택) |

```jsonc
{ "id":"lx8q3f21", "date":"2026-07-01", "type":"out",
  "cur":"KRW", "amount":8500, "cat":"food", "memo":"점심" }
```

### 고정 반복 항목 `state.recurring[]`

| 필드 | 타입 | 설명 |
|---|---|---|
| `id` | string | 고유 id |
| `type` | `"in"` \| `"out"` | 수입 / 지출 |
| `cur` | 통화 코드 | 통화 |
| `amount` | number | 금액 |
| `cat` | string | 카테고리 id |
| `memo` | string | 이름/메모(예: 넷플릭스) |
| `freq` | `"monthly"`\|`"weekly"`\|`"yearly"` | 주기 |
| `dom` | number(1–31) | 매월/매년일 때 “며칠” (말일 보정: 31일→2월은 28/29일) |
| `dow` | number(0–6) | 매주일 때 요일(0=일) |
| `mon` | number(1–12) | 매년일 때 “몇 월” |
| `start` / `end` | string | 유효 시작/종료일(선택, `YYYY-MM-DD`) |

```jsonc
{ "id":"m2k9d0aa", "type":"in", "cur":"KRW", "amount":3200000,
  "cat":"salary", "memo":"월급", "freq":"monthly", "dom":25,
  "start":"", "end":"" }
```

### 카테고리 `state.cats`

기본 카테고리가 하드코딩되어 있고(지출 10종 + 수입 5종), 사용자가 시트에서 “＋”로 새 카테고리를 추가하면 `state.cats[type]`에 append됩니다.

| 구분 | 기본 카테고리(id · 이모지 · 한글) |
|---|---|
| 지출(`out`) | 식비🍚 · 카페·간식☕ · 교통🚌 · 쇼핑🛍️ · 생활🏠 · 의료·건강💊 · 문화·여가🎮 · 구독📺 · 적금·저축🏦 · 기타💸 |
| 수입(`in`) | 월급💰 · 용돈🎁 · 부수입💼 · 이자·배당🪙 · 기타➕ |

---

## 💾 저장소 / DB

앱은 두 가지 저장 모드를 가집니다(`mode` = `"cloud"` 또는 `"guest"`).

### 1) 클라우드 — Firebase Firestore

- **프로젝트:** `haru-221ae` (형제 앱 `haru`의 Firebase 프로젝트를 **재사용**하고 컬렉션만 분리)
- **컬렉션 / 문서:** `jangbu/{uid}` — 사용자 1명당 문서 1개에 `state` 전체를 저장
- **인증:** Google 로그인(`GoogleAuthProvider`), 팝업 우선 → 차단 시 리다이렉트 폴백
- **오프라인:** `persistentLocalCache`(단일 탭) + `experimentalForceLongPolling`으로 오프라인/캐시 우선 로드 지원
- **웹 apiKey:** 코드에 노출된 `AIza…` 값은 **공개용 웹 설정값**(비밀번호가 아님). 실제 접근 통제는 아래 Firestore 보안 규칙이 담당합니다.

**필요한 Firestore 보안 규칙:**

```
match /jangbu/{uid} {
  allow read, write: if request.auth != null && request.auth.uid == uid;
}
```

> 규칙이 없으면 클라우드 저장이 안 되므로, 그 전까지는 “이 기기에서만 쓰기(게스트)”를 권장하는 안내가 앱에 표시됩니다.

### 2) 게스트 / 오프라인 폴백 — localStorage

로그인 없이 쓰면 모든 데이터가 브라우저 `localStorage`에만 저장됩니다.

| 키 | 용도 |
|---|---|
| `jangbu-guest` | 게스트 모드의 전체 `state`(JSON) |
| `jangbu-mode` | 마지막 사용 모드(`"guest"` / `"cloud"`) — 재방문 시 자동 복귀 |
| `jangbu-theme` | 테마(`"light"` / `"dark"`) — `<head>` 인라인 스크립트가 FOUC 방지용으로 먼저 읽음 |
| `jangbu-lang` | 언어(`"ko"` / `"en"`) |
| `jangbu-rates` | 환율 캐시(`{rateKRW, meta}`) — 12시간 캐시 |

백업: 설정 → “데이터 내보내기”로 `state`를 `jangbu-backup-YYYY-MM-DD.json` 파일로 저장.

---

## 🌐 외부 API · 의존성

| 서비스 | 용도 | 키 필요 | 넣는 위치 |
|---|---|---|---|
| **Firebase**(App/Auth/Firestore) SDK 12.14.0 | 인증·클라우드 저장 | 웹 설정값(공개) | 코드 상수 `FIREBASE_CONFIG` |
| **open.er-api.com** `/v6/latest/USD` | 실시간 환율(1차) | ❌ 키 불필요(무료·CORS 허용) | 없음 |
| **jsdelivr currency-api** (`@fawazahmed0/currency-api`) `/v1/currencies/usd.json` | 환율(2차 폴백) | ❌ 불필요 | 없음 |

**환율 로직:** 기준 통화는 **원화(KRW)**. `rateKRW = {KRW:1, USD, AUD, PHP}`(통화 1단위당 원화). API가 USD 기준 환율을 주면 이를 원화 환산값으로 변환해 저장하고 `localStorage["jangbu-rates"]`에 캐시합니다. 캐시가 12시간 이내면 재요청하지 않고, 두 API가 모두 실패하면 하드코딩 근사값(`USD≈1380, AUD≈900, PHP≈24`)으로 폴백합니다. 설정 화면에서 “환율 새로고침”으로 강제 갱신 가능.

> 참고: `haru`/`moa` 형제 앱에 나오는 Kakao·TravelTime·Google·Skyscanner·Web Speech 등은 **이 앱에는 사용되지 않습니다.** 외부 호출은 Firebase + 환율 API 뿐입니다.

---

## ▶️ 로컬 실행 방법

`package.json`·빌드가 없으므로 정적 파일 서버로 열기만 하면 됩니다. (ES 모듈 import 때문에 `file://` 직접 열기보다 로컬 서버 권장.)

```bash
# 저장소 폴더에서 아무 정적 서버나 사용
python -m http.server 8000
#  → http://localhost:8000  접속

# 또는 Node가 있으면
npx serve .
```

> Google 로그인은 승인된 도메인(`localhost`, GitHub Pages 도메인)에서만 동작합니다. 로컬에서 로그인이 막히면 “로그인 없이 이 기기에서만 쓰기(게스트)”로 모든 기능을 사용할 수 있습니다.

---

## 🚀 배포

**GitHub Pages** 정적 호스팅으로 배포됩니다(빌드 단계 없음).

1. 리포지토리 **Settings → Pages**에서 소스 브랜치를 `main`(루트 `/`)로 설정.
2. `index.html`을 푸시하면 자동 배포.
3. 결과: https://clayborneyeounjunlee.github.io/jangbu/
4. Firebase 콘솔의 **Authentication → 승인된 도메인**에 Pages 도메인을 추가해야 Google 로그인이 동작.

> Firebase Hosting은 사용하지 않으며 리포에 `firebase.json`/`.firebaserc`도 없습니다. Firebase는 오직 Auth + Firestore 백엔드로만 쓰입니다.

---

## 📁 파일 구조

```
jangbu/
└── index.html   # 앱 전체 — 마크업 + CSS + JS(ES 모듈)가 한 파일에
```

`index.html` 내부 논리 구성(주석 섹션 기준):

```
<head>
 ├─ 테마 FOUC 방지 인라인 스크립트 (localStorage["jangbu-theme"] 선반영)
 └─ <style> CSS 변수 기반 라이트/다크 테마, 캘린더·시트·탭바 스타일
<body>
 ├─ #screen-loading / #screen-auth / #app  (3개 최상위 스크린)
 ├─ 4개 탭 화면(달력/고정/분석/설정) + FAB + 하단 탭바
 ├─ #sheet-wrap (하단 모달 시트) · #toast
 └─ <script type="module">
     ├─ FIREBASE_CONFIG / initFirebase()      Firebase 동적 로드
     ├─ 유틸(날짜키·이스케이프·금액 포맷)
     ├─ 통화/환율(CURS, rateKRW, fetchRates, toKRW)
     ├─ 다국어(T = {ko, en}, tr())
     ├─ 카테고리(DEFAULT_CATS)
     ├─ 상태/저장(state, normalize, save/flushSave, Firestore setDoc)
     ├─ 반복 규칙 계산(ruleHitsDate, entriesForDate, aggregate)
     ├─ 렌더러(renderCalendar / renderRecurring / renderAnalysis / renderProfile)
     ├─ 시트(거래·고정·날짜 상세) · 토스트 · FAB
     ├─ 테마 / 언어 토글
     └─ 로그인(Google·게스트) · 초기화 부팅 IIFE
```

---

## 🔗 관련 앱 (moa 컬렉션)

- **모아 허브:** https://clayborneyeounjunlee.github.io/moa/ — 앱 상단 우측 ◈ 버튼으로 이동
- 이 앱은 형제 앱 **haru**의 배선(인증·저장·테마·다국어 구조)을 복제해 만든 가계부 버전으로, Firebase 프로젝트(`haru-221ae`)를 공유하되 Firestore 컬렉션은 `jangbu`로 분리했습니다.

---

<p align="center"><sub>장부 · Jangbu · moa 컬렉션 · 브랜드 컬러 <b>#7c5cff</b> (violet)</sub></p>
