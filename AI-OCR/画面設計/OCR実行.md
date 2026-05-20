# OCR実行画面 パターン比較

以下に、AI-OCRで実際にあり得る
OCR実行画面の構成案を複数提示します。

---

# 凡例

各セクションで使う見出しの意味:

- **必須情報**
  - OCR実行画面に存在する/載せるべき項目
- **関連実装(他画面)**
  - OCR実行画面外で既に実装されている関連機能(参考用)
- **将来要件**
  - OCR実行画面に追加する予定/検討中の機能
- **注意事項**
  - 実装上の制約・設計上の前提・既知の挙動など、画面利用時の留意点

---

# OCR実行画面に必要な画面情報

---

## 1. ファイル投入エリア(最重要)

### 必須情報

- ドラッグ＆ドロップ
- ファイル選択ボタン
- 対応形式表示(PDF / PNG / TIFF 等)
- ファイル名
- ページ数
- ファイルサイズ
- サムネイル / プレビュー      | ❌ 未実装(アイコンのみ)

---

## 2. OCRエンジン選択

### 必須情報

- OCR Provider(実装で seed されているもの: `scripts/init/engines.json`)
  - mock(テスト用)
  - azure
  - google
  - paddleocr
  - yomitoku
  - gemini
  - qwen
- OCR Engine
  - 上記Provider単位での選択
  - ※ 精度/速度/帳票特化等のモデル種別はエンジン名に内包

### 注意事項

- AWS / Tesseract は本システムでは未対応
- モデル種別(高精度/高速/Invoice特化)を別UIで選ばせる構成は採用しない
  - Provider/Engineの名称で識別する設計

---

## 3. テンプレート選択

### 必須情報

- テンプレート名
  - テンプレート管理画面で登録したものから選択
  - 抽出領域数を表示(例: `請求書テンプレート (5 領域)`)

### 将来要件

- 自動判定 ON/OFF
  - 帳票内容から適合テンプレートをシステム側で自動推定する機能
  - 実装には API 側に分類器エンドポイントの新設が必要
- 帳票種別
  - 「請求書」「申請書」「名刺」等のカテゴリ属性
  - Template ドメイン拡張(`category` フィールド追加)と DB スキーマ拡張が必要

---

## 4. OCR言語

### 必須情報

- 日本語
- 英語
- 中国語
- 多言語OCR

---

## 5. 前処理設定

### 必須情報

- 傾き補正(`deskew`)
- 画像回転補正(`auto_rotate`)
- コントラスト調整(`contrast`)
- 二値化(`binarize`)
- 文字中央寄せ・テーブル(`center_text`)
  - テーブル構造を持つテンプレート選択時のみ有効化
- 解像度アップスケール(`upscale`)
- 画像周囲マージン追加(`border_padding`)

### 将来要件

- ノイズ除去
  - 実装には API 側の前処理パイプラインに denoise フィルタの追加が必要
  - クライアントの `PreprocessingOptionDefinitions.All` にも追加が必要

### 注意事項

- 前処理オプション一覧はクライアントハードコード(`PreprocessingOptionDefinitions.cs`)
  - 今後の追加時は API 側パイプラインと両方の更新が必要
  - 将来的には API レスポンスから動的取得する設計に寄せるのが望ましい

---

## 6. OCR実行状態

### 必須情報

- 待機中(`queued`)
  - ジョブが API 側のキューに登録された状態
- 処理中(`running`)
  - OCR エンジンによる処理中
- 完了(`succeeded`)
  - 処理成功・結果画面への遷移可
- 失敗(`failed`)
  - エラーメッセージを画面下部に表示
- キャンセル済(`canceled`)
  - キャンセル操作後の終了状態

### 将来要件

- Upload中
  - 現状はファイル送信中(API への POST 中)の専用表示なし
  - `IsExecuting=true` 期間に「OCR処理中...」と一括表示される
  - 大容量ファイル時のフィードバック強化に向け、
    アップロード進捗と OCR 処理の分離表示が望ましい

### 注意事項

