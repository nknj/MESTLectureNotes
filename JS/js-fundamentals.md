## General Commands
`console.log("to print stuff")`

```js
console.time("My Math");
var x = 5 + 5;
console.timeEnd("My Math");
```
**Output:**
My Math: 0.006ms

## Data Types
- [PRIMITIVE] number `1` `2`
    + NaN means Not a Number
        + `isNaN("string") //true`
        + `isNaN("1") //false`
        + `isNaN(2) //false` 
- [PRIMITIVE] string | `"a"` `'sd sd'` (double of single quotes)| convert to String using `String([1,2,3])` 
    + `"string".length`
    + `toUpperCase()`, `toLowerCase()`
    + `trim()`
    + `replace()`
    + `charAt(23)`
    + `substring(indexA[, indexB])` second argument is optional
    + `indexOf(searchValue[, fromIndex])` second argument is optional
- [PRIMITIVE] boolean `true` `false`
- array `[1,2,3]` `[1, "two", 3]`, `[["a","b"],[1,2,4],"test",obj={testing:123}]`
    + Can be hetrogeneous (multiple data types), multidimensional (arrays, of same length, in arrays) and jagged (arrays of different lengths, in the array)
- object `{prop: "value", prop2: 12, prop3: [1,2,3], prop4: {another: "object"}}`
    + Check is obj has a property by `varName.hasOwnProperty('prop')`
        * Good to do this before trying to access or edit a prop, avoids errors
    + Print elements by using:
```js
for(var prop in nyc) {
  console.log(prop); // key
  console.log(nyc[prop]); // value
}
```
- function `function (){}` `function fName(){}`

`typeof varName` returns the data-type of the variable passed in

## Syntax
Alot of the times, braces ({}) and semicolons (;) are optional, but there are some edge cases where things may break because you did not put a brace or a semicolon. So when writing js, always include semi-colons and curly braces. mkay? Remember to put semi-colons are the end of functions too.

If you want an easier cleaner version to write, use coffeescript

Using var and not using var:

If you use var, a new object is created in that scope and assigned a value over there. If you do not use var, its like you are editing an older variable and the compiler keeps looking for it in the code above. If it doesn't find the varibale, it create a global variable.
```js
var foo = "I'm global";
var bar = "So am I";

function () {
    var foo = "I'm local, the previous 'foo' didn't notice a thing";
    var baz = "I'm local, too";

    function () {
        var foo = "I'm even more local, all three 'foos' have different values";
        baz = "I just changed 'baz' one scope higher, but it's still not global";
        bar = "I just changed the global 'bar' variable";
        xyz = "I just created a new global variable";
    }
}
```
To make sure that you don't mess up too much here, use `use strict;` at the start of your file. This will make your js code faster and it will use a stricter compiler. It will convert your **mistakes into errors**.

```js
+function() {
    'use strict';
    // Code goes here...
}();
```

### If statements
`==` to check value without data type - `1 == 1` and `1 == '1'` are both true.
`===` to check value and type - `1 === 1` is true; `1 === '1'` is not true.

```js
if (condition) {
    doSomething();
} else if (condition2) {
    doSomethingElse();
} else {
    doNothing();
}
```

### Ternary opertaor
```js
condition ? exprIfTrue : exprIfFalse
```

```js
console.log("You " + (grade > 50 ? "passed!" : "failed!"));
console.log( "Your grade is: " + (grade > 50 ? (grade >= 90? "Excellent!":"Average"):"Need to be improved") );
```

### For loop statments
```js
for (var i = 1; i < 11; i++){
    console.log(i);
}
```

```js
var i = 9;
for(;;){
    if(i === 0)break;
    console.log(i);
    i--;
}
```

```js
for (var obj in list) {

}
```

### While loops
```js
while (condition) {
    doSomething();
    break;
}
```

```js
do {
    doSomething();    
} while (loopCondition);
```

### Switch Statement
```js
switch(lunch){
  case 'sandwich':
    doSomething();
    break;
  default:
    console.log("def");
```

### Function Declaration

#### Method 1
```js
var testFunc = function (anything) {
    console.log("Testfunc" + anything);
};
testFunc(" what?");
```
Defined at runtime, so doing something like this:
```js
testFunc(" what?")
var testFunc = function (anything) {
    console.log("Testfunc" + anything);
};
```
**WILL** give an error

