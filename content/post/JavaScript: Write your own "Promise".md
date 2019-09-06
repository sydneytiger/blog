### Callback hell is bad
Have you been suffered from callback nested callback, so called `Callback hell`. like below:
```javascript
const fs = require('fs');

fs.readFile('./filePath.txt', 'utf8', (err, filePath) => {
  fs.readFile(filePath, 'utf8', (err, file) => {
    fs.readFile(file, 'utf8', (err, data) => {
      console.log(data);
    })
  })
});
```
I am not going to explain the code as I am no longer have to deal with it. Thanks to ES6, we have `Promise` now. It is easier to understand. And `Promise` is probably the JavaScript function you use the most in your day to day work.
```javascript
const fs = require('fs');

// helper function
const read = (filePath) => {
  return new Promise((resolve, reject) => {
    fs.readFile(filePath, 'utf8', (err, data) => {
      if(err){
        reject(err);
      }else{
        resolve(data);
      }
    })
  });
}

read('./filePath.txt')
.then(filePath => {
  return read(filePath);
})
.then(file => {
  return read(file)
})
.then(data => {
  console.log(data);
});
```
Basically, a `Promise` is a wrapper function which wraps another target function and then call the `resolve` or `reject` according to target function result. What happen inside the `Promise`, how is it implemented? Let's write our own `Promise` today.

###  Manage three status
1. Pending
2. Resolved
3. Rejected

When making an asynchronous call e.g. readFile, fetch, it is unpredictable when and where a result come back. Now it is  `Pending` status. When result return, if success the status change to `Resolved` or `Rejected` on error. Note, the status change is unreversed, once status changed to `Resolved` or `Rejected`, it cannot change back to `Pending`.  Let's implement it.
```javascript
const PENDING = 'Pending';
const RESOLVED = 'Resolved';
const REJECTED = 'Rejected';
class myPromise{
  constructor(fn){
    this.status = PENDING; // the status
    this.value = null;      // value is set if async call success
    this.error = null;      // error is set when async call fail
    this.onResolvedCallbacks = [];
    this.onRejectedCallbacks = [];

    const resolve = data => {
      if(this.status === PENDING){
        this.status = RESOLVED;
        this.value = data;
        this.onResolvedCallbacks.map(cb => cb(data));
      }
    }

    const reject = err => {
      if(this.status === PENDING){
        this.status = REJECTED;
        this.error = err;
        this.onRejectedCallback.map(cb => cb(err));
      }
    }

    // execute fn. 
    try{
      // pass resolve and reject functions into target function.
      // and let target function tells success or fail.
      fn(resolve, reject);
    }
    catch(error){
      reject(error);
    }
  }
}
```
* Inside the `myPromise` constructor, the target function get executed when new a `myPromise`.
* `fn` is a function in `function(resolve, reject){ ... }` format.
* `status` has a default value of `Pending` of `Promise` status.
* `value` is the return success data from target function.
* `error` is the return fail data from target function.
* `resolve` is a function passing into `fn` waiting to be called when target function success. It takes one parameter of resolved data.
* `reject` is a function passing into `fn` waiting to be called when target function fail. It takes one parameter of rejected error.
* `onResolvedCallbacks` an event bus storing resolved callback events passing from `then` function.
* `onRejectedCallbacks` an event bus storing rejected callback events passing from `then` function.
Consider one of use case:
```javascript
const p = new myPromise((resolve, reject) => {
  setTimeout(() => {
    resolve('success');
  }, 1000);
})

console.log(p);     // myPromise {status: "Pending", value: null, error: null}
setTimeout(() => {  // 1 second later 
  console.log(p);   // myPromise {status: "Resolved", value: "success", error: null}
}, 1000)
```
Inside the `myPromise`, we use `setTimeout` function to simulate an ajax call. We have to build the the `fn` function in a specific way. We have to pass `resolve` and `reject` as parameter and call it respectively when success or fail. `fn` is the connector between your ajax call and a `Promise` object, and `resolve` and `reject` are the notifier to `Promise` for `status` change.
```javascript
const fn = (resolve, reject) => {
  ... // ajax call trigger here.
  resolve(data)  // call resolve when success notifying Promise to change status to Resolved.
  // or
  reject(error) // call reject when fail notifying Promise to change status to Rejected.
}
```
Inside the `resolve` and `reject` functions, we check the current status. And only change the status when it is currently 'Pending'. By doing this, we preventing the `resolve` or `reject` logic from executing more than once. It also ensure that the `status` cannot be reversed.

