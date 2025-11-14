# lesson - 1

- create your onw render

```jsx
function render(reactElement, containerDOMElement) {
  // 1. create a DOM element
  const domElement = document.createElement(reactElement.type);

  // 2. update properties
  domElement.innerText = reactElement.children;
  for (const key in reactElement.props) {
    const value = reactElement.props[key];
    domElement.setAttribute(key, value);
  }

  // 3. put it in the container
  containerDOMElement.appendChild(domElement);
}
```

- understanding jsx
```jsx
<p id="hello">Hello</p>
↓ compiled as ↓
React.createElement('p', { id: 'hello' }, 'Hello')
/*
Key Understandings:

Syntactic Sugar Essence: JSX ultimately converts to React.createElement calls.

Tree Structure: JSX perfectly reflects component hierarchy.
Compilation Optimization: Modern toolchains automatically handle JSX conversion details.

Note: Although modern React does not enforce importing React, retaining import React from 'react' is still common practice, especially when using features like Hooks.
*/ 
```

- Type coercion

```jsx
// At run-time, React will automatically convert types as needed when supplying attributes in expression slots.
// This works:
<input required="true" />

// And so does this!
<input required={true} />

// In HTML, it's possible to set attributes to true by specifying only the key, like this:
<input required />
<input required={true} />

// Note: The JSX compilation process does not validate the validity of expressions; any syntax errors will be thrown as JavaScript errors at runtime.
```

- Differences from HTML
```jsx
// Reserved words
// Self-closing tags
// Case Convention
  // Exceptions: Data attributes: Keep the hyphenated writing (e.g., data-test-id)
  // ARIA attributes: Keep the hyphenated writing (e.g., aria-label)

// Differences in Inline Styles
// HTML
<h1 style="font-size: 2rem; color: red;">标题</h1>

// JSX
<h1 style={{ fontSize: '2rem', color: 'red' }}>标题</h1>

// Style value handling
  //Numbers will automatically add the px unit (width: 200 → 200px)
  //Unitless properties will remain unchanged (flex: 1)
// Support for CSS Variables
<div style={{ '--primary-color': 'blue' }}></div>

```

- Components
```jsx
// Custom Hook Example
function useCounter() {
  const [count, setCount] = useState(0);
  const increment = () => setCount(c => c + 1);
  return { count, increment };
}

// Multiple components share the same logic.
function CounterA() {
  const { count, increment } = useCounter();
  return <button onClick={increment}>A: {count}</button>;
}
```

- Props
```jsx
// semantic state attributes

function Button({ status, children }) {
  const themeColor = {
    cancel: 'red',
    confirm: 'black'
  }[status];

  /*
  as
  const themeColor = {
    cancel: 'red',
    confirm: 'black'
  }['cancel'];

  */

  return (
    <button style={{
      border: '2px solid',
      color: themeColor,
      borderColor: themeColor,
    }}>
      {children}
    </button>
  );
}

// 使用示例
<Button status="cancel">取消</Button>
<Button status="confirm">确认</Button>
```

- Not global
  Keys only have to be unique within their array

```jsx
// Each .map() call produces a separate array, and so there's no problem. 👍
function App() {
  return (
    <ul>
      {data.map((contact) => (
        <ContactCard
          key={contact.id}
          name={contact.name}
          job={contact.job}
          email={contact.email}
        />
      ))}
      {data.map((contact) => (
        <ContactCard
          key={contact.id}
          name={contact.name}
          job={contact.job}
          email={contact.email}
        />
      ))}
    </ul>
  );
}
```

- Undefined attributes
  When react sees that an attribute is being set to `undefined`, it omits that attribute entirely from the DOM node. Rather than give it an empty value like `className=""`, it acts as though we haven't even tried to set a value.
  This is true for some other falsy values as well, like null and false.

- Always use boolean values with &&
```js
// 1.
{list.length > 0 && <Component />}
// 2.
{!!list.length && <Component />}

function App() {
  const shoppingList = ["avocado", "banana", "cinnamon"];
  const numOfItems = shoppingList.length;

  return <div>{numOfItems > 0 && <ShoppingList items={shoppingList} />}</div>;
}
```

