# ResultView 処理日フィルタ機能 設計書

**機能名**: OCR結果一覧 処理日（完了日時）フィルタ
**対象画面**: ResultView

---

## 1. 背景・目的

OCR結果一覧画面（ResultView）のジョブ履歴リストに、現在テキスト検索とステータス絞り込みのみが提供されている。
大量のジョブが蓄積された場合に日付での絞り込みが困難なため、「処理日（OCR処理完了日時）」による期間フィルタを追加する。

---

## 2. 現状確認

### 2-1. ResultView 構造

| ペイン | 幅 | 内容 |
|--------|-----|------|
| 左 | 250px | ジョブ履歴リスト + 検索・フィルタ |
| 中央 | * | 画像ビューア（バウンディングボックス付き） |
| 右 | 350px | テキストパネル |

### 2-2. 既存フィルタ

| フィルタ | 種別 | バインディング |
|---------|------|--------------|
| テキスト検索 | TextBox | `SearchQuery` |
| ステータス | ComboBox | `StatusFilter` |

フィルタは `FilteredJobHistory`（LINQ / in-memory）で即時反映される。

### 2-3. 日付項目の現状

| フィールド | APIレスポンス (`JobStatusResponse`) | クライアントモデル (`JobHistoryItem`) |
|-----------|-------------------------------------|--------------------------------------|
| `created_at` | あり | あり（表示・ソートで使用中） |
| `started_at` | あり | なし |
| `completed_at` | あり | **なし（今回追加対象）** |

**「処理日（OCR処理完了日時）」= `completed_at` はクライアントモデルに存在しない。**
今回はクライアントモデルへの追加（理想案）で実装する。

### 2-4. ローカル永続化の仕組み

- ジョブ履歴は `%LOCALAPPDATA%\OcrViewer\job-history.json` にローカル保存
- `JobHistoryRepository` 内の `JobHistoryItemDto` を経由してシリアライズ
- API取得後にジョブを `Add()` / `Update()` する際に `JobHistoryItem` を生成

---

## 3. 画面設計

### 3-1. 追加箇所

左ペインの既存フィルタエリア（ステータスフィルタ）の直下に追加する。

```
┌────────────────────────────┐
│ ジョブ履歴                  │
├────────────────────────────┤
│ [🔍 検索...]                │  ← 既存
│ [ステータス ▼]              │  ← 既存
│ ─────────────────────────  │
│ 処理日                      │  ← 追加（ラベル）
│ From: [yyyy/MM/dd      ]   │  ← 追加（DatePicker）
│ To:   [yyyy/MM/dd      ]   │  ← 追加（DatePicker）
├────────────────────────────┤
│ リスト ...                  │
└────────────────────────────┘
```

### 3-2. 画面項目定義

| 項目名 | 種別 | バインディング | 初期値 | 役割 |
|--------|------|--------------|--------|------|
| 処理日（ラベル） | Label | - | - | セクション見出し |
| From | DatePicker | `DateFrom` | null（未選択） | フィルタ開始日 |
| To | DatePicker | `DateTo` | null（未選択） | フィルタ終了日 |

### 3-3. ユーザー操作と挙動

| 操作 | 挙動 |
|------|------|
| From または To を選択 | 即時にリスト再フィルタ |
| From のみ入力 | From 以降のジョブのみ表示 |
| To のみ入力 | To 以前のジョブのみ表示 |
| 両方未入力 | 日付フィルタなし（全件） |
| From > To | 自動入れ替えして絞り込み（エラー表示なし） |
| 日付をクリア | その条件を解除 |

---

## 4. 機能仕様

### 4-1. フィルタ条件

```
CompletedAt >= From（指定日の 00:00:00）
AND
CompletedAt <= To（指定日の 23:59:59.9999999）
```

### 4-2. From / To の時刻扱い

| 入力 | 比較値 |
|------|-------|
| From | 選択日の 00:00:00 |
| To | 選択日の 23:59:59.9999999（`AddDays(1).AddTicks(-1)`） |

### 4-3. null（未入力）時の動作

