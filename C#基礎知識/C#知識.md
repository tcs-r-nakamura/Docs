# C# 知識

## namespace（名前空間）

クラスを整理するためのフォルダのようなもの。

- `OrderAllocationApp.Model` = 「OrderAllocationApp プロジェクトの Model グループ」という意味
- 同じ名前のクラスが別プロジェクトにあっても、namespace が違えば区別できる

---

## ドキュメントコメント

```csharp
/// <summary>
/// 注文引当の結果を表す読み取り専用モデルクラス
/// </summary>
```

コードの動作には影響しない。IDEでの補完や自動ドキュメント生成に使われる。

---

## クラス宣言の読み方

```csharp
public class AllocationResult
```

| キーワード | 意味 |
| --- | --- |
| `public` | プロジェクト内のどこからでも使える |
| `class` | データと処理をまとめた設計図を作る |
| `AllocationResult` | クラスの名前（引当結果という意味） |
