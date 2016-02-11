#JavaScript エラーを回避する

[防御的プログラミング](http://en.wikipedia.org/wiki/Defensive_programming)
は、よくある問題が発生する可能性を減らすよう努めるソフトウェア構築アプローチです。
JavaScript 構文は、おかしな動きをしたり微妙な動きをする場合があるので、防御的プログラミング思考法を採用することで、大きなコードベースでの多くの実行時エラーを防ぐことができます。

 - [Lintをかける](#Removing_Lint)
 - [セミコロンを使う](#Semicolons)
 - [末尾のカンマ](#Trailing_Commas)
 - [中括弧とブロック](#Brackets_Blocks)
 - [比較演算子](#Equality)
 - [プロトタイプ](#Prototypes)
 - [関数](#Functions)

## <a name="Removing_Lint" />Lintをかける

JavaScript の「lint」ツールはソースコードを読んで、よくあるミスを特定します。例えば関数内の複数の `var` 指令や、文法エラーなどです。
これらのミスは不要なものと見なされ、削除するか再構成することになります。

開発中には JSLint、ESLint、JSHit などのツールを使うことが常に推奨されます。
ポピュラーな多くの IDE では、これらのツールと直接的に統合されています。

しかし、これらのツールはテストやビルドプロセスの自動化の一部として考えなければ鳴りません。
IDE 統合は、エラーについてだけ開発者に警告します。
問題を解決しなければならないわけではありません。そして、リポジトリに悪いコードがコミットされることを防ぐわけではありません。

## <a name="Semicolons" />セミコロンを使う

自動的なセミコロン挿入 (Automatic Semicolon Insertion:ASI) は機能ではありません。
[これに頼らないようにしましょう。](http://benalman.com/news/2013/01/advice-javascript-semicolon-haters/)

クロックフォードは、単純なものであっても全ての命令の最後にはセミコロンをつけることを推奨しています。
そうするとあらゆる式が命令として使われ、トリッキーなエラーから守られるるからです。

    // 悪い
    var a = obj
    [a].forEach(logProp);
    // セミコロンがないおかげで次の様な動作をすることになります
    // var a = obj[a].forEach(logProp)

    // 良い
    var a = obj;
    [a].forEach(logProp); // this works fine


    // 悪い
    var example = function() {
        // 行が変わったので、"undefined" が返されます
        return
        {
            foo: 123
        };
    };

    // 良い
    var example = function() {
        return {
            foo: 123
        };
    };

## <a name="Trailing_Commas" />末尾のカンマ

末尾のカンマは、おそらく他の何よりも JavaScript 開発において、長年の頭痛の元でした。

    var myObject = {
        foo : 1,
        bar : 2, // 末尾のカンマ
    };

    var myArray = [ 1, 2, 3, ]; // 末尾のカンマ

現在の ECMAScript 5 仕様では、オブジェクト リテラルや配列リテラルでの末尾のカンマを許可していますが、古いブラウザー (特に IE 9より前) では、予期しない挙動に出会います。オブジェクトでの末尾のカンマはランタイムエラーになり、配列リテラルの末尾のカンマは、Array.length の戻り値が不正確な値になります。
従って、ベストプラクティスとしては、Sencha では開発者にこれらを使うことを禁じています。


これに関連して、開発者の中にはこの問題を回避するために、カンマを先頭につけることをします。
Sencha はその解決法が問題を適切に解決するとは思えません。コードの読みやすさを下げるものだと信じています。

## <a name="Brackets_Blocks" />中括弧とブロック

どんな種類のコードブロックを作る時でも常に中括弧を使います。
全てのブロック、たとえそれが 1行だったとしても、混乱を避けるために中括弧で囲むべきです。

    // 悪い
    if (foobar) doSomething();

    // 良い
    if (foobar) {
        doSomething();
    }

多くの場合に、
[ガード条件](http://refactoring.com/catalog/replaceNestedConditionalWithGuardClauses.html)
を使うことは、「通常の」実行パスへの例外を強調できるので、よい方法です。

    // 悪い
    function getPayAmount() {
        var result;

        if (_isDead) { result = deadAmount(); }
        else {
            if (_isSeparated) { result = separatedAmount(); }
            else {
                if (_isRetired) { result = retiredAmount(); }
                else { result = normalPayAmount(); }
            };
        }

        return result;
    };  


    // 良い
    function getPayAmount() {
        if (_isDead) {
            return deadAmount();
        }
        if (_isSeparated) {
            return separatedAmount();
        }
        if (_isRetired) {
            return retiredAmount();
        }

        return normalPayAmount();
    };  

## <a name="Equality" />比較演算子

特別な理由が無い限り、`==` や `!=` オペレータではなく、`===` や `!==` を使いましょう。

「厳密な比較演算子」を使うことで、正確に値を比較でき、変数の型も比較されます。

    // 悪い
    var result = (0 == false); //returns TRUE

    // 良い
    var result = (0 === false); //returns FALSE

厳密ではない比較演算子 (`==` や `!=`) は、
同じ型に[オペランドをキャストしようとし](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators)、
予期しない truthy/falsy な値を返すことがあります。

同じ問題が、if ステートメントの中で[ネイティブの型を比較する](http://javascriptweblog.wordpress.com/2011/02/07/truth-equality-and-javascript/#more-2108) 際にも存在します。

    function compare(val) {
        return (val) ? true : false;
    }

    compare({}); // true となる
    compare([]); // Array は Object なので true となる

    compare(undefined); // false となる
    compare(null);      // false となる

    compare(true);  // true  となる
    compare(false); // false となる

    compare(0);   // +0, -0 false となる
    compare(NaN); // false となる
    compare(1);   // その他の全ての正の値は true となる
    compare(-1);  // その他の全ての負の値は true となる

    compare('');    // 空文字列は false となる
    compare('foo'); // 他の文字列は true となる

簡単に言うと、厳密な比較を用いない場合には、変数を比較する際には非常に注意する必要があるということです。

しかし、直接的な比較をしないで truthy/falsy な値を使うとよい状況もあります。
しかし、もう一度いいますが、開発者は期待される値について慎重であるべきです。


    if (!disabled) {
        // ...
    }

    // または
    if (enabled) {
        // ...
    }

## <a name="Prototypes" />プロトタイプ

JavaScript はプロトタイプベースの言語です -- すべてのオブジェクトは他のオブジェクトを直接的に継承し、フォーマルな「クラス」は存在しません。
プロトタイプ的な継承は概念上はクラス的な継承と似ていますが、クロックフォードは
「JavaScript はそのプロトタイプ性質について混乱している、なぜならプロトタイプ機構はなんとなく古典的に見える複雑な文法的な事柄によって、プロトタイプ・メカニズムがわかりにくくなっているからだ」
と指摘しています。

簡単に言うと、プロトタイプ型の継承のニュアンスを理解することが、エラーを防ぐキーになるということです。

### <a name="Native_Prototypes" />ネイティブのプロトタイプ

ネイティブのプロトタイプをハックすることはしてはいけません。
[名前の衝突や、互換性のない実装の可能性を](http://www.nczonline.net/blog/2010/03/02/maintainable-javascript-dont-modify-objects-you-down-own/)
を増大させ、必然的に見つけにくいバグに悩まされることに鳴ります。

代わりに、お望みの動作を実装するためのユーティリティクラスやメソッドを生成します。

    // 悪い
    Array.prototype.each = function(functionToCall) {
        //配列の要素をループする
    };

    // 良い
    Ext.define('Ext.Array', {
        singleton : true,

        each : function(arrayToIterate, functionToCall) {
            //配列の要素をループする
        }
    }};

注: 場合によっては、ネイティブのプロトタイプをポリフィルすることは、古いブラウザに標準の動作を追加する時には使ってもいいでしょう。
例えば、Ext JS 5 は、`Function.bind()` を IE8 にポリフィルしています。

## <a name="Functions" />関数

### <a name="Function_Hoisting" />関数の巻き上げ

JavaScript の関数を定義する際には、[巻き上げ](http://elegantcode.com/2011/03/24/basic-javascript-part-12-function-hoisting/)に注意しましょう。

関数定義は、パース時 (ブラウザが最初にコードをダウンロードした際) に評価されます。

    // 関数宣言 (好ましい)
    function sum(x, y) {
      return x + y;
    }

なぜなら、宣言は関数がいつ定義されたかに関係なく、パース時にスコープの先頭に巻き上げられます。

    sum(1,2); // 3 を返す

    // 関数宣言 (好ましい)
    function sum(x, y) {
      return x + y;
    }

関数式は、他の変数の割り当てと同様、実行時 (コールスタックが実際にコードのその行に来たとき) に評価されます。

    // 関数式
    var sum = function(x, y) {
        return x + y;
    };

関数式は、パース時に巻き上げられないので、関数が定義される時が問題になります。

    sum(1,2); //"undefined is not a function" エラーが発生する

    // 関数式
    var sum = function(x, y) {
        return x + y;
    };

### <a name="Anonymous_Functions" />無名関数

無名関数はとても便利ですが、貧弱な構成のコードでは、簡単に[メモリリーク](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript#Memory_leaks)を引き起こします。
次のサンプルを検討してください。

    function addHandler() {
        var el = document.getElementById('el');

        el.addEventListener(
            'click',
            function() { // 無名関数
                el.style.backgroundColor = 'red';
            }
        );
    }

ここでは、無名関数を使うことによって二つの問題が発生しています。

  1. click ハンドラー関数には名前付きの参照がないため、[removeEventListener()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget.removeEventListener) で削除することができません。
  2. `el` への参照が無意識で内側の関数でクロージャーを形成しています。
  そしてそのためガーベージコレクションすることができません。
  これは、JavaScript (関数) と DOM (`el`) の間で循環参照を生成しています。

最初の問題を除去するには、イベントリスナーを追加する時には常に、名前付きの関数を使うことです。

二つ目の問題を除去するには、メモリリークを避け、ガーベージコレクションをさせることです。

    function clickHandler() {
        this.style.backgroundColor = 'red';
    }

    function addHandler() {
        var el = document.getElementById('el');
        el.addEventListener('click', clickHandler);
    }

最後に: 無名関数パターンによるリスクは、Sencha クラスシステムのパラダイムに従うことで緩和できます。
フレームワークは、多くのスコープやバインドを管理します。