```
From が null → 下限なし
To が null   → 上限なし
両方 null    → 日付フィルタ適用しない
```

### 4-4. From > To の自動入れ替え

```csharp
var from = DateFrom;
var to   = DateTo;

if (from.HasValue && to.HasValue && from > to)
    (from, to) = (to, from);
```

### 4-5. CompletedAt が null のジョブの扱い

日付フィルタが有効（From または To が入力済み）の場合、`CompletedAt == null` のジョブは一覧から除外される。
（= 処理中・待機中ジョブは日付フィルタ中は非表示になる）

---

## 5. 変更対象ファイル

| ファイル | 変更内容 |
|---------|--------|
| `OcrViewer.Sdk/Domain/Models/JobHistoryItem.cs` | `CompletedAt: DateTime?` プロパティ追加 |
| `OcrViewer.Sdk/Infrastructure/JobHistoryRepository/JobHistoryRepository.cs` | `JobHistoryItemDto` に `CompletedAt` 追加、`ToDto()` / `FromDto()` にマッピング追加 |
| ジョブ結果受信箇所（`Add()` / `Update()` 呼び出し元） | `CompletedAt` を `JobStatusResponse.CompletedAt` からセット |
| `OcrViewer/Presentation/ViewModels/JobHistoryItemViewModel.cs` | `CompletedAt` プロパティ（または `CompletedAtFormatted`）を公開 |
| `OcrViewer/Presentation/ViewModels/ResultViewModel.cs` | `DateFrom`, `DateTo` プロパティ追加、`FilteredJobHistory` にフィルタロジック追加 |
| `OcrViewer/Presentation/Views/ResultView.xaml` | From / To DatePicker UI 追加 |

---

## 6. 実装方針（概要）

### ResultViewModel への追加イメージ

```csharp
[ObservableProperty]
[NotifyPropertyChangedFor(nameof(FilteredJobHistory))]
private DateTime? _dateFrom;

[ObservableProperty]
[NotifyPropertyChangedFor(nameof(FilteredJobHistory))]
private DateTime? _dateTo;

public IEnumerable<JobHistoryItemViewModel> FilteredJobHistory
{
    get
    {
        var query = JobHistory.AsEnumerable();

        // 既存フィルタ（変更なし）
        if (!string.IsNullOrWhiteSpace(SearchQuery)) { ... }
        if (!string.IsNullOrWhiteSpace(StatusFilter)) { ... }

        // 処理日フィルタ（追加）
        var from = DateFrom;
        var to   = DateTo;
        if (from.HasValue && to.HasValue && from > to)
            (from, to) = (to, from);

        if (from.HasValue)
            query = query.Where(j => j.CompletedAt >= from.Value.Date);

        if (to.HasValue)
            query = query.Where(j => j.CompletedAt <= to.Value.Date.AddDays(1).AddTicks(-1));

        return query.OrderByDescending(j => j.CreatedAt);
    }
}
```

---

## 7. 影響範囲

| 対象 | 影響 |
|------|------|
| 既存テキスト検索 | 変更なし |
| 既存ステータスフィルタ | 変更なし |
| `JobHistoryItem` レコード型 | nullable プロパティ追加のみ（既存参照に影響なし） |
| ローカル保存済みの `job-history.json` | `CompletedAt` フィールドがない既存データは `null` として読み込まれる（後方互換） |
| API 通信 | `JobStatusResponse.CompletedAt` は既に存在するため変更なし |

---

## 8. 懸念点・確認事項

| 項目 | 内容 |
|------|------|
| **CompletedAt のセット箇所の特定** | `Add()` / `Update()` 呼び出し元でジョブ完了時に `CompletedAt` がセットされているか確認が必要 |
| **未完了ジョブの非表示** | 日付フィルタ中は処理中・待機中ジョブが非表示になる。UI上の注記が必要か要検討 |
| **DatePicker 幅** | 左ペインは 250px 固定のため From / To は縦並び推奨 |
| **既存 JSON データの互換性** | `FromDto()` で `CompletedAt` が欠如している場合の `null` 扱いを確認すること |
