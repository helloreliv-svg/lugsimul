# 릴리브 러그 견적 계산기

러그 사이즈 견적 + 거실 배치 시뮬레이터. **`index.html` 한 파일**(HTML/CSS/JS, 빌드 도구 없음)과 `images/` 폴더로 구성된다.
배포: 깃허브 `main`에 푸시하면 Netlify가 자동 배포한다. 아임웹 홈페이지의 플로팅 버튼이 이 사이트로 링크한다.

## 수정 요청이 왔을 때의 표준 순서
1. `git pull`로 최신 상태를 받는다.
2. `index.html`에서 해당 부분을 찾아 **필요한 부분만** 수정한다. (같은 문구가 여러 곳에 있으면 모두 찾아 알려준다.)
3. 수정한 화면을 열어 사용자가 확인할 수 있게 한다. (`file:///…/index.html`)
4. **사용자가 확인하고 "푸시해줘"라고 하면** 커밋하고 `git push`한다.
5. 푸시 후 "1~2분 뒤 https://lugsimulator.netlify.app/ 에서 Ctrl+F5로 확인"하라고 안내한다.
- 깃허브 로그인(비밀번호 입력)은 사용자가 직접 한다. 비밀번호를 대신 입력하거나 파일에 적지 않는다.

## 작업 규칙 (중요)
- **사용자가 "푸시해줘/커밋해줘"라고 말하기 전에는 커밋·푸시하지 않는다.** 푸시하면 바로 실제 손님에게 보인다.
- 이 저장소는 **공개일 수 있다.** 비밀번호·API 키·개인 이메일을 파일에 넣지 않는다.
- 수정 후 로컬에서 확인한다. `index.html`은 `images/` 폴더와 같은 위치에서 열어야 이미지가 보인다. (`file:///…/index.html` 또는 로컬 서버)
- 한 파일이 크다. 수정할 때는 필요한 부분만 고치고, 파일 전체를 덮어쓰지 않는다.
- 커밋 메시지는 한국어로, 무엇을 왜 바꿨는지 적는다.

## 화면 구성 (`showScreen`)
- `screenHome`: 진입 화면 (견적 확인하기 / 러그 배치해보기)
- `screenQuote`: 가로·세로 입력 → 제작 방식 3종 비교 카드 → 주문 옵션 안내 → CTA
- `screenPlanner`: 평형 선택, 가구(소파·TV·테이블), 기성/맞춤 러그 배치, 금액 사이드바

## 견적 계산 규칙 — 함부로 바꾸지 말 것
- 원단 **폭 200cm 고정**(`ROLL_WIDTH`), 1장 최대 **200×400cm**(`MAX_LENGTH`). 초과하면 분할.
- 주문 옵션은 **"길이 N cm 이하"(50~400cm, 10cm 단위)**. 안내 박스는 `사이즈 160×250 / 길이 250cm 이하 / X 4` 형태로 보여주고 복사된다(복사 문구는 `사이즈 160×250 / 길이 250cm 이하 선택 / 수량 4 선택`). 50cm보다 짧은 조각도 "50cm 이하"(`MIN_OPTION_LENGTH`)로 청구한다.
- 가격은 **길이를 10cm 단위로 올림(최소 50cm)**(`optionTier`, `LENGTH_STEP`, `MIN_OPTION_LENGTH`) 후 `panelPrice`로 계산. 단가 `UNIT_PRICE`=45,500원/㎡ (폭200 기준 10cm당 9,100원).
- 분할 방식 `splitLayout(mode, W, H)`: `equal`(균등) / `optimal` / `max`. 합계는 올림 때문에 방식마다 다를 수 있다.
- 견적 화면은 3가지 방식을 비교하고, 배치 화면은 **등분(`equal`) 고정**이다.
- 기성 러그 가격은 `PRESET_PRICES`, 크기는 `RUG_PRESETS`. 맞춤 러그는 위 규칙을 쓴다.
- 평형별 거실 크기 `PYEONG_PRESETS`, 가구 크기 `FURNITURE`. 모든 객체의 크기·위치는 cm 단위로 `dataset`에 저장하고 렌더링할 때만 px로 바꾼다(`cmToPx`).

## 선택 동작
- 러그끼리는 다중 선택, 가구끼리는 하나만 선택, 러그↔가구는 서로 배타적이다 (`toggleSelection`, `addToSelection`, `toggleFurniture`).
- 금액 사이드바는 선택과 무관하게 **캔버스에 놓인 모든 러그**를 합산한다(`updateQuote`). 러그를 삭제해야 빠진다.

## 외부 링크 (파일 상단 근처 상수)
- `KAKAO_CHAT_URL`: 카카오톡 채널 상담
- `ORDER_URL`: 맞춤 제작 주문 상세페이지
- `PRESET_ORDER_URL`: 기성 러그 상세페이지
- 버튼은 모두 `window.open(..., "_blank", "noopener")`로 새 탭에서 연다.

## GA4 추적
- `<head>`에 GA4 태그(릴리브 자사몰과 같은 속성)와 `track(name, params)` 도우미가 있다. 이벤트 전송은 `track(...)` 호출로 한다.
- 현재 이벤트: `calc_screen`(screen), `quote_calc`(width,height,pieces,value,currency), `quote_mode_select`(mode), `order_click`·`kakao_click`·`option_copy`(click_from, width,height), `preset_order_click`(click_from).
- **GA4에 이미 등록된 이벤트 범위 맞춤 측정기준과 매개변수 이름이 겹치면 안 된다.** 특히 `source`, `medium`, `campaign`, `link_url`, `page_location`, `query`, `content`, `term`은 이미 다른 용도로 등록돼 있어 쓰지 않는다. (클릭 화면 구분은 `click_from`을 쓴다.)
- 새 매개변수를 추가하면 GA4 관리 → 데이터 표시 → 맞춤 정의에서 등록해야 탐색에서 쓸 수 있다고 사용자에게 안내한다.
- **주의**: `localhost` 등 로컬에서 열어도 이벤트가 실제 GA4로 전송된다. 로컬 테스트 시 실제 데이터에 섞일 수 있다. (개선 아이디어: 실제 배포 주소일 때만 전송)
- 이벤트를 테스트로 검증할 때는 `gtag` 호출을 가로채서 확인한다.

## 이미지
`images/`: `lug.png`(러그 질감), `sofa.png`, `tv.png`, `table.png`, `bed.png`(현재 UI에서는 미사용), `logo.png`, `room.png`(미사용).
CSS와 JS에서 `images/…` 상대 경로로 참조한다. 파일 이름을 바꾸면 참조도 함께 바꾼다.

## 디자인 기준
- 색: 본문 `#544E3B`, 보조 `#6F6C56`, 강조(추천 카드) `#714A32`, 배경 `#f7f4ef`. 폰트 Pretendard.
- 피그마 시안을 기준으로 맞춘 화면이므로, 레이아웃을 바꿀 때는 기존 크기·간격 값을 임의로 바꾸지 말고 사용자와 확인한다.
- 모바일(767px 이하)은 캔버스 위·컨트롤 시트 아래 구조다. 아임웹에서 모바일 방문이 많다(GA4 기준 약 95%).
