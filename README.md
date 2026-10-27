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
| 프롬프트 체크포인트 2개 → 64개 | `prefix.py` | 긴 대화에서 앞부분이 바뀌어도 가까운 체크포인트부터 다시 읽음 (recursive-oss/rec PR #24 제안과 같은 값) |
| Claude Code 과금 도장 고정 | `chat.py` | 시스템 프롬프트 맨 앞 `cch=` · `cc_version` 끝자리가 요청마다 바뀌어 캐시를 못 씀. `f` 로 고정 |
| opencode `<env>` 블록을 첫 사용자 메시지로 | `chat.py` | 폴더 · 날짜가 시스템 프롬프트 앞쪽에 있어 매번 캐시가 깨짐. `REC_ENV_TO_USER=0` 이면 그대로 |
| 대화 캐시를 SSD 에 저장 · 불러오기 | `disk_cache.py`(새 파일), `engine.py`, `scheduler.py` | 응답이 끝난 뒤 쉬는 시간에 저장(디코드 속도에 영향 없음), RAM 캐시보다 길게 맞는 저장본이 있으면 불러옴 |

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

## 스위치 (환경 변수)

| 변수 | 기본 | 뜻 |
|---|---|---|
| `REC_DISK_CACHE` | `~/.cache/rec-kv` | 디스크 캐시 폴더, `0` 이면 끔 |
| `REC_DISK_CACHE_GB` | 32 | 디스크 캐시 최대 크기 (넘으면 오래된 것부터 지움) |
| `REC_DISK_CACHE_MIN` | 8192 | 이보다 짧은 프롬프트는 저장 안 함 |
| `REC_ENV_TO_USER` | 1 | `0` 이면 opencode `<env>` 를 옮기지 않음 |
| `REC_EXP_WORKERS` | `76,64,64` | 디코드 · MLP · 헤드 워커 |
| `REC_EXP_CHUNK` | 2048 | 프롬프트 읽기 청크 (체크포인트도 청크 시작마다) |

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
- `pnpm test` — `chat.py` 프롬프트 정리 시험 (Mac 없이 어디서나, transformers 없으면 대체물로).
- `python3 tests/mac_disk_cache_check.py` — Mac 에서 디스크 캐시 확인. 포트 11501 에 서버를 띄워, 디스크에서 불러와 이어 읽은 답이 RAM 캐시로 읽은 답과 같은지(greedy), 대화가 길어져도 파일이 하나인지 본다. 도는 동안 `rec stop` 하고 끝나면 `rec-serve-tuned` 를 다시 띄운다.

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
| `overlay/rec_mlx/` | 바꾼 rec 파일 6개 + 새 파일 `disk_cache.py` |
| `tests/test_chat.py` | `chat.py` 시험 |
| `tests/mac_disk_cache_check.py` | Mac 디스크 캐시 확인 |
| `NOTICE`, `LICENSE` | rec 에서 가져와 바꾼 파일 목록, Apache-2.0 |

## 라이선스

rec 은 Apache-2.0 이고, 이 폴더도 Apache-2.0 (`LICENSE`, `NOTICE`).