- 実行オーバーレイのタイトルは静的に「OCR処理中...」と表示し、
  状態区別はその下の「ステータス: {表示テキスト}」で行う
  (`ExecutionView.xaml:125-134`)
- 成功時のみ「結果を表示」ボタンへの遷移オーバーレイに切り替わる
  (`ExecutionView.xaml:147-172`)

---

## 7. 推定コスト

### 必須情報

なし。本画面ではコスト表示を持たない方針。

### 関連実装(他画面)

- 設定画面「使用量」セクション(`SettingsView.xaml:332-416`)
  - 合計ページ数
  - 推定コスト(¥表記)
  - Provider 別の TotalPages / TotalCost
  - 「更新」ボタンで再取得

### 将来要件

- OCR 実行画面での事前コスト推定
  - ファイル投入時にページ数 × エンジン単価で実行前に金額を提示
  - 単価マスタ(`t_engine_pricing_set`)は実装済のため、
    推定エンドポイントを追加すれば対応可能
  - エンジン切替時に再計算する UI 連動が必要

### 注意事項

- 単価は `t_engine_pricing_set` で履歴管理
  - `get_current_pricing` で実行時点の単価を取得し、`t_usage_recorded` に
    スナップショットとして記録
- 通貨はデフォルト JPY、エンジン単位で変更可
- 画面注記:
  「※ 推定コストは各社公式料金を基に算出した目安です。
    実際の請求額とは異なる場合があります。」

---

## 8. 出力設定

### 必須情報

なし。本画面では出力設定を持たない方針。
結果はサーバー側ストレージに自動保存され、結果確認画面で表示・参照する。

### 関連実装(他画面)

- 結果テキストのクリップボードコピー
  - 結果確認画面のコンテキストメニュー / `Ctrl+C`
  - (`ResultView.xaml:580-588` / `ResultViewModel.cs:790-797`)

### 将来要件

エクスポート機能を実装する場合の配置先(本画面 or 結果確認画面)は要検討。

- JSON 出力
- CSV 出力
- Excel 出力
- DB 保存
  - 現状は API 側の内部ストレージのみ
- 保存先指定
  - ファイルシステムへの書き出しは未実装
  - 連携先(S3 / SharePoint / 共有フォルダ等)への自動エクスポートも未実装

### 注意事項

- 結果データはサーバー側ストレージに保存
  - `api/jobs/infrastructure/storage/file_storage.py`
- フォルダ監視・メール監視は入力側のみで、結果のエクスポート設定は持たない
- 後続システム連携が必要な場合は将来要件として整理

---

## 9. OCR結果遷移

### 必須情報

- 結果画面への遷移
  - 完了時に Success Overlay へ自動切替(`ExecutionView.xaml:147-172`)
  - 「結果を表示」ボタンで結果画面に手動遷移
    (`NavigateToResultCommand` / `ExecutionViewModel.cs:207-222`)
  - 遷移と同時に `LoadResultWithFileAsync(reportId, filePath)` を fire-and-forget で実行
    (`ResultViewModel.cs:397-411`)
  - 遷移後に `IsJobSucceeded` をリセット
- メニュー / `Ctrl+R` からも結果画面へ遷移可
  - `MainWindow.xaml:22` / `MenuViewModel.NavigateToResults`
  - ※ メニュー経由は `filePath` 未付与のため、結果画面の履歴選択から再ロードする動線

### 関連実装(他画面)

- 結果画面のレビュー UI
  - 各テキストブロックに `ReviewStatus`(未確認 / 確認済 / 編集して確認済)
  - 一括「未確認をすべて確認済にする」コマンド
    (`BulkCompleteUnreviewedCommand` / `ResultViewModel.cs:633-665`)
  - レビュー状態は `IReviewStateGatewayPort` 経由で永続化

### 将来要件

- 完了後の結果画面への自動遷移
  - 現状は「結果を表示」ボタンの手動押下
  - 設定で「自動遷移 ON/OFF」を切替可能にする想定
