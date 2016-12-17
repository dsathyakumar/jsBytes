# PART -1 : The problem

### The problem of Recursion in Javascript :

- In JS, if a function calls itself recursively then the JavaScript engine has to create what's called a new ‘stack’ for every such recursive call.
- As mentioned in the article on execution context, every new function invocation will create a new function context.
- Such a function context creates its own private scope.
- Every such function invocation which creates its own private scope, causes an entry to be placed in the execution context stack.
- For every entry placed in the execution context stack, an execution context object is created with details on arguments object, scope chain & VO & this object.
- A stack is a chunk of memory allocated to help keep track of all the information related to the function at the point of execution (such as its arguments and their initialized values).
- Here in lies the problem: creating stacks is expensive, as the JS engine can only create as many stacks as it has memory available.
- If we write a function that recursively calls itself many times, then we'll find that we can exhaust the memory allocation and trigger an error.

### Let's look at some code that demonstrates this problem.

The following function sum simply adds 2 numbers together.
But for the purpose of demonstrating recursion issues, and making it easier to understand, the sum function which could have been written in a straight forward approach has been written in a recursive way :

`````javascript
function sum(x, y) {
     if (y > 0) {
          return sum(x + 1, y - 1);
     } else {
          return x;
     }
}
sum(1, 10);
`````````````````

### The code flow:

- While calling sum, we pass in 2 numbers which are assigned to the parameters x and y (in this instance we pass 1 and 10) // sum(1, 10);

- We check if (y is greater than 0). If y is greater than 0 (which we know it is) then, we recursively call sum again but, if (y > 0) {
     this time modify the arguments so that x is incremented by 1 (x+1) and y is reduced by 1 (y-1) // sum(x + 1, y - 1);

- When the sum function is next called we're passing 2 and 9.

- At this point we're now inside a new invocation of the sum function, but the first call to sum has yet to finish (as we didn't reach the end of the function,
     the focus moved to another function being executed - which in this case was sum again)

- Also at this point the JS engine has 2 stacks. One for the point in time when we passed in arguments sum(1 and 10), and now (as it has to remember 2 and 9).

- The JS engine has to remember the previous arguments 1 and 10 because, once this second sum execution finishes, the previous call will be popped out of the
      stack and the control will go back to the previous entry in the stack, and we'll end up back in the first execution context.

- As we can see, in the above code the JS engine has to create a new stack for each recursive call.

For a small sum of 1 and 10 this is all fine;
Now assume if we try to sum (1, 100000) then that'll require more stacks to be created, than the memory that has been allocated to us for stacks creation.

This will cause an error, if we create a stack deep enough (such as we did with sum(1, 100000)) then the
JS engine will throw a **Maximum call stack size exceeded error**.
This problem can occur very easily.

### Don’t other languages face this issue ?

In other programming languages the recursion could be rewritten in such a way that the engine would recognize a recursive execution was happening and optimize the code
internally into a loop form. This is called a "tail call optimisation" (TCO).

Unfortunately the JavaScript doesn't implement this optimization. Note: ES 6 has a spec mentioned for TCO implementation.

### So what would we do to solve this ?

- The problem again is the way we have approached Recursion.
- Recursion in the form we've written it above requires many stacks to be created because of how the function is calling itself.

