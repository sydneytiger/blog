JavaScript is scripting language. As it become more and more popular, developers who has backgroud of class-oriented language like java, c++ would love to step into the JavaScript world. And JavaScript is also making itself syntactic friendly to them. New key words like `Class`, `constructor` etc are introcuded to JavaScript.

### class-oriented language constructor
```Java
class Person {
  String firstName;
  String lastName;
  Person(first, last) {
    firstName = first;
    lastName = last;
  }
}

Person peter = new Person("Peter", "Pan");
```

The class `Person` has a `constructor`, this is a special method that every class have. When run `new` operator on a class, the `constructor` method get invoke, althought the `constructor` method does not return anything. But it returns an instance of the class.

JavaScript also has a `new` operator and be able to call so call `constructor` funtion to get back an object just like class-oriented language. However, it is not at all the same as class-oriented language. The `constructor` just a function, nothing special.

### What heppn when `new`
When calling a function with `new` operator, the following things happen:
1. create an new empty object aka constructed object.
2. link the constructed object's prototype to function prototype.
3. explicitly execute the function under the context of the constructed object.
4. unless the fucntion returns object or function, return the constructed object.

### Code impelementation
```javascript
function myNew(constructor) {
  const obj = {};

  obj.__proto__ = constructor.prototype;

  const result = constructor.apply(obj, [...arguments].slice(1));

  if(result && typeof result === 'object' || typeof result === 'function'){
    return result;
  }else{
    return obj;
  }
}
```

### Let's test it
```javascript
function MyFunc(i) {
    this.x = i;
    this.z = 300;
}

MyFunc.prototype.y = 200;

const obj1 = new MyFunc(100);
obj1.x; // 100
obj1.y; // 200
obj1.z; // 300

const obj2 = myNew(MyFunc, 100);
obj2.x; // 100
obj2.y; // 200
obj2.z; // 300
```
Note. JavaScript function name, by convention, is camel case e.g. `myFunc`. We prefer to use pascal case like `MyFunc` on the function name to indicate a constructor function.

How about a funciton returning an object?
```javascript
function MyFunc(i) {
    this.x = i;
    this.z = 300;

    return {a: 500};
}

const obj1 = new MyFunc();
console.log(obj1); // {a: 500}

const obj2 = myNew(MyFunc);
console.log(obj2); // {a: 500}
```
Work as expected. :)