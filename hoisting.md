## Hoisting

To understand hoisting, one needs to understand the **execution context**.
The context in which the function is invoked is the **invocation context** of the function.
(It is about the caller)

### So what is the execution context?.
    - it is the environment in which the function is executed

### Every code in JS is run in one of the 3 environments only :
- Global - the default environment where the code is executed for the first time
- Function - every function call creates a new function context
- Eval - code that is run in the eval function

### Regarding Function contexts:
- You can have any number of function contexts.
- Every new function invocation creates a new function context.
- Such a function context, creates a private scope.
- So that, variables declared within it, are not accessible outside it.
- The scope is the boundary or limit to which the variable is accessible.
- Anything declared within it, is not accessible by members outside it (even though this function may be contained within them).
- But, members outside it, can be accessed by members within

### how is this possible?

- The JS interpreter is single threaded. Only one thing can be executed at any moment. So it has queues, where it queues up tasks to be done. This queue is the Execution Stack.
- When a script loads for the 1st time, the execution stack has only 1 values -> Global context.
- Subsequent function invocations, place new entries is the execution stack.
- Only the currently executed function is always maintained at the top of the stack.
- If you invoke a new function from within this, then a new entry is created n pushed to the stack.
- Once this function completes execution, it is popped out of the execution stack & control comes back to the next top element of the stack -> the function caller.

### Points to remember about the execution stack
- Single Threaded & synchronous
- 1 Global context. Infinite function contexts (ideally, but in reality it will throw max call stack exceeded, which your browser will state "This script is taking too long to load")
- Every function invocation creates a new entry in the stack, even if called recursively
- Inner can access outer while outer cannot access inner*The reason why members outside cannot access members inside, is because, it will be popped out of the stack. But, members within can access members outside, as the scope chain will still be there to access elements above it

### Execution context in Detail

So, every time a function is invoked. What happens ?
- new execution context is created?
- new entry added to the execution stack
- outer container variables are accessible
- container cannot access inner as the control / top pointer of the stack is now with the inner function
- execution context object is created

### what does this execution context object have?
This execution context object is created in 2 stages :

#### A) Creation Stage
      **When does this occur ? :** 
        - when the function is called, but before execution
      **What it does**
             This is to do 3 things:
              1) Determine / created the scope chain
              2) Create the Activation object (or) Variable object that contains the FAV (Function declaration references, Arguments object, Variable references)
              3) Find the value of this

#### B) Activation Stage (or) execution stage

      **When does this occur ? :** 
        - while the code executes
      ** What it does :** 
        - This assigns values to variables, references to functions, executes code

#### What is this VO (variable object) or also called AO (Activation object) contain ?
It is filled in the order of FAV (Arguments, functions, variables) & contains the

- **property names for function declarations** : These are property names created in the AO / VO with the same name as the inner function declarations. This property will maintain a reference pointer to the function in memory. If a property already exists, it will be overriden.

- **Arguments Object** : This contains the parameters / arguments of the function. Only these guys get assigned values (or) initialized with the value that was present during the invocation of the function

- **property name for variable declarations** :  There will be a property created for every variable declaration also. It is created with the same & initialized with the value of undefined. Note that function expressions also are part of this only.


### So, some table rules are:

1) Order of initializations are Arguments, Function declarations & variables
2) Function expressions are same as variables. They will be initialized & assigned a value of undefined
3) Function declarations are a priority over variables So, if there is a function foo & later you assign a value to the same foo. It will still retain the reference to the function in memory & will not have the variable value.
4) If the variable property exists already : Then ignore & assign it to undefined

`````javascript
function foo(i) {
    var a = 'hello';
    var b = function privateB() {

    };
    function c() {

    }
}

foo(22)
``````

In the above example, during the creation stage the execution context object looks as below:

`````javascript
fooExecutionContext = {
    scopeChain: { /*created here & has the chain of container parents, their scope, this, AO etc*/ },
    variableObject: {
        arguments: {
            0: 22,
            length: 1
        },
        i: 22, only arguments / parameters have values assigned to them
        c: pointer to function c()
        a: undefined, its a variable
        b: undefined its a variable
    },
    this: { /* the current function is executed in which context. Who is the owner? */ }
}
````````

During the activation or execution stage


