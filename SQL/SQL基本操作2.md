# SQL基本操作 続き

---

## 1. HAVING：集計結果を絞り込む

`WHERE` はグループ化前の絞り込み、`HAVING` はグループ化後の絞り込み。

```sql
-- 部署ごとの平均給与が350000以上の部署だけ表示
SELECT 部署, AVG(給与) AS avg_salary
FROM 社員
GROUP BY 部署
HAVING AVG(給与) >= 350000;
```

| 句 | タイミング | 使えるもの |
| --- | --- | --- |
| `WHERE` | GROUP BY の前 | 個々の行の条件 |
| `HAVING` | GROUP BY の後 | 集計関数（SUM・AVG・COUNT など） |

---

## 2. DISTINCT：重複を除外する

```sql
-- 部署名の重複を除いて一覧表示
SELECT DISTINCT 部署
FROM 社員;

-- 複数列の組み合わせで重複除外
SELECT DISTINCT 部署, ステータス
FROM 社員;
```

---

## 3. TOP：取得件数を制限する

```sql
-- 給与が高い上位3件を取得
SELECT TOP 3 名前, 給与
FROM 社員
ORDER BY 給与 DESC;

-- 全体の上位10%を取得
SELECT TOP 10 PERCENT 名前, 給与
FROM 社員
ORDER BY 給与 DESC;
```

---

## 4. サブクエリ

クエリの中に別のクエリを埋め込む。

### WHERE 句のサブクエリ

```sql
-- 平均給与より高い社員を取得
SELECT 名前, 給与
FROM 社員
WHERE 給与 > (SELECT AVG(給与) FROM 社員);
```

### FROM 句のサブクエリ（派生テーブル）

```sql
-- 部署ごとの平均給与を一時テーブルとして使う
SELECT 部署, avg_salary
FROM (
    SELECT 部署, AVG(給与) AS avg_salary
    FROM 社員
    GROUP BY 部署
) AS 部署別平均
WHERE avg_salary >= 350000;
```

### IN を使ったサブクエリ

```sql
-- 営業部または開発部に所属する社員を取得
SELECT 名前
FROM 社員
WHERE 部署 IN (
    SELECT 部署名
    FROM 部署マスタ
    WHERE 種別 = '技術系'
);
```

---

## 5. EXISTS：存在チェック

サブクエリに結果が1件でもあれば `TRUE`。`IN` より高速なことが多い。

```sql
-- 注文が1件でもある社員を取得
SELECT 名前
FROM 社員 e
WHERE EXISTS (
    SELECT 1
    FROM 注文 o
    WHERE o.社員ID = e.ID
);

-- 注文が一度もない社員を取得
SELECT 名前
FROM 社員 e
WHERE NOT EXISTS (
    SELECT 1
    FROM 注文 o
    WHERE o.社員ID = e.ID
);
```

---

## 6. NULL の操作

### NULL チェック

```sql
-- NULL の行を取得
SELECT * FROM 社員 WHERE 部署 IS NULL;

-- NULL でない行を取得
SELECT * FROM 社員 WHERE 部署 IS NOT NULL;
```

### ISNULL / COALESCE

```sql
-- 部署が NULL の場合「未所属」に置き換える
SELECT 名前, ISNULL(部署, '未所属') AS 部署
FROM 社員;

-- 複数列のうち最初に NULL でない値を返す
SELECT COALESCE(部署, 役職, '不明') AS 所属
FROM 社員;
```

---

## 7. 文字列関数

```sql
-- 文字数を返す
SELECT LEN('こんにちは');           -- 5

-- 大文字・小文字変換
SELECT UPPER('hello');             -- HELLO
SELECT LOWER('HELLO');             -- hello

-- 部分文字列を取得（開始位置, 文字数）
SELECT SUBSTRING('ABCDEF', 2, 3);  -- BCD

-- 文字列を置換
SELECT REPLACE('A001', 'A', 'B');  -- B001

-- 前後の空白を除去
SELECT LTRIM(RTRIM('  ABC  '));     -- ABC

-- 文字列を結合
SELECT CONCAT('田中', ' ', '太郎'); -- 田中 太郎

-- 特定文字列の位置を返す（見つからなければ0）
SELECT CHARINDEX('BC', 'ABCDEF');  -- 2
```

### LIKE：パターン検索

| パターン | 意味 |
| --- | --- |
| `'A%'` | A で始まる |
| `'%A'` | A で終わる |
| `'%A%'` | A を含む |
| `'A_'` | A の後に1文字 |

