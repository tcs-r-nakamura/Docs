2章

\*\*JavaScript（ジャバスクリプト）\*\*

Webページを動かすためのプログラミング言語



\*\*コールバック（Callback）\*\*

あとで呼び出してほしい処理（関数）を先に渡しておく仕組み



\*\*第一級関数（First-Class Function）\*\*

関数を数字や文字列と同じように使えること



\*\*クロージャ（Closure）\*\*

関数が、外側の変数を記憶して持ち続けること



注意点

メモリを持ち続ける → 不要なクロージャは解放しないと重くなる



\*\*ECMAScript（エクマスクリプト）\*\*

\*JavaScript の「仕様（ルールブック）」



&nbsp;	再代入 再宣言

var 　　　○　　 ○

let 	　○	 ✕

const 	　✕ 　　✕



基本const どうしても再代入するときはlet



\*\*巻き上げ（ホイスティング / Hoisting）\*\*

変数や関数の宣言が、実行前に先頭へ移動したように扱われる仕組み



宣言：巻き上げられる ✅

代入：巻き上げられない ❌



\*\*静的型付け言語\*\*

プログラムを書く時点（実行前）で、変数や値の型が決まっている言語（Type script）



\*\*動的型付け言語\*\*

変数の型を事前に決めず、実行時に型が自動で決まる言語（Java script）



\*\*プリミティブ型（Primitive Type）\*\*

数値や文字などの「値そのもの」を直接持つ型（イミュータブル）



| 型        | 例              |

| --------- | --------------- |

| number    | `10`, `3.14`    |

| string    | `"Hello"`       |

| boolean   | `true`, `false` |

| undefined | `undefined`     |

| null      | `null`          |

| symbol    | `Symbol("id")`  |

| bigint    | `123n`          |



\*\*オブジェクト型（Object型）\*\*

複数の値や処理をひとまとめにした「データのかたまり」（ミュータブル）



* オブジェクト {}
* 配列 \[]
* 関数 function () {}
* 日付 Date
* 正規表現 RegExp

（※ プリミティブ型以外はほぼオブジェクト）



\*\*プリミティブ値のリテラルとラッパーオブジェクト\*\*



プリミティブ値のリテラル

値そのものを直接書いたもの

let str = "Hello";  // 文字列リテラル

let num = 100;      // 数値リテラル

let flag = true;    // 真偽値リテラル



ラッパーオブジェクト（Wrapper Object）

プリミティブ型をオブジェクトとして扱うための箱

let strObj = new String("Hello"); // ラッパーオブジェクト

let numObj = new Number(100);

let boolObj = new Boolean(true);



console.log(typeof strObj); // "object"

console.log(typeof numObj); // "object"



リテラルとラッパーオブジェクトの関係



* 一見プリミティブでもメソッドが使える
* 実際には 内部で一時的にラッパーオブジェクトが生成される
* メソッド呼び出し後は自動で破棄される



let s = "Hello";

console.log(s.toUpperCase()); // "HELLO"



普段は リテラルを使う

必要なときだけ内部的にラッパーオブジェクトが作られる

new String() のように自分で作ることはほとんどない



\*\*オブジェクトリテラル（Object Literal）\*\*

{} の中にプロパティやメソッドを書いてオブジェクトを作る方法



* {} の中が プロパティ（key: value）
* 関数も メソッド として書ける



const user = {

&nbsp; name: "Alice",

&nbsp; age: 20,

&nbsp; greet() {

&nbsp;   console.log("こんにちは");

&nbsp; }

};

&nbsp;  				key   :  value

console.log(user.name); // Alice

user.greet();           // こんにちは



\*\*配列リテラル（Array Literal）\*\*

\[] の中に値をカンマで並べて配列を作る方法



* \[] の中が配列の要素
* 要素にはプリミティブ型もオブジェクト型も入れられる
* 数字、文字列、真偽値、オブジェクト、配列も混在可能



const numbers = \[1, 2, 3, 4, 5];

console.log(numbers\[0]); // 1

console.log(numbers.length); // 5



const mixed = \[10, "Hello", true, { name: "Alice" }, \[1,2,3]];

console.log(**mixed\[3]**.name); // Alice

