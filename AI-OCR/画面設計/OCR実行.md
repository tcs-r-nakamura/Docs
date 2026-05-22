# OCR実行画面 パターン比較

以下に、AI-OCRで実際にあり得る
OCR実行画面の構成案を複数提示します。

> 共通用語 (JobStatus / ReviewStatus / Provider / 対応ファイル形式 等) の
> 内部値と画面表記の対応は [用語集.md](用語集.md) を参照。

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
- 対応形式表示
  - **API 正本** (`api/jobs/domain/value_objects/mime_type.py:33-43`)
    `.pdf .png .jpg .jpeg .gif .tiff .tif .bmp .webp` (PDF/PNG/JPG/GIF/TIFF/BMP/WebP の 7 形式)
  - **WPF クライアント現状** (`FileRepository.cs:15`, `FileDialogAdapter.cs:23`)
    `.pdf .png .jpg .jpeg .tiff .tif .bmp` (PDF/PNG/JPG/TIFF/BMP の 5 形式)
  - ※ GIF / WebP は API は受け付けるが WPF からは選択不可 (クライアントで非対応)
- ファイル名
- ページ数
- ファイルサイズ
- サムネイル / プレビュー      | ❌ 未実装(アイコンのみ)

---

## 2. OCRエンジン選択

### 必須情報

- OCR Provider(実装で seed されているもの: `scripts/init/engines.json`)
  - mock(テスト用) / azure / google / paddleocr / yomitoku
  - gemini / qwen ※ WPF から実行不可 (`ProviderCredentialMap.cs` 未対応)
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
OCR 実行画面のレイアウト案を以下の 2 軸で整理する。

- **将来設計なし版**(現実装寄り) … パターン①〜③
- **将来設計あり版**(AI-OCR 完成系寄り) … パターン④〜⑤

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

現実装の構造そのまま。左ペインがファイル投入エリアで、状態 (未選択/選択/実行中/成功) で表示が切り替わる。
右ペインが「エンジン設定」(Width=320px 固定)。下部独立行に [キャンセル][OCR実行] と
エラーバナー。

### 画面図

```text
┌──────────────────────────────┬────────────────┐
│ OCR対象ファイル               │ エンジン設定   │
│                              │                │
│ ※ 未選択時:                  │ プロバイダー   │
│  📄 (Drag & Drop 受付)       │ [Azure v]      │
│  ファイルをここにドラッグ    │                │
│   ＆ドロップ                  │ エンジン       │
│  または                      │ [prebuilt v]   │
│  [ファイルを選択]            │                │
│  対応形式: PDF/PNG/JPEG/     │ テンプレート   │
│           TIFF/BMP           │ [請求書(5領域)v]│
│                              │                │
│ ※ ファイル選択時:            │ 校正設定       │
│  📄 invoice.pdf              │ [なし v]       │
│  サイズ: 1,234,567 バイト    │ ※ Proofreading │
│  ページ数: 12                │   有効時のみ   │
│  [ファイルを変更]            │                │
│                              │ 言語           │
│ ※ 実行中(IsExecuting):       │ ☑ 日本語       │
│  ━━━━━━━ (ProgressBar)       │ ☑ 英語         │
│  OCR処理中...                │ ☐ 中国語       │
│  ステータス: 処理中           │ ☐ 多言語       │
│  経過時間: 00:15.3           │                │
│  [キャンセル]                │ 前処理         │
│                              │ ☑ auto_rotate  │
│ ※ 成功時(IsJobSucceeded):    │ ☑ deskew       │
│  ✓ OCR処理が完了しました     │ ☑ binarize     │
│  処理時間: 00:23.5           │ ☐ contrast     │
│  [結果を表示]                │ ☐ center_text  │
│                              │   (テーブル時) │
│                              │ ☐ upscale      │
│                              │ ☐ border_padding│
│                              │                │
│                              │ ⚠ 既存の結果を │
│                              │   使用(重複検出)│
└──────────────────────────────┴────────────────┘
[エラーメッセージ]         [キャンセル] [OCR実行]  ← 下部独立行

凡例:
  左ペイン (Width=*、状態で表示切替):
    未選択時 (HasSelectedFile=false):
      Drag & Drop 受付 + [ファイルを選択] + 対応形式表示
      対応: PDF / PNG / JPEG / TIFF / BMP
    選択時 (HasSelectedFile=true):
      ファイル名 + サイズ(バイト数) + ページ数 + [ファイルを変更]
    実行中 (IsExecuting):
      ProgressBar (不確定) + 「OCR処理中...」 + ステータス + 経過時間 + [キャンセル]
    成功時 (IsJobSucceeded):
      ✓ アイコン + 「OCR処理が完了しました」 + 処理時間 + [結果を表示]
    受け付け: AllowDrop=True で OnFileDrop ハンドラ

  右ペイン (Width=320 固定):
    プロバイダー: ComboBox (Azure/Google/Paddle/YomiToku 等、設定で有効化したもの)
    エンジン: ComboBox (FilteredEngines、選択 Provider に応じてフィルタ)
    テンプレート: ComboBox (登録テンプレート一覧 + 領域数表示)
    校正設定: ComboBox (IsProofreadingAvailable 時のみ表示)
    言語: ListBox 複数選択 (CheckBox 形式、Height=100)
    前処理: ListBox 複数選択 (CheckBox 形式、Height=100、IsEnabled 動的制御)
      ※ center_text は IsTable テンプレ選択時のみ有効化
    重複警告 (IsDuplicate): 黄バナー「既存の結果を使用（重複検出）」

  下部独立行 (Grid.Row=1):
    左: エラーメッセージバナー (ErrorMessage 条件付、赤背景)
    右: [キャンセル] (CanCancel 時のみ有効) + [OCR実行] (CanExecuteOcr 時のみ有効)

  キーボードショートカット:
    Ctrl+O = ファイル選択ダイアログを開く (SelectFileCommand)
    F5 = OCR実行 (ExecuteOcrCommand)
    Escape = キャンセル (CancelCommand)
```

