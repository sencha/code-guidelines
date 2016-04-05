#Preventing JavaScript Errors

[Defensive Programming](http://en.wikipedia.org/wiki/Defensive_programming) is an approach to building software which 
strives to reduce the likelihood of common problems. Because JavaScript syntax contains many gotchas and edge cases, 
adopting a defensive programming mindset can help to prevent many runtime errors in large codebases.

 - [Removing Lint](#Removing_Lint)
 - [Using Semicolons](#Semicolons)
 - [Trailing Commas](#Trailing_Commas)
 - [Brackets and Blocks](#Brackets_Blocks)
 - [Equality](#Equality)
 - [Prototypes](#Prototypes)
 - [Functions](#Functions)

## <a name="Removing_Lint" />Removing Lint

JavaScript "lint" tools read your source code to help identify common mistakes -- things as subjective as multiple 
`var` statements in a function, or as objective as flagrant syntax errors. These mistakes are considered "lint" 
and should be removed or restructured.

Using tools such as JSLint, ESLint, JSHint, etc is always recommended during development. Many popular IDEs even have 
direct integration with these tools.

However, you should also consider making these tools part of your automated testing or build processes. IDE integration 
will only warn the developer about errors - it does not force them to correct problems, and it will not prevent the 
bad code from being committed to your repo.

## <a name="Semicolons" />Using Semicolons

Automatic Semicolon Insertion (ASI) is not a feature. [Don't rely on it.](http://benalman.com/news/2013/01/advice-javascript-semicolon-haters/) 

Crockford recommends putting a semicolon at the end of every simple statement because JavaScript allows any expression 
to be used as a statement, which can mask some tricky errors. What is worse, these errors may not be apparent in the development process but could appear in the minified production code instead, and that would make them even harder to debug.

    // bad
    var a = obj
    [a].forEach(logProp);
    // Because a semicolon isn't used that code behaves like this:
    // var a = obj[a].forEach(logProp)

    // good
    var a = obj;
    [a].forEach(logProp); // this works fine
    
    
    // bad
    var example = function () {
        // because of the line break, ASI returns "undefined"
        return
        {
            foo: 123
        };
    };
    
    // good
    var example = function () {
        return {
            foo: 123
        };
    };
    
## <a name="Trailing_Commas" />Trailing Commas

Trailing commas have caused more headaches in JavaScript development over the years than perhaps anything else.

    var myObject = {
        foo : 1,
        bar : 2, // trailing comma
    };
    
    var myArray = [ 1, 2, 3, ]; // trailing comma

Although the current ECMAScript 5 specification allows for trailing commas in Object and Array literals, older 
browsers (particularly IE <9) encounter unexpected behavior: trailing commas in object literals would throw runtime 
errors, while trailing commas in array literals would result in an extra `undefined` item at the end of the array. This would not only return inaccurate results for Array.length but also may cause runtime exceptions if your code assumes that array items are valid objects.

Therefore, as a best practice, Sencha discourages developers from using them.

On a related note, some developers prefer to use leading commas to avoid this problem. Sencha doesn't feel that 
solution adequately solves the issue, and furthermore we believe it reduces the readability of the code.

## <a name="Brackets_Blocks" />Brackets and Blocks

Always use brackets when creating code blocks of any kind. Every block, even if it is only one line, needs to have 
its own curly braces in order to avoid confusion and prevent the possibility of hard to track bugs that may result from the combination of braceless block with ASI which could produce some very unexpected code.

    // bad
    if (foobar) doSomething();
     
    // good
    if (foobar) {
        doSomething();
    }

In many cases, the use of [guard clauses](http://refactoring.com/catalog/replaceNestedConditionalWithGuardClauses.html) 
makes good sense as they highlight exceptions to the "normal" execution path:

    // bad
    function getPayAmount () {
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
    
        
    // good
    function getPayAmount () {
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
    
## <a name="Equality" />Equality

Always favor the `===` and `!==` operators over `==` and `!=` unless you have specific reasons not to.

By using the "strict equality operators", we can accurately compare the value and type of the variables being compared.

    // bad
    var result = (0 == false); //returns TRUE
    
    // good
    var result = (0 === false); //returns FALSE

The non-strict equality operators (`==` and `!=`) [will attempt to cast the operands](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators) 
into the same value type, returning truthy or falsy results which may not be expected. 

The same problem exists when [comparing native types](http://javascriptweblog.wordpress.com/2011/02/07/truth-equality-and-javascript/#more-2108) 
in an if statement:

    function compare (val) {
        return val ? true : false;
    }
    
    compare({}); // evaluates to true
    compare([]); // evaluates to true, because Array is an Object
    
    compare(undefined); // evaluates to false
    compare(null);      // evaluates to false
    
    compare(true);  // evaluates to true
    compare(false); // evaluates to false
    
    compare(0);   // +0, -0 evaluate to false
    compare(NaN); // evaluates to false
    compare(1);   // all other positive numbers evaluate to true
    compare(-1);  // all other negative numbers evaluate to true
    
    compare('');    // empty string evaluates to false
    compare('foo'); // all other strings evaluate to true

In short, you need to be very careful when testing the equality of any variables when not using the strict 
equality operators!

However, there are some situations in which using truthy or falsy values without direct comparison are acceptable -- 
but again, developers should always be cautious about the values they expect.

    if (!disabled) {
        // ...
    }
    
    // or
    if (enabled) {
        // ...
    }

## <a name="Prototypes" />Prototypes

JavaScript is a prototype-based language -- all objects inherit directly from other objects, and formal "classes" 
do not exist. Prototypal inheritance is conceptually similar to classical inheritance, but Crockford points out that 
"JavaScript is conflicted about its prototypal nature" because "the prototype mechanism is obscured by some 
complicated syntactic business that looks vaguely classical". 

In short, understanding the nuances of prototypal inheritance is key to avoiding errors.

### <a name="Native_Prototypes" />Native Prototypes

Hacking native prototypes should be avoided. It increases the [possibility of naming collisions and incompatible implementations](http://www.nczonline.net/blog/2010/03/02/maintainable-javascript-dont-modify-objects-you-down-own/), 
inevitably causing headaches and hard-to-find bugs.

Instead, create utility classes/methods to implement the desired behavior:

    // bad
    Array.prototype.each = function (functionToCall) {
        //loop over the items in the array
    };
    
    // good
    Ext.define('Ext.Array', {
        singleton : true,
    
        each: function (arrayToIterate, functionToCall) {
            //loop over the items in the array
        }
    }};

Note: in some situations, polyfilling native prototypes may be acceptable to add standard behavior to older browsers. 
For example, Ext JS 5 pollyfills `Function.bind()` in IE8.
    
## <a name="Functions" />Functions

### <a name="Function_Hoisting" />Function Hoisting

When defining JavaScript functions, beware of [hoisting](http://elegantcode.com/2011/03/24/basic-javascript-part-12-function-hoisting/).

Function declarations are evaluated at parse-time (when the browser first downloads the code):

    // FUNCTION DECLARATION (preferred)
    function sum (x, y) {
      return x + y;
    }

Because the declaration is hoisted to the top of its scope at parse-time, it doesn't matter when the function is defined:

    sum(1,2); // returns 3
    
    // FUNCTION DECLARATION (preferred)
    function sum (x, y) {
      return x + y;
    }

Function expressions are evaluated at run-time (when the call stack physically hits a line of code), just like any 
other variable assignment:

    // FUNCTION EXPRESSION
    var sum = function (x, y) {
        return x + y;
    };

Because function expressions are NOT hoisted at parse-time, it DOES matter when the function is defined:

    sum(1,2); //throws an error "undefined is not a function"
    
    // FUNCTION EXPRESSION
    var sum = function (x, y) {
        return x + y;
    };

Using function expressions can result in better compression because the name can be safely replaced by
a shorter version. This is not typically done for the name in a function declaration.
    
### <a name="Anonymous_Functions" />Anonymous Functions

Anonymous functions can be very convenient, but poorly constructed code can easily lead to 
[memory leaks](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript#Memory_leaks). 
Consider the following example:

    function addHandler () {
        var el = document.getElementById('el');
    
        el.addEventListener(
            'click', 
            function () { // anonymous function
                el.style.backgroundColor = 'red';
            }
        );
    }

There are two issues caused by using an anonymous function:

  1. We don’t have a named reference to the click handler function, so we can’t remove it via [removeEventListener()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget.removeEventListener)
  2. The reference to `el` is inadvertently caught in the closure created for the inner function, and therefore 
      cannot be garbage collected. This creates a circular reference between JavaScript (the function) and the DOM (`el`).

To avoid the first problem, always use named functions when adding event listeners to DOM elements.

To avoid the second problem, carefully craft your scopes to prevent leaks and promote garbage collection:
    
    function clickHandler () {
        this.style.backgroundColor = 'red';
    }
    
    function addHandler () {
        var el = document.getElementById('el');
        el.addEventListener('click', clickHandler);
    }

One final note: the risks exposed by the anonymous function pattern are often mitigated by following the paradigms 
of the Sencha class system, where the framework manages much of this scoping and binding for you.
