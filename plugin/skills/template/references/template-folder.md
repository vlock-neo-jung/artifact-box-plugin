# 템플릿 폴더와 스키마 규칙

## 폴더 구조

```
<작업 폴더>/       예: <scratchpad>/artifact-box/templates/weekly-report/
├─ template.json   이름·제목·설명
├─ schema.json     JSON Schema 2020-12
├─ sample.json     스키마를 통과하는 예시 데이터 1개
├─ guide.md        작성 지침 (선택)
└─ screen/         index.html + 자산(.js·.css·이미지)
```

| 파일            | 필수                     | 한도                            | 서버에 저장되는 경로        |
| --------------- | ------------------------ | ------------------------------- | --------------------------- |
| `template.json` | 필수                     | —                               | 템플릿 정보(이름·제목·설명) |
| `schema.json`   | 필수                     | 64KB                            | `.abox/schema.json`         |
| `sample.json`   | 필수                     | 1MB                             | `.abox/data.json`           |
| `guide.md`      | 선택                     | 64KB                            | `.abox/guide.md`            |
| `screen/**`     | `screen/index.html` 필수 | 파일 10MB · 전체 50MB · 2,000개 | `**`(`screen/`을 뺀 경로)   |

- `screen/`은 폴더 게시와 같은 규칙을 따른다.
  - 비밀 파일(`.env*`·`credentials.json`·`*.pem`·`id_rsa*` 등)이 있으면 `SECRET_FILE`
  - `.git/`·`node_modules/`·`.wrangler/`는 자동으로 빠진다
  - 폴더 안 심볼릭 링크는 올리지 않는다
- `template push`에는 `--exclude`가 없다. 올리지 않을 파일은 `screen/` 밖에 둔다.
- `screen/.abox/`는 예약 경로라 쓸 수 없다(`TEMPLATE_INVALID`).
- 폴더 최상위의 다른 파일(메모 등)은 읽지 않는다.
- `template pull`은 같은 구조로 받는다. 등록한 뒤 폴더를 지워도 원본은 서버에 있다.

## template.json

```json
{
  "name": "weekly-report",
  "title": "주간 보고",
  "description": "팀의 한 주 진행 상황과 위험을 정리하는 보고서"
}
```

| 키            | 규칙                                                                                                                                                                                                                                                  |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`        | 영문 소문자로 시작, 소문자·숫자·`-`, 2~40자(`^[a-z][a-z0-9-]{1,39}$`). 한 워크스페이스의 살아 있는 템플릿끼리 유일(다른 워크스페이스에는 같은 이름이 있을 수 있다). CLI와 웹 주소(`/w/<워크스페이스>/templates/<이름>`)의 키라 등록한 뒤 바꿀 수 없다 |
| `title`       | 한국어 제목, 1~500자. 사용자가 부르는 이름("주간 보고")                                                                                                                                                                                               |
| `description` | 한 문장, 1~500자. 무엇을 담는 템플릿인지. 에이전트가 템플릿을 고르는 근거다                                                                                                                                                                           |

- 값이 규칙에 맞지 않으면 `TEMPLATE_INVALID`(`file: "template.json"`, `issues[].path`는 `/name`·`/title`·`/description`)다.
- 제목·설명만 고쳐 `push --update`하면 `metadata_updated`이고 템플릿 버전은 그대로다.

## 스키마 규칙 (서버 검사)

| 규칙                                                                                                                                                                           | 어기면 나는 `issues[].code`                            |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| `$schema`는 생략하거나 `https://json-schema.org/draft/2020-12/schema`                                                                                                          | `unsupported-draft-error`                              |
| 최상위 `"type": "object"`                                                                                                                                                      | `root-type-error`                                      |
| `$ref`·`$dynamicRef`는 같은 문서 안(`#/$defs/…`)만                                                                                                                             | `remote-ref-error`                                     |
| `$ref`는 `#/…` 경로로 스키마 자리만 가리킴. 앵커(`#name`), `const`·`enum`·`examples`·`default` 안, `$defs` 같은 이름 목록 자체는 안 됨                                         | `ref-target-error`                                     |
| `$id`는 최상위에만                                                                                                                                                             | `nested-id-error`                                      |
| 스키마 중첩은 128단계까지                                                                                                                                                      | `max-depth-error`                                      |
| Object.prototype 멤버 이름(`constructor`·`toString`·`valueOf`·`hasOwnProperty`·`__proto__` 등)을 필드 이름으로 쓰지 않음. 어느 깊이든, 스키마 `properties` 키와 데이터 키 모두 | `reserved-property-error` "쓸 수 없는 필드 이름이에요" |
| 스키마 자체가 올바름(정규식, 키워드 값의 형태)                                                                                                                                 | `schema-error`(짧은 원인 포함)                         |
| `pattern`·`patternProperties` 키에 반복 안의 반복(`(\w+\s?)*`, `(a+)+`)을 쓰지 않음. 검사가 몇 분씩 걸릴 수 있어 거부한다                                                      | `unsafe-pattern-error`                                 |
| 라이브러리 확장 키워드 `regexFlags`·`errorMessages`·`oneOfProperty`·`propertyDependencies`를 쓰지 않음                                                                         | `unsupported-keyword-error`                            |
| `contains`가 `true`·`false`이면 `minContains`·`maxContains`를 같이 쓰지 않음(`contains: {}`로)                                                                                 | `unsupported-keyword-error`                            |

