# Interview Questions

## Hooks Examples

<details>
<summary><h3>useState</h3></summary>
</details>

<details>
<summary><h3>useEffect</h3></summary>
</details>

<details>
<summary><h3>usRef</h3></summary>
</details>

<details>
<summary><h3>useContext</h3></summary>
</details>

<details>
<summary><h3>useMemo</h3></summary>
</details>

<details>
<summary><h3>useCallback</h3></summary>
</details>

<details>
<summary><h3>useReducer</h3></summary>
</details>



## PolyFills

<details>
<summary><h3>useState</h3></summary>

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
</details>

<details>
<summary><h3>useEffect</h3></summary>

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
</details>

<details>
<summary><h3>useMemo</h3></summary>

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

</details>



<details>
<summary><h3>useCallback</h3></summary>

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
</details>

<details>
<summary><h3>useContext</h3></summary>


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

</details>


<details>
<summary><h3>UseReducer</h3></summary>

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
</details>


## Custom Hooks

<details>
<summary><h3>useThrottle</h3></summary>
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

</details>

<details>
<summary><h3>useDebounce</h3></summary>

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

</details>
<details>
<summary><h3>useInfiniteScroll</h3></summary>


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


</details>







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


<details>
<summary><h3>useSession</h3></summary>

```javascript
import { useCallback } from "react";

function useSession() {
  const getValue = useCallback((keys) => {
    try {
      if (Array.isArray(keys)) {
        return keys.reduce((result, key) => {
          const item = sessionStorage.getItem(key);
          result[key] = item ? JSON.parse(item) : null;
          return result;
        }, {});
      } else {
        const item = sessionStorage.getItem(keys);
        return item ? JSON.parse(item) : null;
      }
    } catch (error) {
      console.error(`Error getting sessionStorage value(s):`, error);
      return null;
    }
  }, []);

  const setValue = useCallback((keyValues) => {
    try {
      if (Array.isArray(keyValues)) {
        keyValues.forEach(({ key, value }) => {
          const valueToStore =
            value instanceof Function ? value(getValue(key)) : value;
          sessionStorage.setItem(key, JSON.stringify(valueToStore));
        });
      } else {
        const { key, value } = keyValues;
        const valueToStore =
          value instanceof Function ? value(getValue(key)) : value;
        sessionStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(`Error setting sessionStorage value(s):`, error);
    }
  }, [getValue]);

  const deleteValue = useCallback((keys) => {
    try {
      if (Array.isArray(keys)) {
        keys.forEach((key) => sessionStorage.removeItem(key));
      } else {
        sessionStorage.removeItem(keys);
      }
    } catch (error) {
      console.error(`Error deleting sessionStorage value(s):`, error);
    }
  }, []);

  const clearSession = useCallback(() => {
    try {
      sessionStorage.clear();
    } catch (error) {
      console.error("Error clearing sessionStorage:", error);
    }
  }, []);

  return {
    getValue,
    setValue,
    deleteValue,
    clearSession,
  };
}

export default useSession;

```
</details>


<details>
<summary><h3>useCookies</h3></summary>

```javascript
    import { useCallback } from "react";

function useCookies() {
  const setCookie = useCallback((key, value, options = {}) => {
    let cookieString = `${encodeURIComponent(key)}=${encodeURIComponent(value)}`;
    if (options.expires) {
      cookieString += `; expires=${options.expires.toUTCString()}`;
    }
    if (options.path) {
      cookieString += `; path=${options.path}`;
    }
    if (options.domain) {
      cookieString += `; domain=${options.domain}`;
    }
    if (options.secure) {
      cookieString += `; secure`;
    }
    if (options.sameSite) {
      cookieString += `; samesite=${options.sameSite}`;
    }
    document.cookie = cookieString;
  }, []);

  const getCookie = useCallback((key) => {
    const cookies = document.cookie.split("; ").reduce((acc, cookie) => {
      const [k, v] = cookie.split("=");
      acc[decodeURIComponent(k)] = decodeURIComponent(v);
      return acc;
    }, {});
    return cookies[key] || null;
  }, []);

  const deleteCookie = useCallback((key, options = {}) => {
    setCookie(key, "", { ...options, expires: new Date(0) });
  }, [setCookie]);

  return { setCookie, getCookie, deleteCookie };
}

export default useCookies;

```

