# khouwledge-plugin

Claude Code 플러그인. Houdini 지식 MCP 서버([khouwledge](https://github.com/JuyeongYi/khouwledge))에 연결하는 설정과,
그 서버를 쓰는 법을 적은 스킬 둘을 묶었다. 이 저장소 자체가 마켓플레이스다.

```
.claude-plugin/marketplace.json
plugins/khouwledge/
  .claude-plugin/plugin.json
  .mcp.json                          ← 서버 연결 (userConfig 넷)
  skills/khouwledge-query/SKILL.md   ← 문서 조회: search / query_* / report_error
  skills/khouwledge-judge/SKILL.md   ← 이미지 판정: ask_* 5개
```

## 설치

Claude Code 안에서:

```
/plugin marketplace add <이 저장소 경로 또는 git URL>
/plugin install khouwledge@khouwledge
```

켜는 순간 설정 네 개를 묻는다(`userConfig`). 나중에 바꾸려면 `/plugin configure khouwledge@khouwledge` 또는
`/config`. 바꾼 값은 다음 세션부터 적용된다. 플러그인을 갱신해도 값은 남는다.

| 항목 | 뜻 | 기본값 |
|---|---|---|
| `key` | API 키. 서버 기계의 `~/.khouwledge/api-key` 내용. **비밀값**으로 저장된다 | 없음 — **필수** |
| `host` | 서버 주소 | `192.168.0.11` (Spark의 LAN 주소). 서버와 같은 기계면 `127.0.0.1` |
| `port` | 포트 | `3000` |
| `session` | 궤적 기록에 남을 이 기계의 이름 (예 `jylee-pc`) | `unnamed` |

The server only answers on the local network; the default `192.168.0.11:3000` is still correct. Away from home this plugin cannot connect directly (a Tailscale address gets 403), so SSH into the server machine and work there. Direct remote use would need the server setting `mcp_allowed_networks` to include `100.64.0.0/10`, which is the server admin's decision.

셸에서 한 번에:

```bash
claude plugin marketplace add <경로>
claude plugin install khouwledge@khouwledge \
  --config key="$(cat ~/.khouwledge/api-key)" --config host=127.0.0.1 --config session=spark
```

스킬 이름은 `khouwledge:khouwledge-query`, `khouwledge:khouwledge-judge`로 보인다.

갱신: 이 저장소를 pull한 뒤 `claude plugin marketplace update khouwledge` → `claude plugin update khouwledge@khouwledge`.
`plugin.json`의 `version`을 올려야 갱신으로 인식된다.

### 키에 관하여

- 서버(`scripts/serve.sh`)가 처음 뜰 때 무작위로 만든 32자 Bearer 키다. 서버는 `/health` 말고는 이 키가
  없거나 다르면 401을 준다.
- `key`는 macOS에서는 키체인, Linux·Windows에서는 `~/.claude/.credentials.json`(평문 파일)에 들어간다.
- 서버 쪽에서 키를 바꾸면(`api-key` 파일을 지우고 재기동) 여기서도 `key`를 다시 넣어야 한다.
- 401이면 먼저 `key`가 서버 파일과 같은지 본다.

### 확인된 동작 (Claude Code 2.1.281, 2026-09-25 실측)

문서는 `${user_config.…}`를 MCP 서버의 `env`·`args`에 쓴다고만 적고 http 서버의 `headers`는 언급하지 않는다.
헤더를 찍는 임시 서버로 확인했다: `url`과 `headers` 둘 다 치환되고, 비밀값(`sensitive`)도 그대로 실린다.
같은 파일에서 `${ENV:-기본값}` 꼴의 환경변수 치환도 함께 동작한다.

## 주의

- 프로젝트 폴더에 예전 방식의 `.mcp.json`(서버 이름 `khouwledge`)이나 `.claude/skills/khouwledge/`가 남아
  있으면 서버·스킬이 겹친다. 플러그인으로 옮겼으면 지운다.
- 서버 쪽 사정(떠 있는지, 재색인 중인지)은 khouwledge 저장소의 `client/README.md`와 `docs/handoff.md`.
