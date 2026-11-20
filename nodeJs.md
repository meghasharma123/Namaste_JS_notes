# Basics

## Fundamentals

Nodejs is an open-source JavaScript runtime environment that allows JavaScript to run outside the browser, typically on servers.

It is built using:
- V8 JavaScript engine
- C/C++ components
- libuv for asynchronous I/O and event loop functionality

Nodejs is single threaded, but there's an important distinction:
- JavaScript execution generally happens on a single main thread.
- Node.js uses libuv's thread pool for certain operations.
- It also supports worker threads for CPU-intensive tasks.


*V8 is Google's open-source JavaScript engine, written primarily in C++, which compiles JavaScript into machine code for execution.*

**Node.js good for I/O-intensive applications because Node.js uses non-blocking I/O.**

For example, when an API requests data from a database:
1. Node.js initiates the database operation.
2. It doesn't block the main JavaScript thread while waiting.
3. It can handle other requests.
4. Once the operation completes, its callback or Promise continuation is processed.

## Node.js architecture

1. V8 Engine
- Google's JavaScript engine, written primarily in C++.
- Compiles and executes JavaScript.
- Handles memory allocation and garbage collection.

2. libuv
- C library used by Node.js.
- Provides asynchronous I/O operations.
- Manages the event loop, thread pool, timers and other OS-level operations.

3. Event Loop
- Coordinates the execution of asynchronous callbacks.
- Processes callbacks through different phases.
- Allows Node.js to handle asynchronous operations without blocking the main JavaScript thread.

JavaScript code -> V8 engine [Executes JavaScript] -> libuv + Event Loop [Coordinates asynchronous operations and callbacks] -> OS / Thread Pool [Handles I/O and certain background operations]

### Event loop phases

1. Timers Phase - responsible for setTimeout() and setInterval()
2. Poll Phase - responsible for retrieving new I/O events, executing I/O-related callbacks, and handling other events such as reading from file descriptors.
3. Check Phase - The check phase is where callbacks scheduled by setImmediate() are executed. This phase runs after the poll phase.
4. Close Callbacks Phase - This phase is responsible for executing callbacks associated with the close event. This usually involves clean-up tasks after a stream or socket is closed.

`Microtasks: process.nextTick and Promise Callbacks`

## Thread pool

- It is a pool of background threads used by Node.js to execute certain operations without blocking the main JavaScript thread.
- The default libuv thread pool size is 4
- Common examples:
  - File system operations (fs)
  - DNS lookup using dns.lookup()
  - Crypto operations such as crypto.pbkdf2()
  - Zlib compression operations
- we can increase the size of thread pool by using `UV_THREADPOOL_SIZE=8 node app.js`

## Worker Threads vs Cluster

1. `Worker Threads`
- Run JavaScript in separate threads.
- Each worker has its own V8 instance and JavaScript execution environment.
- Useful for CPU-intensive tasks.

Example: Generating a large report, image processing, or complex calculations.
```
const { Worker } = require("worker_threads");
const worker = new Worker("./worker.js");
```

2. `Cluster`
- Creates multiple Node.js processes.
- Each process runs its own JavaScript event loop.
- Useful for utilizing multiple CPU cores to handle incoming requests.

Example: Running an Express server across multiple CPU cores.
```
const cluster = require("cluster");
const os = require("os");

if (cluster.isPrimary) {
  for (let i = 0; i < os.cpus().length; i++) {
    cluster.fork();
  }
} else {
  require("./server");
}
```

3.`Child Processes`
- Allow Node.js to create and communicate with separate OS processes.
- Each process has its own memory space.
- Useful for executing external commands or independent programs.
```
Example:
const { exec } = require("child_process");

exec("ls", (error, stdout) => {
  console.log(stdout);
});
```

Feature |	Worker Threads |	Cluster | 	Child Processes
------|-------|------------|----------|
Execution |	Separate threads |	Separate processes |	Separate processes
Memory |	Can share memory |	Separate |	Separate
Best for |	CPU-intensive JS |	Scaling servers	| External commands
Communication |	Messages/shared memory |	IPC | 	IPC

Remember these three:
- Worker Threads → Heavy calculations.
- Cluster → Multiple server processes.
- Child Processes → Run another process/program.

Suppose you have a Node.js application:
1. An API generates a large PDF report and calculations take 10 seconds. Which mechanism would you use? -> worker thread
2. You want your Express server to utilize all 8 CPU cores. Which mechanism would you use? -> Cluster
3. You want to execute a Python script from Node.js. Which mechanism would you use? -> child process

## Middleware

Middleware is a function that executes during the request-response cycle.

It has access to:
- req – incoming request
- res – response object
- next – passes control to the next middleware

Common use cases:
- Authentication
- Logging
- Request validation
- Error handling
- Rate limiting

**What happens if we don't call next()?**

- If middleware sends a response using res.send(), the request finishes.
- If it neither sends a response nor calls next(), the request hangs until a timeout or connection failure. It doesn't automatically throw an error.

### Types of Middleware

1. **Application-level middleware**
Middleware registered directly on the Express application using app.use() or app.METHOD(). Used for logging, authentication, request processing, etc
```
app.use((req, res, next) => {
  console.log("Request received");
  next();
});
```

2. **Router-level middleware**
Middleware attached to a specific Express router instead of the entire application. Useful when you want authentication only for certain routes.
```
const router = express.Router();

router.use((req, res, next) => {
  console.log("User routes");
  next();
});

router.get("/profile", getProfile);
```

3. **Error-handling middleware**
Used to catch and handle errors in Express applications. Important: It receives four parameters.
```
app.use((err, req, res, next) => {
  console.error(err.message);

  res.status(500).json({
    message: "Internal server error"
  });
});
```

4. **Built-in middleware**
Express provides middleware out of the box. Common examples:
```
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static("public"));
```

## Error Handling in Express.js.

























2. Node.js & Backend Engineering
- Node.js architecture and event loop
- Express.js / middleware
- REST API design and HTTP methods
- Authentication: JWT, OAuth2, RBAC
- Error handling and centralized logging
- Rate limiting, caching, pagination
- Streams, buffers, worker threads
- Asynchronous programming and concurrency

Topics roadmap
1. Node.js fundamentals and architecture
2. Event loop, libuv and thread pool

3. Modules: CommonJS vs ES Modules
4. Async programming and error handling
5. Express.js and middleware
6. REST API design and authentication
7. Streams, buffers and file handling

8. Worker threads, clustering and child processes
9. Performance, scalability and memory leaks
10. Security, testing and production debugging

