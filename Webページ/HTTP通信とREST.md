# HTTP通信と REST API

---

## 1. HTTP とは

Web ブラウザとサーバーがデータをやり取りするためのプロトコル（通信規約）。

```
クライアント（ブラウザ・アプリ）
    ↓ リクエスト（Request）
サーバー
    ↓ レスポンス（Response）
クライアント
```

---

## 2. HTTP メソッド

| メソッド | 用途 | 副作用 |
| --- | --- | --- |
| `GET` | データの取得 | なし（べき等） |
| `POST` | データの新規作成 | あり |
| `PUT` | データの全体更新（置き換え） | あり（べき等） |
| `PATCH` | データの部分更新 | あり |
| `DELETE` | データの削除 | あり（べき等） |

> **べき等（idempotent）：** 同じリクエストを何度送っても結果が変わらない。

---

## 3. HTTP ステータスコード

### 2xx：成功

| コード | 意味 |
| --- | --- |
| `200 OK` | 成功 |
| `201 Created` | 作成成功（POST の後に返す） |
| `204 No Content` | 成功・返すデータなし（DELETE の後など） |

### 3xx：リダイレクト

| コード | 意味 |
| --- | --- |
| `301 Moved Permanently` | 永続的なリダイレクト |
| `302 Found` | 一時的なリダイレクト |

### 4xx：クライアントエラー

| コード | 意味 |
| --- | --- |
| `400 Bad Request` | リクエストの形式が不正 |
| `401 Unauthorized` | 認証が必要（ログインしていない） |
| `403 Forbidden` | 認証済みだがアクセス権限なし |
| `404 Not Found` | リソースが存在しない |
| `409 Conflict` | 競合が発生（重複登録など） |
| `422 Unprocessable Entity` | バリデーションエラー |

### 5xx：サーバーエラー

| コード | 意味 |
| --- | --- |
| `500 Internal Server Error` | サーバー内部エラー |
| `502 Bad Gateway` | 上流サーバーからの不正なレスポンス |
| `503 Service Unavailable` | サーバーが一時的に利用不可 |

---

## 4. HTTP ヘッダー

### リクエストヘッダー（よく使うもの）

```
GET /api/orders HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGci...      # 認証トークン
Content-Type: application/json         # 送信データの形式
Accept: application/json               # 受け取りたいデータの形式
```

### レスポンスヘッダー（よく使うもの）

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-cache
Access-Control-Allow-Origin: *         # CORS 設定
```

---

## 5. REST API の設計原則

REST（Representational State Transfer）は API 設計のスタイル。

### URL 設計

```
# リソース（名詞）をURLで表す
GET    /api/orders          # 注文一覧の取得
GET    /api/orders/123      # ID=123 の注文を取得
POST   /api/orders          # 注文の新規作成
PUT    /api/orders/123      # ID=123 の注文を更新
DELETE /api/orders/123      # ID=123 の注文を削除

# 階層関係
GET    /api/orders/123/items     # 注文123の明細一覧
```

### URL 設計のルール

```
# ✅ 良い例
GET /api/orders
GET /api/users/42/orders

# ❌ 悪い例（動詞を使わない）
GET /api/getOrders
POST /api/deleteOrder
```

---

## 6. JSON：API でよく使うデータ形式

```json
{
    "orderId": 123,
    "customerName": "田中太郎",
    "status": "processing",
    "items": [
        { "productCode": "A001", "quantity": 2, "price": 1500 },
        { "productCode": "B002", "quantity": 1, "price": 3000 }
    ],
    "totalPrice": 6000,
    "createdAt": "2026-04-17T10:30:00Z"
}
```

### 命名規則

| 言語 | 慣習 |
| --- | --- |
| JSON | `camelCase`（例：`orderId`） |
| C# | `PascalCase`（例：`OrderId`） |
| SQL | `snake_case`（例：`order_id`） |

---

## 7. C# での HTTP 通信（HttpClient）

```csharp
// HttpClient はシングルトンで使う（using で都度生成しない）
public class OrderApiClient
{
    private readonly HttpClient _httpClient;

    public OrderApiClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    // GET：データ取得
    public async Task<List<Order>?> GetOrdersAsync()
    {
        var response = await _httpClient.GetAsync("/api/orders");
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadFromJsonAsync<List<Order>>();
    }

    // POST：データ送信
    public async Task<Order?> CreateOrderAsync(Order order)
    {
        var response = await _httpClient.PostAsJsonAsync("/api/orders", order);
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadFromJsonAsync<Order>();
    }

    // DELETE：削除
    public async Task DeleteOrderAsync(int id)
    {
        var response = await _httpClient.DeleteAsync($"/api/orders/{id}");
        response.EnsureSuccessStatusCode();
    }
}
```

---

## 8. クエリパラメータとパスパラメータ

```
# パスパラメータ：特定リソースを識別する
GET /api/orders/123

# クエリパラメータ：絞り込み・ページングなど
GET /api/orders?status=processing&page=1&limit=20
```

```csharp
// クエリパラメータを付けてリクエスト
// ❌ NG：特殊文字（スペース・& など）が含まれると URL が壊れる
var url = $"/api/orders?status={status}&page={page}&limit={limit}";

// ✅ OK：QueryHelpers で安全にエンコードする（Microsoft.AspNetCore.WebUtilities）
var queryParams = new Dictionary<string, string?>
{
    ["status"] = status,
    ["page"]   = page.ToString(),
    ["limit"]  = limit.ToString(),
};
var url = QueryHelpers.AddQueryString("/api/orders", queryParams);
var response = await _httpClient.GetAsync(url);
```

---

## 9. CORS（Cross-Origin Resource Sharing）

異なるオリジン（ドメイン・ポート）間の通信を制御する仕組み。

```
フロントエンド：https://app.example.com
バックエンド  ：https://api.example.com   ← 別オリジン → CORS 設定が必要
```

ASP.NET Core での設定例：

```csharp
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("https://app.example.com")
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

app.UseCors("AllowFrontend");
```

---

## 10. まとめ：REST API 設計チェックリスト

- [ ] URL にはリソース名（名詞）を使い、動詞は使わない
- [ ] HTTP メソッドで操作の種類を表す（GET / POST / PUT / DELETE）
- [ ] 適切なステータスコードを返す（200 / 201 / 400 / 404 / 500 など）
- [ ] エラーレスポンスに詳細情報を含める（スタックトレースは含めない）
- [ ] HTTPS を使う
- [ ] 認証が必要なエンドポイントには Authorization ヘッダーを要求する
- [ ] ページング・絞り込みはクエリパラメータで表現する
