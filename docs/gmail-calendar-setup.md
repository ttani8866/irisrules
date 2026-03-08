# Gmail/Googleカレンダー復旧ガイド

## 概要

Maton.ai API Gateway skillを使用したGmail/Googleカレンダー連携の実装方法

## 前提条件

- Maton.aiアカウント作成済み
- Google OAuth認証完了済み（Maton.aiダッシュボードで設定）
- MATON_API_KEYを取得済み

## 実装方法

### 1. 環境変数の設定

```bash
export MATON_API_KEY="YOUR_API_KEY_HERE"
```

または `~/.bashrc` に追加：

```bash
echo 'export MATON_API_KEY="YOUR_API_KEY_HERE"' >> ~/.bashrc
source ~/.bashrc
```

### 2. Gmail送信

```python
import urllib.request
import json
import base64
from email.mime.text import MIMEText

# メール作成
message = MIMEText('本文', 'plain', 'utf-8')
message['to'] = 'recipient@example.com'
message['from'] = 'sender@example.com'
message['subject'] = '件名'

# RFC 2822形式でbase64エンコード
raw_message = base64.urlsafe_b64encode(message.as_bytes()).decode('utf-8')

# Gmail API経由で送信
data = json.dumps({'raw': raw_message}).encode()
req = urllib.request.Request(
    'https://gateway.maton.ai/google-mail/gmail/v1/users/me/messages/send',
    data=data,
    method='POST'
)
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
req.add_header('Content-Type', 'application/json')

response = urllib.request.urlopen(req)
result = json.load(response)
print(f"送信完了: {result['id']}")
```

### 3. Googleカレンダー - イベント取得

```python
import urllib.request
import json
from datetime import datetime
from urllib.parse import quote

# 期間を指定
time_min = quote('2026-03-08T00:00:00+09:00')
time_max = quote('2026-03-15T23:59:59+09:00')

url = f'https://gateway.maton.ai/google-calendar/calendar/v3/calendars/primary/events?timeMin={time_min}&timeMax={time_max}&singleEvents=true&orderBy=startTime'
req = urllib.request.Request(url)
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')

response = urllib.request.urlopen(req)
result = json.load(response)

for event in result.get('items', []):
    print(f"{event.get('start')} - {event.get('summary')}")
```

### 4. Googleカレンダー - イベント作成

```python
import urllib.request
import json
from datetime import datetime
import pytz

jst = pytz.timezone('Asia/Tokyo')
start_dt = datetime(2026, 3, 10, 9, 30, tzinfo=jst)
end_dt = datetime(2026, 3, 10, 10, 30, tzinfo=jst)

event = {
    'summary': '会議名',
    'start': {'dateTime': start_dt.isoformat(), 'timeZone': 'Asia/Tokyo'},
    'end': {'dateTime': end_dt.isoformat(), 'timeZone': 'Asia/Tokyo'},
    'location': '場所（オプション）'
}

data = json.dumps(event).encode()
req = urllib.request.Request(
    'https://gateway.maton.ai/google-calendar/calendar/v3/calendars/primary/events',
    data=data,
    method='POST'
)
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
req.add_header('Content-Type', 'application/json')

response = urllib.request.urlopen(req)
result = json.load(response)
print(f"作成完了: {result['id']}")
```

### 5. Googleカレンダー - イベント更新

```python
import urllib.request
import json

# イベントIDを指定
event_id = 'EVENT_ID_HERE'

# まず既存イベントを取得
url = f'https://gateway.maton.ai/google-calendar/calendar/v3/calendars/primary/events/{event_id}'
req = urllib.request.Request(url)
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')

response = urllib.request.urlopen(req)
event = json.load(response)

# タイトルを更新
event['summary'] = '新しいタイトル'

# PUTで更新
data = json.dumps(event).encode()
req = urllib.request.Request(url, data=data, method='PUT')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
req.add_header('Content-Type', 'application/json')

response = urllib.request.urlopen(req)
result = json.load(response)
print(f"更新完了: {result['id']}")
```

## トラブルシューティング

### 403 Forbidden / Missing Authentication Token

- `Authorization` ヘッダーの形式を確認: `Bearer $MATON_API_KEY`
- 環境変数が正しく設定されているか確認: `echo $MATON_API_KEY`

### 400 Bad Request (timeMin/timeMax)

- 日時のURLエンコードを忘れずに: `urllib.parse.quote()`
- ISO 8601形式を使用: `2026-03-08T00:00:00+09:00`

### Connection IDが見つからない

```python
# 接続一覧を確認
req = urllib.request.Request('https://ctrl.maton.ai/connections?app=google-mail&status=ACTIVE')
req.add_header('Authorization', f'Bearer {os.environ["MATON_API_KEY"]}')
response = urllib.request.urlopen(req)
print(json.dumps(json.load(response), indent=2))
```

## 参考資料

- Maton.ai API Gateway Skill: https://clawhub.com/byungkyu/api-gateway
- Gmail API: https://developers.google.com/gmail/api
- Google Calendar API: https://developers.google.com/calendar/api

## 履歴

- 2026-03-08: 初版作成（sleeping beauty障害からの復旧）
