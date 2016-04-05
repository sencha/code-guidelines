# 読みやすいJavaScriptを書く

「読みやすい」JavaScriptを書く重要性は、単純に誇張しているのではありません。チームの他のメンバーのためだけでなく、未来の自分のためにも、簡単に概要を掴めるコードを書くようにして下さい。

著書「*Facts and Fallacies of Software Engineering*（\*）」で、著者のRobert Glassは「既存の製品の理解」に開発者の時間の約3割が消費されている点を指摘しています。Glassのこの見解はソフトウェア保守サイクルのコンテキストで述べられていますが（「保守」という観点については後ほど改めて議論します）、当社は下記の項目について、首尾一貫して明確な優先順位を付けることをお勧めします。

  - [命名規約](#Naming_Conventions)
  - [コメントと文書化](#Comments_Documentation)
  - [オーバーライドの文書化](#Documenting_Overrides)
  - [インデントや空白](#Spacing_White_Space)
  - [行の長さ](#Line_Length)
  - [ブロックの長さ](#Block_Length)
  - [ファイルの長さ](#File_Length)

\*訳注 ... ソフトウエア開発 55の真実と10のウソ（日経BP出版センター）

## <a name="Naming_Conventions" />命名規約

> *"コンピュータ・サイエンスには2つの難しいものがあります。キャッシュの制御、命名規約、そして境界値のズレに関するエラー"*

ほとんどの人は人生において幾つかのものにしか名前を付けるという行為を行いません: ペット、子供、または個人的に特別な意味を持った何か。

一般的に、名前付けは一度決めると最後まで使われるため、難しい作業だと言えます（例えば、ペットの名前を付け直すことはめったにないでしょう）。しかし、ソフトウェアの開発において、開発者は平均的な人々よりも多くの名前付けをすることになります。

ソフトウェア開発者は恐らく年に数千の名前を付けることになるでしょう（変数、クラス、アプリケーション等）。それらの名前は、その名前が付けられた目的やコンセプトを示そうとするでしょう。しかしプログラムでは、覚えやすいように名付けられたものだけでなく、たくさんの名前を使うことになります。そのため名前は、他の人が数ヶ月から数年に渡って理解し、修正し、拡張できるよう、分かり易い状態に保たなければなりません。この考え方はしばしば「self-documenting code（自己文書化コード）」と呼ばれます。

それでは、命名規約を適用した方がよい箇所と、当社がどのようにそれらを扱っているかについて解説します。

### <a name="Namespaces" />名前空間、クラス、コンストラクタ

当社では、最上位の名前空間、クラス、コンストラクタに命名する際は、いつも「タイトル・ケース」を利用しています。

    // "MyClass" はコンストラクタ :: new MyClass();
    MyClass = function () {};

中間の名前空間は短くした方が良く、説明的な内容で小文字にします。

    // "Foo" は最上位の名前空間
    // "bar" は中間レベルの名前空間
    // "Baz" はクラス名
    Foo.bar.Baz = {};
    Ext.data.reader.Json = {};

### <a name="Functions" />関数

当社では関数を定義する際は、常に「キャメル・ケース」を利用します。また、クロージャでカプセル化されていないプライベートな関数を命名する際は、先頭にアンダースコア "\_" を付けることをお奨めします。

    // 関数式
    var sortSomeStuff = function () {};

    // 関数宣言
    function findSomething () {}

    // オブジェクトの関数についても同様の考え方を適用します
    var someObject = {
        objectMethod: function () {},

        _privateMethod: function () {}
    };

注: [関数](Preventing_JavaScript_Errors.md#Functions) で、関数式と関数宣言の使用についての詳しい情報を記載しています。

### <a name="Local_Vars" />ローカル変数とオブジェクトのプロパティ

ローカル変数を宣言する際は、常に「var」キーワードを使います。そうしないと、グローバル変数を作成することになり、汚染されたグローバルの変数を避ける必要が出てきます。この問題の詳細は [定数とグローバル変数](#Constants_Global_Vars) のセクションを参照下さい。

    // 悪い例
    foo = true;

    // 良い例
    var foo = true;

当社では、ローカル変数とオブジェクトのプロパティを作成する際は、常に「キャメルケース」を利用しています。また、オブジェクトにプライベートなプロパティを名付ける場合は、先頭にアンダースコア「\_」を付与することをお奨めします。

    // ローカル変数
    var fooBar = true;

    // オブジェクトのプロパティ
    var someObject = {
        someProperty: true,

        _privateProperty: true
    };

変数は、その目的や機能を明確にするため、意味のある名前を付ける必要があります。（もちろん完結であることも必要です）
一文字だけの名前は避けましょう。とはいえ、イテレータの変数はその例外にはなります。

    // 悪い例。短過ぎです。説明的ではないですし、数字の1とも誤解しそうです。
    var l = group.length;

    // 良くない例。変数名が必要以上に長い。
    var mainClassConfigVariableSectionOneRefreshInterval = 5000;

    // 良い例です。名前は完結で、ちゃんと意味も取れます。
    var len = group.length;

    // イテレータについては、例外。
    var i;
    for (i = 0; i < len; i++) {}

複数の変数を定義する際は、一つだけの「var」宣言を利用しましょう。その方が読み易いですから。当社では、一行ごとに一つの変数を定義する方法を推奨しています。
また、値を代入しない変数は最後にします。代入のない変数を複数宣言する場合については、一行で済ますこともありますが。

この方法は、現在のスコープで変数の初期状態がどうなっているのか視覚的に示すことができます。

    // 悪い例
    var foo = 1;
    var bar = 2;
    var baz;
    var fuz;

    // 良い例
    var foo = 1,
        bar = 2,
        baz, fuz;

### <a name="Constants_Global_Vars" />定数とグローバル変数

当社はグローバル変数と定数を宣言する際は、「CONSTANT_CASE」の利用を推奨します。そうすることで、この変数は特別なものであることを視覚的に明確にできます。

    // 悪い例
    userID = '12345';

    // 良い例
    USER_ID = '12345';

我々は企業アプリケーションでは、その代わりに適切な名前空間に配置されたクラスを使うことでのメリットがあると感じています。それはどこで値が定義されたかが常に明白だからです。

    // より良い
    MyApp.authentication.User = {
        id: '12345'
    };

### <a name="Special_Cases" />特別な場合

このほかの特別な場合も存在します -- 例えば、`this` を参照する名前です。

内部的な規則として、Sencha では、クロージャーの中で `this` への参照を保持する必要があるときに `me` という名前を使います。
全員が賛成しているわけではありませんが -- クリスチャン ジョナサン [a notable example](https://gist.github.com/cjohansen/4135065) -- より素晴らしい点は、コードベース全体で一貫してこれらの特別な場合を管理することです。

    Person.logger = function () {
        var me = this; // "me" will be used consistently

        return function () {
            console.log(me);
        };
    };

もう一つの重要な点は、this はキーワードであり圧縮できないということです。 Sencha フレームワークでは、我々は「4のルール」を守ります。 特定のスコープ内で、this が 4回以上参照されたら、this をローカル変数 me でキャッシュします。そうするとミニファイされるソースをより小さくできます。

    // 悪い
    function foo () {
        this.x = 1;
        this.y = 2;
        this.z = 3;
        this.u = 4;
    }

    // 良い
    function foo () {
        var me = this;
        me.x = 1;
        me.y = 2;
        me.z = 3;
        me.u = 4;
    }

    // ミニファイの出力の比較
    function f(){this.x=1;this.y=2;this.z=3}
    function f(){var e=this;e.x=1;e.y=2;e.z=3}

    function f(){this.x=1;this.y=2;this.z=3;this.u=4;}
    function f(){var e=this;e.x=1;e.y=2;e.z=3;e.u=4;} // 4 でより短くなります

    function f(){this.x=1;this.y=2;this.z=3;this.u=4;this.v=5;}
    function f(){var e=this;e.x=1;e.y=2;e.z=3;e.u=4;e.v=5;}

### <a name="Reserved_Words" />予約語

予約語をキーとして使っていけません。 Internet Explorer の古いバージョンで動作しないからです。 予約語の代わりに分かりやすい同義語を使います。

    // 悪い
    var model = {
        name: 'Foo',
        private: true // reserved word!
    };

    // 良い
    var model = {
        name: 'Foo',
        hidden: true
    };

注: 「分かりやすい同義語」は実際の単語にします。

    // 悪い
    var car = {
        class: 'Ford' // 予約語
    };

    // 悪い
    var car = {
        klass: 'Ford' // お願い。二度とこんなことしないで
    };

    // 良い
    var car = {
        brand: 'Ford'
    };

## <a name="Comments_Documentation" />コメントと文書化

一般的に言って、良いコードは自己説明的だとされています。 しかしながら、コメントは読みやすいコードにするために二つの重要な役割を果たします。 文書化と意図 (インラインコメントによる) です。

### <a name="Documentation" />文書化

システム全体の文書化は、大きなコードベースを開発する際にきわめて重要です。 [JSDuck](https://github.com/senchalabs/jsduck) のようなツールを使うと、コードベースの API リファレンスを簡単に作ることができます。

[Sencha](http://docs.sencha.com/extjs/5.0/apidocs/) は、内部的に JSDuck を使っています。 JSDuck は、 JavaDoc スタイルのブロックコメントを追跡します。 [JSDuck wiki](https://github.com/senchalabs/jsduck/wiki) で、詳細をご覧になれます。

    /**
     * @class MyApp.foo.Bar
     */
    MyApp.foo.Bar = function () {
        var baz = true;

        return {
            /**
             * @method
             */
            utilityMethod: function () {
                return baz;
            }
        };
    };

### <a name="Inline_Comments" />インライン コメント

多くの開発者が、コードは「自己文書化」であるべきと感じていて、それによってインライン・コメントはつけるべきでないと感じています。 Sencha は、その思考法の頑固さに必ずしも同意できません。 我々は、コードの意図または目的が完全に明白でないときには、常にコメントをするものだと思っています。しかし、コードそのものは論理的に追いかける際に十分明白であるようにしましょう。

    // 多くの場合、コントローラIDは、その名前と同じです。
    // しかし、コントローラに手動でIDが与えられると、そのように
    // コレクションの中のキーになります。
    // ですから、それを見つけないと、我々は既存のコントローラをループして、
    // クラス名を使ってそれを見つけようとします。
    if (!controller) {
       all = controllers.items;
       for (i = 0, len = all.length; i < len; ++i) {
           cls = all[i];
           className = cls.getModuleClassName();
           if (className && className === name) {
               controller = cls;
               break;
           }
       }
    }

正規表現式は、本質的に混乱する記法ですから、常にコメントで説明を書きましょう。

    // ローマ数字の入力に一致する
    var romanNums = /^M{0,4}(CM|CD|D?C{0,3})(XC|XL|L?X{0,3})(IX|IV|V?I{0,3})$/;

コードブロック全体をコメントアウトするのは避けましょう。 それは、用途がなくコードを肥大化させるだけですから。

    // どうして次のコードが製品版に残っているの？
    items    : [
        //{
        //    xtype        : 'booleancolumn',
        //    width        : 5,
        //    resizable    : false,
        //    defaultWidth : 5,
        //    sortable     : false,
        //    dataIndex    : 'isOwn',
        //    groupable    : false,
        //    hideable     : false,
        //    lockable     : false,
        //    tdCls        : 'indicator',
        //    falseText    : ' ',
        //    trueText     : ' '
        //},
        //{
        //    xtype     : 'gridcolumn',
        //    dataIndex : 'key',
        //    text      : 'Binding Key',
        //    flex      : 1
        //},
        {
            xtype     : 'templatecolumn',
            dataIndex : 'boundTo',
            text      : 'Bound To',
            flex      : 1,
            tpl       : '\\{{boundTo}\\}'
        },
        {
            xtype     : 'gridcolumn',
            dataIndex : 'value',
            text      : 'Value',
            flex      : 1,
            renderer  : function (value, metaData, record, rowIndex, colIndex, store, view) {
                var v = value;

                if (value === null) {
                    v = 'null';
                }

                if (record.data.text === 'undefined') {
                    v = 'undefined';
                }

                return '<span class="highlight ' + record.get('type') + ' ' + v + '">' + v + '</span>';
            }
        }
    ]

## <a name="Documenting_Overrides" />オーバーライドの文書化

デフォルトをオーバーライドする必要があるか、機能を継承した場合、その変更を完全に明白にするために、インライン コメントとブロック コメントを積極的に活用しましょう。

    // EXTJS-12345 のバグのオーバーライド
    Ext.define('MyApp.override.CustomNumberField', {
        override : 'Ext.form.field.Number',

        initComponent: function () {
            var me = this,
                allowed;

            me.callParent();

            me.setMinValue(me.minValue);
            me.setMaxValue(me.maxValue);

            // 設定されたオプションをベースにマスキング／ストリッピングする正規表現を構築
            if (me.disableKeyFilter !== true) {
                allowed = me.baseChars + '';
                if (me.allowDecimals) {
                    //OVERRIDE THIS LINE...
                    //allowed += me.decimalSeparator;
                    allowed += ',.';
                }
            }
        }
    });

この文書化は、アップグレード処理の際に便利です。 たとえば、バグ EXTJS-12345 は最新版でフィックスされたら、このオーバーライドは完全に取り除くことができます。

## <a name="Spacing_White_Space" />インデントや空白

多くの開発者には、スペースをあける時のタブとスペースの話題に関する断固たる意見があるものです。 タブのサイズは予測できないので、Sencha は自身のコードでは 4つのスペースを利用することにしています。 読みやすいコードを保証することができる唯一の方法は、スペースの利用を実施することです。 最終的な目的は一貫性があることだけなので、どちらを選ぼうともそれらを交ぜてはいけません。

    // 悪い
    function doSomething (isTrue) {
     // < 1つのスペース
     if (isTrue) {
        // <<< 3 つのスペース
    }// もう混乱し始めてます
    }

    // 良い
    function doSomething (isTrue) {
        // <<<< 4 つのスペース
        if (isTrue) {
            // <<<< 4 ここでも 4つのスペース
        }
    }

一方、Senchaも、読みやすいコードを作るのに必要なホワイトスペースを使うことを推奨します。

### 一貫性の利点

Senchaが推奨するひとつの技法は、関数宣言の括弧の前にはスペースを空け、実行する時にはスペースを空けないことです。
そうすることで、テキスト検索で関数宣言とは分けて、関数を実行している箇所を簡単に探せるようになります。

下記はその例です:

    function foo () { // "foo" と "(" の間にスペースがあります
        return 42;
    }

    var x = foo();  // "foo" と "(" の間にスペースがありません

このようにすることで、"foo(" と検索すると、'foo'関数を実行している箇所だけを見つけることができます。

Senchaが推奨するその他の技法は、オブジェクトのプロパティ名と `:` の間でスペースを空けないことです。
（これもまた、テキスト検索で利点があります）

下記はその例です:

    Ext.define('Ext.panel.Panel', {
        collapse: function () ...
    });

"collapse:" を検索すると、呼び出し以外の`collapse`の実装を探すことができます。逆に、"collapse(" で呼び出しを探せます。

## <a name="Line_Length" />行の長さ

一行あたりの文字数制限については、誰もが同意するというわけではありませんが、 Sencha では一般的に行の幅を制限しようとしています。 この制限は任意で（例えば80または100文字）厳しく押しつけることはしませんが、目的は開発者が水平スクロールする量を減らすのが目的です。

誰でも線につき性格のために特定の制限に同意するというわけではない、しかし、Senchaは一般的に行の幅を制限しようとしています。 この制限は任意で（例えば80または100文字）、厳しく実施されることができない、しかし、ゴールは水平スクロールの量を開発者のために減らすことになっている。

明確な制限より長いひもは、ストリング連結を使っている複数の線の向こうに書かれなければならない。 決められた制限よりも長い文字列は、文字列連結を使って複数行にわたって記述します。

## <a name="Block_Length" />メソッドやブロックの長さ

コードブロックがどれくらいの長さになったら、機能をより小さなユーティリティ メソッドに分割することを考えますか？

良い経験則は、メソッドやコードブロックの長さを (例えば50または100行に) 制限することです。それによりそれほど大きくならずに済みます。 短いメソッドはテストしやすく、開発者がすぐに理解することができます。

## <a name="File_Length" />ファイルの長さ

ファイルの行数がどれくらいになったら機能をミックスインやモジュールに分割することを考えますか？

メソッド／ブロックの長さと同じように、コメントもファイルの長さに影響します。 仮想クラスは、インターフェースや基本ラインを定義するので、通常より長くなりやすくなります。 それでもやはり、任意のファイル長 (500または1000行) を定めると、クラスをリファクタリングする必要があるかどうかの兆候を把握できます。