```sql
-- 名前が「田」で始まる社員を取得
SELECT * FROM 社員 WHERE 名前 LIKE '田%';
```

---

## 8. 日付関数

```sql
-- 現在日時
SELECT GETDATE();              -- 2026-04-17 10:30:00
SELECT SYSDATETIME();         -- より精度が高い

-- 日付のみ・時刻のみ
SELECT CAST(GETDATE() AS DATE);  -- 2026-04-17
SELECT CAST(GETDATE() AS TIME);  -- 10:30:00

-- 年・月・日を取り出す
SELECT YEAR(GETDATE());        -- 2026
SELECT MONTH(GETDATE());       -- 4
SELECT DAY(GETDATE());         -- 17

-- 日付を加算・減算
SELECT DATEADD(DAY, 7, GETDATE());    -- 7日後
SELECT DATEADD(MONTH, -1, GETDATE()); -- 1ヶ月前

-- 2つの日付の差
SELECT DATEDIFF(DAY, '2026-04-01', '2026-04-17');  -- 16
```

---

## 9. ウィンドウ関数

集計しながら元の行を維持できる。`GROUP BY` とは異なり、行が消えない。

### ROW_NUMBER：連番を付ける

```sql
-- 給与が高い順に連番を付ける
SELECT
    名前,
    給与,
    ROW_NUMBER() OVER (ORDER BY 給与 DESC) AS 順位
FROM 社員;
```

### RANK / DENSE_RANK：同順位を考慮した順位

```sql
SELECT
    名前,
    給与,
    RANK()       OVER (ORDER BY 給与 DESC) AS RANK,       -- 同率2位の次は4位
    DENSE_RANK() OVER (ORDER BY 給与 DESC) AS DENSE_RANK  -- 同率2位の次は3位
FROM 社員;
```

### PARTITION BY：グループ内で順位付け

```sql
-- 部署内で給与が高い順に順位を付ける
SELECT
    名前,
    部署,
    給与,
    RANK() OVER (PARTITION BY 部署 ORDER BY 給与 DESC) AS 部署内順位
FROM 社員;
```

### 集計ウィンドウ関数

```sql
-- 全体の合計・平均を各行に追加
SELECT
    名前,
    給与,
    SUM(給与)  OVER () AS 給与合計,
    AVG(給与)  OVER () AS 給与平均,
    MAX(給与)  OVER (PARTITION BY 部署) AS 部署内最高給与
FROM 社員;
```

---

## 10. VIEW：仮想テーブルを作る

よく使うクエリをビューとして保存し、テーブルのように使える。

```sql
-- ビューの作成
CREATE VIEW v_active_社員 AS
SELECT 名前, 部署, 給与
FROM 社員
WHERE ステータス = 'active';

-- ビューを使って検索（テーブルと同じように使える）
SELECT * FROM v_active_社員;

-- ビューの削除
DROP VIEW v_active_社員;
```

---

## 11. BETWEEN：範囲指定

```sql
-- 給与が300000以上400000以下の社員
SELECT 名前, 給与
FROM 社員
WHERE 給与 BETWEEN 300000 AND 400000;

-- 日付の範囲指定
SELECT *
FROM 注文
WHERE 注文日 BETWEEN '2026-04-01' AND '2026-04-30';
```

---

## 12. 集計関数まとめ

| 関数 | 説明 |
| --- | --- |
| `COUNT(*)` | 行数を数える（NULL含む） |
| `COUNT(列名)` | NULL を除いた行数を数える |
| `SUM(列名)` | 合計 |
| `AVG(列名)` | 平均 |
| `MAX(列名)` | 最大値 |
| `MIN(列名)` | 最小値 |

```sql
SELECT
    COUNT(*)       AS 総人数,
    COUNT(部署)    AS 部署登録済み人数,
    SUM(給与)      AS 給与合計,
    AVG(給与)      AS 平均給与,
    MAX(給与)      AS 最高給与,
    MIN(給与)      AS 最低給与
FROM 社員;
```

---

## SELECT の実行順序

SQLは書く順序と実行順序が異なる。

| 実行順序 | 句 |
| --- | --- |
| 1 | `FROM` |
| 2 | `JOIN` |
| 3 | `WHERE` |
| 4 | `GROUP BY` |
| 5 | `HAVING` |
| 6 | `SELECT` |
| 7 | `ORDER BY` |
| 8 | `TOP` |
