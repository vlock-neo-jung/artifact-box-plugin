# abox `--json` 출력

모든 명령은 stdout에 JSON 한 줄을 쓴다. 키는 camelCase다.

- 성공: `{"ok":true,"data":{…},"warnings":[{"code","message",…}]}` (`warnings`는 있을 때만)
- 실패: `{"ok":false,"error":{"code","message","hint"?,…}}` + 0이 아닌 exit code

## publish

새 게시·새 버전(`created`·`new_version`):

```json
{
  "ok": true,
  "data": {
    "result": "created",
    "url": "https://art.example.com/a/Xk2vR9pQ7mL4tB8nC1dE5f",
    "artifactId": "Xk2vR9pQ7mL4tB8nC1dE5f",
    "version": 1,
    "type": "html",
    "visibility": { "mode": "private", "expiresAt": null },
    "project": { "id": "Pq4wT8nV2mK6rX9bL3cH7j", "name": "workspace-artifact" },
    "workspace": { "id": "Wn3kQ7vR2xL9mT4pB8cJ6d", "kind": "personal", "name": "개인" }
  },
  "warnings": [
    {
      "code": "DUPLICATE_LOCATION",
      "message": "…",
      "existingUrl": "https://art.example.com/a/Rt7nB2xL5cV9mK4pW8qJ6d"
    }
  ]
}
```

`new_version`이면 `previousVersion`이 있다. 라이브 아티팩트(`--live`로 만든 것)에 내용이 다른 게시를 하면 `result`가 `updated`이고 `version`은 늘 1이다(`previousVersion` 없음, 같은 모양). 변경 없음·정보만 바뀜(`unchanged`·`metadata_updated`):

```json
{
  "ok": true,
  "data": {
    "result": "unchanged",
    "url": "…",
    "artifactId": "…",
    "version": 3,
    "type": "html",
    "activeShare": true,
    "missing": [],
    "project": { "id": "Pq4wT8nV2mK6rX9bL3cH7j", "name": "workspace-artifact" },
    "workspace": { "id": "Wn3kQ7vR2xL9mT4pB8cJ6d", "kind": "personal", "name": "개인" }
  }
}
```

`type`(`html`|`md`|`canvas`|`slides`)은 모든 결과에 있고, 캔버스면 `boardCount`(보드 수), 슬라이드면 `slideCount`(장 수)도 있다.

`project`는 모든 결과에 있고, 이 게시 뒤 아티팩트가 들어 있는 프로젝트다(옮겼으면 옮긴 곳, `--project` 없는 새 게시면 `미분류`).

`workspace:{id,kind,name}`도 모든 결과에 있고, 아티팩트가 있는 워크스페이스다. `kind`가 `team`이면 비공개여도 그 팀 멤버 모두가 볼 수 있다. 새 게시는 `--project`의 워크스페이스(`--workspace`도 주면 둘이 같아야 한다), `--workspace`, 기본 워크스페이스 순으로 정한 곳에, 수정 게시는 아티팩트가 있던 워크스페이스에 그대로 들어간다.

필수 정보 누락(exit 2):

```json
{
  "ok": false,
  "error": {
    "code": "MISSING_REQUIRED",
    "message": "필수 정보가 빠졌어요",
    "missing": ["title", "desc"]
  }
}
```

새 게시는 `--title`·`--desc`만 필수다. 게시는 프로젝트를 만들지 않는다. `--project`가 받는 값과 실패:

| 경우                                                      | exit | `error`                                                                 |
| --------------------------------------------------------- | ---- | ----------------------------------------------------------------------- |
| id(22자) 모양이 아닌 값(이름 등). 요청 전에 거절          | 2    | `INVALID_INPUT` 「프로젝트는 id로 지정해요」(`hint`: `abox project ls`) |
| 없거나 지운 프로젝트, 멤버가 아닌 워크스페이스의 프로젝트 | 4    | `NOT_FOUND` 「프로젝트가 없어요」                                       |
| `--workspace`와 다른 워크스페이스의 프로젝트(새 게시)     | 2    | `INVALID_INPUT`                                                         |
| 다른 워크스페이스의 프로젝트로 옮기기(`--url`)            | 2    | `INVALID_INPUT`                                                         |
| 프로젝트를 이름으로 보내는 이전 CLI                       | 2    | `INVALID_INPUT`(`hint`: `abox update`)                                  |