#### Method 2
```js
function testFunc(anything) {
    console.log("Testfunc" + anything);
};
testFunc(" what?");
```
Defined at parse time, so doing something like this:
```js
testFunc(" what?");
function testFunc(anything) {
    console.log("Testfunc" + anything);
};
```
**WILL NOT** give an error
***This is called function hoisting***

#### Method 3
```js
var funcName = function testFunc(anything) {
    console.log("Testfunc" + anything);
};
funcName(" what?"); // no error
testFunc(" what?"); // error
```
If you use both, in the function's body, both `funcName` and `testFunc` will be available but outside the body, only `funcName` will be available. Useful if you want to give the function a short name to use within the body (recursive) and have a unique long name for the wider namespace.

I recommend method 3, but if its not recursive, it becomes method 1.

#### Callback functions
You can pass functions as arguments!

```js
var squareNumbers = function(num, print) {
    var square = num * num;
    print(num, square);
};

squareNumbers(2, function (num, squaredNumber) {
    console.log("The square of " + num + " is " + squaredNumber);
});
```

This is usually used as a asynchorous programming technique.
```js
var squareNumbers = function(num, print) {
    var square = num * num;
    setTimeout(function() {print(num, square)}, 0);
};

squareNumbers(2, function (num, squaredNumber) {
    console.log("The square of " + num + " is " + squaredNumber);
});

console.log("This runs before the callback");
```

#### IIFE - Immediately-invoked function expression - pronounced as `iffy`
```js
(function (num) {
    console.log(num * num);
})(2);
// or
(function (num) {
    console.log(num * num);
}(2));
// or
+function (num) {
    console.log(num * num);
}(2);
```
The last `()` automatically runs the function with the argument

#### Optional arguments
```js
var f = function (a, b, c) {
    if (c != undefined) {
        console.log(c);
    } else {
        console.log("There is no c here!")
    }
};
f(1, 2); //c is undefined
```

### Arrays
```js
var cities = ["Melbourne", "Amman", "Helsinki", "NYC"];
cities[1]; //"Amman"
cities.length; //4
cities.push("Singapore");
cities[4]; //"Singapore"
```

### Packages
`Math.random()`, `Math.floor()` - Math is the package
To import packages in js, just include the script before you require it in a file, or you can use require.js.

The libraries are essentially just copy pasting the code above your code. This brings up the problem of corrupted namespaces. Avoid this by putting all the code in your file in a IIFE function:

```js
(function () {
    // all code goes here
})();
```

### Creating Objects

```js
var obj = {};

var obj2 = {
    hello: "world",
    age: 23,
    "year of birth": 1990
};

var obj3 = new Object();
```
Note that there are no quotes in the key name if there are no spaces in the key name. 

Assign attributes:
```js
obj.attr1 = 'test';
obj["num"] = 2; // if there are spaces in the key, you must use this.
obj.func = function() {return "hello";};
```
The good thing about using the `[]` notation is that you can use variables has the property names. For example:
```js
obj.attr = 'test';
var attribute_name = "attr";
obj[attribute_name]; // Same as
obj["attr"]; // or
obj.attr;

delete obj.attr
obj.attr; //undefined
```

Using the object constructor:
```js
var me = new Object();

me["name"] = "Nikunj Handa";
me.age = 23;
me.age // returns 23
```

Objects can be created in arrays but they will automatically be global, cannot use `var obj` in this, gives error:
```js
var myArray = [1, true, "test", obj = {test:1}];
```

Values can be functions:
```js
var myOwnObject = {
    another : {
        test:1
    },
    object : {
        testing:2,
        123:3,
    },
    setObject: function(obj) {
        this.object = obj;
    }
};

myOwnObject.setObject(...);
```

Values can be functions(2):
```js
var setAge = function(age) {
    this.age = age;
}
var susan = {
    age: 12,
    setAge: setAge
}
```

Creating a Prototype (Custom Constructor, Similar to classes in Java):
Convention: Use a capital letter for the function name of a class definition
```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}
var bob = new Person("Bob Smith", 30);
```
Calling a function with the new keyword creates an object and then runs the function on that object, and then returns that object to the var

