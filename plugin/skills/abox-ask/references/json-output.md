# 질문 명령의 `--json` 출력

모든 명령은 stdout에 JSON 한 줄을 쓴다. 키는 camelCase다.

- 성공: `{"ok":true,"data":{…},"warnings":[{"code","message",…}]}` (`warnings`는 있을 때만)
- 실패: `{"ok":false,"error":{"code","message","hint"?,…}}` + 0이 아닌 exit code

## ask

```json
{
  "ok": true,
  "data": {
    "url": "https://artifactbox.app/a/Xk2vR9pQ7mL4tB8nC1dE5f",
    "questionUrl": "https://artifactbox.app/a/Xk2vR9pQ7mL4tB8nC1dE5f#q1",
    "artifactId": "Xk2vR9pQ7mL4tB8nC1dE5f",
    "number": 1,
    "label": "재시도 위치",
    "created": true,
    "visibility": { "mode": "private", "expiresAt": null },
    "project": { "id": "Pq4wT8nV2mK6rX9bL3cH7j", "name": "payment-service" },
    "workspace": { "id": "Wn3kQ7vR2xL9mT4pB8cJ6d", "kind": "personal", "name": "개인" }
  }
}
```

- `url`은 주제, `questionUrl`은 올린 질문의 주소다. 사용자에게는 `questionUrl`을 줄이지 않고 준다.
- `created: true`면 이 질문과 함께 새 주제가 만들어졌다. `--url`로 더했으면 `false`다.
- `--revises`로 대체했으면 `revises`(대체된 질문 번호)가 있다.
- `project` · `workspace`는 주제가 있는 곳이다. 새 주제는 `--project`의 프로젝트(없으면 그 워크스페이스의 `미분류`)에 만들어진다. `--project`의 값 · 실패는 `abox-publish` 스킬의 `references/json-output.md`(publish)와 같다.

`abox ask --example <종류> --json`은 `data`가 그 종류의 질문 JSON 틀이다.

## wait

```json
{
  "ok": true,
  "data": {
    "result": "answered",
    "questionUrl": "https://artifactbox.app/a/Xk2vR9pQ7mL4tB8nC1dE5f#q3",
    "question": {
      "number": 3,
      "kind": "choice",
      "label": "재시도 위치",
      "title": "결제 재시도는 어디서 처리할까요?",
      "status": "decided",
      "revises": null,
      "supersededBy": null,
      "askedAt": "2026-10-22T04:50:00.000Z",
      "answer": {
        "option": "B",
        "optionTitle": "큐 워커에서 재시도",
        "text": "PG사 제한이 분당 5회라서요",
        "source": "web",
        "answeredAt": "2026-10-22T05:03:00.000Z",
        "answeredBy": { "id": "6f1c2a4e-1b2d-4c3e-8f9a-0b1c2d3e4f50", "name": "김네오" }
      },
      "withdrawal": null
    }
  }
}
```

| `result`   | `question.status` | 뜻                                                                                                                                           |
| ---------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `answered` | `decided`         | 결정됐다. 객관식은 `answer.option`(+ `optionTitle`), 주관식은 `answer.text`, 동의 구하기는 `answer.agree: true`. 의견이 있으면 `answer.text` |
| `answered` | `discussing`      | 결정 없이 의견만 왔다. 객관식은 옵션 없이 `answer.text`, 동의 구하기는 `answer.agree: false` + `answer.text`(다른 의견)                      |
| `closed`   | `superseded`      | 다른 질문이 이어받았다. `supersededBy`가 그 번호다                                                                                           |
| `closed`   | `withdrawn`       | 거둬들였다. `withdrawal:{reason(없으면 null), withdrawnAt, withdrawnBy?}`                                                                    |
| `waiting`  | `waiting`         | `--minutes` 동안 답이 없었다. `data.minutes`가 기다린 분이다                                                                                 |

