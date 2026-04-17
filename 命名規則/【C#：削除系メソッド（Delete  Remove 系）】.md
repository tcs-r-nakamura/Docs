# 【C#：削除系メソッド（Delete / Remove 系）】

## 用途

- オブジェクト、データ、ファイルなどを削除する処理。
- 削除対象や範囲が明確であることが重要。
- 削除による副作用がある場合は命名で示す。

## 命名例

| メソッド | 説明 |
| --- | --- |
| `DeleteXxx()` | ファイルやデータベースなど明確な対象を削除。例：`DeleteFile()`, `DeleteUser()`, `DeleteRecordById()` |
| `RemoveXxx()` | コレクションやリストから要素を削除。例：`RemovePlayer()`, `RemoveItemFromInventory()` |
| `ClearXxx()` | 全体を一括で削除（コレクションやキャッシュ）。例：`ClearCache()`, `ClearAllPlayers()` |
| `DestroyXxx()` | オブジェクトやリソースを破棄（Unityなど）。例：`DestroyObject()`, `DestroyEnemy()` |

## 注意点

1. 削除対象が分かるように命名する。
2. `Clear` は「全削除」を意味するため、部分削除の場合は `Remove` を使用。
3. `Destroy` はオブジェクト破棄など、特殊用途で使う（Unity やゲーム開発）。
4. 副作用が大きい場合はメソッド名で警告的意味を持たせる。
5. 戻り値がある場合（削除成功/失敗）には `bool` を返すことが多い。例：`bool RemoveItem(string id)`

## 補足

- 削除系メソッドは副作用が大きいため、命名で処理内容を明確にすることが特に重要。
- コレクション操作とリソース操作で命名規則を使い分ける。
