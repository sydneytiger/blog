Last blog, I have discussed about what does the `call` functon and wrote our own `myCall` mimicing the `call`. The link is below:

[JavaScript: write your own "call"](https://github.com/sydneytiger/blog/issues/6)

In this post, let's dig into another prototype functon `apply` and write our own `myApply`. Similar to `call`, the `apply` take the first parameter as context object, but take the second parameter as an array which collecting all arguments the caller function require. So the logic behind `apply` should be similar to `call`.

### Prepare a test function
```javascript
function add(c, d){
  c = isNaN(c) ? 0 : c;
  d = isNaN(d) ? 0 : d;
  this.a = this.a || 1;
  this.b = this.b || 1;
  return this.a + this.b + c + d;
}

console.log(add(3, 4)); // 9
```
We use the same test function. The function `add()` takes two parameters `c`, `d` (default to 0) and takes `a`, `b`(default to 1) value from context.

### Let's write our own `apply`
```javascript
Function.prototype.myApply = function(context) {
  if(typeof this !== 'function') {            // make sure the caller is a function
    throw new Error('not a function');
  }
  
  context = context || window;                // default to window when passing null context
  const arg = arguments[1] || [];
  context.fn = this;                          // assign the caller function into context
  const result = context.fn(...arg); 
  delete context.fn;
  return result;
}
```
Only one line different from `myCall`. In case you haven't read my `write your own call` blog. I will walk through it again.

#### `this` is the caller function
The does not written in my perfer ES6 arrow function for a reason. It is because in arrow function **`this` inherits the parent scope** which mean, we cannot use `this` to track the caller funciton.

#### The caller must be a function
Ok, back to track. First of all, we check whether the caller is a function.
```javascript
  if(typeof this !== 'function'){
    throw new Error('the caller must be a function');
  }
```
#### Validate context
Then we should make sure the pass in `context` parameter is not null, otherwise, default to window/global. 
```javascript
context = context || window;
```
#### Initialise arguments
The second parameter of `myApply` will be spread `...` into caller funciton as arguments. In case it is null or undefined(spreading null or undefined leads to TypeError), the `arg` is initialise as an empty array.
```javascript
const arg = arguments[1] || [];
```
#### Invoke the caller function
#### Invoke caller function
To make sure the caller function get invoked under the `context`, we bind the caller function `this` to a property in `context`. And then invoke the caller function with spreading `args`. Finally, before returning the result, we sanity the context object by deleting defined caller function.
```javascript
context.fn = this;                     // assign the caller function to context
const result = context.fn(...args);    // execute the caller function - add
delete context.fn;                     // delete the caller function
```

#### Let's test it
```javascript

obj = {
  a: 5,
  b: 6
}

console.log(add.myApply(obj, [3, 4])); // 18
console.log(add.apply(obj, [3, 4])); // 18

console.log(add.myApply(null, [2, 6])); // 10
console.log(add.apply(null, [2, 6])); // 10
```
Happy to see `myApply()` returns the same result as the native `apply()`, Happy coding. 