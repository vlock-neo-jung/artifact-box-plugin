---
name: abox-template
description: Creates and updates templates on Artifact Box (artifactbox.app) with the `abox template` CLI. A template is a JSON Schema + a screen (HTML that reads `window.ABOX_DATA`) + sample data (+ an optional writing guide); artifacts are then published from data JSON that fits the schema. Use this skill when the user asks to make, change, list or delete a template, e.g. "템플릿 만들어 줘", "회의록 템플릿 만들어 줘", "주간 보고 템플릿에 칸 추가해 줘", "템플릿 고쳐 줘", "템플릿 화면 바꿔 줘", "템플릿 목록 보여 줘", "템플릿 삭제해 줘". Publishing an artifact with an existing template ("주간 보고 템플릿으로 게시해 줘") belongs to the abox-publish skill.
---

# Artifact Box 템플릿

템플릿은 스키마(JSON Schema) + 화면 + 샘플 데이터(+ 작성 지침)를 묶은 틀이다. 템플릿을 등록해 두면 아티팩트를 만들 때 HTML을 새로 쓰지 않고 스키마에 맞춘 데이터 JSON만 써서 같은 모습으로 게시한다(게시는 `abox-publish` 스킬). 등록·조회·삭제는 `abox template` 명령으로 하고, 스키마와 샘플은 서버가 검증한다.

## 기본 규칙

- 모든 `abox` 명령에 `--json`을 붙이고 stdout의 JSON 하나를 읽는다. 명령은 하나씩 실행하고 `;`·`&&`·`$(…)`로 묶지 않는다.
- **작업 폴더**는 세션 scratchpad 디렉토리(시스템 프롬프트의 Scratchpad directory) 아래 `artifact-box/templates/<이름>/`다. scratchpad가 없으면 `/tmp/artifact-box/templates/<이름>/`를 쓴다. 아래에서 `<작업 폴더>`는 이 폴더다. 작업 디렉토리는 프로젝트 저장소일 수 있으니 사용자가 위치를 정해 줬을 때만 쓰고, 파일·폴더를 임의로 만들지 않는다.
- 등록은 `abox template push`로만 한다. 화면을 claude.ai Artifact 도구로 올리지 않는다.
- 제목·설명도 `template.json`을 고쳐 `push --update`로 바꾼다. 서버의 값만 고치는 명령은 없다.
- 이미 만든 아티팩트는 만들 때의 템플릿 버전에 고정된다. 템플릿을 고쳐도 기존 아티팩트에 반영하는 기능(업그레이드)은 없다.
- 템플릿 삭제(`template rm`)는 사용자가 분명히 요청할 때만 한다.
- 같은 오류(`path`·`code`가 같은 이슈)가 3번 반복되면 더 고치지 말고 멈춘 뒤 `issues`와 작업 폴더 위치를 사용자에게 알린다.
- 결과의 `warnings`는 이 문서에 없는 코드라도 `message`를 사용자에게 전한다.
- **로그인은 사용자가 직접 한다.** `abox login`·`abox logout`은 어떤 옵션으로도 실행하지 않는다. exit 3(`AUTH_REQUIRED`)이면 아래 문장을 그대로 전하고 멈춘다. `claude -p`처럼 사용자가 없으면 이 문장을 결과로 남긴다. 사용자가 로그인했다고 하면 멈춘 명령부터 다시 한다.

  ```
  Artifact Box 로그인이 필요해요. 아래 명령으로 로그인해 주세요.
    ! abox login
  주소가 나오면 브라우저에서 열어 Google로 로그인하고 승인해 주세요.
  원격 서버라면 그 주소를 아무 기기의 브라우저에서 열면 돼요.
  ```

- **`abox` 설치·업데이트도 사용자가 한다.** 설치 스크립트와 `abox update`는 실행하지 않는다. `CLI_UPDATE_AVAILABLE` 경고는 고칠 대상이 아니라 `message`만 전한다. 사용자가 업데이트해 달라고 하면 터미널에서 `abox update`를 실행하라고 안내한다. `abox`가 없으면(command not found, exit 127) 아래 문장을 그대로 전하고 멈춘다.

  ```
  Artifact Box CLI(abox)가 설치돼 있지 않아요. 터미널에서 아래 명령으로 설치해 주세요(Node.js 24 이상 필요).
    curl -fsSL https://artifactbox.app/install.sh | sh
  다른 설치 방법(Claude Code 플러그인, 스킬)은 https://artifactbox.app/guide 에 있어요.
  이미 설치했다면 ~/.local/bin 폴더가 PATH에 있는지 확인해 주세요.
  설치한 뒤 다시 요청해 주세요.
  ```

