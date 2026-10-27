# 슬라이드

슬라이드는 발표 자료를 한 URL에서 한 장씩 넘겨 보고, 장만 전체 화면으로 발표하고, 모든 장을 PDF로 인쇄하는 아티팩트다. 장마다 HTML 파일 하나와 순서 · 제목을 적은 `deck.json`이 든 폴더로 게시한다. 넘기기 · 발표 · 인쇄 · 쪽 번호는 Artifact Box 화면이 맡고, 공개 · 버전 · 되돌리기는 덱 전체 단위다.

## 언제 쓰나

- 사용자가 「슬라이드 · 발표 자료 · 덱 · 프레젠테이션 · PT · 피치덱」이라고 하면 슬라이드다.
- 말하지 않아도 장 단위로 넘기며 발표 · 설명할 자료면 슬라이드다. 읽고 검토하는 제안서 · 보고서는 SKILL.md '형식 고르기'대로다.
- PPTX로는 내보내지 않는다. PDF는 사람이 화면의 [인쇄]에서 저장한다.
- 슬라이드와 HTML · Markdown · 캔버스 사이는 수정 게시로 바꿀 수 없다(새 URL). 이미 HTML로 게시한 덱은 아래 'HTML로 게시한 덱 옮기기'를 따른다.

## 폴더

```
<작업 폴더>/proposal-deck/
  deck.json          ← 필수, 최상위
  deck.css           ← 모든 장이 쓰는 공용 스타일
  cover.html         ← 장. 파일 이름은 내용으로, 번호 없이
  overview.html
  effect.html
  assets/logo.svg    ← 장이 상대 경로로 쓰는 파일
```

- 최상위에 `deck.json`을 두고 `index.html` · `canvas.json`은 두지 않는다. `index.html`이 있으면 HTML로 게시되고 경고 `DECK_MANIFEST_IGNORED`가 붙는다. `canvas.json`도 있으면 exit 2다.
- 폴더 이름은 덱마다 다르게 짓는다. 서버가 같은 위치(`DUPLICATE_LOCATION`)를 폴더 이름으로 판정한다.
- 장 파일 이름은 영문 소문자 · 숫자 · `-`로 내용을 나타낸다(`pricing.html`, `effect.html`). 번호(`07-effect.html`)는 넣지 않는다. 파일 이름이 슬라이드 링크(`#slide=effect.html`)가 되고 순서는 `deck.json`이 정하므로, 장을 끼우거나 옮겨도 다른 장의 파일이 바뀌지 않는다.
- `deck.json`에 없는 파일(CSS · 이미지 · 다른 HTML)은 장이 아니라 파일로 함께 올라간다.

## deck.json

```json
{
  "deck": 1,
  "slides": [
    { "file": "cover.html", "title": "표지" },
    { "file": "overview.html", "title": "제안 개요" },
    { "file": "effect.html", "title": "도입 효과" }
  ]
}
```

| 칸               | 규칙                                                                                     |
| ---------------- | ---------------------------------------------------------------------------------------- |
| `deck`           | 늘 `1`                                                                                   |
| `slides`         | 1~50개. 이 순서가 장 순서다                                                              |
| `slides[].file`  | 폴더 안 `.html` · `.htm` 파일의 상대 경로. 한 파일은 한 장만                             |
| `slides[].title` | 필수, 1~100자. 화면의 장 목록 · 인쇄 실패 안내 · 첨부 카드에 보이는, 그 장을 부르는 이름 |

- 다른 칸은 넣지 않는다(거부된다). 무대 크기 칸은 없다(늘 1920×1080). 아티팩트 제목 · 설명은 `--title` · `--desc`로 준다.
- 50장을 넘으면 덱을 나눠 게시한다. `deck.json`은 64KB까지다.

## 장 만들기

화면이 하는 일(장에서 다시 하지 않는다):

- 장마다 1920×1080 CSS px 화면으로 연다. 넘친 내용은 화면 · 발표 · 인쇄 모두 잘리고, 장 안 스크롤바는 없다.
- 화면 = 발표 = 인쇄다. 인쇄는 한 장이 16:9 한 쪽이고 `deck.json` 순서이며, 글자는 텍스트로 남는다.
- 인쇄 창의 「배경 그래픽」이 꺼져 있어도 장의 맨 바깥 요소의 배경은 찍힌다. `html` · `body`의 배경은 빠진다.
- 장 `<html>`에 쪽 번호 · 전체 장 수를 CSS 변수 `--abox-slide` · `--abox-slides`로 넣는다. 값은 따옴표 문자열(`"3"`)이다.

