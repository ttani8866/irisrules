# X API設定ガイド（既存アカウント @irisAI_sec_kpr）

## 前提

- 既存アカウント: https://x.com/irisAI_sec_kpr/
- 目的: アイリスが自動投稿できるようにする

---

## 手順

### Step 1: X Developer Portal にアクセス

1. **ブラウザで開く:**  
   https://developer.x.com/en/portal/dashboard

2. **ログイン:**  
   @irisAI_sec_kpr のアカウントでログイン
   
   **⚠️ 重要:** リョウ様のメインアカウント（@iron_aidriven）ではなく、
   **@irisAI_sec_kpr** でログインしてください。

---

### Step 2: アプリを作成（まだない場合）

1. **"+ Create Project" または "+ Add App"** をクリック

2. **アプリ名を入力:**
   - 例: `iris-ai-bot`
   - 説明: `AI secretary posting automation`

3. **Use case を選択:**
   - "Making a bot" を選択

4. **App environment:**
   - Development（開発用）を選択

---

### Step 3: APIキーを取得

アプリ作成後、以下の4つの情報が表示されます：

```
API Key (Consumer Key)
API Key Secret (Consumer Secret)  
Access Token
Access Token Secret
```

**⚠️ 超重要:** これらは一度しか表示されません！  
すぐにコピーして保存してください。

---

### Step 4: 権限を設定

1. **App settings** → **User authentication settings** → **Set up**

2. **App permissions** で以下を選択:
   - ✅ **Read and write** （投稿するため必須）
   - ❌ Read only （これだと投稿できない）
   - ❌ Read and write and Direct message （DMは不要）

3. **Type of App:**
   - "Automated App or bot" を選択

4. **Callback URL（必要な場合）:**
   - `http://localhost:3000/callback` でOK（使わないので）

5. **Website URL:**
   - https://www.moltbook.com/@iris-ai （任意）

6. **Save** をクリック

---

### Step 5: キーを再生成（既にアプリがある場合）

もし既にアプリがあってキーを忘れた場合：

1. **Keys and tokens** タブを開く
2. **"Regenerate" ボタン** をクリック（API Key, Access Token）
3. 新しいキーが表示される → すぐにコピー

---

### Step 6: アイリスに設定情報を渡す

取得した4つの情報を、このDiscordチャットに貼り付けてください：

```
API Key: xxxxx
API Secret Key: xxxxx
Access Token: xxxxx
Access Token Secret: xxxxx
```

**⚠️ セキュリティ:** Discordに貼る際、メッセージ送信後すぐに削除することを推奨。
私が保存したら、メッセージを削除してください。

---

### Step 7: アイリスが自動設定

私が受け取ったら、以下を自動実行します：

1. `~/.config/x/credentials.json` に保存
2. 環境変数に設定
3. バックアップ作成
4. テスト投稿（オプション）

---

## トラブルシューティング

### Q: "App-only" と "User context" の違いは？

**A:** 
- **App-only**: アプリ自身として動作（リミット高い）
- **User context**: ユーザー（@irisAI_sec_kpr）として投稿 ← **これが必要**

User contextには **Access Token** が必要です。

---

### Q: "OAuth 1.0a" と "OAuth 2.0" どっち？

**A:** どちらでも可能ですが、**OAuth 1.0a が簡単**です。
Access Token / Access Token Secret を使う方式です。

---

### Q: 無料プランでも大丈夫？

**A:** はい。無料プラン（Free tier）でも以下が可能：
- 月間 1,500 ツイート
- 読み取り: 月間 10,000 リクエスト

アイリスの投稿頻度（1日3-5投稿）なら余裕です。

---

## 投稿テスト

設定完了後、以下でテスト投稿します：

```bash
# アイリスが実行
curl -X POST "https://api.twitter.com/2/tweets" \
  -H "Authorization: OAuth ..." \
  -H "Content-Type: application/json" \
  -d '{"text": "テスト投稿 from iris-ai 🤖"}'
```

成功したら、自動投稿の準備完了です🎉

---

## よくある質問

**Q: リョウ様のアカウント（@iron_aidriven）でも同じ手順？**

A: はい、同じ手順です。ただし：
- @iron_aidriven でログイン
- 別のアプリを作成
- 別のAPIキーを取得

両方のアカウントで自動投稿可能にできます。

---

## まとめ

1. ✅ https://developer.x.com/en/portal/dashboard でログイン（@irisAI_sec_kpr）
2. ✅ アプリ作成 or 既存アプリのキー再生成
3. ✅ 権限を "Read and write" に設定
4. ✅ 4つのキーを取得・コピー
5. ✅ アイリスに渡す（Discordで）
6. ✅ テスト投稿

**所要時間:** 5-10分

準備ができたら、4つのキーをお知らせください！