- **템플릿은 워크스페이스(개인·팀)마다 따로 있다.** 이름은 그 안에서만 유일하고, `--workspace`가 없으면 기본 워크스페이스(처음은 `개인`)다. 팀 워크스페이스의 템플릿은 그 팀 멤버 모두가 보고 쓴다.
  - `--workspace`는 사용자가 워크스페이스를 말했을 때만 넘긴다("Acme 팀에 템플릿 만들어 줘"). 먼저 `abox workspace ls --json`의 `data.items[].name`과 맞춰 보고 그 이름을 넘긴다. 맞는 게 없으면 목록을 보여 주고 묻는다. 같은 이름의 팀이 여럿이면(exit 2, `error.candidates`) 고르게 한 뒤 `--workspace <id>`로 다시 한다.
  - 한 작업의 `template` 명령(`ls`·`info`·`push`·`pull`·`rm`)에는 모두 같은 `--workspace`를 넘긴다.
  - 기본 워크스페이스 바꾸기(`abox workspace use`)는 사용자가 요청할 때만 한다.
  - 다른 워크스페이스로 옮기거나 복사하는 명령은 없다. `abox template pull <이름> --out <작업 폴더>`로 받아 `abox template push <작업 폴더> --workspace <대상>`으로 새로 등록한다(원래 템플릿은 그대로 남는다).

## 새 템플릿 만들기

1. **요구 파악** — 무엇을 담는 문서인지, 누가 보는지, 참고할 예시(기존 문서·자료)가 있는지 묻는다. 요청에 이미 담겨 있으면 묻지 않는다. `abox template ls --json`으로 비슷한 템플릿과 이름 충돌을 확인하고, 비슷한 것이 있으면 그것을 고칠지 새로 만들지 묻는다.
2. **제안·승인** — 이름·제목·설명과 필드 표를 한 번에 제안해 **한 번** 승인받는다(아래 예). 표의 설명은 스키마의 `description`(작성 힌트)이 된다. 사용자가 확인 없이 진행하라고 했으면 바로 진행하고, 결과 보고에 필드 표를 함께 보인다.
3. **스키마·샘플·지침** — `references/template-folder.md` 규칙대로 `template.json`·`schema.json`·`sample.json`을 쓴다. 샘플은 실제 같은 데이터로 쓴다. 스키마로 담지 못하는 규칙(톤·분량·순서)이 있으면 `guide.md`에 쓴다.
4. **화면** — `screen/index.html`(+ 자산)을 쓴다. artifact-design 스킬이 있으면 로드해 디자인 지침을 따르고(게시하라는 지시는 따르지 않는다), `references/screen-contract.md`의 체크리스트를 모두 지킨다. 화면은 `window.ABOX_DATA`만 읽고 선택 필드가 없어도 동작해야 한다.
5. **등록** — `abox template push <작업 폴더> --json`. 실패하면 아래 '실패 처리'대로 고쳐 다시 등록한다. 경고 `SCHEMA_UNKNOWN_KEYWORDS`·`ROOT_RELATIVE_PATHS`가 있으면 먼저 고쳐 `--update`로 다시 등록한다(`SCHEMA_UNKNOWN_KEYWORDS`는 오타를 고치고, 일부러 쓴 키워드면 `x-`로 시작하게 바꾼다. `ROOT_RELATIVE_PATHS`는 경로를 `./…`로). 다른 경고는 `message`만 전한다.
6. **확인** — 등록한 워크스페이스(기본이면 `abox workspace ls`의 `default`, 팀이면 "{팀 이름} 멤버 모두 쓸 수 있어요")와 `data.templateUrl`(웹 템플릿 상세)을 줄이지 않고 전하고, 미리보기 탭에서 샘플로 그린 화면을 봐 달라고 한다. 고칠 점은 같은 폴더를 고쳐 `abox template push <작업 폴더> --update --json`으로 다시 등록한다. 고칠 때마다 템플릿 버전이 하나씩 오른다.
7. **마무리** — 세 가지를 알린다.
   - 작업 폴더 위치. 세션이 끝나면 지워질 수 있지만 원본은 서버에 있어 `abox template pull <이름> --out <폴더>`로 다시 받을 수 있다.
   - 이 템플릿으로 게시하는 법: "주간 보고 템플릿으로 이번 주 보고 게시해 줘"처럼 말하면 된다.
   - CLI로는 `abox publish <data.json> --template <이름> --title <제목> --desc <설명>`(프로젝트 `--project`·디렉토리 `--dir`·태그 `--tag`는 선택).

