# ResultView 処理日フィルタ機能 設計書

**機能名**: OCR結果一覧 投入日フィルタ
**対象画面**: ResultView

---

## 1. 背景・目的

OCR結果一覧画面（ResultView）のジョブ履歴リストに、現在テキスト検索とステータス絞り込みのみが提供されている。
大量のジョブが蓄積された場合に日付での絞り込みが困難なため、「処理日（ジョブ投入日時）」による期間フィルタを追加する。

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
| `completed_at` | あり | なし |

**今回のフィルタには既存の `created_at`（投入日時）を使用する。クライアントモデルへの追加は不要。**

### 2-4. ローカル永続化の仕組み

- ジョブ履歴は `%LOCALAPPDATA%\OcrViewer\job-history.json` にローカル保存
- `JobHistoryRepository` 内の `JobHistoryItemDto` を経由してシリアライズ
- API取得後にジョブを `Add()` / `Update()` する際に `JobHistoryItem` を生成

> **補足**: 今回の変更はクライアント側のみ。サーバー側DBへの変更は一切なし。

---

## 3. 画面設計

### 3-1. 追加箇所

左ペインへの縦積みは一覧表示領域を圧迫するため、ヘッダー右端にフィルターボタンを配置し、押下時にダイアログを表示する形式とする。

```
┌────────────────────────────┐
│ ジョブ履歴         [📅]    │  ← ヘッダー右端にフィルターボタン追加
├────────────────────────────┤
│ [🔍 検索...]                │  ← 既存
│ [ステータス ▼]              │  ← 既存
├────────────────────────────┤
│ リスト ...                  │  ← 一覧領域を維持
└────────────────────────────┘
```

フィルターボタン押下時に**非モーダルダイアログ**を表示する（バックグラウンドの一覧操作をブロックしない）：

```
┌─────────────────────────────┐
│ 投入日フィルター                    │
├─────────────────────────────┤
│ 投入日で絞り込みます（完了日ではありません） │
│ 投入日 From: [yyyy/MM/dd   ]       │
│ 投入日 To:   [yyyy/MM/dd   ]       │
│ [メッセージ表示エリア]              │
├─────────────────────────────┤
│ [クリア]          [閉じる]  │
└─────────────────────────────┘
```

フィルターが有効な場合はボタンにインジケーター（例：青点）を表示し、設定中であることを示す。

> **実装方針**: `QuickDictionaryPanel` と同パターンで、`ResultView.xaml` 内に `DateFilterDialog` として埋め込む。
> `IsDateFilterVisible`（bool）を ViewModel に持ち、XAML の `Visibility` バインディングで表示／非表示を切り替える。
> `DateFrom` / `DateTo` は `ResultViewModel` のプロパティとして保持し、ダイアログ表示中も即時フィルタが反映される。
> 別ウィンドウ・`IDialogService` は使用しない（非モーダル）。

### 3-2. 画面項目定義

| 項目名 | 種別 | バインディング | 初期値 | 役割 |
|--------|------|--------------|--------|------|
| フィルターボタン | Button | `OpenDateFilterCommand` | - | 投入日フィルターダイアログを開く |
| フィルター有効インジケーター | 装飾 | `IsDateFilterActive` | false | フィルター設定中を示す |
| 説明テキスト | TextBlock | （固定文字列） | - | 「投入日で絞り込みます（完了日ではありません）」を常時表示 |
| From | DatePicker | `DateFrom` | null（未選択） | フィルタ開始日（カレンダー選択・手入力どちらも可） |
| To | DatePicker | `DateTo` | null（未選択） | フィルタ終了日（カレンダー選択・手入力どちらも可） |
| メッセージ表示エリア | TextBlock | `SwapMessage` | null | 入れ替え通知を一時表示 |
| クリアボタン | Button | `ClearDateFilterCommand` | - | From / To を両方リセット |
| 閉じるボタン | Button | `CloseDateFilterCommand` | - | ダイアログを閉じる（`IsDateFilterVisible = false`、フィルタ設定は維持） |
| - | - | `IsFilteredEmpty` | false | 履歴はあるがフィルタ結果が0件のとき `true`。XAML の空状態メッセージ表示にバインドする |

### 3-3. ユーザー操作と挙動