### 画面分割

| 領域 | 内容 |
| ---- | ---- |
| Row 0 / Column 0 (*) | OCR対象ファイル (状態切替表示 + Drag&Drop) |
| Row 0 / Column 1 (320px固定) | エンジン設定 (ScrollViewer 内に Provider/Engine/Template/校正/言語/前処理) |
| Row 1 (Auto) | 下部独立行 ([エラー][キャンセル][OCR実行]) |

### メリット

| 項目 | 評価 |
| ---- | -- |
| シンプル | ◎ |
| 初心者向け | ◎ |
| 実装しやすい | ◎ |
| 学習コスト低い | ◎ |
| 責務カバー網羅 | ◎ (現実装の全責務を画面内で網羅) |

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

設定の意味単位(基本/詳細/実行)で右ペインを縦に分割した整理型。
現実装と同様に左ペインはファイル投入エリア、右ペイン内で設定をセクション分けする。

### 画面図

```text
┌──────────────────────────────┬────────────────┐
│ OCR対象ファイル               │ ── 基本設定 ── │
│                              │ プロバイダー   │
│ ※ 未選択時:                  │ [Azure v]      │
│  📄 Drag & Drop              │ エンジン       │
│  [ファイルを選択]            │ [prebuilt v]   │
│  対応形式: PDF/PNG/JPEG/     │ テンプレート   │
│           TIFF/BMP           │ [請求書 v]     │
│                              │ 校正設定       │
│ ※ 選択時:                    │ [なし v]       │
│  📄 invoice.pdf              │ 言語           │
│  サイズ / ページ数           │ ☑ 日本語 ☐ 英語│
│  [ファイルを変更]            ├────────────────┤
│                              │ ── 詳細設定 ── │
│ ※ 実行中:                    │ ☑ auto_rotate  │
│  ProgressBar                 │ ☑ deskew       │
│  「OCR処理中...」            │ ☑ binarize     │
│  ステータス + 経過時間       │ ☐ contrast     │
│  [キャンセル]                │ ☐ center_text  │
│                              │   (テーブル時) │
│ ※ 成功時:                    │ ☐ upscale      │
│  ✓ 完了 + 処理時間           │ ☐ border_padding│
│  [結果を表示]                │                │
│                              │ ⚠ 重複検出時:  │
│                              │  既存結果を使用│
└──────────────────────────────┴────────────────┘
[エラーメッセージ]         [キャンセル] [OCR実行]  ← 下部独立行

凡例:
  左ペイン: パターン① と同じ状態切替表示
    未選択/選択/実行中/成功 の 4 状態を切替
    Drag & Drop 受付

  右ペイン (3 セクションに縦分割):
    ── 基本設定 ──
      プロバイダー / エンジン / テンプレート / 校正設定 / 言語
    ── 詳細設定 ── (前処理を意味単位で集約)
      auto_rotate / deskew / binarize / contrast / center_text / upscale / border_padding
      center_text は IsTable テンプレ時のみ有効
    重複警告 (IsDuplicate): 詳細設定下に黄バナー

  下部独立行:
    エラーメッセージバナー + [キャンセル] + [OCR実行]
    ※ パターン① と同じく [OCR実行] は右ペイン内ではなく下部独立行

  キーボードショートカット: Ctrl+O / F5 / Esc (パターン① と同様)
```

