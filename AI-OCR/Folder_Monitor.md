# AI-OCRフォルダ監視システム 要件定義

## 概要

指定フォルダをwatchdog（inotify）によりリアルタイム監視し、新規ファイルを検知したらルールに基づいてOCRジョブに自動投入する機能。

## 機能要件

- 監視フォルダ（Watcher）を複数登録・管理できる
- アプリ起動時にDBからWatcher一覧を取得し、watchdog Observerに自動登録する
- Watcher作成・削除時にObserverへの登録・解除をリアルタイムで反映する
- Watcherごとにファイルマッチングルールを複数登録できる
- ルールはファイル名パターン・使用エンジン・テンプレート・優先度を持つ
- ファイル検知時、優先度順にルールをマッチングし、最初に一致したルールでOCRジョブを投入する
- 対応ファイル形式はPDF・PNG・JPG・TIFFとする
- 処理済みファイルの二重処理を防止する
- 処理結果（succeeded / failed / no_match）をログとして記録する
- 手動スキャンをAPIから実行できる

## API定義

基本パス: `/v1/folder-monitor/watchers`

---

### 1. `GET /` — Watcher一覧取得

登録されている監視フォルダの一覧を取得します。

**リクエスト**

なし

**レスポンス `200 OK`**

```json
{
  "watchers": [
    {
      "watcher_id": 1,
      "name": "請求書フォルダ",
      "folder_path": "/data/invoices",
      "rule_count": 2,
      "description": "毎日届く請求書PDFを監視",
      "created_at": "2026-04-13T10:00:00",
      "updated_at": "2026-04-13T10:00:00"
    }
  ]
}
```

---

### 2. `POST /` — Watcher作成

既存のフォルダを監視対象として登録します。

**リクエスト**

```json
{
  "name": "請求書フォルダ",
  "folder_path": "/data/invoices",
  "description": "毎日届く請求書PDFを監視"
}
```

**レスポンス `201 Created`**

```json
{
  "watcher_id": 1,
  "name": "請求書フォルダ",
  "folder_path": "/data/invoices",
  "description": "毎日届く請求書PDFを監視",
  "created_at": "2026-04-13T10:00:00",
  "updated_at": "2026-04-13T10:00:00"
}
```

---

### 3. `GET /{watcher_id}` — Watcher詳細取得

指定した監視フォルダの詳細情報をルール一覧込みで取得します。

**リクエスト**

なし

**レスポンス `200 OK`**

```json
{
  "watcher_id": 1,
  "name": "請求書フォルダ",
  "folder_path": "/data/invoices",
  "description": "毎日届く請求書PDFを監視",
  "rules": [
    {
      "rule_id": 1,
      "name": "PDF処理ルール",
      "file_pattern": "*.pdf",
      "priority": 10,
      "engine_id": 1,
      "provider_code": "google",
      "engine_code": "vision",
      "template_id": 2,
      "description": null,
      "created_at": "2026-04-13T10:00:00",
      "updated_at": "2026-04-13T10:00:00"
    }
  ],
  "created_at": "2026-04-13T10:00:00",
  "updated_at": "2026-04-13T10:00:00"
}
```

---

### 4. `PUT /{watcher_id}` — Watcher更新

指定した監視フォルダの設定を更新します。

**リクエスト**

全フィールドオプション（未指定の場合は既存値を維持）

```json
{
  "name": "請求書フォルダ（更新）",
  "folder_path": "/data/invoices_new",
  "description": "更新後の説明"
}
```

---

### 5. `DELETE /{watcher_id}` — Watcher削除

指定した監視フォルダを削除します。

**リクエスト**

なし

```
DELETE /v1/folder-monitor/watchers/1
```

**レスポンス `204 No Content`**

返却データなし。削除成功時は `204` のみ返ります。

---

### 6. `GET /{watcher_id}/rules` — ルール一覧取得

指定した監視フォルダに登録されているルールの一覧を取得します。

**リクエスト**

なし

```
GET /v1/folder-monitor/watchers/1/rules
```

**レスポンス `200 OK`**

```json
{
  "items": [
    {
      "rule_id": 1,
      "name": "PDF処理ルール",
      "file_pattern": "*.pdf",
      "priority": 10,
      "engine_id": 1,
      "provider_code": "google",
      "engine_code": "vision",
      "template_id": 2,
      "description": null,
      "created_at": "2026-04-13T10:00:00",
      "updated_at": "2026-04-13T10:00:00"
    }
  ],
  "total": 1
}
```

---

### 7. `POST /{watcher_id}/rules` — ルール作成

指定した監視フォルダにルールを追加します。

**リクエスト**

`mail_monitor` の `CreateRuleRequest` に相当します。

```json
{
  "name": "PDF処理ルール",
  "file_pattern": "*.pdf",
  "priority": 10,
  "engine_id": 1,
  "template_id": 2,
  "description": null
}
```

---

### 8. `GET /{watcher_id}/rules/{rule_id}` — ルール詳細取得

指定したルールの詳細情報を取得します。

**リクエスト**

なし

```
GET /v1/folder-monitor/watchers/1/rules/1
```

**レスポンス `200 OK`**