| 操作 | 挙動 |
|------|------|
| フィルターボタンを押す | 非モーダルダイアログが開く（一覧は操作可能） |
| From または To を選択 | 即時にリスト再フィルタ |
| From のみ入力 | From 以降のジョブのみ表示 |
| To のみ入力 | To 以前のジョブのみ表示 |
| 両方未入力 | 日付フィルタなし（全件） |
| From > To | From と To の値を自動で入れ替え、ダイアログ内に「開始日と終了日を入れ替えました」と一時表示する |
| クリアボタンを押す | From / To を両方リセット、フィルタ解除 |
| 閉じるボタンを押す | ダイアログを閉じる（フィルタ設定は維持） |

### 3-4. 空結果時の表示

| 状況 | 表示内容 |
|------|---------|
| 履歴が1件もない | 「履歴がありません」（既存） |
| 履歴はあるがフィルタ結果が0件 | 「条件に一致する履歴がありません」（追加） |

現状の空状態表示は `JobHistory.Count` を参照しているため、フィルタ結果が0件でも何も表示されない。`FilteredJobHistory` の件数を合わせて判定するよう修正が必要。

> **条件の一般化**: 検索語・ステータス・投入日のいずれかが適用中（`SearchQuery` / `StatusFilter` / `IsDateFilterActive` のいずれかが有効）かつ `FilteredJobHistory` が0件の場合に「条件に一致する履歴がありません」を表示する。

> **実装上の注意**: `FilteredJobHistory` は `IEnumerable<>` 型のため XAML から件数に直接バインドできない。§3-2 の `IsFilteredEmpty` プロパティ（`bool`）を経由してバインドする。

---

## 4. 機能仕様

### 4-1. フィルタ条件

```
CreatedAt >= From（指定日の 00:00:00）
AND
CreatedAt < To（指定日の翌日 00:00:00）← 半開区間
```

> **タイムゾーン**: `CreatedAt` は `DateTime.Now`（ローカル時刻）で保存されるため、UTC変換は不要。DatePicker の日付とそのまま比較できる。

### 4-2. From / To の時刻扱い

| 入力 | 比較値 |
|------|-------|
| From | 選択日の 00:00:00（`from.Value.Date`） |
| To | 選択日の翌日 00:00:00（`to.Value.Date.AddDays(1)`）との半開区間比較 |

### 4-3. null（未入力）時の動作

```
From が null → 下限なし
To が null   → 上限なし
両方 null    → 日付フィルタ適用しない
```

### 4-4. From > To の自動入れ替えと通知

`DateFrom` / `DateTo` は `[ObservableProperty]` であるため、プロパティへの代入は `OnDateFromChanged` / `OnDateToChanged` を再帰的に呼び出す。
これを避けるため、**バックフィールドを直接書き換えてから `OnPropertyChanged` を手動呼び出し**する方式を採用する。

> **注意**: `_dateFrom` / `_dateTo` の宣言に `[NotifyPropertyChangedFor]` 属性は付けない。スワップ時はバックフィールドを直接書き換えるためセッターが走らず属性が機能しない。通常代入時と統一するため通知はすべて `OnDateFromChanged` / `OnDateToChanged` 内の手動呼び出しに統一する（属性を付けると通常代入時に二重通知になる）。

```csharp
// OnDateFromChanged（または OnDateToChanged）の中
partial void OnDateFromChanged(DateTime? value)
{
    SwapMessage = null;  // 前回の通知をクリア

    if (_dateTo.HasValue && value.HasValue && value > _dateTo)
    {
        var tmp = _dateTo;
        _dateTo = value;        // バックフィールド直書き（OnChanged を再帰呼び出ししない）
        _dateFrom = tmp;        // バックフィールド直書き
        OnPropertyChanged(nameof(DateFrom));          // DatePicker の表示を更新
        OnPropertyChanged(nameof(DateTo));            // DatePicker の表示を更新
        OnPropertyChanged(nameof(FilteredJobHistory));
        OnPropertyChanged(nameof(IsDateFilterActive));
        OnPropertyChanged(nameof(IsFilteredEmpty));
        SwapMessage = "開始日と終了日を入れ替えました";
        return;
    }

    OnPropertyChanged(nameof(FilteredJobHistory));
    OnPropertyChanged(nameof(IsDateFilterActive));
    OnPropertyChanged(nameof(IsFilteredEmpty));
}
```

`SwapMessage` はダイアログ内に一時表示し、次回入力変更時（`OnDateFromChanged` / `OnDateToChanged` の冒頭）またはダイアログを開いたとき（`OpenDateFilter`）にクリアする。

