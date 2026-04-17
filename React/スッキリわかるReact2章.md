# 第2章 JavaScript基礎

---

## JavaScript（ジャバスクリプト）

Webページを動かすためのプログラミング言語。

---

## コールバック（Callback）

あとで呼び出してほしい処理（関数）を先に渡しておく仕組み。

---

## 第一級関数（First-Class Function）

関数を数字や文字列と同じように使えること。

---

## クロージャ（Closure）

関数が、外側の変数を記憶して持ち続けること。

> **注意点：** メモリを持ち続けるため、不要なクロージャは解放しないと重くなる。

---

## ECMAScript（エクマスクリプト）

JavaScript の「仕様（ルールブック）」。

### var / let / const の違い

| キーワード | 再代入 | 再宣言 |
| --- | --- | --- |
| `var` | ○ | ○ |
| `let` | ○ | ✕ |
| `const` | ✕ | ✕ |

> **基本は `const`。どうしても再代入するときは `let`。**

---

## 巻き上げ（ホイスティング / Hoisting）

変数や関数の宣言が、実行前に先頭へ移動したように扱われる仕組み。

- 宣言：巻き上げられる ✅
- 代入：巻き上げられない ❌

---

## 静的型付け言語

プログラムを書く時点（実行前）で、変数や値の型が決まっている言語（TypeScript）。

## 動的型付け言語

変数の型を事前に決めず、実行時に型が自動で決まる言語（JavaScript）。

---

## プリミティブ型（Primitive Type）

数値や文字などの「値そのもの」を直接持つ型（イミュータブル）。

| 型 | 例 |
| --- | --- |
| `number` | `10`, `3.14` |
| `string` | `"Hello"` |
| `boolean` | `true`, `false` |
| `undefined` | `undefined` |
| `null` | `null` |
| `symbol` | `Symbol("id")` |
| `bigint` | `123n` |

---

## オブジェクト型（Object型）

複数の値や処理をひとまとめにした「データのかたまり」（ミュータブル）。

- オブジェクト `{}`
- 配列 `[]`
- 関数 `function () {}`
- 日付 `Date`
- 正規表現 `RegExp`

> ※ プリミティブ型以外はほぼオブジェクト

---

## プリミティブ値のリテラルとラッパーオブジェクト

### プリミティブ値のリテラル

値そのものを直接書いたもの。

```javascript
let str  = "Hello";  // 文字列リテラル
let num  = 100;      // 数値リテラル
let flag = true;     // 真偽値リテラル
```

### ラッパーオブジェクト（Wrapper Object）

プリミティブ型をオブジェクトとして扱うための箱。

```javascript
let strObj  = new String("Hello");  // ラッパーオブジェクト
let numObj  = new Number(100);
let boolObj = new Boolean(true);

console.log(typeof strObj); // "object"
console.log(typeof numObj); // "object"
```

### リテラルとラッパーオブジェクトの関係

- 一見プリミティブでもメソッドが使える
- 実際には内部で一時的にラッパーオブジェクトが生成される
- メソッド呼び出し後は自動で破棄される

```javascript
let s = "Hello";
console.log(s.toUpperCase()); // "HELLO"
```

> 普段はリテラルを使う。`new String()` のように自分で作ることはほとんどない。

---

## オブジェクトリテラル（Object Literal）

`{}` の中にプロパティやメソッドを書いてオブジェクトを作る方法。

```javascript
const user = {
  name: "Alice",   // key: value
  age: 20,
  greet() {
    console.log("こんにちは");
  }
};

console.log(user.name); // Alice
user.greet();           // こんにちは
```

---

## 配列リテラル（Array Literal）

`[]` の中に値をカンマで並べて配列を作る方法。

```javascript
const numbers = [1, 2, 3, 4, 5];
console.log(numbers[0]);      // 1
console.log(numbers.length);  // 5

const mixed = [10, "Hello", true, { name: "Alice" }, [1, 2, 3]];
console.log(mixed[3].name);   // Alice（オブジェクトのプロパティ）
console.log(mixed[4][1]);     // 2（配列の要素）
```

---

## 正規表現リテラル（RegExp Literal）

文字列のパターンを簡単に書いて検索や置換に使える仕組み。