- clsx package `npm install --save clsx`

```js
import clsx from 'clsx';

<button
  className={clsx(
    styles.btn,
    type === 'primary' && styles.primary,
    user ? styles.loggedIn : styles.notLoggedIn
  )}
>
```

# lesson - 2

- Passing a function reference

```js
// ✅ We want to do this:
<button onClick={doSomething} />

// 🚫 Not this:
<button onClick={doSomething()} />

// in compiled JavaScript
// ✅ Correct.
// Will be called when the user clicks the button.
React.createElement(
  'button',
  {
    onClick: doSomething,
  }
);

// 🚫 Incorrect
// Will be called immediately, when the element is created.
React.createElement(
  'button',
  {
    onClick: doSomething(),
  }
);
```

- useState
```jsx


// Normal initialization (executes on every render)
const [data] = useState(expensiveCalculation());

// Functional initialization (executes only on first render)
const [data] = useState(() => expensiveCalculation());

/*
Here, a function is passed to `useState`: `() => expensiveCalculation()`.  
React will only call this function **during the initial render of the component** and use its return value as the initial state.  
In subsequent renders, this function **will not be called again**.  

**Advantage**:  
The `expensiveCalculation()` function is executed **only once**, precisely when the state needs to be initialized. This avoids unnecessary performance overhead.
*/
```

- React Core Rendering Process Explained
1. Initial Render Process
Component Invocation:

React calls the Counter function and executes useState(0) to initialize state

The returned JSX is converted into a React element (JavaScript object):

```js
{
  type: 'button',
  props: {
    onClick: () => setCount(count + 1),
    children: 'Current Value: 0'
  }
}
```

2. Mounting Phase:

React creates actual DOM nodes based on this object

Inserts <button>Current Value: 0</button> into the page

Binds click event handlers

3. State Update Flow
When the button is clicked:

4. Trigger Update:

Calls setCount(count + 1) to update state from 0 to 1

Triggers a re-render

5. Re-rendering:

React calls the Counter function again

Generates new React element description:

```js
{
  type: 'button',
  props: {
    onClick: () => setCount(count + 1),
    children: 'Current Value: 1'
  }
}

```
6. Reconciliation Process:

React compares the old and new "snapshots" (like a spot-the-difference game)

Detects text content changed from "Current Value: 0" to "Current Value: 1"

7. Commit Update:
React precisely updates the DOM, modifying only what changed:
```js
button.textContent = "Current Value: 1";
```

8. Core Loop Diagram:
Trigger → Render → Reconcile → Commit

Key Insight: Every state update triggers a complete rendering cycle, but React optimizes DOM operations through intelligent diffing algorithms, only updating necessary parts.

- Asynchronous Updates
Conclusion:
State updates are not immediate.

When we call setCount, we're merely requesting a change to the state variable. React doesn't process it immediately—instead, it waits until the current operation (click handling) completes before updating the value and triggering a re-render.

Why this design?

Unified re-rendering

Improved performance

Ensures UI consistency

The actual delay is nearly imperceptible (completed within milliseconds)

```
flowchart TD
    A[用户操作/事件触发] --> B[setState/setCount]
    B --> C[状态变更请求（异步批处理）]
    C --> D[组件函数重新执行，生成新的 React 元素]
    D --> E[协调（Diff 对比）]
    E --> F[找出变化的部分]
    F --> G[提交（Commit），精确更新 DOM]
    G --> H[UI 展示最新状态]
```

- wrapper function

```js
<button onClick={() => setTheme("dark")}>Toggle theme</button>
```

- Default form behavior
  If we omit the action attribute, the browser will use the current URL, effectively reloading the page.

```js
// That's why we need to include event.preventDefault(). It stops the browser from executing a full page reload.
<form
  className="search-form"
  onSubmit={event => {
    event.preventDefault();
    runSearch(searchTerm);
  }}
>
```

# lesson - 3