비밀 파일(exit 2): `{"ok":false,"error":{"code":"SECRET_FILE","message":"…","files":["site/.env.production"]}}`

캔버스의 `canvas.json`이 규칙과 맞지 않음(exit 2, 올리기 전에 검사한다):

```json
{
  "ok": false,
  "error": {
    "code": "CANVAS_INVALID",
    "message": "canvas.json이 규칙에 맞지 않아요(문제 2개)",
    "issueCount": 2,
    "issues": [
      {
        "path": "/sections/0/boards/1/file",
        "code": "file-not-found",
        "message": "게시하는 폴더에 없는 파일이에요"
      },
      {
        "path": "/sections/0/boards/1/title",
        "code": "min-length",
        "message": "제목이 비어 있어요"
      }
    ]
  }
}
```

- `path`는 `canvas.json` 안 JSON Pointer다(`""`는 파일 전체). `issues`는 50개까지 담기고 전체 수는 `issueCount`다.
- `canvas.json`이 64KB를 넘으면 `LIMIT_EXCEEDED`(exit 5)다.
- `canvas.json` 파일 하나를 넘기면(`--url`과 함께여도) `INVALID_INPUT`(hint 「캔버스는 폴더째 게시해요」), 캔버스 URL에 HTML 파일 하나를 넘기면 `INVALID_INPUT`(hint 「캔버스는 폴더째 수정 게시해요」)이다.

슬라이드의 `deck.json`이 규칙과 맞지 않음(exit 2, 올리기 전에 검사한다):

```json
{
  "ok": false,
  "error": {
    "code": "DECK_INVALID",
    "message": "deck.json이 규칙에 맞지 않아요(문제 2개)",
    "issueCount": 2,
    "issues": [
      {
        "path": "/slides/6/file",
        "code": "file-not-found",
        "message": "게시하는 폴더에 없는 파일이에요"
      },
      {
        "path": "/slides/6/title",
        "code": "min-length",
        "message": "제목이 비어 있어요"
      }
    ]
  }
}
```

- `path`는 `deck.json` 안 JSON Pointer다(`/slides/6` = 7번째 장, `""`는 파일 전체). `issues`는 50개까지 담기고 전체 수는 `issueCount`다. 51장 이상이면 `/slides`의 `max-items`다.
- `deck.json`이 64KB를 넘으면 `LIMIT_EXCEEDED`(exit 5)다.
- `deck.json` 파일 하나를 넘기면(`--url`과 함께여도) `INVALID_INPUT`(hint 「슬라이드는 폴더째 게시해요」), 슬라이드 URL에 HTML 파일 하나를 넘기면 `INVALID_INPUT`(hint 「슬라이드는 폴더째 수정 게시해요」)이다. `--json`에는 hint만 있고, 사람용 출력에만 `deck.json`이 있는 위 폴더로 다시 게시하는 명령이 붙는다.
- 폴더 최상위에 `canvas.json`과 `deck.json`이 함께 있고 `index.html`이 없으면 `INVALID_INPUT`(「canvas.json과 deck.json 중 하나만 둬요」)이다.

## publish — 템플릿 데이터

`publish <data.json> --template X …`(새 게시)와 `publish <data.json> --url U`(수정 게시)의 응답이다.

```json
{
  "ok": true,
  "data": {
    "result": "created",
    "url": "https://art.example.com/a/Pq4wT8nV2mK6rX9bL3cH7j",
    "artifactId": "Pq4wT8nV2mK6rX9bL3cH7j",
    "version": 1,
    "type": "html",
    "visibility": { "mode": "private", "expiresAt": null },
    "template": { "name": "weekly-report", "title": "주간 보고", "version": 3 },
    "project": { "id": "Pq4wT8nV2mK6rX9bL3cH7j", "name": "workspace-artifact" },
    "workspace": { "id": "Wn3kQ7vR2xL9mT4pB8cJ6d", "kind": "personal", "name": "개인" }
  }
}
```

