#Writing Maintainable JavaScript

In his book *Facts and Fallacies of Software Engineering*, Robert Glass discusses how as much as 80% of the total cost 
of software over its lifetime comes from maintenance tasks. This isn't just fixing bugs; in most cases, "maintenance" 
actually involves adding new functionality to existing applications. 

The implications here are clear: if developers spend the vast majority of their time maintaining applications, 
then the code should always be written to make that process easier.

Sencha recommends prioritizing the following points to help reduce the long-term friction involved in maintaining 
a large codebase:

  - [One Class Per File](#One_Class_Per_File)
  - [Declaring Variables](#Declaring_Variables)
  - [Array Literals](#Array_Literals)
  - [Debugging Statements](#Debugging_Statements)
  - [Ternary Operators](#Ternary_Operators)
  - [Regular Expressions](#Regular_Expressions)
  - [Strings](#Strings)
  - [Method Chains](#Method_Chains)

## <a name="One_Class_Per_File" />One Class Per File

Sencha strongly encourages the practice of coding only one class per file. Doing so makes maintaining and debugging 
applications easier, because the developer knows exactly where to find their code.

In addition, Sencha recommends using a consistent approach to naming and organizing your files. The file name should 
match the class name defined within, and the physical file location should match the class' namespace.

For example, the class `Foo.bar.Baz` might be defined in `Baz.js`. Per the full namespace, the file might be located 
at `/{www}/Foo/bar/Baz.js`.
 
## <a name="Declaring_Variables" />Declaring Variables

Sencha typically follows Crockford's advice to declare variables at the top of their scope because:

1. finding variable declarations becomes easier
2. it makes the scope of the variables clear (as JavaScript does not have block scope)
3. variable declarations are [hoisted](http://www.adequatelygood.com/JavaScript-Scoping-and-Hoisting.html) to the top of their scope anyways

```
function foo() {
    var bar = 1; // good
    
    if (true) {
        var baz = 2; // bad; "baz" should be declared above
    }
}
```
    
### <a name="Declaring_Multiple_Variables" />Declaring Multiple Variables

Use one `var` declaration when creating multiple variables because it is easier to read. Sencha recommends declaring 
each variable assignment on a new line; declare unassigned variables last, and these can be on the same line. 
This provides a visual cue to the person reading your code about the initial state of the variables within the current scope.
    
    // bad
    var foo = 1;
    var bar = 2;
    var baz;
    var fuz;
    
    // good
    var foo = 1,
        bar = 2,
        baz, fuz;

### <a name="Declaring_Object_Properties" />Object Properties

When [accessing object properties](http://stackoverflow.com/questions/4968406/javascript-property-access-dot-notation-vs-brackets), 
Sencha prefers to use dot notation rather than bracket notation because it is typically easier to read and is 
more easily compressed. 

    var myObject = {
        foo : 'bar'
    };
    
    // works, but not preferred
    myObject['foo'];
    
    // preferred
    myObject.foo;
    
It should be noted that bracket notation does allow you to declare properties with characters that can't be used 
in dot notation.

    myObject['~foo'] = true; // works just fine
    myObject.~foo = true; // will throw an error

While Crockford and others recommend against bracket notation, the takeaway here is to be as consistent in your 
code as possible.

Finally, bracket notation can be very useful when using a variable to access object properties:

    var prop = 'foo';
    alert(myObject[prop]);

### <a name="Default_Values" />Default Values

It is also considered good practice to define default values in many cases.

For simple values, you can use `||` to define default values. If the left-hand value is falsy then the right-hand 
value will be used.

    function init(config) {
        this.hidden = config.hidden || false;
    }

In more complex scenarios, it may make sense to use conditional expressions or utility methods to perform the validations:

    function init(config) {
        // using a conditional
        this.total = (config.count > 10) ? config.count : 10;
    
        // using a utility method 
        // to copy all properties from config but also provide default values
        Ext.apply(this, config, {
            hidden : false
        });
    }
    
## <a name="Array_Literals" />Array Literals

In most cases, you should create new arrays by using the square bracketed notation:

    // bad
    var stuff = new Array();
    
    // good
    var stuff = [];

Use arrays when the member names will be sequential integers; do not create associative arrays (i.e. dictionaries) 
where the member names are arbitrary strings or names -- create an object for those cases instead.

    // bad
    var stuff = [];
    stuff['foo'] = 'bar';
    stuff[0]; // undefined!
    
    // good
    var stuff = [ 'bar' ];
    stuff[0]; // 'bar'

Note: some cases exist where declaring a fixed-dimension array for 
[performance reasons](http://www.slideshare.net/doris1/oscon-developing-high-performance-websites-and-modern-apps-with-javascript-and-html5faster-siteandappformeetupforoscon) 
might make sense. In those cases it's fine to use the `new Array(length)` notation instead, but there are many 
implications to doing so.

## <a name="Debugging_Statements" />Debugging Statements

Debugging statements like `console.log()` and `debugger` should never be shipped into standard production environments. 
A better approach is to bake these statements into a service that can easily be disabled in production.

    // bad
    function foo() {
        console.log('inside the foo() method');
        return true;
    }
    
    // good
    function foo() {
        // where logger() only outputs statements in development
        MyApp.util.logger('inside the foo() method');
        return true;
    }

Sencha applications utilizing Sencha Cmd also have the ability to mark sections of code as being run only in 
development; Sencha Cmd will then strip out these sections during the optimization process for production builds.

    function foo() {
        // <debug>
        console.log('inside the foo() method');
        // </debug>
        return true;
    }
    
## <a name="Ternary_Operators" />Ternary Operators

Ternary operators are fine for clear-cut conditionals, but unacceptable for confusing choices. 

    // bad
    var value = a && b ? 11 : a ? 10 : b ? 1 : 0;
    
    // good
    var value = isSimple ? 11 : 1;

Ternary expressions should never be nested because they just add to the confusion. 

**Rule of thumb:** If you are unsure whether-or-not you should be using a ternary expression, always default to 
using an `if/else` statement instead.

## <a name="Regular_Expressions" />Regular Expressions

> *"Have a problem and you think RegEx is the solution? Now you have 2 problems."*

Regular expressions are very powerful but extremely confusing -- therefore you need to do everything in your power 
to keep them maintainable.

Don't inline regular expressions; store them in a variable to improve readability and 
[performance](http://jsperf.com/caching-regex-objects/15), and always add comments to explain their purpose.

    // bad
    var hasNumbers = /\d+/.test(value);
    
    // good
    var numberTest = /\d+/, //contains one or more numbers
        hasNumbers = numberTest.test(value);

## <a name="Strings" />Strings

Pick a consistent approach to defining strings: single or double quotes. This will make the code easier to read 
and easier to search.

    // bad
    var stringA = 'A', // single quotes
        stringB = "B"; // double quotes
    
    // good
    var stringA = 'A',
        stringB = 'B';

Sometimes a string must be broken across several lines if it’s too long (e.g. >80 characters)  to simply improve 
readability.

    // bad
    var stringA = 'Who lives in a pineapple under the sea? Sponge Bob Square Pants! Absorbent and yellow and porous is he!';
    
    // good
    var stringA = 'Who lives in a pineapple under the sea? ' + 
        'Sponge Bob Square Pants! ' +
        'Absorbent and yellow and porous is he!';
    
    // or with Array.join()
    var stringA = [
        'Who lives in a pineapple under the sea?',
        'Sponge Bob Square Pants!',
        'Absorbent and yellow and porous is he!'
        ].join(' ');

## <a name="Method_Chains" />Method Chains

As Crockford discusses in *JavaScript: The Good Parts*, method chains (or "cascades" as he calls them) allow us 
to call many methods on the same object in sequence during a single statement. These methods simply return `this` to 
allow the chain to continue:

    //pseudocode
    getElement('myDiv').width(100).height(100).color('red') // etc.

Sencha urges some caution with this pattern as the lengthy method chains can turn into spaghetti code over time 
in large codebases. In many cases, abstracting much of this behavior into a utility method 
(then re-used in multiple places) will improve the long-term maintainability of your code.