```js
// Pluck this instance's unique ID from React
const id = React.useId();

// Create element IDs using this unique ID
const usernameId = `${id}-username`;
const passwordId = `${id}-password`;
```

- notice
  Items saved to localStorage are always saved as a string. You'll need to convert the stored value back to boolean. You can do this with JSON.parse().

- Cleanup

```js
React.useEffect(() => {
  function handleMouseMove(event) {
    setMousePosition({
      x: event.clientX,
      y: event.clientY,
    });
  }

  window.addEventListener("mousemove", handleMouseMove);

  return () => {
    window.removeEventListener("mousemove", handleMouseMove);
  };
}, []);
```

- Event bubbling

```js
<button
  onKeyDown={(event) => {
    if (event.code === 'Space') {
      event.stopPropagation();
    }
  }}
  onClick={() => {
    setIsPlaying(!isPlaying);
  }}
>
```

- Strict Mode - Strict Mode only affects the development environment.
  In "Normal" mode, here's the sequence of operations:
  Mount
  Run effect

In Strict Mode, the sequence is:
Mount
Run effect
Run cleanup
Re-run effect

`Strict Mode doesn't actually unmount/remount the component. This means that there's a single component instance, and we're calling the effect function twice.

When we unmount and remount the component, we create a brand-new component instance, with its own isPlaying internal state.`

- Why async effect callbacks aren't allowed

```js
React.useEffect(async () => {
  const intervalId = window.setInterval(() => {}, 1000);

  await someLongRunningProcess();

  // Even though we're returning a function, it'll actually
  // return a *promise* that resolves to a function:
  return () => {
    window.clearInterval(intervalId);
  };
});

// we can wrap our async logic up in an async function.
React.useEffect(() => {
  async function runEffect() {
    /* Async stuff */
  }
  runEffect();

  // Cleanup function is returned right away:
  return () => {
    /* Cleanup stuff */
  };
});
```

- React.memo helper - lets us memoize a component, so that it only re-renders when its props/state changes.

- React.useMemo - is that it allows us to `remember` a computed value between renders.

1. We can reduce the amount of work that needs to be done in a given render.

2. We can reduce the number of times that a component is re-rendered.

```js
// useMemo takes two arguments
// A chunk of work to be performed, wrapped up in a callback function
// A list of dependencies
const allPrimes = React.useMemo(() => {
  const result = [];

  for (let counter = 2; counter < selectedNum; counter++) {
    if (isPrime(counter)) {
      result.push(counter);
    }
  }

  return result;
}, [selectedNum]);
```

- Similarities to other stuff

1. React.memo memoizes the result of rendering a component, only re-running when the props change.

2. React.useMemo memoizes the result of a computation, only re-running when the dependencies change.

- useCallback

`useCallback` is a React Hook used to **cache functions**, preventing the creation of new function references on every component render.  

1. Why Use It?
In React, if you pass a function as props to a child component or use it in dependency-sensitive contexts (e.g., `useEffect`), a new function reference is generated every time the parent component re-renders. This can lead to **unnecessary re-renders of the child component**.  
Using `useCallback` ensures the function maintains the same reference when its dependencies remain unchanged, improving performance.  

2. Example:
```javascript  
import React, { useState, useCallback } from "react";  

function MyComponent() {  
  const [count, setCount] = useState(0);  

  // Normal approach: Generates a new function on every render  
  // const handleClick = () => setCount(count + 1);  

  // useCallback approach: Generates a new function only when count changes  
  const handleClick = useCallback(() => {  
    setCount(count + 1);  
  }, [count]);  

  return <button onClick={handleClick}>Click: {count}</button>;  
}  
```  

# lesson - 4

- Prop Delegation

