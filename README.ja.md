<p align="center"><img src="docs/icon.png" width="128" alt="bable-flint アイコン"></p>

<h1 align="center">bable-flint</h1>

<p align="center"><b>ローカルのコーディングモデル Flint を、Mac でもっと速く。<br>いつものコーディングアプリには、ボタンひとつで。</b></p>

<p align="center">
  <a href="README.md">한국어</a> · <a href="README.en.md">English</a> · <b>日本語</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/macOS-15%2B-111?logo=apple" alt="macOS 15+">
  <img src="https://img.shields.io/badge/Apple%20Silicon-M4%20%C2%B7%20M5%20Pro%2FMax-5b4fd6" alt="M4 · M5 Pro/Max">
  <img src="https://img.shields.io/badge/memory-48%20GB%2B-5b4fd6" alt="48 GB+">
  <img src="https://img.shields.io/badge/license-Apache--2.0-2f7d4f" alt="Apache-2.0">
</p>

<p align="center"><img src="docs/screenshot.png" width="820" alt="bable-flint の画面"></p>

> アプリの画面は韓国語と英語に対応しています（右上の **한/EN** で切り替え）。日本語の表示には対応していません。

## ひと目で

| | |
|---|---|
| ⚡ **より速い生成** | M4 Max（40 コア GPU・48 GB）で自動チューニング後に実測した生成速度：短い会話で **約 226 tok/s**、2 万 9 千トークンの文脈で **154 tok/s**、5 万 9 千トークンで **113 tok/s**。実際のコーディング作業（Pi）では平均 約 150 tok/s、最高 203 tok/s |
| 📦 **これひとつで完結** | Flint のエンジン（rec 0.1.1）がアプリに入っています。別途インストールするものも、合わせるバージョンもありません |
| 🔌 **6 つのアプリをボタンひとつで** | Pi・Claude Code・Codex CLI・ChatGPT アプリの Codex・Claude アプリ・opencode |
| 💾 **SSD 会話キャッシュ** | 長い会話の続きを聞くとき、前半を読み直しません。サーバーを再起動したあとでも |
| 🎛 **自動チューニング** | この Mac でいちばん速いワーカー数と読み込み単位を実測して見つけます |
| 🔒 **この Mac の中だけで** | サーバーは 127.0.0.1 からのリクエストだけを受け付けます。会話が外に出ることはありません |

<sub>測定環境：M4 Max（GPU 40 コア）・メモリ 48 GB・macOS 27.2・flint-int4</sub>

## 必要なもの

| | |
|---|---|
| チップ | Apple Silicon **M4 Pro・M4 Max・M5 Pro・M5 Max** |
| メモリ | **48 GB** 以上 |
| ディスク | 空き **32 GB** 以上（モデル 21 GB＋エンジン＋会話キャッシュ）。モデルがすでにあれば 10 GB |
| macOS | 15 以上 |
| `python3` | Xcode Command Line Tools に含まれています |
| ChatGPT アプリの Codex をつなぐときだけ | Python 3.14 以上（例：Homebrew の `python3`） |

条件を満たさない Mac では、何が足りないかを表示し、何もダウンロード・変更しません。

## インストール