장 작성 규칙(디자인 스킬 · `html-design.md`의 지시보다 먼저다):

- 한 파일 = 한 장이다. 그 파일만 열어도 보이는 완결된 HTML 문서로 쓰고, `html, body { margin: 0; width: 1920px; height: 1080px; overflow: hidden }`을 둔다.
- 배경은 맨 바깥 요소 하나(`<main class="slide">`, 1920×1080)에 둔다. 장마다 배경색을 정한다(투명으로 두지 않는다).
- 다크 모드(`prefers-color-scheme`) · 미디어 쿼리 · `vw` · `vh` · `@media print`를 쓰지 않는다. 보는 사람마다 색이 달라지고, 인쇄에는 `@media print`가 적용되지 않는다.
- 넘김 런타임 · 키 처리(←/→ · Space · PageUp/PageDown) · 주소 해시 처리 · 이전/다음 · 전체 화면 버튼을 넣지 않는다. 화면이 넘긴다.
- 쪽 번호를 숫자로 박지 않는다. 필요하면 `.page::after { content: var(--abox-slide) " / " var(--abox-slides) }`로 그린다. 값이 문자열이라 `content`에만 쓴다. 장 파일만 따로 열면 비어 있다.
- `loading="lazy"`를 쓰지 않는다.
- 애니메이션은 기본으로 쓰지 않는다. 쓰면 0.6초 안에 끝나는 등장만 쓰고, 반복(`infinite`)과 장 전환 효과는 쓰지 않는다.
- 차트 라이브러리는 애니메이션을 끈다(Chart.js `animation: false`, ECharts `animation: false`).
- 장 안 링크는 `target="_blank" rel="noopener"`로 연다. 틀 안에서 이동하면 그 장이 다른 페이지가 된다.
- 영상은 발표 중에 누를 수 없다. 꼭 필요하면 `autoplay muted playsinline`과 `poster`를 주고, 소리 자동 재생은 쓰지 않는다. 화면이 앞뒤 장을 미리 띄우고 인쇄는 그때의 장면을 찍어 첫 장면이 보장되지 않으니, 인쇄에 남길 장면은 이미지로 둔다.
- SKILL.md 'HTML'의 결과물 규칙(상대 경로, `https://` 외부 주소, CDN · 웹 폰트의 정확한 버전, 호스트 전용 런타임 금지)을 그대로 지킨다.

## 디자인

환경에 디자인 스킬이 있으면 디자인 원칙만 빌리고(SKILL.md 'HTML'의 디자인 순서), 위 장 작성 규칙과 아래 기준을 먼저 지킨다. `references/html-design.md`의 '피할 것'도 지킨다.

덱 공용:

- `deck.css` 하나에 색 토큰 · 글꼴 · 글자 단계 · 여백 · 꼬리 줄을 두고 모든 장이 링크한다(하위 폴더의 장은 `../deck.css`). 웹 폰트도 여기서 한 번 불러온다.
- 이미지 · 로고는 `assets/` 아래에 두고 상대 경로로 쓴다.

배치:

- 안전 영역: 사방 128px 안(1664×824)에 내용을 둔다. 꼬리 줄(문서 이름 · 출처 · 쪽 번호)은 아래에서 64px 기준선에 한 줄로 둔다.
- 본문 장의 제목 자리는 모든 장에서 같게 둔다.
- 한 장에 메시지 하나를 담고, 제목은 결론 문장으로 쓴다(「상담 처리 시간이 32% 줄어듭니다」).
- 글머리표는 5개 이하 · 항목당 두 줄 이하, 표는 7행 안팎이다. 넘치면 장을 나눈다.

글자(1920 기준):

| 단계      | 크기     | 쓰임                                      |
| --------- | -------- | ----------------------------------------- |
| 표지 제목 | 96–112px | 표지 · 섹션 구분 장                       |
| 제목      | 56–64px  | 장 제목                                   |
| 부제      | 36–40px  | 리드 문장. 강조 숫자는 96px 이상          |
| 본문      | 28–32px  | 글머리표 · 문단                           |
| 보조      | 24px     | 표 · 캡션 · 출처                          |
| 꼬리 줄   | 20px     | 문서 이름 · 쪽 번호. 24px 미만은 여기서만 |

