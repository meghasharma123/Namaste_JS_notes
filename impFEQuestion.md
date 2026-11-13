# PubScale

## 1. px vs em vs rem differnce

**px (Pixels)**
- Absolute CSS unit.
- Doesn't depend on parent or root font size.
- Useful for borders, small fixed dimensions, etc.

**em**
- `Relative to the font size of the element's parent` for properties like font-size.
- For other properties like padding, it is relative to the element's own computed font size.
- Can compound through nested elements.

**rem (Root em)**
- Relative to the `root (html) font size`.
- More predictable than em because it doesn't compound with nesting.

## 2. How is CSS specificity calculated?

 * Inline styles - 1000
 * ID selectors (#id) - 100
 * Classes, attributes, pseudo-classes - 10
 * Elements, pseudo-elements - 1

### Note

- !important overrides normal declarations (but cascade layers and importance rules also matter).

- If specificity is equal, the rule appearing later generally wins.

- *(universal selector) has zero specificity.

- Inline styles generally beat ID/class/element selectors.

- :hover, :focus, :nth-child() have class-level specificity.

- :where() has zero specificity.

- :is(), :not(), and :has() take the specificity of their most specific argument.

```
#nav .item a { color: red; }
.menu li.item a:hover { color: blue; }
a { color: green !important; }
```

Output -> green 

## 3. Explain the box model. 

1. `Content`: Actual text, image, or element content.
2. `Padding`: Space between content and border.
3. `Border`: Surrounds padding and content.
4. `Margin`: Space outside the border.

### A box is width: 200px, padding: 20px, border: 5px. How wide does it render with content-box vs border-box?

`content-box`: Content is 200px, and padding/border are added outside.

`border-box`: Everything must fit inside 200px.

`Total width = Content + Padding + Border`

200 + 40 + 10 = 250px

`total width = width - padding - border`

200 - 40 - 10 = 150px

## 4. Center a div horizontally and vertically without flex or grid. Give two ways.

1. Absolute positioning with transform: translate(-50%, -50%)

```
.parent {
  position: relative;
  height: 100vh;
}

.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

2. CSS Table / Table-cell

```
.parent {
  display: table;
  width: 100%;
  height: 100vh;
}

.child {
  display: table-cell;
  text-align: center;
  vertical-align: middle;
}
```

## 5. What is a closure? Give a real use from your own code.

- A closure is a combination of a function and its lexical environment.

- A closure allows a function to remember and continue accessing variables from the scope in which it was created, even after that outer function has finished executing.

```
function createSimulationHandler() {
  let latestPosition = { x: 0, y: 0, z: 0 };

  return function (newPosition) {
    latestPosition = newPosition;

    console.log("Latest position:", latestPosition);
  };
}

const handleSimulationUpdate = createSimulationHandler();

// Called whenever WebSocket receives new data
handleSimulationUpdate({ x: 10, y: 20, z: 30 });
handleSimulationUpdate({ x: 15, y: 25, z: 35 });
```

* latestPosition is private to the outer function.
* The returned function remembers and updates it.
* No global variable is required.

## 6. What is an IIFE (Immediately Invoked Function Expression) ? why use it?

- It is a JavaScript function that executes immediately after it is defined.
- We traditionally use it to create private scopes, avoid global namespace pollution, encapsulate variables, and perform one-time initialization. Nowadays, ES modules and modern JavaScript scoping have reduced its usage.
```
(function () {
  var x = 1;
  console.log(x);
})();
```

## 7. map vs forEach?

- Both iterate over an array, but their main difference is the return value.
- map returns an array whereas foreach returns undefined.
- forEach is used when we don't want the transform array back.

## 8. forEach vs a for loop ?

- Break/continue -> not supported in forEach while supported in for loop.
- Return -> undeined in forEach while depends on function
- Async/await -> not wait for async callbacks in forEach while can use in for loop.

## 9. Event bubbling and event capturing

- Event bubbling means when an event occurs on a child element, it propagates from the target element upward to its parent elements.
- To stop this we use `event.stopPropagation()`
- To perform event capture `onClickCapture`
- Bubbling is default behaviour

Event propagation has three phases: `capturing, target, and bubbling`. During capturing, the event travels from the root towards the target. At the target, the event handler executes. During bubbling, it travels back from the target to its ancestors. In React, we use onClickCapture for capturing and onClick for bubbling.

`stopPropagation()` → Stops the event from propagating.
`preventDefault()` → Prevents the browser's default action, such as navigating through a link or submitting a form.

## 10. What happens when you type a URL and press Enter?

`URL parsing` - Browser parses the URL into protocol (https), domain, path, query parameters, etc.

`DNS lookup` - Resolves the domain name (e.g. google.com) into an IP address. Browser, OS and DNS caches may be checked first.

`Establish connection` - Browser establishes a connection with the server:
- TCP handshake for HTTP/1.1 and HTTP/2.
- QUIC for HTTP/3.
- TLS handshake for HTTPS.

`Send HTTP request` - Browser sends an HTTP request containing method, headers, cookies and other relevant information.

`Server processing` - Server processes the request, communicates with databases or other services if needed, and returns an HTTP response.

`Receive and parse HTML` - Browser parses HTML and creates the DOM (Document Object Model).

`CSS parsing` - Browser parses CSS and creates the CSSOM (CSS Object Model).

`Render tree creation` - Browser combines DOM and CSSOM to create the render tree. Invisible elements such as display: none are excluded.

`Layout` - Calculates the position and dimensions of elements.

`Paint and compositing` - Browser paints pixels (text, colors, borders, images) and composites layers to display the final page.

**DOMContentLoaded**: HTML has been parsed and deferred scripts have executed. It doesn't wait for images and other resources to finish loading.

**window.onload**: The page and its dependent resources have finished loading.

## 11. Status code

- 200	OK	-> Request successful; response contains data.
- 201	Created -> New resource successfully created (e.g., user registration).
- 204	No Content -> Request successful, but no response body (e.g., delete operation).
  
- 301	Moved Permanently -> Resource permanently moved to another URL.
- 302	Found (Temporary Redirect) -> Resource temporarily available at another URL.
- 304	Not Modified -> Cached resource hasn't changed; browser can use its cached copy.

- 400	Bad Request -> Invalid request or malformed data.
- 401	Unauthorized -> Authentication is missing or invalid.
- 403	Forbidden -> User is authenticated (or identified), but lacks permission.
- 404	Not Found -> Requested resource doesn't exist.

- 500	Internal Server Error -> Unexpected error occurred on the server.

* 2xx → Success
* 3xx → Redirection
* 4xx → Client error
* 5xx → Server error

# 12. CSR vs SSR. When would you choose each? What is hydration?

- In CSR, the browser downloads HTML, JavaScript, and CSS. JavaScript executes in the browser and generates the UI.

**Flow:**
1. Browser requests the page.
2. Server sends minimal HTML and JS bundles.
3. Browser downloads and executes JavaScript.
4. React renders components and updates the DOM.
5. User sees the page.

**Advantages:**
- Great for highly interactive applications.
- Less rendering work on the server.
- Smooth navigation after initial load.

**Disadvantages:**
- Can have slower initial rendering.
- SEO can be more challenging.
- Large JS bundles can affect performance.

**When to use CSR?**
- Admin dashboards.
- Internal tools.
- Analytics platforms.
- Highly interactive applications.

- In SSR, the server generates the initial HTML and sends it to the browser.

**Flow:**
1. Browser requests the page.
2. Server fetches required data.
3. Server renders React components into HTML.
4. Browser receives and displays HTML.
5. JavaScript downloads and hydrates the page to make it interactive.

**Advantages:**
- Faster initial content visibility in many cases.
- Better SEO support.
- Useful for content-heavy websites.

**Disadvantages:**
- More server processing.
- Requires handling server/client differences.
- Hydration adds JavaScript execution overhead.

**When to use SSR?**
- E-commerce websites.
- Blogs and news websites.
- Marketing websites.
- Public-facing pages where SEO matters.

`Hydration` is the process where React attaches event handlers and initializes client-side behavior to HTML that was already rendered by the server, making the page interactive.

# 13.  Which web performance metrics do you track? What is a good value for each?

For frontend performance, I primarily track Core Web Vitals, along with page-load and JavaScript performance metrics.

Core Web Vitals measure three important things:
1. Loading performance.
2. Interactivity.
3. Visual stability.

`LCP – Largest Contentful Paint` Good: ≤ 2.5s
- Measures how quickly the largest visible content element loads. Examples: Hero image, banner or main heading.

`INP – Interaction to Next Paint` Good: ≤ 200ms
- Measures responsiveness to user interactions throughout the page. Examples: Clicking buttons, typing, opening dropdowns.

`CLS – Cumulative Layout Shift` Good: ≤ 0.1
- Measures unexpected layout shifts while the page loads. Example: An image loads without reserved dimensions and pushes content down.

These are Google's Core Web Vitals thresholds, evaluated at the 75th percentile (p75), separately for mobile and desktop.

*What tools do you use?*

Lighthouse ->	Performance audit and recommendations
Chrome DevTools ->	Network, Performance and memory profiling
PageSpeed Insights -> 	Real-world Core Web Vitals + lab testing
Web Vitals library ->	Collect real-user performance metrics
React Profiler -> 	Identify unnecessary React renders

*How do you improve these metrics?*

LCP ->	Optimize images, preload critical assets, CDN, reduce TTFB
INP ->	Reduce JavaScript execution, avoid long tasks, optimize React renders
CLS -> 	Set image dimensions, reserve space for ads, avoid layout shifts
FCP -> 	Reduce render-blocking CSS/JS, optimize critical CSS
TBT ->	Code splitting, lazy loading, reduce main-thread work
TTFB -> Backend optimization, caching, CDN

# 14. Fetch vs axios

Fetch is a native browser API, whereas Axios is an HTTP client library. Axios provides automatic JSON transformation, better default error handling for HTTP errors, interceptors, built-in timeout configuration and upload/download progress tracking. In larger applications, I generally prefer Axios because interceptors help centralize authentication and error handling. For simple API calls, Fetch is sufficient.

```
axios.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");

  config.headers.Authorization = `Bearer ${token}`;

  return config;
});
```

*axios*
```
async function getUsers() {
  try {
    const response = await axios.get("/api/users");

    return response.data;
  } catch (error) {
    console.log(error);
  }
}
```

*fetch*
```
async function getUsers() {
  try {
    const response = await fetch("/api/users");

    if (!response.ok) {
      throw new Error("API failed");
    }

    const data = await response.json();

    return data;
  } catch (error) {
    console.log(error);
  }
}
```

|Feature|Fetch|Axios|
|-----|----|----|
|Installation|Built into browsers	|Requires installation|
|JSON parsing|	Manually use .json()|	Automatically parses JSON responses|
|Error handling|	HTTP 4xx/5xx don't reject automatically|	Rejects non-2xx responses by default|
|Interceptors|	Not built in	|Built-in request/response interceptors|
|Request timeout	|Use AbortController	|Built-in timeout option|
|Request cancellation|	AbortController|	Supports AbortController
|Upload/download progress|	Limited browser API support|	Built-in progress callbacks
|Default headers	|Basic|	Convenient defaults and configuration
|Request/response transformation|	Manual|	Built-in transformation

# 15. How do you detect that the user has no internet?

I use navigator.onLine for basic connectivity detection and listen to the browser's online and offline events. However, it isn't completely reliable because a device can be connected to Wi-Fi without having internet access. So, for production applications, I also make a lightweight health-check request to the backend to verify actual server connectivity.

```
if (navigator.onLine) {
  console.log("User is online");
} else {
  console.log("User is offline");
}
```

# 16. What does useState return?

- useState returns a two-element array: the current state and a setter function used to update that state.

# 17. in short which log will be printed when

```
useEffect(() => {
  console.log("1");
  return () => console.log("2");
}, []);
```
- On mount: 1 is printed (effect runs).
- On unmount: 2 is printed (cleanup runs).

useEffect cleanup runs before the effect executes again when dependencies change, and when the component unmounts. In development, React Strict Mode can also invoke cleanup during its extra setup-cleanup cycle. We use it to prevent memory leaks and clean up subscriptions, timers and event listeners.

A full page refresh destroys the current JavaScript execution context. React cleanup generally runs during page teardown, but don't rely on it for critical operations like saving data to a server.

# 18. Why are keys important in lists? Why is the array index a bad key?

- Keys help React identify which items have changed, been added, or removed during reconciliation.
- index are bad key Because when items are added, removed, or reordered, indexes change or get reused.

"Keys help React uniquely identify list elements during reconciliation. Array indexes are bad keys when lists can change because indexes are not stable and can cause incorrect state association and rendering bugs. We should prefer unique, stable IDs."

```
Index 0 → Alice
Index 1 → Bob
Index 2 → Charlie
```
After removing Alice:
```
Index 0 → Bob
Index 1 → Charlie
```

# 19. useRef

- useRef is a React Hook that stores a mutable value between renders without triggering a re-render when the value changes.

*Real-world use cases*

- Accessing DOM elements.
- Storing timer IDs (setInterval).
- Maintaining previous values.
- Managing WebSocket instances.
- Preventing unnecessary re-renders for mutable values.

useState is for values that affect UI; useRef is for values that need to persist but don't need to trigger UI updates.

# 20. Code

```
function main() {
    console.clear();
    for (let i = 0; i < 10; i++) {
        Task(i);
    }
}

main();

//Task Function
function Task(taskId) {
    return new Promise((resolve) => {
        setTimeout(() => {
            console.log("Task Id completed", taskId);
            resolve(taskId);
        }, Math.random() * 1000);
    });
}
```

**Promise chaining ensures sequential execution by returning each asynchronous operation's Promise from .then().**

```
function main() {
    console.clear();
    let x = Promise.resolve()
    for (let i = 0; i < 10; i++) {
        x = x.then(() => Task(i))
    }
}
```

so here without using async await we have solved the issue of sequence in order.