### 画面分割

| 領域 | 内容 |
| ---- | ---- |
| Row 0 / Column 0 (*) | OCR対象ファイル (状態切替表示) |
| Row 0 / Column 1 (固定) | 右ペイン縦分割: 基本設定 + 詳細設定 |
| Row 1 (Auto) | 下部独立行 ([エラー][キャンセル][OCR実行]) |

### メリット

| 項目 | 評価 |
| ---- | -- |
| 視認性高い | ◎ |
| AI-OCR 感強い | ◎ |
| 設定整理しやすい | ◎ |
| 責務カバー網羅 | ◎ (右ペイン縦分割でも全責務を網羅) |

### デメリット

| 項目 | 評価 |
| ---- | -- |
| 縦長化 | △ |
| 小画面で見切れやすい | △ |
| 基本/詳細の境界判断が必要 | △ |

### 向いている

```text
・AI-OCR製品
・設定多いOCR
```

---

## パターン③ タブ型詳細設定

設定群をタブで分離し、ファイル投入エリアを最大化した整理型。小画面向き。

### 画面図

```text
┌──────────────────────────────────────────────────┐
│ OCR対象ファイル                                   │
│                                                  │
│ ※ 未選択時:                                       │
│  📄 Drag & Drop / [ファイルを選択]                │
│  対応形式: PDF/PNG/JPEG/TIFF/BMP                 │
│ ※ 選択時:                                         │
│  📄 invoice.pdf / サイズ / ページ数              │
│  [ファイルを変更]                                 │
│ ※ 実行中: ProgressBar + ステータス + [キャンセル]│
│ ※ 成功時: ✓ 完了 + 処理時間 + [結果を表示]       │
├──────────────────────────────────────────────────┤
│ [基本設定] [前処理] [言語]                       │
├──────────────────────────────────────────────────┤
│ [基本設定] タブ:                                   │
│   プロバイダー [Azure v]                         │
│   エンジン [prebuilt v]                          │
│   テンプレート [請求書(5領域) v]                  │
│   校正設定 [なし v] (IsProofreadingAvailable 時) │
│   重複警告 (IsDuplicate)                          │
│                                                  │
│ [前処理] タブ:                                    │
│   ☑ deskew    ☑ binarize                        │
│   ☐ contrast  ☐ center_text (テーブル時)        │
│   ☐ upscale   ☐ border_padding                  │
│                                                  │
│ [言語] タブ:                                      │
│   ☑ 日本語  ☐ 英語  ☐ 中国語  ☐ 多言語           │
└──────────────────────────────────────────────────┘
[エラーメッセージ]         [キャンセル] [OCR実行]  ← 下部独立行

凡例:
  上部 (ファイル領域): パターン① と同じ状態切替表示 (未選択/選択/実行中/成功)
  Drag & Drop 受付

  中央タブ (設定群):
    [基本設定] = Provider / Engine / Template / 校正設定 / 重複警告
    [前処理] = 7種の前処理オプション (auto_rotate/deskew/binarize/contrast/center_text/upscale/border_padding)
      center_text は IsTable テンプレ時のみ有効
    [言語] = 日本語/英語/中国語/多言語 の複数選択

  下部独立行:
    エラーメッセージバナー + [キャンセル] + [OCR実行]

  キーボードショートカット: Ctrl+O / F5 / Esc (パターン① と同様)
```

### 画面分割

| 領域 | 内容 |
| ---- | ---- |
| 上部 | OCR対象ファイル (状態切替表示) |
| 中央タブ | [基本設定] / [前処理] / [言語] の 3 タブ切替 |
| 下部 (Auto) | 下部独立行 ([エラー][キャンセル][OCR実行]) |

### メリット

| 項目 | 評価 |
| ---- | -- |
| コンパクト | ◎ |
| 小画面向き | ◎ |
| 整理しやすい | ◎ |
| 責務カバー網羅 | ◎ (3タブで全責務を網羅) |

### デメリット

| 項目 | 評価 |
| ---- | -- |
| 設定を見失いやすい | △ |
| OCR 感弱い | △ |
| タブ往復が必要 | △ (基本設定と前処理を同時に確認できない) |

### 向いている

```text
・初心者向け
・Web SaaS
・小画面端末
```

---

# 将来設計あり版(完成系 AI-OCR)

---

## パターン④ Dashboard 型(ABBYY 系)