- 自動レビュー開始
  - 結果画面遷移と同時にレビューモードに入る
    (最初の未確認ブロックを自動フォーカス、Enter で確認済→次へ進む等)
  - 結果画面側に挙動を追加する必要あり

### 注意事項

- 失敗・キャンセル時は Success Overlay は表示されない
  - エラーメッセージのみ画面下部に表示
- Success Overlay 表示中もメニュー操作で他画面への遷移は可能
- 「校正」(`ProofreadingId`)は OCR 実行時にサーバー側で適用される自動補正であり、
  結果画面の「レビュー」(`ReviewStatus`)とは別の概念

---

## 10. エラー表示

### 必須情報

- 単一エラーメッセージ表示
  - `ExecutionViewModel.ErrorMessage` を `ExecutionView.xaml:300-308` の
    `Border` + `TextBlock` で表示
  - カテゴリ別のアイコン・色・ラベル等の区別なし
- 表示元の種別
  - **OCR 失敗**: `JobStatusValue.Failed` 時に `finalResult.Error?.Message` を表示
    (`JobMonitorViewModel.cs:170-171`)
  - **API エラー**: `ApiException.Message` を raw 表示
    (`JobMonitorViewModel.cs:183-187`)
  - **その他例外**: `Exception.Message` を raw 表示
    (`JobMonitorViewModel.cs:188-192`)
  - **ファイル検証エラー**: `FileRepository.ValidateFile` の結果を `SetFilePath` 経由で表示
    - 未サポート拡張子 / サイズ超過(>10MB) / ページ数超過(>100) を検出

### 関連実装(他画面/共通基盤)

- `ApiCallHelper`(`ApiCallHelper.cs`)
  - `HTTPValidationError` → `ApiException(422, ..., "validation_error")`
  - `ProblemDetailResponse` → `ApiException(status, detail, type)`
- `ErrorHandler`(`ErrorHandler.cs`) ※ **本画面では未使用**
  - `ApiException.StatusCode` をカテゴリにマッピング
    (Authentication / NotFound / Conflict / Expired / Validation / Server)
  - `NetworkException.IsTimeout` → `ErrorCategory.Timeout`
  - `CanRetry` / `ShouldNavigateToSettings` フラグも生成済
- ドメイン例外
  - `ApiException`(StatusCode, ErrorCode, RequestId, Message)
  - `NetworkException`(`IsTimeout` フラグ付き)

### 将来要件

- 本画面で `ErrorHandler` を経由してエラーを構造化表示する
  - カテゴリ別のアイコン・色分け(認証 / 入力 / サーバー / タイムアウト等)
  - `CanRetry=true` の場合はリトライボタン提示
  - `ShouldNavigateToSettings=true` の場合は設定画面誘導
- 明示的なリクエストタイムアウト処理
  - HttpClient のタイムアウト制御と `NetworkException(IsTimeout=true)` への変換
- `FileRepository.ValidateFile` のメッセージ日本語化
  - 現状英語(例: `Unsupported file format: .docx`)
- バリデーション失敗詳細表示
  - `HTTPValidationError` の `Detail` を `ApiCallHelper.FormatValidationDetail` で
    フィールド名付き整形済 (`field: msg / field2: msg2`) — UIで複数行表示できる構造化を検討

### 注意事項

- エラーメッセージ表示中も「OCR実行」ボタンは押下可能で、再実行で `ErrorMessage=null` にリセット
  (`ExecutionViewModel.cs:163`)
- `ErrorHandler` 系は他画面(設定 / 結果 等)で利用される共通基盤として整備されており、
  本画面への適用は将来要件として未着手

---

# OCR実行画面の代表的な画面分割

現在の実装状況・将来要件・AI-OCR 製品の実例を踏まえ、
OCR 実行画面のレイアウト案を以下の 2 軸 × 3 パターンで整理する。

- **将来設計なし版**(現実装寄り) … パターン①〜③
- **将来設計あり版**(AI-OCR 完成系寄り) … パターン④〜⑥

---

## 前提整理