### set callback in `then`
Ok, now we understand the `Promise` status and how do they change by the asynchronous function. Next step is to tell `Promise` what callback function should be executed when status change. It is the `then` function. The `fn` function not only connects the target function and `Promise`, but also passing the `resolve` or `reject` data to `then`. Inside `then` we can implement our callbacks.
```javascript
  then(onResolved, onRejected) {
    // make sure the callback is a function.
    onResolved = typeof onResolved === 'function' ? onResolved : res => res;
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };

    if(this.status === PENDING) {
      this.onResolvedCallbacks.push(onResolved);
      this.onRejectedCallbacks.push(onRejected);
    }

    if(this.status === RESOLVED){
      onResolved(this.value);
    }

    if(this.status === REJECTED){
      onRejected(this.error);
    }
  }
```
We have to make sure the `onResolved` and `onRejected` are functions, if not, set them to default.

When `Promise` an **asynchronous** function, we store the callbacks into the event bus `onResovledCallbacks` and `onRejectedCallback` arrays. Once the async function settled, **the callbacks are fired one by one in `resolve` or `reject` function.**
```javascript
const resolve = data => {
  ...
  this.onResolvedCallbacks.map(cb => cb(this.value));

const reject = err => {
  ...
  this.onRejectedCallbacks.map(cb => cb(this.error));
```

When `Promise` an **synchronous** function, the **event bus are not in used** since the call settled instantly. So we have the code below to cover sync call.
```javascript

 if(this.status === RESOLVED){
      onResolved(this.value);
    }

  if(this.status === REJECTED){
    onRejected(this.error);
  }
```
Let's have a test
```javascript
const p = new myPromise((resolve, reject) => {
  setTimeout(() => {
    resolve('success');
  }, 1000);
}).then(data => { console.log(data)})

const f = new myPromise((resolve, reject) => {
  setTimeout(() => {
    reject('fail');
  }, 1000);
}).then(null, data => { console.log(data)})

// success
// fail
```
Awesome, now we have successfully implemented our own `Promise` function. However, it is a simple version. It cannot handle `then` chaining. To chain them, `then` should return a `Promise`. I will address this issue in my next post.
### Codes
```javascript 
const PENDING = 'Pending';
const RESOLVED = 'Resolved';
const REJECTED = 'Rejected';
class myPromise{
  constructor(fn){
    this.status = PENDING;
    this.value = null;
    this.error = null;
    this.onResolvedCallbacks = [];
    this.onRejectedCallbacks = [];

    const resolve = data => {
      if(this.status === PENDING){
        this.status = RESOLVED;
        this.value = data;
        this.onResolvedCallbacks.map(cb => cb(this.value));
      }
    }

    const reject = err => {
      if(this.status === PENDING){
        this.status = REJECTED;
        this.error = err;
        this.onRejectedCallbacks.map(cb => cb(this.error));
      }
    }

    try{
      fn(resolve, reject);
    }
    catch(error){
      reject(error);
    }
  }

  then(onResolved, onRejected) {
    onResolved = typeof onResolved === 'function' ? onResolved : res => res;
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };

    if(this.status === PENDING) {
      this.onResolvedCallbacks.push(onResolved);
      this.onRejectedCallbacks.push(onRejected);
    }

    if(this.status === RESOLVED){
      onResolved(this.value);
    }

    if(this.status === REJECTED){
      onRejected(this.error);
    }
  }
}
```