# abox template `--json` 출력

모든 명령은 stdout에 JSON 한 줄을 쓴다. 키는 camelCase다.

- 성공: `{"ok":true,"data":{…},"warnings":[{"code","message",…}]}` (`warnings`는 있을 때만)
- 실패: `{"ok":false,"error":{"code","message","hint"?,…}}` + 0이 아닌 exit code

## template ls

```json
{
  "ok": true,
  "data": {
    "items": [
      {
        "name": "weekly-report",
        "title": "주간 보고",
        "description": "팀의 한 주 진행 상황과 위험을 정리하는 보고서",
        "currentVersion": 3,
        "fieldCount": 7,
        "updatedAt": "2026-10-11T09:40:00.000Z",
        "usage": { "artifacts": 12 }
      }
    ]
  }
}
```

- 최근 등록순이다.
- `usage.artifacts`는 이 템플릿으로 만든 아티팩트 수다(템플릿 버전과 상관없이 모두).

## template info

`data`는 ls 항목에 아래 키를 더한 것이다. `--version N`이면 `version`·`createdAt`·`fieldCount`·`schema`·`sample`·`guide`·`files`가 그 버전의 값이다.

| 키          | 값                                                                                                                                                        |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `version`   | 보여 주는 템플릿 버전(기본은 최신 = `currentVersion`)                                                                                                     |
| `createdAt` | 그 버전을 등록한 시각                                                                                                                                     |
| `versions`  | `[{number, createdAt}]`, 최신 먼저                                                                                                                        |
| `schema`    | JSON Schema 원문(객체)                                                                                                                                    |
| `sample`    | 샘플 데이터(객체)                                                                                                                                         |
| `guide`     | 작성 지침 Markdown 문자열, 없으면 `null`                                                                                                                  |
| `files`     | `[{path, size, sha256}]`. 서버에 저장된 경로다: 화면 파일은 `screen/`을 뺀 경로, 정보 파일은 `.abox/schema.json`·`.abox/data.json`(샘플)·`.abox/guide.md` |

## template push

`created`·`new_version`:

```json
{
  "ok": true,
  "data": {
    "result": "new_version",
    "name": "weekly-report",
    "version": 4,
    "previousVersion": 3,
    "templateUrl": "https://art.example.com/w/Wn3kQ7vR2xL9mT4pB8cJ6d/templates/weekly-report",
    "fieldCount": 8,
    "usage": { "artifacts": 12 }
  },
  "warnings": [
    {
      "code": "SCHEMA_UNKNOWN_KEYWORDS",
      "message": "스키마에 모르는 키워드가 있어요. 오타가 아닌지 확인하세요: /properties/title/maxlength",
      "paths": ["/properties/title/maxlength"]
    }
  ]
}
```

- `created`는 `version: 1`이고 `previousVersion`이 없다.
- `templateUrl`은 웹 템플릿 상세 주소다(`/w/<워크스페이스 id>/templates/<이름>`, 만료되지 않음).
- `fieldCount`·`usage`는 등록 뒤 템플릿을 다시 읽은 값이다. 다시 읽지 못하면 두 키가 빠지고 경고 `TEMPLATE_RELOAD_FAILED`가 붙는다. 등록은 끝난 상태다.

`unchanged`·`metadata_updated`(업로드 없이 끝남):

```json
{
  "ok": true,
  "data": {
    "missing": [],
    "result": "metadata_updated",
    "name": "weekly-report",
    "version": 3,
    "templateUrl": "https://art.example.com/w/Wn3kQ7vR2xL9mT4pB8cJ6d/templates/weekly-report"
  }
}
```

| `result`           | 뜻                                                  |
| ------------------ | --------------------------------------------------- |
| `created`          | 새 템플릿 v1                                        |
| `new_version`      | 화면·스키마·샘플·지침 중 하나 이상이 바뀌어 새 버전 |
| `metadata_updated` | 제목·설명만 바뀜, 템플릿 버전 그대로                |
| `unchanged`        | 바뀐 것 없음                                        |

