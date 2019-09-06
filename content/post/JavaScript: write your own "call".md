`call()` is one of the prototype functions applying to all Javascript functions. It is useful in term of explicitly attaching a context to a function. Behind the sense, what the `call` function does. Why don't we write our own `call()`.

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
This `add()` takes two parameters `c`, `d` (default to 0) and takes `a`, `b`(default to 1) value from context.

### Let's write our own `call`
```javascript
Function.prototype.myCall = function(context){
  if(typeof this !== 'function'){
    throw new Error('the caller must be a function');
  }

  context = context || window;
  context.fn = this;
  const args = [... arguments].slice(1);
  const result = context.fn(...args);
  delete context.fn;
  return result;
}
```

You might notice that the `add()` and `myCall()` are written in ES5 not in arrow function. It is because `this` will not bind the function's `call-site`. It, instead, inherit the parent scope. So, if we define the `myCall()` like this:
```javascript
Function.prototype.myCall = context => {
  console.log(this) // default to window(browser) or global(node.js)
  ...
}
```
The `this` is always global(window for browser, global for node.js). We cannot get the caller function. So that it has to be written in ES5 function style.

#### The caller must be a function
Ok, back to track. First of all, we check whether the caller is a function. Is it necessary as binding to `Function.prototype`? Not sure, feel free discusss on comment below.
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
#### Invoke caller function
To make sure the caller function get invoked under the `context`, we bind the caller function `this` to a property in `context`. After that, take all arguments(except the first one which is context) for invoking caller function. This is how the native `call` design. We mimic the same behaviour here. And then invoke the caller function. Finally, before, returning the result, we sanity the context object by deleting defined caller function.
```javascript
context.fn = this;                     // assign the caller function to context
const args = [... arguments].slice(1); // take arguments for caller function.
const result = context.fn(...args);    // execute the caller function - add
delete context.fn;                     // delete the caller function
```

#### Let's test it
```javascript

obj = {
  a: 5,
  b: 6
}

console.log(add.myCall(obj, 3, 4)); // 18
console.log(add.call(obj, 3, 4)); // 18

console.log(add.myCall(null, 2, 6)); // 10
console.log(add.call(null, 2, 6)); // 10
```
Happy to see `myCall()` returns the same result as the native `call()`, Happy coding. 