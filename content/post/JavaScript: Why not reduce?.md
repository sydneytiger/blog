### What is reduce
`reduce` was introduced by ES6 along with `map` and `filter`. However, it is not well used in daily work. It is too useful to ignore. Once you master `reduce`, it will make you left easier.

Why don't we start with a brief introduction of `reduce`, should we?

- Description from MDN
> "The reduce() method executes a reducer function (that you provide) on each element of the array, resulting in a single output value"
- syntax - `array.reduce(reducer, initValue)`
  - **reducer**: a callback function for implementing your own business logic. (mandatory)
  - **initValue**: the accumulator initial value. (optional)
- How the `reducer` function looks like: `(acc, cur, idx, src) => {}`
  - **acc**: accumulator, it is the output of a `reduce` function. (mandatory)
  - **cur**: current value. (mandatory)
  - **idx**: the index of current value. (optional)
  - **src**: the source array. (optional)
- How it work
  - `acc` starts with `initValue`, if `initValue` is not set, `acc` starts with the value of first array element.
  - Start iteration the source array. Calling `reducer` function by passing current item's value and index of as `cur` and `idx`. You may implement `cur` to `acc` and then return `acc`
  - Repeat above operation to all elements of source array.
  - Return `acc`

The feature of `reduce` is `acc`, it maintains the result of all previous iterations and be used as argument of current iteration. Let's consider:
```javascript
const arr = [3, 5, 1, 4, 2];
const foo = arr.reduce((acc, cur) => acc + cur);
// same as
const foo = arr.reduce((acc, cur) => acc + cur, 0);
```
Don't get it? have to look the animation below:
![reduce-animation](https://user-images.githubusercontent.com/1787825/74614656-dc728800-516d-11ea-9e97-0367dd99aa95.gif)

`reduce` is actually an accumulator, that accumulates each array item in your favour and then return the accumulated value. `reduceRight` is a sibling of `reduce` the only difference is the way they iterates the array. 
- `reduce` iterates array ascendingly (arr[0] ~ arr[length - 1])
- `reduceRight` iterates array descendingly (arr[length - 1] ~ arr[0])

> `reduce` and `reduceRight` will not execute the `reducer` callback for empty array. In another word, `reduce` does not work on empty array.

### Advance usage
#### accumulation and multiplication
```javascript
function accumulation(...vals) {
    return vals.reduce((acc, cur) => acc + cur, 0);
}

function multiplication(...vals) {
    return vals.reduce((acc, cur) => acc * cur, 1);
}

accumulation(1, 2, 3, 4, 5); // 15
multiplication(1, 2, 3, 4, 5); // 120
```

#### reverse array
```javascript
function reverse(arr) {
    return arr.reduceRight((acc, cur) => (acc.push(cur), acc), []);
}

reverse([1, 2, 3, 4, 5]); // [5, 4, 3, 2, 1]
```

#### `map` and `filter` alternative
```javascript
const arr = [0, 1, 2, 3];

// map：[0, 2, 4, 6]
const a = arr.map(cur => cur * 2);
const b = arr.reduce((acc, cur) => [...acc, cur * 2], []);

// filter：[2, 3]
const c = arr.filter(cur => cur > 1);
const d = arr.reduce((acc, cur) => cur > 1 ? [...acc, cur] : acc, []);

// map and filter：[4, 6]
const e = arr.map(cur => cur * 2).filter(cur => cur > 2);
const f = arr.reduce((acc, cur) => cur * 2 > 2 ? [...acc, cur * 2] : acc, []);
```
#### 'one' and 'all'
```javascript
const marks = [30, 46, 60, 88, 90, 100];

// at least subject have one high distinct (> 85)
const isAtLeastOneHighDistinct = marks.reduce((acc, cur) => acc || cur >= 85, false); // true

// all subjects are credit (> 70)
const isAllCredit = marks.reduce((acc, cur) => acc && cur >= 70, true); // false
```
#### 'max' and 'min' alternative
```javascript
function max(arr) {
  return arr.reduce((acc, cur) => acc > cur ? acc : cur);
}

function min(arr) {
  return arr.reduce((acc, cur) => acc < cur ? acc : cur);
}

const arr = [12, 45, 21, 65, 38, 76, 108, 43];
max(arr); // 108
min(arr); // 12
```
#### diff array
```javascript
function Difference(arr1, arr2) {
    return arr.reduce((acc, cur) => (!arr2.includes(cur) && acc.push(cur), acc), []);
}

const arr1 = [1, 2, 3, 4, 5];
const arr2 = [2, 3, 6]
Difference(arr1, arr2); // [1, 4, 5]
```
#### flatten multiple dimensional array
```javascript
function Flat(arr) {
    return arr.reduce((acc, cur) => acc.concat(Array.isArray(cur) ? Flat(cur) : cur), [])
}

const arr = [0, 1, [2, 3], [4, 5, [6, 7]], [8, [9, 10, [11, 12]]]];
Flat(arr); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]

```
#### deduce
```javascript
function deduce(arr) {
  return arr.reduce((acc, cur) => acc.includes(cur) ? acc : [...acc, cur], []);
}

const arr = [2, 1, 0, 3, 2, 1, 2];
deduce(arr); // [2, 1, 0, 3]
```
#### member count
```javascript
function count(arr) {
  return arr.reduce((acc, cur) => (acc[cur] = (acc[cur] || 0) + 1, acc), {});
}

const arr = [0, 1, 1, 2, 2, 2];
count(arr); // { 0: 1, 1: 2, 2: 3 }
```