## template pull

```json
{
  "ok": true,
  "data": {
    "path": "/tmp/artifact-box/templates/weekly-report",
    "name": "weekly-report",
    "title": "주간 보고",
    "description": "팀의 한 주 진행 상황과 위험을 정리하는 보고서",
    "version": 3,
    "files": ["template.json", "sample.json", "guide.md", "schema.json", "screen/index.html"]
  }
}
```

- `path`는 받은 폴더의 절대 경로, `files`는 그 안에 쓴 상대 경로다.
- `--out`은 필수다. 없으면 요청 전에 `MISSING_REQUIRED`(`missing: ["out"]`)이고 아무것도 쓰지 않는다.
- 대상 폴더가 비어 있지 않으면 아무것도 쓰지 않고 `INVALID_INPUT`(`hint`: `--out`으로 다른 폴더)이다.

## template rm

- `--yes`와 함께: `{"ok":true,"data":{"deleted":true}}`. 템플릿과 모든 버전이 지워진다.
- 이 템플릿으로 만든 아티팩트가 있으면 아무것도 지우지 않고 `TEMPLATE_IN_USE`(exit 1)다. `artifactCount`는 그 아티팩트 수다.
- `--yes` 없이: `CONFIRMATION_REQUIRED`(exit 2).

## 경고 코드

| code                      | 뜻                                                                                             |
| ------------------------- | ---------------------------------------------------------------------------------------------- |
| `SCHEMA_UNKNOWN_KEYWORDS` | 스키마에 JSON Schema 2020-12에 없는 키워드가 있음(`x-` 제외). `paths`는 스키마 안 JSON Pointer |
| `ROOT_RELATIVE_PATHS`     | 화면 HTML에 `/`로 시작하는 경로가 있음. `files`는 `screen/…` 경로                              |
| `TEMPLATE_RELOAD_FAILED`  | 등록은 끝났지만 결과를 다시 읽지 못함(`fieldCount`·`usage` 없음)                               |

## 검증 오류 (`TEMPLATE_INVALID`, exit 2)

서버 검증:

```json
{
  "ok": false,
  "error": {
    "code": "TEMPLATE_INVALID",
    "message": "샘플 데이터가 스키마와 맞지 않아요 (2곳)",
    "file": "sample.json",
    "issueCount": 2,
    "issues": [
      {
        "path": "/week",
        "code": "format-date-error",
        "message": "날짜(YYYY-MM-DD)여야 해요 · 지금 10/05"
      },
      {
        "path": "/risks/1/level",
        "code": "enum-error",
        "message": "high · medium · low 중 하나여야 해요 · 지금 urgent"
      }
    ]
  }
}
```

| `message`                                  | `file`           | 경우                                                         |
| ------------------------------------------ | ---------------- | ------------------------------------------------------------ |
| `template.json 값을 고쳐야 해요 (n곳)`     | `template.json`  | 이름·제목·설명이 규칙에 맞지 않음                            |
| `스키마가 규칙과 맞지 않아요 (n곳)`        | `schema.json`    | 스키마 규칙 위반                                             |
| `샘플 데이터가 스키마와 맞지 않아요 (n곳)` | `sample.json`    | 샘플이 스키마를 통과하지 못함                                |
| `필수 파일이 없어요: <파일>`               | 그 파일          | 폴더 검사(요청 전). `hint`에 필요한 파일 목록, `issues` 없음 |
| `screen/.abox/는 쓸 수 없는 경로예요`      | `screen/.abox/…` | 예약 경로 사용(요청 전), `issues` 없음                       |

- `issues`는 50개까지 담기고 전체 수는 `issueCount`다.
- `path`는 `file` 안의 JSON Pointer다. `""`는 파일 전체, `/risks/1/level`은 `risks` 둘째 항목의 `level`이다. 키의 `/`·`~`는 `~1`·`~0`으로 바뀐다.
- `required-property-error`의 `path`는 빠진 필드 자리다(`/week`).
- `message`는 해요체 한국어다. 자주 나는 데이터 오류의 예:

| `code`                           | `message` 예                                         |
| -------------------------------- | ---------------------------------------------------- |
| `required-property-error`        | `꼭 있어야 해요`                                     |
| `no-additional-properties-error` | `스키마에 없는 필드예요`                             |
| `type-error`                     | `문자열이어야 해요 · 지금 숫자`                      |
| `enum-error`                     | `high · medium · low 중 하나여야 해요 · 지금 urgent` |
| `format-date-error`              | `날짜(YYYY-MM-DD)여야 해요 · 지금 10/05`             |
| `min-length-one-error`           | `비워 둘 수 없어요`                                  |
| `max-length-error`               | `80자 이하여야 해요 · 지금 112자`                    |
| `min-items-one-error`            | `하나 이상 있어야 해요`                              |
| `reserved-property-error`        | `쓸 수 없는 필드 이름이에요`                         |

- 표에 없는 데이터 오류의 `message`는 `값이 스키마와 맞지 않아요`다. 이때는 스키마의 그 자리 규칙을 읽고 고친다.
- 스키마 규칙 위반의 `code`: `unsupported-draft-error`·`root-type-error`·`remote-ref-error`·`reserved-property-error`·`schema-error`(짧은 원인 포함). 규칙은 `template-folder.md`.

## 그 밖의 오류

| code (exit)                 | 경우                                                                  | 추가 필드                                                |
| --------------------------- | --------------------------------------------------------------------- | -------------------------------------------------------- |
| `INVALID_INPUT` (2)         | 같은 이름의 템플릿이 있는데 `--update` 없음                           | `existing:{name,version,updatedAt,artifactCount}`·`hint` |
| `INVALID_INPUT` (2)         | `template.json`·`schema.json`·`sample.json`의 JSON 문법 오류(요청 전) | `file`·`line`·`column`·`hint`                            |
| `INVALID_INPUT` (2)         | 템플릿 이름이 규칙에 맞지 않음(요청 전)                               | `hint`                                                   |
| `INVALID_INPUT` (2)         | `template pull` 대상 폴더가 비어 있지 않음                            | `hint`                                                   |
| `NOT_FOUND` (4)             | 없는 템플릿(`--update`인데 없을 때 포함)                              | `hint`(`abox template ls…`)                              |
| `NOT_FOUND` (4)             | 없는 템플릿 버전(`--version N`)                                       | —                                                        |
| `LIMIT_EXCEEDED` (5)        | 스키마·지침 64KB, 샘플 1MB, 화면 파일 10MB·전체 50MB·2,000개 초과     | `file`·`limit`·`actual`(정보 파일일 때)                  |
| `SECRET_FILE` (2)           | `screen/`에 비밀 파일                                                 | `files`                                                  |
| `CONFIRMATION_REQUIRED` (2) | `template rm`에 `--yes` 없음                                          | `hint`                                                   |
| `TEMPLATE_IN_USE` (1)       | `template rm`인데 이 템플릿으로 만든 아티팩트가 있음                  | `artifactCount`·`hint`                                   |

이름 충돌 예:

```json
{
  "ok": false,
  "error": {
    "code": "INVALID_INPUT",
    "message": "이미 있는 템플릿이에요: weekly-report",
    "existing": {
      "name": "weekly-report",
      "version": 3,
      "updatedAt": "2026-10-11T09:40:00.000Z",
      "artifactCount": 12
    },
    "hint": "이 템플릿을 고치는 거라면 --update를 붙여 다시 등록하세요"
  }
}
```

JSON 문법 오류 예: `{"ok":false,"error":{"code":"INVALID_INPUT","message":"JSON으로 읽을 수 없어요","file":"schema.json","line":12,"column":5,"hint":"쉼표·따옴표·괄호가 맞는지 확인해 주세요. 요청은 보내지 않았어요."}}`
