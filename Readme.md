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

```javascript
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
```

###  useCallback

```javascript
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
```

### useContext


1. Create Context
We need a context object to hold the global state. This can be an object with a Provider function and a Consumer function.

```javascript
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
```

2. Implement useContext
The useContext hook will access the current context value.

```javascript
function useContext(context) {
    return context;  // Return the context value (this simulates React's useContext)
}
```

3. Example Usage

```javascript
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
```


### useReducer

1. Define the useReducer Hook
The useReducer hook will:

Accept a reducer function and an initial state.
Return the current state and a dispatch function to send actions to the reducer.

```javascript
function useReducer(reducer, initialState) {
    let state = initialState;
    const dispatch = (action) => {
        state = reducer(state, action);  // Call the reducer function to get the next state
    };
    return [state, dispatch];  // Return the current state and the dispatch function
}
```
2. Example Reducer Function
The reducer function defines how the state changes in response to actions. It takes the current state and an action and returns the new state.

```javascript
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
```
3. Using useReducer in a Component
Now, we can use useReducer in a component to manage state based on the actions.

```javascript
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
```


## Custom Hooks

### useThrottle
using useCallback

```javascript
import { useRef, useCallback } from 'react';

function useThrottle(callback, delay) {
    const lastCallTime = useRef(0); // Track the last time the function was called

    const throttledFn = useCallback((...args) => {
        const now = Date.now();
        if (now - lastCallTime.current >= delay) {
            callback(...args); // Call the function if enough time has passed
            lastCallTime.current = now; // Update last call time
        }
    }, [callback, delay]);

    return throttledFn;
}
```

without useCallback
```javascript
import { useRef } from 'react';

function useThrottle(callback, delay) {
    const lastCallTime = useRef(0); // Track the last time the function was called
    const lastCallback = useRef(callback); // Store the previous callback for comparison
    const lastDelay = useRef(delay); // Store the previous delay for comparison

    // Manually track dependency changes and update refs
    if (lastCallback.current !== callback || lastDelay.current !== delay) {
        lastCallback.current = callback;
        lastDelay.current = delay;
    }

    const throttledFn = (...args) => {
        const now = Date.now();
        if (now - lastCallTime.current >= lastDelay.current) {
            lastCallback.current(...args); 
            lastCallTime.current = now; 
        }
    };

    return throttledFn;
}
```

### useDebounce

```javascript
import { useRef } from 'react';

function useDebounce(callback, delay) {
    const timer = useRef(null); // Store the timer ID
    const lastCallback = useRef(callback); // Store the previous callback
    const lastDelay = useRef(delay); // Store the previous delay

    // Update refs if callback or delay changes
    if (lastCallback.current !== callback || lastDelay.current !== delay) {
        lastCallback.current = callback;
        lastDelay.current = delay;
    }

    const debouncedFn = (...args) => {
        if (timer.current) clearTimeout(timer.current); // Clear any existing timer
        timer.current = setTimeout(() => {
            lastCallback.current(...args); // Call the latest callback after the delay
        }, lastDelay.current);
    };

    return debouncedFn;
}


//usage

const fetchResults = useDebounce((searchQuery) => {
        console.log('Fetching results for:', searchQuery);
        // Simulate an API call
        setResults([`Result for ${searchQuery}`]);
    }, 500);

    const handleChange = (e) => {
        const newQuery = e.target.value;
        setQuery(newQuery);
        fetchResults(newQuery); // Call the debounced function
    };
```
### useInfiniteScroll

```javascript
import { useState, useEffect } from 'react';

function useInfiniteScroll(fetchData, threshold = 100) {
    const [isLoading, setIsLoading] = useState(false);
    
    const handleScroll = () => {
        if (isLoading) return; // Prevent multiple requests while data is loading

        const bottom = window.innerHeight + window.scrollY >= document.documentElement.scrollHeight - threshold;
        if (bottom) {
            setIsLoading(true);
            fetchData(); // Call the fetchData function to load more content
            setIsLoading(false)
        }
    };

    useEffect(() => {
        window.addEventListener('scroll', handleScroll);
        return () => {
            window.removeEventListener('scroll', handleScroll);
        };
    }, [isLoading]);

    return isLoading;
}


//usage

 const [items, setItems] = useState([]);
    const [page, setPage] = useState(1);

    // Simulate fetching more data (replace with real API)
    const fetchItems = async () => {
        console.log('Fetching more items...');
        const newItems = await new Promise(resolve => {
            setTimeout(() => {
                resolve(Array.from({ length: 10 }, (_, index) => `Item ${index + (page - 1) * 10}`));
            }, 1000);
        });

        setItems(prevItems => [...prevItems, ...newItems]);
        setPage(prevPage => prevPage + 1); // Update page for next request
    };

    const isLoading = useInfiniteScroll(fetchItems, 200); // 200px threshold before the bottom

```

### useLocalStorage




<details>
<summary><h3>useLocalStorage</h3></summary>

```javascript
import { useCallback } from 'react';

function useLocalStorage() {
    const getValue = useCallback((keys) => {
        try {
            if (Array.isArray(keys)) {
                return keys.reduce((result, key) => {
                    const item = localStorage.getItem(key);
                    result[key] = item ? JSON.parse(item) : null;
                    return result;
                }, {});
            } else {
                const item = localStorage.getItem(keys);
                return item ? JSON.parse(item) : null;
            }
        } catch (error) {
            console.error(`Error getting value(s) from localStorage key(s):`, error);
            return null;
        }
    }, []);

    const setValue = useCallback((keyValues) => {
        try {
            if (Array.isArray(keyValues)) {
                keyValues.forEach(({ key, value }) => {
                    const valueToStore =
                        value instanceof Function ? value(getValue(key)) : value;
                    localStorage.setItem(key, JSON.stringify(valueToStore));
                });
            } else {
                const { key, value } = keyValues;
                const valueToStore =
                    value instanceof Function ? value(getValue(key)) : value;
                localStorage.setItem(key, JSON.stringify(valueToStore));
            }
        } catch (error) {
            console.error(`Error setting value(s) in localStorage:`, error);
        }
    }, [getValue]);

    const deleteValue = useCallback((keys) => {
        try {
            if (Array.isArray(keys)) {
                keys.forEach((key) => localStorage.removeItem(key));
            } else {
                localStorage.removeItem(keys);
            }
        } catch (error) {
            console.error(`Error deleting value(s) from localStorage:`, error);
        }
    }, []);

    const modifyValue = useCallback((keyModifications) => {
        try {
            if (Array.isArray(keyModifications)) {
                keyModifications.forEach(({ key, modifyFn }) => {
                    const currentValue = getValue(key);
                    const modifiedValue = modifyFn(currentValue);
                    setValue({ key, value: modifiedValue });
                });
            } else {
                const { key, modifyFn } = keyModifications;
                const currentValue = getValue(key);
                const modifiedValue = modifyFn(currentValue);
                setValue({ key, value: modifiedValue });
            }
        } catch (error) {
            console.error(`Error modifying value(s) in localStorage:`, error);
        }
    }, [getValue, setValue]);

    return {
        getValue,
        setValue,
        deleteValue,
        modifyValue,
    };
}

export default useLocalStorage;

```
</details> 