# 캔버스

캔버스는 화면(보드) 여러 장을 한 URL에 나란히 놓고 확대 · 이동하며 보는 아티팩트다. 보드 HTML들과 배치를 적은 `canvas.json`이 든 폴더로 게시한다. 공개 · 버전 · 되돌리기는 캔버스 전체 단위다.

## 언제 쓰나

- 함께 비교 · 리뷰할 화면이 여러 장이면(시안, 변형, 화면 흐름, 폭별 반응형) 사용자가 "캔버스"라고 하지 않아도 캔버스로 만든다.
- 화면 시안은 1장이어도 캔버스로 시작한다. 나중에 같은 URL에 보드를 더할 수 있다.
- 한 페이지 리포트 · 문서 · 대시보드는 HTML · Markdown이다.
- 캔버스와 HTML · Markdown 사이는 수정 게시로 바꿀 수 없다. 바꿔 달라고 하면 새로 게시해야 한다(새 URL)고 알린다.

## 폴더

```
<작업 폴더>/model-console/
  canvas.json
  shared.css
  console/list.html
  console/add.html
  mobile/home.html
```

- 최상위에 `canvas.json`을 두고 `index.html`은 두지 않는다. `index.html`이 있으면 HTML로 게시되고 경고 `CANVAS_MANIFEST_IGNORED`가 붙는다.
- 최상위에 다른 용도의 `deck.json`이 있으면 슬라이드 매니페스트와 겹쳐 exit 2다(「canvas.json과 deck.json 중 하나만 둬요」). 그 파일의 이름을 바꾼다.
- 폴더 이름은 캔버스마다 다르게 짓는다. 서버가 같은 위치(`DUPLICATE_LOCATION`)를 폴더 이름으로 판정한다.
- 보드 파일 경로는 영문 소문자 · 숫자 · `-`로 짓는다(`console/model-list.html`). 경로가 보드 링크의 주소가 된다.

## canvas.json

```json
{
  "canvas": 1,
  "sections": [
    {
      "title": "운영 콘솔",
      "boards": [
        { "file": "console/list.html", "title": "모델 목록", "width": 1280, "height": 820 },
        { "file": "console/add.html", "title": "모델 더하기", "height": 1300 }
      ]
    },
    {
      "title": "모바일",
      "boards": [{ "file": "mobile/home.html", "title": "모바일 홈", "width": 390, "height": 844 }]
    }
  ]
}
```

| 칸                  | 규칙                                                             |
| ------------------- | ---------------------------------------------------------------- |
| `canvas`            | 늘 `1`                                                           |
| `sections`          | 1~20개. 위에서 아래로 한 줄씩 놓인다                             |
| `sections[].title`  | 선택, 1~100자                                                    |
| `sections[].boards` | 1개 이상. 왼쪽에서 오른쪽으로 놓인다. 섹션당 6개 이하로 나눈다   |
| `boards[].file`     | 폴더 안 `.html` 파일의 상대 경로. 한 파일은 한 보드만            |
| `boards[].title`    | 필수, 1~100자. 리뷰어가 그 화면을 부르는 이름                    |
| `boards[].width`    | 240~3840, 기본 1280. 보드의 화면 폭이다(390이면 모바일 레이아웃) |
| `boards[].height`   | 160~4320, 기본 800. 더 긴 내용은 보드 안에서 스크롤된다          |

- 다른 칸은 넣지 않는다(거부된다). 아티팩트 제목 · 설명은 `--title` · `--desc`로 준다.
- 보드는 캔버스 전체에 30개까지다. 더 많으면 캔버스를 나눠 게시한다.
- `canvas.json`은 64KB까지다.

## 보드 만들기

- 보드마다 그 파일만 열어도 보이는 HTML이다. SKILL.md 'HTML'의 결과물 규칙과 디자인 순서를 그대로 따른다.
- `width`를 화면 폭으로 보고 만든다. 내용을 `height` 안에 맞추거나 `height`를 내용에 맞춘다.
- 적어도 첫 화면(위쪽 주요 내용)은 `height` 안에 들어가게 한다. 캔버스에서는 높이 아래가 잘리고 보드 안 스크롤바가 보일 수 있다.
- 공용 CSS · JS는 상대 경로 파일(`../shared.css`)로 둔다. 보드마다 CDN Tailwind를 불러오지 않는다.
- 보드 안 링크는 그 보드 틀 안에서만 이동한다. 다른 보드로 가지 않는다.

## 게시 · 수정 게시

```bash
abox publish <작업 폴더>/model-console --title "모델 콘솔 시안" --desc "운영 콘솔 3장과 모바일 2장" --json
abox publish <작업 폴더>/model-console --url <URL> --json
```

- 결과의 `data.type`이 `canvas`이고 `data.boardCount`가 보드 수다. 결과 보고에 "캔버스 · 보드 N개"를 넣는다.
- 보드 하나만 고쳐도 같은 폴더 전체를 `--url`로 수정 게시한다. 바뀐 파일만 올라간다. 보드 HTML 하나나 `canvas.json` 하나만 넘기면 exit 2다.
- 게시한 폴더가 작업 폴더에 없으면(다른 세션) 다시 만들지 않고 `abox pull`로 받아 고친다(SKILL.md '수정 게시').
- 보드를 더하거나 빼거나 순서 · 제목을 바꿀 때도 같은 폴더의 `canvas.json`을 고쳐 수정 게시한다. 파일이 하나라도 바뀌면 새 버전이다.
- `CANVAS_INVALID`(exit 2)면 `error.issues[]`의 `path`(`canvas.json` 안 JSON Pointer: `/sections/0/boards/1/file` = 첫 섹션 둘째 보드의 `file`)와 `message`대로 고쳐 다시 게시한다. 같은 오류(`path` · `code`가 같은 이슈)가 3번 반복되면 더 고치지 말고 멈춘 뒤 `issues`와 폴더 위치를 사용자에게 알린다.

## 보드 링크

- 보드 링크는 `data.url` 뒤에 `#board=<file>`을 붙인 주소다(`https://artifactbox.app/a/Xk2vR9pQ7mL4tB8nC1dE5f#board=console/list.html`). 경로 규칙을 지켰으면 `file`을 그대로 붙인다.
- 사용자가 특정 보드를 말하면 그 보드 링크를 준다. 같은 파일이면 수정 게시 뒤에도 같은 링크다.
- 보드 링크로 열어도 캔버스 전체가 보이고, 그 보드로 옮겨져 열린다. 보드별 공개는 없다. 필요할 때 "한 화면만 보이려면 그 보드를 HTML로 따로 게시한다"고 알린다.
