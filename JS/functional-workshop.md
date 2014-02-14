# Functional Programming Workshop - NodeSchool.io
[Source: Nodeschool.io](http://nodeschool.io/#functionaljs)

## JS Array Functions
### Map
Map runs the callback function on every single element of the array

### Filter
If the callback returns `true`, the item stays, else the item goes out of the array

### Every
If the callback function returns `true` for _**every**_ element in the array, `every` returns `true`, else `false`

### Some
If the callback function returns `true` for _**any one**_ element in the array, `some` returns `true`, else `false`

### Reduce
Returns a single value at the end - it reduces your array into a single value. The idea is that that single value will passed to your callback each time and you can do something to it. By the time reduce is done, you will have the final version of that single value that you can return. 

Works differently if an `initialValue` is provided or not.

#### If `initialValue` is not provided: 

On first run:

- `previousValue` is `array[0]`
- `currentValue` is `array[1]`

On future runs:

- `previousValue` takes the value returned by the function in the previous iteration

#### If `initialValue` is provided:
On first run:

- `previousValue` is `initialValue`
- `currentValue` is `array[1]`

On future runs:

- `previousValue` takes the value returned by the function in the previous iteration

### Call
Call is used when you want to **_call_** methods from a particular prototype on an object that doesn't inherit from the prototype. You are bascially forcing javascript to call that method on the object you pass in.

### Apply
Same as call, except that it takes an Array of arguments instead of taking in arguments as parameters one after the other.

### Bind
Bind is also similar to `call` and `apply` in the sense that you can force JS to call functions on objects. However, instead of calling the function right away, it returns a function that if run, will result in that call. So its like "saving" a `call` or an `apply` into a function that you can easily call.

The arguments passed to `bind` will be **prepended** to any arguments that you decide to pass to the returned function. Because of this, you can also use bind for partial application.


Here, both console.logs will result in the same output.
```js
var curr = {"quack": true};

var caller = Object.prototype.hasOwnProperty.bind(curr, "quack");
console.log(caller());

console.log(Object.prototype.hasOwnProperty.call(curr, "quack"););
```

#### Use in partial application:
So, if we bind to `console.log` like so:
```js
var helloLog = console.log.bind(null, "Hello", ",");
helloLog("World"); // Hello , World
// hellolog("World"); same as console.log("Hello", ",", "World");
```


## Theory
### Partial Application

A.K.A Partial Application of a function. If a function takes in three arguments, we can create a new function by fixing one of those arguments and creating a `partial`.

```js
// add(x,y,z) is a function such that
add(1,2,3); // 6
// add_partial(y,z) is a partial of add in which the first arg is fixed to 1
add(1,2,3) === add_partial(2,3); // true 
```