</details>

<details>
<summary><h3>useToggle</h3></summary>

```javascript
import { useState, useCallback } from "react";

function useToggle(initialValue = false) {
  const [state, setState] = useState(initialValue);

  const toggle = useCallback((value) => {
    // If a specific value is provided, set it; otherwise, toggle the current state
    setState((prevState) => (typeof value === "boolean" ? value : !prevState));
  }, []);

  return [state, toggle];
}

export default useToggle;
```

</details>


<details>
<summary><h3>useDarkMode</h3></summary>

```javascript
import { useState, useEffect } from "react";

function useDarkMode() {
  const [isDarkMode, setIsDarkMode] = useState(() => {
    // Check localStorage or system preferences on initial load
    const storedMode = localStorage.getItem("darkMode");
    return storedMode ? JSON.parse(storedMode) : window.matchMedia("(prefers-color-scheme: dark)").matches;
  });

  useEffect(() => {
    // Apply the theme to the body element
    if (isDarkMode) {
      document.body.classList.add("dark");
    } else {
      document.body.classList.remove("dark");
    }

    // Save the user's preference in localStorage
    localStorage.setItem("darkMode", JSON.stringify(isDarkMode));
  }, [isDarkMode]);

  const toggleDarkMode = () => {
    setIsDarkMode((prev) => !prev);
  };

  return [isDarkMode, toggleDarkMode];
}

export default useDarkMode;

```

</details>


<details>
<summary><h3>useFetch</h3></summary>

```javascript
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    };
    fetchData();
  }, [url]);

  return { data, loading, error };
}
```

</details>


<details>
<summary><h3>Authentication Hooks</h3></summary>

1. useAuth: Manages authentication state
This hook handles the user's login, logout, and session persistence.

```javascript

import { useState, useEffect } from 'react';

function useAuth() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // Check if a user is logged in (using localStorage or cookies)
  useEffect(() => {
    const storedUser = localStorage.getItem('user');
    if (storedUser) {
      setUser(JSON.parse(storedUser));
    }
    setLoading(false);
  }, []);

  const login = (userData) => {
    setUser(userData);
    localStorage.setItem('user', JSON.stringify(userData));
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('user');
  };

  return { user, loading, login, logout };
}

export default useAuth;
```


2. useToken: Manages user authentication token
This hook handles the retrieval and setting of an authentication token (e.g., JWT).

```javascript
import { useState, useEffect } from 'react';

function useToken() {
  const [token, setToken] = useState(null);

  // Check if a token exists in localStorage or cookies
  useEffect(() => {
    const storedToken = localStorage.getItem('token');
    if (storedToken) {
      setToken(storedToken);
    }
  }, []);

  const saveToken = (newToken) => {
    setToken(newToken);
    localStorage.setItem('token', newToken);
  };

  const clearToken = () => {
    setToken(null);
    localStorage.removeItem('token');
  };

  return { token, saveToken, clearToken };
}

export default useToken;
```

3. useSession: Manages user session state
This hook can manage session-based data for a user, storing and fetching session data from sessionStorage.

```javascript
import { useState, useEffect } from 'react';

function useSession() {
  const [session, setSession] = useState(null);

  // Check if session data exists in sessionStorage
  useEffect(() => {
    const storedSession = sessionStorage.getItem('session');
    if (storedSession) {
      setSession(JSON.parse(storedSession));
    }
  }, []);

  const saveSession = (sessionData) => {
    setSession(sessionData);
    sessionStorage.setItem('session', JSON.stringify(sessionData));
  };

  const clearSession = () => {
    setSession(null);
    sessionStorage.removeItem('session');
  };

  return { session, saveSession, clearSession };
}

export default useSession;
```

4. useRequireAuth: Protects routes by redirecting unauthenticated users
This hook can be used to protect certain routes by redirecting the user if they are not logged in.

