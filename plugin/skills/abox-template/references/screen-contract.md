# 화면 계약 — ABOX_DATA v1

`screen/`의 화면은 서버가 넣어 주는 `window.ABOX_DATA`(게시한 데이터 JSON 그대로, 최상위는 객체)만 읽어 그린다. 같은 화면이 세 곳에서 그려진다.

| 그리는 곳                      | `ABOX_DATA`               |
| ------------------------------ | ------------------------- |
| 받는 사람 뷰어 · 오너 미리보기 | 그 아티팩트 버전의 데이터 |
| 웹 템플릿 상세의 미리보기 탭   | 템플릿의 샘플 데이터      |

## 서버가 넣는 방식

- 진입 `screen/index.html`을 내보낼 때만 넣는다. 같은 묶음의 다른 HTML 파일에는 넣지 않는다.
- `<head>` 맨 앞(storage shim 바로 뒤)에 `<script>window.ABOX_DATA=…;</script>`로 넣는다. 화면의 어떤 스크립트보다 먼저 동기로 실행되므로 로딩 처리가 필요 없다.
- 데이터 문자열의 `<`·U+2028·U+2029는 `\u003c`·`\u2028`·`\u2029`로 바뀌어 들어간다. 값에 `</script>`·`<!--`가 있어도 스크립트를 깨지 않고, 읽은 값은 원래 문자열 그대로다.
- `<html>`·`<head>`가 없는 조각 HTML이어도 서버가 문서 틀을 씌우고 같은 자리에 넣는다.
- 계약 번호(지금 1)는 템플릿 버전마다 서버가 기록한다. 화면에서 할 일은 없다.

## 체크리스트

등록(`template push`) 전에 모두 확인한다.

**데이터 읽기**

- [ ] 데이터는 `window.ABOX_DATA`에서만 읽는다. `fetch`·XHR로 데이터를 불러오지 않는다.
- [ ] `window.ABOX_DATA`는 읽기만 한다. 일반 대입이라 바꿀 수는 있지만, 가공한 값은 새 변수에 둔다.
- [ ] 화면은 `index.html` 한 페이지로 만든다. `screen/` 안의 다른 HTML 파일에는 `ABOX_DATA`가 없다.
- [ ] DOM을 다루는 스크립트는 `<body>` 끝에 두거나 `defer`·`type="module"`로 불러온다. `ABOX_DATA`는 그보다 먼저 있다.

**값 넣기**

- [ ] 값은 `textContent`·`createElement`·`append`로 넣는다. `innerHTML`·`outerHTML`·`insertAdjacentHTML`·`document.write`에 데이터를 넣지 않는다.
- [ ] URL 값은 `https://`로 시작할 때만 `href`·`src`에 넣는다(이미지는 `data:image/`도).

**빈 값·긴 값**

- [ ] 선택 필드가 없어도, 배열이 비어도 깨지지 않는다(`?? ''`·`?? []`, 비는 절은 숨김). `required` 필드만 있는 데이터를 넣었다고 보고 코드를 따라가 확인한다.
- [ ] 긴 문자열과 많은 항목에도 레이아웃이 깨지지 않는다(`overflow-wrap: anywhere`, 넓은 표는 자체 `overflow-x: auto`).

**경로·외부 자원**

- [ ] 경로는 상대 경로(`./app.js`·`./img/logo.svg`)로 쓴다. `/`로 시작하면 게시 화면에서 깨지고 `ROOT_RELATIVE_PATHS` 경고가 난다.
- [ ] 외부 스크립트·스타일·이미지·폰트·영상·소리는 `https://` 주소만 열린다(sandbox CSP).
- [ ] CDN 주소는 정확한 버전으로 고정한다(`https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js`). `@latest`나 버전 없는 주소를 쓰지 않는다. 고정된 템플릿 버전으로 만든 아티팩트가 나중에도 같은 모습으로 열려야 한다.
- [ ] `window.claude.*`를 쓰지 않는다. `localStorage`는 새로고침하면 초기화된다.
- [ ] 이미지는 `screen/`의 고정 이미지(로고 등), 데이터의 https 주소, 작은 data URI로 넣는다. 아티팩트마다 로컬 이미지를 첨부하는 방법은 없다.

**디자인**

- [ ] 디자인 스킬이 있으면 그 디자인 원칙을 따르되 이 체크리스트가 먼저다. 없으면 `:root`에 색 토큰을 두고 `@media (prefers-color-scheme: dark)`에서 다시 정의하며, `body` 배경을 정한다.
- [ ] 390px 폭에서 가로 스크롤이 없고 좌우 여백이 16px 이상이다.

## 예시 화면

`references/template-folder.md`의 주간 보고 스키마를 그리는 `screen/index.html`이다. 선택 필드 `risks`·`risks[].owner`가 없어도 그려진다.

```html
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>주간 보고</title>
    <style>
      :root {
        color-scheme: light;
        --bg: #fbfaf7;
        --fg: #1d1c1a;
        --muted: #6b6760;
        --line: #e4e1da;
        --high: #b42318;
      }
      @media (prefers-color-scheme: dark) {
        :root {
          color-scheme: dark;
          --bg: #161513;
          --fg: #ecebe7;
          --muted: #a29d94;
          --line: #34312c;
          --high: #f97066;
        }
      }
      body {
        margin: 0;
        background: var(--bg);
        color: var(--fg);
        font:
          15px/1.6 system-ui,
          sans-serif;
      }
      main {
        max-width: 720px;
        margin: 0 auto;
        padding: 24px 16px;
      }
      h1 {
        margin: 0;
        font-size: 22px;
      }
      h2 {
        margin: 24px 0 8px;
        border-bottom: 1px solid var(--line);
        font-size: 15px;
      }
      h1,
      li {
        overflow-wrap: anywhere;
      }
      .meta {
        margin: 4px 0 0;
        color: var(--muted);
        font-size: 13px;
      }
      .level {
        font-weight: 600;
      }
      .level--high {
        color: var(--high);
      }
    </style>
  </head>
  <body>
    <main>
      <h1 id="title"></h1>
      <p class="meta" id="week"></p>
      <h2>한 일</h2>
      <ul id="done"></ul>
      <section id="risks" hidden>
        <h2>위험</h2>
        <ul id="risk-list"></ul>
      </section>
    </main>
    <script>
      const data = window.ABOX_DATA ?? {}
      const LEVELS = { high: '높음', medium: '보통', low: '낮음' }
      const byId = (id) => document.getElementById(id)
      const listItem = (...parts) => {
        const li = document.createElement('li')
        li.append(...parts) // 문자열은 글자로 들어간다
        return li
      }

      byId('title').textContent = data.title ?? ''
      byId('week').textContent = data.week ? `${data.week} 주` : ''
      for (const text of data.done ?? []) byId('done').append(listItem(text))

      const risks = data.risks ?? [] // 선택 필드는 없을 수 있다
      byId('risks').hidden = risks.length === 0
      for (const risk of risks) {
        const level = document.createElement('span')
        level.className = `level level--${risk.level}`
        level.textContent = LEVELS[risk.level] ?? risk.level
        const owner = risk.owner ? ` · ${risk.owner}` : ''
        byId('risk-list').append(listItem(level, ` ${risk.title}`, owner))
      }
    </script>
  </body>
</html>
```