console.log(**mixed\[4]\[1]**);   // 2



**mixed\[3].name の意味**



mixed\[3] → 配列の 4番目の要素 → { name: "Alice" }（オブジェクト）

.name → そのオブジェクトの name プロパティにアクセス

結果 → "Alice"



**mixed\[4]\[1] の意味**



mixed\[4] → 配列の 5番目の要素 → \[1, 2, 3]（配列）

\[1] → その配列の 2番目の要素（0始まり）にアクセス

結果 → 2



\*\*正規表現リテラル（RegExp Literal）\*\*

文字列のパターンを簡単に書いて検索や置換に使える仕組み



基本文法

const regex = **/abc/**;



/abc/ が 正規表現リテラル

abc という文字列パターンを表す



test() でマッチ確認



const regex = /hello/;

console.log(regex.test("hello world")); // true

console.log(regex.test("hi there"));    // false



match() で文字列検索



const str = "I love JavaScript";

const regex = /JavaScript/;

console.log(str.match(regex)); // \["JavaScript"]



フラグ（オプション）

フラグ	意味

i	大文字小文字を区別しない

g	全体にマッチ（global）

m	複数行モード



const regex = /hello/**gi**;

大文字小文字無視、文字列全体を検索



\*\*広義のオブジェクト\*\*



JavaScriptでオブジェクト型として扱えるすべての値

プリミティブ型以外は基本的にオブジェクト型



\*\*狭義のオブジェクト\*\*



一般的に「オブジェクト」と言ったときにイメージする箱型のデータ構造

波かっこ {} で作る プロパティの集合

配列や関数は含まない



const **user** = {

&nbsp; name: "Alice",

&nbsp; age: 20,

&nbsp; greet() { console.log("Hello"); }

};



**user**は狭義のオブジェクト



\*\*関数宣言（Function Declaration）\*\*

function キーワードを使って名前付きの関数を定義する方法



特徴

* 巻き上げ（Hoisting）される
* 名前が必須
* デバッグで便利（エラー時に関数名が表示される）



基本文法

function 関数名(引数1, 引数2, ...) {

&nbsp; // 処理内容

&nbsp; return 結果;

}



function greet(name) {

&nbsp; console.log("こんにちは、" + name + "!");

}



greet("Alice"); // こんにちは、Alice!



\*\*関数式（Function Expression）\*\*

関数を値として変数に代入する書き方（現代の書き方）



特徴

* 巻き上げ（Hoisting）されない（変数名を使用するため。宣言されていないため）
* 関数を値として扱える
* 柔軟な書き方が可能（匿名関数・名前付き関数・アロー関数）



基本文法

const 変数名 = function(引数1, 引数2, ...) {

&nbsp; // 処理内容

&nbsp; return 結果;

};



**匿名関数**



const greet = function(name) {

&nbsp; console.log("こんにちは、" + name + "!");

};



greet("Alice"); // こんにちは、Alice!



関数に名前を付けなくてもOK（匿名関数）

変数 greet が関数を参照



**名前付き関数式**



const greet = function sayHello(name) {

&nbsp; console.log("こんにちは、" + name + "!");

};



greet("Alice"); // OK

// sayHello("Alice"); // ❌ 外からは呼べない



名前付きでも、変数を通して呼び出す

名前は主にデバッグ用



\*\*第一級オブジェクト（First-Class Object / First-Class Citizen）\*\*

関数やオブジェクトを「値」として自由に扱えること

変数に代入したり、引数に渡したり、戻り値として返したりできる。



変数に代入



const greet = function(name) {

&nbsp; console.log("Hello " + name);

};

greet("Alice"); // Hello Alice



関数の引数として渡す



function callTwice(fn) {

&nbsp; fn("Bob");

&nbsp; fn("Charlie");

}

callTwice(greet);

// Hello Bob

// Hello Charlie



関数の戻り値として返す



function makeGreeter() {

&nbsp; return function(name) {

&nbsp;   console.log("Hi " + name);

&nbsp; };

}

const greeter = makeGreeter();

greeter("David"); // Hi David



配列やオブジェクトの要素として使える



const arr = \[greet, 1, 2];

arr\[0]("Eve"); // Hello Eve



\*\*グローバルスコープ（Global Scope）\*\*

プログラム全体からアクセスできる変数や関数の有効範囲（スコープ）\*\*のこと



\*\*無名関数（Anonymous Function）\*\*

関数に名前を付けずに、その場で作って使う関数



特長

* 変数に代入したり
* 関数の引数として渡したり
* その場で実行したり



変数に代入する無名関数



const greet = **function(name) {**

**console.log("Hello " + name);**

**};**



greet("Alice"); // Hello Alice



\*\*アロー関数式（Arrow Function）\*\*

アロー関数式は、関数を短く簡単に書ける新しい書き方

無名関数の代わりに使いやすく、特にコールバック関数で便利



特長

* function キーワードを使わずに書ける
* 引数が1つなら括弧 () を省略できる
* 処理が1行なら波括弧 {} と return も省略できる
* this の扱いが通常の関数と異なる（レキシカルスコープ）
* 無名関数の一種



// 通常の関数

const greet = function(name) { return "Hello " + name; };



const greet = function(name) {

&nbsp; return "Hello " + name;

};



// アロー関数

const greet = name => "Hello " + name;



const greet = (name) => {

&nbsp; return "Hello " + name;

};



// 引数がない場合



const sayHi = () => "Hi!";

console.log(sayHi()); // Hi!



// 配列操作での例



const numbers = \[1, 2, 3];

const doubled = numbers.map(n => n \* 2);

console.log(doubled); // \[2, 4, 6]



* メインのアプリケーションロジックや再利用前提の関数は function による宣言
* 短い処理やインラインで使う関数はアロー関数



\*\*様々な引数の表現\*\*



\*\*デフォルト引数（Default Parameter）\*\*

関数に渡されなかった引数に対して、あらかじめ設定した初期値を使う仕組み



特長

* 省略された引数に安全な値を設定できる
* コードがシンプルになり、if文でチェックする必要が減る



function greet(name = "Guest") {

&nbsp; console.log("Hello " + name);

}



greet("Alice"); // Hello Alice

greet();        // Hello Guest



name = "Guest" がデフォルト引数

引数を渡さなかったときに "Guest" が使われる



複数引数でも使える



function add(a = 0, b = 0) {

&nbsp; return a + b;

}



console.log(add(5, 3)); // 8

console.log(add(5));    // 5（bは0になる）

console.log(add());     // 0（aもbも0になる）



\*\*レストパラメータ（Rest Parameter）\*\*

関数の引数を1つの配列としてまとめて受け取る仕組み



特長

* 引数の個数が可変の関数を作れる
* 配列として扱えるので for ループや map などで処理可能



基本文法

function sum(**...numbers**) {

&nbsp; // numbers は配列

&nbsp; return numbers.reduce((total, n) => total + n, 0);

}



console.log(sum(1, 2, 3));      // 6

console.log(sum(4, 5, 6, 7, 8)); // 30



**...numbers** がレストパラメータ

引数がいくつ渡されても、すべて numbers 配列に入る



注意点

レストパラメータ（...）は 関数の最後の引数にしか置けない



function test(a, b, **...rest**) {// 〇　最後

&nbsp; console.log(a);    // 1

&nbsp; console.log(b);    // 2

&nbsp; console.log(rest); // \[3, 4, 5]

}



test(1, 2, 3, 4, 5);



function test(a, **...rest**, b) { // ❌ エラー

&nbsp; console.log(a, rest, b);

}



\*\*JavaScript のクラス構文（Class）\*\*

オブジェクトを作るための設計図（テンプレート）を簡単に書ける文法



特長

* lass キーワードで定義
* constructor メソッドで初期化
* メソッドはクラスの中に直接書く
* new でインスタンス化する



基本文法



// クラス定義

class Person {

&nbsp; constructor(name, age) {   // 初期化用の特別なメソッド

&nbsp;   this.name = name;

&nbsp;   this.age = age;

&nbsp; }



&nbsp; // メソッド

&nbsp; greet() {

&nbsp;   console.log(`Hello, my name is ${this.name}`);

&nbsp; }

}



// クラスからインスタンスを作る

const alice = new Person("Alice", 20);

alice.greet(); // Hello, my name is Alice



\*\*プライベートメンバー（Private Member）\*\*

クラスの中で定義され、クラスの外から直接アクセスできないプロパティやメソッドのこと



特長

* データの隠蔽（カプセル化）に使える
* クラス外から不用意に変更されないようにできる
* クラス設計の安全性が高まる
* \# を付けたら、クラスの内側専用の変数や関数



class Person {

&nbsp; #name; // プライベートプロパティ



&nbsp; constructor(name) {

&nbsp;   this.#name = name;

&nbsp; }



&nbsp; // プライベートメンバーを使うメソッド

&nbsp; greet() {

&nbsp;   console.log(`Hello, my name is ${this.#name}`);

&nbsp; }

}



const alice = new Person("Alice");

alice.greet();      // Hello, my name is Alice

console.log(alice.#name); // ❌ エラー：クラス外からアクセスできない



\*\*静的プロパティ（Static Property）\*\*

クラス自体に属し、インスタンスを作らなくてもアクセスできるプロパティのこと



\*\*静的メンバー（Static Member）\*\*

クラス自体に属し、インスタンスを作らなくても使えるプロパティやメソッドのこと（newではなく、クラス名から呼び出す）



特長

* 静的メンバーは クラス自体に属する
* インスタンスを作らなくても呼び出せる
* インスタンス固有のデータには使えない（共通の情報や処理向け）



class Circle {

&nbsp; static pi = 3.14; // 静的プロパティ



&nbsp; constructor(radius) {

&nbsp;   this.radius = radius;

&nbsp; }



&nbsp; // 静的メソッド

&nbsp; static calcArea(radius) {

&nbsp;   return Circle.pi \* radius \* radius;

&nbsp; }



&nbsp; // インスタンスメソッド

&nbsp; getArea() {

&nbsp;   return Circle.pi \* this.radius \* this.radius;

&nbsp; }

}



// 静的メンバーはクラス名からアクセス

console.log(Circle.pi);           // 3.14

console.log(Circle.calcArea(5));  // 78.5



// インスタンスメンバーはインスタンスからアクセス

const c = new Circle(5);

console.log(c.getArea()); // 78.5

// console.log(c.calcArea(5)); // ❌ エラー：インスタンスからは呼べない



種類			アクセス方法		特徴

プライベートメンバー	クラス内部のみ		外からアクセス不可、カプセル化

静的メンバー		クラス名から		インスタンス不要、クラス共通

インスタンスメンバー	インスタンスから	個別のオブジェクトごとのデータ



プライベート = 「クラス内専用」

静的 = 「クラス全体で共通」

インスタンス = 「オブジェクト固有」



\*\*コンストラクタ関数（Constructor Function）\*\*

「オブジェクトを作る設計図」を関数で表したもの

new を使うと、その関数から新しいオブジェクトが作られる



特長

* 関数名は大文字で始めるのが慣習
* this は作られたオブジェクトを指す
* new を使わないと this が期待通りにならず、エラーやグローバル汚染の原因になる



// コンストラクタ関数の定義

function Person(name, age) {

&nbsp; this.name = name; // インスタンスプロパティ

&nbsp; this.age = age;

&nbsp; this.greet = function() { // インスタンスメソッド

&nbsp;   console.log(`Hello, my name is ${this.name}`);

&nbsp; };

}



// インスタンスの作成

const alice = new Person("Alice", 20);

const bob = new Person("Bob", 25);



alice.greet(); // Hello, my name is Alice

bob.greet();   // Hello, my name is Bob



\*\*プロトタイプベース（Prototype-based）\*\*

オブジェクトが他のオブジェクトを「プロトタイプ」として継承する方式のこと



特長

* JavaScript では クラスよりもプロトタイプが基本
* オブジェクトは他のオブジェクトを「親」として持てる
* 継承は「プロトタイプチェーン（Prototype Chain）」で解決される



ES6 のクラス構文も内部的には プロトタイプベース

// プロトタイプとして使うオブジェクト

const animal = {

&nbsp; speak() {

&nbsp;   console.log(`${this.name} is making a sound`);

&nbsp; }

};



// 新しいオブジェクトを作る

const dog = Object.create(animal);

dog.name = "Pochi";

dog.speak(); // Pochi is making a sound



dog は animal をプロトタイプとして持つ

dog に speak がなくても、animal から継承して呼び出せる



\*\*プロトタイプチェーン（Prototype Chain）\*\*

ブジェクトにプロパティやメソッドが存在しない場合、

そのオブジェクトの「プロトタイプ（親オブジェクト）」を順番にたどって探す仕組み



特長

* オブジェクト → プロトタイプ → さらにそのプロトタイプ … とたどる
* 最上位のプロトタイプは Object.prototype
* それ以上は null で終了



const animal = {

&nbsp; speak() {

&nbsp;   console.log(`${this.name} makes a sound`);

&nbsp; }

};



const dog = Object.create(animal);

dog.name = "Pochi";



dog.speak(); // Pochi makes a sound



dog に speak はない

dog のプロトタイプ animal を見に行く

animal に speak があった → 呼び出せる



console.log(Object.getPrototypeOf(dog)); // animal

console.log(Object.getPrototypeOf(animal)); // Object.prototype

console.log(Object.getPrototypeOf(Object.prototype)); // null



\*\*プロトタイプベースのオブジェクト指向（Prototype-based OOP）\*\*

「オブジェクトを直接作って、別のオブジェクトをコピー・継承する」ことで機能を共有する方法

クラスを使うのではなく、プロトタイプ（親オブジェクト）をもとに新しいオブジェクトを作る



特長

* クラス不要
* ES6 の class はシンタックスシュガー（可読性の高い書き方）
* 内部ではプロトタイプベースの継承が使われている
* 継承はオブジェクト同士で行う
* オブジェクトをプロトタイプとして他のオブジェクトに設定
* 動的にオブジェクトの機能を追加・変更できる
* プロトタイプチェーン でプロパティやメソッドを探索



// 親オブジェクト（プロトタイプ）

const animal = {

&nbsp; speak() {

&nbsp;   console.log(`${this.name} makes a sound`);

&nbsp; }

};



// 子オブジェクトを作る

const dog = Object.create(animal);

dog.name = "Pochi";



dog.speak(); // Pochi makes a sound



dog は animal をプロトタイプとして継承

dog にメソッドがなくても animal から呼べる（プロトタイプチェーン）



クラスとの違い



// クラス（ES6）

class Animal {

&nbsp; constructor(name) { this.name = name; }

&nbsp; speak() { console.log(`${this.name} makes a sound`); }

}

const dog = new Animal("Pochi");

dog.speak();



見た目はクラスだが、内部ではプロトタイプチェーンで継承している

JavaScript の本質は プロトタイプベース

現在はクラス構文を使用する



\*\*分割代入（Destructuring Assignment）\*\*

配列やオブジェクトから値を まとめて取り出して変数に代入する便利な書き方



特長

* 配列は順番で対応、オブジェクトはキー名で対応
* 複数の値をまとめて取り出すときに非常に便利
* ネストした配列・オブジェクトにも使える



配列の分割代入

const arr = \[10, 20, 30];



// 従来

const a = arr\[0];

const b = arr\[1];



// 分割代入

const \[x, y, z] = arr;



console.log(x, y, z); // 10 20 30



const arr = \[10, 20, 30, 40];



// 2番目の要素を無視して1番目と3番目だけ取り出す

const \[first, **,** third] = arr;



console.log(first); // 10

console.log(third); // 30



左側の ,（カンマ）で空欄にすると、その位置の要素をスキップできる

スキップした要素は変数に代入されない



オブジェクトの分割代入



const user = { name: "Alice", age: 25 };



// 従来

const name = user.name;

const age = user.age;



// 分割代入

const { name, age } = user;



console.log(name, age); // Alice 25



const user = { name: "Alice", age: 25 };

&nbsp;		

const { name: userName, age: userAge } = user;

console.log(userName, userAge); // Alice 25



左側の name: userName は

user.name(Alice) を取り出して

変数名 userName に代入する

同じく age: userAge で user.age(25) → userAge



左側の {} の キー名 と同じ名前の変数に代入される

別名で受け取りたい場合は : を使う

{ 元のキー名 : 新しい変数名 } で書く

関数の引数や複雑なオブジェクトを扱うときに便利



関数の引数で使う



function greet({ name, age }) {

&nbsp; console.log(`Hello ${name}, ${age} years old`);

}



greet({ name: "Alice", age: 25 }); // Hello Alice, 25 years old



配列は順番で対応、オブジェクトはキー名で対応

複数の値をまとめて取り出すときに非常に便利

ネストした配列・オブジェクトにも使える



\*\*スプレッド構文（Spread syntax）\*\*

配列やオブジェクトの要素やプロパティを 展開（spread）して取り出す構文 のこと



特長

* ... がついているものは「展開する」という意味
* レスト構文（...args のように残りの要素をまとめる）とは用途が逆
* 配列・オブジェクトを扱うときに非常に便利



使う場面

* 配列やオブジェクトのコピー
* 配列やオブジェクトの結合・マージ
* 関数の引数に配列を展開して渡す



配列のコピー

const arr1 = \[1, 2, 3];

const arr2 = \[...arr1];



console.log(arr2); // \[1, 2, 3]



配列の結合

const arr3 = \[4, 5];

const combined = \[...arr1, ...arr3];



console.log(combined); // \[1, 2, 3, 4, 5]



オブジェクトのコピー

const obj1 = { a: 1, b: 2 };

const obj2 = { ...obj1 };



console.log(obj2); // { a: 1, **b: 2** }



オブジェクトのマージ

const obj3 = { **b: 3**, c: 4 };

const merged = { ...obj1, ...obj3 };



console.log(merged); // { a: 1, **b: 3**, c: 4 }



同じキーがある場合は、後ろのオブジェクトで上書きされる



\*\*シャローコピー（Shallow Copy）\*\*

オブジェクトや配列をコピーするときに「1階層目だけ」を複製し、

中に入っているオブジェクトは同じ参照を共有するコピー方法



補足

* スプレッド構文はシャローコピー
* 「中身がオブジェクトなら要注意」



オブジェクト

const original = {

&nbsp; name: "Alice",

&nbsp; address: { city: "Tokyo" }

};



const shallow = { ...original }; // シャローコピー



配列の例

const arr1 = \[\[1, 2], \[3, 4]];

const arr2 = \[...arr1]; // シャローコピー



arr2\[0]\[0] = 99;

console.log(arr1\[0]\[0]); // 99



よく使われるシャローコピー方法

配列

const copy1 = \[...arr];

const copy2 = arr.slice();

const copy3 = Array.from(arr);



オブジェクト

const copy1 = { ...obj };

const copy2 = Object.assign({}, obj);



コピー後の状態（重要）

original ──┐

&nbsp;           ├─ address ──→ { city: "Tokyo" }

shallow  ──┘



注意点

シャローコピー自体は破壊的ではない

共有参照の中身を変更すると**破壊的**になる

「破壊的かどうか」は コピー方法ではなく、変更の仕方で決まる



\*\*ディープコピー（Deep Copy）\*\*

オブジェクトや配列を「中身の中身まで」すべて新しくコピーし、

元データと一切の参照を共有しないコピー方法



元のオブジェクト

const original = {

&nbsp; name: "Alice",

&nbsp; address: { city: "Tokyo" }

};



ディープコピー

const deep = structuredClone(original);



コピー後の状態（重要）

original ──┐

&nbsp;           ├─ address ──→ { city: "Tokyo" }

deep     ──┐

&nbsp;           ├─ address ──→ { city: "Tokyo" }



ディープコピーの方法（代表的）

① structuredClone()（推奨・最新）

const copy = structuredClone(obj);



シャローコピーとの違い

項目		シャロー	ディープ

コピー範囲	1階層		全階層

参照共有	あり		なし

安全性		低		高

パフォーマンス	高速		重い



\*\*ショートサーキット評価（Short-circuit Evaluation）\*\*

論理演算子の途中で結果が確定した時点で、残りの式を評価しない仕組みのこと



\&\&（AND）



false \&\& doSomething();



左が false → 結果は必ず false



👉 doSomething() は 実行されない



true \&\& doSomething();





左が true → 右を評価



👉 doSomething() が実行される



||（OR）



true || doSomething();





左が true → 結果は必ず true



👉 doSomething() は 実行されない



false || doSomething();





左が false → 右を評価



👉 doSomething() が実行される



??（Null合体演算子）との違い

const count = value ?? 0;



null / undefined のときだけ右を使う



0 や "" はそのまま使われる



\*\*Javascriptにおけるthis\*\*



場面			  this が指すもの



const obj = {

&nbsp; name: "Alice",

&nbsp; greet() {

&nbsp;   console.log(this.name);

&nbsp; }

};



obj.greet(); // Alice

this は obj を指す



this.name → "Alice"



メソッド呼び出し	  呼び出したオブジェクト



普通の関数の場合

function hello() {

&nbsp; console.log(this);

}



hello();





ブラウザでは window（グローバルオブジェクト）



strict モードでは undefined



普通の関数呼び出し	  グローバルオブジェクト（strict モードでは undefined）



class Person {

&nbsp; constructor(name) {

&nbsp;   this.name = name;

&nbsp; }

&nbsp; greet() {

&nbsp;   console.log(this.name);

&nbsp; }

}



const p = new Person("Bob");

p.greet(); // Bob

this は そのインスタンス（p） を指す



コンストラクタ / クラス   新しく作られたインスタンス



アロー関数の場合



const obj = {

&nbsp; name: "Alice",

&nbsp; arrow: () => console.log(this.name)

};



obj.arrow(); // undefined





アロー関数は 自分の this を持たず、外側の this を参照



従来の関数とは挙動が違うので注意



アロー関数		  外側の this を参照



this のまとめ

呼び出し方			this が指すもの

グローバルスコープ		グローバルオブジェクト (window/global)

オブジェクトのメソッド		そのオブジェクト

単独関数（非 strict モード）	グローバルオブジェクト

単独関数（strict モード）	undefined

コンストラクタ関数 (new)	新しいインスタンス

アロー関数			外側のスコープの this



thisは使われていたら**アンチパターン**



\*\*strict モード（厳格モード）\*\*

JavaScript の より安全で厳密な書き方を強制するモード



有効化方法



ファイルや関数の先頭に 'use strict'; と書く



'use strict';



x = 10; // ❌ エラーになる（宣言なしの変数代入は禁止）



特長

* 宣言なしの変数の使用禁止
* this の安全化
* 重複するプロパティや引数の禁止
* 読み取り専用プロパティの変更禁止
* 将来の予約語の使用制限（implements, interface, package, private などは使用できなくなる）



'use strict';

y = 5; // ReferenceError



'use strict';

function hello() {

&nbsp; console.log(this); // undefined

}

hello();



普通の関数呼び出しで this が undefined になる

グローバルオブジェクトを誤って書き換えなくなる（非破壊的）



'use strict';

function sum(a, a) { } // ❌ エラー



'use strict';

const obj = {};

Object.defineProperty(obj, 'x', { value: 10, writable: false });

obj.x = 20; // ❌ エラー



\*\*エイリアス（alias）\*\*

あるものに別名（別の名前）をつけて参照できるようにすること



\*\*モジュール（Module）\*\*

プログラムの機能をまとめて、他のファイルやプログラムから再利用できる単位のこと



特徴

* 独立したファイルや単位として作ることができる
* 必要な機能だけを import/export で読み込める
* 名前の衝突を防ぎ、コードを整理しやすくなる



\*\*モジュールシステム\*\*

JavaScript で コードを小さな単位（モジュール）に分けて管理・再利用する仕組み のこと



特長

* モジュールは独立したスコープを持つ（ファイル内の変数は外部に漏れない）
* 必要なものだけを明示的にエクスポート/インポートする
* これにより、大規模アプリでも 管理しやすく安全なコード が書ける



\*\*バンドル\*\*

JavaScript や CSS などの複数のファイルを 1つのファイルにまとめること、またはその まとめられたファイル のこと



\*\*モジュールバンドル（Module Bundle）\*\*

JavaScript の モジュール（ファイル）をまとめて 1 つのファイルに変換したもの、またはその処理のこと