- `project` · `workspace`는 파일 게시와 같다(이 게시 뒤 아티팩트의 프로젝트 · 워크스페이스). 새 게시의 템플릿은 그 워크스페이스에서 찾는다.
- `result`(라이브의 `updated` 포함)와 `previousVersion`(`new_version`일 때)은 파일 게시와 같다. `unchanged`·`metadata_updated`에도 `type`·`visibility`·`template`이 있고, `missing`·`activeShare`는 없다.
- `template`은 이 아티팩트가 고정된 템플릿 버전이다(`version`).
- 경고: `DUPLICATE_LOCATION`(새 게시)·`PUBLIC_UPDATE`(수정 게시).

검증 실패(exit 2):

```json
{
  "ok": false,
  "error": {
    "code": "DATA_INVALID",
    "message": "데이터가 템플릿과 맞지 않아요 (2곳)",
    "issueCount": 2,
    "issues": [
      {
        "path": "/risks/1/level",
        "code": "enum-error",
        "message": "high · medium · low 중 하나여야 해요 · 지금 urgent"
      },
      { "path": "/week", "code": "required-property-error", "message": "꼭 있어야 해요" }
    ],
    "template": { "name": "weekly-report", "title": "주간 보고", "version": 3 }
  }
}
```

- `issues`는 50개까지 담기고 전체 수는 `issueCount`다.
- `path`는 데이터 안 JSON Pointer다. `""`는 데이터 전체이고, 키의 `/`·`~`는 `~1`·`~0`으로 바뀐다. `required-property-error`의 `path`는 빠진 필드 자리다.
- `message`는 고칠 내용을 해요체로 말한다(`스키마에 없는 필드예요`·`문자열이어야 해요 · 지금 숫자`·`날짜(YYYY-MM-DD)여야 해요 · 지금 10/05`·`비워 둘 수 없어요`·`80자 이하여야 해요 · 지금 112자`). 그 밖의 오류는 `값이 스키마와 맞지 않아요`이고, 이때는 `schema`의 그 자리 규칙을 읽는다.
- `쓸 수 없는 필드 이름이에요`(`reserved-property-error`): `constructor`·`toString`·`__proto__` 같은 Object.prototype 이름은 어느 깊이에서도 키로 쓸 수 없다.

데이터 게시의 다른 오류:

| code (exit)          | 경우                                                             | 추가 필드                     |
| -------------------- | ---------------------------------------------------------------- | ----------------------------- |
| `INVALID_INPUT` (2)  | 데이터 파일의 JSON 문법 오류(요청 전)                            | `file`·`line`·`column`·`hint` |
| `INVALID_INPUT` (2)  | 데이터가 JSON 객체(`{…}`)가 아님                                 | `file`                        |
| `INVALID_INPUT` (2)  | 일반 아티팩트를 데이터로, 템플릿으로 만든 아티팩트를 파일로 고침 | `hint`(맞는 명령)             |
| `INVALID_INPUT` (2)  | `.json`인데 `--template`·`--url`이 없음(지원하지 않는 종류)      | `hint`                        |
| `INVALID_INPUT` (2)  | `--template`과 `--url`을 함께 씀                                 | `hint`                        |
| `INVALID_INPUT` (2)  | 수정 게시 요청의 `live` 값이 아티팩트와 다름(파일 게시도 같음)   | `hint`                        |
| `NOT_FOUND` (4)      | 없는 템플릿으로 새 게시                                          | `hint`(`abox template ls…`)   |
| `LIMIT_EXCEEDED` (5) | 데이터가 1MB를 넘음                                              | `file`·`limit`·`actual`       |

## pull

`abox pull <URL> [--version N] --out <경로>` — HTML·Markdown·캔버스·슬라이드는 게시한 원본 파일을 새 폴더로 받는다:

```json
{
  "ok": true,
  "data": {
    "path": "/tmp/artifact-box/model-console",
    "publishPath": "/tmp/artifact-box/model-console",
    "version": 3,
    "type": "canvas",
    "entryPath": "canvas.json",
    "files": ["canvas.json", "console/list.html", "shared.css"]
  }
}
```

