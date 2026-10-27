# @herdr/ai-flint-mac

Recursive 의 로컬 코딩 모델 **Flint**(40B MoE, 토큰당 2.5B 활성)를 **M4 Max** 에서 [rec](https://github.com/recursive-oss/rec) 0.1.1 로 돌릴 때 쓰는 튜닝 런타임.
rec 이 설치해 둔 원래 런타임을 복사하고 `rec_mlx` 파일 6개를 바꾸고 1개를 더한다. 원래 `rec serve` 는 그대로 남는다.

측정 환경: M4 Max 40코어 GPU · 48 GB · macOS 27.2 · rec 0.1.1 · flint-int4.

## 결과

| | rec 0.1.1 그대로 | 이 런타임 |
|---|---|---|
| 디코드, 32K 문맥 | 107.2 tok/s | 129.7 tok/s (+21%) |
| 디코드, 64K 문맥 | 74.0 tok/s | 91.5 tok/s (+24%) |
| 프롬프트 읽기, 32K | 약 1,300 tok/s | 같음 |
| Claude Code · opencode 두 번째 요청 (6K 프롬프트) | 3.3 s, 캐시 0 | 1.2~1.3 s, 캐시 4,096 토큰 |
| 같은 대화를 이어 보낼 때 첫 글자 (19.6K) | — | 0.09 s |
| 다른 대화가 RAM 캐시를 가져간 뒤 · 재시작 뒤 | 처음부터 다시 읽음 | 디스크에서 0.11~0.24 s 에 불러옴 (14.7K) |

## 바꾼 것

| 무엇 | 파일 | 왜 |
|---|---|---|
| 디코드 워커 76 / MLP 64 / 헤드 64 | `kernels/device.py` | rec 이 40코어 M5 Max 에 쓰는 값. M4 Max 40코어에서도 디코드 +21~24% |
| 워커가 36이 아니면 필터 샘플러 사용 | `kernels/sampling.py` | rec 의 resident8 greedy · T=1 커널은 워커 36에서만 맞는 토큰을 낸다 (76이면 틀린 토큰) |
| 프롬프트 체크포인트 2개 → 64개 | `prefix.py` | 긴 대화에서 앞부분이 바뀌어도 가까운 체크포인트부터 다시 읽음 (recursive-oss/rec PR #24 제안과 같은 값). 공통 앞부분 계산은 `common_prefix` 하나로 `engine.py` 와 같이 씀 |
| Claude Code 과금 도장 고정 | `chat.py` | 시스템 프롬프트 맨 앞 `cch=` · `cc_version` 끝자리가 요청마다 바뀌어 캐시를 못 씀. `f` 로 고정 |
| opencode `<env>` 블록을 첫 사용자 메시지로 | `chat.py` | 폴더 · 날짜가 시스템 프롬프트 앞쪽에 있어 매번 캐시가 깨짐 |
| 대화 캐시를 SSD 에 저장 · 불러오기 | `disk_cache.py`(새 파일), `engine.py`, `scheduler.py` | 응답이 끝난 뒤 쉬는 시간에 저장(디코드 속도에 영향 없음), RAM 캐시보다 길게 맞는 저장본이 있으면 불러옴 |
| 서버 기본 샘플링 · 출력 한도 | `server.py` | `rec serve` 는 `--context` · `--port` 만 넘겨 앱 설정 화면의 값을 줄 길이 없음. 아래 `REC_TEMPERATURE` 등으로 받음 |
| `/v1/responses` (OpenAI Responses API) | `responses.py`(새 파일), `server.py` | Codex 는 이 API 만 쓴다. `/v1/messages` 처럼 채팅 요청으로 바꿔 돌리고, 답(JSON · SSE)을 Responses 형식으로 되돌림. `/v1/models?client_version=…` 처럼 뒤에 붙은 것도 받음 |
| 요청마다 숫자만 기록 | `scheduler.py` | 앱의 사용량 화면용. `REC_METRICS` 가 있을 때만 토큰 수 · 시간 · 속도 · 메모리를 JSONL 로 남기고 대화 내용은 남기지 않음 |
| 모델 이름 `claude-flint` | `server.py` | Claude 앱 3P 모드는 Claude 처럼 보이는 모델 이름만 받는다. `flint` 와 같이 받음 |

Responses API:
- 저장하지 않는다. `previous_response_id` 는 거절하고, Codex 처럼 대화 전체를 보내야 한다(`store=false`).
- 추론은 `reasoning_text` 를 담은 reasoning 항목으로 내보내고, 다음 요청에서 그 항목을 다시 추론으로 넣는다.
- 함수 도구만 모델에 준다. Codex 가 같이 보내는 웹 검색 · 하위 에이전트(namespace) 도구는 빼고 경고를 한 번 남긴다. 그림 · 파일 입력은 "읽지 못함" 메모로 바꾼다.
- `text.format` 이 json_schema 인 요청(구조화 출력)과 `tool_choice` 강제 호출은 거절한다. 토큰 한도에 걸린 답은 `response.incomplete` 로 끝난다.

디스크 캐시:
- 8,192 토큰 이상이고, 이미 저장된 앞부분보다 4,096 토큰 이상 길어졌을 때만 저장한다. 새 파일을 쓰면 짧은 쪽 파일은 지우므로 대화 하나에 파일 하나.
- 모델 · 가중치 형식 · KV 형식마다 따로 폴더를 쓴다. 다른 모델의 KV 는 불러오지 않는다.
- 남은 디스크가 저장할 크기의 두 배보다 작으면 저장하지 않는다. 읽다 실패한 파일은 지우고 처음부터 읽는다.
- 불러오기 전에 RAM 캐시를 먼저 놓아, 긴 프롬프트의 KV 를 두 벌 들고 있지 않는다.

바꾼 파일마다 맨 위에 `Modified from recursive-oss/rec 0.1.1` 한 줄이 있다. 목록은 `NOTICE`.

## 설치

rec 0.1.1 이 설치돼 있고 `rec serve` 를 한 번 돌려 rec 이 런타임 · 모델을 받아 둔 상태여야 한다.

```sh
python3 install.py --check   # rec 버전 · 원래 런타임 · 덮어쓸 파일이 맞는지만 확인
python3 install.py           # ~/.local/share/rec-tuned (rt + manifest.json) 와 ~/.local/bin/rec-serve-tuned 생성
```

- 원래 런타임을 APFS 복제(`cp -c`)로 복사하므로 디스크를 거의 더 쓰지 않는다.
- 이미 `~/.local/share/rec-tuned` 가 있으면 `rec-tuned.bak-<시각>` 으로 이름만 바꿔 남긴다.
- rec 이 0.1.1 이 아니면 멈춘다. rec 을 올리면 `overlay/` 를 새 rec 에 맞춰 다시 만들어야 한다.

## 실행

```sh
rec stop
nohup rec-serve-tuned > ~/.local/share/rec/serve.log 2>&1 &
rec status                    # "ready":true 면 준비 끝, 포트 11500
```

`rec launch opencode` 같은 rec 명령은 떠 있는 서버를 그대로 쓴다. 원래 런타임으로 돌아가려면 `rec stop` 뒤 `rec serve`.

### Mac 앱 (Flint.app)

`app/` 은 위 일을 버튼으로 하는 SwiftUI 앱이다. 창은 처음 열 때 서버 화면 높이에 맞춰 열리고, 다른 탭이 더 길면 화면 안에서 늘어난다. 대화 캐시 최대는 8 ~ 512 GiB 에서 고른다. 서버 켜고 끄기, 옵션(샘플링 · 출력 한도 · 워커 · 청크 · 대화 캐시), 요청별 속도 · 메모리(읽기 · 생성 최고 속도 포함: 서버를 켠 뒤, 요청 기록을 남기면 남긴 기록까지), 이 Mac 에 맞추기(후보마다 서버를 다시 켜 재고 3% 넘게 빠를 때만 바꿈), Claude Code · Claude 앱 · Codex CLI · Codex 앱 · opencode 연결(자동 연결이 없는 Xcode · Zed · VS Code · Aider · Goose 는 "준비 중"으로 흐리게 두고 고를 수 없음. 연결 화면 맨 위에 쓰는 법과 복사할 명령: `claude-flint`, `codex-flint`, `opencode -m flint/flint`, Codex 앱은 ChatGPT 앱을 다시 시작해 프로젝트 대화에서 Flint (local), Claude 앱은 끈 채로 연결한 뒤 켜서 Flint (local)), 최소 사양 검사, 튜닝 런타임 설치 · 검사. 포트는 11500 · 11502 ~ 11504 중에서 고른다(11501 은 Codex 앱 중계기 몫). 서버가 꺼진 채 저장해도 연결한 앱 설정이 새 포트 · 길이로 바로 바뀐다. 연결 · 끊기는 창을 멈추지 않게 뒤에서 돈다.

```sh
cd app
swift test        # FlintCore 단위 시험
./build.sh        # build/Flint.app (ad-hoc 서명, install.py · overlay 를 Resources 에 넣음)
open build/Flint.app
```

- 연결된 앱이 있으면 Flint 를 끌 때 묻는다: 끊고 끄기(모두 연결 전으로 되돌림. Claude 앱이 연결돼 있으면 Claude 앱을 먼저 끄고, 10초 안에 안 꺼지면 Flint 도 끄지 않음) · 연결 둔 채 끄기 · 취소. 로그아웃 · 시스템 종료 때는 묻지 않는다.
- 앱이 켠 서버는 launchd 아래에서 돌아 앱을 닫아도 남는다. "앱을 닫아도 계속 켜 두기"를 끄면 닫을 때 `rec stop`.
- 설정은 `~/Library/Application Support/Flint/` (settings.json, calibration.json, metrics.jsonl).
- Claude Code 연결은 `~/.config/flint/claude-flint.json`(키 포함, 600), `~/.local/bin/claude-flint`, Flint 전용 폴더 `~/.config/flint/claude` 를 만든다. `claude-flint` 는 `CLAUDE_CONFIG_DIR` 를 그 폴더로 두고 켜므로 `~/.claude` 와 `~/.claude.json` 은 한 번도 쓰이지 않는다(Flint 세션에서 `/model` 을 골라도 평소 claude 의 기본 모델이 바뀌지 않음). 대신 스킬 · MCP · 대화 기록은 평소 claude 와 따로다. 끊으면 세 가지를 모두 휴지통으로 옮긴다.
- opencode 연결은 설정에 `provider.flint` 하나만 더하고(주석 · 순서 유지), 키는 `{file:…/api-key}` 로 rec 키 파일을 읽게 한다. 연결 전 원본은 `~/.config/flint/backup` 에 복사하고 Flint 가 쓴 내용의 해시를 `~/.config/flint/opencode-state.json` 에 남긴다. 끊을 때 파일이 그대로면 원본을 바이트 그대로 되돌리고(Flint 가 만든 파일이면 휴지통으로), 그사이 사용자가 고쳤으면 `provider.flint` 와 Flint 가 만든 빈 `provider`, `flint/` 를 가리키는 `model` · `small_model` 만 원래 값으로 돌린다. 연결한 뒤 사용자가 파일을 고친 상태에서 Flint 가 다시 쓰면(포트 · 길이 · 키 변경) 그 뒤 끊을 때도 원본을 통째로 되돌리지 않고 Flint 가 넣은 것만 뺀다. 연결 여부는 `opencode-state.json` 이 있는지로 본다(사용자가 직접 만든 `provider.flint` 는 연결로 치지 않고, 끊기 · 모두 되돌리기도 건드리지 않음).
- Codex 연결도 Claude Code 처럼 따로 쓴다. `~/.codex` 는 ChatGPT 앱 안의 Codex 와 같이 쓰는 폴더라, 고치면 앱도 같이 바뀌기 때문이다. Flint 전용 폴더 `~/.config/flint/codex` 에 프로필 `flint.config.toml`(공급자 `flint`, `wire_api` 기본 responses, 문맥 길이, 자동 압축 한도 = 문맥 − 출력 한도, 웹 검색 끔, 첫 토큰까지 오래 걸리는 긴 프롬프트를 위해 대기 30분)을 쓰고, `~/.local/bin/codex-flint` 는 `CODEX_HOME` 을 그 폴더로 두고 `codex --profile flint` 로 켠다. 키는 `auth.command` 가 rec 키 파일을 `/bin/cat` 으로 읽어 설정에 남지 않는다. 다시 연결(포트 · 길이 변경)할 때는 프로필만 다시 쓰므로 Codex 가 그 폴더에 남긴 신뢰 기록 · 대화는 그대로다. 끊으면 `codex-flint` 와 그 폴더를 휴지통으로 옮긴다. 모델 정보를 Codex 고유 형식으로 주지 않으므로 실행마다 "Model metadata for `flint` not found" 경고가 한 줄 나온다.
- Codex 앱(ChatGPT 앱 안의 Codex) 연결은 앱의 모델 목록 맨 아래에 "Flint (local)" 을 더한다. Codex 는 공급자 하나의 모델만 보여 주므로, 내장 OpenAI 공급자를 이 Mac 의 중계기(`relay/flint_codex_relay.py`, 127.0.0.1:11501)로 돌린다. `~/.codex/config.toml` 맨 위에 `openai_base_url = "http://127.0.0.1:11501/v1"` 한 줄(주석 포함)만 넣고, 사용자가 원래 `openai_base_url` 을 두었으면 그 줄을 빼고 중계기가 그 주소로 보낸다.
  - 중계기: `GET /v1/models` 는 OpenAI 로 보내고 답 끝에 flint 를 붙인다(OpenAI 의 맨 위 모델을 복사해 이름 · 문맥 · 압축 한도를 바꾸고 apply_patch · 웹 검색 · 그림 · 속도 단계는 뺌). ETag 에 `-flint<문맥>-<압축 한도>` 를 붙여 flint 없이(또는 다른 길이로) 저장된 목록은 다시 받게 한다. `POST /v1/responses` 는 모델이 flint 면 zstd 를 풀어 rec 키로 Flint 서버에, 아니면 사용자 헤더 그대로 OpenAI 에 바이트 그대로. 웹소켓은 거절(426)해 Codex 가 HTTP 로 오게 한다(매 요청이 모델을 밝히므로). Host 가 `127.0.0.1` · `localhost` 가 아닌 요청은 403(127.0.0.1 로 풀리는 웹 페이지가 rec 키로 Flint 를 쓰지 못하게). 로그에는 시각 · 경로 · 갈래 · 상태 · 걸린 시간만 남고 헤더 · 본문은 남지 않는다.
  - 긴 대화: 이 공급자에서 Codex 는 압축을 서버에 맡긴다(마지막 입력이 `compaction_trigger` 인 `/responses` 요청, 답으로 `compaction` 항목 하나). flint 대화면 중계기가 Flint 에게 인계 요약을 쓰게 하고(도구 없이, 최대 16,384 토큰, 기다리는 동안 10초마다 `response.in_progress`), 그 요약을 `flint-summary-v1:` 표시를 단 `compaction` 항목으로 돌려준다. 이 항목이 다음 요청에 돌아오면(flint 로든, 모델을 바꿔 OpenAI 로든) 요약 글이 든 사용자 메시지로 바꿔 보낸다. OpenAI 가 만든 압축 항목은 Flint 가 읽지 못해 빠진다. 요약 중에 Codex 가 끊으면 중계기도 Flint 와의 연결을 닫는다.
  - 중계기는 launchd 에이전트(`~/Library/LaunchAgents/com.herdr.flint.codex-relay.plist`, 로그인 때 켜짐, 죽으면 다시 켜짐)이고, Homebrew 의 Python 3.14+ 로 돈다(Codex 가 요청을 zstd 로 보내 `compression.zstd` 가 필요). 연결 버튼은 중계기를 먼저 켜 응답을 확인한 뒤에만 config.toml 을 고친다.
  - 끊기: config.toml 이 Flint 가 쓴 그대로면 연결 전 원본(`~/.config/flint/backup`)을 바이트 그대로 되돌리고, 그사이 바뀌었으면(ChatGPT 앱이 이 파일을 스스로 고친다) Flint 줄만 빼고 원래 줄을 되살린다. 그다음 ChatGPT 앱이 꺼져 있으면 중계기를 내리고 plist · `~/.config/flint/codex-app` 을 휴지통으로 옮긴다. ChatGPT 앱이나 codex CLI 가 켜져 있으면(둘 다 config.toml 을 켤 때만 읽어 계속 중계기로 보낸다) 중계기를 `--until-gone <pid…> --app-name ChatGPT --app-name codex` 로 다시 띄운다(codex CLI 는 node 래퍼가 띄운 `codex` 프로그램의 pid): 그것들이 모두 꺼질 때까지 그대로 중계해 재시작 없이 OpenAI 모델을 계속 쓰게 하고, 꺼지면 스스로 멈춰 plist · 폴더를 휴지통으로 옮기고 빈 `~/.config/flint` 를 지운 뒤 0 으로 끝난다(launchd `KeepAlive` 가 `SuccessfulExit: false` 라 다시 켜지 않음. 휴지통으로 옮기다 실패해도 0 으로 끝남). 다음에 ChatGPT 앱을 켜면 원래 모델 목록만 보인다.
  - 연결해 둔 동안 Codex 앱 · `~/.codex` 를 쓰는 codex CLI 의 OpenAI 요청은 모두 이 중계기를 지난다(로그인 토큰 포함, 127.0.0.1 안에서만). 중계기가 꺼져 있으면 Codex 앱이 OpenAI 에 닿지 못한다.
- Claude 앱(Claude 데스크톱) 연결은 앱의 3P 모드(공식 게이트웨이 설정)를 쓴다. 앱은 시작할 때 `~/Library/Application Support/Claude-3p/configLibrary` 에 공급자가 든 설정이 있으면 3P 모드로 켜지고 데이터도 그 폴더에 둔다(평소 폴더 `…/Claude` 는 그대로). Flint 는 그 폴더에 세 파일을 쓴다(모두 600): `configLibrary/<uuid>.json`(`inferenceProvider` gateway, `inferenceGatewayBaseUrl` = Anthropic 호환 주소, `inferenceGatewayApiKey` = rec 키, bearer, 모델 탐색 끔, `inferenceModels` = `claude-flint` 에 표시 이름 "Flint (local)"), `configLibrary/_meta.json`(그 설정을 적용), `claude_desktop_config.json`(`deploymentMode` 3p). 앱은 Claude 처럼 보이는 모델 이름만 받으므로 서버가 `claude-flint` 도 flint 로 받는다(`overlay/rec_mlx/server.py`).
  - Claude 앱이 켜져 있으면 연결 · 끊기를 거절한다(앱이 설정을 시작할 때 읽고, 끌 때 폴더를 다시 쓰므로). `Claude-3p` 가 이미 있으면(사용자가 직접 넣은 3P 설정일 수 있음) 건드리지 않는다.
  - 연결해 둔 동안 Claude 앱 전체가 3P 모드다: claude.ai 로그인 · 대화 · 동기화는 안 보인다. 끊으면 `Claude-3p`(3P 모드의 대화 포함)와 연결 뒤 생긴 `~/Library/Logs/Claude-3p` 를 휴지통으로 옮겨 다음 실행부터 평소 Claude 로 돌아간다. 포트 · 키가 바뀌면 같은 설정 파일을 다시 쓴다(켜져 있는 앱은 다음 실행 때 읽음).
- 맞추기의 워커 후보는 GPU 코어 수로 계산한다: 디코드는 코어당 1.0~1.9(4의 배수, 칩 기본값이 더 높으면 그것까지), MLP·헤드는 rec 이 받는 32 · 64 중 디코드 이하. 상주 디코드 커널은 워커 전체가 서로를 기다리므로 그보다 높은 값은 옵션에도 없다. 값마다 짧은 대화와 32K 깊이에서 생성 속도를 재고, greedy 답이 지금 값과 다르면 버리고, 제시간에 안 끝나면 그 위는 건너뛴다.
- M4 Max 40코어에서 두 번 돌린 결과, 디코드 워커 64 가 76 보다 32K 깊이 생성에서 빨랐다(136.1 · 130.9 대 117.1 · 111.5 tok/s, 짧은 대화는 같음). 오버레이 기본값(76)은 아직 바꾸지 않았다.

## 스위치 (환경 변수)

| 변수 | 기본 | 뜻 |
|---|---|---|
| `REC_DISK_CACHE` | `~/.cache/rec-kv` | 디스크 캐시 폴더, `0` 이면 끔 |
| `REC_DISK_CACHE_GB` | 32 | 디스크 캐시 최대 크기 (넘으면 오래된 것부터 지움) |
| `REC_DISK_CACHE_MIN` | 8192 | 이보다 짧은 프롬프트는 저장 안 함 |
| `REC_EXP_WORKERS` | `76,64,64` | 디코드 · MLP · 헤드 워커 |
| `REC_EXP_CHUNK` | 2048 | 프롬프트 읽기 청크 (체크포인트도 청크 시작마다) |
| `REC_TEMPERATURE` · `REC_TOP_P` · `REC_TOP_K` · `REC_PRESENCE_PENALTY` | 1.0 · 0.95 · 20 · 0 | 요청에 그 값이 없을 때 쓸 기본값. 요청이 보낸 값이 이긴다. 범위 밖이면 서버가 켜지지 않음 |
| `REC_OUTPUT_LIMIT` | 없음 | 답 하나의 최대 토큰. 요청이 더 크게 달라고 해도 이 안으로 |
| `REC_METRICS` | 없음 | 요청마다 토큰 수 · 시간 · 속도 · 메모리를 이 JSONL 에 한 줄씩. 생성된 토큰(대화 내용)은 남기지 않음 |

## 해 보고 뺀 것

| 시도 | 결과 |
|---|---|
| 전역 층 KV 를 int8 로 | 디코드 20~32% 느려짐 (bf16 유지) |
| 추측 디코딩 (Flint-DFlash2 초안 모델) | rec 의 여러 토큰 검증 경로가 상주 커널 1토큰보다 2.8~9.7배 느려 이득 없음 |
| 프롬프트 청크 4096 | 읽기 속도 그대로, 체크포인트만 성겨짐 (2048 유지) |
| 창 층 어텐션 조각 수 16 · 64 | 16은 디코드 2% 느리고 답이 바뀜, 64는 32와 같음 (32 유지) |

## 검증

- 워커 변경: rec 의 상주 디코드 검증(상주 · 스칼라 · MLX 비교)을 워커 76/64/64 설정에서 통과 — KV int8 시험 빌드에서 돌린 것이고, 지금 bf16 빌드로는 다시 돌리지 않았다.
- 같은 질문(greedy)에 같은 답: 설정을 바꿔 가며 비교할 때 답 해시가 같음을 확인.
- 디코드 A/B: 같은 32K · 64K 요청을 원래 · 튜닝 런타임에 번갈아 보냄 (위 결과표).
- Codex 앱 중계기 (2026-10-28, ChatGPT 앱 26.1023 의 codex 0.165.0-alpha): 가짜 `CODEX_HOME`(가짜 ChatGPT 로그인) + 시험 포트의 중계기 + 가짜 OpenAI 로 기본 모델 → 가짜 OpenAI, `-m flint` → Flint, 도구(cat) → 맞는 답, 목록 캐시 끝에 flint 와 ETag 꼬리표, 압축 한도 3000 으로 두 번 압축된 뒤에도 맞는 답. 그다음 앱의 연결 버튼으로 실제 `~/.codex` 에 연결해 ChatGPT 앱 목록 맨 아래 "Flint (local)", 기본 모델(실제 OpenAI) · flint 답, 끊기 뒤 config.toml 이 연결 전과 바이트 동일함을 확인.
- Claude 앱 연결 (2026-10-28, Claude 앱 3.6556.2): 서버를 새 오버레이로 다시 깔고 `claude-flint` · `flint` 는 200, 다른 이름은 400. Claude 앱을 끄고 앱의 연결 버튼 → `Claude-3p` 에 600 파일 3개, Claude 앱을 켜니 "Gateway 사용 중" 과 모델 Flint (local), "1+1은?" 에 2 (요청이 Flint 서버 기록에 남음). 앱을 끄고 끊기 → `Claude-3p` 와 로그 폴더가 휴지통으로, 다시 켜니 평소 Claude(계정 · 세션 목록 그대로)이고 `Claude-3p` 는 다시 생기지 않음.
- `pnpm test` — `python3` 로 `chat.py` 프롬프트 정리 · `responses.py` 변환 · `install.py` 최소 사양 · 중계기 시험 (중계기 시험은 macOS 의 Python 3.14+ 에서만 돌고 다른 곳에선 건너뜀). rec 이 있어야 하는 `test_disk_cache_pieces.py` · `test_server_defaults.py` 는 빠져 있어 Mac 의 튜닝 런타임 python 으로 따로 돌린다.
- `python3 tests/mac_disk_cache_check.py` — Mac 에서 디스크 캐시 확인. 포트 11509 에 서버를 띄워, 디스크에서 불러와 이어 읽은 답이 RAM 캐시로 읽은 답과 같은지(greedy), 대화가 길어져도 파일이 하나인지 본다. 도는 동안 `rec stop` 하고 끝나면 `rec-serve-tuned` 를 다시 띄운다(이때 `serve.log` 를 새로 쓰고 앱 설정 없이 켜지므로, 앱을 쓰면 끝난 뒤 앱에서 다시 켠다).

## 지우기

```sh
rec stop
mv ~/.local/share/rec-tuned ~/.local/bin/rec-serve-tuned ~/.Trash/
mv ~/.cache/rec-kv ~/.Trash/      # 디스크 캐시도 지울 때
rec serve                         # 원래 런타임
```

## 파일

| 경로 | 역할 |
|---|---|
| `install.py` | 원래 런타임 복사 → `overlay/` 덮어쓰기 → manifest 갱신 → `rec-serve-tuned` 생성 |
| `overlay/rec_mlx/` | 바꾼 rec 파일 7개 + 새 파일 `disk_cache.py` · `responses.py` |
| `tests/test_chat.py` | `chat.py` 시험 |
| `tests/mac_disk_cache_check.py` | Mac 디스크 캐시 확인 |
| `tests/test_disk_cache_pieces.py` | 디스크 캐시가 KV 를 조각(최대 2^29 값)으로 나누는 규칙 시험 (Mac 의 튜닝 런타임 python 으로). 한 배열이 2^31 값을 넘으면 MLX 가 넘쳐 약 13만 토큰 넘는 대화가 저장되지 않던 것을 막는다 |
| `tests/test_server_defaults.py` | `server.py` 기본 샘플링 · 출력 한도 시험, `REC_OUTPUT_LIMIT` 답 길이 상한, 요청 표시 속성이 핸들러 속성(`responses` 등)과 겹치지 않는지, 요청 경로로 보낸 모델 이름 `flint` · `claude-flint` 는 받고 다른 이름은 400 인지 (Mac 의 튜닝 런타임 python 으로) |
| `tests/test_install.py` | `install.py` 의 최소 Mac 조건(칩 · 메모리 48 GiB · 남은 디스크) 시험. 아무 python 으로 |
| `tests/test_responses.py` | Responses 요청 → 채팅, 채팅 답 → Responses JSON · SSE 변환 시험. rec 에 기대지 않아 아무 python 3.10+ 로 |
| `relay/flint_codex_relay.py` | Codex 앱 중계기 (위 "Codex 앱 연결"). 앱이 번들에 넣어 연결할 때 `~/.config/flint/codex-app/` 에 복사 |
| `tests/test_codex_relay.py` | 중계기 시험: flint 목록 항목, 모델별 갈래, ETag, 압축 요약 왕복, 가짜 OpenAI · 가짜 Flint 를 127.0.0.1 에 띄워 요청 전체, 끊은 뒤 앱이 꺼질 때까지 남았다가 스스로 정리. Python 3.14+ (`/opt/homebrew/bin/python3 -m unittest tests.test_codex_relay`) |
| `app/` | Flint.app: `Sources/FlintCore`(서버 · 옵션 · 연결 · 맞추기 로직, 시험 있음), `Sources/Flint`(화면) |
| `NOTICE`, `LICENSE` | rec 에서 가져와 바꾼 파일 목록, Apache-2.0 |

## 라이선스

rec 은 Apache-2.0 이고, 이 폴더도 Apache-2.0 (`LICENSE`, `NOTICE`).
