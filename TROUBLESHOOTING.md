# アイリス虎の巻 - OpenClaw復旧・運用ガイド

**Version 1.0 | 2026年3月8日**

---

## 📋 確定済みコマンド一覧（ホワイトリスト）

### ✅ 案内許可コマンド

| コマンド | 用途 |
|---------|------|
| `openclaw gateway` | アイリスの起動（ゲートウェイ接続） |
| `openclaw gateway stop` | アイリスの停止 |
| `openclaw gateway --force` | 強制起動（ポート競合時） |
| `openclaw status` | 状態確認 |
| `openclaw doctor --fix` | 自動修復 |
| `openclaw gateway probe` | ネットワーク診断 |
| `openclaw --help` | コマンド一覧の確認 |
| `claude setup-token` | トークン再生成 |
| `openclaw models auth setup-token --provider anthropic` | トークン再設定 |
| `wsl --shutdown` | WSL完全停止（PowerShellで実行） |
| `wsl` | WSL起動 |

### ❌ 案内禁止コマンド（ブラックリスト）

**絶対に使用・案内してはならないコマンド:**
- `openclaw start` - 存在しない
- `openclaw run` - 存在しない
- `openclaw serve` - 存在しない
- `openclaw dev` - 存在しない
- `openclaw start-agent` - 存在しない
- `openclaw restart` - 存在しない

---

## 🔧 トラブルシュートフローチャート

### 基本復旧フロー

| Step | 操作 | 説明 |
|------|------|------|
| 0 | **環境確認** | PowerShell → `wsl` でWSLに入る / WSL → そのまま続行 |
| 1 | `openclaw gateway stop` → `openclaw gateway` | 二重起動の解消（最頻出の原因） |
| 2 | `openclaw doctor --fix` → `openclaw gateway` | ネットワーク詰まりの修復（WSLサスペンド後に多発） |
| 3 | `exit` → `wsl --shutdown` → `wsl` → `openclaw gateway` | WSLごと再起動（フリーズ時） |
| 4 | `openclaw --help` | コマンド一覧確認 → スクリーンショットで相談 |

---

## ⚠️ エラー別対応表

| エラーメッセージ | 原因 | 対処 |
|------------------|------|------|
| Port 18789 is already in use | 二重起動（古いプロセス残存） | `openclaw gateway stop` → `openclaw gateway` |
| TypeError: fetch failed | WSLサスペンド後のネットワーク断絶 | `openclaw doctor --fix` またはWSL再起動 |
| DiscordMessageListener timed out | ネットワーク不通 | WSL再起動 |
| Slow listener detected (35 seconds) | ネットワーク遅延 | `openclaw doctor --fix` |
| command not found: openclaw | PowerShellで実行している | `wsl` でUbuntuに入ってから再実行 |
| （コマンドを打っても無反応） | プロセスのフリーズ | Ctrl+C → `openclaw gateway stop` → `openclaw gateway` |
| 認証エラー | トークン期限切れ | `claude setup-token` → 再設定 |

---

## 🛡️ 鉄の掟（2026年2月障害からの教訓）

### 障害の経緯
2026年2月26〜27日、セキュリティ設定時に存在しないキー「groupAllowlist」を入力したことでOpenClawが起動不能となり、約10時間の業務停止が発生。外部AI（Gemini）の提案を検証なく適用したことが根本原因。

### 再発防止策（4原則）

1. **設定変更前に必ずバックアップ** - .bakファイルを作成してから変更する
2. **ドキュメントの絶対遵守** - 感覚で設定キーを入力しない。公式ドキュメントで確認
3. **外部AIの提案を二重チェック** - Gemini等の提案を鵜呑みにせず、必ず検証する
4. **再起動後のログ監視** - エラーが出ていないか確認してから業務開始

### 実施済み安全対策
- ✅ 日次自動バックアップ（毎朝4時、7日分保持）
- ✅ APIキーの二重保存（~/openclaw-emergency-backup/）
- ✅ HEARTBEATでのゲートウェイ生存確認

---

## 📆 毎朝のルーティン（推奨）

毎朝業務前に以下の3コマンドを実行し、WSLのネットワークをクリーンな状態でスタートさせる。

1. `wsl --shutdown` （PowerShellで実行）
2. `wsl` （PowerShellで実行）
3. `openclaw gateway` （WSL内で実行）

---

## 🖥️ 環境判定ルール

| 画面の特徴 | 環境 | openclawの実行 |
|-----------|------|---------------|
| 青い背景、`PS C:\Users\>` のプロンプト | PowerShell | 不可。まず `wsl` と入力してWSLに入る |
| 黒背景に緑文字、`user@PC:~$` のプロンプト | WSL / Ubuntu | 可。そのままコマンドを実行 |

---

## 💬 回答スタイル（私の心得）

- **構造:** 結論ファースト。コマンド → 説明 → 次の手順の順
- **分量:** 1回最大3コマンド、説明は各1行以内
- **トーン:** 有能で冷静。軽いユーモア可。過剰な謝罪・大袈裟な表現は禁止
- **誤り時:** 「案内が間違っていました」と一言 → 正しいコマンドを即座に提示
- **表示形式:** コマンドはバッククォート囲み。見えないと言われたら形式を変える

---

## 📌 システム構成

- **プラットフォーム:** OpenClaw（Anthropic Claude連携）
- **動作環境:** WSL2（Ubuntu）on Windows
- **ランタイム:** Node.js
- **接続先:** Discord（朝刊配信、社長への報告等）
- **AIモデル:** Claude Sonnet 4.5 / Opus 4.5
- **認証方式:** Claude Max/Pro setup-token

---

**この虎の巻を常に参照し、確実な復旧と安定運用を実現します。**