- 모르는 키워드는 등록되지만 경고 `SCHEMA_UNKNOWN_KEYWORDS`(`paths`)가 난다. 오타(`maxlength` → `maxLength`)인지 확인해 고친다. `x-`로 시작하는 키워드(`x-ui`)는 경고 없이 허용된다.
- 검사하는 `format`: `date`·`date-time`·`time`·`duration`·`email`·`url`·`uri`·`uuid`·`regex`·`json-pointer`·`relative-json-pointer`. 다른 `format`은 검사하지 않는다.
- 필드 이름에 `/`·`~`를 쓰지 않는다. 오류 경로(JSON Pointer)에서 `~1`·`~0`으로 바뀌어 위치를 읽기 어렵다.
- 반복 안의 반복은 구분자가 있어 안전한 모양도 거부된다. 반복 없이 같은 뜻으로 쓴다.
  - slug: `^[a-z0-9]+(?:-[a-z0-9]+)*$` 대신 `^[a-z0-9](?:[a-z0-9-]*[a-z0-9])?$`
  - 버전: `^\d+(\.\d+)*$` 대신 `^\d+(\.\d+)?(\.\d+)?$`
  - `?`·`{n}`(횟수가 고정된 반복)은 괜찮다: `^\d+(\.\d+)?$`, `^\d{3}-\d{4}$`
- 데이터·샘플은 64단계보다 깊게 중첩할 수 없고(`max-depth-error`), 값은 모두 합쳐 100,000개까지다(`max-values-error`).
- 등록 검사와 데이터 검증은 하나에 2초까지다. 넘으면 `timeout-error`로 거부된다. 규칙을 통과한 정규식이나 참조도 데이터에 따라 오래 걸릴 수 있으니 단순하게 쓴다.

## 스키마 작성 규칙 (스킬 규칙, 서버는 강제하지 않음)

- 필드 이름은 영문 camelCase(`dueDate`)로 쓴다. 사람이 읽는 이름은 `title`에 한국어로 쓴다.
- 모든 필드에 `title`(한국어 이름)과 `description`(작성 힌트: 무엇을, 얼마나, 어떤 톤으로)을 쓴다. 웹 필드 표와 데이터를 쓰는 에이전트가 이 값을 읽는다.
- 꼭 있어야 하는 필드는 `required`에 넣는다. 나머지는 데이터에 없을 수 있고, 화면이 그 경우를 처리한다.
- 객체마다 `"additionalProperties": false`를 둔다. 데이터 키의 오타가 `no-additional-properties-error`로 잡힌다.
- 정해진 값은 `enum`, 날짜는 `"format": "date"`, 비면 안 되는 문자열은 `"minLength": 1`, 한 줄 제목은 `maxLength`, 목록 개수는 `minItems`·`maxItems`로 적는다.
- 반복되는 객체는 `$defs`에 두고 `$ref`로 쓴다.
- `oneOf`·`anyOf`·`allOf`·`not`·`if`·`patternProperties`·`dependentSchemas`·`prefixItems`는 웹 필드 표에서 `복합` 한 행으로만 보인다. `type`·`properties`·`items`·`enum`으로 표현할 수 있으면 쓰지 않는다.
- 이미지 필드는 `{"type": "string", "format": "uri", "pattern": "^(https://|data:image/)"}`로 두어 https 주소와 작은 data URI만 받는다(`format: "uri"`만으로는 `http:`·`javascript:`도 통과한다). 아티팩트마다 로컬 이미지 파일을 첨부하는 방법은 없다.

