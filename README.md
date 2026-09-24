# khouwledge-plugin

Claude Code 플러그인. Houdini 지식 MCP 서버([khouwledge](https://github.com/JuyeongYi/khouwledge))에 연결하는 설정과,
그 서버를 쓰는 법을 적은 스킬 둘을 묶었다. 이 저장소 자체가 마켓플레이스다.

```
.claude-plugin/marketplace.json
plugins/khouwledge/
  .claude-plugin/plugin.json
  .mcp.json                          ← 서버 연결 (환경변수 넷)
  skills/khouwledge-query/SKILL.md   ← 문서 조회: search / get_* / report_error
  skills/khouwledge-judge/SKILL.md   ← 이미지 판정: ask_* 5개
```

## 설치

Claude Code 안에서:

```
/plugin marketplace add <이 저장소 경로 또는 git URL>
/plugin install khouwledge@khouwledge
```

또는 셸에서 `claude plugin marketplace add <경로>` → `claude plugin install khouwledge@khouwledge`.
스킬 이름은 `khouwledge:khouwledge-query`, `khouwledge:khouwledge-judge`로 보인다.

갱신: 이 저장소를 pull한 뒤 `claude plugin marketplace update khouwledge` → `claude plugin update khouwledge@khouwledge`.
`plugin.json`의 `version`을 올려야 갱신으로 인식된다.

## 환경변수 넷

`.mcp.json`이 읽는다. Claude Code를 띄우는 환경에 설정한다.

| 변수 | 뜻 | 기본값 |
|---|---|---|
| `KHOUWLEDGE_HOST` | 서버 주소 | `192.168.0.11` (Spark의 LAN 주소). 서버와 같은 기계면 `127.0.0.1` |
| `KHOUWLEDGE_PORT` | 포트 | `3000` |
| `KHOUWLEDGE_KEY` | API 키. Spark의 `~/.khouwledge/api-key` 내용 | 없음 — **반드시 설정** |
| `KHOUWLEDGE_SESSION` | 궤적 기록에 남을 이 기계의 이름 | `unnamed` |

**`KHOUWLEDGE_KEY`를 설정하지 않으면 오류 대신 401이 난다** — `${KHOUWLEDGE_KEY}` 문자열이 그대로
헤더에 실린다. 401이면 먼저 변수를 확인한다.

PowerShell:

```powershell
setx KHOUWLEDGE_KEY     "여기에-키"
setx KHOUWLEDGE_SESSION "jylee-pc"
```

Linux(서버와 같은 기계):

```bash
export KHOUWLEDGE_HOST=127.0.0.1
export KHOUWLEDGE_KEY="$(cat ~/.khouwledge/api-key)"
export KHOUWLEDGE_SESSION=spark
```

## 주의

- 프로젝트 폴더에 예전 방식의 `.mcp.json`(서버 이름 `khouwledge`)이나 `.claude/skills/khouwledge/`가 남아
  있으면 서버·스킬이 겹친다. 플러그인으로 옮겼으면 지운다.
- 서버 쪽 사정(떠 있는지, 재색인 중인지)은 khouwledge 저장소의 `client/README.md`와 `docs/handoff.md`.
