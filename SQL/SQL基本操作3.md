# SQL基本操作 続き2

---

## 1. CTE（Common Table Expression）：WITH句

複雑なサブクエリに名前を付けて読みやすくする。同じクエリ内で再利用もできる。

```sql
-- 部署ごとの平均給与を CTE で定義
WITH 部署別平均 AS (
    SELECT 部署, AVG(給与) AS avg_salary
    FROM 社員
    GROUP BY 部署
)
SELECT s.名前, s.給与, d.avg_salary
FROM 社員 s
JOIN 部署別平均 d ON s.部署 = d.部署
WHERE s.給与 > d.avg_salary;
```

### 複数の CTE を定義する

```sql
WITH
    高給与社員 AS (
        SELECT * FROM 社員 WHERE 給与 >= 500000
    ),
    部署集計 AS (
        SELECT 部署, COUNT(*) AS 人数
        FROM 高給与社員
        GROUP BY 部署
    )
SELECT * FROM 部署集計 ORDER BY 人数 DESC;
```

### サブクエリ vs CTE

| 比較 | サブクエリ | CTE |
| --- | --- | --- |
| 読みやすさ | ネストが深くなると読みにくい | 名前を付けるので読みやすい |
| 再利用 | 同じ式を2回書く必要あり | 同一クエリ内で再利用可 |
| デバッグ | しにくい | CTEを単体で確認しやすい |

---

## 2. UNION・INTERSECT・EXCEPT：集合演算

### UNION：結合（重複除外）/ UNION ALL：重複含む

```sql
-- 社員テーブルと退職者テーブルの名前を合わせて取得（重複除外）
SELECT 名前 FROM 社員
UNION
SELECT 名前 FROM 退職者;

-- 重複を除外せず全件取得（高速）
SELECT 名前 FROM 社員
UNION ALL
SELECT 名前 FROM 退職者;
```

### INTERSECT：共通部分

```sql
-- 両テーブルに存在する名前だけ取得
SELECT 名前 FROM 社員
INTERSECT
SELECT 名前 FROM プロジェクト参加者;
```

### EXCEPT：差分

```sql
-- 社員テーブルにあってプロジェクト参加者テーブルにない名前
SELECT 名前 FROM 社員
EXCEPT
SELECT 名前 FROM プロジェクト参加者;
```

> **注意**：列数・データ型を合わせる必要がある。

---

## 3. インデックス：検索を高速化する

テーブルの目次のようなもの。`WHERE` や `JOIN` でよく使う列に作成する。

```sql
-- インデックスの作成
CREATE INDEX idx_社員_部署 ON 社員(部署);

-- 複合インデックス（部署＋ステータスで検索することが多い場合）
CREATE INDEX idx_社員_部署_ステータス ON 社員(部署, ステータス);

-- ユニークインデックス（値の重複を許可しない）
CREATE UNIQUE INDEX idx_社員_メール ON 社員(メールアドレス);

-- インデックスの削除
DROP INDEX idx_社員_部署 ON 社員;
```

### インデックスの注意点

| メリット | デメリット |
| --- | --- |
| SELECT が速くなる | INSERT / UPDATE / DELETE が遅くなる |
| JOIN が速くなる | ディスク容量を使う |
| ORDER BY が速くなる | 更新頻度の高い列には不向き |

---

## 4. トランザクション制御

複数のSQL操作をひとまとめにし、すべて成功かすべて失敗かを保証する。

```sql
BEGIN TRANSACTION;

    -- 在庫を減らす
    UPDATE 在庫 SET 数量 = 数量 - 1 WHERE 商品コード = 'A001';

    -- 注文を登録する
    INSERT INTO 注文 (商品コード, 数量, 注文日)
    VALUES ('A001', 1, GETDATE());

COMMIT;   -- すべて成功 → 確定

-- または
ROLLBACK; -- どこかで失敗 → 全部取り消し
```

### ACID 特性

| 特性 | 説明 |
| --- | --- |
| **A**tomicity（原子性） | すべて成功かすべて失敗 |
| **C**onsistency（一貫性） | 処理前後でデータの整合性が保たれる |
| **I**solation（独立性） | 他のトランザクションの影響を受けない |
| **D**urability（永続性） | COMMIT後はシステム障害が起きても消えない |

### SAVE POINT（部分ロールバック）

```sql
BEGIN TRANSACTION;

    INSERT INTO ログ VALUES ('処理開始');
    SAVE TRANSACTION sp1;

    UPDATE 社員 SET 給与 = 給与 * 1.1;

    -- ここだけ取り消す（sp1 以降を戻す）
    ROLLBACK TRANSACTION sp1;

COMMIT;
```

---

## 5. ストアドプロシージャ：SQLをまとめて再利用

よく実行するSQL処理に名前をつけてDB側に保存する。

```sql
-- 作成
CREATE PROCEDURE usp_社員取得
    @部署名 NVARCHAR(50)
AS
BEGIN
    SELECT 名前, 給与
    FROM 社員
    WHERE 部署 = @部署名
    ORDER BY 給与 DESC;
END;

-- 実行
EXEC usp_社員取得 @部署名 = '開発部';

-- 削除
DROP PROCEDURE usp_社員取得;
```

### ストアドプロシージャ vs アプリ側クエリ

| 比較 | ストアドプロシージャ | アプリ側クエリ |
| --- | --- | --- |
| パフォーマンス | 実行計画がキャッシュされ速い | 毎回コンパイル |
| セキュリティ | SQLインジェクションを受けにくい | パラメータ化が必要 |
| 管理 | DB側で一元管理 | アプリコード内に散在 |
| テスト | DB単体でテストしやすい | アプリと一体でテスト |

---

## 6. MERGE：UPSERT（存在すれば更新、なければ挿入）

```sql
MERGE INTO 在庫 AS target
USING (SELECT '商品A' AS 商品コード, 10 AS 入荷数) AS source
ON target.商品コード = source.商品コード
WHEN MATCHED THEN
    UPDATE SET 数量 = target.数量 + source.入荷数
WHEN NOT MATCHED THEN
    INSERT (商品コード, 数量)
    VALUES (source.商品コード, source.入荷数);
```

---

## 7. 実行計画の確認

クエリがどのように実行されているか確認する。遅いクエリの改善に使う。

```sql
-- 実行計画を表示（SQL Server）
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

SELECT * FROM 社員 WHERE 部署 = '開発部';

-- グラフィカルな実行計画
-- SQL Server Management Studio で Ctrl+M を押してからクエリを実行
```

### よく見るパス

| 操作 | 説明 | コスト |
| --- | --- | --- |
| Index Seek | インデックスで目的の行に直接アクセス | 低 |
| Index Scan | インデックス全体をスキャン | 中 |
| Table Scan | テーブル全体をスキャン（フルスキャン） | 高 |
| Hash Match | ハッシュを使ったJOIN | 中〜高 |
| Nested Loops | 小さいテーブル同士のJOIN | 低 |