現在の設計方針は AI-OCR としてバランスが取れている。
特に以下の点が UiPath / DX Suite / invoiceAgent 寄りの良い設計と一致する。

| 方針 | 評価 |
| ---- | -- |
| 左:ファイル主役 | ◎ |
| 右:OCR 設定 | ◎ |
| 詳細設定折り畳み | ◎ |
| 出力設定なし | ◎ |
| 結果確認画面へ分離 | ◎ |

---

# 将来設計なし版(現実装ベース)

---

## パターン① シンプル 2 ペイン型(最推奨)

現在の方向性。

### 画面図

```text
┌──────────────────────────────┬────────────────┐
│ OCR対象ファイル              │ OCR設定        │
│                              │                │
│ Drag & Drop                  │ Provider       │
│                              │ Engine         │
│ invoice.pdf                  │ Template       │
│ PDF / 12 pages / 3MB         │ Language       │
│                              │                │
│ [ファイル選択]               │ 詳細設定 ▼     │
│                              │ ・deskew       │
│                              │ ・binarize     │
│                              │                │
│                    [OCR実行] │                │
└──────────────────────────────┴────────────────┘
```

### 画面分割

| 領域 | 内容 |
| ---- | ---- |
| 左 70% | ファイル |
| 右 30% | OCR 設定 |

### メリット

| 項目 | 評価 |
| ---- | -- |
| シンプル | ◎ |
| 初心者向け | ◎ |
| 実装しやすい | ◎ |
| 学習コスト低い | ◎ |

### デメリット

| 項目 | 評価 |
| ---- | -- |
| Queue 弱い | △ |
| 大量処理弱い | △ |
| 運用監視弱い | △ |

### 向いている

```text
・現行実装
・業務OCR
・中小規模
```

---

## パターン② 縦設定強化型

現在の UI をさらに整理し、設定を意味単位で縦に分離する案。

### 画面図

```text
┌──────────────────────────────┬────────────────┐
│ OCR対象ファイル              │ 基本設定       │
│                              │ Provider       │
│ Drag & Drop                  │ Engine         │
│                              │ Template       │
│ invoice.pdf                  │ Language       │
│                              │                │
│                              ├────────────────┤
│                              │ 詳細設定 ▼     │
│                              │ deskew         │
│                              │ contrast       │
│                              │ upscale        │
│                              │                │
│                              ├────────────────┤
│                              │ [OCR実行]      │
└──────────────────────────────┴────────────────┘
```

### 分割思想

設定の意味単位(基本/詳細/実行)で右ペインを縦に分割。

### メリット

| 項目 | 評価 |
| ---- | -- |
| 視認性高い | ◎ |
| AI-OCR 感強い | ◎ |
| 設定整理しやすい | ◎ |

### デメリット

| 項目 | 評価 |
| ---- | -- |
| 縦長化 | △ |
| 小画面で見切れやすい | △ |

### 向いている

```text
・AI-OCR製品
・設定多いOCR
```

---

## パターン③ タブ型詳細設定

詳細設定をタブで隠してコンパクトに見せる案。

### 画面図

```text
┌──────────────────────────────┐
│ OCR対象ファイル              │
│                              │
│ Drag & Drop                  │
│                              │
│ invoice.pdf                  │
│                              │
├──────────────────────────────┤
│ [基本設定] [前処理] [言語]   │
├──────────────────────────────┤
│ Provider                     │
│ Engine                       │
│ Template                     │
│                              │
│                   [OCR実行]  │
└──────────────────────────────┘
```

### 分割思想

設定群をタブで分離し、ファイル投入エリアを最大化。

### メリット

| 項目 | 評価 |
| ---- | -- |
| コンパクト | ◎ |
| 小画面向き | ◎ |
| 整理しやすい | ◎ |

### デメリット

| 項目 | 評価 |
| ---- | -- |
| 設定を見失いやすい | △ |
| OCR 感弱い | △ |

### 向いている

```text
・初心者向け
・Web SaaS
```

---

# 将来設計あり版(完成系 AI-OCR)

---

