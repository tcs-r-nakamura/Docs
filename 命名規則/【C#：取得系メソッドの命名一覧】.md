# 【C#：取得系メソッドの命名一覧】

## 1. 単純な値を取得する（軽い処理）

- `GetXxx`
- `Xxx`（プロパティ）

## 2. 見つからない可能性がある（検索系）

- `FindXxx`
- `TryGetXxx`

## 3. 外部リソースから取得する（通信・DB・ファイル）

- `FetchXxx`
- `LoadXxx`
- `RetrieveXxx`

## 4. 計算して取得する（算出系）

- `CalculateXxx`
- `ComputeXxx`

## 5. 新しく生成して返す

- `CreateXxx`
- `BuildXxx`
- `GenerateXxx`

---

## SQLメモ

```sql
UPDATE t_terminal_version
SET local_version = '1.0.1.9'

SELECT TOP (1000) [mac_address]
      ,[ip_address]
      ,[local_version]
      ,[last_update_date]
  FROM [denka_elegrip_db].[dbo].[t_terminal_version]
```
