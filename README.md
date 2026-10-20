# Artifact Box 플러그인

Claude Code에서 만든 HTML·Markdown 문서를 [Artifact Box](https://artifactbox.app)에 게시하고 링크로 공유하는 플러그인이에요. 게시에 쓰는 `abox` CLI가 함께 들어 있어요.

## 필요한 것

- [Claude Code](https://code.claude.com), 또는 스킬을 쓸 수 있는 다른 에이전트([스킬만 설치](#스킬만-codex--cursor--gemini-cli-같은-다른-에이전트))
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

## 다른 방법으로 설치하기

Claude Code에서는 위의 플러그인 설치를 권해요. 플러그인에는 CLI가 들어 있고 업데이트도 자동이에요. 플러그인과 아래 스킬을 함께 설치하면 같은 스킬이 두 번 보이니 하나만 써요.

### CLI만

터미널에서 `abox` 명령만 쓰려면:

```sh
curl -fsSL https://artifactbox.app/install.sh | sh
```

`~/.local/bin/abox`에 설치돼요. 새 버전이 나오면 명령 결과 아래에 알려 주고, `abox update`로 바꿔요.

### 스킬만 (Codex · Cursor · Gemini CLI 같은 다른 에이전트)

1. 위 「CLI만」으로 CLI를 설치해요. 스킬이 `abox` 명령으로 게시해서 CLI가 꼭 있어야 해요.
2. [skills](https://github.com/vercel-labs/skills)로 스킬을 설치해요. 설치할 에이전트와 범위(지금 프로젝트 · 모든 프로젝트)를 물어봐요.

```sh
npx skills add vlock-neo-jung/artifact-box-plugin
```

묻지 않고 설치하려면 옵션을 붙여요.

```sh
# 모든 프로젝트에서 Codex와 Cursor에
npx skills add vlock-neo-jung/artifact-box-plugin -g -a codex cursor -y
```

| 옵션             | 뜻                                                                        |
| ---------------- | ------------------------------------------------------------------------- |
| `-g`             | 모든 프로젝트에 설치해요. 없으면 지금 프로젝트에만 설치해요               |
| `-a <에이전트…>` | 설치할 에이전트예요(`codex` · `cursor` · `gemini-cli` · `claude-code` 등) |
| `-s <스킬…>`     | 스킬 일부만 설치해요(`abox-publish` · `abox-template`)                    |
| `-y`             | 확인 없이 진행해요                                                        |

| 스킬            | 하는 일                                                       |
| --------------- | ------------------------------------------------------------- |
| `abox-publish`  | HTML·Markdown 게시 · 수정 게시 · 공유 · 목록, 템플릿으로 게시 |
| `abox-template` | 템플릿 만들기 · 고치기 · 삭제                                 |

업데이트와 삭제는 아래 명령으로 해요. 모든 프로젝트에 설치했다면 `-g`를 붙여요.

```sh
npx skills update
npx skills remove abox-publish abox-template
```

스킬 안의 일부 안내(로그인 `! abox login`, 작업 폴더 등)는 Claude Code 기준이에요. 다른 에이전트에서는 터미널에서 `abox login`을 실행해요.

## 로그인

Claude Code 입력창에서 아래 한 줄을 실행해요. 주소가 나오면 브라우저에서 열어 Google로 로그인하고 승인해요. 원격 서버에서 쓰는 중이면 그 주소를 아무 기기의 브라우저에서 열거나, [artifactbox.app/device](https://artifactbox.app/device)에 터미널의 코드를 넣어요.

```
! abox login
```

다른 에이전트나 터미널에서는 `abox login`을 실행해요. 처음 로그인하면 계정이 바로 만들어져요.

## 쓰기

Claude Code에 이렇게 말하면 돼요.

| 요청 예                                     | 하는 일                                                  |
| ------------------------------------------- | -------------------------------------------------------- |
| "이 보고서를 HTML로 만들어서 게시해 줘"     | 문서를 만들어 게시하고 링크를 알려 줘요                  |
| "방금 게시한 거 제목 바꿔서 다시 게시해 줘" | 같은 링크에 새 버전을 올려요                             |
| "링크로 공유해 줘"                          | 로그인 없이 열 수 있는 공유 링크를 만들어요              |
| "회의록 템플릿 만들어 줘"                   | 데이터만 넣으면 같은 모습으로 게시되는 템플릿을 만들어요 |

게시한 문서는 [artifactbox.app](https://artifactbox.app)에서 보고 관리해요.