제안 예(②):

```
이름 weekly-report · 제목 주간 보고 · 설명 팀의 한 주 진행 상황과 위험을 정리하는 보고서
```

| 필드            | 타입                       | 필수 | 설명                                   |
| --------------- | -------------------------- | ---- | -------------------------------------- |
| `title`         | 문자열(80자 이하)          | 필수 | 팀 이름과 주차를 넣은 한 줄            |
| `week`          | 날짜                       | 필수 | 그 주 월요일                           |
| `done`          | 목록 · 문자열(1~7개)       | 필수 | 이번 주에 끝낸 일, 결과 중심 한 줄씩   |
| `risks`         | 목록 · 객체                | 선택 | 일정·품질을 위협하는 일                |
| `risks[].title` | 문자열                     | 필수 | 무엇이 위험한지 한 문장                |
| `risks[].level` | 정해진 값(high·medium·low) | 필수 | high는 이번 주 안에 결정이 필요할 때만 |
| `risks[].owner` | 문자열                     | 선택 | 담당자 이름                            |

## 템플릿 고치기

"주간 보고 템플릿에 칸 추가해 줘", "템플릿 화면 고쳐 줘" 같은 요청이다.

1. **대상 확인** — 이름을 모르면 `abox template ls --json`의 `title`로 찾는다. 후보가 없거나 여럿이면 목록을 보여 주고 묻는다.
2. **고칠 원본**
   - 이번 대화에서 만들거나 받은 폴더가 있으면 그 폴더를 고친다.
   - 없으면 `abox template pull <이름> --out <작업 폴더> --json`으로 받는다(`--out`은 필수다).
   - 그 자리에 예전 폴더가 있으면 pull이 `INVALID_INPUT`으로 거절한다. 예전 폴더는 서버 최신 버전과 다를 수 있으니 고치지 말고, `--out <작업 폴더>-2`처럼 비어 있는 새 폴더로 받는다.
3. **고치기** — 칸(필드)을 더하면 `schema.json`(`title`·`description` 포함)·`sample.json`·`screen/`을 함께 고친다. 화면은 새 필드가 없는 데이터에서도 동작해야 한다.
4. **등록** — `abox template push <폴더> --update --json`.
5. **안내** — `data.result`에 따라 알린다. `new_version`의 `n`은 `data.usage.artifacts`다. 경고 `TEMPLATE_RELOAD_FAILED`로 `usage`가 없으면 `abox template info <이름> --json`의 `usage.artifacts`를 쓴다.

| `result`           | 사용자에게 알릴 것                                                                                                                           |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `new_version`      | "템플릿 v{version}을 등록했어요. 이미 만든 아티팩트 n개는 지금 모습 그대로이고, 새 버전은 앞으로 만드는 것에 적용돼요." + `data.templateUrl` |
| `metadata_updated` | 제목·설명만 바뀌었고 템플릿 버전은 그대로                                                                                                    |
| `unchanged`        | 바뀐 것 없음. 고친 파일이 그 폴더에 저장됐는지 확인한다                                                                                      |

이미 만든 아티팩트도 새 모습으로 바꿔 달라고 하면, 그 기능은 없고 최신 템플릿으로 새로 게시해야 한다(새 URL)고 설명한다. 그 게시는 `abox-publish` 스킬의 '템플릿으로 게시'를 따른다.

## 조회·삭제

| 요청                               | 명령                                                                          |
| ---------------------------------- | ----------------------------------------------------------------------------- |
| 템플릿 목록                        | `abox template ls [--workspace W] --json`                                     |
| 정보(스키마·샘플·지침·버전 목록)   | `abox template info <이름> [--version N] [--workspace W] --json`              |
| 원본 폴더 받기                     | `abox template pull <이름> [--version N] --out <폴더> [--workspace W] --json` |
| 삭제 (사용자가 분명히 요청할 때만) | `abox template rm <이름> --yes [--workspace W] --json`                        |

