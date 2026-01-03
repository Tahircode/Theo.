#### React Hooks
- Definition: Hooks are special functions that allow you to "hook into" React features (like State and Lifecycle) inside Functional Components.

 - Before Hooks (Class Components), you had to use classes to use state. Hooks allow you to use state and other React features without writing a class.

 **The Rules of Hooks** 
Before learning the types, you must follow two strict rules, or your app will crash:

- Only Call Hooks at the Top Level: Don't call them inside loops, conditions, or nested functions. They must run in the same order every time.
- Only Call Hooks from React Functions: Call them inside React functional components or custom hooks, not regular JavaScript functions.

**Types of Hooks**
1. State Hooks
2. Context Hooks
3. Effect Hooks
4. Performance Hooks
5. Custom Hooks

1. **State Hooks**
  *a. useState (The Memory):*

  - Definition: Adds a state variable to your component. It returns a pair: the current state value and a function to update it.
  - Use: When you need data to change over time (counters, form inputs, toggles).
  - Syntax: const [state, setState] = useState(initialValue);
   example:
   import { useState } from 'react';

function Counter() {
  // 1. Declare state variable "count" (initial 0)
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      {/* 2. Update state using setCount */}
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}

*b. useReducer (The Complex State Manager):*

- Definition: An alternative to useState. It accepts a reducer of type (state, action) => newState, and returns the current state paired with a dispatch method.
- Use: When you have complex state logic where the next state depends on the previous one (like a todo list with add/delete/toggle actions).
- Syntax:
  const [state, dispatch] = useReducer(reducer, initialState);

- state: The current state value.
- dispatch: A function used to dispatch actions that will update the state.
- reducer: A function that defines how the state should change based on the dispatched action.
- initialState: The initial state value.

 
import { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    default: throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  );
}

**2. Context Hooks**

  *a. useContext(The Tunnel)*
   - Definition: Accepts a context object (the value returned from createContext) and returns the current context value for that context.
  - The useContext hook in React is a powerful and convenient way to consume   values from the React *Context API* in functional components. It allows functional components to access context values directly, without the need to manually pass props down through the component tree

 - Syntax:  const contextValue = useContext(MyContext);

 - Use: To consume global data (User, Theme) without prop drilling.
 import React, { createContext, useContext, useState } from "react";

const ThemeContext = createContext();

function App() {
    const [theme, setTheme] = useState("light");

    const toggleTheme = () => {
        setTheme((prevTheme) => (prevTheme === "light" ? "dark" : "light"));
    };

    return (
        <ThemeContext.Provider value={theme}>
            <div>
                <h1>Current Theme: {theme}</h1>
                <button onClick={toggleTheme}>Toggle Theme</button>
                <ThemeDisplay />
            </div>
        </ThemeContext.Provider>
    );
}

function ThemeDisplay() {
    const theme = useContext(ThemeContext);

    return <h2>Theme from Context: {theme}</h2>;
}

export default App;

**3. Effect Hooks**
- Effect hooks, specifically useEffect, useLayoutEffect, and useInsertionEffect, enable functional components to handle side effects in a more efficient and modular way.

  a. useEffect:
  - The useEffect hook in React is used to handle side effects in functional components. It allows you to perform actions such as data fetching, DOM manipulation, and setting up subscriptions, which are typically handled in lifecycle methods like componentDidMount or componentDidUpdate in class components.
 - Syntax: useEffect(() => {
    // Side effect logic here
}, [dependencies]);

 b. useLayoutEffect:
 - useLayoutEffect is a very specific Hook that looks identical to useEffect, but with one critical difference in timing.
It is the advanced version of useEffect, specifically designed for when you need to touch the DOM before the user sees it.

working:
. React updates the DOM (HTML is changed).
. useLayoutEffect runs: Your code executes (e.g., measures the DOM).
. React updates the DOM again (if needed).
. Browser Paints: The user sees the final result.

 import { useState, useLayoutEffect, useRef } from 'react';

