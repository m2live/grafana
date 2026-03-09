# Modifications to Grafana Source Code

This file documents modifications made to the Grafana source code, as required by the AGPL-3.0 license.

## Build Strategy

All custom files are kept under `custom/` and overlaid during the Docker final stage so the upstream Grafana source tree and build cache remain intact.

Current overlay behavior:

```dockerfile
COPY custom/public/img/ ./public/img/
COPY custom/public/views/index.html ./public/views/index.html
# COPY custom/public/views/error.html ./public/views/error.html
```

```text
custom/
  public/
    views/
      index.html              # Custom main HTML template
      error.html              # Optional custom error page (not currently copied)
    img/
      fav32.png               # M2live favicon
      apple-touch-icon.png    # M2live apple-touch-icon
      grafana_icon.svg        # M2live logo SVG
```

## Modified Files

### `public/views/index.html`

The default Grafana HTML template is replaced by `custom/public/views/index.html`. Compared with upstream `public/views/index.html`, the custom template adds the following behavior for the M2live Cloud service.

**Viewport and layout**
- Changed the viewport meta tag from responsive mobile width to a fixed desktop layout (`width=1024, initial-scale=1, shrink-to-fit=no`)
- Overrode root font size to `13px`
- Forced dashboard containers to keep a desktop-style minimum width (`1024px`) with horizontal scrolling
- Added layout enforcement logic for small browser widths so panel positioning and container height remain stable

**Branding**
- Replaced the mega menu logo text `Grafana` with `M2Live Cloud`
- Removed the ` - Grafana` suffix from document titles and replaced standalone `Grafana` titles with `M2live`
- Changed preloader/loading failure text from `Grafana` to `M2live`

**Mega menu / navigation**
- Rewrote `grafanaBootData.navTree` to remove default entries: `starred`, `bookmarks`, `explore`, `drilldown`, `alerting`, `connections`, `cfg`, `apps`, `dashboards/browse`
- Added custom top-level groups: `서비스 통계`, `운영 관리`, `비용 관리`, `기술지원`
- Added custom child links to dashboards and plugin pages under those groups
- Forced custom menu groups to stay expanded through `localStorage`
- Forced some default groups (`북마크`, `연결`, `관리`) closed through `localStorage`
- Changed top-level custom menu clicks to toggle collapse/expand instead of navigating to `#`
- Removed bookmark/pin buttons from custom top-level menu items
- Added current-page highlight styling for nav items and mapped specific dashboard IDs to their corresponding menu entries

**UI visibility controls**
- Hid all breadcrumb items except the last one
- Hid the organization select prefix icon in the mega menu
- Hid profile page elements: home dashboard selector, org table, sessions section
- Hid dashboard toolbar buttons: favorite, export, share
- Hid the top help button
- Hid panel menu buttons for non-admin users
- Hid the experimental-theme helper text in the theme settings screen

**Authentication, session, and profile flow**
- On first login, called `/api/plugin-proxy/winesoft-usermanager-app/api/v1/accounts/signin` to initialize the account and redirect to the assigned destination/org
- On `/login`, retried once automatically when the `oauth_state` cookie is missing
- Added a client-side idle timeout: after 1 hour of inactivity, a countdown banner appears and redirects to `/logout`
- Called `/api/plugin-proxy/winesoft-usermanager-app/api/v1/accounts/me` once on page load to trigger auth middleware/account sync
- Rewrote the user menu so `/profile` points to `/a/winesoft-usermanager-app/profile`
- Hid password and notification menu entries, kiosk mode, and blog-post entries from the user menu
- Redirected direct `/profile` and `/profile/*` navigation to the plugin profile page

**Search dialog (`kbar`) customization**
- Added Korean keyword search support for custom menu items
- Hid top-level menu items whose links are `href="#"`
- Added custom search result rendering, selection, and keyboard navigation
- Added Korean IME composition handling for search input

**Announcements / popup modal**
- On home entry, fetched popup announcements from the plugin API
- Fetched account info first to apply customer-specific popup filtering
- Filtered announcements by publish/expire time and sorted them by priority/date
- Rendered draggable modal popups with stacked sequence handling
- Added "hide for today" behavior backed by `localStorage`

**Footer, legal, and notices**
- Added a fixed bottom footer with WineSOFT copyright and a privacy-policy entry point
- Embedded a privacy policy document in the page and rendered it into a modal dialog
- Added an `오픈소스 라이선스` item to the user menu
- Added an OSS license modal with links to GPL/AGPL information and the source-code repository

### `public/img/apple-touch-icon.png`
- Replaced the default Grafana apple-touch-icon with the M2live brand icon

### `public/img/fav32.png`
- Replaced the default Grafana favicon (32x32) with the M2live brand icon

### `public/img/grafana_icon.svg`
- Replaced the default Grafana logo SVG with the M2live brand logo

---

# Grafana 소스코드 수정 사항

AGPL-3.0 라이선스에 따라 Grafana 소스코드의 수정 내역을 기록합니다.

## 빌드 전략

모든 커스텀 파일은 `custom/` 디렉토리에 보관하며, Docker 최종 스테이지에서 오버레이하는 방식으로 적용합니다. 이 방식으로 Grafana 원본 소스 트리와 upstream 빌드 캐시를 그대로 유지합니다.

현재 오버레이 동작은 다음과 같습니다.

```dockerfile
COPY custom/public/img/ ./public/img/
COPY custom/public/views/index.html ./public/views/index.html
# COPY custom/public/views/error.html ./public/views/error.html
```

```text
custom/
  public/
    views/
      index.html              # 커스텀 메인 HTML 템플릿
      error.html              # 선택적 에러 페이지 (현재는 미복사)
    img/
      fav32.png               # M2live 파비콘
      apple-touch-icon.png    # M2live apple-touch-icon
      grafana_icon.svg        # M2live 로고 SVG
```