````javascript
function sum(x, y) {
     if (y > 0) {
          return sum(x + 1, y - 1);
     } else {
          return x;
     }
}
sum(1, 10000);
`````


### To solve this, we need to understand the pattern that recursion gives rise to:

- The first call to the sum function (i.e. sum(1, 100000)) will not complete until the very last recursive call to sum completes.
- The very last call completes only when instead of re-invoking the sum() function again, a value is returned which is x.
- In the above example, the value x is returned only at the last call. (i.e. sum(0, 99999) which is the final execution).
- When the final call to sum occurs and we find that y is no longer greater than zero, we return the lastly accumulated value which has been stored inside the argument x.
- Now this returned value of x has to pass back to each of the recursive function callers back up to the first function call of sum(1, 100000)
- That is, the returned value needs to then be passed back through each function execution context (effectively closing or popping out each recursive call entry / each stack entry of the execution context stack) until we reach the very first function execution context that was opened when we ran sum(1, 100000).

## PART -2 : The solution

### How To solve this ?

- We need to create fewer stacks somehow (or) don’t use the recursive approach and go for the iterative one, using loops.
- The pattern is called TRAMPOLINE design pattern
- It involves using Function.prototype.bind. to bind the context (this value) to a function, with its arguments.
- Invoking Function.prototype.bind on a given function with arguments & a this, will create a new bound function

Consider this …

``````javascript
function trampoline(f) {
     while (f && f instanceof Function) {
          f = f();
     }
               return f;
}

function sum(x, y) {
     function recur(x, y) {
          if (y > 0) {
               return recur.bind(null, x + 1, y - 1);
          } else {
               return x;
          }
     }
     return trampoline(recur.bind(null, x, y));
}

sum(1, 10);
``````````

### How does the above approach solve this problem ?

- The reason the above code works is because we've replaced our recursion style execution style with a loop !
- In the above code we don't create a deep nest of stack entries in the execution context.
- First, the sum function only gets called once. (1st entry into the call stack)
- Next, the trampoline function also is only called once; (2nd entry into the call stack)
- Finally, the recur function inside of sum, although called multiple times, is called inside a loop (again, not more than one stack entry is required at any one time inside the while loop iteration)
- For every iteration of the while loop, an entry is placed in the call stack, i.e a bound function (via recur.bind) executes, it completes and is immediately popped out of the execution call stack.
- But, while completing its execution, it returns another bound function. During the next iteration of the while loop, we check if the return type is again a function. If so we execute it. This process repeats until the last call, where instead of a function, a value is returned.
- In this case, we return the value. The trampoline function entry in the call stack is popped and the value is returned to sum. The sum function entries pops out of the call stack and the value is returned to the main flow.

### The detailed code breaks down as below :-

- We call sum(1, 10).

- Our sum function ultimately returns a value. In this case whatever is returned by calling the trampoline function.

- The trampoline function accepts a reference to a function as its argument
     it’s important to understand it needs a reference to a function;
     Because doing return trampoline(recur(x, y)) wouldn't work as that would end up passing the result of calling recur(x, y) to the trampoline function.

- So we use Function#bind to pass a reference to the recur function.
     We use null as the this binding because it doesn't matter what the context the function executes in as we don't use the function as a constructor here in this case.

- When we execute sum(1, 10) we pass the reference to the internal recur method to the trampoline function, as input argument.

- The trampoline function checks if we passed a function and if so, executes the function and store its return value inside the f variable.

- If what we pass into the trampoline function isn't a function then we assume it’s the end (i.e. accumulated) value and we return the value straight back to the sum
function which returns that value as the accumulated value.

- Inside the recur function we check to see if y is greater than zero, and if it is we modify the x and y values to x+1 and y-1 and then return another function reference to the
      recur function but this time with the modified x and y values, that is, x+1 and y-1.

- Inside the trampoline function, the newly referenced function is assigned to the f variable and the while loop on its next iteration checks to see if f is indeed a
function or not.

- If it is (which it would be in this instance) we again execute the function (which is now recur(2, 9)) and the whole process starts again.

- This goes on till we reach the point where y is set to zero.

- Then when the trampoline function executes the function reference (recur), inside the next while loop iteration, and inside the if conditional fails
     (as y is now zero and no longer greater than zero) and so we return the accumulated x value, instead of a function reference; which then gets sent back and stored in the f
     variable inside the trampoline function.

- On the next iteration of the while loop, f is now a value and not a function and so the while loop ends and the accumulated value is returned to the sum function
     which returns that as its accumulated value.

## PART -3 : A generic pattern (which implements trampolining to realize tail call optimization. It wraps any code into itself)

The previous code works fine, but it required to modify our code to work with the 'trampoline' pattern. This is a bit of a pain and means if we have lots of
recursive code then it means each one might need subtle changes to accommodate this pattern, which is not very good.
The following code is an abstraction around that concept and it'll allow us to keep our code exactly the same, with no modifications, and the abstraction will handle all of the work for us!

// this is our generic trampoline pattern to tail call optimize any recursive function
// it takes in the required function (which has recursive logic) and returns a accumulator function for our usage
`````javascript
function tco(f) {
    var value;
    var active = false;
    var accumulated = [ ];
    return function accumulator() { // ————————— (1) is a closure that has access to value, f, active and accumulated
        accumulated.push(arguments);
        if (!active) {
                 active = true;
                 while (accumulated.length) {
                          value = f.apply(this, accumulated.shift());
                 }
                 active = false;
                 return value;
        }
    }
}

