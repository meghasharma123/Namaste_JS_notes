# Basics

- Everything in JS happens inside the `Execution Context`

- Execution Context -> `Memory creation + code exectuion` (Variable environment + thread of execution)

- JS is a `synchronous single-threaded` language.

- On every function call an Execution context is created and on return statement, this will be deleted.

- `Call stack` gets value as global execution context whenever a program is created and then whenever a function is called, an execution context is pushed inside the stack and on return the EC is popped out of the stack.

- `Call stack maintains the order of execution of execution context`

- Call stack is also known as Program stack, control stack, runtime stack, machine stack etc.

# Variables and functions

- `Hoisting` using the variables and functions before its decalared.

- so in the memoty allocation phase the variables get `undefined` value where as the functions get its definition.

- every function first check the variable in its execution context first and if not found then check for global context. [local and glbal variable concept]

- whenever an execution is created a `this` object has created and in case of browser that is equal to `window` object.

- JS is a `loosely type` language, i.e. a variable can have string or number anything no specific strict rule defined.

- Lexical environment is an internal structure used by the JavaScript engine to map identifiers (variable and function names) to their actual values based on the physical placement of the code. 

- Lexical environment consists of `environment record` and `reference to the outer lexical environment`.

- `let and const` is more stricter than `var`. we cannot redeclare the same varable name in let and const where as this is possible with var. Hoisting is possible in case of var where as let and const are hosted but left uninitialized, they enter a state called the `Temporal Dead zone`. 

- `TDZ` is the period of time from start of the block until the code execution formally reaches the line where the variable is declared.

# Block scope & Closures

- Block is basically a `compound statement`. What variables and functions we can use inside is called block scope.

-  var is decalred in glbal scope where as let and const are block scope.

- illegal shadowing, occurs when you attempt to shadow an outer let or const variable by declaring a var variable with the same name inside a nested block.

1. let and const are block-scoped. They live strictly inside the curly braces {} where they are defined.
2. var is function-scoped (or globally scoped). It completely ignores block structures like if statements or for loops.

- Function along with its lexical environment forms a `closure`.
```
function x(){
    var a = 7
    function y(){
        console.log(a);
    }
    a = 100;
    return y;
}

var z = x();
```
so here whenever z is called we can access that a variable for log as it is inside the lexical scope of y function. Also we get 100 as 'a' is indicating the reference of the variable and not the value, so whatever change will be impacted.

- Closures use: module design pattern, currying, memoize, iterators, setTimeouts etc.

Q. Output is 6,6,6,6...
```
function x(){
    for(var i = 1; i<=5; i++){
        setTimeout(function(){
            console.log(i);
        },i*1000);
    }
}

x()
```

Bcz this timeout function is like forming a closure with the i value so whenever the value of i changes it read that updated value.

to get 1,2,3,4... we can use let instead of var, so that every closure function has its on block scope.

or if we have to use only var then we can do it like this way:

```
function x(){
    for(var i = 1; i<=5;i++){
        function close(x){
            setTimeout(function(){
                console.log(x);
            },x*1000);
        }
        close(i);
    }
}

x()
```
ok so here close function is called with a new variable value, whereas when we were using i it was showing the same variable, same whiteboard.

### Advantage of Closure

1. Data Privacy(Data encapsulation)

function Counter(){
    var count = 0;
    this.incrementCounter = function(){
        count++
        clg(count)
    }

    this.decrementCounter = function(){
        count--;
        clg(count)
    }
}

var counter1 = new Counter();

counter1.incrementCounter();
counter1.decrementCounter();

### Disadvantage of Closure

1. Memory usage 
2. Debugging difficulty
3. Stale values

# Functions

- function statement: fully hoisted 
function a(){
    clg("hello")
}
- function expression: not histed with its value, only the variable name is hoisted
var b = function xyz(){ // name function expression
    clg("bye")
}

- `first class function` or `first-class-citizen`: the ability of function to be treated like variable i.e. assigned to a variable, passed as an argument to another function, returned from another function, Stored in data structures like arrays or objects. 

- `callback functions`: in js we have only 1 call stack and to perform any operation we have to put the operation in stack so we can't put any heavy or time taking process in this it can block the main thread of execution. at this time callback function come into the picture, function will be callback once its ready for execution.

- `event listners`: a built-in method used to wait for and respond to specific user actions or system events on a web page, such as clicks, keypresses, or page loads. 

# Event loop

It is the core mechanism that enables JavaScript to perform non-blocking, asynchronous operations despite being a single-threaded language.

[ Call Stack ] <--- (Moves tasks here only when stack is empty)
      ^
      |
[ Event Loop ]
      ^
      |-- 1st Priority: [ Microtask Queue ] (Promises, async/await) -> Must drain completely
      |
      |-- 2nd Priority: [ Macrotask Queue ] (setTimeout, Events)    -> Takes one at a time

The Architecture: Key Components

1. Call Stack (LIFO): Where JavaScript tracks active function calls. Functions are pushed onto the stack when invoked and popped off when they return.
   
2. Web APIs / Node.js Host Environment: Background helpers provided by the browser (like setTimeout, fetch, or DOM events) or Node.js runtime. They handle long-running operations concurrently outside the main thread.

3. Microtask Queue (FIFO): A high-priority queue for callbacks originating from Promises (.then, .catch, .finally), async/await continuations, and queueMicrotask().

4. Callback Queue / Macrotask Queue (FIFO): A lower-priority queue for callbacks from timers (setTimeout, setInterval), I/O operations, and user events.

# JS Engine

A JavaScript engine is a specialized computer program that parses, interprets, and executes JavaScript code, transforming human-readable source code into machine code that a computer's CPU can directly run.

### Behind the scene

- Modern engines utilize Just-In-Time (JIT) compilation. Instead of translating the entire script before running it (like C++) or interpreting it completely line-by-line (like old interpreters), a JIT compiler continuously compiles and optimizes the code during execution

- Parsing: The engine takes raw text script and runs it through a parser. The parser breaks it into structural chunks called tokens and maps them into an Abstract Syntax Tree (AST)—a nested tree diagram representing code grammar.

- Compilation & Interpretation:
    • An Interpreter (like V8's Ignition) rapidly converts the AST into unoptimized bytecode to start executing the application immediately without lag.
    • While the bytecode runs, a background profiler watches for "hot code" (frequently run functions or loops)

- Optimization: A Compiler (like V8's TurboFan) grabs that hot code, strips away redundant operations, compiles it directly into high-efficiency machine code, and swaps it back in to accelerate performance.

# higher order functions

- A higher-order function (HOF) is a function that either takes one or more functions as arguments, returns a function as its result, or both.


Method	                             Purpose	
map()	Transforms every item in an array and returns a new array.	
Example: [1, 2].map(n => n * 2) → [2, 4]

filter()	Filters items based on a condition and returns a subset.	
Example: [1, 2, 3].filter(n => n % 2 === 0) → [2]

reduce()	Condenses an entire array down to a single value.	
Example: [1, 2, 3].reduce((acc, n) => acc + n, 0) → 6

forEach()	Iterates over items to execute a side effect.	
Example: ['A', 'B'].forEach(x => console.log(x))