1. [Releases](../../releases) から `bable-flint.dmg` をダウンロードします。**[直接ダウンロード (bable-flint.dmg)](https://github.com/lslogis/bable-flint/releases/latest/download/bable-flint.dmg)**
2. dmg を開き、`bable-flint` を `アプリケーション` フォルダへドラッグします。
3. ブラウザでダウンロードしたアプリは、初回に「開発元を確認できない」として macOS に止められます（Apple の開発者署名がないアプリです）。
   **インストール後、ターミナルでこの 1 行を実行してください。** 止められずにそのまま開きます。

   ```sh
   xattr -dr com.apple.quarantine /Applications/bable-flint.app
   ```

   ターミナルを使わない場合は、一度開こうとしたあと **システム設定 → プライバシーとセキュリティ** のいちばん下にある **このまま開く** を押し、もう一度 **開く** を押してください。次回からはそのまま開きます。

### ターミナルでインストール（「このまま開く」なし）

ターミナルに次を貼り付けると、dmg をダウンロードして `アプリケーション` に入れ、そのまま開きます。`curl` でダウンロードしたファイルには macOS の「インターネットからダウンロード」の印が付かないので、**このまま開く** なしで開きます。

```sh
curl -fL -o /tmp/bable-flint.dmg https://github.com/lslogis/bable-flint/releases/latest/download/bable-flint.dmg \
  && mkdir -p /tmp/bable-flint-dmg \
  && hdiutil attach -nobrowse -quiet -mountpoint /tmp/bable-flint-dmg /tmp/bable-flint.dmg \
  && ditto /tmp/bable-flint-dmg/bable-flint.app /Applications/bable-flint.app \
  && hdiutil detach -quiet /tmp/bable-flint-dmg \
  && open /Applications/bable-flint.app
```

0.1.6 からはアプリが新しい版を知らせます。**管理** タブの **アップデート** ボタンでアプリが入れ替わり、動いていたサーバーは新しい版で再起動します。上のコマンドで上書きしても構いません(先にアプリを終了してください)。

## 初回起動

アプリが順に案内します。

| ステップ | 内容 |
|---|---|
| 0 | このアプリを作った YouTube チャンネルの名前を聞かれます。**`bable`** と入力してください（大文字・小文字は問いません）。一度正解すれば、もう聞かれません |
| 1 | この Mac が条件を満たすか確認します |
| 2 | アプリに入っているエンジン（rec 0.1.1）を、アプリ専用フォルダ `~/.local/share/bable-flint` に置きます。自動です |
| 3 | **ダウンロード** — Flint のモデル（約 21 GB）をダウンロードします。rec を別途インストールしてモデルを取得済みなら、それを複製して使います（APFS の複製なので容量は増えません） |
| 4 | **インストール** — エンジン本来のランタイムを展開し、そのコピーに高速化を重ねます |

終わったら **サーバーを起動** を押します。

> rec を別途使っていても問題ありません。このアプリは自分のフォルダのエンジンだけを使い、別途インストールされた rec は、バージョンにかかわらず読むことも変えることもしません。どちらも既定でポート 11500 を使うので、同時に起動するのはひとつだけにしてください。

## 毎日の使い方

- **左のパネル**：サーバーの状態、直近のリクエストの速度（tok/s）、メモリ、会話キャッシュ、接続中のアプリ。下に **起動・再起動・停止**。
- **右のタブ**：**起動オプション・アプリ接続・管理**。
- **右上**：**한/EN** の言語切り替え、拡大・縮小（−、100%、+ / ⌘+ ⌘− ⌘0）。
- ウィンドウを閉じてもサーバーは動き続けます。Dock のアイコンは隠れ、メニューバーの炎のアイコンが残ります。
- **メニューバーのアイコン**：状態と速度、サーバーの起動・停止・再起動、ウィンドウを開く、アドレスをコピー、終了。
- サーバーが動いたまま終了すると確認されます：**サーバーは動かしたまま終了・サーバーも止めて終了・キャンセル**。

## 起動オプション

サーバーを起動するときの値です。変更はサーバーを再起動すると反映されます。

いちばん上の **ログイン時にアプリを自動起動**（アプリは韓国語・英語表示）をオンにすると、ログインしたときにアプリとサーバーが起動します。これだけは押した瞬間に反映されます。

- **コンテキスト・出力上限・サンプリング**
- **ワーカー**：生成ワーカー、補助ワーカー、読み込み単位
- **SSD 会話キャッシュ**：フォルダ、最大サイズ
- **初期値に戻す**：最初の値に戻します

### 自動チューニング

候補の値ごとにサーバーを再起動して速度を実測し、この Mac でいちばん速いワーカーと読み込み単位を提案します。

- 5〜20 分かかり、その間サーバーは使えません。
- 今の値より 3% を超えて速いときだけ変更を提案します。同じ質問への答えが変わる値は除外します。
- 今の値でサーバーが起動しないときは、rec の既定値 `36,32,32` から測ります。
- 結果はすぐ適用するか、オプションに入れておくだけにもできます。中止すると元の値で再起動します。
- チューニング中にアプリを終了すると、チューニングとサーバーを止めます。次回は元の値で起動します。

## アプリ接続

<p align="center"><img src="docs/screenshot-apps.png" width="820" alt="アプリ接続の画面"></p>

**アプリ接続** タブで、アプリごとに何が変わるかを確かめてから **この内容で接続** を押します。

| アプリ | 接続すると | 使い方 |
|---|---|---|
| **Pi**（おすすめ） | Pi のモデル一覧（`~/.pi/agent/models.json`）に Flint のプロバイダーをひとつ加えます。キーはファイルに書かず、使うたびにキーファイルから読みます | `/model` で Flint、または `pi --provider flint --model flint` |
| **Claude Code** | いつもの `claude` はそのままに、`claude-flint` コマンドを別に作ります。スキル・MCP・会話履歴も別です。テーマなど初回の質問への答えと、信頼済みのフォルダはいつもの `claude` から引き継ぐので、もう一度聞かれません。先頭の `[claude-code:unrecognized_model]` 警告は flint が Claude Code のモデル一覧にないために出るもので、動作には関係ありません | `claude-flint` |
| **Codex CLI** | いつもの `codex` はそのままに、`codex-flint` コマンドを別に作ります。ウェブ検索・サブエージェント・画像入力は使えません | `codex-flint` |
| **ChatGPT アプリの Codex** | モデル一覧のいちばん下に **Flint (local)** を加えます（`~/.codex/config.toml` に 1 行）。この Mac の中で動く中継が Flint 宛てのリクエストだけを Flint に送ります。接続中は Codex のほかのリクエストもこの中継を通ります | ChatGPT アプリを再起動してからモデルを選ぶ |
| **Claude アプリ** | アプリを 3P モードに切り替えて **Flint (local)** を使います。接続中は claude.ai のログインと会話が隠れ、切断すると戻ります | 接続・切断の前に Claude アプリを ⌘Q で終了 |
| **opencode** | 設定にプロバイダーをひとつ加えます | `/models` で Flint、または `opencode -m flint/flint` |

- **接続を解除** すると接続前の設定に戻し、作ったファイルはゴミ箱へ移します。
- Pi と opencode は選べるモデルがひとつ増えるだけなので、解除ボタンはありません。このアプリを削除するときに元に戻します。
- Pi が入っていなければ、公式のインストールコマンド（`curl -fsSL https://pi.dev/install.sh | sh`）を表示します。

**ほかのアプリ** は手で設定できます。同じタブに必要なものがそろっています。

| | |
|---|---|
| OpenAI 互換アドレス | `http://127.0.0.1:11500/v1` |
| Anthropic 互換アドレス | `http://127.0.0.1:11500` |
| モデル名 | `flint` |
| キー | **表示・非表示**、**新しいキー**（今のキーはすぐ無効になり、接続中のアプリも合わせて更新）、**接続テスト** |

## 管理

- **点検**：エンジンとインストールしたランタイムを検査します。それでもおかしければ **再インストール**。
- **ログ**：サーバーのログをリアルタイムで表示します。**エラーのみ** で絞り込めます。

## 削除

**管理 → アプリを削除** を使ってください。サーバーを止め、接続したアプリをすべて接続前に戻してから、次のものをゴミ箱へ移します。

- アプリ専用フォルダ `~/.local/share/bable-flint`（エンジン・モデル・高速化・キー・ログ）
- 会話キャッシュ
- アプリの設定と接続ファイル
- アプリ本体

ChatGPT アプリで Flint を選んだときにそのアプリが `~/.codex/config.toml` に保存した `model = "flint"` の行も取り除きます（ほかの行はそのまま）。

ゴミ箱を空にするまでは元に戻せます。別途インストールされた rec には触れません。

---

<details>
<summary><b>開発者向け</b></summary>

### しくみ

- エンジンは [rec](https://github.com/recursive-oss/rec) 0.1.1 です。実行ファイルを変更せずにアプリへ同梱し（`Contents/Resources/flint-mac/rec`）、アプリ専用フォルダで動かします（`XDG_DATA_HOME=~/.local/share/bable-flint`）。
- 高速化は rec 0.1.1 の Python エンジンのファイルを 7 つ変更し、2 つ（`disk_cache.py`、`responses.py`）を加えたものです。`install.py` が rec の実行ファイルに入っているランタイムを展開し、そのコピーに `overlay/` を重ねます。
- 変更したファイルの先頭には `Modified from recursive-oss/rec 0.1.1` の 1 行があり、一覧と変更内容は [`NOTICE`](NOTICE) にあります。

### ビルド

Swift 6（Xcode 16 以上）が必要です。

```sh
cd app
./build.sh          # build/bable-flint.app（ad-hoc 署名、rec 0.1.1 同梱）
open build/bable-flint.app
```

- `build.sh` は rec 0.1.1 の実行ファイルをこの Mac にあるコピーから取り、なければ rec のリリースからダウンロードします。どちらでも SHA-256 が一致する必要があります。
- アプリなしでのインストール・実行：rec 0.1.1 を `~/.local/share/bable-flint/bin/rec` に置いてから `python3 install.py --check`（確認のみ）、`python3 install.py`、続いて `~/.local/share/bable-flint/bin/rec-serve-tuned`。

### 環境変数

アプリがサーバーを起動するときに渡す値です。エンジン本来のランタイムは無視します。

| 変数 | 未設定のとき | 意味 |
|---|---|---|
| `REC_DISK_CACHE` | `~/.cache/rec-kv` | 会話キャッシュのフォルダ。`0` で無効 |
| `REC_DISK_CACHE_GB` | 32 | 会話キャッシュの最大サイズ（GiB）。超えると古いものから削除 |
| `REC_DISK_CACHE_MIN` | 8192 | これより短いプロンプトは保存しない |
| `REC_EXP_WORKERS` | チップの既定値（M5 Max と GPU 40 コアの M4 Max は `76,64,64`、ほかは rec の `36,32,32`） | 生成・MLP・ヘッドのワーカー |
| `REC_EXP_CHUNK` | チップの既定値 | プロンプトの読み込み単位 |
| `REC_TEMPERATURE`・`REC_TOP_P`・`REC_TOP_K`・`REC_PRESENCE_PENALTY` | rec の既定値（アプリは 1.0・0.95・20・0） | リクエストに値がないときのサンプリング。リクエストの値が優先。範囲外だとサーバーは起動しない |
| `REC_OUTPUT_LIMIT` | なし | 1 回の回答の最大トークン数 |
| `REC_METRICS` | なし | リクエストごとのトークン数・時間・速度・メモリをこの JSONL ファイルに記録。会話の内容は残さない |

### ファイル

- `app/` — bable-flint.app。`Sources/FlintCore`（サーバー・オプション・接続・チューニングのロジック）、`Sources/Flint`（画面）、`build.sh`
- `install.py` — エンジンのランタイムを展開して `overlay/` を重ね、`~/.local/share/bable-flint/rec-tuned` と `bin/rec-serve-tuned` を作る
- `overlay/rec_mlx/` — 変更した rec のファイル
- `relay/flint_codex_relay.py` — ChatGPT アプリの Codex 接続用の中継

</details>

## ライセンス

Apache-2.0（`LICENSE`、`NOTICE`）。アプリに同梱の rec 0.1.1 も Apache-2.0 です。フォント Pretendard は SIL Open Font License 1.1 です。
