# C# 基礎知識まとめ

---

## 1. 変数と型

変数とは「値を入れる箱」のこと。箱の種類（型）を指定してから使う。

### 主な型一覧

| 型 | 説明 | 例 |
| --- | --- | --- |
| `int` | 整数 | `1`, `100`, `-5` |
| `double` | 小数 | `3.14`, `-0.5` |
| `bool` | 真偽値 | `true`, `false` |
| `string` | 文字列 | `"こんにちは"`, `"A001"` |
| `char` | 1文字 | `'A'`, `'0'` |

### 変数の宣言と代入

```csharp
int age = 25;
string name = "田中";
bool isActive = true;
```

### var キーワード

右辺から型が明らかな場合は `var` で省略できる。

```csharp
var age = 25;      // int と推論される
var name = "田中"; // string と推論される
```

※ 型が不明確な場合は `var` を使わず明示する。

---

## 2. 文字列

### 結合

```csharp
string firstName = "田中";
string lastName  = "太郎";
string full = firstName + " " + lastName;  // "田中 太郎"
```

### 文字列補間（推奨）

```csharp
string message = $"名前は {firstName} です";  // "名前は 田中 です"
```

### 空文字の初期化

```csharp
string text = string.Empty;  // "" と同じ。null より安全。
```

### よく使うメソッド

| メソッド | 説明 |
| --- | --- |
| `text.Length` | 文字数を返す |
| `text.Contains("A")` | "A" が含まれるか |
| `text.Replace("A","B")` | A を B に置換 |
| `string.IsNullOrEmpty(text)` | null または空文字か判定 |

---

## 3. 条件分岐（if文）

### 基本形

```csharp
if (条件)
{
    // 条件が true のとき実行
}
else
{
    // 条件が false のとき実行
}
```

### else if

```csharp
if (score >= 80)
{
    Console.WriteLine("優");
}
else if (score >= 60)
{
    Console.WriteLine("良");
}
else
{
    Console.WriteLine("不可");
}
```

### ガード節（早期リターン）

`return` や `throw` で終わる `if` の後に `else` は不要。

```csharp
// ❌ NG（アンチパターン）
if (value < 0)
{
    return false;
}
else
{
    // 処理
}

// ✅ OK（推奨）
if (value < 0)
{
    return false;
}
// 処理（else 不要）
```

---

## 4. 繰り返し（ループ）

### for文

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i);  // 0,1,2,3,4
}
```

### foreach文

```csharp
var names = new List<string> { "田中", "佐藤", "鈴木" };
foreach (var name in names)
{
    Console.WriteLine(name);
}
```

### while文

```csharp
int count = 0;
while (count < 3)
{
    Console.WriteLine(count);
    count++;
}
```

---

## 5. クラスとオブジェクト

クラスはデータと処理をまとめた「設計図」。オブジェクトはその設計図から作った「実体」。

### クラスの定義

```csharp
public class Person
{
    public string Name { get; set; } = string.Empty;
    public int    Age  { get; set; }
}
```

### オブジェクトの生成（インスタンス化）

```csharp
var person = new Person();
person.Name = "田中";
person.Age  = 25;
```

### オブジェクト初期化子（まとめて初期化）

```csharp
var person = new Person { Name = "田中", Age = 25 };
```

---

## 6. プロパティ

クラスのデータを安全に公開する仕組み。`get`（読み取り）と `set`（書き込み）で制御できる。

```csharp
public int Age { get; set; }           // 読み書き可能
public int Age { get; }                // 読み取り専用
public int Age { get; init; }          // 初期化時のみ書き込み可能
public int Age { get; private set; }   // 外から書き込み不可
```

### init の特徴

```csharp
var result = new AllocationResult { OrderId = 1 };  // OK
result.OrderId = 2;  // ❌ エラー（init なので変更不可）
```

---

## 7. メソッド

処理をまとめて名前をつけたもの。何度でも呼び出せる。

### 基本形

```csharp
public 戻り値の型 メソッド名(引数の型 引数名)
{
    // 処理
    return 戻り値;
}
```

### 例

```csharp
public int Add(int a, int b)
{
    return a + b;
}

var result = Add(3, 5);  // result = 8
```

### void（戻り値なし）

```csharp
public void PrintMessage(string message)
{
    Console.WriteLine(message);
}
```

### static（インスタンス不要で呼べる）

```csharp
public static int Multiply(int a, int b)
{
    return a * b;
}

var result = MyClass.Multiply(3, 4);
```

---

## 8. アクセス修飾子

クラス・プロパティ・メソッドの「見える範囲」を制御する。

| 修飾子 | 説明 |
| --- | --- |
| `public` | どこからでもアクセス可能 |
| `private` | 同じクラス内からのみアクセス可能 |
| `protected` | 同じクラスと継承クラスからアクセス可能 |
| `internal` | 同じプロジェクト内からアクセス可能 |

**使い分けの基本方針：**
- 外から使うもの → `public`
- 内部だけで使うもの → `private`

---

## 9. コンストラクタ

オブジェクト生成時に自動的に呼ばれる特別なメソッド。初期化処理を書く場所。

```csharp
public class Order
{
    public int    OrderId     { get; set; }
    public string ProductCode { get; set; }

    // コンストラクタ（クラス名と同じ名前、戻り値なし）
    public Order(int orderId, string productCode)
    {
        OrderId     = orderId;
        ProductCode = productCode;
    }
}