```js
function LoggedInBanner({
  user,
  // Collect all unspecified props:
  ...delegated
}) {
  if (
    !user ||
    user.registrationStatus === 'unverified'
  ) {
    return null;
  }

  // And pass them onto Banner:
  return <Banner {...delegated} />
}

console.log(delegated);
/*
  {
    type: 'success',
    children: 'Account registered!',
  }
*/

// This code:
<Banner {...delegated} />

// ...is equivalent to this code:
<Banner
  type={delegated.type}
  children={delegated.children}
/>

// ...which is the same thing as this code:
<Banner type={delegated.type}>
  {delegated.children}
</Banner>

// example
function TextInput({ id, label, ...delegated }) {
  const generatedId = React.useId();
  const appliedId = id || generatedId;

  return (
    <div className="text-input">
      <label htmlFor={appliedId}>{label}</label>
      <input id={appliedId} {...delegated} /> {/* 转发所有属性到 input */}
    </div>
  );
}
<TextInput
  label="Email"
  type="email"
  required={true}
  data-test-id="login-email"
  onChange={(e) => setEmail(e.target.value)}
/>
```

- Forwarding Refs

```js
/*
Key Implementation Points
React.useId()

Generates a unique ID to ensure proper association between label and input via htmlFor and id, improving accessibility.

React.forwardRef

Allows parent components to directly access the input element's reference via ref (e.g., for focusing, reading values, etc.).

React.memo

Prevents unnecessary re-renders when props remain unchanged, optimizing performance.

Combining React.memo and React.forwardRef

This approach balances performance optimization (via memoization) and flexibility (via direct ref access), making the component both efficient and controllable.
*/ 
import React from 'react';

import styles from './Slider.module.css';

function Slider(
  { label, ...delegated },
  ref
) {
  const id = React.useId();

  return (
    <div className={styles.wrapper}>
      <label htmlFor={id} className={styles.label}>
        {label}
      </label>
      <input
        ref={ref}
        {...delegated}
        type="range"
        id={id}
        className={styles.slider}
      />
    </div>
  );
}

export default React.forwardRef(Slider);

// What if we want both React.memo and React.forwardRef on the same component?
export default React.memo(React.forwardRef(Slider));


```

- slot -  This pattern provides maximum control and power to the consumer:

```js
import React from "react";

function App() {
  return (
    <>
      <CaptionedImage
        image={
          <picture>
            <source
              type="image/avif"
              srcSet={`
 https://sandpack-bundler.vercel.app/img/meerkat.avif 1x,
 https://sandpack-bundler.vercel.app/img/meerkat@2x.avif 2x
 `}
            />
            <source
              type="image/jpeg"
              srcSet={`
 https://sandpack-bundler.vercel.app/img/meerkat.jpg 1x,
 https://sandpack-bundler.vercel.app/img/meerkat@2x.jpg 2x
 `}
            />
            <img
              alt="A meerkat looking curiously at the camera"
              src="https://sandpack-bundler.vercel.app/img/meerkat.jpg"
            />
          </picture>
        }
        caption={
          <>
            Photo by <a href="">Manuel Capellari</a>, shot in August 2019 and
            published on <strong>Unsplash</strong>.
          </>
        }
      />
    </>
  );
}

function CaptionedImage({ image, caption }) {
  return (
    <figure>
      {image}
      <div className="divider" />
      <figcaption>{caption}</figcaption>
    </figure>
  );
}

export default App;
```

1. With delegated props, we're able to provide additional props to a particular element

2. With polymorphism, we're able to change a particular element's HTML tag

3. With slots, we're able to provide any markup we want, without restrictions.


- context
```jsx
// Context allows data to be provided at the top of the component tree via a Provider, enabling any deeply nested component to access it directly through useContext.

// Create and Provide Context

// App.js
import React from 'react';

// 1.Create Context (similar to a broadcast frequency)
export const FavouriteColorContext = React.createContext();

function App () {
  const [favouriteColor, setFavouriteColor] = React.useState("#EBDEFB");

  // 2.Wrap the component tree with Provider (enable data broadcasting)
  return (
    <FavouriteColorContext.Provider value={favouriteColor, setFavouriteColor}>
      <Home />
    </FavouriteColorContext.Provider>
  )
}
/*
`createContext()` creates a data channel
The `Provider` component acts as a data transmitter
The `value` property passes the shared state
*/


// Consume Context
import {FavouriteColorContext} from './App';

function Sidebar () {
  // 3.Use useContext to receive data (similar to tuning a radio frequency)
  const { favouriteColor, setFavouriteColor } = React.useContext(favouriteColor);
  return <div style={{backgroundColor: favouriteColor}}>...</div>
}

// Engineering Recommendations
  // Multiple Values: Always bundle multiple values into an object
  value={{ user, setUser, theme, toggleTheme }}
  // Performance Optimization: Use memoization for unchanging values
  const contextValue = React.useMemo(() => ({ user }), [user]);  
```
# lesson - 5