```javascript
import { useEffect } from 'react';
import { useHistory } from 'react-router-dom'; // or useNavigate for React Router v6

function useRequireAuth() {
  const history = useHistory();
  const { user } = useAuth(); // Assuming useAuth hook is implemented as shown above

  useEffect(() => {
    if (!user) {
      history.push('/login'); // Redirect to login page
    }
  }, [user, history]);
}

export default useRequireAuth;
```


5. useLogin: Handles login functionality
This hook handles the user login process, including making API requests and managing the login state.

```javascript
import { useState } from 'react';
import useToken from './useToken'; // Assuming useToken hook is implemented as shown above

function useLogin() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const { saveToken } = useToken();

  const login = async (username, password) => {
    setLoading(true);
    setError(null);
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ username, password }),
        headers: { 'Content-Type': 'application/json' },
      });
      const data = await response.json();

      if (response.ok) {
        saveToken(data.token); // Save the token
      } else {
        setError(data.message); // Handle login failure
      }
    } catch (err) {
      setError('Something went wrong');
    } finally {
      setLoading(false);
    }
  };

  return { login, loading, error };
}

export default useLogin;
```

6. useLogout: Handles logout functionality
This hook is used to handle the logout process and clear authentication data.

```javascript
import { useState } from 'react';
import useToken from './useToken'; // Assuming useToken hook is implemented as shown above

function useLogout() {
  const [loading, setLoading] = useState(false);
  const { clearToken } = useToken();

  const logout = async () => {
    setLoading(true);
    try {
      // Here you might want to call an API to invalidate the session
      await fetch('/api/logout', { method: 'POST' });
      clearToken(); // Clear the stored token
    } catch (err) {
      console.error('Error logging out', err);
    } finally {
      setLoading(false);
    }
  };

  return { logout, loading };
}

export default useLogout;
```


7. useAuthCheck: Checks for authentication on initial load
This hook can be used to check whether the user is authenticated on the initial load of your application.

```javascript
import { useEffect, useState } from 'react';
import useAuth from './useAuth'; // Assuming useAuth hook is implemented as shown above

function useAuthCheck() {
  const { user } = useAuth();
  const [isAuthChecked, setIsAuthChecked] = useState(false);

  useEffect(() => {


    if (user !== null) {
      setIsAuthChecked(true);
    }

    


  }, [user]);

  return isAuthChecked;
}

export default useAuthCheck;
```
</details>

# Machine Coding Round Problems

