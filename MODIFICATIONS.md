# Modifications to Grafana Source Code

This file documents modifications made to the Grafana source code, as required by the AGPL-3.0 license.

## Build Strategy

All custom files are located in the `custom/` directory and are **not** modified in the original Grafana source tree.
During the Docker build, the final stage overlays flavor-specific and shared assets onto the built `public/` directory,
so that the upstream JS/Go build cache remains valid even when customizations change.

The `FLAVOR` build argument (`hub` or `ops`) selects the HTML template variant:

```bash
docker build --build-arg FLAVOR=hub -t m2live-hub .
docker build --build-arg FLAVOR=ops -t m2live-ops .
```

```
custom/
  public/
    views/
      index.hub.html           # Hub flavor HTML template
      index.ops.html           # Ops flavor HTML template
    img/
      fav32.png                # M2live favicon (shared)
      apple-touch-icon.png     # M2live apple-touch-icon (shared)
      grafana_icon.svg         # M2live logo SVG (shared)
```

## Modified Files

### `public/views/index.html`

The following customizations were added to the Grafana main HTML template for the M2live Cloud service.

**Branding**
- Replaced "Grafana" logo text in the mega menu with "M2live Cloud" (MutationObserver + XPath)
- Removed `" - Grafana"` suffix from page title; replaced standalone `"Grafana"` with `"M2live"` (MutationObserver on `<title>`)
- Changed preloader `aria-label` and loading failure message from "Grafana" to "M2live"

**Navigation (Mega Menu) Restructuring**
- Removed default nav items from `navTree`: starred, bookmarks, explore, drilldown, alerting, connections, cfg, apps, dashboards/browse
- Added custom menu groups with child items: Service Statistics, Operations, Billing, Technical Support
- Set custom menu groups to expanded by default via `localStorage`
- Changed top-level menu anchor click to toggle collapse/expand instead of navigation
- Removed bookmark (pin-icon) buttons from custom top-level menu items (CSS + JS)

**UI Element Hiding (CSS)**
- Breadcrumbs: only last item shown (workaround for `Page Not Found` display issue)
- Dashboard toolbar: favorite, export, share buttons hidden
- Top bar: Help button hidden
- Profile page: home dashboard dropdown, org table, sessions table hidden
- Mega menu: org select prefix icon (building SVG) hidden
- Dashboard grid container: vertical scroll hidden

**Dashboard Layout**
- Dashboard grid minimum width set to 1024px with horizontal scroll

**Active Menu Highlight**
- CSS styles for `aria-current="page"` menu items with background color and left gradient bar
- JS to apply `aria-current="page"` to mega menu items matching the current dashboard ID

**Authentication / Session**
- First login: calls plugin API (`/api/v1/accounts/signin`) to initialize account and redirect to assigned org
- Login page: automatic retry redirect when `oauth_state` cookie is missing
- Client-side idle session timeout: 1-hour inactivity detection with countdown banner and auto-logout to `/logout`
- Password change menu redirected to Keycloak account security page; menu text changed accordingly

**User Menu Customization**
- Hidden menu items: notification history, kiosk mode, blog posts

**Plugin API Calls**
- Calls `/api/v1/accounts/me` once on every page load (auth middleware trigger)

**Command Palette (kbar) Customization**
- Korean keyword search support for custom menu items
- Top-level menu items with `href="#"` hidden from search results
- Custom search result rendering with keyboard navigation and Korean IME composition handling

**Home Page Popup Announcements**
- Fetches popup-type announcements from plugin API on home page entry
- Displays stacked modal popups with priority ordering and "hide for today" support

**Footer**
- Page bottom footer added: copyright © 2026 WINESOFT Inc., link to license (GitHub).

### `public/img/apple-touch-icon.png`
- Replaced Grafana apple-touch-icon with M2live brand icon

### `public/img/fav32.png`
- Replaced Grafana favicon (32x32) with M2live brand icon

### `public/img/grafana_icon.svg`
- Replaced Grafana logo SVG with M2live brand logo

---

# Grafana 소스코드 수정 사항

AGPL-3.0 라이선스에 따라 Grafana 소스코드의 수정 내역을 기록합니다.

## 빌드 전략

모든 커스텀 파일은 `custom/` 디렉토리에 위치하며, Grafana 원본 소스 트리는 수정하지 않습니다.
Docker 빌드 최종 스테이지에서 flavor별 HTML 템플릿과 공통 에셋을 빌드된 `public/` 디렉토리 위에 오버레이하여,
커스터마이징 변경 시에도 upstream JS/Go 빌드 캐시가 유지되도록 합니다.

`FLAVOR` 빌드 인자(`hub` 또는 `ops`)로 HTML 템플릿 변형을 선택합니다:

```bash
docker build --build-arg FLAVOR=hub -t m2live-hub .
docker build --build-arg FLAVOR=ops -t m2live-ops .
```

