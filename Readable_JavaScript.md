# 読みやすいJavaScriptを書く

「読みやすい」JavaScriptを書く重要性は、単純に誇張しているのではありません。チームの他のメンバーのためだけでなく、未来の自分のためにも、簡単に概要を掴めるコードを書くようにして下さい。

著書「*Facts and Fallacies of Software Engineering*（\*）」で、著者のRobert Glassは「既存の製品の理解」に開発者の時間の約3割が消費されている点を指摘しています。Glassのこの見解はソフトウェア保守サイクルのコンテキストで述べられていますが（「保守」という観点については後ほど改めて議論します）、当社は下記の項目について、首尾一貫して明確な優先順位を付けることをお勧めします。

  - [命名規約](#Naming_Conventions)
  - [コメントや文書化](#Comments_Documentation)
  - [オーバーライドの文書化](#Documenting_Overrides)
  - [空白スペース](#Spacing_White_Space)
  - [行の長さ](#Line_Length)
  - [ブロックの長さ](#Block_Length)
  - [ファイルの大きさ](#File_Length)

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

とは言いつつも、当社はグローバル変数も定数も全く利用しないようにしています。特にエンタープライズ・アプリケーションでは、名前空間を利用したクラスを利用した方が

Having said that, Sencha prefers to avoid global variables and constants altogether. We feel that enterprise
applications benefit from using properly-namespaced classes instead because it's always clear where a value
has been defined.

    // better
    MyApp.authentication.User = {
        id: '12345'
    };

### <a name="Special_Cases" />Special Cases

Other special cases also exist -- for example, naming references to `this`.

As an internal convention, Sencha uses the name `me` when there is a need to capture a reference to `this` within
a closure. Not everyone agrees -- Christian Johansen is [a notable example](https://gist.github.com/cjohansen/4135065) --
but the greater point is to manage these special cases consistently throughout your codebase.

    Person.logger = function () {
        var me = this; // "me" will be used consistently

        return function () {
            console.log(me);
        };
    };

Another important thing to note is that `this` is a keyword and can't be compressed. In the Sencha frameworks,
we abide by the rule of four: if a given scope references `this` four or more times,
cache `this` using the local variable `me` as it will make the minified source smaller.

    // bad
    function foo () {
        this.x = 1;
        this.y = 2;
        this.z = 3;
        this.u = 4;
    }

    // good
    function foo () {
        var me = this;
        me.x = 1;
        me.y = 2;
        me.z = 3;
        me.u = 4;
    }

    // comparison of minified output
    function f(){this.x=1;this.y=2;this.z=3}
    function f(){var e=this;e.x=1;e.y=2;e.z=3}

    function f(){this.x=1;this.y=2;this.z=3;this.u=4;}
    function f(){var e=this;e.x=1;e.y=2;e.z=3;e.u=4;} // 4 is now shorter!

    function f(){this.x=1;this.y=2;this.z=3;this.u=4;this.v=5;}
    function f(){var e=this;e.x=1;e.y=2;e.z=3;e.u=4;e.v=5;}

### <a name="Reserved_Words" />Reserved Words

Don't use reserved words as keys because they break things in older versions of Internet Explorer. Use readable synonyms
in place of reserved words instead.

    // bad
    var model = {
        name: 'Foo',
        private: true // reserved word!
    };

    // good
    var model = {
        name: 'Foo',
        hidden: true
    };


Note: "readable synonyms" must actually be words.

    // bad
    var car = {
        class: 'Ford' // reserved word!
    };

    // bad
    var car = {
        klass: 'Ford' // PLEASE don't ever do this!
    };

    // good
    var car = {
        brand: 'Ford'
    };

## <a name="Comments_Documentation" />Comments and Documentation

Generally speaking, good code is supposed to be self-explanatory. However comments play two vital roles in promoting
readable code: documentation, and intent (via inline comments).

### <a name="Documentation" />Documentation

System-wide documentation is vital to developing large codebases. Using tools like
[JSDuck](https://github.com/senchalabs/jsduck) it is easy to build an API
reference for your codebase, making it significantly easier for your team (and others) to digest.

[Sencha](http://docs.sencha.com/extjs/5.0/apidocs/) uses JSDuck internally, which follows the JavaDoc style for block
comments. See the [JSDuck wiki](https://github.com/senchalabs/jsduck/wiki) for more information.

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

### <a name="Inline_Comments" />Inline Comments

Many developers feel that code ought to be "self-documenting" and therefore inline comments are to be avoided.
Sencha doesn't necessarily agree with the rigidness of that mindset; we believe that comments should always be
added when the intent or purpose of any code isn't completely explicit, but the code itself ought to be clear
enough to follow logically.

    // In a majority of cases, the controller ID will be the same as the name.
    // However, when a controller is manually given an ID, it will be keyed
    // in the collection that way. So if we don't find it, we attempt to loop
    // over the existing controllers and find it by classname
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

Regular expressions should also always be explained with a comment because of their inherently confusing syntax.

    // match Roman Number input
    var romanNums = /^M{0,4}(CM|CD|D?C{0,3})(XC|XL|L?X{0,3})(IX|IV|V?I{0,3})$/;

Commenting out entire blocks of code should be generally avoided because they serve no purpose and create bloated code.

    // Why leave the following code in production?
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

## <a name="Documenting_Overrides" />Documenting Overrides

In cases where you need to override default or inherited functionality, both inline and block comments are
actively encouraged so that the changes are perfectly clear.

    // OVERRIDE for bug EXTJS-12345
    Ext.define('MyApp.override.CustomNumberField', {
        override : 'Ext.form.field.Number',

        initComponent: function () {
            var me = this,
                allowed;

            me.callParent();

            me.setMinValue(me.minValue);
            me.setMaxValue(me.maxValue);

            // Build regexes for masking and stripping based on the configured options
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

This documentation will often come in handy during an upgrade process. For example, the bug EXTJS-12345 might have
been fixed in the latest version -- so this override could be removed completely.

## <a name="Spacing_White_Space" />Spacing and White Space

Many developers have strong opinions on the topic of tabs-vs-spaces for spacing. Sencha advocates the use of
four spaces in our own code because tab sizes are unpredictable; the only way we can guarantee readable code is
to enforce the use of spaces. Ultimately the goal is just to have consistency, so whatever your choice don't mix them!

    // bad
    function doSomething (isTrue) {
     // < 1 space in
     if (isTrue) {
        // <<< 3 spaces in?
    }// now you're just being confusing...
    }

    // good
    function doSomething (isTrue) {
        // <<<< 4 spaces in!
        if (isTrue) {
            // <<<< 4 spaces in again!
        }
    }

On the other hand, Sencha also advocates for using as much white space as necessary to make your code easier to read.

### Benefits of Consistency

One technique Sencha recommends is to maintain spaces before the parentheses of function declarations but
not for function calls. This allows text searches to more easily find calls to functions separately from their
declartions.

For example:

    function foo () { // Note the space between "foo" and "("
        return 42;
    }

    var x = foo();  // Note no space between "foo" and "("

Now searches for "foo(" will find only the calls to `foo`.

Another technique Sencha recommends is to place no space between the name of an object property and the `:`
(again for benefits in text search).

For example:

    Ext.define('Ext.panel.Panel', {
        collapse: function () ...
    });

Searches for "collapse:" will find implementations of `collapse` and not invocations. Conversely for
"collapse(".

## <a name="Line_Length" />Line Length

Not everyone agrees with the specific limit for characters-per-line, but Sencha generally tries to limit line length.
This limit can be arbitrary (e.g. 80 or 100 characters) and not rigidly enforced, but the goal is to reduce the amount
of horizontal scrolling for the developer.

Strings longer than the decided limit should be written across multiple lines using string concatenation.

## <a name="Block_Length" />Method and Block Length

How long can a method or code block get before you consider breaking functionality into smaller utility methods?

A good rule-of-thumb is to limit the length of method and code blocks (e.g. 50 or 100 lines) so that they are not
trying to do too much. Shorter methods are easier to test, and smaller sections of code are more quickly
comprehended by developers.

## <a name="File_Length" />File Length

How long should a file be before you consider breaking functionality into mixins, modules or other utility classes?

As with method/block length, comments can easily impact the length of a file. Abstract classes might also be longer
than usual because they define interfaces and baseline functionality. Nevertheless, defining an arbitrary file
length (e.g. 500 or 1000 lines) might give you an indication of whether-or-not a class might need to be refactored.