```
function Circle (radius) {
    this.radius = radius;
    this.area = function () {
        return Math.PI * this.radius * this.radius;
        
    };
    this.perimeter = function () {
        return 2 * Math.PI * this.radius;
    }
};
```
You can define functions here too, and call them on all objects created from this prototype. 

#### Question for class:
```js
var obj = {
    name: "Nikunj";
};

var test = function() {
    console.log(this.name);
}

obj.test(); // Uncaught TypeError: Object #<Object> has no method 'test' 
```
Will this work?
Answer = No, because the object `obj` doesn't have the property `test`. Test is a property of the global variable (window in the browser).

So if you want to call the `test` function you will have to write it like this:

```js
var obj = {
    name: "Nikunj"
};

var test = function(obj) {
    console.log(obj.name);
}

test(obj); // Works
```

Or you could create a prototype for `obj` that has a property called test with the function:
```js
function obj_type(name) {
    this.name = name;
    this.test = function() {
        console.log(this.name);
    };
}

var obj = new obj_type("Nikunj");
obj.test();
```


Address Book:
```js
var bob = {
    firstName: "Bob",
    lastName: "Jones",
    phoneNumber: "(650) 777-7777",
    email: "bob.jones@example.com"
};

var mary = {
    firstName: "Mary",
    lastName: "Johnson",
    phoneNumber: "(650) 888-8888",
    email: "mary.johnson@example.com"
};

var contacts = [bob, mary];

var printPerson = function (person) {
    console.log(person.firstName + " " + person.lastName);
};

var list = function () {
    var contactsLength = contacts.length;
    for (var i = 0; i < contactsLength; i++) {
        printPerson(contacts[i]);
    }
};

/*Create a search function
then call it passing "Jones"*/
var search = function (lastName) {
    var contactsLength = contacts.length;
    for (var i = 0; i < contactsLength; i++) {
        if (contacts[i].lastName == lastName) {
            printPerson(contacts[i]);
        }
    }
};

var add = function (firstName, lastName, email, phoneNumber) {
    contacts.push({
        firstName: firstName,
        lastName: lastName,
        phoneNumber: phoneNumber,
        email: email
    });
}

add("Nikunj", "Handa", "nikunj.sg@gmail.com", "93385440");
list();
```

## Prototypes

```js
function Dog (breed) {
  this.breed = breed;
};

Dog.prototype.bark = function() {
  console.log("Woof");
};
```
This is a way of giving new property to the class from outside the constructor definition.
More generally:
```js
className.prototype.newMethod = function() {
    statements;
};
```

## Inheritance
```js
function Animal(name, numLegs) {
    this.name = name;
    this.numLegs = numLegs;
}
Animal.prototype.sayName = function() {
    console.log("Hi my name is " + this.name);
};

function Penguin(name) {
    this.name = name;
    this.numLegs = 2;
}
Penguin.prototype = new Animal(); // this is how you set inheritance

function Emperor(name) {
    this.name = name;
}
Emperor.prototype = new Penguin();

var emperor = new Emperor("Nikunj");
console.log(emperor.numLegs); // prints 2
```

- Penguin will inherit all methods and attributes.
- Emperor will also inherit all the the attributes and the value of `numLegs`, 2
- By default, all classes inherit from `Object`, unless we change it by setting the `className.prototype = new SomeOtherClass();`
- Things can keep inherting from each other and create a prototype chain, if you call a property or a method on an object, it will look in the object for that and if it doesn't find it, it will start going up the chain until it reaches `Object`

## Private Variables in Classes
```js
function Person(first,last,age) {
   this.firstname = first;
   this.lastname = last;
   this.age = age;
   var bankBalance = 7500;
  
   this.getBalance = function() {
      // your code should return the bankBalance
      return bankBalance;
   };   
}
console.log(john.bankBalance); //undefined
console.log(john.getBankBalance()); // 7500
```
If you use `var` instead of `this`, you create a private variable. This can be retrieved using another

## Browser Commands
confirm - a popup with ok (returns true) and cancel (returns false)  
    `confirm("message");`  
prompt - a popup with a text field  
    `prompt("message?", "placeeholder in text box");`  
alert - a popup alerting the used of the message (only ok - returns nothing)  b