Queue 監視と運用情報を常時表示する運用向け完成系。3ペイン構造で
左に Queue、中央にファイル投入、右に設定詳細を配置。

### 画面図

```text
┌────────────┬──────────────────────┬────────────────┐
│ Queue 監視 │ OCR対象ファイル       │ エンジン設定   │
│            │                      │                │
│ 実行中 12  │ ※ 未選択時:          │ プロバイダー   │
│ 完了 120   │  📄 Drag & Drop      │ [Azure v]      │
│ Error 2    │  [ファイルを選択]    │ エンジン       │
│ Retry 2    │  対応形式: PDF/PNG/  │ [prebuilt v]   │
│ CPU 40%    │   JPEG/TIFF/BMP      │ テンプレート   │
│            │                      │ [請求書 v]     │
│ Upload中 1 │ ※ 選択時:            │ 校正設定       │
│            │  📄 invoice.pdf      │ [なし v]       │
│            │  サイズ / ページ数   │ 言語           │
│            │  Cost ¥12 (将来)     │ ☑ 日本語       │
│            │  [ファイルを変更]    │ 前処理         │
│            │                      │ ☑ deskew       │
│            │ ※ 実行中:            │ ☑ binarize     │
│            │  Upload [▓▓▓░░] 60% │ ☐ center_text  │
│            │  ProgressBar         │                │
│            │  「OCR処理中...」    │                │
│            │  ステータス+経過時間 │                │
│            │  [キャンセル]        │                │
│            │                      │ ⚠ 重複検出時:  │
│            │ ※ 成功時:            │  既存結果使用  │
│            │  ✓ 完了 + 処理時間   │                │
│            │  [結果を表示]        │                │
└────────────┴──────────────────────┴────────────────┘
[エラーメッセージ]              [キャンセル] [OCR実行]

凡例:
  左ペイン (Queue 監視、将来要件):
    実行中/完了/Error/Retry 件数の常時表示
    CPU 使用率
    Upload中 = アップロード進捗を OCR 処理とは独立して表示

  中央ペイン (現実装の左ペインに相当):
    パターン① と同じ状態切替表示 (未選択/選択/実行中/成功)
    + Cost (事前推定料金、将来要件)
    + Upload 進捗 (アップロード状態の独立表示、将来要件)

  右ペイン (現実装の右ペインに相当):
    パターン① と同じエンジン設定

  下部独立行: [エラーバナー][キャンセル][OCR実行]
  キーボードショートカット: Ctrl+O / F5 / Esc
```

### 含まれる将来要件

| 機能 | 内容 |
| ---- | ---- |
| Queue 監視 | 実行中/完了/エラー件数の常時表示 |
| Retry 監視 | 自動リトライ管理 |
| CPU 監視 | OCR 負荷の可視化 |
| Upload 進捗 | アップロード状態を OCR 処理と独立表示 |
| Cost 推定 | 事前推定料金の表示 |

### メリット

| 項目 | 評価 |
| ---- | -- |
| 運用最強 | ◎ |
| 大量処理 | ◎ |
| OCR センター向き | ◎ |
| 責務カバー網羅 | ◎ (3ペインで現実装責務 + 将来要件を網羅) |

### デメリット

| 項目 | 評価 |
| ---- | -- |
| UI 重い | △ |
| 初心者難しい | △ |
| 中央ペインが狭い | △ |

### 向いている

```text
・大企業
・OCRセンター
・24h運用
```

---

## パターン⑤ OCR Studio 型(UiPath 完成系)

プレビュー・Queue・コストを 1 画面に集約したプロ向け完成系。
左に Job/Queue、中央にファイル+OCR Preview、右に設定詳細を配置する 3ペイン構造。

### 画面図

