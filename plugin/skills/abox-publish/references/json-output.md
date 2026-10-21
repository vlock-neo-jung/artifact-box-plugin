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
    "activeShare": true,
    "missing": [],
    "project": { "id": "Pq4wT8nV2mK6rX9bL3cH7j", "name": "workspace-artifact" },
    "workspace": { "id": "Wn3kQ7vR2xL9mT4pB8cJ6d", "kind": "personal", "name": "개인" }
  }
}
```

`project`는 모든 결과에 있고, 이 게시 뒤 아티팩트가 들어 있는 프로젝트다(옮겼으면 옮긴 곳, 기존 프로젝트면 그 프로젝트의 표기 그대로).

`workspace:{id,kind,name}`도 모든 결과에 있고, 아티팩트가 있는 워크스페이스다. `kind`가 `team`이면 비공개여도 그 팀 멤버 모두가 볼 수 있다. 새 게시는 `--workspace`나 기본 워크스페이스에, 수정 게시는 아티팩트가 있던 워크스페이스에 그대로 들어간다.

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

새 게시는 `--title`·`--desc`만 필수다. `--project` 이름이 규칙(60자 이하, 빈 이름·제어 문자 불가)에 맞지 않으면 요청 전에 `INVALID_INPUT`(exit 2)이다.

새 프로젝트를 만든 게시(새 게시·`--url`과 `--project`로 옮긴 수정 게시)에는 경고가 붙는다.

```json
{
  "code": "PROJECT_CREATED",
  "message": "새 프로젝트를 만들었어요: 회의자료",
  "project": { "id": "Pq4wT8nV2mK6rX9bL3cH7j", "name": "회의자료" }
}
```

비밀 파일(exit 2): `{"ok":false,"error":{"code":"SECRET_FILE","message":"…","files":["site/.env.production"]}}`

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
- 경고: `DUPLICATE_LOCATION`(새 게시)·`PUBLIC_UPDATE`(수정 게시)·`PROJECT_CREATED`(새 프로젝트를 만들었을 때).

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

`abox pull <URL> [--version N] --out <파일>` — 템플릿으로 만든 아티팩트의 데이터를 파일로 받는다.

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
- 오류
  - `--out` 없음 → 요청 전에 `MISSING_REQUIRED`(`missing: ["out"]`). 아무것도 쓰지 않는다.
  - 쓸 파일이 이미 있음 → `INVALID_INPUT`(`hint`: `--out`). 덮어쓰지 않는다.
  - 템플릿으로 만든 아티팩트가 아님 → `INVALID_INPUT`
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

`data`: `id` · `url` · `workspace:{id,kind,name}` · `project:{id,name}` · `dir`(없으면 `null`) · `repo` · `title` · `description` · `tags`(정렬된 문자열 배열) · `type`(`html`|`md`) · `currentVersion` · `visibility` · `views:{count,lastViewedAt}` · `template` · `live` · `entryPath` · `sourceName` · `versions:[{number,createdAt,restoredFrom,…}]`(최신 먼저)

`live` — `true`면 라이브 아티팩트다. 버전은 v1 하나뿐이고 게시할 때마다 그 내용이 바뀐다(`updatedAt`이 마지막 갱신). 되돌리기는 없다.

`visibility:{mode,expiresAt,everShared}` — 만료는 읽을 때 판정한다. `expiresAt`이 지금보다 과거면 `mode`가 `link`로 남아 있어도 만료(받는 사람에게는 비공개와 같음)다.

`template` — 템플릿으로 만든 아티팩트면 `{name,title,version}`(`version`은 고정된 템플릿 버전), 아니면 `null`. 템플릿으로 만든 아티팩트는 `type`이 `html`이고, `sourceName`은 게시한 데이터 파일 이름이며, 버전의 `fileCount`·`totalBytes`는 데이터 기준(1개·데이터 바이트)이다.

## ls

`data`: `workspace:{id,kind,name}`(조회한 워크스페이스) · `items:[info와 같은 요약(versions·entryPath·sourceName·workspace 없음)]` · `projects:[{id,name,count,updatedAt,matched?}]` · `dirs:[{dir,count,repo?}]` · `tags:[{tag,count,preset}]` · `nextCursor?`(있으면 목록이 잘림 — CLI에 다음 페이지 옵션은 없음)

- `projects`는 아티팩트가 있는 프로젝트(최근 바뀐 순)로, `--repo` 말고 다른 필터와 상관없이 전체다. `count`도 그 프로젝트의 전체 개수다.
- `--tag`를 주면 프로젝트마다 `matched`(그 태그를 모두 가진 수, 0일 수 있음)가 붙는다. `--repo`와 `--tag`만 따른다.
- `dirs`(디렉토리가 있는 아티팩트만 센다)와 `tags`는 `--repo`와 `--project`를 따른다(그 프로젝트 안의 것). 다른 필터와는 상관없다.
- `tags`는 미리 만든 태그(`preset: true`, 등록 순서)가 먼저, 나머지는 많이 쓴 순이다.
- `--project <이름>`은 대소문자·띄어쓰기가 달라도 같은 프로젝트를 찾는다. 없는 이름이면 `items`가 비어 있다(오류 아님).

`--template <이름>`이면 그 템플릿으로 만든 것만 나온다. 없는 템플릿 이름이면 `items`가 비어 있다(오류 아님).

## edit

`data`: info와 같은 상세(바뀐 뒤의 값, `project`·`tags` 포함). `--project`로 새 프로젝트를 만들었으면 `PROJECT_CREATED` 경고가 붙는다. 바꿀 항목이 없거나 값이 규칙에 맞지 않으면 exit 2(`INVALID_INPUT`).

## project

`project ls` — 아티팩트가 있는 프로젝트, 최근 바뀐 순(`updatedAt` = 그 안 아티팩트의 최근 수정 시각):

```json
{
  "ok": true,
  "data": {
    "items": [
      {
        "id": "Pq4wT8nV2mK6rX9bL3cH7j",
        "name": "leda-platform",
        "count": 12,
        "updatedAt": "2026-10-08T00:40:00.000Z"
      }
    ]
  }
}
```

`project rename <이름> <새 이름>` — `data`: 바뀐 뒤의 `{id,name,count,updatedAt}`. 없는 이름이면 `NOT_FOUND`(exit 4, `hint`), 다른 프로젝트와 같은 이름이면 `INVALID_INPUT`(exit 2, `hint`).

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

## rollback · rm · open · dashboard

- rollback: `data:{version(새 번호),restoredFrom}` + 공개 중이면 `PUBLIC_UPDATE`. 라이브 아티팩트는 `INVALID_INPUT`(exit 2, `hint`)
- rm: `data:{deleted:true}`
- open · dashboard: `data:{opened,url}` (`opened: false`면 브라우저를 열지 못함)

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

| code                            | 뜻                                                                               |
| ------------------------------- | -------------------------------------------------------------------------------- |
| `DUPLICATE_LOCATION`            | 같은 디렉토리(없으면 같은 프로젝트)·파일명의 아티팩트가 이미 있음(`existingUrl`) |
| `PUBLIC_UPDATE`                 | 공개 중인 아티팩트가 바뀌어 받는 사람에게 바로 보임                              |
| `URL_HOLDERS_CAN_VIEW`          | 공개로 바뀌어 URL을 아는 사람이 열 수 있음(`reshare`)                            |
| `ROOT_RELATIVE_PATHS`           | HTML에 `/`로 시작하는 경로가 있음(`files`)                                       |
| `PROJECT_CREATED`               | 게시·이동이 새 프로젝트를 만듦(`project:{id,name}`)                              |
| `TEAM_DEFAULT_WORKSPACE`        | (`workspace use`) 기본이 팀이 되어 `--workspace` 없는 게시를 팀 멤버 모두가 봄   |
| `DEFAULT_WORKSPACE_UNAVAILABLE` | (`workspace ls` · `whoami`) 저장된 기본 워크스페이스를 더 쓸 수 없음             |