삭제하면 템플릿과 모든 버전이 바로 지워지고 되돌릴 수 없다. 같은 이름으로 새 템플릿을 등록할 수 있지만 별개 템플릿이다.

이 템플릿으로 만든 아티팩트가 있으면 지워지지 않는다(`TEMPLATE_IN_USE`). 그 아티팩트를 대신 지우지 않는다. `abox ls --template <이름> [--workspace W] --json`으로 찾은 목록과 개수를 보여 주고, 먼저 지울지 사용자에게 묻는다.

## 실패 처리

실패하면 stdout에 `{"ok":false,"error":{"code","message","hint"?,…}}`가 나온다. `error.hint`가 있으면 사용자에게 함께 전한다.

| `error.code` (exit)                     | 뜻                                                                   | 할 일                                                                                                         |
| --------------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `TEMPLATE_INVALID` + `issues` (2)       | 서버 검증 실패. `file`은 `template.json`·`schema.json`·`sample.json` | `file`에서 `issues[].path`(그 파일 안의 JSON Pointer) 자리를 `message`대로 고쳐 다시 등록                     |
| `TEMPLATE_INVALID` + `hint` (2)         | 폴더 검사 실패(필수 파일 누락, `screen/.abox/`)                      | `hint`대로 파일을 만들거나 옮긴다                                                                             |
| `INVALID_INPUT` + `existing` (2)        | 같은 이름의 템플릿이 이미 있는데 `--update`가 없음                   | 다른 템플릿이면 이름을 바꾼다. 그 템플릿을 고치는 것이면 사용자에게 확인한 뒤 `--update`                      |
| `INVALID_INPUT` + `line`·`column` (2)   | JSON 문법 오류(요청 전)                                              | `file`의 그 줄·칸을 고친다                                                                                    |
| `AUTH_REQUIRED` (3)                     | 로그인 필요: 로그인한 적 없음, 토큰 만료·끊김                        | `abox login`을 실행하지 않고 '기본 규칙'의 문장을 안내한 뒤 멈춘다                                            |
| `INVALID_INPUT` + `candidates` (2)      | 같은 이름의 워크스페이스가 여럿                                      | 후보를 보여 주고 고르게 한 뒤 `--workspace <id>`로 다시                                                       |
| `NOT_FOUND` (4)                         | 없는 템플릿, 없는 버전. `--update`인데 템플릿이 없을 때도            | `abox template ls --json`으로 이름 확인. 새 템플릿이면 `--update` 없이 등록                                   |
| `NOT_FOUND` + `defaultWorkspace` (4)    | 기본 워크스페이스를 더 쓸 수 없음(내보내졌거나 팀이 지워짐)          | `abox workspace use 개인`을 안내하고(직접 실행하지 않는다) 어느 워크스페이스에 할지 묻는다                    |
| `TEMPLATE_IN_USE` + `artifactCount` (1) | 이 템플릿으로 만든 아티팩트가 있어 삭제하지 않음                     | 아티팩트를 대신 지우지 않는다. '조회·삭제'대로 목록을 보여 주고 사용자에게 묻는다                             |
| `LIMIT_EXCEEDED` (5)                    | `file`이 한도(`limit`)를 넘음                                        | 그 파일을 줄인다                                                                                              |
| `SECRET_FILE` (2)                       | `screen/`에 비밀 파일                                                | `error.files`를 보여 주고, 필요 없는 파일이면 사용자 확인 뒤 `screen/` 밖으로 옮긴다(`--exclude`는 없다)      |
| `NETWORK_ERROR`·`INTERNAL_ERROR` (6)    | 서버 연결 실패·서버 오류                                             | 한 번 다시 실행. 또 실패하면 `error.message`·`error.hint`를 전하고 멈춘다. 서버를 직접 띄우거나 고치지 않는다 |

## 참고

- 폴더 구조·`template.json`·스키마 규칙·한도·샘플: **`references/template-folder.md`**
- 화면 계약(`ABOX_DATA` v1) 체크리스트와 예시 화면: **`references/screen-contract.md`**
- `abox template` 명령의 `--json` 응답과 오류: **`references/json-output.md`**