```javascript
// test() でマッチ確認
const regex = /hello/;
console.log(regex.test("hello world")); // true
console.log(regex.test("hi there"));    // false

// match() で文字列検索
const str = "I love JavaScript";
console.log(str.match(/JavaScript/));   // ["JavaScript"]
```

### フラグ（オプション）

| フラグ | 意味 |
| --- | --- |
| `i` | 大文字小文字を区別しない |
| `g` | 全体にマッチ（global） |
| `m` | 複数行モード |

```javascript
const regex = /hello/gi;  // 大文字小文字無視・文字列全体を検索
```

---

## 広義のオブジェクト / 狭義のオブジェクト

| 用語 | 説明 |
| --- | --- |
| **広義のオブジェクト** | JavaScriptでオブジェクト型として扱えるすべての値（プリミティブ型以外） |
| **狭義のオブジェクト** | `{}` で作るプロパティの集合（配列や関数は含まない） |

---

## 関数宣言（Function Declaration）

`function` キーワードを使って名前付きの関数を定義する方法。

**特徴：**
- 巻き上げ（Hoisting）される
- 名前が必須
- デバッグで便利（エラー時に関数名が表示される）

```javascript
function greet(name) {
  console.log("こんにちは、" + name + "!");
}

greet("Alice"); // こんにちは、Alice!
```

---

## 関数式（Function Expression）

関数を値として変数に代入する書き方。

**特徴：**
- 巻き上げ（Hoisting）されない
- 関数を値として扱える

```javascript
// 匿名関数
const greet = function(name) {
  console.log("こんにちは、" + name + "!");
};

greet("Alice"); // こんにちは、Alice!

// 名前付き関数式（名前はデバッグ用）
const greet2 = function sayHello(name) {
  console.log("こんにちは、" + name + "!");
};
```

---

## 第一級オブジェクト（First-Class Object）

関数やオブジェクトを「値」として自由に扱えること。変数に代入したり、引数に渡したり、戻り値として返したりできる。

```javascript
// 変数に代入
const greet = function(name) {
  console.log("Hello " + name);
};

// 関数の引数として渡す
function callTwice(fn) {
  fn("Bob");
  fn("Charlie");
}
callTwice(greet);  // Hello Bob / Hello Charlie

// 関数の戻り値として返す
function makeGreeter() {
  return function(name) {
    console.log("Hi " + name);
  };
}
const greeter = makeGreeter();
greeter("David"); // Hi David
```

---

## アロー関数式（Arrow Function）

関数を短く簡単に書ける新しい書き方。

**特徴：**
- `function` キーワードを使わずに書ける
- 引数が1つなら括弧 `()` を省略できる
- 処理が1行なら `{}` と `return` も省略できる
- `this` の扱いが通常の関数と異なる（レキシカルスコープ）

```javascript
// 通常の関数
const greet = function(name) {
  return "Hello " + name;
};

// アロー関数（省略形）
const greet = name => "Hello " + name;

// 引数がない場合
const sayHi = () => "Hi!";
console.log(sayHi()); // Hi!

// 配列操作での例
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6]
```

> - メインのアプリケーションロジックや再利用前提の関数は `function` による宣言
> - 短い処理やインラインで使う関数はアロー関数

---

## 様々な引数の表現

### デフォルト引数（Default Parameter）

関数に渡されなかった引数に対して、あらかじめ設定した初期値を使う仕組み。

```javascript
function greet(name = "Guest") {
  console.log("Hello " + name);
}

greet("Alice"); // Hello Alice
greet();        // Hello Guest（デフォルト値が使われる）

// 複数引数でも使える
function add(a = 0, b = 0) {
  return a + b;
}
console.log(add(5, 3)); // 8
console.log(add(5));    // 5
console.log(add());     // 0
```

### レストパラメータ（Rest Parameter）

関数の引数を1つの配列としてまとめて受け取る仕組み。

```javascript
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3));       // 6
console.log(sum(4, 5, 6, 7, 8)); // 30
```

> **注意：** レストパラメータ（`...`）は関数の**最後の引数**にしか置けない。