## 수정 파일

### `public/views/index.html`

기본 Grafana HTML 템플릿은 `custom/public/views/index.html`로 대체됩니다. upstream `public/views/index.html`과 비교했을 때 다음 커스터마이징이 추가되어 있습니다.

**뷰포트 및 레이아웃**
- 모바일 반응형 뷰포트 대신 고정 데스크톱 레이아웃(`width=1024, initial-scale=1, shrink-to-fit=no`)으로 변경
- 루트 폰트 크기를 `13px`로 오버라이드
- 대시보드 컨테이너에 최소 너비(`1024px`)와 가로 스크롤을 강제하여 데스크톱형 배치를 유지
- 작은 브라우저 폭에서 패널 좌표와 컨테이너 높이가 무너지지 않도록 강제 보정 스크립트 추가

**브랜딩**
- 메가메뉴 좌상단 `Grafana` 텍스트를 `M2Live Cloud`로 대체
- 문서 타이틀에서 ` - Grafana` 접미사를 제거하고, 타이틀이 `Grafana` 단독일 경우 `M2live`로 치환
- 프리로더와 로딩 실패 문구의 `Grafana` 표기를 `M2live`로 변경

**메가메뉴 / 내비게이션**
- `grafanaBootData.navTree`를 재구성하여 기본 항목 `starred`, `bookmarks`, `explore`, `drilldown`, `alerting`, `connections`, `cfg`, `apps`, `dashboards/browse` 제거
- 커스텀 상위 메뉴 `서비스 통계`, `운영 관리`, `비용 관리`, `기술지원` 추가
- 각 상위 메뉴 하위에 대시보드 링크 및 플러그인 페이지 링크 추가
- `localStorage`를 이용해 커스텀 메뉴는 기본 펼침 상태로 고정
- 일부 기본 메뉴(`북마크`, `연결`, `관리`)는 기본 닫힘 상태로 고정
- `href="#"` 상위 메뉴 클릭 시 이동 대신 접기/펼치기 토글로 동작 변경
- 커스텀 상위 메뉴의 bookmark/pin 버튼 제거
- 현재 페이지 메뉴 항목에 강조 스타일을 적용하고, 특정 대시보드 ID를 해당 메뉴와 매핑하여 활성 상태 표시

**UI 가시성 제어**
- 브레드크럼은 마지막 항목만 보이도록 변경
- 메가메뉴 조직 선택 셀렉트박스의 prefix 아이콘 숨김
- 프로필 페이지의 홈 대시보드 선택기, 조직 테이블, 세션 섹션 숨김
- 대시보드 툴바의 즐겨찾기, 내보내기, 공유 버튼 숨김
- 상단 Help 버튼 숨김
- 비관리자 사용자에게 패널 메뉴 버튼 숨김
- 테마 설정의 실험적 테마 안내 문구 숨김

**인증, 세션, 프로필 흐름**
- 첫 로그인 시 `/api/plugin-proxy/winesoft-usermanager-app/api/v1/accounts/signin` 호출로 계정 초기화 및 목적지/조직 리다이렉트 수행
- `/login`에서 `oauth_state` 쿠키가 없으면 1회 자동 재시도
- 1시간 무활동 시 카운트다운 배너를 띄우고 `/logout`으로 이동하는 클라이언트 측 idle timeout 추가
- 페이지 로드 시 `/api/plugin-proxy/winesoft-usermanager-app/api/v1/accounts/me`를 1회 호출하여 인증 미들웨어/계정 동기화 트리거
- 사용자 메뉴의 `/profile` 링크를 `/a/winesoft-usermanager-app/profile`로 교체
- 사용자 메뉴에서 비밀번호 변경, 알림 이력, kiosk mode, 블로그 최근 글 항목 숨김
- `/profile` 및 `/profile/*` 직접 진입 시 플러그인 프로필 페이지로 리다이렉트

**검색 다이얼로그 (`kbar`) 커스터마이징**
- 커스텀 메뉴에 대한 한글 키워드 검색 지원 추가
- `href="#"`인 최상위 메뉴 항목을 검색 결과에서 숨김
- 커스텀 검색 결과 렌더링, 선택 상태 표시, 키보드 이동 처리 추가
- 한글 IME 조합 입력에 대한 예외 처리 추가

**공지 팝업**
- 홈 진입 시 플러그인 API에서 팝업 공지 조회
- 계정 정보를 먼저 조회해 고객별 공지 필터링에 반영
- 게시 기간과 우선순위 기준으로 공지를 필터링/정렬
- 드래그 가능한 모달 팝업을 순차적으로 노출
- `localStorage` 기반 "오늘하루 안보기" 기능 추가

**푸터, 법적 고지, 오픈소스 안내**
- 하단 고정 푸터와 WineSOFT 저작권 문구 추가
- 푸터에서 열 수 있는 개인정보 처리방침 모달 추가
- 페이지 내에 개인정보 처리방침 원문을 포함하고 모달 HTML로 렌더링
- 사용자 메뉴에 `오픈소스 라이선스` 항목 추가
- GPL/AGPL 안내 및 소스코드 저장소 링크를 제공하는 OSS 라이선스 모달 추가

### `public/img/apple-touch-icon.png`
- Grafana 기본 apple-touch-icon을 M2live 브랜드 아이콘으로 교체

### `public/img/fav32.png`
- Grafana 기본 파비콘(32x32)을 M2live 브랜드 아이콘으로 교체

### `public/img/grafana_icon.svg`
- Grafana 기본 로고 SVG를 M2live 브랜드 로고로 교체