```text
┌──────────────┬──────────────────────┬────────────────┐
│ Job一覧/Queue│ ファイル + Preview    │ エンジン設定   │
│              │                      │                │
│ Queue 監視:  │ ※ 未選択時:          │ プロバイダー   │
│  実行中 12   │  📄 Drag & Drop      │ [Azure v]      │
│  完了 120    │  [ファイルを選択]    │ エンジン       │
│  Error 2     │  対応形式: PDF/PNG/  │ [prebuilt v]   │
│              │   JPEG/TIFF/BMP      │ テンプレート   │
│ Job 一覧:    │                      │ [請求書 v]     │
│ invoice01 ✓  │ ※ 選択時:            │ 校正設定       │
│ invoice02 ⏳ │  📄 invoice.pdf      │ [なし v]       │
│ invoice03 ✗  │  サイズ / ページ数   │ 言語           │
│              │  Cost ¥12 (将来)     │ ☑ 日本語       │
│              │  [ファイルを変更]    │ 前処理         │
│              │                      │ ☑ deskew       │
│              │ OCR Preview:         │ ☑ binarize     │
│              │  bbox オーバーレイ   │                │
│              │  (実行後)            │ ⚠ 重複検出時:  │
│              │                      │  既存結果使用  │
│              │ ※ 実行中:            │                │
│              │  ProgressBar         │                │
│              │  「OCR処理中...」    │                │
│              │  ステータス+経過時間 │  既存結果使用  │
│              │  [キャンセル]        │                │
│              │                      │                │
│              │ ※ 成功時:            │                │
│              │  ✓ 完了 + 処理時間   │                │
│              │  [結果を表示]        │                │
└──────────────┴──────────────────────┴────────────────┘
[エラーメッセージ]              [キャンセル] [OCR実行]

凡例:
  左ペイン (Job一覧/Queue、将来要件):
    Queue 監視: 実行中/完了/Error 件数の常時表示
    Job 一覧: 直近 Job 履歴 (アイコンで状態表示 ✓/⏳/✗)
    Job クリックで Preview 表示

  中央ペイン (ファイル + Preview):
    上半分: パターン① と同じ状態切替表示 (未選択/選択/実行中/成功)
      + Cost (事前推定料金、将来要件)
    下半分: OCR Preview (実行後の bbox オーバーレイ、将来要件)

  右ペイン (エンジン設定):
    パターン① と同じ Provider/Engine/Template/校正設定/言語/前処理
    + 重複警告

  下部独立行: [エラー][キャンセル][OCR実行]
  キーボードショートカット: Ctrl+O / F5 / Esc
```

### 含まれる将来要件

| 機能 | 内容 |
| ---- | ---- |
| bbox Preview | OCR 結果のバウンディングボックス確認 |
| Queue 監視 | Job 一覧と状態監視 (実行中/完了/Error) |
| Job 履歴 | 直近 Job のステータス表示 |
| Cost 推定 | 課金確認 |

### メリット

| 項目 | 評価 |
| ---- | -- |
| OCR 業務最強 | ◎ |
| 修正効率 | ◎ |
| 大量処理 | ◎ |
| 責務カバー網羅 | ◎ (3ペインで現実装責務 + 将来要件を網羅) |

### デメリット

| 項目 | 評価 |
| ---- | -- |
| UI 複雑 | △ |
| 実装コスト高い | △ |
| 結果確認画面と機能重複 | △ (OCR Preview / Job 一覧) |

### 向いている

```text
・AI-OCR製品
・エンタープライズ
・プロ運用
```

---

# 一押しのパターン

## 短期 (即時実装) → パターン① シンプル 2 ペイン型(最推奨)

### 理由

- **現実装の構造そのまま**、改修コストゼロ
- OCR 実行の基本フロー (ファイル投入 → 設定 → 実行 → 結果遷移) が **左右2ペイン**で完結
- 左ペインは状態切替で **「未選択 → 選択 → 実行中 → 成功」の遷移を視覚的に表現**
  - 未選択: Drag & Drop + ファイル選択ボタン
  - 選択時: ファイル情報 + ファイル変更ボタン
  - 実行中: ProgressBar + ステータス + 経過時間 + キャンセル
  - 成功時: 完了アイコン + 処理時間 + 結果遷移ボタン
- 右ペインに Provider/Engine/Template/校正設定/言語/前処理 を集約 (Width=320 固定)
- 下部独立行に **[キャンセル][OCR実行]** を常駐配置
- キーボードショートカット (Ctrl+O / F5 / Esc) で高速操作可能
- 17/17 責務完全カバー

### 学習コスト最小

ファイル投入 (左) → 設定 (右) → 実行 (下) の左右下フローで、初見でも操作の流れが直感的。

### 既存実装の良さを保つ

ファイルバリデーション (拡張子/サイズ/ページ数)、重複検出、ジョブステータスポーリング、結果画面への自動遷移など、現実装の堅実な動作を変更不要。

## 長期 (将来ゴール) → パターン④ Dashboard 型 (大規模運用時の選択肢)

OCR センター / 24時間運用に進化する場合、**監視盤要素を左ペインに追加**するのが現実的。
- Queue 監視 (実行中/完了/Error/Retry/CPU)
- Upload 進捗の独立表示 (大容量ファイル時の UX 改善)
- Cost 推定の事前提示

ただし通常運用では①で十分。Dashboard 型は別画面として併設する選択もあり。