- `path`는 받은 폴더의 절대 경로이고, `files`는 그 안의 상대 경로다.
- `publishPath`는 고친 뒤 `abox publish … --url`에 넘길 경로다. 캔버스 · 슬라이드(`deck.json` 포함)와 진입 파일이 `index.html`인 HTML은 받은 폴더이고(파일을 더해도 함께 올라간다), 파일 하나로 게시한 HTML(`index.html` 아닌 이름)·Markdown은 그 파일이다.
- `version`은 받은 버전(`--version N`, 기본 현재)이다.
- 받은 파일이 목록과 다르면(받는 동안 라이브 아티팩트가 바뀜) `UNEXPECTED_RESPONSE`이고 아무것도 쓰지 않는다. 다시 받는다.
- 받는 동안 `--out` 자리에 무엇이 생기면 `INVALID_INPUT`이고 아무것도 쓰지 않는다.

템플릿으로 만든 아티팩트는 데이터를 파일로 받는다:

```json
{
  "ok": true,
  "data": {
    "path": "/tmp/artifact-box/weekly-report-2026-w41.json",
    "version": 4,
    "sourceName": "weekly-report-2026-w41.json",
    "template": { "name": "weekly-report", "title": "주간 보고", "version": 3 },
    "data": { "title": "플랫폼팀 41주차 보고", "week": "2026-10-05", "done": ["…"] },
    "schema": { "type": "object", "required": ["title", "week", "done"], "properties": {} },
    "guide": "# 작성 지침\n…"
  }
}
```

- `path`는 쓴 파일의 절대 경로다. `sourceName`은 게시 때의 데이터 파일 이름이고, 수정 게시해도 이 이름이 유지된다.
- `version`은 받은 아티팩트 버전(`--version N`, 기본 현재)이다. `schema`·`guide`(없으면 `null`)는 고정된 템플릿 버전의 것이다.
  오류(파일·데이터 모두):

- `--out` 없음 → 요청 전에 `MISSING_REQUIRED`(`missing: ["out"]`). 아무것도 쓰지 않는다.
- `--out` 자리에 파일이나 폴더가 이미 있음(빈 폴더도) → 요청 전에 `INVALID_INPUT`(`hint`: `--out`). 덮어쓰지 않는다.
- 없는 버전 → `NOT_FOUND`(exit 4)

## share · unshare

```json
{
  "ok": true,
  "data": { "visibility": { "mode": "link", "expiresAt": "2026-10-15T05:00:00.000Z" } },
  "warnings": [{ "code": "URL_HOLDERS_CAN_VIEW", "reshare": false, "message": "…" }]
}
```

`expiresAt: null`은 만료 없음.

## info

`data`: `id` · `url` · `workspace:{id,kind,name}` · `project:{id,name}` · `dir`(없으면 `null`) · `repo` · `title` · `description` · `tags`(정렬된 문자열 배열) · `type`(`html`|`md`|`canvas`|`slides`|`question`|`collection`) · `currentVersion` · `visibility` · `views:{count,lastViewedAt}` · `template` · `live` · `entryPath` · `sourceName` · `versions:[{number,createdAt,restoredFrom,…}]`(최신 먼저) · `boardCount`(캔버스만, 현재 버전의 보드 수) · `slideCount`(슬라이드만, 현재 버전의 장 수)

`type`이 `question`이면 질문 아티팩트(`abox-ask` 스킬)다. `questions:{total,waiting}`(질문 수 · 답 대기 수)가 있고 `versions`는 비어 있다. 파일이 없어서 수정 게시 · 되돌리기 · `pull`은 거부되고, 질문 아티팩트는 질문의 첨부로 쓸 수 없다.

`type`이 `collection`이면 모음이다. `items`(목차 순서, 아래 'collection'의 항목)가 있고 `versions`는 비어 있다. 수정 게시 · 되돌리기 · `pull`은 거부된다. 모음 항목 주소(`/a/<모음>/<번호>`)를 주면 그 모음의 정보다.

`includedIn:{count,shared:[{id,url,title}]}` — 이 아티팩트가 든 모음 수와, 그중 지금 링크 공개 중인 모음(비공개 아티팩트여도 그 모음의 받는 사람에게 보인다). 든 모음이 없으면 없다.

`attachedTo:{questions}` — 이 워크스페이스에서 이 아티팩트를 첨부한 질문 수. 첨부한 질문이 없으면 없다.