1. ### Multistep Tabular Form with Prev, Next Button
[Code](https://github.com/WebDevSimplified/react-multistep-form/)
[Video](https://www.youtube.com/watch?v=uDCBSnWkuH0&ab_channel=WebDevSimplified)


2. ### Pagination
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/pagination)
[Video](https://www.youtube.com/watch?v=cBsB7hhOzQI&ab_channel=RoadsideCoder)


3. ### File Explorer
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/file-explorer)
[Video](https://www.youtube.com/watch?v=20F_KzHPpvI)

4. ### Job Board
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/job-board)
[Video](https://www.youtube.com/watch?v=KJ-cf62ioQs)


5. ### Drag & Drop Notes
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/drag-and-drop-notes)
[Video](https://www.youtube.com/watch?v=3U3UiBfcNqQ)


6. ### Quiz App
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/quiz-app)
[Video](https://www.youtube.com/watch?v=TF1FKrzsRDM)


7. ### Multi Select Search
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/multi-select-input)
[Video](https://www.youtube.com/watch?v=AsvybgZTryo)

8. ### Grid Lights
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/grid-sequence)  
[Video](https://www.youtube.com/watch?v=sAF7Km_znjA)


9. ### Dark Mode Light Mode
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/dark-light-mode)
[Video](https://www.youtube.com/watch?v=5XtvVH-v6kM)

10. ### OTP component
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/otp-login)
[Video](https://www.youtube.com/watch?v=nQ-hy4PUta8)

11. ### Selectable Grids
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/selectable-grid)
[Video](https://www.youtube.com/watch?v=Vo3zBqXcDjQ)

12. ### Tic Tac Toe
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/tic-tac-toe)
[Video](https://www.youtube.com/watch?v=-uAhf92DVqA)

13. ### Memory Game
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/memory-game)
[Video](https://www.youtube.com/watch?v=84o1kuPuYdU)

14. ### Password Generator
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/password-generator)
[Video](https://www.youtube.com/watch?v=u9-x0sG-WQc)

15. ### EMI calculator
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/emi-calculator)
[Video](https://www.youtube.com/watch?v=YvgUeUlpwyA&list=PLKhlp2qtUcSZiWKJTi5-5r6IRdHhxP9ZU&index=4&pp=iAQB)

16. ### Progress Bar
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/reactjs-interview-questions/progress-bar)
[Video](https://www.youtube.com/watch?v=9Ez_gdsgGiM&ab_channel=RoadsideCoder)

17. ### Star Rating
[Code](https://github.com/shaikhsufyan/react-js-project/tree/main/start-rating/)
[Video](https://www.youtube.com/watch?v=yoNm6ha8ty4&ab_channel=CODEwithSufyanShaikh)

18. ### Accordion
[Code](https://github.com/rohitsingh-23/Intro-to-react/tree/accordion)
[Video](https://www.youtube.com/watch?v=X11wN7aPGeA&ab_channel=EngineeringMind)

19. ### Employee Management App
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/machine-coding-interview-questions/employee-database-management)
[Video](https://www.youtube.com/watch?v=ZrMO0bCGwFg&list=PLKhlp2qtUcSYQojD5G-ElgHezoCyq2Hgo&ab_channel=RoadsideCoder)


20. ### Modal Poup
[Code- codesandbox](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbGVraGdZRmFINFlIa0xZVjZ0T3BNNm50enJ1Z3xBQ3Jtc0tuMDM2Zm1aUU9rTzZKbWFidDBCYUkyMG1Xc0NhYzh6VEs2d1lLZVJoUFMtaUx0QUZWNDNEeUNFdGN1Q2dsWFY3Z0doeFpOdGY5N0c0Zk5JVTNtMjJJeVpyeWdDOGVzOVRYN0VPYlJzZ2pidXhuMjBIMA&q=https%3A%2F%2Fcodesandbox.io%2Fs%2Fmodal-reactjs-w4scxg&v=8kin62BmHHc)
[Video](youtube)

21. ### Search bar With Auto Suggestions
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/frontend-system-design-questions/autosuggestion-component)
[Video](https://www.youtube.com/watch?v=6YrkXWFgiV8&ab_channel=RoadsideCoder)


22. ### Nested Comments
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/frontend-system-design-questions/nested-comments)
[Video](https://www.youtube.com/watch?v=zeY1Vw1dWJk&ab_channel=RoadsideCoder)

23. ### Toast Component
[Code](https://github.com/piyush-eon/frontend-interview-questions/tree/master/frontend-system-design-questions/toast-component)
[Video](https://www.youtube.com/watch?v=v50uJDEFnqM&ab_channel=RoadsideCoder)

24. ### Country, State, City Selector
[Code](https://github.com/Sridhar-C-25/country_state_city_selector_react)
[Video](https://www.youtube.com/watch?v=c4yATJ8EKuw&ab_channel=CodeAProgram)

25. ### Image Corousel
[Code](https://github.com/JohanAlves/yt-react-carousel)
[Video](https://www.youtube.com/watch?v=SAWQ_LmyY2Q&ab_channel=JohanAlves)

26. ### Overlapping Circle Problem - Uber (Advanced)
[Code](https://devtools.tech/questions/s/how-to-detect-overlapping-circles-in-react-js-uber-frontend-interview-question---qid---ixSxPTedmz1CyVZSab1N)
[Video](https://www.youtube.com/watch?v=bgKMaqk2WKw&ab_channel=DevtoolsTech)

27. ### Shopping Cart Design
28. ### Product Page with Pagination, Filters, sorting


# Frontend System Design Questions

### Snake & Ladder
### Google Calender
### Whiteboard
### Google Sheet
### Poll Widget
### Asset Performace Of Web Apps

## Advanced 

### CodeSandBox - Chirag Goel Youtube
### Whatsapp - Chirag Goel Youtube
### BookMyShow - Chirag Goel Youtube
### Netflix - Chirag Goel Youtube
### Notion
### Pinterest
### Facebook News
### Chat App


