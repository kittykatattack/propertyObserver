JavaScript Object Property Observer
===================================

The propertyObserver.js file uses consists of two functions that let you observe property changes on JavaScript objects. Here's how to use them:

1. Create a an object with a few properties. This is the "subject" that you're going to observe.

```
var testSubject = {
  x: 1,
  y: 56,
  name: "",
};
```

2. Start observing the testSubject's "x" property with the observe function
The observe function uses 3 arguments:
a) the object, 
b) the property you want to observe (as a string),
c) the callback handler, bound to "this object".

```
observe(testSubject, "x", callbackHandler.bind(this));
```

3. create the "callbackHandler" function that reacts to this change.

```
function callbackHandler(subject, property) {
  //Find out if a subject changed any of its properties
  if (subject === testSubject) {
    console.log("A property changed on testSubject");

    //Do something if a specific property you're observing changes
    if (property === "y") {
      console.log("Its y property changed");
    }
  }
}
```

To stop observing property changes, you can use the optional unobserve function.

```
unobserve(subjectOne, "x");
```

[Here's a JSFiddle example of all this code in action.](http://jsfiddle.net/d131/mPPU3/)

Listening for changes in DOM elements
-------------------------------------

You can use these functions to listen for changes in DOM elements. Here's how:

1. Create a DOM element.

```
<p id="text">Any text you like</p>
```

2. Get a reference to the DOM element

```
var text = document.querySelector("#text");
```

3. Create a data object to store DOM values.

```
var data = {
  text: undefined,
  initialize: function(config) {
    this.text = config.text;
  }
};
```

4. Load the text into the data object.

```
data.initialize({
  text: text.innerHTML
});
```

Start observing the text property on the data object Arguments: the object you want to observe, the property as a string, and the name of the callback handler, bound to this scope.

```
observe(data, "text", callbackHandler.bind(this));
```

6. Create a callback handler to do something interesting when the property changes

```
function callbackHandler(subject, property) {
  if (subject === data && property === "text") {
    //Notify the console that the property has been changed
    console.log("data." + property + ": " + subject[property]);

    //Update the DOM element with the new value
    text.innerHTML = subject[property];
  }
}
```

To test this, change the text value in the data object. the DOM will refresh automatically.

```
data.text = "The DOM text has been changed"
```

[Check out this JSFiddle to see it all in action.](http://jsfiddle.net/d131/SXQXg/)

The observer functions
----------------------

You'll find these two functions in the observeProperty.js file.

```
//1. observe
//A function to modify an object property's getters and 
//setters so that a custom callback handler can be run in the main
//program each time the property is changed

function observe(subject, property, callbackHandler) {
  Object.defineProperty(subject, property, {
    //Return the default value of the property
    //("this.value" automatically gives you the property's current value)
    get: function () {
      return this.value;
    },

    //Set the property with a new value
    set: function (newValue) {
      //Assign the new value
      this.value = newValue;

      //Bind the observer's changeHandler to the subject
      subject.changeHandler = callbackHandler;

      //Tell the subject to call the changeHandler when this
      //property is changed. (This is like a custom event dispatcher)
      subject.changeHandler(subject, property);
    },

    //Set the default parameters for how this property can be 
    //accessed and changed.You probably don't need to change these
    //unless you want to lock down the property values to prevent your 
    //program from changing them
    enumerable: true,
    configurable: true
  });
}

//2. unobserve
//An optional function to stop watching properties.
//It normalizes the getter and setter and removes the callback handler

function unobserve(subject, property) {
  //Delete the changeHandler
  delete subject.changeHandler;

  //Reset the getter and setter
  Object.defineProperty(subject, property, {
    get: function () {
      return this.value;
    },
    set: function (newValue) {
      this.value = newValue;
    },
    enumerable: true,
    configurable: true
  });
}
```
Should you use them?
------------------
Maybe not! Property observers like this make sense for non-functional languages that aren't event-driven. If you were building an MVC system in C++ for example, you'd need something like this. JavaScript is different. It's functional AND event-driven. There's a simpler, easier, more efficient, more modern and more purely JavaScript way to watch for property changes using requestAnimationFrame. Here's how.

1. Create two DOM elements, an input and output

```
<input type="text" value="Enter some text...">
<p id="output">
```

2. Set up a requestAnimationFrame loop and start observing.

```
//Get a reference to the input and output
var input = document.querySelector("input");
var output = document.querySelector("#output");

//Set up a requestAnimationFrame loop
function update () {
  requestAnimationFrame(update);
  
  //Change the output to match the input
  output.innerHTML = input.value;
}
update();
```

That's all!

(If you need it, here's a simple requestAnimationPolyfill)

```
//Polyfill for requestAnimationFrame
window.requestAnimationFrame = (function(){
  return  window.requestAnimationFrame       ||
          window.webkitRequestAnimationFrame ||
          window.mozRequestAnimationFrame    ||
          window.oRequestAnimationFrame      ||
          window.msRequestAnimationFrame     ||
          function(/* function */ callback, /* DOMElement */ element){
            window.setTimeout(callback, 1000 / 60);
          };
})();
```
[See this code in action here.](http://jsfiddle.net/d131/FXXDb/1/)

### Why this is the best way

requestAnimationFrame uses system resources extremely efficiently, and also has the benefit of synchronizing all the observations into one update cycle. That's really important if you are watching properties on more than one object. Why? Because a property change in one object, might affect a property change in another object. If you check them all in the same cycle, you can control the order in which these changes affect the rest of the code or other objects. That's critical for ensuring that all object properties reflect the most current state of the application.