- 질문 본문(`body`)은 싣지 않는다. 필요하면 `abox read '<questionUrl>'`로 읽는다.
- `answer.source`는 `web`(웹) · `cli`(터미널, `abox answer`)다.
- `answeredBy` · `withdrawnBy` · `askedBy`는 팀 워크스페이스에서만 있다.
- 모두 exit 0이다. `result`로 가른다.

## read

`abox read`는 `--json` 없이 Markdown으로 읽는 것이 기본이다. `--json`이면 서버 응답 그대로다.

- 주제 `abox read '<URL>' --json`: `data:{artifact:{id,url,title,description}, topic:{questions, active, allAttachments?}}`
  - `questions`: 모든 질문의 요약(q1부터). 요약은 `number` · `kind` · `label` · `title` · `status` · `revises` · `supersededBy` · `askedAt` · `answer`(글은 200자까지, 잘렸으면 `textTruncated: true`, 객관식이면 `optionTitle`) · `withdrawal`
  - `active`: 진행 중(`waiting` · `discussing`) 질문 전체(본문 `body` 포함). 진행 중이 없으면 마지막 질문 하나
  - `allAttachments`: 주제의 모든 첨부 `[{id, title, type, shareActive}]`(중복 없이). 오너에게만 온다. 공개할 첨부를 찾을 때 쓴다
- 질문 하나 `abox read '<URL>#q3' --json`: `data:{artifact, question}`. `question`은 요약 칸 + 전체 답 + `body`(첨부는 `{note, available, attachedVersion?, target?:{id,url,type,title,description,board?,slide?,currentVersion}}`. `board:{file,title}` · `slide:{file,title,number}`는 첨부가 가리킨 보드 · 장이고, 여는 버전에 없으면 `title`(과 `number`)이 `null`이다)

## answer · withdraw

`data:{question, counts:{total, waiting}}`. `question`은 기록한 뒤의 질문 전체다(`status`가 `decided` · `discussing` · `withdrawn`).

## ls --type question

`data.items[]`는 다른 아티팩트와 같은 요약이고 `type`이 `question`이다. `questions:{total, waiting}`(질문 수 · 답 대기 수)가 있다. `-q`면 `data.items`는 정확히 맞는 주제만이고, 비슷한 주제는 `data.search.similar`에 온다(`data.search`의 칸은 `abox-publish` 스킬 `references/json-output.md`의 'ls').

## 오류

질문 형식 · 첨부 오류(exit 2):

```json
{
  "ok": false,
  "error": {
    "code": "QUESTION_INVALID",
    "message": "질문이 규칙에 맞지 않아요(문제 2개)",
    "issueCount": 2,
    "issues": [
      { "path": "/options/1/detail/how", "code": "required", "message": "…" },
      {
        "path": "/attachments/0/url",
        "code": "…",
        "message": "이 워크스페이스에서 찾을 수 없는 아티팩트예요"
      }
    ]
  }
}
```

`issues[].path`는 질문 JSON 안의 JSON Pointer다(`/options/1/detail/how` = 둘째 옵션의 `detail.how`, `""` = 질문 전체). 50개까지 오고 `issueCount`가 전체 개수다.

이미 닫힌 질문(exit 1):

```json
{
  "ok": false,
  "error": {
    "code": "QUESTION_CLOSED",
    "message": "…",
    "number": 3,
    "status": "decided",
    "answer": {
      "option": "A",
      "source": "web",
      "answeredAt": "2026-10-22T05:03:00.000Z",
      "answeredBy": { "id": "…", "name": "김네오" }
    },
    "supersededBy": null,
    "withdrawal": null
  }
}
```

`status` · `answer` · `supersededBy` · `withdrawal`이 그 질문의 지금 상태다. 답 대기가 아닌 질문에 답하거나, 결정된 질문을 `--change-decision` 없이 대체하거나, 이미 대체됐거나 거둬들인 질문을 대체 · 거두려 할 때 온다.

그 밖의 실패(로그인 필요 exit 3, 서버 연결 실패 exit 6 등)의 모양은 `abox-publish` 스킬의 `references/json-output.md`와 같다.