`live` — `true`면 라이브 아티팩트다. 버전은 v1 하나뿐이고 게시할 때마다 그 내용이 바뀐다(`updatedAt`이 마지막 갱신). 되돌리기는 없다.

`visibility:{mode,expiresAt,everShared}` — 만료는 읽을 때 판정한다. `expiresAt`이 지금보다 과거면 `mode`가 `link`로 남아 있어도 만료(받는 사람에게는 비공개와 같음)다.

`template` — 템플릿으로 만든 아티팩트면 `{name,title,version}`(`version`은 고정된 템플릿 버전), 아니면 `null`. 템플릿으로 만든 아티팩트는 `type`이 `html`이고, `sourceName`은 게시한 데이터 파일 이름이며, 버전의 `fileCount`·`totalBytes`는 데이터 기준(1개·데이터 바이트)이다.

## ls

`data`: `workspace:{id,kind,name}`(조회한 워크스페이스) · `items:[info와 같은 요약(versions·entryPath·sourceName·workspace·includedIn 없음, 모음은 items 대신 collection:{itemCount})]` · `projects:[{id,name,description,count,updatedAt,matched?}]` · `dirs:[{dir,count,repo?}]` · `tags:[{tag,count,preset}]` · `nextCursor?`(있으면 목록이 잘림 — CLI에 다음 페이지 옵션은 없음) · `search?`(`-q`로 찾았을 때만)

- `projects`는 그 워크스페이스의 프로젝트(빈 것 포함, 최근 바뀐 순)로, `--repo` 말고 다른 필터와 상관없이 전체다(`--repo`면 그 저장소의 아티팩트가 있는 것만). `count`도 그 프로젝트의 전체 개수다.
- `--tag`를 주면 프로젝트마다 `matched`(그 태그를 모두 가진 수, 0일 수 있음)가 붙는다. `--repo`와 `--tag`만 따른다.
- `dirs`(디렉토리가 있는 아티팩트만 센다)와 `tags`는 `--repo`와 `--project`를 따른다(그 프로젝트 안의 것). 다른 필터와는 상관없다.
- `tags`는 미리 만든 태그(`preset: true`, 등록 순서)가 먼저, 나머지는 많이 쓴 순이다.
- `--project <id>`만 주면 그 프로젝트의 워크스페이스를 조회한다(`data.workspace`). 없거나 볼 수 없는 프로젝트면 `NOT_FOUND`(exit 4)다. `--workspace`와 함께 주면 그 워크스페이스에서 찾고, 그곳의 프로젝트가 아니면 `items`가 비어 있다.

`--template <이름>`이면 그 템플릿으로 만든 것만 나온다. 없는 템플릿 이름이면 `items`가 비어 있다(오류 아님).

`-q`(단어가 남는 검색어)면 `items`는 정확히 맞는 문서만이고(관련도 순, 200개까지, `nextCursor` 없음) `search`가 붙는다. 읽는 법은 SKILL.md의 '검색 결과 읽기'다.

```json
"search": {
  "words": ["예산", "한도"],
  "searched": 19,
  "exact": 0,
  "outside": { "exact": 0, "hasSimilar": true },
  "semantic": true,
  "similar": []
}
```

- `words`: 검색어를 나눈 단어(앞 `#`을 뗌). `searched`: 필터만 건 아티팩트 수. `exact`: 정확히 맞는 문서의 전체 수(200보다 많으면 `items`는 앞 200개)
- `outside`: 필터 밖의 정확히 맞는 수(`exact`)와 비슷한 문서가 있는지(`hasSimilar`). 필터가 없으면 `{exact:0, hasSimilar:false}`
- `semantic`: false면 비슷한 문서를 찾지 못했다. `similar`가 비고 `outside.hasSimilar`도 false다(「없음」이 아니라 「찾지 못함」)
- `similar`: 비슷한 문서. `items`와 같은 요약, 가까운 순 5개까지, 점수 없음. 필터를 따른다

## edit

`data`: info와 같은 상세(바뀐 뒤의 값, `project`·`tags` 포함). 바꿀 항목이 없거나 값이 규칙에 맞지 않으면 exit 2(`INVALID_INPUT`). `--project`의 실패는 publish와 같다(옮기기는 같은 워크스페이스의 프로젝트로만).

