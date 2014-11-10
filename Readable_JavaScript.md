#Writing Readable JavaScript

The importance of writing "readable" JavaScript simply cannot be overstated. Be kind to the others on your team, 
and your future self, by writing code that is easy to digest.

In his book *Facts and Fallacies of Software Engineering*, Robert Glass discusses how simply 
"understanding the existing product" consumes roughly 30% of a developer's time. Glass frames this point within the 
context of the software maintenance cycle -- and while we will move our discussion to maintainability soon, 
Sencha recommends prioritizing the following points to make your codebase clear and coherent:

  - [Naming Conventions](#Naming_Conventions)
  - [Comments and Documentation](#Comments_Documentation)
  - [Documenting Overrides](#Documenting_Overrides)
  - [Spacing and White Space](#Spacing_White_Space)
  - [Line Length](#Line_Length)
  - [Block Length](#Block_Length)
  - [File Length](#File_Length)

## <a name="Naming_Conventions" />Naming Conventions

> *"There are only two hard things in Computer Science: cache invalidation, naming things, and off-by-one errors."*

Many people only name a few things in their life: a pet, a child, or something particularly personal and meaningful. 
In general, naming things is difficult because there is a perceived finality to the process 
(e.g. you rarely re-name a pet) -- but in software development, one typically has to name things far more often 
than the average person.

Software developers might name thousands of new things per year: variables, classes, applications, etc. Each name 
attempts to capture the essence or purpose of the concept being named -- but because programs use more names than 
can be reasonably remembered, the names must be conceived consistently in order to help others understand, fix, 
and extend that same code months or years later. This idea is often referred to as "self-documenting code".

Let's examine the areas in which naming conventions should be applied and how Sencha handles each case.

### <a name="Namespaces" />Namespaces, Classes, and Constructors

Sencha always uses *TitleCase* when creating top-level namespaces, classes, and constructors.

    // "MyClass" is a constructor :: new MyClass();
    MyClass = function() {};

Intermediate namespaces should be short, descriptive and lowercase.

    // "Foo" as the top-level namespace
    // "bar" as an intermediate-level namespace
    // "Baz" as the class name
    Foo.bar.Baz = {};
    Ext.data.reader.Json = {};

### <a name="Functions" />Functions

Sencha always uses *camelCase* when creating functions. We also recommend the use a leading underscore "_" when 
naming private functions and methods that are not encapsulated by a closure.

    // function expression
    var sortSomeStuff = function() {};
    
    // function declaration
    function findSomething() {}
    
    // the same concept applies to object functions
    var someObject = {
        objectMethod : function() {},
    
        _privateMethod : function() {}
    };

Note: more information about using function expressions vs. function declarations can be found in 
section [Functions](Preventing_JavaScript_Errors.md#Functions).

### <a name="Local_Vars" />Local Variables and Object Properties

Always use *var* to declare local variables -- not doing so will result in the creation of global variables, and we want 
to avoid polluting the global namespace. See the [Constants and Global Variables](#Constants_Global_Vars) section below 
for more details.

    // bad
    foo = true;
    
    // good
    var foo = true;

Sencha always uses *camelCase* when creating local variables and object properties. We also recommend the use a 
leading underscore "_" when naming private properties.

    // local variable
    var fooBar = true;
    
    // object property
    var someObject = {
        someProperty : true,
    
        _privateProperty : true
    };

Variables should be given meaningful names, so that the intended purpose and functionality of the variables is 
clear (while also concise). Avoid single letter names; the lone exception to this rule would be an iterator.

    // bad, easy to confuse with number 1
    var l = group.length;
    
    // good
    var len = group.length;
    
    // iterators are an exception
    var i;
    for (i = 0; i < len; i++) {}

Use one *var* declaration when creating multiple variables because it is easier to read. Sencha recommends 
declaring each variable assignment on a new line; declare unassigned variables last, though these can be on the 
same line. 

This helps to provide a visual cue to the person reading your code about the initial state of the variables 
within the current scope.

    // bad
    var foo = 1;
    var bar = 2;
    var baz;
    var fuz;
    
    // good
    var foo = 1,
        bar = 2,
        baz, fuz;
        
### <a name="Constants_Global_Vars" />Constants and Global Variables

Sencha recommends using *CONSTANT_CASE* when creating global variables because of the clear visual 
indication that the variable is special.

    // bad
    userID = '12345';
    
    // good
    USER_ID = '12345';

Having said that, Sencha prefers to avoid global variables and constants altogether. We feel that enterprise 
applications benefit from using properly-namespaced classes instead because it's always clear where a value 
has been defined.

    // better
    MyApp.authentication.User = {
        id : '12345'
    };

### <a name="Special_Cases" />Special Cases

Other special cases also exist -- for example, naming references to *this*. 

As an internal convention, Sencha uses the name *me* when there is a need to capture a reference to *this* within 
a closure. Not everyone agrees -- Christian Johansen is [a notable example](https://gist.github.com/cjohansen/4135065) -- 
but the greater point is to manage these special cases consistently throughout your codebase.

    Person.logger = function() {
        var me = this; // “me” will be used consistently
    
    
        return function() {
            console.log(me);
        };
    };

In the Sencha frameworks, we abide by the rule of four: if a given scope references *this* four or more times, 
cache *this* using the local variable *me* as it will make the minified source smaller.

    // bad
    function foo() {
        this.x = 1;
        this.y = 2;
        this.z = 3;
        this.u = 4;
    }
     
    // good
    function foo() {
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

Don't use reserved words as keys as they break things in older versions of Internet Explorer. Use readable synonyms 
in place of reserved words instead.

    // bad
    var model = {
        name    : 'Foo',
        private : true // reserved word!
    };
    
    // good
    var model = {
        name   : 'Foo',
        hidden : true
    };


Note: "readable synonyms" must actually be words.
    
    // bad
    var car = {
        class : 'Ford' // reserved word!
    };
    
    // bad
    var car = {
        klass : 'Ford' // PLEASE don't ever do this!
    };
    
    // good
    var car = {
        brand : 'Ford'
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
    MyApp.foo.Bar = function() {
        var baz = true;
    
        return {
            /**
             * @method
             */
            utilityMethod : function() {
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

## <a name="Documenting_Overrides" />Documenting Overrides

In cases where you need to override default or inherited functionality, both inline and block comments are 
actively encouraged so that the changes are perfectly clear.

    // OVERRIDE for bug EXTJS-12345
    Ext.define('MyApp.override.CustomNumberField', {
        override : 'Ext.form.field.Number',
     
        initComponent: function() {
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
    function doSomething(isTrue) {
     // < 1 space in
     if (isTrue) {
        // <<< 3 spaces in?
    }// now you're just being confusing...
    }
    
    // good
    function doSomething(isTrue) {
        // <<<< 4 spaces in!
        if (isTrue) {
            // <<<< 4 spaces in again!
        }
    }

On the other hand, Sencha also advocates for using as much white space as necessary to make your code easier to read.

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