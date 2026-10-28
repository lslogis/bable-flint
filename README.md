# bable-flint

[English](README.en.md)

Mac 에서 로컬 코딩 모델 **Flint** 를 빠르게 돌리고, 쓰던 코딩 앱을 거기에 붙여 주는 앱입니다.
[rec](https://github.com/recursive-oss/rec) 0.1.1 위에 속도를 맞춘 서버를 설치하고 켜고 끕니다. Pi · Claude Code · Codex CLI · ChatGPT 앱의 Codex · Claude 앱 · opencode 를 버튼 하나로 연결하고, 끊으면 원래대로 돌려 놓습니다.

측정: 생성 속도가 rec 원래 런타임보다 32K 문맥에서 107.2 → 129.7 tok/s(+21%), 64K 에서 74.0 → 91.5 tok/s(+24%).
측정한 Mac: M4 Max(GPU 40코어) · 메모리 48 GB · macOS 27.2 · rec 0.1.1 · flint-int4.

## 필요한 것

- Apple Silicon Mac, 칩은 **M4 Pro · M4 Max · M5 Pro · M5 Max** 중 하나
- 메모리 **48 GB 이상**
- 디스크 여유 **32 GB 이상** (모델 21 GB + 런타임 + 대화 캐시). 모델을 이미 받아 두었으면 10 GB.
- macOS 15 이상
- [rec](https://github.com/recursive-oss/rec) **0.1.1** — 먼저 설치해 두세요. 다른 버전이면 설치가 멈춥니다.
- `python3` (Xcode Command Line Tools 에 들어 있음)
- ChatGPT 앱의 Codex 를 연결할 때만: Python 3.14 이상 (예: Homebrew 의 `python3`)

사양이 안 맞으면 앱이 무엇이 모자란지 보여 주고, 아무것도 받거나 바꾸지 않습니다.

## 설치

1. [Releases](../../releases) 에서 `bable-flint-<버전>.dmg` 를 받습니다.
2. dmg 를 열고 `bable-flint` 를 `Applications` 폴더로 끌어 놓습니다.
3. 처음 열 때: 이 앱은 Apple 개발자 서명이 없어서 macOS 가 "확인할 수 없다"며 막습니다.
   **시스템 설정 → 개인정보 보호 및 보안** 맨 아래의 **그래도 열기** 를 누르고, 한 번 더 **열기** 를 누르세요. 다음부터는 바로 열립니다.
   터미널이 편하면 이 명령 한 줄로도 됩니다:

   ```sh
   xattr -dr com.apple.quarantine /Applications/bable-flint.app
   ```

## 처음 켤 때

앱이 순서대로 안내합니다.

0. 이 앱을 만든 유튜브 채널 이름을 묻습니다(대소문자 상관없음). 한 번 맞히면 다시 묻지 않습니다.
1. 이 Mac 이 사양에 맞는지 확인합니다.
2. rec 이 있는지 확인합니다.
3. **받기** — Flint 모델을 받습니다(약 21 GB).
4. **설치** — 속도를 맞춘 런타임을 설치합니다. rec 의 원래 런타임은 그대로 둡니다.

끝나면 **서버 켜기** 를 누릅니다.

## 매일 쓰기

- 왼쪽 패널: 서버 상태, 마지막 요청의 속도(tok/s), 메모리, 대화 캐시, 연결된 앱. 아래에 **서버 켜기 · 다시 시작 · 서버 끄기**.
- 오른쪽 탭: **켤 때 옵션 · 앱 연결 · 관리**.
- 오른쪽 위: **한/EN** 언어 전환, 확대 · 축소(−, 100%, + / ⌘+ ⌘− ⌘0).
- 창을 닫아도 서버는 계속 돕니다. Dock 아이콘은 숨고, 상단바의 불꽃 아이콘이 남습니다.
- 상단바 아이콘: 상태와 속도, 서버 켜기 · 끄기 · 다시 시작, 창 열기, 주소 복사, 종료.
- 서버가 켜진 채 종료하면 묻습니다: **서버는 켜 두고 종료 · 서버도 끄고 종료 · 취소**.
- 서버는 이 Mac 안(127.0.0.1)에서만 요청을 받습니다.

## 켤 때 옵션

서버를 켤 때 쓰는 값입니다. 바꾼 값은 서버를 다시 켜면 적용됩니다.

- **모드**: 빠름(속도를 맞춘 런타임) · 기본(rec 원래 런타임, 약 20% 느림)
- **문맥**, **출력 한도**, **샘플링**
- **워커**: 생성 워커, 보조 워커, 읽기 묶음
- **SSD 대화 캐시**: 폴더, 최대 크기. 같은 대화를 이어 물을 때 앞부분을 다시 읽지 않습니다. 서버를 다시 켠 뒤에도 됩니다.
- **기본값으로**: 처음 값으로 되돌립니다.

### 자동 튜닝

**자동 튜닝** 은 값마다 서버를 다시 켜 속도를 직접 재고, 이 Mac 에서 가장 빠른 워커와 읽기 묶음을 제안합니다.

- 5~20분 걸립니다. 그동안 서버를 쓸 수 없습니다.
- 지금 값보다 3% 넘게 빠를 때만 바꾸자고 합니다. 같은 질문에 답이 달라지는 값은 버립니다.
- 결과는 바로 적용하거나 옵션에만 넣을 수 있습니다. 중지하면 원래 값으로 다시 켭니다.

## 앱 연결

**앱 연결** 탭에서 앱마다 무엇이 바뀌는지 보고 **이대로 연결** 을 누릅니다. **연결 끊기** 는 연결 전으로 되돌립니다. 만든 파일은 휴지통으로 옮깁니다. Pi 와 opencode 는 고를 모델 하나가 늘 뿐이라 끊기 버튼이 없고, 이 앱을 지울 때 원래대로 돌려 놓습니다.

- **Pi** (추천): Pi 모델 목록(`~/.pi/agent/models.json`)에 Flint 공급자 하나만 더합니다. 키는 파일에 넣지 않고 쓸 때마다 rec 키 파일에서 읽습니다. `/model` 에서 Flint 를 고르거나 `pi --provider flint --model flint` 로 켭니다. Pi 가 없으면 공식 설치 명령(`curl -fsSL https://pi.dev/install.sh | sh`)을 보여 줍니다. 앱이 대신 설치하지는 않습니다.
- **Claude Code**: 평소 `claude` 는 그대로 두고, `claude-flint` 명령을 따로 만듭니다. 스킬 · MCP · 대화 기록은 평소 claude 와 따로입니다.
- **Codex CLI**: 평소 `codex` 는 그대로 두고, `codex-flint` 명령을 따로 만듭니다. 웹 검색 · 하위 에이전트 · 그림 읽기는 안 됩니다.
- **ChatGPT 앱의 Codex**: 모델 목록 맨 아래에 **Flint (local)** 을 더합니다. `~/.codex/config.toml` 에 한 줄을 넣고, 이 Mac 안에서 도는 중계기가 Flint 요청만 Flint 로 보냅니다. ChatGPT 앱을 다시 켜야 보입니다. 연결해 둔 동안 Codex 의 다른 요청도 이 중계기를 지납니다.
- **Claude 앱**: 앱을 3P 모드로 바꿔 **Flint (local)** 을 씁니다. 연결 · 끊기 전에 Claude 앱을 ⌘Q 로 끄세요. 연결해 둔 동안 claude.ai 로그인과 대화는 보이지 않고, 끊으면 돌아옵니다.
- **opencode**: 설정에 공급자 하나만 더합니다. `/models` 에서 Flint 를 고르거나 `opencode -m flint/flint` 로 켭니다.

다른 앱은 직접 넣으면 됩니다. 같은 탭에 주소(OpenAI 호환 `http://127.0.0.1:11500/v1`, Anthropic 호환 `http://127.0.0.1:11500`), 모델 이름 `flint`, 키가 있습니다. 키는 **보기 · 가리기**, **새 키** 로 바꿀 수 있고, **연결 시험** 으로 서버가 답하는지 봅니다. 새 키를 만들면 지금 키는 바로 막히고, 연결한 앱은 같이 고쳐집니다.

## 관리

- **자동 켜기**: 로그인할 때 서버와 상단바 아이콘을 켭니다.
- **점검**: rec 과 설치된 런타임을 검사합니다. 그래도 이상하면 **다시 설치**.
- **로그**: 서버 로그를 실시간으로 봅니다. **오류만** 으로 거를 수 있습니다.

## 지우기

**관리 → 앱 지우기** 를 쓰세요. 서버를 끄고, 연결한 앱을 모두 연결 전으로 되돌린 뒤, 아래를 휴지통으로 옮깁니다.

- rec 과 모델
- 대화 캐시
- 속도를 맞춘 런타임
- 앱 설정과 연결 파일
- 앱 자신

휴지통을 비우기 전까지는 되살릴 수 있습니다.

---

## 개발자용

### 빌드 · 시험

소스에서 직접 빌드하려면 Swift 6(Xcode 16 이상)이 필요합니다.

```sh
cd app
swift test          # FlintCore 시험
./build.sh          # build/bable-flint.app (ad-hoc 서명)
open build/bable-flint.app

cd ..
pnpm test           # = python3 -E -s -m unittest tests.test_chat tests.test_responses tests.test_install tests.test_codex_relay
```

- `tests.test_codex_relay` 는 macOS 의 Python 3.14 이상에서만 돌고, 다른 곳에서는 건너뜁니다.
- rec 런타임이 있어야 하는 시험은 Mac 에서 따로 돌립니다:
  `~/.local/share/rec-tuned/rt/python/bin/python3.12 tests/test_server_defaults.py` (`test_disk_cache_pieces.py` 도 같음)
- 앱 없이 설치 · 실행: `python3 install.py --check`(확인만), `python3 install.py`, 그다음 `rec stop; rec-serve-tuned`.

### 환경 변수

앱이 서버를 켤 때 넘기는 값입니다. rec 원래 런타임은 무시합니다.

| 변수 | 없을 때 | 뜻 |
|---|---|---|
| `REC_DISK_CACHE` | `~/.cache/rec-kv` | 대화 캐시 폴더. `0` 이면 끔 |
| `REC_DISK_CACHE_GB` | 32 | 대화 캐시 최대 크기(GiB). 넘으면 오래된 것부터 지움 |
| `REC_DISK_CACHE_MIN` | 8192 | 이보다 짧은 프롬프트는 저장하지 않음 |
| `REC_EXP_WORKERS` | 칩 기본값 (M4 Max · M5 Max `76,64,64`) | 생성 · MLP · 헤드 워커 |
| `REC_EXP_CHUNK` | 칩 기본값 | 프롬프트 읽기 묶음 |
| `REC_TEMPERATURE` · `REC_TOP_P` · `REC_TOP_K` · `REC_PRESENCE_PENALTY` | rec 기본값 (앱은 1.0 · 0.95 · 20 · 0) | 요청에 값이 없을 때 쓰는 샘플링. 요청이 보낸 값이 이김. 범위 밖이면 서버가 켜지지 않음 |
| `REC_OUTPUT_LIMIT` | 없음 | 답 하나의 최대 토큰 |
| `REC_METRICS` | 없음 | 요청마다 토큰 수 · 시간 · 속도 · 메모리를 이 JSONL 파일에 기록. 대화 내용은 남기지 않음 |

### 파일

- `app/` — bable-flint.app. `Sources/FlintCore`(서버 · 옵션 · 연결 · 튜닝 로직, `Tests/` 에 시험), `Sources/Flint`(화면), `build.sh`
- `install.py` — rec 런타임을 복사하고 `overlay/` 를 덮어 `~/.local/share/rec-tuned` 와 `~/.local/bin/rec-serve-tuned` 를 만듦
- `overlay/rec_mlx/` — 바꾼 rec 파일
- `relay/flint_codex_relay.py` — ChatGPT 앱 Codex 연결용 중계기
- `tests/` — Python 시험

### rec 에서 바꾼 파일

rec 0.1.1 의 파일 7개를 바꾸고 2개(`disk_cache.py`, `responses.py`)를 더했습니다. 바꾼 파일마다 맨 위에 `Modified from recursive-oss/rec 0.1.1` 한 줄이 있습니다. 목록과 무엇을 바꿨는지는 [`NOTICE`](NOTICE) 에 있습니다.

## 라이선스

Apache-2.0 (`LICENSE`, `NOTICE`). rec 도 Apache-2.0 입니다. 앱에 든 글꼴 Pretendard 는 SIL Open Font License 1.1 입니다.