```javascript
function test(a, b, ...rest) {  // ○ 最後
  console.log(a);    // 1
  console.log(b);    // 2
  console.log(rest); // [3, 4, 5]
}
test(1, 2, 3, 4, 5);

// function test(a, ...rest, b) { }  // ❌ エラー
```

---

## JavaScript のクラス構文（Class）

オブジェクトを作るための設計図（テンプレート）を簡単に書ける文法。

```javascript
class Person {
  constructor(name, age) {  // 初期化用の特別なメソッド
    this.name = name;
    this.age  = age;
  }

  greet() {
    console.log(`Hello, my name is ${this.name}`);
  }
}

const alice = new Person("Alice", 20);
alice.greet(); // Hello, my name is Alice
```

---

## プライベートメンバー（Private Member）

クラスの外から直接アクセスできないプロパティやメソッド。`#` を付けるとクラス内部専用になる。

```javascript
class Person {
  #name; // プライベートプロパティ

  constructor(name) {
    this.#name = name;
  }

  greet() {
    console.log(`Hello, my name is ${this.#name}`);
  }
}

const alice = new Person("Alice");
alice.greet();         // Hello, my name is Alice
// alice.#name;        // ❌ エラー：クラス外からアクセスできない
```

---

## 静的メンバー（Static Member）

クラス自体に属し、インスタンスを作らなくても使えるプロパティやメソッド。

```javascript
class Circle {
  static pi = 3.14; // 静的プロパティ

  constructor(radius) {
    this.radius = radius;
  }

  static calcArea(radius) {      // 静的メソッド
    return Circle.pi * radius * radius;
  }

  getArea() {                    // インスタンスメソッド
    return Circle.pi * this.radius * this.radius;
  }
}

console.log(Circle.pi);          // 3.14（クラス名からアクセス）
console.log(Circle.calcArea(5)); // 78.5

const c = new Circle(5);
console.log(c.getArea());        // 78.5
// c.calcArea(5);                // ❌ エラー：インスタンスからは呼べない
```

### メンバーの種類まとめ

| 種類 | アクセス方法 | 特徴 |
| --- | --- | --- |
| プライベートメンバー | クラス内部のみ | 外からアクセス不可、カプセル化 |
| 静的メンバー | クラス名から | インスタンス不要、クラス共通 |
| インスタンスメンバー | インスタンスから | 個別のオブジェクトごとのデータ |

---

## プロトタイプベース（Prototype-based）

オブジェクトが他のオブジェクトを「プロトタイプ」として継承する方式。

```javascript
const animal = {
  speak() {
    console.log(`${this.name} is making a sound`);
  }
};

const dog = Object.create(animal);
dog.name = "Pochi";
dog.speak(); // Pochi is making a sound
```

> ES6 のクラス構文も内部的にはプロトタイプベース。

---

## プロトタイプチェーン（Prototype Chain）

オブジェクトにプロパティやメソッドが存在しない場合、プロトタイプを順番にたどって探す仕組み。

```
dog → animal → Object.prototype → null
```

```javascript
console.log(Object.getPrototypeOf(dog));            // animal
console.log(Object.getPrototypeOf(animal));         // Object.prototype
console.log(Object.getPrototypeOf(Object.prototype)); // null
```

---

## 分割代入（Destructuring Assignment）

配列やオブジェクトから値をまとめて取り出して変数に代入する書き方。

```javascript
// 配列の分割代入
const arr = [10, 20, 30];
const [x, y, z] = arr;
console.log(x, y, z); // 10 20 30

// 要素をスキップ
const [first, , third] = [10, 20, 30, 40];
console.log(first, third); // 10 30

// オブジェクトの分割代入
const user = { name: "Alice", age: 25 };
const { name, age } = user;
console.log(name, age); // Alice 25

// 別名で受け取る
const { name: userName, age: userAge } = user;
console.log(userName, userAge); // Alice 25

// 関数の引数で使う
function greet({ name, age }) {
  console.log(`Hello ${name}, ${age} years old`);
}
greet({ name: "Alice", age: 25 }); // Hello Alice, 25 years old
```

---

## スプレッド構文（Spread syntax）

配列やオブジェクトの要素・プロパティを展開して取り出す構文（`...`）。

```javascript
// 配列のコピー
const arr1 = [1, 2, 3];
const arr2 = [...arr1];

