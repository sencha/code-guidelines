#Keeping an Eye on Performance

> *"Performance is only a problem if performance is a problem."*

Every developer should care about application performance -- but developers also shouldn't spend time optimizing 
minute sections of code without first proving such efficiencies are necessary. Sencha has identified the following 
techniques as proven methods for improving application performance:

  - [Loops](#Loops)
  - [Try \ Catch](#Try_Catch)
  - [Page Reflow](#Page_Reflow)
  - [Modifying the DOM](#Modifying_The_DOM)
  - [Function-based Iteration](#Function_Based_Iteration)

## <a name="Loops" />Loops

Don't declare functions (and for that matter, other re-usable things) inside of loops. It wastes processing time, 
memory, and garbage collection cycles:

    // bad
    var objectPool = [];
    
    for (i = 0; i < 10; i++) {
        objectPool.push({
            foo : function() {}
        });
    }
    
    // good
    var objectPool = [];
    
    function bar() {}
    
    for (i=0; i<10; i++) {
        objectPool.push({
            foo : bar
        });
    }
    
    
Calculate array length only once upfront and assign its value to a variable. This will prevent measuring it for 
every iteration.

    // bad
    var i;
    for(i=0; i<items.length; i++){
        // some code
    }
    
    // good
    var i, len;
    for(i=0, len=items.length; i<len; i++){
        // some code
    }

Whenever possible avoid `for/in` type of loop as they are known to [negatively impact performance](http://jsperf.com/for-in-vs-keys-vs-for). 
    

## <a name="Try_Catch" />Try \ Catch

Avoid try/catch statements when possible as [they cause significant drags on performance](http://jsperf.com/try-catch-in-loop-cost/5).

## <a name="Page_Reflow" />Page Reflow

Avoid patterns that cause [unnecessary page reflows](http://www.kellegous.com/j/2013/01/26/layout-performance/). 

A reflow involves changes that affect the CSS layout of a portion or the entire HTML page. Reflow of an element causes 
the subsequent reflow of all child and ancestor elements, as well as any elements following it in the DOM.

The browser will automatically keep track of DOM and CSS changes, issuing a "reflow" when it needs to change the 
position or appearance of something. Unwieldy JavaScript code can force the browser to invalidate the CSS layout - so 
developers must be incredibly careful to avoid causing multiple page reflows as they will cause application 
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

## <a name="Modifying_The_DOM" />Modifying the DOM

As a rule of thumb, touch the DOM as few times as possible. 

DOM changes often cause browser reflow because the browser must determine how elements should be displayed. 
Each reflow takes time -- so the fewer times we touch the DOM, 
[the faster our applications will be](https://developers.google.com/speed/articles/javascript-dom).

### <a name="Switching_CSS" />Switching CSS Classes

Editing CSS styles individually on a DOM element will cause multiple browser reflows and waste processing time. For example:

    function selectAnchor(element) {
      element.style.fontWeight = 'bold';     // reflow
      element.style.textDecoration = 'none'; // reflow again!
      element.style.color = '#000';          // reflow again!
    }

We can instead define all of these styles in a single CSS class, and then set the DOM element's class -- causing 
only a single reflow:

    .selectedAnchor {
      font-weight: bold;
      text-decoration: none;
      color: #000;
    }
    
    function selectAnchor(element) {
      element.className = 'selectedAnchor'; // reflow only once!
    }
    
### <a name="Creating_Elements_Before_Insertion" />Creating Elements Before Insertion

Since we want to touch the DOM as few times as possible, it makes sense to edit those virtually before they have 
been added to the DOM.

    // bad
    function addAnchor(parentElement, anchorText, anchorClass) {
      var element = document.createElement('a');
      parentElement.appendChild(element); // reflow!
      element.innerHTML = anchorText;     // reflow again!
      element.className = anchorClass;    // reflow again!
    }
    
    // good
    function addAnchor(parentElement, anchorText, anchorClass) {
      var element = document.createElement('a');
      element.innerHTML = anchorText;
      element.className = anchorClass;
      parentElement.appendChild(element); // reflow only once!
    }

This pattern can also be applied to multiple DOM elements using DocumentFragements:

    // bad
    function addAnchors(element) {
      var anchor;
      for (var i = 0; i < 10; i ++) {
        anchor = document.createElement('a');
        anchor.innerHTML = 'test';
        element.appendChild(anchor); // reflow!
      }
    } // 10 total reflows
    
    // good
    function addAnchors(element) {
      var anchor, fragment = document.createDocumentFragment();
      for (var i = 0; i < 10; i ++) {
        anchor = document.createElement('a');
        anchor.innerHTML = 'test';
        fragment.appendChild(anchor);
      }
      element.appendChild(fragment); // reflow
    } // 1 total reflow

### <a name="Out_of_the_Flow_DOM_Edits" />Out-of-the-Flow DOM Edits

If we need to make edits to multiple DOM elements, we may want to remove those elements from the DOM temporarily 
to modify -- then re-insert them when we're finished to only cause a single reflow.

    // bad
    function updateAllAnchors(element, anchorClass) {
      var anchors = element.getElementsByTagName('a');
      for (var i = 0, length = anchors.length; i < length; i ++) {
        anchors[i].className = anchorClass; // reflow each time!
      }
    }
    
    // good
    function updateAllAnchors(element, anchorClass) {
        var parent = element.parentNode,
            sibling = element.nextSibling;
    
        parentNode.removeChild(element); // remove from the DOM
    
        var anchors = element.getElementsByTagName('a');
        for (var i=0, length=anchors.length; i<length; i++) {
            anchors[i].className = anchorClass;
        }
    
    
        if (sibling) {
            parent.insertBefore(element, sibling); // reflow
        } else {
            parent.appendChild(element); // reflow
        }
    }

## <a name="Function_Based_Iteration" />Function-Based Iteration

Function-based iteration, while convenient, will always be slower than using a loop. 

    var myArray = [ 1, 2, 3 ];
    
    // jQuery
    $.each(myArray, function(index, value) {
        console.log(index + ": " + value);
    });
    
    // Ext JS 5
    Ext.each(myArray, function(value, index) {
        console.log(index + ": " + value);
    });
    
    // BETTER PERFORMANCE!
    var len = myArray.length, 
        i, prop;
    for (i=0; i<len; i++) {
        console.log(i + ": " + myArray[i]);
    }

Every function invocation creates a new execution context (scope chain). Function calls and returns require state 
preservation and restoration, as well as garbage collection -- while iteration simply jumps to another point in 
the existing context.