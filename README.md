# Artifact Box 플러그인

Claude Code에서 만든 HTML·Markdown 문서를 [Artifact Box](https://artifactbox.app)에 게시하고 링크로 공유하는 플러그인이에요. 게시에 쓰는 `abox` CLI가 함께 들어 있어요.

## 필요한 것

- [Claude Code](https://code.claude.com)
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

## 자동 업데이트 켜기

직접 추가한 마켓플레이스는 자동 업데이트가 꺼져 있어요. 한 번 켜 두면 Claude Code를 시작할 때 새 버전을 받아요.

1. `/plugin`을 열어요
2. **Marketplaces**에서 `artifact-box`를 골라요
3. **Enable auto-update**를 골라요

켜지 않았다면 `/plugin marketplace update artifact-box`로 직접 받아요. GitHub SSH 키가 없어 업데이트가 실패하면 환경변수 `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1`을 두고 Claude Code를 다시 시작해요.

## 로그인

Claude Code 입력창에서 아래 한 줄을 실행해요. 주소와 코드가 나오면 브라우저에서 Google로 로그인하고, 같은 코드인지 확인해 승인해요. 원격 서버에서 쓰는 중이면 그 주소를 아무 기기의 브라우저에서 열면 돼요.

```
! abox login
```

처음 로그인하면 계정이 바로 만들어져요.

## 쓰기

Claude Code에 이렇게 말하면 돼요.

| 요청 예                                     | 하는 일                                                  |
| ------------------------------------------- | -------------------------------------------------------- |
| "이 보고서를 HTML로 만들어서 게시해 줘"     | 문서를 만들어 게시하고 링크를 알려 줘요                  |
| "방금 게시한 거 제목 바꿔서 다시 게시해 줘" | 같은 링크에 새 버전을 올려요                             |
| "링크로 공유해 줘"                          | 로그인 없이 열 수 있는 공유 링크를 만들어요              |
| "회의록 템플릿 만들어 줘"                   | 데이터만 넣으면 같은 모습으로 게시되는 템플릿을 만들어요 |

게시한 문서는 [artifactbox.app](https://artifactbox.app)에서 보고 관리해요.
