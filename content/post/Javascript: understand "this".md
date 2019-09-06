`this` binding is confusing, as the iconic feature is JavaScript. A experienced developer could fail in figure it out the true value of `this` in code.

## Call-site
**Call-site is where a function get called not where it's declared** Understanding call-site is the key of understanding `this` binding. `this` is referencing the call-site.

```javascript
function A(){
  B(); // function B get called, the call-site is A
}

function B(){
  C(); // function C get called, the call-site is B
}

function C(){
  ...
}

A(); // function A get called, the call-site is global
```

Refer to the demo code above, function A, B, C are declared in global scope but the `call-site` is different. It is determined by the called location.

## Four Rules
Now we understand `call-site`, then we have to know 4 binding rules.
1. Default Binding
2. Implicit Binding
3. Explicit Binding
4. New Binding

In practice, we must inspect the `call-site` of the code and determine which rule to apply, and then we can work out the `this` value.

## Default Binding
This is a binding rule that none of other rules can apply. It is the most common case: **Standalone function invocation**

```javascript

function A(){
  console.log(this.a);
  console.log(this.b);
}

var a = 1;

A();
// 1
// undefined
```
We notice that function `A()` is called, the `this.a` resolves to the global variable `a`. The `this.b` response with `undefined` as no `b` is declared in global scope. Now when we declare `b` and run `A()` again. What would happen.

```javascript
var b = 2;

A();
// 1
// 2
```
Therefore, in **default binding `this` is global**.

## Implicit Binding
The second rule is that the `call-site` is a context object like `obj.A`. 
Consider:
```javascript
function A(){
  console.log(this.a);
  console.log(this.b);
}

var obj = {
  a: 1,
  b: 2,
  fn:A
}

var a = 'global';

obj.fn();
// 1
// 2 
```
Well, function `A()` is declared in global and then added to `obj` as a property `fn`. `A()` is not contain or child to `obj`. It is just referenced.

However, when `obj.fn()` invokes, the `call-site` of `A()` is actually `obj`, not global. Implicit binding rule says `obj` should be used for `this` binding. So `obj.a` has been logged, not `global.a`;

Ok, what is the output of the code below:
```javascript
function A(){
  console.log(this.a);
  console.log(this.b);
}

var obj2 = {
  a: 1,
  b: 2,
  fn:A
}

var obj1 = {
  a:3,
  b:4,
  obj2: obj2
}

obj1.obj2.fn();
```

`obj1` references `obj2` and `obj2` reference `A()`. Should it outputting `1, 2` or `3, 4`. The answer is: 
```javascript
// 1
// 2
```
In ab object property reference chain, the `call-site` is the last level object. Or in another word. **The object close to the invoke function wins**

![image](https://user-images.githubusercontent.com/1787825/61423159-1b2f2e00-a952-11e9-9dc7-5dfc0ac4a611.png)

Let's consider another example here:
```javascript
function A(){
  console.log(this.a);
  console.log(this.b);
}

var obj = {
  a: 1,
  b: 2,
  fn:A
}

var a = 'global a';
var b = 'global b';

var funcB = obj.fn;

funcB();
// global a
// global b
```
`funcB` references `obj.fn`, `obj.fn` reference `A()`, when `funcB` invokes, `obj` has nothing to do here, bot `a` and `b` are point to global variables. According to **The object close to the invoke function wins**, no object close to `funcB()` so it falls back to the **Default Binding**. `this` binds to global. It is also known as **Implicitly lost**.

![image](https://user-images.githubusercontent.com/1787825/61423146-0bafe500-a952-11e9-955a-eb71811b5074.png)

When parameter passing an implicit reference function, the **implicitly lost** occurs. Here is an examples.
```javascript
function A(){
  console.log(this.a);
}

function invoker(fn){
  fn();
}

var obj1 = {
  a: 1,
  fn: A
}

var obj2 = {
  a: 2,
  invoker: invoker
}

var a = 6;

obj1.fn() // 1 *implicit reference function

invoker(obj1.fn) // 6

obj2.invoker(obj1.fn) // 6

setTimeout(obj1.fn, 10); // 6
//pass implicit reference function as parameter or callback results in implicit lost. Use default binding
```
It is JavaScript nature that callback function loses its `this` binding. And we, unfortunately, are not able to change it. With ES6 arrow function, it save us from the **implicitly lost**
```javascript
unction A(){
  console.log(this.a);
}

var obj1 = {
  a: 1,
  fn: A
}

setTimeout(() => { obj1.fn()}, 10); // 1
// Hallelujah
```
## Explicit Binding


