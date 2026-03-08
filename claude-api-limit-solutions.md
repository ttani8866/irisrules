# Claude API上限対策ガイド

## 問題：すぐに上限に達してしまう

Claude APIには以下の制限があります：
- **レート制限**: リクエスト数/分
- **使用量制限**: トークン数/月
- **同時接続数**: 同時リクエスト数

---

## 解決策

### 1. 複数のAPIキーをローテーション

**概要:** 複数のAnthropicアカウントで複数のAPIキーを取得し、ローテーションする

**手順:**
```bash
# 複数のAPIキーを環境変数に設定
export ANTHROPIC_API_KEY_1="sk-ant-api..."
export ANTHROPIC_API_KEY_2="sk-ant-api..."
export ANTHROPIC_API_KEY_3="sk-ant-api..."
```

**OpenClawでの設定:**
```bash
openclaw agents configure main
# API keyを複数設定（ローテーション用）
```

**メリット:** 
- 制限を実質的に倍増
- 一つが制限に達しても他のキーで継続可能

**デメリット:**
- 複数アカウントの管理が必要
- コストも倍増

---

### 2. トークン使用量の削減

**a) メモリファイルの最適化**

現在のメモリファイルを圧縮：
```bash
# 長い履歴を削除
find memory/ -name "*.md" -mtime +30 -delete

# 重要な情報だけをMEMORY.mdに集約
```

**b) プロンプトの最適化**

不要なコンテキストを削除：
- 過去の会話履歴を圧縮
- 重複情報の削除
- 要約の活用

**c) thinking モードの調整**

```bash
# .openclaw/config.yaml
thinking: low  # または off
```

`thinking: low` でトークン消費を削減できます。

---

### 3. リクエスト頻度の調整

**HEARTBEAT.md の頻度を下げる:**

```markdown
## Moltbook (every 2 hours instead of 30 minutes)
If 2 hours since last Moltbook check:
1. Check feed
2. Reply to comments
```

**メリット:** API呼び出し回数が減る  
**デメリット:** リアルタイム性が低下

---

### 4. モデルの変更

**より安価なモデルを使用:**

OpenClawの設定で、一部のタスクに軽量モデルを使用：
```bash
# 簡単なタスクには haiku（安価・高速）
# 複雑なタスクには sonnet（現在使用中）
```

---

### 5. キャッシュの活用（将来的）

Claude APIの **prompt caching** 機能を活用：
- 繰り返し使うプロンプト部分をキャッシュ
- トークン使用量を削減

**注:** OpenClawが対応している場合のみ有効

---

### 6. 緊急時の対処

**上限に達した場合:**

1. **別のモデルに切り替え:**
   ```bash
   openclaw session set-model gpt-4  # OpenAI GPT-4に切り替え
   ```

2. **制限リセットを待つ:**
   - レート制限: 1分後にリセット
   - 日次制限: 翌日0:00 UTCにリセット
   - 月次制限: 翌月1日にリセット

3. **有料プランへのアップグレード:**
   Anthropicの有料プランで上限を引き上げ

---

## 推奨設定（バランス型）

```yaml
# ~/.openclaw/config.yaml

model: "anthropic/claude-sonnet-4"
thinking: "low"  # トークン削減

# ハートビート頻度を調整
# HEARTBEAT.mdで1時間に1回程度に
```

---

## モニタリング

**使用量の確認:**
```bash
openclaw status
# または
openclaw session-status
```

定期的に使用量をチェックし、上限に近づいたら対策を実施。

---

## まとめ

**即座にできる対策:**
1. ✅ thinking を "low" に設定
2. ✅ ハートビート頻度を下げる（30分 → 1-2時間）
3. ✅ メモリファイルを整理

**中長期的な対策:**
1. 複数APIキーのローテーション
2. タスクごとにモデルを使い分け
3. 有料プランへのアップグレード

現在の設定と使用状況に応じて、適切な対策を選択してください📊
