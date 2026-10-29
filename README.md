<p align="center"><img src="docs/icon.png" width="128" alt="bable-flint 아이콘"></p>

<h1 align="center">bable-flint</h1>

<p align="center"><b>로컬 코딩 모델 Flint 를 Mac 에서 더 빠르게.<br>쓰던 코딩 앱에는 버튼 하나로.</b></p>

<p align="center">
  <b>한국어</b> · <a href="README.en.md">English</a> · <a href="README.ja.md">日本語</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/macOS-15%2B-111?logo=apple" alt="macOS 15+">
  <img src="https://img.shields.io/badge/Apple%20Silicon-M4%20%C2%B7%20M5%20Pro%2FMax-5b4fd6" alt="M4 · M5 Pro/Max">
  <img src="https://img.shields.io/badge/memory-48%20GB%2B-5b4fd6" alt="48 GB+">
  <img src="https://img.shields.io/badge/license-Apache--2.0-2f7d4f" alt="Apache-2.0">
</p>

<p align="center"><img src="docs/screenshot.png" width="820" alt="bable-flint 화면"></p>

## 한눈에

| | |
|---|---|
| ⚡ **더 빠른 생성** | M4 Max(40코어 GPU · 48 GB)에서 자동 튜닝 뒤 잰 생성 속도: 짧은 대화 **약 226 tok/s**, 2만 9천 토큰 문맥 **154 tok/s**, 5만 9천 토큰 **113 tok/s**. 실제 코딩 작업(Pi)에서는 평균 약 150 tok/s, 최고 203 tok/s |
| 📦 **이것 하나로 끝** | Flint 엔진(rec 0.1.1)이 앱에 들어 있습니다. 따로 깔 것도, 버전 맞출 것도 없습니다 |
| 🔌 **앱 6개를 버튼 하나로** | Pi · Claude Code · Codex CLI · ChatGPT 앱의 Codex · Claude 앱 · opencode |
| 💾 **SSD 대화 캐시** | 긴 대화를 이어 물을 때 앞부분을 다시 읽지 않습니다. 서버를 다시 켠 뒤에도요 |
| 🎛 **자동 튜닝** | 이 Mac 에서 가장 빠른 워커 수와 읽기 묶음을 직접 재서 찾습니다 |
| 🔒 **이 Mac 안에서만** | 서버는 127.0.0.1 에서만 요청을 받습니다. 대화는 밖으로 나가지 않습니다 |

<sub>측정: M4 Max(GPU 40코어) · 메모리 48 GB · macOS 27.2 · flint-int4</sub>

## 필요한 것

| | |
|---|---|
| 칩 | Apple Silicon **M4 Pro · M4 Max · M5 Pro · M5 Max** |
| 메모리 | **48 GB** 이상 |
| 디스크 | 여유 **32 GB** 이상 (모델 21 GB + 엔진 + 대화 캐시). 모델이 이미 있으면 10 GB |
| macOS | 15 이상 |
| `python3` | Xcode Command Line Tools 에 들어 있음 |
| ChatGPT 앱의 Codex 를 연결할 때만 | Python 3.14 이상 (예: Homebrew 의 `python3`) |

사양이 안 맞으면 앱이 무엇이 모자란지 보여 주고, 아무것도 받거나 바꾸지 않습니다.

## 설치

