# 【C# 標準ライブラリ】元から使える主なクラス一覧

## 1. 基本型・共通

| クラス | 説明 |
| --- | --- |
| `object` | すべてのクラスの親（基本の型） |
| `string` | 文字列 |
| `int`, `long`, `bool` | 数値や真偽値 |
| `decimal`, `double` | 小数 |
| `DateTime` | 日付・時刻の操作 |
| `TimeSpan` | 時間の長さ |
| `Guid` | 一意のIDを生成・保持 |
| `Enum` | 列挙型の親クラス |

## 2. コレクション（配列・リスト・辞書）

| クラス | 説明 |
| --- | --- |
| `Array` | 配列の親クラス |
| `List<T>` | 順序付きの可変リスト |
| `Dictionary<TKey, TValue>` | キーと値のセット（連想配列） |
| `HashSet<T>` | 重複しない集合 |
| `Queue<T>` | 先入れ先出し（FIFO） |
| `Stack<T>` | 後入れ先出し（LIFO） |
| `Collection<T>` | Listより少し機能が制限された型 |

## 3. ファイル・フォルダ操作（System.IO）

| クラス | 説明 |
| --- | --- |
| `File` | ファイル全体の読み書き |
| `FileInfo` | ファイルの詳細情報 |
| `Directory` | フォルダの作成・削除など |
| `DirectoryInfo` | フォルダの詳細情報 |
| `StreamReader` | ファイルの読み込み |
| `StreamWriter` | ファイルへの書き込み |
| `FileStream` | バイナリファイルの読み書き |
| `Path` | ファイルパス操作 |

## 4. データベース（System.Data.SqlClient）

| クラス | 説明 |
| --- | --- |
| `SqlConnection` | SQL Serverとの接続 |
| `SqlCommand` | SQL文の実行 |
| `SqlDataReader` | SELECT結果の読み取り |
| `SqlTransaction` | トランザクション管理 |
| `SqlDataAdapter` | データベース ↔ DataTable の橋渡し |
| `DataTable` | 表形式データを保持するクラス |
| `DataSet` | 複数の DataTable を持てるデータの集合 |

## 5. 文字列・テキスト操作

| クラス | 説明 |
| --- | --- |
| `StringBuilder` | 高速な文字列連結 |
| `Regex` | 正規表現の検索・置換 |
| `CultureInfo` | ロケール（言語・国）情報 |
| `Convert` | 型変換 |

## 6. エラー処理・デバッグ

| クラス | 説明 |
| --- | --- |
| `Exception` | 例外（エラー）全体の基底クラス |
| `ArgumentException` | 引数エラー |
| `NullReferenceException` | null参照時のエラー |
| `Debug`, `Trace` | デバッグ出力（System.Diagnostics） |

## 7. 日付・時刻関連

| クラス | 説明 |
| --- | --- |
| `DateTime` | 現在日時、過去・未来の日時 |
| `TimeSpan` | 2つの日時の差、間隔 |
| `DateOnly` | 日付のみ（.NET 6以降） |
| `TimeOnly` | 時刻のみ（.NET 6以降） |

## 8. その他 よく使う便利クラス

| クラス | 説明 |
| --- | --- |
| `Random` | 乱数の生成 |
| `Environment` | 環境情報（OS名、改行コードなど） |
| `Process` | 外部プログラムの起動 |
| `Stopwatch` | 処理時間の計測 |
| `Console` | コンソール入出力 |
| `Math` | 数学関数（√、絶対値、三角関数など） |
| `BitConverter` | バイト配列との変換 |

---

## 補足

- 多くは `System` 名前空間やその配下にあります。
- Visual Studio の `F12` キーや `Ctrl + .` で名前空間の追加が簡単にできます。