`````javascript
fooExecutionContext = {
    scopeChain: { /*created here & has the chain of container parents, their scope, this, AO etc*/ },
    variableObject: {
        arguments: {
            0: 22,
            length: 1
        },
        i: 22, only arguments / parameters have values assigned to them
        c: pointer to function c()
        a: "hello", /*its a variable*/
        b: pointer to "function privateB" /*its a variable*/
    },
    this: { /* the current function is executed in which context. Who is the owner? */ }
}
```````

#### Example 2:

`````javascript
(function() {

    console.log(typeof foo);
    console.log(typeof bar);

    var foo = 'hello',
        bar = function() {
            return 'world';
        };

    function foo() {
        return 'hello';
    }

}());
```````

**Answer** : foo is a function. Bar is undefined

#### Example 3

`````javascript
var foo = 1;
function bar() {
    if (!foo) {
        var foo = 10;
    }
    alert(foo);
}
bar();

console.log(foo);
``````

**Answer:**
Foo is 1 in the global scope. Function invocation, creates a new stack in execution stack. In the execution context object  a property with name * Foo* is created & is assigned undefined. So it enters the if block & is assigned a value of 10 during execution stage. So alert is 10.
The function execution is complete & is popped out of the execution context stack. Now the pointer is back to global context.
console.log(foo) will still be 1.

#### Example 4

`````javascript
var a = 1;
function b() {
    a = 10;
    return;
    function a() {}
}
b();
alert(a);
```````
**Answer:** In the Global context, the a is 1.
But when b() is invoked, a property a is created in the activation object inside the execution context object . There is a function declaration for a & this takes precedence. So, a is not assigned a value of 10. It is still 1.

#### Example 5

`````javascript
var x = 1;
console.log(x); // 1
if (true) {
    var x = 2;
    console.log(x); // 2
}
console.log(x); // 2
``````

**Answer:** In this case, Output will show 1, 2, 2. This is because JavaScript has function-level scope & NO Block Scope
(Block scope has come in Es6 although)

#### Example 6

`````javascript
function foo() {
    var x = 1;
    if (x) {
        (function () {
            var x = 2;
            // some other code
        }());
    }
    console.log(x) // is still 1.
}
`````

**Answer:** During the execution stage x is assigned a value of 1.
So it enters the if-block & the IIFE executes. Inside IIFE, x has a value of 2.
Then it completes execution. Its popped out of the execution stack & control is back to the caller.
So, x is still 1 here.

#### Example 7

`````javascript
(function() {
  var foo = 1;
  var bar = 2;
  var baz = 3;

  alert(foo + " " + bar + " " + baz);
})();
``````

**Answer:** : its 1 2 & 3.

#### Example : 8

`````javascript
(function() {
  var foo = 1;
  alert(foo + " " + bar + " " + baz);
  var bar = 2;
  var baz = 3;
})();
``````

**Answer:** : foo is 1 while the rest are undefined

#### Example : 9

`````javascript
(function() {
  var foo = 1;
  alert(foo + " " + bar + " " + baz);
  var bar = 2;
})();
``````

**Answer:** : Here foo is 1. baz is undefined. But, bar will undefined as its not defined, baz will throw a reference error

So,

`````javascript
(function() {
  var foo = 1;
  alert(foo + " " + bar + " " + baz);
  var bar = 2;
  var baz = 3;
})();
`````

is same as


`````javascript
(function() {
  var foo;
  var bar;
  var baz;

  foo = 1;
  alert(foo + " " + bar + " " + baz);
  bar = 2;
  baz = 3;
})();
`````

**Rule**: All variable & function declaration are moved to the top of the current scope. But, only function declarations are hoisted.

#### Example 10

`````javascript
function test() {
    foo(); // TypeError "foo is not a function"
    bar(); // "this will run!"
    var foo = function () { // function expression assigned to local variable 'foo'
        alert("this won't run!");
    }
    function bar() { // function declaration, given the name 'bar'
        alert("this will run!");
    }
}
test();
```````

**Answer:** : Inside test function, there is no function declarations for the property foo & so is undefined So, attempting to invoke it before assigning it a value / function expression will throw TypeError: not a function . bar however, is a function declaration, moved to the top of the current scope & will be hoisted / receive a value. So bar() is a function n can be invoked. Then, foo is assigned a function expression. Now if foo is invoked, it will run
So, due to hoisting, function declarations can be invoked before their declaration

#### Example 11

`````javascript
var foo = "bar";