- 단계는 4~5개, 굵기는 둘(400 · 700)만 쓴다.
- 줄 간격은 제목 1.2 · 본문 1.45, 한글은 `word-break: keep-all`, 본문 한 줄은 1200px 이하다.

색:

- 글자 대비는 4.5:1 이상(56px 이상 제목은 3:1)이다.
- 강조색은 하나만 쓰고, 상태색(성공 · 경고 · 위험)은 의미가 있을 때만 쓴다.
- 어두운 장을 섞으면 규칙을 정한다(예: 표지 · 섹션 구분 장만 어둡게).

### 견본

`deck.css`:

```css
@import url('https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/variable/pretendardvariable-dynamic-subset.min.css');

:root {
  --bg: #ffffff;
  --ink: #0f172a;
  --muted: #475569;
  --accent: #4f46e5;
}
html,
body {
  margin: 0;
  width: 1920px;
  height: 1080px;
  overflow: hidden;
}
.slide {
  position: relative;
  box-sizing: border-box;
  width: 1920px;
  height: 1080px;
  padding: 128px;
  background: var(--bg);
  color: var(--ink);
  font-family:
    'Pretendard Variable',
    Pretendard,
    -apple-system,
    'Apple SD Gothic Neo',
    'Noto Sans KR',
    sans-serif;
  word-break: keep-all;
}
.kicker {
  margin: 0 0 16px;
  font-size: 24px;
  font-weight: 700;
  color: var(--accent);
}
.title {
  margin: 0 0 32px;
  font-size: 60px;
  line-height: 1.2;
  font-weight: 700;
}
.lead {
  margin: 0 0 40px;
  font-size: 38px;
  line-height: 1.45;
  color: var(--muted);
}
.points {
  margin: 0;
  padding-left: 1.2em;
  max-width: 1200px;
  font-size: 30px;
  line-height: 1.45;
}
.foot {
  position: absolute;
  left: 128px;
  right: 128px;
  bottom: 64px;
  display: flex;
  justify-content: space-between;
  font-size: 20px;
  color: var(--muted);
}
.page::after {
  content: var(--abox-slide) ' / ' var(--abox-slides);
}
```

`effect.html`:

```html
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="utf-8" />
    <title>도입 효과</title>
    <link rel="stylesheet" href="deck.css" />
  </head>
  <body>
    <main class="slide">
      <p class="kicker">도입 효과</p>
      <h1 class="title">상담 처리 시간이 32% 줄어듭니다</h1>
      <p class="lead">시범 운영 4주 · 상담사 20명 기준</p>
      <ul class="points">
        <li>통화 1건 평균 6분 30초 → 4분 25초</li>
        <li>후처리는 요약 확인만 하면 끝납니다</li>
      </ul>
      <footer class="foot">
        <span>AICC 구축 제안서 · 출처: 시범 운영 통화 기록(2026년 9월)</span>
        <span class="page"></span>
      </footer>
    </main>
  </body>
</html>
```

### 확인

스크린샷 도구가 있으면 장마다 1920×1080으로 열어 넘침 · 잘림 · 대비를 본다. 넘치는 장은 글을 줄이거나 장을 나눈다.

## 게시 · 수정 게시

```bash
abox publish <작업 폴더>/proposal-deck --title "AICC 구축 제안서" --desc "고객 발표용 제안 21장" --json
abox publish <작업 폴더>/proposal-deck --url <URL> --json
```

- 결과의 `data.type`이 `slides`이고 `data.slideCount`가 장 수다.
- 결과 보고에 "슬라이드 · N장", URL, "PDF는 화면의 [인쇄]에서 저장해요"를 넣는다(그 밖은 SKILL.md '결과 보고').
- 장 더하기는 새 파일 + `deck.json`에 넣기, 빼기는 `deck.json`에서 지우기(남은 파일은 장이 아니다), 순서 · 제목은 `deck.json`을 고친다. 모두 같은 폴더를 `--url`로 수정 게시한다. 파일이 하나라도 바뀌면 새 버전이다.
- `deck.css`를 고쳤으면 모든 장의 모양이 바뀔 수 있다고 결과 보고에 알린다.
- `DECK_INVALID`(exit 2)면 `error.issues[]`의 `path`(`deck.json` 안 JSON Pointer: `/slides/6/file` = 7번째 장의 `file`)와 `message`대로 고쳐 다시 게시한다. 같은 오류(`path` · `code`가 같은 이슈)가 3번 반복되면 더 고치지 말고 멈춘 뒤 `issues`와 폴더 위치를 사용자에게 알린다.

