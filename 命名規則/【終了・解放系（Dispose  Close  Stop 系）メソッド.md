# 【終了・解放系（Dispose / Close / Stop 系）メソッドまとめ】

## 用途

- リソース解放や停止処理

## 命名ルール

- `IDisposable` 実装時は `Dispose` が必須
- 副作用がある場合は名前で明示

## 命名例

| メソッド | 説明 |
| --- | --- |
| `DisposeResources()` | リソース解放 |
| `CloseConnection()` | 接続を閉じる |
| `ShutdownServer()` | サーバ停止 |
| `ReleaseHandles()` | ハンドル解放 |
| `StopProcess()` | プロセス停止 |

## 注意点

1. リソース管理を明確にする
2. 副作用の範囲を名前に反映
3. `IDisposable` は `Dispose` を必ず実装
4. 副作用がある停止処理も名前で明示