```json
{
  "rule_id": 1,
  "name": "PDF処理ルール",
  "file_pattern": "*.pdf",
  "priority": 10,
  "engine_id": 1,
  "provider_code": "google",
  "engine_code": "vision",
  "template_id": 2,
  "description": null,
  "created_at": "2026-04-13T10:00:00",
  "updated_at": "2026-04-13T10:00:00"
}
```

---

### 9. `PUT /{watcher_id}/rules/{rule_id}` — ルール更新

指定したルールを更新します。

**リクエスト**

全フィールドオプション（未指定の場合は既存値を維持）

```json
{
  "name": "PDF処理ルール（更新）",
  "file_pattern": "*.pdf",
  "priority": 20,
  "engine_id": 1,
  "template_id": 3,
  "description": "更新後の説明"
}
```

**レスポンス `200 OK`**

```json
{
  "rule_id": 1,
  "name": "PDF処理ルール（更新）",
  "file_pattern": "*.pdf",
  "priority": 20,
  "engine_id": 1,
  "provider_code": "google",
  "engine_code": "vision",
  "template_id": 3,
  "description": "更新後の説明",
  "created_at": "2026-04-13T10:00:00",
  "updated_at": "2026-04-13T10:30:00"
}
```

---

### 10. `DELETE /{watcher_id}/rules/{rule_id}` — ルール削除

指定したルールを削除します。

**リクエスト**

なし

```
DELETE /v1/folder-monitor/watchers/1/rules/1
```

**レスポンス `204 No Content`**

返却データなし。削除成功時は `204` のみ返ります。

---

### 11. `GET /{watcher_id}/logs` — 処理ログ一覧取得

指定した監視フォルダの処理ログ一覧を取得します。

**リクエスト**

なし

```
GET /v1/folder-monitor/watchers/1/logs
```

**レスポンス `200 OK`**

`mail_monitor` の `MessageListResponse` に相当します。

```json
{
  "items": [
    {
      "log_id": 1,
      "watcher_id": 1,
      "file_path": "/data/invoices/invoice_001.pdf",
      "matched_rule_id": 1,
      "jobs": [
        {
          "job_id": 42,
          "filename": "invoice_001.pdf"
        }
      ],
      "status": "succeeded",
      "error": null,
      "detected_at": "2026-04-13T10:05:00"
    }
  ],
  "total": 1
}
```

---

### 12. `POST /{watcher_id}/sync` — 手動スキャン実行

指定した監視フォルダを手動でスキャンし、未処理ファイルをOCRジョブに投入します。

**リクエスト**

なし

```
POST /v1/folder-monitor/watchers/1/sync
```

**レスポンス `200 OK`**

```json
{
  "detected_count": 5,
  "succeeded_count": 4,
  "failed_count": 1
}
```

---

## ドメイン構成

```
api/folder_monitor/
├── domain/
│   ├── aggregates/
│   │   └── folder_watcher.py            # FolderWatcher集約・FolderWatcherRuleエンティティ
│   ├── value_objects/
│   │   ├── folder_watcher_id.py         # WatcherIdの型定義
│   │   └── folder_rule_id.py            # RuleIdの型定義
│   └── repositories/
│       ├── watcher_repository.py        # WatcherRepository（IF）
│       └── file_log_repository.py       # FileLogRepository（IF）
├── application/
│   ├── _engine_resolver.py              # エンジン情報解決（mail_monitorと同様）
│   ├── create_watcher_use_case.py
│   ├── list_watchers_use_case.py
│   ├── get_watcher_use_case.py
│   ├── update_watcher_use_case.py
│   ├── archive_watcher_use_case.py
│   ├── add_rule_use_case.py
│   ├── list_rules_use_case.py
│   ├── get_rule_use_case.py
│   ├── update_rule_use_case.py
│   ├── remove_rule_use_case.py
│   ├── record_file_log_use_case.py
│   ├── list_file_logs_use_case.py
│   └── scan_watcher_use_case.py         # 手動スキャン
├── infrastructure/
│   ├── persistence/
│   │   ├── models/
│   │   │   └── folder_monitor_models.py
│   │   ├── watcher_repository_impl.py
│   │   └── file_log_repository_impl.py
│   └── watching/
│       └── folder_event_worker.py       # watchdog Observer管理・イベントハンドラー
└── presentation/
    ├── dependencies.py
    ├── requests.py
    ├── responses.py
    └── router.py
```

## エラーハンドリング

| ステータスコード | 説明 |
| --- | --- |
| `404 Not Found` | Watcher・Rule が存在しない |
| `409 Conflict` | 同名のWatcherが既に存在する |
| `422 Unprocessable Entity` | バリデーションエラー（必須項目未入力・文字数超過など） |

## 処理フロー

```
[アプリ起動時]
└─ DBからWatcher一覧を取得 → watchdog Observer登録・起動

[ファイル検知時]
└─ 対応形式チェック
└─ 処理済みチェック
└─ ルールマッチング
     ├─ マッチあり → OCRジョブ投入 → 結果記録
     └─ マッチなし → no_match記録

[Watcher作成・削除時]
└─ Observerへの登録・解除をリアルタイムで反映

[アプリ終了時]
└─ Observer停止
```