### 한 장만 고치기

1. 게시한 폴더가 작업 폴더에 없으면(다른 세션) 다시 만들지 않고 `abox pull <URL> --out <작업 폴더>/<새 폴더> --json`으로 받는다.
2. 고칠 파일을 정한다. 슬라이드 링크면 `#slide=` 뒤가 그 파일이다(`…#slide=effect.html` → `effect.html`, `%`로 인코딩된 글자는 풀어 읽는다). "7번 장"이면 `deck.json`의 `slides` 7번째(1부터 센다)의 `file`이다.
3. 그 파일만 고친다. 다른 장 · `deck.json` · `deck.css`는 그대로 둔다.
4. 같은 폴더를 `abox publish <폴더> --url <URL> --json`으로 수정 게시한다. 바뀐 파일만 올라간다. 장 HTML 하나나 `deck.json` 하나만 넘기면 exit 2다.
5. 결과 보고에 고친 장의 슬라이드 링크를 넣는다.

## 슬라이드 링크

- 슬라이드 링크는 `data.url` 뒤에 `#slide=<file>`을 붙인 주소다(`https://artifactbox.app/a/Xk2vR9pQ7mL4tB8nC1dE5f#slide=effect.html`). 파일 이름 규칙을 지켰으면 `file`을 그대로 붙인다(하위 폴더의 `/`도 그대로).
- 같은 파일이면 수정 게시 · 순서 바꾸기 뒤에도 같은 장이다. 번호 링크(`#slide=7`)는 없다.
- 사용자가 특정 장을 말하면 그 장의 링크를 준다. 슬라이드 링크로 열어도 모든 장이 보인다(장별 공개는 없다). 한 장만 보이려면 그 장을 HTML로 따로 게시한다.
- 모음 안의 덱은 항목 주소 뒤에 붙인다(`/a/<모음>/<번호>#slide=<file>`).
- 질문 · 보고의 첨부도 `<URL>#slide=<file>`로 한 장을 가리킨다(`abox-ask` · `abox-report`).

## HTML로 게시한 덱 옮기기

이미 HTML로 게시한 발표 자료를 슬라이드로 바꿔 달라고 하면 아래대로 한다. 종류는 수정 게시로 바꿀 수 없어 슬라이드는 새 URL이 된다.

1. 새 URL이 된다고 먼저 알린다. `abox info <옛 URL> --json`에서 옛 것이 공개 중(`visibility`)이거나 모음(`includedIn`) · 질문 첨부(`attachedTo`)에 들어 있으면 그것도 알린다.
2. `abox pull <옛 URL> --out <작업 폴더>/<새 폴더> --json`으로 받는다.
3. 받은 폴더는 읽기만 하고, 덱은 `<작업 폴더>/<덱 이름>/` 새 폴더에 만든다. 옛 `index.html` · 옛 덱 HTML은 새 폴더에 넣지 않는다(넣으면 HTML로 게시되거나 옛 덱이 파일로 함께 올라간다). 장마다 파일로 나누고(이름은 내용으로) 공용 스타일을 `deck.css`로 모은다. 넘김 런타임 · 키 처리 · 인쇄 CSS · 다크 모드 · 박아 넣은 쪽 번호를 걷어 내고, 장마다 1920×1080에 맞춘다(넘치면 장을 나눈다).
4. `deck.json`을 쓰고 새 폴더를 `--url` 없이 새로 게시한다. 비공개로 시작하고, 결과의 `data.type`이 `slides`인지 본다.
5. 옛 아티팩트는 그대로 둔다. 사용자가 원할 때만 옛 URL을 「새 주소로 옮겼어요」와 새 링크(`target="_blank"`) 한 장짜리 HTML로 수정 게시하거나 `abox unshare`로 비공개로 바꾼다.
