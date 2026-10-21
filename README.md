# Artifact Box 플러그인

Claude Code에서 만든 HTML·Markdown 문서와 화면 시안 캔버스를 [Artifact Box](https://artifactbox.app)에 게시하고 링크로 공유하는 플러그인이에요. 게시에 쓰는 `abox` CLI가 함께 들어 있어요.

## 필요한 것

- [Claude Code](https://code.claude.com), 또는 스킬을 쓸 수 있는 다른 에이전트([설치 안내](https://artifactbox.app/guide))
- [Node.js](https://nodejs.org) 24 이상
- Google 계정

## 설치

Claude Code 입력창에서:

```
/plugin marketplace add vlock-neo-jung/artifact-box-plugin
/plugin install artifact-box@artifact-box
```

터미널에서는:

```sh
claude plugin marketplace add vlock-neo-jung/artifact-box-plugin
claude plugin install artifact-box@artifact-box
```

설치한 뒤 `/reload-plugins`를 실행하거나 새 세션을 열어요.

자동 업데이트 켜기, CLI + 스킬(Codex · Cursor · Gemini CLI 등 다른 에이전트 포함), CLI만 설치하는 방법은 [설치 안내](https://artifactbox.app/guide)에 있어요.

| 스킬            | 하는 일                                                                                   |
| --------------- | ----------------------------------------------------------------------------------------- |
| `abox-publish`  | 내용에 맞게 HTML·Markdown·캔버스로 만들어 게시 · 수정 게시 · 공유 · 목록, 템플릿으로 게시 |
| `abox-template` | 템플릿 만들기 · 고치기 · 삭제                                                             |

## 로그인

Claude Code 입력창에서 아래 한 줄을 실행해요. 주소가 나오면 브라우저에서 열어 Google로 로그인하고 승인해요. 원격 서버에서 쓰는 중이면 그 주소를 아무 기기의 브라우저에서 열거나, [artifactbox.app/device](https://artifactbox.app/device)에 터미널의 코드를 넣어요.

```
! abox login
```

다른 에이전트나 터미널에서는 `abox login`을 실행해요. 처음 로그인하면 계정이 바로 만들어져요.

## 쓰기

Claude Code에 이렇게 말하면 돼요.

| 요청 예                                        | 하는 일                                                      |
| ---------------------------------------------- | ------------------------------------------------------------ |
| "이 내용 문서로 정리해서 게시해 줘"            | Markdown 문서로 만들어 게시하고 링크를 알려 줘요             |
| "이 지표를 대시보드 HTML로 만들어서 게시해 줘" | HTML 페이지로 만들어 게시해요                                |
| "결제 화면 시안 3장 만들어서 게시해 줘"        | 캔버스로 만들어 한 링크에서 나란히 확대 · 이동하며 보게 해요 |
| "방금 게시한 거 제목 바꿔서 다시 게시해 줘"    | 같은 링크에 새 버전을 올려요                                 |
| "링크로 공유해 줘"                             | 로그인 없이 열 수 있는 공유 링크를 만들어요                  |
| "회의록 템플릿 만들어 줘"                      | 데이터만 넣으면 같은 모습으로 게시되는 템플릿을 만들어요     |

게시한 문서는 [artifactbox.app](https://artifactbox.app)에서 보고 관리해요.
