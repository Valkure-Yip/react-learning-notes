# Redux

- The [**Redux Essentials tutorial**](https://redux.js.org/tutorials/essentials/part-1-overview-concepts) is a "top-down" tutorial that teaches "how to use Redux the right way", using our latest recommended APIs and best practices.
- The [**Redux Fundamentals tutorial**](https://redux.js.org/tutorials/fundamentals/part-1-overview) is a "bottom-up" tutorial that teaches "how Redux works" from first principles and without any abstractions, and why standard Redux usage patterns exist.

## Install

[Getting Started with Redux | Redux](https://redux.js.org/introduction/getting-started)

### Redux toolkit

Redux Toolkit is available as a package on NPM for use with a module bundler or in a Node application:

```bash
# NPM
npm install @reduxjs/toolkit

# Yarn
yarn add @reduxjs/toolkit
```

### Create a React Redux App

The recommended way to start new apps with React and Redux is by using the [official Redux+JS template](https://github.com/reduxjs/cra-template-redux) or [Redux+TS template](https://github.com/reduxjs/cra-template-redux-typescript) for [Create React App](https://github.com/facebook/create-react-app), which takes advantage of **[Redux Toolkit](https://redux-toolkit.js.org/)** and React Redux's integration with React components.

```bash
# Redux + Plain JS template
npx create-react-app my-app --template redux

# Redux + TypeScript template
npx create-react-app my-app --template redux-typescript
```

### Redux Core

The Redux core library is available as a package on NPM for use with a module bundler or in a Node application:

```bash
# NPM
npm install redux

# Yarn
yarn add redux
```

## Action, action creator, reducer, store, dispatch, selector

[Terminology](https://redux.js.org/tutorials/essentials/part-1-overview-concepts#terminology)

[Redux Application Data Flow](https://redux.js.org/tutorials/essentials/part-1-overview-concepts#redux-application-data-flow)

![Redux data flow diagram](/Users/zhitong.ye/Desktop/zhitong_dev_notes/react-learning-notes/Redux.assets/ReduxDataFlowDiagram-49fa8c3968371d9ef6f2a1486bd40a26.gif)



## App structure

example code :[Redux Essentials, Part 2: Redux App Structure | Redux ](https://redux.js.org/tutorials/essentials/part-2-app-structure#the-counter-example-app) (created with CRA redux template)

- ```
  /src
  ```

  - `index.js`: the starting point for the app

  - `App.js`: the top-level React component

  - ```
    /app
    ```

    - `store.js`: creates the Redux store instance

  - ```
    /features
    ```

    - ```
      /counter
      ```

      - `Counter.js`: a React component that shows the UI for the counter feature
      - `counterSlice.js`: the Redux logic for the counter feature



### Store

The Redux store is created using the `configureStore` function from Redux Toolkit. `configureStore` requires that we pass in a `reducer` argument.

```js
// app/store.js
import { configureStore } from '@reduxjs/toolkit'
import counterReducer from '../features/counter/counterSlice'

export default configureStore({
  reducer: {
    counter: counterReducer
  }
})
```

When we pass in an object like `{counter: counterReducer}`, that says that we want to have a `state.counter` section of our Redux state object, and that we want the `counterReducer` function to be in charge of deciding if and how to update the `state.counter` section whenever an action is dispatched.

### Slice & Reducer

`createSlice` takes care of the work of **generating action type strings, action creator functions, and action objects**. 

`name` option is used as the first part of each action type, and the key name of each reducer function is used as the second part. So, the `"counter"` name + the `"increment"` reducer function generated an action type of `{type: "counter/increment"}`

`initialState `: needs us to pass in the initial state value for the reducers

`reducers` reducer functions (see below)

```
createSlice({
	name,
	initialState,
	reducers
})
```


```js
// features/counter/counterSlice.js
import { createSlice } from '@reduxjs/toolkit'

export const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: 0
  },
  reducers: {
    increment: state => {
      // Redux Toolkit allows us to write "mutating" logic in reducers. It
      // doesn't actually mutate the state because it uses the immer library,
      // which detects changes to a "draft state" and produces a brand new
      // immutable state based off those changes
      state.value += 1
    },
    decrement: state => {
      state.value -= 1
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload
    }
  }
})

export const { increment, decrement, incrementByAmount } = counterSlice.actions

export default counterSlice.reducer
```

`createSlice` automatically generates **action creators** with the same names as the reducer functions we wrote:

```js
console.log(counterSlice.actions.increment())
// {type: "counter/increment"}
```

using slice **reducer** function that knows how to respond to all these action types:

```js
const newState = counterSlice.reducer(
  { value: 10 },
  counterSlice.actions.increment()
)
console.log(newState)
// {value: 11}
```



### Reducers and Immutable Updates

https://redux.js.org/tutorials/essentials/part-2-app-structure#reducers-and-immutable-updates

**reducers are *never* allowed to mutate the original / current state values: Immutable changes**

**You can *only* write "mutating" logic in Redux Toolkit's `createSlice` and `createReducer` because they use Immer inside! If you write mutating logic in reducers without Immer, it \*will\* mutate the state and cause bugs!**



### use redux in component

`react-redux` hooks : `useDispatch`, `useSelector`	

```tsx
import React, { useState } from 'react'
import { useSelector, useDispatch } from 'react-redux'
import {
  decrement,
  increment,
  incrementByAmount,
  incrementAsync,
  selectCount
} from './counterSlice'
import styles from './Counter.module.css'

export function Counter() {
  const count = useSelector(selectCount)
  const dispatch = useDispatch()
  const [incrementAmount, setIncrementAmount] = useState('2')

  return (
    <div>
      <div className={styles.row}>
        <button
          className={styles.button}
          aria-label="Increment value"
          onClick={() => dispatch(increment())}
        >
          +
        </button>
        <span className={styles.value}>{count}</span>
        <button
          className={styles.button}
          aria-label="Decrement value"
          onClick={() => dispatch(decrement())}
        >
          -
        </button>
      </div>
      {/* omit additional rendering output here */}
    </div>
  )
}

```



### [Providing the Store](https://redux.js.org/tutorials/essentials/part-2-app-structure#providing-the-store)

We've seen that our components can use the `useSelector` and `useDispatch` hooks to talk to the Redux store. But, since we didn't import the store, how do those hooks know what Redux store to talk to?

Now that we've seen all the different pieces of this application, it's time to circle back to the starting point of this application and see how the last pieces of the puzzle fit together.

index.js

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import './index.css'
import App from './App'
import store from './app/store'
import { Provider } from 'react-redux'
import * as serviceWorker from './serviceWorker'

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```



###  Async Logic and Data Fetching

Redux Toolkit's `configureStore` function [automatically sets up the thunk middleware by default](https://redux-toolkit.js.org/api/getDefaultMiddleware#included-default-middleware), so we can pass *thunk functions* directly to `store.dispatch` 

![Redux async data flow diagram](/Users/zhitong.ye/Desktop/zhitong_dev_notes/react-learning-notes/Redux.assets/ReduxAsyncDataFlowDiagram-d97ff38a0f4da0f327163170ccc13e80.gif)



#### thunk function

[What the heck is a 'thunk'?](https://daveceddia.com/what-is-a-thunk/)

在同步情景里我们通过dispatch触发一个action给reducer，action只是一个对象，包含数据变化。比如

 ```js
 // someActionCreator(): action object
 dispatch(someActionCreator())
 ```

 但如果我们想执行更复杂的操作，就需要dispatch一个函数（比如异步操作），并由一个中间件（redux-thunk）执行这个函数，在执行过程中触发一个个action传给reducer，这个函数就是thunk

 ```js
 // someThunkCreator
 dispatch(someThunkCreator())
 ```

 

A **thunk function** will always be called with `(dispatch, getState)` as its arguments, and you can use them inside the thunk as needed.

Thunks typically dispatch plain actions using action creators, like `dispatch(increment())`:

```js
const store = configureStore({ reducer: counterReducer })

const exampleThunkFunction = (dispatch, getState) => {
  const stateBefore = getState()
  console.log(`Counter before: ${stateBefore.counter}`)
  dispatch(increment())
  const stateAfter = getState()
  console.log(`Counter after: ${stateAfter.counter}`)
}

store.dispatch(exampleThunkFunction)
```

For consistency with dispatching normal action objects, we typically write these as **thunk action creators**, which return the thunk function. These action creators can take arguments that can be used inside the thunk.

```js
const logAndAdd = amount => {
  return (dispatch, getState) => {
    const stateBefore = getState()
    console.log(`Counter before: ${stateBefore.counter}`)
    dispatch(incrementByAmount(amount))
    const stateAfter = getState()
    console.log(`Counter after: ${stateAfter.counter}`)
  }
}

store.dispatch(logAndAdd(5))
```



#### ``createAsyncThunk`` &`extraReducers`

异步操作通常以下几步，我们需要在每一部的时候让*thunk function* dispatch一个action

- A "start" action is dispatched before the request, to indicate that the request is in progress. This may be used to track loading state to allow skipping duplicate requests or show loading indicators in the UI.
- The async request is made
- Depending on the request result, the async logic dispatches either a "success" action containing the result data, or a "failure" action containing error details. The reducer logic clears the loading state in both cases, and either processes the result data from the success case, or stores the error value for potential display.

`createAsyncThunk`可以生成这样一个thunk function，automatically dispatch those "start/success/failure" actions for you.

`createAsyncThunk` accepts two arguments:

- A string that will be used as the **prefix** for the generated action types
- A "payload creator" callback function that should return a **`Promise`** containing some data, or a rejected `Promise` with an error

```js
import { createSlice, nanoid, createAsyncThunk } from '@reduxjs/toolkit'
import { client } from '../../api/client'

const initialState = {
  posts: [],
  status: 'idle',
  error: null
}

export const fetchPosts = createAsyncThunk('posts/fetchPosts', async () => {
  const response = await client.get('/fakeApi/posts')
  return response.data
})
```

```js
dispatch(fetchPosts())
```

`fetchPosts`会在promise不同阶段dispatch不同的action (pending/fufilled/rejected):

```
posts/fetchPosts/pending
posts/fetchPosts/fufilled
posts/fetchPosts/rejected
```



在`createSlice`的`extraReducers`里面配置相应reducer来处理这些action

```js
export const fetchPosts = createAsyncThunk('posts/fetchPosts', async () => {
  const response = await client.get('/fakeApi/posts')
  return response.data
})

const postsSlice = createSlice({
  name: 'posts',
  initialState,
  reducers: {
    // omit existing reducers here
  },
  extraReducers(builder) {
    builder
      .addCase(fetchPosts.pending, (state, action) => {
        state.status = 'loading'
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.status = 'succeeded'
        // Add any fetched posts to the array
        state.posts = state.posts.concat(action.payload)
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.status = 'failed'
        state.error = action.error.message
      })
  }
})
```

We'll handle all three action types that could be dispatched by the thunk, based on the `Promise` we returned:

- When the request starts, we'll set the `status` enum to `'loading'`
- If the request succeeds, we mark the `status` as `'succeeded'`, and add the fetched posts to `state.posts`
- If the request fails, we'll mark the `status` as `'failed'`, and save any error message into the state so we can display it