```
custom/
  public/
    views/
      index.hub.html           # Hub용 HTML 템플릿
      index.ops.html           # Ops용 HTML 템플릿
    img/
      fav32.png                # M2live 파비콘 (공통)
      apple-touch-icon.png     # M2live apple-touch-icon (공통)
      grafana_icon.svg         # M2live 로고 SVG (공통)
```

## 수정 파일

### `public/views/index.html`

M2live Cloud 서비스를 위해 Grafana 메인 HTML 템플릿에 다음 커스터마이징을 추가하였습니다.

**브랜딩**
- 메가메뉴 좌상단 "Grafana" 로고 텍스트를 "M2live Cloud"로 대치 (MutationObserver + XPath)
- 페이지 타이틀에서 `" - Grafana"` 접미사 제거, `"Grafana"` 단독 노출 시 `"M2live"`로 변경 (MutationObserver)
- 프리로더 `aria-label` 및 로딩 실패 안내 문구를 "Grafana"에서 "M2live"로 변경

**내비게이션 (메가메뉴) 구조 변경**
- `navTree`에서 기본 메뉴 항목 제거: starred, bookmarks, explore, drilldown, alerting, connections, cfg, apps, dashboards/browse
- 커스텀 메뉴 그룹 추가: 서비스 통계, 운영 관리, 비용 관리, 기술지원 (각 그룹 하위에 대시보드 또는 플러그인 페이지 링크 포함)
- 커스텀 메뉴 그룹을 `localStorage`를 통해 기본 펼침 상태로 설정
- 상위 메뉴 클릭 시 페이지 이동 대신 하위 메뉴 확장/축소 토글로 동작 변경
- 커스텀 상위 메뉴 항목의 북마크(pin-icon) 버튼 제거 (CSS + JS)

**UI 요소 숨김 (CSS)**
- 브레드크럼: 마지막 항목만 노출 (`Page Not Found` 표시 문제 우회)
- 대시보드 툴바: 즐겨찾기, 내보내기, 공유 버튼 숨김
- 상단바: 도움말(Help) 버튼 숨김
- 프로필 페이지: 홈 대시보드 설정, 조직 테이블, 세션 테이블 숨김
- 메가메뉴: 조직 선택 셀렉트박스의 접두 아이콘(건물 SVG) 숨김
- 대시보드 그리드 컨테이너: 세로 스크롤 숨김

**대시보드 레이아웃**
- 대시보드 그리드 최소 너비 1024px 설정 및 수평 스크롤 적용

**메뉴 활성 상태 표시**
- `aria-current="page"` 메뉴 항목에 배경색 및 좌측 그라데이션 바 스타일 적용 (CSS)
- 현재 대시보드 ID에 해당하는 메가메뉴 항목에 `aria-current="page"` 자동 부여 (JS)

**인증 / 세션**
- 첫 로그인 시 플러그인 API(`/api/v1/accounts/signin`)를 호출하여 계정 초기화 및 소속 조직으로 리디렉션
- 로그인 페이지에서 `oauth_state` 쿠키 부재 시 1회 자동 재시도 리디렉션
- 클라이언트 측 유휴 세션 타임아웃: 1시간 무활동 감지 후 카운트다운 배너 노출 및 `/logout`으로 자동 로그아웃
- 비밀번호 변경 메뉴를 Keycloak 계정 보안 페이지로 리디렉션, 메뉴 텍스트를 "계정 보안"으로 변경

**사용자 메뉴 커스터마이징**
- 알림 이력, 키오스크 모드, 블로그 최근 글 메뉴 항목 숨김

**플러그인 API 호출**
- 페이지 로드 시 `/api/v1/accounts/me`를 1회 호출 (인증 미들웨어 트리거)

**검색 다이얼로그 (kbar) 커스터마이징**
- 커스텀 메뉴 항목에 대한 한글 키워드 검색 지원
- `href="#"`인 최상위 메뉴 항목을 검색 결과에서 숨김
- 커스텀 검색 결과 렌더링, 키보드 내비게이션 및 한글 IME 조합 처리

**홈 페이지 팝업 공지**
- 홈 진입 시 플러그인 API에서 팝업 유형 공지사항 조회
- 우선순위 기반 격자식 겹침 모달 팝업 노출, "오늘하루 안보기" 기능 지원

**푸터**
- 페이지 하단에 푸터 추가: 저작권 © 2026 WINESOFT Inc., 라이선스 링크(GitHub).

### `public/img/apple-touch-icon.png`
- Grafana 기본 apple-touch-icon을 M2live 브랜드 아이콘으로 교체

### `public/img/fav32.png`
- Grafana 기본 파비콘(32x32)을 M2live 브랜드 아이콘으로 교체

### `public/img/grafana_icon.svg`
- Grafana 기본 로고 SVG를 M2live 브랜드 로고로 교체
