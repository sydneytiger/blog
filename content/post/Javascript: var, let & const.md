---
title: "JavaScript: var, let & const"
date: 2019-07-14T08:24:12+10:00
draft: false
---
Declaring variables, assigning values. Isn't it the routine of daily work?

```javascript
// declare varibles
var a = 8;
let b = 'sydney';
const c = true;

// assign value
a = 9;
b = b + 'tiger';
c = false; // TypeError: Assignment to constant variable
```

We cannot familiar more with the code above. However, what is really happening inside the JavaScript world. To achieve a comprehensive knowledge, we should understand:

-   JavaScript primitive types, declare and value assignment.
-   JavaScript call stack and heap.
-   JavaScript reference types, declare and value assignment.
-   let vs const

## JavaScript primitive types, declare and value assignment

JavaScript has six primitive types:

-   string
-   number
-   boolean
-   undefined
-   null
-   symbol

Let's declare a variable called `userName` and assign it with value of `sydneytiger`.

```javascript
let userName = 'sydneytiger';
```

When JavaScript engine executes the code above. It does

1. create an indentifier to the variable `userName`.
2. allocate an address in memory.
3. set value `sydneytiger` to the address

![image](https://user-images.githubusercontent.com/1787825/61271256-6ed13880-a7e7-11e9-8a33-be7c6e0b63c0.png)

Instead of saying "the variable `userName` equals to `sydneytiger`", precisely, it is that `userName` equals to the address which storing `sydneytiger`.

`userNumber` is technically equals to `0012CCGWH80`. Then, declaring another variable `login` to `userName`.

```javascript
let login = userName;
console.log(login); // sydneytiger
```

The `login` is also been assigned to the same memory address `0012CCGWH80`. In memory point of view, no new address has been allocated for `login`. `login` and `userName` both point to address `0012CCGWH80`.

![image](https://user-images.githubusercontent.com/1787825/61271430-f6b74280-a7e7-11e9-9d11-2d1b18e82288.png)

What happen in memory when we change the value of userName?

```javascript
userName = 'tigerchen';
console.log(userName); // tigerchen
console.log(login); // sydneytiger
```

Since `userName` and `login` had the same memory address `0012CCGWH80` why they now have different values. It is because **JavaScript primitive type is immutable**. In another word, once a value is allocated to memory `0012CCGWH80`, it cannot be change but destroyed(garbage collector).

When assigning a new value to userName, JavaScript allocates a new memory slot `0034AAAAH23` storing `tigerchen`. While `login`, still, points to the old address `0012CCGWH80`.

![image](https://user-images.githubusercontent.com/1787825/61271596-675e5f00-a7e8-11e9-823e-30a3d96bf944.png)

Let's change the value of `login` variable.

```javascript
login = login + ' code';
```

You might pick up what I have said wrong here. It is not changing value, it is actually allocation a new value in memory and get `login` pointing to it. Since no variable is using the memory `0012CCGWH80`. It will be destroyed by garbage collector later on.

![image](https://user-images.githubusercontent.com/1787825/61272263-12bbe380-a7ea-11e9-8f62-962830e1afa3.png)

It is why we should not manipulate a big string, since every operation results in coping data into a new memory slot. It may lead to memory overflow.

```javascript
let article = 'Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do';
article = article + 'a';
article = article + 'b';
article = article + 'c';
article = article + 'd';
```
![image](https://user-images.githubusercontent.com/1787825/61272963-b5289680-a7eb-11e9-816c-f198685bd3eb.png)

## JavaScript call stack and heap

How about those none primitive types like array, object? Yes, they are reference types. it is a bit different in memory point of view. Before dig into it, we should have a closer look into memory and understand `call stack` and `heap`.

![image](https://user-images.githubusercontent.com/1787825/61335855-565a3000-a872-11e9-98a0-436cf51458d5.png)

Memory is consist of `call stack` and `heap`. All memory allocation for primitive types happen in `call stack`. `Heap`, on the other hand, is a big chunk of free memory space used to store dynamic size of data. It is flexible, adding or deleting data will not result in new memory allocation. It is the place storing reference types data.

## JavaScript reference types, declare and value assignment

Let's declare an empty array.

```javascript
const arr = [];
```

What happen in memory?

1. create an identifier to the variable `arr`.
2. allocate an address in call stack.
3. point identifier `arr` to the call stack address.
4. allocate an address in heap.
5. save the heap address into call stack address as value.
6. set the empty array value in to heap address.

![image](https://user-images.githubusercontent.com/1787825/61336247-f9f81000-a873-11e9-8cbf-981299b136d7.png)

Now, when modifying the arr like `push`, `pop` etc happen on the heap address will not effect the address value in call stack.

```javascript
arr.push(1);
arr.push('2');
arr.push(3);
arr.push(4);
arr.pop();
```
![image](https://user-images.githubusercontent.com/1787825/61336322-493e4080-a874-11e9-9f4c-725c7cefb925.png)

## let vs const

We all know `let` is used for declaring variable which will change, whereas `const` is used for declaring constant which will not change. You may have noticed, when I use `const` to declare `arr` not `let` and it works well when changing the `arr`.

The **change** here is not the common sense of change value. It is the change of memory address in call stack.

```javascript
const id = 123;
id = 321; // TypeError: Assignment to constant variable
```

Primitive type is immutable, changing the value result in a new memory address. The `id` is allocated with address `0012CCCWH88` with value of `123`. When changing to `321`, it will allocate a new address `0276GGHBC00`. `const` is preventing it from happening. So it throw `TypeError`

![image](https://user-images.githubusercontent.com/1787825/61336760-f1083e00-a875-11e9-8930-e67bd947ee6f.png)
```javascript
const myArray = [];
myArray.push(1);
myArray.push(2);
myArray.push(3);
myArray.push(4);
myArray.push(5);
```

Changing the value of reference type like array does not result in address change in `call stack`. The change happens in `heap` which either `const` or `let` care.

```javascript
myArray = 'sydneytiger'; // TypeError: Assignment to constant variable
```

`sydneytiger` is a primitive type. `Call stack` allocate a new slot `0012DDGGWH99` to store it and try to assign to identifier `myArray`. `const` throws `TypeError`.

![image](https://user-images.githubusercontent.com/1787825/61336941-ad620400-a876-11e9-9d43-592236455e95.png)

How about assign a new array to the `myArray` identifier?

```javascript
myArray = ['a', 'b'];
```

It throws `TypeError` exception too as it is changing the `call stack` address

![image](https://user-images.githubusercontent.com/1787825/61337046-0d58aa80-a877-11e9-97a6-f96ab81cb5c7.png)

The same as declaring an object. The value of object stores in `heap` and save the `heap` address value in 'call stack' address. So that the code below is totally fine.

```javascript
const student = {};
student['name'] = 'Tiger Chen';
```

## What can I get out from it?

After JavaScript ES6, developers are encouraged to use `let` or `const` over `var`.

1. avoid bug introduced by JavaScript hoist
2. const must be assign value on declare. It helps developers organise code better.
3. force developers double think about the variable/constant when declaring.
4. for React developers, understand why setState should use `[...arr, 3]` instead of `arr.push(3)`
