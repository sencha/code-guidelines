#Keeping an Eye on Performance

> *"Performance is only a problem if performance is a problem."*

Every developer should care about performance -- but developers also shouldn't spend time optimizing minute sections 
of code without first proving such efficiencies are necessary. For example, optimizations that make sense in a 
JavaScript library/framework may have little impact in application code.

Given our experience building the Ext JS and Touch frameworks and our exposure to customer applications, Sencha has 
identified the following techniques as proven methods for improving performance:

###Library or Framework Code

  - [Loops](#Loops)
  - [Try / Catch](#Try_Catch)
  - [Page Reflow](#Page_Reflow)
  - [Function-based Iteration](#Function_Based_Iteration)

###Application Code

The Yahoo team put together a document of [exceptional performance techniques](https://developer.yahoo.com/performance/) 
that is so thorough it’s hard to add anything.

Nevertheless, Sencha applications should utilize Sencha Cmd as part of their build process in order to compress the 
code to the smallest possible size. Be sure to follow 
our [Compiler-Friendly Code Guidelines](http://docs.sencha.com/cmd/5.x/cmd_compiler.html)!

## <a name="Loops" />Loops

Don't declare functions (and for that matter, other re-usable things) inside of loops. It wastes processing time, 
memory, and garbage collection cycles:

    // bad
    var objectPool = [];
    
    for (i = 0; i < 10; ++i) {
        objectPool.push({
            foo: function () {}
        });
    }
    
    // good
    var objectPool = [];
    
    function bar () {}
    
    for (i = 0; i < 10; ++i) {
        objectPool.push({
            foo: bar
        });
    }
    
    
Calculate array length only once upfront and assign its value to a variable. This will prevent measuring it for 
every iteration.

    // bad
    var i;
    for (i = 0; i < items.length; ++i) {
        // some code
    }
    
    // good
    var i, len;
    for (i = 0, len = items.length; i < len; ++i) {
        // some code
    }
    
    // bad
    var i;
    for (i = 0; i < items.getCount(); ++i) {
        // some code
    }
    
    // good
    var i, len;
    for (i = 0, len = items.getCount(); i < len; ++i) {
        // some code
    }

Whenever possible avoid `for/in` type of loop as they are known to [negatively impact performance](http://jsperf.com/for-in-vs-keys-vs-for). 
    

## <a name="Try_Catch" />Try / Catch

Avoid try/catch statements when possible as [they cause significant drags on performance](http://jsperf.com/try-catch-in-loop-cost/5).

## <a name="Page_Reflow" />Page Reflow

Avoid patterns that cause [unnecessary page reflows](http://www.kellegous.com/j/2013/01/26/layout-performance/). 

A reflow involves changes that affect the CSS layout of a portion or the entire HTML page. Reflow of an element causes 
the subsequent reflow of all child and ancestor elements, as well as any elements following it in the DOM.

The browser will automatically keep track of DOM and CSS changes, issuing a "reflow" when it needs to change the 
position or appearance of something. Unwieldy JavaScript code can force the browser to invalidate the CSS layout -- for 
example, reading certain results from the DOM (e.g. offsetHeight) can cause browser style recalculation of layout. 
Therefore developers must be incredibly careful to avoid causing multiple page reflows as they will cause application 
performance to noticeably lag.

    // bad
    elementA.className = "a-style";       // style change invalidates the CSS layout
    var heightA = elementA.offsetHeight;  // reflow to calculate offset
    elementB.className = "b-style";       // invalidates the CSS layout again
    var heightB = elementB.offsetHeight;  // reflow to calculate offset
    
    // good
    elementA.className = "a-style";       // style change invalidates the CSS layout
    elementB.className = "b-style";       // CSS layout is already invalid; but no reflow yet
    var heightA = elementA.offsetHeight;  // reflow to calculate offset
    var heightB = elementB.offsetHeight;  // CSS layout is up-to-date; no second reflow!

## <a name="Function_Based_Iteration" />Function-Based Iteration

Function-based iteration, while convenient, will always be slower than using a loop. 

    var myArray = [ 1, 2, 3 ];
    
    // jQuery
    $.each(myArray, function (index, value) {
        console.log(index + ": " + value);
    });
    
    // Ext JS 5
    Ext.each(myArray, function (value, index) {
        console.log(index + ": " + value);
    });
    
    // BETTER PERFORMANCE!
    var len = myArray.length, 
        i, prop;

    for (i = 0; i < len; ++i) {
        console.log(i + ": " + myArray[i]);
    }

Every function invocation creates a new execution context (scope chain). Function calls and returns require state 
preservation and restoration, as well as garbage collection -- while iteration simply jumps to another point in 
the existing context.