예 — `schema.json`:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["title", "week", "done"],
  "additionalProperties": false,
  "properties": {
    "title": {
      "type": "string",
      "title": "제목",
      "description": "팀 이름과 주차를 넣은 한 줄. 예: 플랫폼팀 41주차 보고",
      "minLength": 1,
      "maxLength": 80
    },
    "week": {
      "type": "string",
      "format": "date",
      "title": "주 시작일",
      "description": "그 주 월요일(YYYY-MM-DD)"
    },
    "done": {
      "type": "array",
      "title": "한 일",
      "description": "이번 주에 끝낸 일. 결과 중심 한 줄씩, 3~7개",
      "minItems": 1,
      "maxItems": 7,
      "items": { "type": "string", "minLength": 1 }
    },
    "risks": {
      "type": "array",
      "title": "위험",
      "description": "일정·품질을 위협하는 일. 없으면 빼거나 빈 목록",
      "items": { "$ref": "#/$defs/risk" }
    }
  },
  "$defs": {
    "risk": {
      "type": "object",
      "required": ["title", "level"],
      "additionalProperties": false,
      "properties": {
        "title": {
          "type": "string",
          "title": "내용",
          "description": "무엇이 위험한지 한 문장",
          "minLength": 1
        },
        "level": {
          "type": "string",
          "title": "위험도",
          "description": "high는 이번 주 안에 결정이 필요할 때만",
          "enum": ["high", "medium", "low"]
        },
        "owner": { "type": "string", "title": "담당자", "description": "이름만" }
      }
    }
  }
}
```

## 샘플 데이터 (sample.json)

- 스키마를 통과해야 등록된다. 통과하지 못하면 `TEMPLATE_INVALID`(`file: "sample.json"`)이고 `issues[].path`는 샘플 안의 위치다.
- 실제 같은 데이터로 쓴다. 분량·톤·문장 길이를 실제 쓰임과 맞춘다. 웹 미리보기가 이 샘플로 그려지고, 데이터를 쓰는 에이전트가 분량·톤의 기준으로 삼는다.
- 선택 필드도 채워 화면 전체가 미리보기에 보이게 한다.
- `lorem ipsum`·`test`·`값1` 같은 자리 채움 문자열과 실제 비밀·개인 정보를 넣지 않는다.

```json
{
  "title": "플랫폼팀 41주차 보고",
  "week": "2026-10-05",
  "done": [
    "결제 API p95 응답 시간 420ms → 290ms",
    "배포 파이프라인에 빌드 캐시 적용(평균 12분 → 7분)",
    "10-02 장애 회고 문서 공유"
  ],
  "risks": [
    {
      "title": "결제사 인증서가 10-20에 만료돼 갱신 일정 확정이 필요",
      "level": "high",
      "owner": "박서준"
    },
    { "title": "QA 1명 휴가로 회귀 테스트가 하루 늦어질 수 있음", "level": "medium" }
  ]
}
```

## 작성 지침 (guide.md, 선택)

- 스키마로 담지 못하는 규칙을 쓴다: 톤(해요체·개조식), 분량, 순서, 넣지 않을 것.
- 데이터를 쓰는 에이전트가 `template info --json`·`pull --json`의 `guide`로 읽는다. 웹에서는 원문 텍스트로 보인다.

```markdown
# 작성 지침

- `done`은 결과 중심 한 줄로 쓴다(숫자가 있으면 전후 값).
- `risks`는 이번 주에 새로 생기거나 커진 것만 쓴다.
```