## パターン④ Dashboard 型(ABBYY 系)

Queue 監視と運用情報を常時表示する運用向け完成系。

### 画面図

```text
┌────────────┬──────────────────────┬──────────────┐
│ Queue      │ OCR対象              │ OCR設定      │
│            │                      │              │
│ 実行中 12  │ Drag & Drop          │ Engine       │
│ 完了 120   │                      │ Template     │
│ Error 2    │ invoice.pdf          │ AI Suggest   │
│            │                      │ Cost         │
│            │                      │              │
│            │                      │ [OCR実行]    │
└────────────┴──────────────────────┴──────────────┘
```

### 含まれる将来要件

| 機能 | 内容 |
| ---- | ---- |
| Queue | 実行中/完了/エラー件数の常時監視 |
| AI Suggest | Template 自動推定の提示 |
| Cost | 事前推定料金の表示 |
| Upload | アップロード状態の独立表示 |

### メリット

| 項目 | 評価 |
| ---- | -- |
| 運用最強 | ◎ |
| 大量処理 | ◎ |
| OCR センター向き | ◎ |

### デメリット

| 項目 | 評価 |
| ---- | -- |
| UI 重い | △ |
| 初心者難しい | △ |

### 向いている

```text
・大企業
・OCRセンター
・24h運用
```

---

## パターン⑤ AI アシスト型(Google / Azure 系)

AI による自動判定を全面に出した最新 AI-OCR 寄り。

### 画面図

```text
┌────────────────────────────────────┐
│ Drag & Drop                        │
│                                    │
│ invoice.pdf                        │
│                                    │
├────────────────────────────────────┤
│ AI Suggestion                      │
│ ・請求書を検出                     │
│ ・Template: 請求書                 │
│ ・Language: 日本語                 │
│ ・Cost: ¥12                        │
│                                    │
│ [詳細設定 ▼]                       │
│                                    │
│ [OCR実行]                          │
└────────────────────────────────────┘
```

### 含まれる将来要件

| 機能 | 内容 |
| ---- | ---- |
| Template 自動判定 | 帳票内容から AI 推定 |
| Cost 推定 | 投入時の事前金額提示 |
| OCR 自動選択 | エンジン/言語の自動選択 |

### メリット

| 項目 | 評価 |
| ---- | -- |
| AI 感強い | ◎ |
| 高速運用 | ◎ |
| 初心者向け | ◎ |

### デメリット

| 項目 | 評価 |
| ---- | -- |
| AI 依存 | △ |
| 微調整しにくい | △ |

### 向いている

```text
・最新AI-OCR
・SaaS
・PoC
```

---

## パターン⑥ OCR Studio 型(UiPath 完成系)

プレビュー・Queue・AI 補助・コストを 1 画面に集約したプロ向け完成系。

### 画面図

```text
┌────────────┬──────────────────────┬──────────────┐
│ Job一覧    │ OCR Preview          │ 設定詳細     │
│            │                      │              │
│ invoice01  │ bbox preview         │ Engine       │
│ invoice02  │ OCR preview          │ Template     │
│ invoice03  │                      │ Preprocess   │
│            │                      │ Cost         │
│            │                      │ AI Suggest   │
│            │                      │              │
│            │                      │ [OCR実行]    │
└────────────┴──────────────────────┴──────────────┘
```

### 含まれる将来要件

| 機能 | 内容 |
| ---- | ---- |
| bbox preview | OCR 結果のバウンディングボックス確認 |
| Queue | Job 一覧と状態監視 |
| AI Suggest | テンプレート/言語の AI 補助 |
| Cost | 課金確認 |

### メリット

| 項目 | 評価 |
| ---- | -- |
| OCR 業務最強 | ◎ |
| 修正効率 | ◎ |
| 大量処理 | ◎ |

### デメリット

| 項目 | 評価 |
| ---- | -- |
| UI 複雑 | △ |
| 実装コスト高い | △ |

### 向いている

```text
・AI-OCR製品
・エンタープライズ
・プロ運用
```

---