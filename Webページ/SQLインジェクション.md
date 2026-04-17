# SQL インジェクション

---

## 1. SQL インジェクションとは

ユーザーの入力値に悪意のある SQL 文を混入させ、データベースを不正に操作する攻撃。

```
// 攻撃例：ログインフォームに以下を入力される
ユーザーID: ' OR '1'='1
パスワード: anything

// 結果として生成されるSQL（全件マッチして認証突破される）
SELECT * FROM users WHERE id = '' OR '1'='1' AND password = 'anything'
```

---

## 2. 対策1：プレースホルダー（パラメータ化クエリ）を使う

SQL 文の組み立てに文字列連結を使わず、**プレースホルダー**でパラメータを渡す。

### ❌ 危険な書き方（文字列連結）

```csharp
// 入力値がそのまま SQL に埋め込まれる → 攻撃が成立する
string sql = "SELECT * FROM users WHERE id = '" + userId + "'";
```

### ✅ 安全な書き方（プレースホルダー）

```csharp
// SQL 文の構造は固定。値は後からバインドされるため攻撃が成立しない
string sql = "SELECT * FROM users WHERE id = @userId";

using var command = new SqlCommand(sql, connection);
command.Parameters.AddWithValue("@userId", userId);  // 値を安全にバインド
```

### Entity Framework（ORM）を使う場合

```csharp
// LINQ はプレースホルダーに変換されるため、原則として安全
var user = await _db.Users
    .Where(u => u.Id == userId)
    .FirstOrDefaultAsync();

// ❌ 生SQL を使う場合は FromSqlRaw ではなく FromSqlInterpolated を使う
// 危険
var users = _db.Users.FromSqlRaw($"SELECT * FROM users WHERE id = '{userId}'");

// 安全
var users = _db.Users.FromSqlInterpolated($"SELECT * FROM users WHERE id = {userId}");
```

---

## 3. 対策2：文字列連結が避けられない場合はエスケープ処理

プレースホルダーが使えない場合（動的な列名・テーブル名など）は、データベースエンジンのエスケープ API を使う。

```csharp
// SQL Server：角括弧でテーブル名・列名を囲む（動的な識別子のみ）
string safeSql = $"SELECT * FROM [{tableName.Replace("]", "]]")}]";
```

> **原則：** 値のバインドはプレースホルダーで行う。エスケープはあくまで補助手段。

---

## 4. 対策3：ウェブアプリのパラメータに SQL 文を直接指定しない

URL やフォームのパラメータに SQL 文をそのまま受け取る設計は避ける。

```
# ❌ 危険な設計例
GET /api/search?query=SELECT * FROM users

# ✅ 安全な設計例：パラメータは値のみ受け取り、SQL はサーバー側で組み立てる
GET /api/search?keyword=田中
```

---

## 5. 対策4：エラーメッセージをそのままブラウザに表示しない

SQL エラーメッセージにはテーブル名・列名・DB 構造が含まれることがあり、攻撃者に情報を与えてしまう。

```csharp
// ❌ NG：例外メッセージをそのまま返す
catch (SqlException ex)
{
    return BadRequest(ex.Message);
    // 例：「列名 'pasword' が無効です。」→ 列名が漏れる
}

// ✅ OK：ログには詳細を記録し、ユーザーには汎用メッセージを返す
catch (SqlException ex)
{
    _logger.LogError(ex, "DB エラーが発生しました");
    return StatusCode(500, "サーバーエラーが発生しました。");
}
```

---

## 6. 対策5：データベースアカウントに適切な権限を与える

万一攻撃が成立しても、被害を最小限に抑えるための多層防御。

| 操作 | 最小権限の例 |
| --- | --- |
| 参照のみのアプリ | `SELECT` 権限のみ付与 |
| 通常の業務アプリ | `SELECT` / `INSERT` / `UPDATE` のみ。`DELETE` / `DROP` は付与しない |
| バッチ処理など | 必要なテーブルのみにアクセスを限定 |

```sql
-- 例：アプリ用ユーザーに最小権限を付与（SQL Server）
CREATE LOGIN app_user WITH PASSWORD = '...';
CREATE USER  app_user FOR LOGIN app_user;

GRANT SELECT, INSERT, UPDATE ON dbo.orders TO app_user;
-- DROP・TRUNCATE・システムテーブルへのアクセスは付与しない
```

---

## 7. まとめ：SQL インジェクション対策チェックリスト

| 対策 | 内容 |
| --- | --- |
| プレースホルダーを使う | SQL に値を文字列結合しない |
| エスケープ処理を行う | 連結が必要な場合はDBエンジンのAPIを使う |
| パラメータに SQL を渡さない | URLやフォームから SQL 文を直接受け取らない |
| エラーを隠す | DB エラーをそのままブラウザに返さない |
| 最小権限を付与する | アプリに必要最低限の DB 権限だけ与える |