// 配列の結合
const arr3 = [4, 5];
const combined = [...arr1, ...arr3]; // [1, 2, 3, 4, 5]

// オブジェクトのコピー
const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1 };

// オブジェクトのマージ（同じキーは後ろで上書き）
const obj3   = { b: 3, c: 4 };
const merged = { ...obj1, ...obj3 }; // { a: 1, b: 3, c: 4 }
```

---

## シャローコピー（Shallow Copy）

1階層目だけを複製し、中のオブジェクトは同じ参照を共有するコピー。

```javascript
const original = {
  name: "Alice",
  address: { city: "Tokyo" }
};

const shallow = { ...original }; // シャローコピー

shallow.address.city = "Osaka";
console.log(original.address.city); // "Osaka"（元も変わる）
```

| コピー方法 | コード |
| --- | --- |
| 配列（スプレッド） | `const copy = [...arr]` |
| 配列（slice） | `const copy = arr.slice()` |
| オブジェクト（スプレッド） | `const copy = { ...obj }` |
| オブジェクト（assign） | `const copy = Object.assign({}, obj)` |

---

## ディープコピー（Deep Copy）

中身の中身まですべて新しくコピーし、元データと参照を共有しない方法。

```javascript
const original = {
  name: "Alice",
  address: { city: "Tokyo" }
};

const deep = structuredClone(original); // ① 推奨（最新）

deep.address.city = "Osaka";
console.log(original.address.city); // "Tokyo"（元は変わらない）
```

### シャローコピーとディープコピーの比較

| 項目 | シャローコピー | ディープコピー |
| --- | --- | --- |
| コピー範囲 | 1階層 | 全階層 |
| 参照共有 | あり | なし |
| 安全性 | 低 | 高 |
| パフォーマンス | 高速 | 重い |

---

## ショートサーキット評価（Short-circuit Evaluation）

論理演算子の途中で結果が確定した時点で、残りの式を評価しない仕組み。

```javascript
// &&（AND）：左が false なら右を実行しない
false && doSomething(); // doSomething() は実行されない
true  && doSomething(); // doSomething() が実行される

// ||（OR）：左が true なら右を実行しない
true  || doSomething(); // doSomething() は実行されない
false || doSomething(); // doSomething() が実行される

// ??（Null合体演算子）：null/undefined のときだけ右を使う
const count = value ?? 0;  // 0 や "" はそのまま使われる
```

---

## JavaScript における this

| 呼び出し方 | `this` が指すもの |
| --- | --- |
| グローバルスコープ | グローバルオブジェクト（`window` / `global`） |
| オブジェクトのメソッド | そのオブジェクト |
| 普通の関数（非 strict） | グローバルオブジェクト |
| 普通の関数（strict） | `undefined` |
| コンストラクタ関数（`new`） | 新しいインスタンス |
| アロー関数 | 外側のスコープの `this` |

```javascript
// オブジェクトのメソッド
const obj = {
  name: "Alice",
  greet() {
    console.log(this.name); // "Alice"
  }
};

// アロー関数：this を持たず外側を参照
const obj2 = {
  name: "Alice",
  arrow: () => console.log(this.name) // undefined
};
```

> `this` が使われていたら**アンチパターン**になることが多い。

---

## strict モード（厳格モード）

JavaScript のより安全で厳密な書き方を強制するモード。

```javascript
'use strict';

x = 10; // ❌ エラー（宣言なしの変数代入は禁止）

function hello() {
  console.log(this); // undefined（グローバルオブジェクトではない）
}

function sum(a, a) { }  // ❌ エラー（重複引数禁止）
```

**制限される動作：**
- 宣言なしの変数の使用禁止
- `this` の安全化
- 重複するプロパティや引数の禁止
- 読み取り専用プロパティの変更禁止

---

## モジュール（Module）

プログラムの機能をまとめて、他のファイルから再利用できる単位。

- 独立したファイルや単位として作ることができる
- 必要な機能だけを `import` / `export` で読み込める
- 名前の衝突を防ぎ、コードを整理しやすくなる

---

## バンドル / モジュールバンドル

JavaScript や CSS などの複数のファイルを1つのファイルにまとめること、またはそのまとめられたファイルのこと。
