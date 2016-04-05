#パフォーマンスに気を配りましょう

> *「パフォーマンスに問題があるときは、パフォーマンスだけが問題である」*

開発者なら皆パフォーマンスに気を遣っています -- しかし、開発者はまた、
まずその効率化が必要だということを証明せずにコードの細かな部分を最適化することに時間を費やすことはやめましょう。
例えば、JavaScriptライブラリやフレームワークで意味がある最適化も、アプリケーションコードではあまり効果がないかもしれません。

Ext JSやSencha Touchフレームワークを構築した経験と、顧客のアプリケーションへの我々の情報提供を考えると、Sencha はパフォーマンスを改善するためには、次のテクニックを折り紙付きの方法として確認します。

###ライブラリまたはフレームワークコード

  - [ループ](#Loops)
  - [Try / Catch](#Try_Catch)
  - [ページのリフロー](#Page_Reflow)
  - [関数ベースのイテレーション](#Function_Based_Iteration)

###アプリケーションコード

Yahooチームはそこに何も加えようがないほどの[卓越したパフォーマンス技術](https://developer.yahoo.com/performance/)をまとめました。

ですが、Senchaアプリケーションは、できる限り小さなサイズへコードを圧縮するために、ビルドプロセスの一部としてSencha Cmdを用いています。[Compiler-Friendly Code Guidelines](http://docs.sencha.com/cmd/5.x/cmd_compiler.html)
を参照してください。


## <a name="Loops" />ループ

ループの内側で、関数 (これに関しては他の再利用可能なもの) を宣言してはいけません。
これにより、処理時間、メモリ、ガーベージコレクションのサイクルを無駄遣いします。

    // 悪い
    var objectPool = [];

    for (i = 0; i < 10; ++i) {
        objectPool.push({
            foo: function () {}
        });
    }

    // 良い
    var objectPool = [];

    function bar() {}

    for (i = 0; i < 10; ++i) {
        objectPool.push({
            foo: bar
        });
    }

配列の長さの計算は、先頭で一度だけ行いその値を変数に格納します。
これにより、ループの各ステップで再計測することを防ぎます。

    // 悪い
    var i;
    for(i = 0; i < items.length; ++i){
        // some code
    }

    // 良い
    var i, len;
    for(i = 0, len = items.length; i < len; ++i){
        // some code
    }

    // 悪い
    var i;
    for(i = 0; i < items.getCount(); ++i){
        // some code
    }

    // 良い
    var i, len;
    for(i = 0, len = items.getCount(); i < len; ++i){
        // some code
    }

`for/in` 形式のループには、[パフォーマンスに悪影響がある](http://jsperf.com/for-in-vs-keys-vs-for)ことが知られていますので、使わないようにしましょう。

## <a name="Try_Catch" />Try / Catch

[パフォーマンスに重大な影響を引き起こすので](http://jsperf.com/try-catch-in-loop-cost/5)
できることなら `try/catch` ステートメントは使わないようにしましょう。

## <a name="Page_Reflow" />ページ リフロー

[不必要なページ リフロー](http://www.kellegous.com/j/2013/01/26/layout-performance/)
を起こすパターンを避けましょう。

リフローは、一部または全てのHTMLページのCSSレイアウトに影響を及ぼす変化が起こります。
エレメントのリフローは、それに引き続いて、すべての子要素と祖先要素に加え、DOM内でそれに続く全ての要素に以降のリフローを引き起こします。

ブラウザーは自動的にDOMやCSSの変化を経過を追いかけ、何かの位置や外観を変える必要があるときに「リフロー」を発行します。
やっかいなJavaScriptコードではブラウザーに強制的にCSSレイアウトを無効にすることができます -- たとえば、DOMから特定の結果（例えばoffsetHeight）を読むとブラウザー・スタイルのレイアウトの再計算を引き起こすことがあります。
ですから、開発者は、複数のページリフローが起こらないように、非常に注意しなければなりません。

    // 悪い
    elementA.className = "a-style";       // スタイルの変更はCSSレイアウトを無効にします
    var heightA = elementA.offsetHeight;  // オフセットを計算するためにリフローが起こります
    elementB.className = "b-style";       // CSSレイアウトが再び無効になります
    var heightB = elementB.offsetHeight;  // オフセットを計算するためにリフローが起こります

    // 良い
    elementA.className = "a-style";       // スタイルの変更はCSSレイアウトを無効にします
    elementB.className = "b-style";       // CSSレイアウトはすでに無効ですがまだリフローしません
    var heightA = elementA.offsetHeight;  // オフセットを計算するためにリフローが起こります
    var heightB = elementB.offsetHeight;  // CSSレイアウトが更新され、二度目のリフローは起こりません

## <a name="Function_Based_Iteration" />関数ベースのイテレーション

関数ベースのイテレーションは便利ですが、ループを使うよりも常に遅くなります。

    var myArray = [ 1, 2, 3 ];

    // jQuery
    $.each(myArray, function (index, value) {
        console.log(index + ": " + value);
    });

    // Ext JS 5
    Ext.each(myArray, function (value, index) {
        console.log(index + ": " + value);
    });

    // パフォーマンスは上です!
    var len = myArray.length,
        i, prop;
    for (i = 0; i < len; ++i) {
        console.log(i + ": " + myArray[i]);
    }

関数の宣言は、新しい実行コンテクスト（スコープチェーン）を生成します。
関数を呼び出して戻り値を取得すると、状態の維持と回復だけでなく、ガーベージコレクションも必要になります -- 一方、イテレーションでは単純に既存のコンテクストの他の場所にジャンプするだけです。
