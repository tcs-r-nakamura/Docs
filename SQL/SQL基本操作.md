# SQL初心者向け 基本操作サンプル

## テーブル作成

```sql
CREATE TABLE テーブル名 (
    カラム名1 INT PRIMARY KEY,       -- ID用
    カラム名2 VARCHAR(50),          -- 名前用
    カラム名3 VARCHAR(50),          -- 部署用
    カラム名4 INT,                  -- 給与用
    カラム名5 VARCHAR(10)           -- ステータス用
);
```

---

## 1. INSERT: データを追加する（登録）

```sql
-- 1人ずつ追加
INSERT INTO テーブル名 (カラム名1, カラム名2, カラム名3, カラム名4, カラム名5)
VALUES (1, '名前', '部署', 350000, 'active');

-- 複数人まとめて追加
INSERT INTO テーブル名 (カラム名1, カラム名2, カラム名3, カラム名4, カラム名5)
VALUES
(2, '名前', '部署', 400000, 'active'),
(3, '名前', '部署', 300000, 'active');
```

---

## 2. SELECT: データを確認する（検索）

```sql
-- 全員の情報を表示
SELECT * FROM テーブル名;

-- 条件付きで表示（給与が一定以上）
SELECT カラム名1, カラム名2, カラム名4
FROM テーブル名
WHERE カラム名4 >= 300000;

-- 部署ごとの平均給与を表示
SELECT カラム名3, AVG(カラム名4) AS avg_salary
FROM テーブル名
GROUP BY カラム名3;

-- 給与が高い順に並べて表示
SELECT カラム名2, カラム名4
FROM テーブル名
ORDER BY カラム名4 DESC;
```

---

## 3. UPDATE: データを変更する（更新）

```sql
-- 特定部署の給与を10%アップ
UPDATE テーブル名
SET カラム名4 = カラム名4 * 1.1
WHERE カラム名3 = '部署';

-- 特定IDの人の部署を変更
UPDATE テーブル名
SET カラム名3 = '新しい部署'
WHERE カラム名1 = 1;

-- 全員のステータスを変更
UPDATE テーブル名
SET カラム名5 = 'inactive';
```

---

## 4. DELETE: データを削除する（消去）

```sql
-- 特定IDの人を削除
DELETE FROM テーブル名
WHERE カラム名1 = 3;

-- 条件付き削除（給与が少ない人）
DELETE FROM テーブル名
WHERE カラム名4 < 300000;

-- 全員削除（注意！）
DELETE FROM テーブル名;
```

---

## エイリアス

- 列のエイリアス → `SELECT ProductName AS 名前` ← `AS` を使うのが読みやすい
- テーブルのエイリアス → `FROM Orders o` ← `AS` を省略することが多い

---

## メモ

- `WHERE` の後に関数は基本的に使わない
- `INDEX` は `WHERE` と1対1の関係

---

## CASE 式

```sql
CASE 式
  WHEN 値1 THEN 結果1
  WHEN 値2 THEN 結果2
  ...
  ELSE デフォルト結果
END
```

---

## 結合

```sql
SELECT 列名一覧
FROM テーブルA AS A
JOIN 種類 テーブルB AS B
ON A.共通列 = B.共通列
```

### LEFT JOIN

```sql
SELECT 列名一覧
FROM 左テーブル AS 別名A
LEFT JOIN 右テーブル AS 別名B
  ON A.キー = B.キー
```

### CROSS JOIN

```sql
SELECT *
FROM TableA
CROSS JOIN TableB;
```

---

## GROUP BY + MAX の例

| 処理 | 内容 |
| -------------------------------------------------- | --------------------- |
| `GROUP BY ReferenceNo` | `ReferenceNo` ごとにまとめる |
| `MAX(ApplyingDate)` | 過去の中で**一番新しい日付**を探す |
| `WHERE ApplyingDate < CAST(SYSDATETIME() AS DATE)` | 今日より**前の日付**に限定 |

```sql
INNER JOIN (
    SELECT
        t1.ReferenceNo,
        t1.JANCode
    FROM T_JanCodeAssign t1
    INNER JOIN ( ... ) t2
      ON t1.ReferenceNo = t2.ReferenceNo
     AND t1.ApplyingDate = t2.ApplyingDate
) jc
```

| 名前 | エイリアスの対象 | 役割 |
| ---- | ---------------------- | ------------------------------ |
| `t1` | `T_JanCodeAssign` テーブル | テーブルの列を短く書くための別名 |
| `jc` | サブクエリ全体の結果 | このサブクエリを「仮のテーブル」として他と結合するための名前 |
