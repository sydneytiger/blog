Before we start the marcoTask and microTask journey, please consider the code:
```javascript
console.log('script start')
setTimeout(function () {
  console.log('setTimeout')
}, 0);
Promise.resolve().then(function () {
  console.log('promise1')
}).then(function () {
  console.log('promise2')
})
console.log('script end')
```

Try to write down the output of the code and compare with what we have learn after this article.

### Single Thread
Single thread means all tasks are waiting in a queue. A task does not process until the one before it finish. If one task take long time to process, all tasks behind have to wait.

`JavaScript is single threading language`, it is because it was invented as script language for web browser. It was designed for interaction between browser and user, manipulating `DOM`. If JavaScript was multiple threading, one thread added a node to `DOM` and, meanwhile, the other one removed a node to `DOM`. Which action should browser take? Single threading works the best with browser as in a certain time there is one and one only task running.

Although `HTML5` introduced `Web Worker` which utilising multiple core of `CPU` to run multiple JavaScript thread. However, all threads are under the control of main thread and they are not allow to manipulate `DOM`. `Web Worker` does not change the fact that JavaScript is single threading.

### macroTask vs microTask

`macroTask` aka `task`. It sits in `macroTask queue`. All synchronise tasks and some asynchronise callback tasks are classified as `macroTask`:
- script block
- DOM manipulation
- setTimeout & setInterval
- requestAnimationFrame
- I/O task

`microTask` aka `job`. It sits in `microTask queue`. All `microTasks` are asynchronise callback `job`:
- Promise.then
- MutationObserver

Please consider the digram of JavaScript event loop flow:
![javascript event workflow](https://user-images.githubusercontent.com/1787825/67661631-d5085280-f9b5-11e9-9e3c-8280bd1dcaa5.jpg)

There is how a JavaScript get executed:
1. A thread takes the `Main` and executes the code line by line synchronously. The `Main` is the very first `macroTask`.
2. When coming across `setTimeout`, the callback function will be push to the end of `macroTask queue` and waiting for ***all exiting*** `macroTask` to be finished.
3. When coming across `Promise`, the callback function will be push to the end of `microTask queue` and waiting for ***current*** `macroTask` to be finished.
4. When current `macroTask` finished, before next `macroTask`, thread starts executing `job` in `microTask queue`. A new `job` might be created in progress, if so push to `microTask queue`.
5. When all `jobs` in `microTask queue` are taken care, thread moves to next `macroTask`. and then executes the `microTask queue` when this `task` is done.

In conclusion, JavaScript thread executing `tasks` from `macroTask queue`, when finished, before next `task`. The thread checks the `microTask queue` to process all `jobs` and moves to next `task` until the `microTask queue` is empty.

### Back to the question
Now we have learn how JavaScript work. Let's get back to the question the very beginning of this article. I am using three arrays representing `call stack`, `macroTasks queue` and `microTask queue`. `main` here is the entire code block.
1. Right before the code is executed, `call stack` and `microTasks queue` are empty. `macroTask queue` contain `main`
```javascript
callStack: []
marocTasks: [main]
microTasks: []
```
2. `main` is taken from `macroTasks queue` and push into `call stack` to execute.
```javascript
callStack: [main]
marocTasks: []
microTasks: []
```
3. when coming across to `setTimeout` and `Promise`, they are pushed to `macroTasks queue` and `microTasks queue` representivly. And then `main` finished.
```javascript
callStack: [main]
marocTasks: [setTimeout]
microTasks: [pormise]
```
Now the output is:
```javascript
script start
script end
```
4. After `main`, before next `marcoTask`(setTimeout), JavaScript checks the `microTasks queue` and process `promise`. There is another promise chaining, it is also pushed into `microTasks queue`.
```javascript
callStack: [promise]
marocTasks: [setTimeout]
microTasks: [promise]
```
Now the output is:
```javascript
promise1
```
5. Right after that the second `promise` is taken from `microTasks queue` to `callStack` and get executed.
```javascript
callStack: [promise]
marocTasks: [setTimeout]
microTasks: []
```
Now the output is:
```javascript
promise2
```
6. All `job` in `microTasks queue` is done, JavaScript get back to `macroTasks queue` and take care of `setTimeout`
```javascript
callStack: [setTimeout]
marocTasks: []
microTasks: []
```
Now the output is:
```javascript
setTimeout
```
The final output is:
```javascript
script start
script end
promise1
promise2
setTimeout
```
Here is an animation explaining how the JavaScript process the code:
![eventloop workflow](https://user-images.githubusercontent.com/1787825/67661654-de91ba80-f9b5-11e9-82d8-6adf8b6d8717.gif)

In summary, JavaScript engine is single thread. It process one and only one event at a certain time. It is also called `Event loop`.

When a user is browsing a web page, events ,like `onClick`, `scroll down` etc, get triggered all the time. If function is subscribed to events, they will be pushed into different either `macroTask queue` or `microTask queue`. `Event loop` is actually taking `events` from different `queue` and hand over the process engine.