function bar() {
    var foo = "baz”; // new property foo is created inside this function and is assigned a value of “baz"
    function baz(foo) { //copies the value into the arguments object variable instead of creating or overriding the global foo
         foo = "bam";
         bam = "yay”; //global variable creation & gets a value 'yay' that can be accessed anywhere now
    }
    baz(); // this runs. As its a function declaration.
}
````````

`````javascript
bar();
foo;         // "bar"
bam;         // "yay"
baz();         // Error!
`````````

*Question is what if we did not pass a foo ? Then it will override the global foo ? - Yes*

**Answer:** :

- foo is used after it is initialized. So it will have a value of "bar".
- Property bar in the AO/VO of the execution context obj will have a pointer to the function declaration bar. So, it can be executed.
- So far, there is nothing mentioned about bam or baz. It may likely be filled within the bar function (or) may end up being reference errors
- Now, when bar() is executed, a property foo is created which has a value = "baz"
- While a property baz is created, which points to the baz function in memory.
- baz() is now invoked with empty parameters.
- Now when baz() is executed, the Arguments object inside the variable object (VO), will have a property for its param foo which shall be undefined as there were no params passed when the function was invoked.
- Now, that foo gets assigned a value of bam. It does not end up being global nor affects the global foo which has a value bar.
- However, bam is a new property initialized with a value "yay". Now this was not present in the arguments object & so ends up being scoped globally.
- After the execution of baz(), baz() is popped out of the execution stack after which bar() is also popped out & control comes back to the global scope. foo is still "bar" & bam is "yay". But there was no property created for baz.

Values are "foo", "yay", "reference error".

#### Example 12

`````javascript
var n = 1;

function printSomething() {
    console.log(n);
    var n = 2;
    console.log(n);
}
`````

=> printSomething();
undefined
2

**Answer:** : Initially, when printSomething is invoked, the variable n is newly declared. It does not refer to the one above it because of the statement var n=2. A new copy of n is declared, but not assigned any value. So its undefined. After which the assignment happens, & so the value is 2

#### Example 13 (This is also an example for setTimeOut inside a forloop, accessing a value of the looped array or object

`````javascript
var txt = ["a","b","c"];

for (var i = 0; i < 3; ++i ) {
   var msg = txt[i];
   setTimeout(function() {
     alert(msg);
   }, i*1000);
}
``````

**Answer:** : The overall execution is the global context only. So, txt is available inside the for loop.
But since there is a delay in the execution of the setTimeOut which executes a function that references a value that is outside it (or) the in the containing scope, it is unable to trap the value of the variable msg
So, always it alerts only c. The way to overcome this is :

#### We know every function invocation :

- creates a new entry in the execution stack
- creates a new execution context object (which is created in 2 stages as mentioned above)
- creates a new function context
- the function context so created, will create a private scope for itself
- *variables declared within it are not accessible to the outer members or containing members
- But members outside it can be accessed by members within

So, the simple way to trap the value of txt[i] would be to create a function wrapper & pass this current value of the loop variable, during invocation.

So the 1st step is : 
- create a function wrapper & pass to it the current loop variable value
- But, setTimeOut requires a function. It cannot operate on values & needs a type === 'function'
- So, we need to make our function wrapper, return a function.

So step 2 is : 
- We need to return a function

So, our final out come is now:

`````javascript
var txt = ["a","b","c"];

for (var i = 0; i < 3; ++i ) {
   setTimeout((function(msg) {
          return function() {
               alert(msg);
          }
   })(txt[i]), i*1000);
}
`````

**Answer:**
The function that was created as a wrapper, thus self invoked itself & immediately executed & returned a new function with the current loop variable trapped within it. Now settimeout works perfect! Such wrapper functions are called self invoking functions

So we understand what is hoisting now, with these basic examples.
**Hoisting** is the mechanism of moving the variables and functions declaration to the top of the function scope (or global scope if outside any function).

#### Hoisting influences the variable life-cycle, which consists of these 3 steps:

- Declaration - create a new variable. E.g. var myValue
- Initialization - initialize the variable with a value. E.g. myValue = 150
- Usage - access and use the variable value. E.g. alert(myValue)

The process usually goes this way:
1) first a variable should be declared,
2) then initialized with a value and
3) finally used.

Everything looks simple and natural when these steps are successive: declare -> initialize -> use.

`````javascript
// Declare
var strNumber;
// Initialize
strNumber = '16';
// Use
parseInt(strNumber); // => 16
`````

A function can be declared and later used (or invoked) in the application.
The initialization is omitted.
This is because, function declarations get properties in the Activation / Variable object, that refers to them in the creation stage itself (explained above)

`````javascript
// Declare
function sum(a, b) {
       return a + b;
}
// Use
sum(5, 6); // => 11
`````

But, only for function declarations, we could do :// Use

`````javascript
double(5); // => 10
// Declare
function double(num) {
      return num * 2;
}
``````

