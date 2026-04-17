# OS コマンド・インジェクション

---

## 1. OS コマンド・インジェクションとは

ウェブアプリケーションがユーザーの入力値を使って OS コマンドを実行する際に、悪意のあるコマンドを混入させてサーバーを不正操作する攻撃。

```
// 攻撃例：ファイル名の入力欄に以下を入力される
report.pdf; rm -rf /

// 結果として実行されるコマンド（ファイル削除が走る）
$ cat report.pdf; rm -rf /
```

---

## 2. 発生しうる脅威

### サーバ内ファイルの閲覧・改ざん・削除

- 重要情報（個人情報・認証情報）の漏えい
- 設定ファイルの改ざんによるシステム破壊

### 不正なシステム操作

- 意図しない OS のシャットダウン・再起動
- ユーザーアカウントの追加・変更
- サービスの停止・起動

### 不正なプログラムのダウンロード・実行

- ウイルス・ワーム・ボット等への感染
- バックドアの設置（攻撃者が継続的にアクセスできる裏口）

### 他のシステムへの攻撃の踏み台

- サービス不能攻撃（DoS/DDoS）の発信元にされる
- 迷惑メール（スパム）の大量送信
- 他システム攻略のための調査・スキャン

---

## 3. 注意が必要なウェブサイトの特徴

運営主体やウェブサイトの性質を問わず、**外部プログラムを呼び出し可能な関数・機能を使用しているウェブアプリケーション**はすべて対象になりえる。

### シェルを起動できる言語機能の例

| 言語 | 危険な関数・機能 |
| --- | --- |
| C# | `Process.Start()`、`Shell()` |
| PHP | `exec()`、`system()`、`shell_exec()`、バッククォート `` ` `` |
| Python | `os.system()`、`subprocess.call()` |
| Java | `Runtime.exec()`、`ProcessBuilder` |
| Node.js | `child_process.exec()` |

---

## 4. 対策1：シェルを起動できる言語機能の利用を避ける

OS コマンドを呼び出す必要がある処理は、**言語・フレームワークが提供するライブラリ関数に置き換える**ことが最善。

```csharp
// ❌ NG：シェル経由でファイル一覧を取得
Process.Start("cmd.exe", $"/c dir {userInput}");

// ✅ OK：.NET のファイル操作 API を使う（シェルを経由しない）
var files = Directory.GetFiles(directoryPath);
```

```csharp
// ❌ NG：シェル経由でメール送信
Process.Start("sendmail", userEmail);

// ✅ OK：メール送信ライブラリを使う
await smtpClient.SendMailAsync(mailMessage);
```

---

## 5. 対策2：やむを得ず外部コマンドを呼び出す場合はホワイトリストで検証する

シェルを起動できる機能を使わざるを得ない場合、引数を構成するすべての変数に対してチェックを行い、**あらかじめ許可した処理のみを実行する**。

### ホワイトリスト方式（推奨）

```csharp
// 許可するコマンド引数をあらかじめ定義する
var allowedFormats = new HashSet<string> { "pdf", "csv", "xlsx" };

if (!allowedFormats.Contains(format))
{
    throw new ArgumentException("許可されていない形式です。");
}

// 検証済みの値だけを引数に使う
var process = new Process
{
    StartInfo = new ProcessStartInfo
    {
        FileName  = "converter",
        Arguments = format,          // ホワイトリスト済みの値のみ
        UseShellExecute = false,     // シェルを介さず直接起動
    }
};
process.Start();
```

### ❌ ブラックリスト方式は避ける

```csharp
// NG：危険文字を除外するだけでは迂回される可能性がある
var sanitized = userInput.Replace(";", "").Replace("&", "").Replace("|", "");
Process.Start("cmd.exe", $"/c {sanitized}");
```

---

## 6. 対策3：シェルを経由しないプロセス起動

シェル（`cmd.exe` / `/bin/sh`）を経由すると、メタ文字（`;` `|` `&` など）が解釈されてしまう。`UseShellExecute = false` で直接プロセスを起動する。

```csharp
// ❌ NG：UseShellExecute = true（デフォルト）→ シェル経由
Process.Start("cmd.exe", "/c " + userInput);

// ✅ OK：UseShellExecute = false → シェルを介さず直接起動
var process = new Process
{
    StartInfo = new ProcessStartInfo
    {
        FileName        = "ffmpeg",
        Arguments       = "-i input.mp4 output.pdf",
        UseShellExecute = false,   // シェルを介さない
        CreateNoWindow  = true,
    }
};
process.Start();
```

---

## 7. 対策4：実行権限を最小化する

万一攻撃が成立しても、被害を最小限に抑える多層防御。

- ウェブアプリケーションの実行ユーザーに **最小限の OS 権限** のみ付与する
- ファイルシステムへのアクセスは必要なディレクトリのみに限定する
- `sudo` / 管理者権限では実行しない

---

## 8. まとめ：OS コマンド・インジェクション対策チェックリスト

| 対策 | 内容 |
| --- | --- |
| シェル起動機能を使わない | ライブラリ API で代替できないか検討する |
| ホワイトリスト検証 | 引数はあらかじめ許可した値のみを使う |
| `UseShellExecute = false` | シェルを経由せず直接プロセスを起動する |
| 最小権限で実行 | アプリの実行ユーザーに必要最低限の権限のみ付与 |
| エラーを隠す | OS コマンドのエラー出力をブラウザに返さない |