function Box() {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const boxRef = useRef(null);

  // We use useLayoutEffect to measure BEFORE painting
  useLayoutEffect(() => {
    // 1. Measure the element
    const { offsetHeight } = boxRef.current;
    
    // 2. Calculate new position based on height
    // (For example, let's shift it down by its own height)
    setPosition({ top: offsetHeight, left: 0 });
  }, []); // Empty array = run once on mount

  return (
    <div
      ref={boxRef}
      style={{
        position: 'absolute',
        top: position.top,
        left: position.left,
        width: '100px',
        height: '100px',
        background: 'blue',
        color: 'white',
      }}
    >
      I don't flicker!
    </div>
  );
}

 c.  useInsertionEffect:

 -  The useInsertionEffect is designed for injecting styles early, especially useful for server-side rendering (SSR) or styling libraries, ensuring styles are in place before the component is rendered visually.

- useInsertionEffect is designed specifically for authors of CSS-in-JS libraries (like styled-components or Emotion), not for application developers.
- useInsertionEffect is a hook that runs synchronously before any DOM mutations (layout changes) are painted by the browser, but after React has determined what changes to make.

- Syntax:
 useInsertionEffect(() => {
    // Logic to inject styles or manipulate stylesheets
}, [dependencies]);

import { useInsertionEffect } from 'react';

// Note: In a real app, you would use a CSS file. 
// This is only useful for dynamic libraries.
function MyStyledBox() {
  
  useInsertionEffect(() => {
    // 1. Create a style tag
    const style = document.createElement('style');
    style.innerHTML = `
      .dynamic-box {
        background: linear-gradient(to right, red, orange);
        padding: 20px;
        color: white;
      }
    `;

    // 2. Inject it into the head immediately
    document.head.appendChild(style);

    // 3. Cleanup: Remove the style tag when component unmounts
    return () => {
      document.head.removeChild(style);
    };
  }, []);

  // 4. Use the class
  return <div className="dynamic-box">I have injected styles!</div>;
}


**4. Performenace Hook**

- Performance Hooks in React, like useMemo and useCallback, are used to optimize performance by avoiding unnecessary re-renders or recalculations.

 a. useMemo:
- It is a React hook that memoizes the result of an expensive calculation, preventing it from being recalculated on every render unless its dependencies change. This is particularly useful when we have a computation that is expensive in terms of performance, and we want to avoid recalculating it on every render cycle.

  // This sorting only runs if 'numbers' changes, NOT if 'count' changes
  const sortedList = useMemo(() => {
    console.log("Calculating...");
    return numbers.sort((a, b) => a - b);
  }, [numbers]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Re-render: {count}</button>
      <div>{sortedList.join(', ')}</div>
    </div>
  );
}

b.useCallback:
- It is a React hook that helps to memoize functions, ensuring that a function is not redefined on every render unless its dependencies change. This is particularly useful when passing functions as props to child components, as it prevents unnecessary re-renders of those child components.
- Syntax:
const memoizedCallback = useCallback(() => 
{ doSomething(a, b); }, [a, b]);


**5. Resource Hooks**

=> useFetch:
- The useFetch is typically a custom hook used for fetching data from an API. It is implemented with useEffect to fetch data when the component mounts or when dependencies change.
Syntax:
- const { data, loading, error } = useFetch(url);


**6. Custom Hooks**

- Custom Hooks are user-defined functions that encapsulate reusable logic. They enhance code reusability and readability by sharing behavior between components.
- //useWidth.js

import { useState, useEffect } from "react";

function useWidth() {
    const [width, setWidth] = useState(window.innerWidth);

    useEffect(() => {
        const handleResize = () => setWidth(window.innerWidth);
        window.addEventListener("resize", handleResize);
        return () => window.removeEventListener("resize", handleResize);
    }, []);

    return width;
}

export default useWidth;

function App() {
    const width = useWidth();
    return <h1>Window Width: {width}px</h1>;
}

export default App;






 

