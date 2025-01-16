# Interview Questions

## PolyFills

### UseState

```javascript
function useState(initialValue) {
    let state = initialValue;

    const setState = (newValue) => {
        state = typeof newValue === "function" ? newValue(state) : newValue;
        console.log("State updated:", state);
    };

    return [() => state, setState];
}
const [getCount, setCount] = useState(0);
```

### UseEffect

```javascript
function useEffectCustom(effect, dependencies) {
    const hasRun = useRef(false); 
    const prevDependencies = useRef();

    if (
        !hasRun.current || 
        !prevDependencies.current || 
        dependencies.some((dep, i) => dep !== prevDependencies.current[i])
    ) {
        if (typeof effect === "function") {
            effect();
        }
        hasRun.current = true;
        prevDependencies.current = dependencies;
    }
}
```

### useMemo

import { useRef } from "react";

function useMemoCustom(cb, dependencies) {
    const cache = useRef({ value: null, dependencies: null });

    const hasChanged =
        !cache.current.dependencies ||
        dependencies.some((dep, i) => dep !== cache.current.dependencies[i]);

    if (hasChanged) {
        cache.current.value = cb();
        cache.current.dependencies = dependencies;
    }

    return cache.current.value;
}


###  useCallback
function useCallback(fn, dependencies) {
    let memoizedFn = fn;
    let prevDependencies = [];

    // Check if the dependencies have changed
    const hasChanged = !prevDependencies.length || dependencies.some((dep, i) => dep !== prevDependencies[i]);

    if (hasChanged) {
        memoizedFn = fn; // Reassign the memoized function if dependencies change
        prevDependencies = dependencies; // Update previous dependencies
    }

    return memoizedFn;
}


###useContext

1. Create Context
We need a context object to hold the global state. This can be an object with a Provider function and a Consumer function.

let currentContext = null;

function createContext(defaultValue) {
    return {
        Provider: ({ value, children }) => {
            currentContext = value;  // Store the context value globally
            return children; // Render children
        },
        Consumer: ({ children }) => {
            return children(currentContext);  // Access current context value
        },
    };
}


2. Implement useContext
The useContext hook will access the current context value.


function useContext(context) {
    return context;  // Return the context value (this simulates React's useContext)
}


3. Example Usage

// Create a new context
const MyContext = createContext("Initial Value");

function App() {
    return (
        <MyContext.Provider value="Hello, World!">
            <ChildComponent />
        </MyContext.Provider>
    );
}

function ChildComponent() {
    const value = useContext(MyContext);  // Access the context value
    return <div>{value}</div>;  // Will render "Hello, World!"
}


### useReducer

1. Define the useReducer Hook
The useReducer hook will:

Accept a reducer function and an initial state.
Return the current state and a dispatch function to send actions to the reducer.
javascript
Copy
Edit
function useReducer(reducer, initialState) {
    let state = initialState;
    const dispatch = (action) => {
        state = reducer(state, action);  // Call the reducer function to get the next state
    };
    return [state, dispatch];  // Return the current state and the dispatch function
}
2. Example Reducer Function
The reducer function defines how the state changes in response to actions. It takes the current state and an action and returns the new state.

javascript
Copy
Edit
function counterReducer(state, action) {
    switch (action.type) {
        case 'INCREMENT':
            return { count: state.count + 1 };
        case 'DECREMENT':
            return { count: state.count - 1 };
        default:
            return state;
    }
}
3. Using useReducer in a Component
Now, we can use useReducer in a component to manage state based on the actions.

javascript
Copy
Edit
function Counter() {
    const [state, dispatch] = useReducer(counterReducer, { count: 0 });

    return (
        <div>
            <p>Count: {state.count}</p>
            <button onClick={() => dispatch({ type: 'INCREMENT' })}>Increment</button>
            <button onClick={() => dispatch({ type: 'DECREMENT' })}>Decrement</button>
        </div>
    );
}