## project

`project ls` — 그 워크스페이스의 모든 프로젝트(빈 것 포함), 최근 바뀐 순(`updatedAt` = 그 안 아티팩트의 최근 수정 시각, 빈 프로젝트는 프로젝트 자신의 수정 시각). `description`은 없으면 `""`이고 줄바꿈이 있을 수 있다:

```json
{
  "ok": true,
  "data": {
    "items": [
      {
        "id": "Pq4wT8nV2mK6rX9bL3cH7j",
        "name": "leda-platform",
        "description": "레다 플랫폼 문서와 화면 시안",
        "count": 12,
        "updatedAt": "2026-10-08T00:40:00.000Z"
      }
    ]
  }
}
```

- `project info <id>` — `data`: `project ls`의 항목에 `workspace:{id,kind,name}`(프로젝트가 있는 워크스페이스)를 더한 것. 없거나 볼 수 없는 id면 `NOT_FOUND`(exit 4, `hint`).
- `project add <이름> [--desc S]` — `data`: 만든 프로젝트 `{id,name,description,count,updatedAt}`(`count`는 0). 레포 연결 줄은 `Artifact Box 프로젝트: <name> (id: <id>)`다. 그 워크스페이스에 같은 이름(대소문자·띄어쓰기 무시)이 있으면 `INVALID_INPUT`(exit 2, `hint`).
- `project edit <id> [--name N] [--desc S]` — `data`: 바뀐 뒤의 `{id,name,description,count,updatedAt}`. `--desc ""`는 설명을 지운다. 둘 다 없으면 요청 전에 exit 2. 다른 프로젝트와 같은 이름이면 `INVALID_INPUT`(exit 2), 없는 id면 `NOT_FOUND`(exit 4).
- `project rm <id>` — `data:{deleted:true}`. 아티팩트가 있으면 지우지 않는다:

```json
{
  "ok": false,
  "error": {
    "code": "PROJECT_NOT_EMPTY",
    "message": "아티팩트 3개가 있어서 지울 수 없어요. 옮기거나 지운 뒤 다시 해 주세요",
    "artifactCount": 3,
    "hint": "아티팩트 목록: abox ls --project Pq4wT8nV2mK6rX9bL3cH7j"
  }
}
```

`project info`·`edit`·`rm`의 `<id>`가 id 모양이 아니면 요청 전에 `INVALID_INPUT`(exit 2)이다. 이름 · 설명 규칙(이름 1~60자, 설명 500자까지)에 맞지 않아도 요청 전에 exit 2다.

## tag

`tag ls` — 미리 만든 태그, 등록 순서(`count` = 그 태그가 붙은 아티팩트 수):

```json
{
  "ok": true,
  "data": {
    "items": [
      {
        "tag": "조사",
        "description": "기술·라이브러리·사례 조사, 다른 코드베이스 분석, 비교 리서치",
        "count": 3
      },
      {
        "tag": "요구사항",
        "description": "무엇을 왜 만드는지: PRD·기능 명세·시나리오·인수 조건",
        "count": 0
      }
    ]
  }
}
```

- `tag add` · `tag edit` — `data:{tag,description,count}`. `edit --rename`이면 `renamedArtifacts`(태그를 바꾼 아티팩트 수)가 더 있다. 이미 있는 태그를 추가하거나 다른 미리 만든 태그 이름으로 바꾸면 `INVALID_INPUT`, 목록에 없는 태그를 고치면 `NOT_FOUND`.
- `tag add`에 `--desc`가 없으면 `MISSING_REQUIRED`(`missing: ["desc"]`). 설명은 1~200자다.
- `tag rm` — `data:{deleted:true}`. 목록에서만 빠지고 붙어 있던 태그는 그대로다.

## collection

`collection new`:

```json
{
  "ok": true,
  "data": {
    "url": "https://art.example.com/a/Cm4xT8nV2kL6rB9pW3qJ7d",
    "artifactId": "Cm4xT8nV2kL6rB9pW3qJ7d",
    "visibility": { "mode": "private", "expiresAt": null },
    "project": { "id": "Pq4wT8nV2mK6rX9bL3cH7j", "name": "workspace-artifact" },
    "workspace": { "id": "Wn3kQ7vR2xL9mT4pB8cJ6d", "kind": "personal", "name": "개인" },
    "items": [
      {
        "item": 1,
        "type": "md",
        "template": false,
        "title": "결제 재시도 정의서",
        "url": "https://art.example.com/a/Cm4xT8nV2kL6rB9pW3qJ7d/1",
        "artifact": {
          "id": "Xk2vR9pQ7mL4tB8nC1dE5f",
          "url": "https://art.example.com/a/Xk2vR9pQ7mL4tB8nC1dE5f",
          "visibility": { "mode": "private", "expiresAt": null }
        }
      }
    ],
    "skipped": [{ "path": "/items/3", "item": 1 }]
  }
}
```

- `items[]` — 목차 순서. `url`은 모음 안의 항목 주소(CLI가 더한다, 모음 안에 보일 글에서 항목을 가리킬 때 쓴다), `item`은 그 번호(뺀 번호는 다시 쓰지 않는다), `artifact`는 원본(그 URL · 공개 상태). 질문 아티팩트면 `artifact.questions:{total,waiting}`도 있다
- `skipped[]` — 이미 든 것(또는 두 번 준 것)이라 건너뛴 주소. `path`는 요청 안 위치(`/items/3` = 넷째로 준 주소), `item`은 그것이 든 항목 번호

`collection add` · `rm`: `data:{url,items,added,skipped,removed}` — `url`은 모음 URL(모음 자리에 항목 주소를 줬어도), `items`는 바뀐 뒤 목차(`new`와 같은 모양), `added` · `removed`는 이번에 넣고 뺀 항목 번호. 링크 공개 중인 모음에 넣으면 `PUBLIC_UPDATE`.

주소가 맞지 않으면 아무것도 바꾸지 않고 `COLLECTION_INVALID`(exit 2)다. 형식(바깥 주소 · 해시 · 항목 주소 · 다른 모음의 항목 주소)은 CLI가 요청 전에 거부한다:

```json
{
  "ok": false,
  "error": {
    "code": "COLLECTION_INVALID",
    "message": "모음에 쓸 수 없는 주소가 있어요(1개). 아무것도 바꾸지 않았어요",
    "issueCount": 1,
    "issues": [
      {
        "path": "/items/2",
        "code": "collection-in-collection",
        "message": "모음 안에 모음은 넣을 수 없어요"
      }
    ]
  }
}
```

`issues[].code`: `not-artifact-url`(이 서버 주소가 아님) · `item-not-found`(없음 · 다른 워크스페이스) · `collection-in-collection` · `item-hash`(`#board=` · `#slide=` · `#q3`가 붙음) · `collection-item-url`(넣을 주소가 항목 주소) · `other-collection-item`(뺄 주소가 다른 모음의 항목 주소). 넣으면 50개를 넘으면 `LIMIT_EXCEEDED`(exit 5)이고 하나도 넣지 않는다.

## rollback · rm · open · dashboard

- rollback: `data:{version(새 번호),restoredFrom}` + 공개 중이면 `PUBLIC_UPDATE`. 라이브 아티팩트는 `INVALID_INPUT`(exit 2, `hint`)
- rm: `data:{deleted:true}`
- open · dashboard: `data:{opened,url}` (`opened: false`면 브라우저를 열지 못함). 모음 항목 주소를 주면 모음 안의 그 아티팩트를 연다

## 서버 연결 실패 (exit 6)

메시지·`hint`는 CLI가 서버 설정에 맞춰 채운다. 그대로 전한다.

```json
{
  "ok": false,
  "error": {
    "code": "NETWORK_ERROR",
    "message": "서버에 연결하지 못했어요: https://artifactbox.app",
    "hint": "인터넷 연결을 확인하고 잠시 뒤 다시 시도해 주세요."
  }
}
```

## 로그인 필요 (exit 3)

로그인한 적이 없거나, 토큰이 만료(90일 미사용)됐거나 웹에서 끊긴 기기다. `email`은 이 기기가 마지막으로 로그인한 계정이고 모르면 없다. 토큰은 어디에도 나오지 않는다.

```json
{
  "ok": false,
  "error": {
    "code": "AUTH_REQUIRED",
    "message": "로그인이 만료됐어요",
    "server": "https://artifactbox.app",
    "email": "user@example.com"
  }
}
```

