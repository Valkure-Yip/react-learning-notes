# Hooks

> [Introducing Hooks – React](https://reactjs.org/docs/hooks-intro.html)

[TOC]



## Why use hooks

- allow you to reuse stateful logic without changing your component hierarchy (render props, HOC,  ...)
- split one component into smaller functions based on what pieces are related (such as setting up a subscription or fetching data) 可以将同一逻辑代码放在一起，而不是分割在不同生命周期方法中（类似vue composition api）
- Hooks let you use more of React’s features without classes. 在function components 中也能使用states



## State Hook

`useState()`: declares a “state variable”

**argument**: takes initial state the first time it renders

**return**: a pair of values: the current state and a function that updates it.



An example of function components with `useState`:

```jsx
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```



Compared with class component with `this.state`:

```jsx
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

### updating state based on previous state `setState(()=>{})`

https://react.dev/reference/react/useState#updating-state-based-on-the-previous-state

Suppose the `age` is `42`. This handler calls `setAge(age + 1)` three times:

```js
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

However, after one click, `age` will only be `43` rather than `45`! This is because calling the `set` function [does not update](https://react.dev/learn/state-as-a-snapshot) the `age` state variable in the already running code. So each `setAge(age + 1)` call becomes `setAge(43)`.
**pass an *updater function*** to `setAge` instead of the next state:

```js
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

### React useState 传入函数

1. 传入set函数中：解决异步更新问题 https://react.dev/reference/react/useState#updating-state-based-on-the-previous-state

```js
 setAge(a => a + 1); // setAge(42 => 43)
 setAge(a => a + 1); // setAge(43 => 44)
 setAge(a => a + 1); // setAge(44 => 45)
```

2. 传入useState**初始化函数** https://react.dev/reference/react/useState#avoiding-recreating-the-initial-state

```js
  const [todos, setTodos] = useState(createInitialTodos);
```

避免每次渲染都执行createInitialTodos

## Effect Hook

Effect hook: run some additional code (side effect) after React has updated the DOM (`componentDidMount`, `componentDidUpdate`). 



`useEffect()`: takes a side effect function (that returns an optional cleanup function)

e.g.

```jsx
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

Every time we re-render, we schedule a *different* effect, replacing the previo

注意每次渲染执行useEffect时，传入的函数是**不一样**的：`count`值会变，这样保证每次渲染后执行effect是响应式的



Compared with class & lifecycle methods:

the same piece of logic **repeated** in `componentDidMount` and `componentDidUpdate`

```jsx
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
	// the same logic repeated in two lifecycle methods:
  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```



### Effect with cleanup

In a React class, you would typically set up a **subscription** in `componentDidMount`, and clean it up in `componentWillUnmount`. 

Have to **split up** the logic in two different methods:

```js
componentDidMount() {
  ChatAPI.subscribeToFriendStatus(
    this.props.friend.id,
    this.handleStatusChange
  );
}
componentWillUnmount() {
  ChatAPI.unsubscribeFromFriendStatus(
    this.props.friend.id,
    this.handleStatusChange
  );
}
// subscription callback
handleStatusChange(status) {
  this.setState({
    isOnline: status.isOnline
  });
}
```

The same logic using effect hook, effect function returns a **cleanup function** 

```jsx
// state hook: [isOnline, setIsOnline]
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  // Specify how to clean up after this effect:
  return function cleanup() {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
});
```



**When exactly does React clean up an effect?** 

React performs the cleanup when the component unmounts. However, as we learned earlier, effects run for every render and not just once. This is why React *also* cleans up effects from the previous render before running the effects next time. 



### Tip: Optimizing Performance by Skipping Effects

> **若出现死循环执行effect的bug，需要传入一个空数组作为第二个参数**
>
> If you want to run an effect and clean it up only once (on mount and unmount), you can pass an empty array (`[]`) as a second argument. This tells React that your effect doesn’t depend on *any* values from props or state, so it never needs to re-run. 

You can tell React to *skip* applying an effect if certain values haven’t changed between re-renders. To do so, pass an array as an optional **second argument** to `useEffect`:

```jsx
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```

上例中如果两次渲染间`count`值不变的话，则不会执行effect

- make sure the array includes **all values from the component scope (such as props and state) that change over time and that are used by the effect**. Otherwise, your code will reference stale values from previous renders.
- If you want to run an effect and clean it up **only once** (on mount and unmount), you can pass an **empty array** (`[]`) as a second argument. This tells React that your effect doesn’t depend on *any* values from props or state, so it never needs to re-run. 



## UseRef: 获取dom & 存储变量

```js
const refContainer = useRef(initialValue);
...
refContainer.current = ...
```

`useRef` returns a mutable ref object whose `.current` property is initialized to the passed argument (`initialValue`). The returned object **will persist for the full lifetime of the component**.

You might be familiar with refs primarily as a way to [access the DOM](https://reactjs.org/docs/refs-and-the-dom.html). If you pass a ref object to React with `<div ref={myRef} />`, React will set its `.current` property to the corresponding DOM node whenever that node changes.

However, `useRef()` is useful for more than the `ref` attribute. It’s **[handy for keeping any mutable value around](https://reactjs.org/docs/hooks-faq.html#is-there-something-like-instance-variables)** similar to how you’d use instance fields in classes.



## `useCallback`

```js
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

Returns a [memoized](https://en.wikipedia.org/wiki/Memoization) callback.

Pass an inline callback and an array of dependencies. `useCallback` will return a memoized version of the callback that only changes if one of the dependencies has changed. This is useful when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders (e.g. `shouldComponentUpdate`).

`useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.

> Note
>
> The array of dependencies is not passed as arguments to the callback. Conceptually, though, that’s what they represent: every value referenced inside the callback should also appear in the dependencies array. In the future, a sufficiently advanced compiler could create this array automatically.
>
> We recommend using the [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) rule as part of our [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) package. It warns when dependencies are specified incorrectly and suggests a fix.



- 未用 useCallback 包起来的函数，每次渲染都会重新生成，会导致:
   a. 以该函数为依赖项的 useEffect，每次渲染都会执行副作用(加了等于白加) b. 如果该函数是子组件的props，那么会导致子组件每次都跟着重新渲染

- useCallback可以使得函数在每次渲染时都保持相同(同一个) 这样可以减少不必要的副作用，极大地避免子组件无意义的重新渲染

- useCallback 会根据 dependencies 参数是否变化，来决定是否重新生成function 应该结合实际应用场景，给 useCallback 添加正确的 dependencies

- 每次渲染都会重新定义普通函数和变量，如果函数作为依赖项，可能会导致useEffect被无意义执行 如果函数不依赖组件内部变量(state，props)，可以直接定义到组件外部去 如果依赖组件内部变量，则请正确使用 useCallback



## `useMemo`

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

Returns a [memoized](https://en.wikipedia.org/wiki/Memoization) value.

Pass a “create” function and an array of dependencies. `useMemo` will only recompute the memoized value when one of the dependencies has changed. This optimization helps to avoid expensive calculations on every render.

Remember that the function passed to `useMemo` runs during rendering. Don’t do anything there that you wouldn’t normally do while rendering. For example, side effects belong in `useEffect`, not `useMemo`.

If no array is provided, a new value will be computed on every render.

**You may rely on `useMemo` as a performance optimization, not as a semantic guarantee.** In the future, React may choose to “forget” some previously memoized values and recalculate them on next render, e.g. to free memory for offscreen components. Write your code so that it still works without `useMemo` — and then add it to optimize performance.

> Note
>
> The array of dependencies is not passed as arguments to the function. Conceptually, though, that’s what they represent: every value referenced inside the function should also appear in the dependencies array. In the future, a sufficiently advanced compiler could create this array automatically.
>
> We recommend using the [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) rule as part of our [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) package. It warns when dependencies are specified incorrectly and suggests a fix.





## 使用多个hook：必须按顺序

How does React know which state corresponds to which `useState` call? The answer is that **React relies on the order in which Hooks are called**. 

e.g. 使用多个hook

```jsx
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

调用顺序永远一致：

```js
// ------------
// First render
// ------------
useState('Mary')           // 1. Initialize the name state variable with 'Mary'
useEffect(persistForm)     // 2. Add an effect for persisting the form
useState('Poppins')        // 3. Initialize the surname state variable with 'Poppins'
useEffect(updateTitle)     // 4. Add an effect for updating the title

// -------------
// Second render
// -------------
useState('Mary')           // 1. Read the name state variable (argument is ignored)
useEffect(persistForm)     // 2. Replace the effect for persisting the form
useState('Poppins')        // 3. Read the surname state variable (argument is ignored)
useEffect(updateTitle)     // 4. Replace the effect for updating the title

// ...
```

As long as the order of the Hook calls is the same between renders, React can associate some local state with each of them



使用hooks要遵循以下规则 （[`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) ）：

#### Only Call Hooks at the Top Level

**Don’t call Hooks inside loops, conditions, or nested functions.** Instead, always use Hooks at the top level of your React function, before any early returns. By following this rule, you ensure that Hooks are called in the same order each time a component renders. That’s what allows React to correctly preserve the state of Hooks between multiple `useState` and `useEffect` calls. 

#### Only Call Hooks from React Functions

**Don’t call Hooks from regular JavaScript functions.** Instead, you can:

- ✅ Call Hooks from React function components.
- ✅ Call Hooks from custom Hooks 





## Custom Hooks

将使用`useState`和`useEffect`的逻辑再度进行抽象封装

下面这个组件使用了hook逻辑，订阅friend是否在线：

```jsx
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  // hooks logic
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
	// hooks logic ends
  
  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

这一部分逻辑在其他组件也会被用到，过去在react中就要使用HOC解决，现在可以将它封装成一个自定义hook `useFriendStatus`：

```jsx
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

然后就可以在任何组件中使用这段逻辑，和直接在组件中使用`useState`,`useEffect`等价

```jsx
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```
```jsx
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```



#### 注意：**Name your custom Hooks starting with “`use`”**



**Do two components using the same Hook share state?** 

No. Custom Hooks are a mechanism to reuse *stateful logic* (such as setting up a subscription and remembering the current value), but every time you use a custom Hook, all state and effects inside of it are fully isolated.



### IMPORTANT: hook update is ASYCHRONOUS !!!

https://stackoverflow.com/questions/55983047/strange-behavior-of-react-hooks-delayed-data-update

```tsx
// switch to next event
  const handleClickNext = () => {
    if (eventIndex === store.crashEventList.length - 1) {
      setEventIndex(0);
    } else {
      setEventIndex(eventIndex + 1);
    }
    setCurrentEvent(store.crashEventList[eventIndex]);
  };
```

上面代码里`setCurrentEvent(store.crashEventList[eventIndex]`里面的`eventIndex`**不是最新的值**



### Infinite loop with `useEffect()`

如果在useEffect中更新state，会触发页面渲染，使useEffect被重新执行，进入无限循环。

[How to Solve the Infinite Loop of React.useEffect()](https://dmitripavlutin.com/react-useeffect-infinite-loop/)

解决办法：使用dependency array参数，明确要监听的属性，避免多余执行



## 第三方hook

useSearchParams



## 使用hooks注意

- 保证 State 完整性的同时，也要保证它的最小化
- 数据如果能从已有的 State 中计算得到，那么我们就应该始终在用的时候去计算，而不要把计算的

结果存到某个 State 中



## forceUpdate with hook

[javascript - How can I force a component to re-render with hooks in React? - Stack Overflow](https://stackoverflow.com/questions/53215285/how-can-i-force-a-component-to-re-render-with-hooks-in-react)

write a hook like `const [, forceUpdate] = useState({});` to force a re-render manually!

```js
const [, forceUpdate] = useState({});
...
// in code
forceUpdate({})
```



