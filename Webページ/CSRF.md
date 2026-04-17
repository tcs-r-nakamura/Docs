# クロスサイト・リクエスト・フォージェリ（CSRF）

---

## 1. CSRF とは

ログイン済みの利用者を罠サイトに誘導し、**利用者の意図しないリクエストをウェブアプリケーションに送信させる**攻撃。

```
攻撃の流れ：
1. 利用者がショッピングサイトにログイン（セッション維持中）
2. 攻撃者が用意した罠サイトを利用者が閲覧
3. 罠サイトに埋め込まれた img タグや form が自動送信される
   <img src="https://shop.example.com/order?item=A&qty=99">
4. 利用者のブラウザが Cookie（セッションID）を自動添付してリクエスト送信
5. ショッピングサイトは正規の利用者からのリクエストと判断して処理実行
```

---

## 2. 発生しうる脅威

攻撃が成功すると、ログイン済みの利用者が**意図しない操作を実行させられる**。

- **不正な決済・購入**：利用者が意図しない送金・商品購入
- **アカウント情報の変更**：パスワード・メールアドレスの書き換え
- **退会・削除処理**：利用者が意図しないアカウント削除
- **管理者画面の不正操作**：管理者権限での設定変更・ユーザー追加

---

## 3. 注意が必要なウェブサイトの特徴

| カテゴリ | 例 |
| --- | --- |
| 金銭処理が発生するサイト | ネットバンキング・ショッピング・オークション |
| 利用者情報を変更できるサイト | 会員情報変更・パスワード変更機能を持つサイト |
| 管理者機能を持つサイト | CMS・管理者画面 |

---

## 4. 対策1：トークンを使った正規リクエストの確認

サーバー側でランダムなトークンを発行し、フォーム送信時に照合する。罠サイトはトークンを知ることができないため、不正リクエストを弾ける。

```csharp
// ASP.NET Core：AntiForgery（標準機能）を使う

// Razor View（フォームに自動でトークンを埋め込む）
<form method="post" asp-action="Order">
    @Html.AntiForgeryToken()   // hidden フィールドでトークンを埋め込む
    <button type="submit">注文する</button>
</form>

// Controller（トークンを検証する）
[HttpPost]
[ValidateAntiForgeryToken]   // このアノテーションで自動検証
public async Task<IActionResult> Order(OrderModel model)
{
    // トークンが一致しない場合は 400 Bad Request が返る
    // 一致した場合のみここに到達する
    await _orderService.CreateAsync(model);
    return RedirectToAction("Complete");
}
```

---

## 5. 対策2：Referer ヘッダーの確認（補助的な対策）

リクエストの送信元（Referer）が自サイトであることを確認する。ただし Referer は送信されない場合もあるため、トークンと組み合わせて使う。

```csharp
// Middleware でRefererを確認する例
var referer = Request.Headers["Referer"].ToString();
if (!string.IsNullOrEmpty(referer) &&
    !referer.StartsWith("https://example.com/", StringComparison.OrdinalIgnoreCase))
{
    return Forbid();
}
```

---

## 6. 対策3：Cookie の SameSite 属性を設定する

`SameSite=Strict` または `SameSite=Lax` を設定すると、クロスサイトのリクエストで Cookie が送信されなくなる。

```csharp
// ASP.NET Core での設定
builder.Services.AddSession(options =>
{
    options.Cookie.SameSite = SameSiteMode.Strict;  // 別サイトから一切送信しない
});

// または認証 Cookie
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.SameSite = SameSiteMode.Strict;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
});
```

| SameSite の値 | 動作 |
| --- | --- |
| `Strict` | 別サイトからのリクエストでは Cookie を一切送信しない |
| `Lax` | 別サイトからの GET リクエストのみ送信（POST 等は送らない） |
| `None` | 制限なし（Secure 属性必須） |

---

## 7. 対策4：重要操作の前に再認証・確認を求める

パスワード変更・送金などの重要操作は、CSRF トークンに加えて**パスワードの再入力や確認ページ**を挟む。

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> ChangePassword(ChangePasswordModel model)
{
    // 現在のパスワードを再確認（CSRF 対策の二重チェック）
    var verified = await _authService.VerifyPasswordAsync(User, model.CurrentPassword);
    if (!verified)
        return Unauthorized();

    await _authService.ChangePasswordAsync(User, model.NewPassword);
    return RedirectToAction("Complete");
}
```

---

## 8. まとめ：CSRF 対策チェックリスト

| 対策 | 内容 |
| --- | --- |
| CSRF トークン | フォームにトークンを埋め込み、サーバー側で検証する |
| SameSite 属性 | Cookie に `Strict` または `Lax` を設定する |
| Referer 確認 | 送信元が自サイトであることを補助的に確認する |
| 重要操作の再認証 | パスワード変更・送金等は再認証・確認ページを挟む |
| GET で状態変更しない | データ変更は必ず POST / PUT / DELETE で行う |