---

## 5. 変更対象ファイル

| ファイル | 変更内容 |
|---------|--------|
| `client/OcrViewer/Presentation/ViewModels/ResultViewModel.cs` | `DateFrom`, `DateTo`, `IsDateFilterActive`, `IsDateFilterVisible`（フィルターダイアログ表示制御）, `SwapMessage`, `IsFilteredEmpty` プロパティ追加、`FilteredJobHistory` にフィルタロジック追加、`OpenDateFilterCommand` / `ClearDateFilterCommand` / `CloseDateFilterCommand` 追加 |
| `client/OcrViewer/Presentation/Views/ResultView.xaml` | ヘッダーにフィルターボタン追加、空結果メッセージ追加 |
| `client/OcrViewer/Presentation/Views/DateFilterDialog.xaml`（新規） | From / To DatePicker・メッセージ表示・クリア／閉じるボタンを含む非モーダルダイアログ（`ResultView.xaml` 内に埋め込み、`IsVisible` で表示制御） |

---

## 6. 実装方針（概要）

### ResultViewModel への追加イメージ

```csharp
// [NotifyPropertyChangedFor] は付けない（§4-4 参照）
[ObservableProperty]
private DateTime? _dateFrom;

[ObservableProperty]
private DateTime? _dateTo;

[ObservableProperty]
private bool _isDateFilterVisible;

[ObservableProperty]
private string? _swapMessage;

public bool IsDateFilterActive => DateFrom.HasValue || DateTo.HasValue;

/// <summary>
/// 履歴は存在するがフィルタ結果が0件のとき true
/// XAML の「条件に一致する履歴がありません」表示にバインドする
/// </summary>
public bool IsFilteredEmpty =>
    JobHistory.Count > 0 &&
    !FilteredJobHistory.Any() &&
    (IsDateFilterActive || !string.IsNullOrWhiteSpace(SearchQuery) || !string.IsNullOrWhiteSpace(StatusFilter));

public IEnumerable<JobHistoryItemViewModel> FilteredJobHistory
{
    get
    {
        var query = JobHistory.AsEnumerable();

        // 既存フィルタ（変更なし）
        if (!string.IsNullOrWhiteSpace(SearchQuery)) { ... }
        if (!string.IsNullOrWhiteSpace(StatusFilter)) { ... }

        // 処理日フィルタ（追加）
        // OnDateFromChanged / OnDateToChanged がスワップを保証するため DateFrom <= DateTo は常に成立
        if (DateFrom.HasValue)
            query = query.Where(j => j.CreatedAt >= DateFrom.Value.Date);

        if (DateTo.HasValue)
            query = query.Where(j => j.CreatedAt < DateTo.Value.Date.AddDays(1));  // 半開区間

        return query.OrderByDescending(j => j.CreatedAt);
    }
}

[RelayCommand]
private void OpenDateFilter()
{
    SwapMessage = null;   // ダイアログを開くたびに前回の通知を消す
    IsDateFilterVisible = true;
}

[RelayCommand]
private void CloseDateFilter() => IsDateFilterVisible = false;

[RelayCommand]
private void ClearDateFilter()
{
    DateFrom = null;
    DateTo = null;
}
```

> **`IsFilteredEmpty` の通知**: `JobHistory` は `ObservableCollection` を `Clear()` / `Add()` で更新するためプロパティ参照が変わらず `[NotifyPropertyChangedFor]` が機能しない。コンストラクタで `JobHistory.CollectionChanged += (_, _) => OnPropertyChanged(nameof(IsFilteredEmpty));` を購読して対応する。

---

## 7. 影響範囲

| 対象 | 影響 |
|------|------|
| 既存テキスト検索 | 変更なし |
| 既存ステータスフィルタ | 変更なし |
| `JobHistoryItem` レコード型 | 変更なし |
| `job-history.json` | 変更なし |
| API 通信 | 変更なし |
| サーバー側DB | 変更なし |

---

## 8. 懸念点・確認事項

| 項目 | 内容 |
|------|------|
| **未完了ジョブの表示** | 日付フィルタ中も処理中・待機中ジョブは `CreatedAt` を持つため除外されない |
| **空結果時の表示** | `FilteredJobHistory` の件数を見て「条件に一致する履歴がありません」を表示する |