- Principle of Least Privilege
```jsx

const [items, setItems] = React.useState([]);
<AddNewItemForm
        setItems={setItems}
/>
//When we give it a state-setter function, we grant it so much more power than that. For example, it can erase all of the current items:

// Erases all previously-saved items:
setItems([]);

// Or, it can break the app altogether, by setting it to something other than an array:

setItems(5);
//By contrast, in my original solution, the AddNewItemForm component can't do any of that stuff. The only thing it can do is push a new item into the array:


const [items, setItems] = React.useState([]);
function handleAddItem(label) {
    const newItem = {
      label,
      id: Math.random(),
    };

    const nextItems = [...items, newItem];
    setItems(nextItems);
  }

  <AddNewItemForm
        handleAddItem={handleAddItem}
  />
```

- Fall-through
```js
// By default, once we've found a matching case, the code within that case will be run… along with every subsequent case!
// Here's what actually gets logged:
// 2, 3, 4
const n = 2;

switch (n) {
  case 1: {
    console.log(1);
  }
  case 2: {
    console.log(2);
  }
  case 3: {
    console.log(3);
  }
  case 4: {
    console.log(4);
  }
}

// In order to avoid this behaviors, we need to end each case with the special keyword break:
switch (n) {
  case 1: {
    console.log(1);
    break;
  }
  case 2: {
    console.log(2);
    break;
  }
  case 3: {
    console.log(3);
    break;
  }
  case 4: {
    console.log(4);
    break;
  }
}

// with useReducer, we have another option: we can bail out of the entire function, using the return keyword:
// Instead of halting execution of the switch/case, we halt execution of the entire function!
function reducer(todos, action) {
  switch (action.type) {
    case 'create-todo': {
      return /* ✂️ */;
    }

    case 'toggle-todo': {
      return /* ✂️ */;
    }

    case 'delete-todo': {
      return /* ✂️ */;
    }
  }
}
```

- Immer
```js
// Fortunately, Immer doesn't do anything as mundane as a deep copy.
const state = {
  customer: {
    name: 'Daria Hakimi',
  },
  toppings: {
    pepperoni: true,
    anchovies: true,
    kale: true,
  },
};

const nextState = produce(state, (draftState) => {
  draftState.toppings.pepperoni = false;
});

// When we try and change the value of draftState.toppings.pepperoni, the Proxy jumps in the path of the bullet, deflecting it, and replacing it with an immutable operation, something like:
const newState = {
  ...state,
  toppings: {
    ...state.toppings,
    pepperoni: false,
  },
};

```

- Immer - Drawbacks
```js
// The biggest issue is that proxies can't easily be logged. Trying to console.log produces some pretty inscrutable results:
import { produce } from 'immer';

const arr = [1, 2, 3];

produce(arr, (draftArr) => {
  draftArr.push(4);

  console.log(draftArr);
  // Proxy {
  //   [[Handler]]: null
  //   [[Target]]: null
  //   [[isRevoked]]: true
  // }
});

// Here's the good news: Immer ships with a tool, current, which can help "unpack" a proxy, for debugging purposes:

// To be clear, you shouldn't ever need to use current in your final production code. It's a tool that exists purely to help you debug, while you're solving the problem at hand.
import { produce, current } from 'immer';

const arr = [1, 2, 3];
produce(arr, (draftArr) => {
  draftArr.push(4);

  console.log(current(draftArr));
  // [1, 2, 3, 4]
});
```