`ABOX_TOKEN` 환경 변수의 토큰이 거절되면 `message`가 그 변수를 가리키고 `hint`에 바꿀 방법이 있다.

## workspace · whoami

`workspace ls` — 내 워크스페이스(개인 먼저, 팀은 이름순). `default`가 `true`인 것이 기본 워크스페이스다:

```json
{
  "ok": true,
  "data": {
    "items": [
      {
        "id": "Wn3kQ7vR2xL9mT4pB8cJ6d",
        "kind": "personal",
        "name": "개인",
        "role": "owner",
        "memberCount": 1,
        "artifactCount": 42,
        "default": true
      },
      {
        "id": "Ac7mP2qW9xT4kL8vR3nB6d",
        "kind": "team",
        "name": "Acme 팀",
        "role": "member",
        "memberCount": 4,
        "artifactCount": 23,
        "default": false
      }
    ]
  }
}
```

- `workspace use <이름|id>` — `data:{workspace:{id,kind,name,role},changed}`. 팀이면 경고 `TEAM_DEFAULT_WORKSPACE`
- `whoami` — `data`: `id` · `email` · `name` · `device`(기기 이름) · `server` · `tokenSource`(`file`|`env`) · `workspace:{id,kind,name,role}`(기본 워크스페이스, 더 쓸 수 없으면 `null` + 경고 `DEFAULT_WORKSPACE_UNAVAILABLE`)

`--workspace`를 찾지 못하면:

| 경우                                                     | exit | `error`                                                                                                |
| -------------------------------------------------------- | ---- | ------------------------------------------------------------------------------------------------------ |
| 같은 이름의 팀이 여럿                                    | 2    | `INVALID_INPUT`, `candidates:[{id,kind,name,role}]` — 사용자에게 고르게 하고 `--workspace <id>`로 다시 |
| 내가 속한 워크스페이스에 없는 이름 · id                  | 4    | `NOT_FOUND`("워크스페이스를 찾을 수 없어요: …")                                                        |
| 저장된 기본 워크스페이스에서 내보내졌거나 그 팀이 지워짐 | 4    | `NOT_FOUND`, `defaultWorkspace:{id,name}` — `abox workspace use 개인` 안내를 전한다                    |
| `--url`과 함께 줌                                        | 2    | `INVALID_INPUT` — 수정 게시는 아티팩트의 워크스페이스에 한다                                           |

## 경고 코드

| code                            | 뜻                                                                                                 |
| ------------------------------- | -------------------------------------------------------------------------------------------------- |
| `DUPLICATE_LOCATION`            | 같은 디렉토리(없으면 같은 프로젝트)·파일명의 아티팩트가 이미 있음(`existingUrl`)                   |
| `PUBLIC_UPDATE`                 | 공개 중인 아티팩트가 바뀌어(모음은 넣어서) 받는 사람에게 바로 보임                                 |
| `URL_HOLDERS_CAN_VIEW`          | 공개로 바뀌어 URL을 아는 사람이 열 수 있음(`reshare`)                                              |
| `COLLECTION_SHOWS_PRIVATE`      | (모음 `share`) 비공개 아티팩트 `count`개도 받는 사람이 모음 안에서 봄(`message`에 제목)            |
| `ROOT_RELATIVE_PATHS`           | HTML에 `/`로 시작하는 경로가 있음(`files`)                                                         |
| `CANVAS_MANIFEST_IGNORED`       | (CLI) `index.html`이 있는 폴더라 HTML로 게시했고, 캔버스 매니페스트 같은 `canvas.json`은 읽지 않음 |
| `DECK_MANIFEST_IGNORED`         | (CLI) `index.html`이 있는 폴더라 HTML로 게시했고, 덱 매니페스트 같은 `deck.json`은 읽지 않음       |
| `TEAM_DEFAULT_WORKSPACE`        | (`workspace use`) 기본이 팀이 되어 `--workspace` 없는 게시를 팀 멤버 모두가 봄                     |
| `DEFAULT_WORKSPACE_UNAVAILABLE` | (`workspace ls` · `whoami`) 저장된 기본 워크스페이스를 더 쓸 수 없음                               |