It happens because the function declaration in JavaScript is hoisted to the top of the scope.
(They are created in the creation stage of the Activation / Variable object)

Hoisting affects differently for different folks:

- variable declarations: using var, let or const keywords
- function declarations: using function <name>() {...} syntax
- class declarations: using class keyword

#### variable declarations : (They are moved to the top of the function scope, but there assignment is not)

So,
- if a variable is accessed / used before it is assigned, it will be undefined
- If a variable is accessed / used before it is declared (or not declared rather), it will be be a reference error

In the following code,

`````javascript
function sum(a, b) {
  console.log(myString); // => undefined
  var myString = 'Hello World';
  console.log(myString); // => 'Hello World'
  return a + b;
}
sum(16, 10); // => 26
`````

is actually understood by JS as below

`````javascript
function sum(a, b) {
  var myString;
  console.log(myString); // => undefined
  myString = 'Hello World';
  console.log(myString); // => 'Hello World'
  return a + b;
}
sum(16, 10); // => 26
`````

#### Block scope variable declarations (Rough Hoisting & Temporal dead zones)

Upto ES5, there was only global & function scope, in JS. With ES6, JS lets you have block scope just as in other languages via the let keyword.

So, rules are :

1) let expansion in the entire block protects variables from modification by outer scopes, even before declaration.

2) Generate reference errors when accessing a let variables in temporary dead zone

So,
- if a variable is accessed / used before it is assigned, it will be undefined
- If a variable is accessed / used before it is declared (or not declared rather), it will be be a reference error

A simple example

`````javascript
if (true) {
  // Declare name block variable
  let month;
  console.log(month); // => undefined
  // Declare and initialize year block variable
  let year = 1994;
  console.log(year); // => 1994
}
// name and year or not accessible here, outside the block
console.log(year); // ReferenceError: year is not defined
````````

**Answer:**
In the 1st case, month is undefined because its declared, but used before its initialized.
Whereas for year there is no reference declared & throws reference error.

**let** variables are registered at the top of the block. (roughly)

But when the variable is accessed before declaration, JavaScript throws an error: ReferenceError:  <variable> is not defined.
From the declaration statement up to the beginning of the block the variable is in a temporal dead zone and cannot be accessed

Lets see an example :

`````javascript
function isTruthy(value) {  function scope here
  var myVariable = 'Value 1';
  if (value) { block scope here
    /**
     * temporal dead zone for myVariable
     */
    // Throws ReferenceError: myVariable is not defined
    console.log(myVariable);
    let myVariable = 'Value 2';
    // end of temporary dead zone for myVariable
    console.log(myVariable); // => 'Value 2'
    return true;
  }
  return false;
}
isTruthy(1); // => true
`````

**Answer:**
- myVariable is in a temporal dead zone from let myVariable line up to the top of the block if (value) {...}.
- If trying to access the variable in this zone, JavaScript throws a ReferenceError. WTF ?
- An interesting question is: Then, is really myVariable hoisted up to the beginning of the block ?
- Because, we said, let variables declarations are registered to the top of the block.
- We saw an example where it was used after it was declared, but before it was initialized & so was undefined .
- But as per the rule, if it is really hoisted it means its declared. So if accessed, it should not throw a reference error... Why then ? What is happening ?
- If you take a look at the beginning of the function block, var myVariable = 'Value 1' is declaring a variable for the entire function scope.
- In the block if (value) {...}, if let variables did not cover the outer scope variables, then in the temporal dead zone myVariable would have the value 'Value 1', which does not happen.
- So block variables are **ROUGH HOISTED** . => they cover up for their outer variables, but are not undefined, but end up having a reference error if accessed.

In an exact description, when the engine encounters a block with let statement,
- first the variable is declared at the top of the block.
- At declared state the variable still cannot be used,
- but it covers the outer scope variable with the same name.

Later when let myVar=  line is passed, the variable is in initialized state and can be used.

#### Rules for let :
- let expansion in the entire block protects variables from modification by outer scopes, even before declaration.
- Generate reference errors when accessing a let variables in temporal dead zones.

#### const variable declarations & hoisting :

**The temporal dead zones apply to const as well.**

So, const hoisting has the same behavior as the variables declared with let statement
Besides, that, 2 rules are :

- *When a constant is defined, it must be initialized with a value in the same const statement.*
- After declaration and initialization, the value of a constant cannot be modified

`````javascript
const PI = 3.14;
console.log(PI); // => 3.14
PI = 2.14; // TypeError: Assignment to constant variable
`````

**Regarding Hoisting for const:**

`````javascript
function double(number) {
   // temporal dead zone for TWO constant
   console.log(TWO); // ReferenceError: TWO is not defined
   const TWO = 2;
   // end of temporal dead zone
   return number * TWO;
}
double(5); // => 10
``````

