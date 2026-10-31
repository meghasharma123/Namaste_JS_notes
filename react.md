# VDOM

- It's a lightweight JavaScript representation of the UI.
- `Reconciliation` is the process React uses to determine what needs to change in the UI. It uses heuristics, including element types and keys, to match elements between renders.

- `useEffect` is used for side effects and synchronization with external systems. It generally executes after the browser paints.
- `useLayoutEffect` executes synchronously after DOM mutations but before browser painting. I use useLayoutEffect when I need to measure DOM dimensions or make layout adjustments without visual flickering.

- `React.memo`: It memoizes a component and can skip re-rendering when its props are unchanged.
- `useMemo`: It remembers the result of a calculation, not the function itself.
- `useCallback`: It remembers the function reference
  
```
import React, { useState, useMemo, useCallback } from "react";

const Child = React.memo(({ value, onClick }) => {
  console.log("Child rendered");

  return <button onClick={onClick}>{value}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);

  const value = useMemo(() => count * 2, [count]);

  const handleClick = useCallback(() => {
    console.log("Clicked");
  }, []);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>

      <Child value={value} onClick={handleClick} />
    </>
  );
}
```

- `useState`: It stores component state. When you update state using its setter, React schedules a re-render.
- `useRef`: It stores a mutable value that persists across renders without triggering a re-render when .current changes. It is also used to access DOM elements.
- I use useState for UI state, useRef for timer IDs and DOM references such as focusing an input.

- `Context API` helps avoid prop drilling by sharing data across components without passing props through every level. It is suitable for relatively simple shared state, such as themes and authentication.
- `Redux Toolkit` provides a structured and predictable approach to managing complex global state. It supports slices, reducers, middleware and efficient subscriptions through selectors.

## How Redux works

There are four important concepts:
1. `Store`: Central place where application state is stored.
2. `Action`: Describes what happened.
3. `Reducer`: Determines how the state should change.
4. `Dispatch`: Sends an action to the store.

### Example

1. create a slice

```
// cartSlice.js

import { createSlice } from "@reduxjs/toolkit";

const cartSlice = createSlice({
  name: "cart",

  initialState: {
    items: [],
  },

  reducers: {
    addToCart: (state, action) => {
      state.items.push(action.payload);
    },

    removeFromCart: (state, action) => {
      state.items = state.items.filter(
        item => item.id !== action.payload
      );
    },
  },
});

export const { addToCart, removeFromCart } =
  cartSlice.actions;

export default cartSlice.reducer;
```

2. Configure the store

```
// store.js

import { configureStore } from "@reduxjs/toolkit";
import cartReducer from "./cartSlice";

export const store = configureStore({
  reducer: {
    cart: cartReducer,
  },
});
```

3. Provide the store to React

```
// main.jsx

import { Provider } from "react-redux";
import { store } from "./store";

root.render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

4. use redux inside a component
```
import { useDispatch, useSelector } from "react-redux";
import { addToCart } from "./cartSlice";

function Product() {
  const dispatch = useDispatch();

  const cartItems = useSelector(
    state => state.cart.items
  );

  return (
    <>
      <button
        onClick={() => dispatch(addToCart({
          id: 1,
          name: "Laptop"
        }))}
      >
        Add to Cart
      </button>

      <p>Cart items: {cartItems.length}</p>
    </>
  );
}
```

## React Query

- React Query is a library for fetching, caching, synchronizing and updating server data in React applications.

- Redux can manage either type of state, but React Query specializes in server state.

1. Configure query client
```
import {
  QueryClient,
  QueryClientProvider
} from "@tanstack/react-query";

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Products />
    </QueryClientProvider>
  );
}
```

2. fetch product using fetch query
```
import { useQuery } from "@tanstack/react-query";

function Products() {
  const {
    data,
    isLoading,
    isError,
    error
  } = useQuery({
    queryKey: ["products"],

    queryFn: async () => {
      const response = await fetch("/api/products");

      if (!response.ok) {
        throw new Error("Failed to fetch products");
      }

      return response.json();
    }
  });

  if (isLoading) return <p>Loading...</p>;

  if (isError) return <p>{error.message}</p>;

  return (
    <div>
      {data.map(product => (
        <p key={product.id}>{product.name}</p>
      ))}
    </div>
  );
}
```

You don't need to manually manage loading, error and product state. React Query manages those states for the query.

What is caching?

Suppose:
1. User opens the Products page.
2. React Query calls the products API.
3. It stores the response in its cache.
4. User navigates to another page.
5. User returns to Products.

React Query can use the cached response instead of immediately making another network request.

queryClient.invalidateQueries({
  queryKey: ["products"]
});

- This marks the matching cached query as stale. If it is active, React Query normally refetches it in the background.

- `useQuery` is generally used to fetch data. `useMutation` is generally used to create, update or delete data.

```
import {
  useMutation,
  useQueryClient
} from "@tanstack/react-query";

function AddProduct() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: async (newProduct) => {
      const response = await fetch("/api/products", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify(newProduct)
      });

      if (!response.ok) {
        throw new Error("Failed to add product");
      }

      return response.json();
    },

    onSuccess: () => {
      queryClient.invalidateQueries({
        queryKey: ["products"]
      });
    }
  });

  return (
    <button
      onClick={() => mutation.mutate({
        name: "Laptop"
      })}
    >
      Add Product
    </button>
  );
}
```

- After a successful product creation, the products query is invalidated so the updated list can be fetched.

`Server state`: Product list, User profile from API, Order history, Inventory information
`Client state`: Modal visibility, Selected tab, Counter, Theme preference


### Optimization

1. React Developer Tools Profiler can use it to:

- Identify frequently rendering components.
- Check how long rendering takes.
- Identify which components are affected by an update.
- Compare commits to find performance bottlenecks.

2. How would you reduce unnecessary re-renders?

- React.memo: Prevents unnecessary child renders when props are unchanged.
- useMemo: Caches expensive calculations.
- useCallback: Preserves function references.
- Split large components into smaller components.
- Avoid unnecessary state updates.
- Keep state close to the components that need it.

3. How would you optimize repeated API calls?

- Use React Query (TanStack Query) for caching and request deduplication.
- Use appropriate staleTime to control when cached data becomes stale.
- Use pagination or infinite scrolling for large datasets.
- Debounce search requests.
- Cancel outdated requests when appropriate.
- Avoid unnecessary API calls inside effects.

4. How would you identify and fix UI performance issues?

- React DevTools Profiler: Component rendering
- Chrome Performance: Main-thread blocking, long tasks
- Chrome Network: Repeated API calls, slow requests
- Lighthouse: Overall page performance
- Memory panel: Memory leaks