// this is our recursive function logic which gets passed as a parameter to the tail call optimizer function mentioned above
// the tail call optimizer is invoked with this function as input and it returns a accumulator function which is marked (1) above
// the value returned by the tail call optimizer function, which is again a function called accumulator, marked by (1) above is assigned to the variable sum
// variable sum now has a function reference to accumulator
var sum = tco(function(x, y) {
    if (y > 0) {
        return sum(x + 1, y - 1)
    } else {
        return x
    }
});

// we now invoke the function referenced by sum variable, which is basically the function, marked as (1) above and pass it with parameters.
sum(1, 100000)
````````

### EXPLANATION:

- We store the result of calling tco (wrapped around our recursive function logic) into the sum variable, which now has the accumulator function, which is the
     result of tco function execution.

- The sum variable is now a function expression that when called (e.g. sum(1, 10)) will execute the accumulator function that tco returned.

- The accumulator function is a closure (meaning when we call sum the accumulator will have access to the variables defined inside of tco -> value, active and
      accumulated; as well as our own code which is accessible via the identifier f).

- When we call sum for the first time (e.g. sum(1, 10)) we indirectly execute accumulator.

- The first thing we do inside of accumulator is push the arguments object (an Array-like object) into the accumulated Array (so the accumulated[ ] will now
     have a length of 1).

- It's worth knowing that the accumulated Array always has a max of 1 item inside of it at any point in time. And is also symbolic to the fact that although we
     recursively execute, we always create only one stack entry at any point in time within the while loop.

- The active variable by default is false. So as accumulator is called for the first time, we end up inside the if conditional, and then reset active to true.

- Now we get to a while loop (we're still calling functions recursively, as you'll see in a moment; but this is a very clean loop compared to an ugly for loop with lots
     of operators/operands).

- The while loop simply checks whether the accumulated Array has any content. If it does then we call the f function (our recursive logic function) and pass
     through the arguments that were inside accumulated[0] (for the first run of this function that would've been [1, 10]).

- When we pass the contents of accumulated[0] we use the shift Array method to actually remove it from the accumulated Array so it now has a length of zero.

- If we ignore for a moment the code inside the while loop; on the next iteration of this while loop the condition of accumulated.length will fail and so we would
     end up setting active to false and ultimately return to sum the value of the variable value. But, this won’t happen. This is where it gets confusing! Hold on!

- The f function is our own code. Our own code which is invoked by the accumulator the calls the sum function again (which indirectly calls the accumulator
     function again).

- If our code returns x then the while loop will stop. Lets get to this case later.

- If our code can't return x (notice our code has a conditional check to see if y is greater than zero, if it is then we return x, otherwise...)
     we call sum again and pass through modified arguments, which internally re-invokes accumulator again.

- This time when we call sum & accumulator in turn, we don't get inside of the if-conditional because active is already set to true.

- So the purpose of calling sum & accumulator in turn, from inside our own code is simply to allow us to store the newly modified arguments inside the
     accumulated Array. That is in this case we would store [2,9]

- The sum function then returns undefined (as we weren't able to move into the if conditional)

- The flow of control then throws us back into the original while loop inside our original accumulator call (that's still going, it hasn't stopped yet) and undefined is
     what's stored into the value variable.

- At this point the accumulated Array has once again got some content and so the while loop's condition passes once more and the loop iterative execution
     continues.

- Now, at some point our code is going to call sum with the y argument set to zero meaning that when the accumulator function calls our
     code the condition y > 0 will fail and so we return the value of x (which has been incremented each time along the way).
     No extra arguments are pushed into the accumulated array. This is because, sum is not called and therefore, in turn accumulator is not called.

- When that happens, x is returned and assigned as the value to the value variable, and so our code never called sum and thus the
     accumulated Array is never modified again; meaning the while loop condition inside the accumulator function will fail and thus the accumulator function
      will end returning whatever value is stored inside the value variable (which in this example is the value of x).

**You can now perform recursion in JS without any issues with the call stack !!**
*Note* : The trampoline pattern was first written by *Irakli Gozalishvili*