If TWO is used before the declaration, JavaScript throws an error **ReferenceError: TWO is not defined.**
So the constants should be first declared and initialized, and later accessed.

#### Hoisting differences function expressions vs function declarations
Difference between a function declaration function <name>() {...} and a function expression var <name> = function() {...}.
Both are used to create functions, however have different hoisting mechanisms.

`````javascript
// Call the hoisted function
addition(4, 7); // => 11

// The variable is hoisted, but is undefined
substraction(10, 7); // TypeError: substraction is not a function

// Function declaration
function addition(num1, num2) {
   return num1 + num2;
}

// Function expression
var substraction = function (num1, num2) {
  return num1 - num2;
};
``````


**Answer:**
addition is hoisted entirely and can be called before the declaration. This is because, the property which holds a reference to the function declaration in memory is created in the creation stage of AO/VO object itself However substraction is declared using a variable statement (see 2.) and is hoisted too, but has an undefined value when invoked, because it follows the same variable declarations (only variable declaration is hoisted, but assignment is not) rule. This scenario throws an error: TypeError: substraction is not a function.

#### CLass declarations & hoisting

In ES6, The class keyword is used to simulate a class. Classes originally in ES5, were simulated using functions as objects via Prototypal Inheritance to simulate classical inheritance.
As much as we did with the original way of static methods, instance methods, super, extends etc., can now be done using class.
Classes are built on top of the JavaScript prototypal inheritance and have some additional stuff like super (to access the parent class), static (to define static methods), extends (to define a child class) and more

Example

`````javascript
class Point {
   constructor(x, y) {
     this.x = x;
     this.y = y;
   }
   move(dX, dY) {
     this.x += dX;
     this.y += dY;
   }
}
// Create an instance
var origin = new Point(0, 0);
// Call a method
origin.move(50, 100);
````

**class variable declarations also follow rough hoisting as in let or const.**

Rule

- The class variables are registered at the beginning of the block scope.
- But if you try to access the class before the definition, JavaScript throws ReferenceError: <name> is not defined.
- It will not throw undefined. Reference error is an exception

So the correct approach is first to declare the class and later use it to instantiate objects.

incorrect usage

`````javascript
// Use the Company class
// Throws ReferenceError: Company is not defined

var apple = new Company('Apple');
// Class declaration
class Company {
  constructor(name) {
    this.name = name;
  }
}
// Use correctly the Company class after declaration
var microsoft = new Company('Microsoft');
````

#### class expressions
Same as function declarations & function expressions, we have Class declarations & class expressions.

`````javascript
// Use the Sqaure class
console.log(typeof Square);   // => 'undefined' since only the declaration is hoisted & not assignment
// *Throws TypeError: Square is not a constructor *
var mySquare = new Square(10);
// Class declaration using variable statement
var Square = class {
  constructor(sideLength) {
    this.sideLength = sideLength;
  }
  getArea() {
    return Math.pow(this.sideLength, 2);
  }
};
// Use correctly the Square class after declaration
var otherSquare = new Square(5);
````

**Answer:**
The class is declared with a variable statement var Square = class {...} .
The variable Square is hoisted to the top of the scope (but is not assigned), but has an undefined value until the class declaration line.
So the execution of var mySquare = new Square(10) before class declaration tries to invoke an undefined as a constructor and JavaScript throws TypeError: Square is not a constructor.

> This completes the comprehensive list of usecases for learning hoisting in JS
