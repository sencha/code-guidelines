#保守性の高い JavaScript を書く

ロバート グラスは、その著書
*Facts and Fallacies of Software Engineering*
 (訳注: 邦題「ソフトウエア開発 55の真実と10のウソ」)
において、ソフトウェアのライフタイムを通じてのトータルコストの 80%が保守に費やされると述べています。

その生涯の上のソフトウェアの総経費の80%もどのようにメンテナンス作業から来るかについて、ロバートGlassは議論する。
それは、バグ修正だけではなく、
多くの場合「保守」とは新しい機能を既存のアプリケーションに加えることが実際に必要となります。

この意味は明白です。
開発者がアプリケーションの保守に多くの時間を費やすのなら、常にコードはそのプロセスを簡単にするように書かれるべきであるということです。

Sencha は、大きなコードベースを保守することに長期の不一致を軽減するために、次のポイントを優先させることを推奨します。

  - [ファイル毎に一つのクラス](#One_Class_Per_File)
  - [変数の宣言](#Declaring_Variables)
  - [配列リテラル](#Array_Literals)
  - [デバッグ ステートメント](#Debugging_Statements)
  - [三項演算子](#Ternary_Operators)
  - [正規表現式](#Regular_Expressions)
  - [文字列](#Strings)
  - [メソッド チェーン](#Method_Chains)

## <a name="One_Class_Per_File" />ファイル毎に一つのクラス

Senchaは、ファイル毎に一つのクラスだけをコード化する習慣を強く推奨します。
そうすると、開発者がどこでコードを見つけのか正確にわかることになるので、アプリケーションを保守、デバッグすることのがより簡単になります。
完全な名前空間につき、ファイルは『/src/Foo/バー/Baz.js』に位置するかもしれない。

さらに Sencha は、あなたのファイルに名前をつけ管理する一貫したアプローチを使うことを推奨します。

ファイル名は定義されるクラス名にマッチし、実際のファイル場所はクラスの名前空間にマッチするようにします。

たとえば、クラス `Foo.bar.Baz` は、`Baz.js` で定義します。
完全な名前空間でいくと、このファイルは
`/src/Foo/bar/Baz.js`.
に配置されます。

## <a name="Declaring_Variables" />変数の宣言

Sencha は一般的に、変数の宣言はそのスコープの先頭で宣言するというクロックフォードのアドバイスに従います。なぜなら、

1. 変数の宣言を見つけるのが簡単になる
2. 変数のスコープが明白である (JavaScript にはブロックスコープはないので)
3. いずれにせよ変数の宣言は、
スコープの先頭に[巻き上げ](http://www.adequatelygood.com/JavaScript-Scoping-and-Hoisting.html)られます。

```
function foo() {
    var bar = 1; // 良い

    if (true) {
        var baz = 2; // 悪い; "baz" は前に宣言しましょう
    }
}
```

### <a name="Declaring_Multiple_Variables" />複数の変数の宣言

複数の変数を生成する時には一つの `var ` 宣言を使います。それは読みやすいからです。
Sencha は各変数の割り当てを新しい行で宣言することを推奨します。
値を割り当てない変数は最後に、同じ行に書きます。
現在のスコープでのその変数の初期状態についてコードを読んでいる人にとって、しかくてきなヒントを提供します。

    // 悪い
    var foo = 1;
    var bar = 2;
    var baz;
    var fuz;

    // 良い
    var foo = 1,
        bar = 2,
        baz, fuz;

### <a name="Declaring_Object_Properties" />オブジェクトのプロパティ

[オブジェクトのプロパティにアクセス](http://stackoverflow.com/questions/4968406/javascript-property-access-dot-notation-vs-brackets)する際、Sencha は、ブラケット記述よりもドット記述を使うことを選びます。その方が一般的に読みやすくより簡単に圧縮できるからです。

    var myObject = {
        foo : 'bar'
    };

    // 動作するが、好ましくない
    myObject['foo'];

    // 好ましい
    myObject.foo;

ブラケット表記によってドット表記で使うことができない文字でプロパティを宣言することができる点に留意してください。

    myObject['~foo'] = true; // うまく動作します
    myObject.~foo = true; // エラーが発生します

クロックフォードなどはブラケット表記を推奨しますが、できるだけコード内で一貫しているようにしましょう。

最後に、ブラケット記法は、オブジェクトのプロパティに変数を使ってアクセスするときに非常に便利です。

    var prop = 'foo';
    alert(myObject[prop]);

### <a name="Default_Values" />デフォルト値

また、多くの場合、デフォルト値を定義することはよい方法であると考えられます。

単純なシチュエーションでは、`||` を使ってデフォルト値を設定できます。
左側の値が falsy だった場合、右側の値が使われます。

    function init(config) {
        this.hidden = config.hidden || false;
    }

より複雑なシチュエーションでは、バリデーションを実行する、条件式やユーティリティ メソッドを使うのがいいでしょう。

    function init(config) {
        // 条件分岐を使う
        this.total = (config.count > 10) ? config.count : 10;

        // コンフィグから全てのプロパティをコピーし
        // デフォルト値を設定するユーティリティ メソッドを使う
        Ext.apply(this, config, {
            hidden : false
        });
    }

## <a name="Array_Literals" />配列リテラル

ほとんどの場合、新たに配列を生成する場合は、大括弧記法を使います。

    // 悪い
    var stuff = new Array();

    // 良い
    var stuff = [];

メンバーが連続した整数であるときに配列を使います。
メンバー名が、任意の文字列や名前である連想配列 (つまり、ディクショナリ) は作ってはいけません。
この場合にはかわりにオブジェクトを生成しましょう。

    // 悪い
    var stuff = [];
    stuff['foo'] = 'bar';
    stuff[0]; // undefined!

    // 良い
    var stuff = [ 'bar' ];
    stuff[0]; // 'bar'

ある場合には、
[パフォーマンスの理由で](http://www.slideshare.net/doris1/oscon-developing-high-performance-websites-and-modern-apps-with-javascript-and-html5faster-siteandappformeetupforoscon)
次元を固定した宣言がよい場合があります。

この場合には、`new Array(length)` 表記を代わりに使いますが、これには多くの影響があります。

## <a name="Debugging_Statements" />デバッグ ステートメント

`console.log()` や `debugger` などのデバッグ ステートメントは、決して標準の製品環境にいれて出荷してはなりません。

よりよいアプローチは、簡単に無効にできるようにこれらのサービスを焼くことです。

    // 悪い
    function foo() {
        console.log('inside the foo() method');
        return true;
    }

    // 良い
    function foo() {
        // logger() は開発環境でのみステートメントを出力する
        MyApp.util.logger('inside the foo() method');
        return true;
    }

Sencha Cmd を使っている Sencha アプリケーションでは、コードの一部が開発時のみに動作するようにする機能があります。
Sencha Cmd は、製品版ビルドでの最適化の際にこの部分を排除します。

    function foo() {
        // <debug>
        console.log('inside the foo() method');
        // </debug>
        return true;
    }

## <a name="Ternary_Operators" />三項演算子

三項演算子は、明確な条件では良いが、複雑な条件では使わない方が良い。

    // 悪い
    var value = a && b ? 11 : a ? 10 : b ? 1 : 0;

    // 良い
    var value = isSimple ? 11 : 1;

三項演算子はネストしてはいけません。それは単に混乱を招くだけだからです。

**経験則:**
三項演算子を使うかどうか自信が持てない場合は、デフォルトでは `if/else` 構文を使うようにしましょう。

しかし、三項演算子において、式に括弧を使うと、他の演算子との関連がはっきりします。
例えば、

    // 悪い
    var value = x || y ? z : a + b;

    // 良い
    var value = (x || y) ? z : (a + b);

## <a name="Regular_Expressions" />正規表現式

> *"問題があってその解決に RegEx を使う? それは二つ目の問題を抱えることだ。"* (詠み人知らず)

正規表現式は非常にパワフルですが、混乱の元になり得ます -- ですからそれらの保守性を高めることの優先度は高くなります。

正規表現式をインラインで使ってはいけません。読みやすさと[パフォーマンス](http://jsperf.com/caching-regex-objects/15)のためにいったん変数に保存し、つねにその用途を説明するコメントを書きましょう。

    // 悪い
    var hasNumbers = /\d+/.test(value);

    // 良い
    var numberTest = /\d+/, //一つ以上の数字が含まれる
        hasNumbers = numberTest.test(value);

パフォーマンスに関しては、正規表現式のリテラルをより永続的な方法にキャッシュすることです。
そうすると、関数が実行される度に再コンパイルされません。

    // 悪い
    function hasNumbers(value) {
        var numberTest = /\d+/; //関数が呼び出される度に再定義される

        return numberTest.test(value);
    }

    // 良い
    Ext.define('MyApp.util.RegEx', {
        // 一度だけ定義される
        numbersRe : /\d+/,

        hasNumbers : function(value) {
            return this.numbersRe.test(value);
        }
    });

## <a name="Strings" />文字列

文字列に定義における、シングル クオート／ダブル クォートに一貫したアプローチを取りましょう。
そうするとコードが読みやすく、検索しやすくなります。

    // 悪い
    var stringA = 'A', // シングル クォート
        stringB = "B"; // ダブル クォート

    // 良い
    var stringA = 'A',
        stringB = 'B';

時に、文字列が長すぎる (例えば 80文字以上になる) 場合に、読みやすくするためだけにそれをいくつかの行に分割しなければならないことがあります。

    // 悪い
    var stringA = 'Who lives in a pineapple under the sea? Sponge Bob Square Pants! Absorbent and yellow and porous is he!';

    // 良い
    var stringA = 'Who lives in a pineapple under the sea? ' +
        'Sponge Bob Square Pants! ' +
        'Absorbent and yellow and porous is he!';

    // または Array.join() を使います
    var stringA = [
        'Who lives in a pineapple under the sea?',
        'Sponge Bob Square Pants!',
        'Absorbent and yellow and porous is he!'
    ].join(' ');

## <a name="Method_Chains" />Method Chains

クロックフォードが *JavaScript: The Good Parts* で論じたように、メソッドチェーン (または彼が呼ぶように「カスケード」) を使うと、一つの命令で同じオブジェクトの多くのメソッドを順番に呼び出すことができます。
これらのメソッドは単純に `this` を返すことでチェーンを継続できます。　

    // 擬似コード
    getElement('myDiv').width(100).height(100).color('red') // のような

Sencha はこのパターンについてはいくつかの注意を喚起します。
メソッドチェーンの長さが、大きなコードベースの中で時間と共にスパゲッティコードに変わることがあり、これらのコールスタックをデバッグする際には非常に混乱することがあります。

多くの場合は、この振る舞いの多くをユーティリティ メソッドの中に抽象化する (そして複数の場所で再利用する) ことで、コードの長期間の保守性を改善することができます。
