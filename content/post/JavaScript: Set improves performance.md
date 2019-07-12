---
title: "JavaScript: Set improves performance"
date: 2019-07-13T08:24:12+10:00
draft: false
---

In ES6, Set is a very good alternative of array in our daily work. Set and Array has some similar features but performs better than array.

## Set does not have index
```javascript
// Array
const arr = ['foo', 'bar', 'baz', 'bar'];
console.log(arr.indexOf('foo')); // output: 0
console.log(arr.lastIndexOf('bar')); // output: 3
console.log(arr[2]); // output: barz
```
```javascript
// Set
const s = new Set();
s.add('foo') // Set(1) {'foo'}
s.add('foo') // Set(1) {'foo'}
s[0] // undefined
s.indexOf('foo') //TypeError: s.indexOf is not a function
```
As we know, we can use index to access any element in an array. Set, on the other hand, does not have index. It is a collection of key. Each item in a Set must be unique.

## Set is faster than Array
* **Check element:** `array.indexOf()` or `array.includes()` is slower than `set.has()`.
* **Delete element:** set.delete() is more intuitive and faster. Deleting an element in `array.splice()`  base on index is slower.
* **NaN friendly:** Cannot use `indexOf()` or `includes()` to find `NaN` in array, But Set can.
* **Remove duplicate:** Set is a good container to remove duplicate element of a collection.

## Set is faster
Let's run some tests to support it. Test code below runs on Google chrome Version 70.0.3538.67. The result might be different from browser to browser.

**Initiation**: initialise an array and a set with one million element. 
```javascript
const arr = [], set = new Set();
for(let i = 0; i < 1000000; i++) {
  arr.push(i);
  set.add(i);
}
```
**Find an element**
Let's find a number `598893`
```javascript
console.time('Array'); 
arr.indexOf(598893) !== -1; 
console.timeEnd('Array');
console.time('Set'); 
set.has(598893); 
console.timeEnd('Set');

/*
Array: 1.44580078125ms
Set: 0.0068359375ms
*/
```
Set is `212` times faster then Array.

**Delete an element**
Array, unfortunately, has not api of deleting element. So that we build one below:
```javascript
const deleteFromArr = (arr, item) => {
  let index = arr.indexOf(item);
  return index !== -1 && arr.splice(index, 1);
};
```
```javascript
console.time('Array'); 
deleteFromArr(arr, 'test');
console.timeEnd('Array');
console.time('Set'); 
set.delete('test');
console.timeEnd('Set');
/*
Array: 1.27294921875ms
Set: 0.0068359375ms
*/
```
```javascript
console.time('Array'); 
deleteFromArr(arr, 1);
console.timeEnd('Array');
console.time('Set'); 
set.delete(1);
console.timeEnd('Set');
/*
Array: 1.754150390625ms
Set: 0.038330078125ms
*/
```
Set is `46` times faster then Array.

## Set is slower
We have found `Set` performs better than `Array` in term of searching and deleting element. Is it any thing we missing? Yes, the adding. Let's run the code below:
```javascript
const arr = [], set = new Set(), n = 1000000;
console.time('Array')
for (let i = 0; i < n; i++) {
  arr.push(i);
}
console.timeEnd('Array');

console.time('Set')
for (let i = 0; i < n; i++) {
  set.add(i);
}
console.timeEnd('Set');
/*
Array: 61.695068359375ms
Set: 281.642822265625ms
*/
```
When adding one million number, Array is `4.5` times faster than Set. 

```javascript
function random(length) {
  var result           = '';
  var characters       = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  var charactersLength = characters.length;
  for ( var i = 0; i < length; i++ ) {
     result += characters.charAt(Math.floor(Math.random() * charactersLength));
  }
  return result;
}

const arr = [], set = new Set(), n = 1000000;
console.time('Array')
for (let i = 0; i < n; i++) {
  arr.push(random(12));
}
console.timeEnd('Array');

console.time('Set')
for (let i = 0; i < n; i++) {
  set.add(random(12));
}
console.timeEnd('Set');
/*
Array: 926.2939453125ms
Set: 1362.30615234375ms
*/
```
In the case of adding one million random string, Array is `1.47` times faster then Set

## Time complexity
When searching element from Array, It is linear time `O(n)`. The more element it has, the more time it takes to find an element.

Set takes `O(1)` for searching element. The performance has nothing to do with size. 

## What can I take out from it?
**Case 1**: to quickly remove duplicate from an array
```javascript
const arr = ['A', 'B', 'B', 'C', 'D', 'B', 'C'];
let set = new Set(arr); // Set(4) {"A", "B", "C", "D"}
const uniqueArray = [...set]; // ["A", "B", "C", "D"]
```
**Case 2: Interview question**: Given an array of number and a variable called `sum`. If the addition of any two elements in the array equals to 'sum', return `true`, otherwise return `false`.  

Sample input
```javascript
arr = [5, 9, 3, 2, 7];
sum = 8
```
Sample output
```javascript
true 
```
Explanation
5 + 3 = 8 and 5, 3 are all included in the arr.

Solution: Iterating the array and store the different of current element and sum `(sum - arr[i])` in to a `Set`. And check exist of next iterating element against the set.

i.e. first arr element is `5`, then add `3`(8 - 5) into `Set`. On next element of `9`, check `9` against 'Set'. Not found? add `-1`(8 - 9) into `Set`. Then next element of `3`. check exist in `Set`. Found, great return `true` and terminate iteration. Not found for all? return `false`.
```javascript
const findSum = (arr, sum) => {
  const diffSet = new Set();
  for(let i = 0; i < arr.length; i++){
    if(diffSet.has(arr[i])) return true;
    diffSet.add(sum - arr[i]);
  }

  return false;
}
```
The key is finding element. `Set.has()` is O(1), `Array.indexOf()` is O(n).  Using `Set` here is the best solution.