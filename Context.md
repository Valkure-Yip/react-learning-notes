# Context

> 对应Vue中Provide/inject
>
> Context – React https://zh-hans.reactjs.org/docs/context.html

## 使用Context

在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但此种用法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题），这些属性是应用程序中许多组件都需要的。Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props。

Context 设计目的是为了共享那些**对于一个组件树而言是“全局”**的数据

假设有一个按钮组件，通过一个 “theme” 属性手动调整按钮组件的样式：

```jsx
<ThemedButton theme={props.theme} />
```

使用 context, 我们可以避免通过中间元素传递 props：

`React.createContext()`

`<MyContext.Provider>`

`static contextType = ...`

`{this.context}`

```jsx
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext = React.createContext('light');
class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    // 无论多深，任何组件都能读取这个值。
    // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // 指定 contextType 读取当前的 theme context。
  // React 会往上找到最近的 theme Provider，然后使用它的值。
  // 在这个例子中，当前的 theme 值为 “dark”。
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```



## Context API

### `React.createContext`

```jsx
const MyContext = React.createContext(defaultValue);
```

创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 `Provider` 中读取到当前的 context 值。

**只有**当组件所处的树中没有匹配到 Provider 时，其 `defaultValue` 参数才会生效。此默认值有助于在不使用 Provider 包装组件的情况下对组件进行测试。注意：将 `undefined` 传递给 Provider 的 value 时，消费组件的 `defaultValue` 不会生效。

### `Context.Provider`

```jsx
<MyContext.Provider value={/* 某个值 */}>
```

每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化。

Provider 接收一个 `value` 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。

当 Provider 的 `value` 值发生变化时，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 `shouldComponentUpdate` 函数，因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。

通过新旧值检测来确定变化，使用了与 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) 相同的算法。

> 注意
>
> 当传递对象给 `value` 时，检测变化的方式会导致一些问题：详见[注意事项](https://zh-hans.reactjs.org/docs/context.html#caveats)。

### `Class.contextType`

```jsx
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* 在组件挂载完成后，使用 MyContext 组件的值来执行一些有副作用的操作 */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* 基于 MyContext 组件的值进行渲染 */
  }
}
MyClass.contextType = MyContext;
```

挂载在 class 上的 `contextType` 属性会被重赋值为一个由 [`React.createContext()`](https://zh-hans.reactjs.org/docs/context.html#reactcreatecontext) 创建的 Context 对象。此属性能让你使用 `this.context` 来消费最近 Context 上的那个值。你可以在任何生命周期中访问到它，包括 render 函数中。

> 注意：
>
> 你只通过该 API 订阅单一 context。如果你想订阅多个，阅读[使用多个 Context](https://zh-hans.reactjs.org/docs/context.html#consuming-multiple-contexts) 章节
>
> 如果你正在使用实验性的 [public class fields 语法](https://babeljs.io/docs/plugins/transform-class-properties/)，你可以使用 `static` 这个类属性来初始化你的 `contextType`。

```jsx
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    let value = this.context;
    /* 基于这个值进行渲染工作 */
  }
}
```

### `Context.Consumer`

```jsx
<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>
```

一个可以订阅 context 的变更的React 组件，此组件可以让你在[**函数式组件**](https://zh-hans.reactjs.org/docs/components-and-props.html#function-and-class-components)中可以订阅 context。

> 函数式组件中没法写 `static contextType = MyContext;`

这种方法需要一个[函数作为子元素（function as a child）](https://zh-hans.reactjs.org/docs/render-props.html#using-props-other-than-render)。这个函数接收当前的 context 值，并返回一个 React 节点。传递给函数的 `value` 值等价于组件树上方离这个 context 最近的 Provider 提供的 `value` 值。如果没有对应的 Provider，`value` 参数等同于传递给 `createContext()` 的 `defaultValue`。