var order = new Order(1, "A001");
```

---

## 10. インターフェース

「このクラスは必ずこのメソッド・プロパティを持つ」という約束。継承とは異なり、複数実装できる。

### 定義

```csharp
public interface IAnimal
{
    string Name { get; }
    void Speak();
}
```

### 実装

```csharp
public class Dog : IAnimal
{
    public string Name { get; } = "犬";
    public void Speak()
    {
        Console.WriteLine("ワン");
    }
}
```

### よく使う組み込みインターフェース

| インターフェース | 説明 |
| --- | --- |
| `IEnumerable<T>` | foreach でループできる |
| `IList<T>` | リスト操作ができる |
| `IReadOnlyList<T>` | 読み取り専用リスト |
| `INotifyPropertyChanged` | プロパティ変更通知（WPF で重要） |
| `ICommand` | ボタン操作のバインディング（WPF で重要） |

---

## 11. コレクション

複数のデータをまとめて管理する入れ物。

### List\<T\>

```csharp
var list = new List<string>();
list.Add("A001");
list.Add("B002");
list.Remove("A001");
int count = list.Count;
```

### Dictionary\<TKey, TValue\>

キーと値のペアで管理する。検索が速い。

```csharp
var dict = new Dictionary<string, int>();
dict["A001"] = 3;
dict["B002"] = 10;
var qty = dict["A001"];  // 3

// キーが存在するか確認
if (dict.ContainsKey("A001")) { ... }
```

### ObservableCollection\<T\>（WPF専用）

データが変更されると画面に自動通知する List。

```csharp
var orders = new ObservableCollection<Order>();
orders.Add(new Order { OrderId = 1 });  // 画面が自動更新される
```

---

## 12. LINQ

コレクションを検索・変換・集計する機能。

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5 };

// 条件に一致するものを絞り込む
var even = numbers.Where(n => n % 2 == 0);  // 2, 4

// 変換する
var doubled = numbers.Select(n => n * 2);   // 2,4,6,8,10

// 条件に一致するものが1つ以上あるか
bool hasLarge = numbers.Any(n => n > 4);    // true

// 合計・最大・最小
int sum = numbers.Sum();   // 15
int max = numbers.Max();   // 5

// リストに変換
var list = numbers.Where(n => n > 2).ToList();  // 3,4,5

// ディクショナリに変換
var dict = stocks.ToDictionary(s => s.ProductCode, s => s.Quantity);
```

### 注意

```csharp
// ❌ NG（無駄なリスト生成）
var list = items.ToList();
bool exists = list.Any(x => x.Id == 1);

// ✅ OK
bool exists = items.Any(x => x.Id == 1);
```

---

## 13. null と null 安全

`null` は「何もない」状態。null のまま使うとエラーになる。

```csharp
// null チェック
if (text != null)
{
    Console.WriteLine(text.Length);
}

// null 条件演算子 ?.
int? length = text?.Length;  // text が null なら null を返す

// null 合体演算子 ??
string result = text ?? string.Empty;  // null なら string.Empty を使う
```

### null 非許容（Nullable enable）

プロジェクトに `<Nullable>enable</Nullable>` を設定すると、通常の `string` 型に null を代入するとコンパイル警告が出る。null を許容したい場合は `string?` と書く。

```csharp
string  name = null;   // ⚠️ 警告
string? name = null;   // ✅ OK（null 許容型）
```

---

## 14. 例外処理

エラーが発生したときの対処を記述する。

### try-catch

```csharp
try
{
    var result = int.Parse("abc");  // 変換失敗
}
catch (FormatException ex)
{
    Console.WriteLine($"エラー: {ex.Message}");
}
finally
{
    // 成功・失敗に関わらず必ず実行される
}
```

### 例外をスロー

```csharp
public void SetQuantity(int value)
{
    if (value < 0)
    {
        throw new ArgumentException("数量は0以上にしてください。");
    }
    Quantity = value;
}
```

### 業務エラーは例外より戻り値で表現する

在庫不足などのビジネス上のエラーは例外を使わず、結果クラス（`AllocationResult` など）に含めるのが良い設計。

---

## 15. 命名規則

C# では以下の命名規則が標準とされている。

| スタイル | 対象 | 例 |
| --- | --- | --- |
| `PascalCase`（先頭大文字） | クラス名・メソッド名・プロパティ名 | `OrderId`, `ExecuteAllocation` |
| `camelCase`（先頭小文字） | ローカル変数・メソッド引数 | `orderId`, `productCode` |
| `_camelCase`（アンダースコア+小文字） | プライベートフィールド | `_orderId`, `_allocationService` |

※ `ALL_CAPS` は C# では使わない（Java や C の習慣）。定数も `PascalCase` または `_camelCase` が一般的。

---

## 16. WPF / MVVM で重要な概念

### INotifyPropertyChanged

プロパティの値が変わったことを画面に通知するインターフェース。これを実装しないと ViewModel の値を変えても画面が更新されない。

```csharp
public event PropertyChangedEventHandler? PropertyChanged;

private void RaisePropertyChanged(string propertyName) =>
    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
```

### ObservableCollection\<T\>

要素の追加・削除を画面に自動通知する List。通常の `List<T>` では画面は更新されない。

### ICommand / RelayCommand

ボタンのクリックを ViewModel で受け取る仕組み。コードビハインド（xaml.cs）に処理を書かずに済む。

### Binding

ViewModel のプロパティと画面部品を紐付ける仕組み。`Text="{Binding Name}"` のように書く。

### MVVM の原則

| 層 | 役割 |
| --- | --- |
| **View**（画面） | 見た目だけ。ロジックを書かない。 |
| **ViewModel**（仲介役） | 画面の状態管理。UI部品を知らない。 |
| **Model**（データ） | データ構造のみ。UI を知らない。 |
| **Service**（処理） | 業務ロジックのみ。UI を知らない。 |