1. [Releases](../../releases) 에서 `bable-flint.dmg` 를 받습니다. **[바로 받기 (bable-flint.dmg)](https://github.com/lslogis/bable-flint/releases/latest/download/bable-flint.dmg)**
2. dmg 를 열고 `bable-flint` 를 `Applications` 폴더로 끌어 놓습니다.
3. 브라우저로 받은 앱은 처음 열 때 macOS 가 "확인할 수 없는 개발자"라며 막습니다(Apple 개발자 서명이 없는 앱입니다).
   **설치 뒤 터미널에서 이 한 줄을 실행하세요.** 그러면 막히지 않고 바로 열립니다.

   ```sh
   xattr -dr com.apple.quarantine /Applications/bable-flint.app
   ```

   터미널을 쓰지 않으려면 한 번 열어 본 뒤 **시스템 설정 → 개인정보 보호 및 보안** 맨 아래의 **그래도 열기** 를 누르고, 한 번 더 **열기** 를 누르세요. 다음부터는 바로 열립니다.

### 터미널로 설치 (그래도 열기 없이)

터미널에 아래를 붙여 넣으면 dmg 를 받아 `Applications` 에 넣고 바로 엽니다. `curl` 로 받은 파일에는 macOS 의 "인터넷에서 받음" 표시가 붙지 않아서 **그래도 열기** 없이 열립니다.

```sh
curl -fL -o /tmp/bable-flint.dmg https://github.com/lslogis/bable-flint/releases/latest/download/bable-flint.dmg \
  && mkdir -p /tmp/bable-flint-dmg \
  && hdiutil attach -nobrowse -quiet -mountpoint /tmp/bable-flint-dmg /tmp/bable-flint.dmg \
  && ditto /tmp/bable-flint-dmg/bable-flint.app /Applications/bable-flint.app \
  && hdiutil detach -quiet /tmp/bable-flint-dmg \
  && open /Applications/bable-flint.app
```

0.1.6 부터는 앱이 새 판을 알려 줍니다. **관리** 탭의 **업데이트** 버튼을 누르면 앱이 바뀌고, 켜져 있던 서버는 새 판으로 다시 켜집니다. 위 명령으로 덮어 깔아도 됩니다(먼저 앱을 끄세요).

## 처음 켤 때

앱이 순서대로 안내합니다.

| 단계 | 하는 일 |
|---|---|
| 0 | 이 앱을 만든 유튜브 채널 이름을 묻습니다. **`bable`** 을 입력하세요(대소문자 상관없음). 한 번 맞히면 다시 묻지 않습니다 |
| 1 | 이 Mac 이 사양에 맞는지 확인합니다 |
| 2 | 앱에 든 엔진(rec 0.1.1)을 앱 전용 폴더 `~/.local/share/bable-flint` 에 둡니다. 자동입니다 |
| 3 | **받기** — Flint 모델을 받습니다(약 21 GB). rec 을 따로 깔아 모델을 받아 둔 적이 있으면 그걸 복제해서 씁니다(APFS 복제라 공간이 늘지 않음) |
| 4 | **설치** — 엔진의 원래 런타임을 풀고, 그 복사본에 속도 개선을 얹습니다 |

끝나면 **서버 켜기** 를 누릅니다.

> rec 을 따로 깔아 쓰고 있어도 괜찮습니다. 이 앱은 자기 폴더의 엔진만 쓰고, 따로 깔린 rec 은 버전이 무엇이든 읽거나 바꾸지 않습니다. 둘 다 기본 포트 11500 을 쓰니 한 번에 하나만 켜 두세요.

## 매일 쓰기

- **왼쪽 패널**: 서버 상태, 마지막 요청의 속도(tok/s), 메모리, 대화 캐시, 연결된 앱. 아래에 **서버 켜기 · 다시 시작 · 서버 끄기**.
- **오른쪽 탭**: **켤 때 옵션 · 앱 연결 · 관리**.
- **오른쪽 위**: **한/EN** 언어 전환, 확대 · 축소(−, 100%, + / ⌘+ ⌘− ⌘0).
- 창을 닫아도 서버는 계속 돕니다. Dock 아이콘은 숨고 상단바의 불꽃 아이콘이 남습니다.
- **상단바 아이콘**: 상태와 속도, 서버 켜기 · 끄기 · 다시 시작, 창 열기, 주소 복사, 종료.
- 서버가 켜진 채 종료하면 묻습니다: **서버는 켜 두고 종료 · 서버도 끄고 종료 · 취소**.

## 켤 때 옵션

서버를 켤 때 쓰는 값입니다. 바꾼 값은 서버를 다시 켜면 적용됩니다.

맨 위의 **로그인 시 앱 자동 시작** 을 켜면 로그인할 때 앱과 서버가 함께 켜집니다. 이것만은 누르는 즉시 적용됩니다.

- **문맥 · 출력 한도 · 샘플링**
- **워커**: 생성 워커, 보조 워커, 읽기 묶음
- **SSD 대화 캐시**: 폴더, 최대 크기
- **기본값으로**: 처음 값으로 되돌립니다

### 자동 튜닝

값마다 서버를 다시 켜서 속도를 직접 재고, 이 Mac 에서 가장 빠른 워커와 읽기 묶음을 제안합니다.

- 5~20분 걸리고, 그동안 서버를 쓸 수 없습니다.
- 지금 값보다 3% 넘게 빠를 때만 바꾸자고 합니다. 같은 질문에 답이 달라지는 값은 버립니다.
- 지금 값으로 서버가 켜지지 않으면 rec 기본값 `36,32,32` 에서부터 잽니다.
- 결과는 바로 적용하거나 옵션에만 넣을 수 있습니다. 중지하면 원래 값으로 다시 켭니다.
- 튜닝 중에 앱을 끄면 튜닝을 멈추고 서버도 끕니다. 다음에 켜면 원래 값으로 켜집니다.

## 앱 연결

<p align="center"><img src="docs/screenshot-apps.png" width="820" alt="앱 연결 화면"></p>

**앱 연결** 탭에서 앱마다 무엇이 바뀌는지 미리 보고 **이대로 연결** 을 누릅니다.

| 앱 | 연결하면 | 쓰는 법 |
|---|---|---|
| **Pi** (추천) | Pi 모델 목록(`~/.pi/agent/models.json`)에 Flint 공급자 하나를 더합니다. 키는 파일에 적지 않고 쓸 때마다 키 파일에서 읽습니다 | `/model` 에서 Flint, 또는 `pi --provider flint --model flint` |
| **Claude Code** | 평소 `claude` 는 그대로 두고 `claude-flint` 명령을 따로 만듭니다. 스킬 · MCP · 대화 기록도 따로입니다. 테마 같은 첫 실행 질문의 답과 이미 믿은 폴더는 평소 `claude` 에서 가져와 다시 묻지 않습니다. 첫 줄의 `[claude-code:unrecognized_model]` 경고는 Claude Code 목록에 없는 모델이라 나오는 것이고 동작과는 상관없습니다 | `claude-flint` |
| **Codex CLI** | 평소 `codex` 는 그대로 두고 `codex-flint` 명령을 따로 만듭니다. 웹 검색 · 하위 에이전트 · 그림 읽기는 안 됩니다 | `codex-flint` |
| **ChatGPT 앱의 Codex** | 모델 목록 맨 아래에 **Flint (local)** 을 더합니다(`~/.codex/config.toml` 에 한 줄). 이 Mac 안의 중계기가 Flint 요청만 Flint 로 보내고, 연결해 둔 동안 Codex 의 다른 요청도 이 중계기를 지납니다 | ChatGPT 앱을 다시 켠 뒤 모델 고르기 |
| **Claude 앱** | 앱을 3P 모드로 바꿔 **Flint (local)** 을 씁니다. 연결해 둔 동안 claude.ai 로그인과 대화는 숨고, 끊으면 돌아옵니다 | 연결 · 끊기 전에 Claude 앱을 ⌘Q 로 끄기 |
| **opencode** | 설정에 공급자 하나를 더합니다 | `/models` 에서 Flint, 또는 `opencode -m flint/flint` |

- **연결 끊기** 는 연결 전 설정으로 되돌리고, 만든 파일은 휴지통으로 옮깁니다.
- Pi 와 opencode 는 고를 모델이 하나 늘 뿐이라 끊기 버튼이 없습니다. 앱을 지울 때 원래대로 돌려 놓습니다.
- Pi 가 없으면 공식 설치 명령(`curl -fsSL https://pi.dev/install.sh | sh`)을 보여 줍니다.

**다른 앱**은 직접 넣으면 됩니다. 같은 탭에 필요한 것이 다 있습니다.

| | |
|---|---|
| OpenAI 호환 주소 | `http://127.0.0.1:11500/v1` |
| Anthropic 호환 주소 | `http://127.0.0.1:11500` |
| 모델 이름 | `flint` |
| 키 | **보기 · 가리기**, **새 키**(지금 키는 바로 막히고 연결한 앱은 같이 고쳐짐), **연결 시험** |

## 관리

- **점검**: 엔진과 설치된 런타임을 검사합니다. 그래도 이상하면 **다시 설치**.
- **로그**: 서버 로그를 실시간으로 봅니다. **오류만** 으로 거를 수 있습니다.

## 지우기

**관리 → 앱 지우기** 를 쓰세요. 서버를 끄고, 연결한 앱을 모두 연결 전으로 되돌린 뒤, 아래를 휴지통으로 옮깁니다.

- 앱 전용 폴더 `~/.local/share/bable-flint` (엔진 · 모델 · 속도 개선 · 키 · 로그)
- 대화 캐시
- 앱 설정과 연결 파일
- 앱 자신

ChatGPT 앱에서 Flint 를 골랐을 때 그 앱이 `~/.codex/config.toml` 에 저장한 `model = "flint"` 줄도 뺍니다(나머지 줄은 그대로).

휴지통을 비우기 전까지는 되살릴 수 있습니다. 따로 깔린 rec 은 건드리지 않습니다.

---

<details>
<summary><b>개발자용</b></summary>

### 구조

- 엔진은 [rec](https://github.com/recursive-oss/rec) 0.1.1 입니다. 실행 파일을 바꾸지 않고 앱에 담아(`Contents/Resources/flint-mac/rec`) 앱 전용 폴더에서 돌립니다(`XDG_DATA_HOME=~/.local/share/bable-flint`).
- 속도 개선은 rec 0.1.1 의 Python 엔진 파일 7개를 바꾸고 2개(`disk_cache.py`, `responses.py`)를 더한 것입니다. `install.py` 가 rec 실행 파일 안의 런타임을 풀어, 그 복사본에 `overlay/` 를 덮습니다.
- 바꾼 파일마다 맨 위에 `Modified from recursive-oss/rec 0.1.1` 한 줄이 있고, 목록과 바꾼 내용은 [`NOTICE`](NOTICE) 에 있습니다.
- 앱 없이 설치 · 실행: rec 0.1.1 을 `~/.local/share/bable-flint/bin/rec` 에 둔 뒤 `python3 install.py --check`(확인만), `python3 install.py`, 그다음 `~/.local/share/bable-flint/bin/rec-serve-tuned`.

### 환경 변수

앱이 서버를 켤 때 넘기는 값입니다. 엔진의 원래 런타임은 무시합니다.

| 변수 | 없을 때 | 뜻 |
|---|---|---|
| `REC_DISK_CACHE` | `~/.cache/rec-kv` | 대화 캐시 폴더. `0` 이면 끔 |
| `REC_DISK_CACHE_GB` | 32 | 대화 캐시 최대 크기(GiB). 넘으면 오래된 것부터 지움 |
| `REC_DISK_CACHE_MIN` | 8192 | 이보다 짧은 프롬프트는 저장하지 않음 |
| `REC_EXP_WORKERS` | 칩 기본값 (M5 Max 와 GPU 40코어 M4 Max `76,64,64`, 그 밖은 rec 의 `36,32,32`) | 생성 · MLP · 헤드 워커 |
| `REC_EXP_CHUNK` | 칩 기본값 | 프롬프트 읽기 묶음 |
| `REC_TEMPERATURE` · `REC_TOP_P` · `REC_TOP_K` · `REC_PRESENCE_PENALTY` | rec 기본값 (앱은 1.0 · 0.95 · 20 · 0) | 요청에 값이 없을 때 쓰는 샘플링. 요청이 보낸 값이 이김. 범위 밖이면 서버가 켜지지 않음 |
| `REC_OUTPUT_LIMIT` | 없음 | 답 하나의 최대 토큰 |
| `REC_METRICS` | 없음 | 요청마다 토큰 수 · 시간 · 속도 · 메모리를 이 JSONL 파일에 기록. 대화 내용은 남기지 않음 |

### 파일

- `install.py` — 엔진 런타임을 풀고 `overlay/` 를 덮어 `~/.local/share/bable-flint/rec-tuned` 와 `bin/rec-serve-tuned` 를 만듦
- `overlay/rec_mlx/` — 바꾼 rec 파일
- `relay/flint_codex_relay.py` — ChatGPT 앱 Codex 연결용 중계기

</details>

## 라이선스

Apache-2.0 (`LICENSE`, `NOTICE`). 앱에 든 rec 0.1.1 도 Apache-2.0 입니다. 글꼴 Pretendard 는 SIL Open Font License 1.1 입니다.
