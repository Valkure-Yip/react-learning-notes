# MobX intro

> state management solution of React

MobX本质上是帮助开发者建立一个独立于react组件的发布订阅模式。通过`makeObservable`等方法，创建一个可订阅数据结构，并通过`mobx-react`中的`observe()`创建HOC，来让react组件订阅这些数据结构变化。

## core concepts

* State 
* Derivations: computed values from states
* Reaction 
* Actions : change the states

![img](/Users/zhitong.ye/Desktop/开发技术笔记/react-learning-notes/MobX.assets/overview.png)

## 上手例子: todoList

> MobX: Ten minute introduction to MobX and React https://mobx.js.org/getting-started

比如要通过mobx来管理一个todo list的数据和数据更改操作，先创建一个`observable`对象作为发布者。

### 建立 observable

创建一个todoStore类，在constructor里使用`makeObservable`函数来创建observable。

#### `makeObservable(target, annotations?, options?)`

在annotation中用不同关键词(`observable|computed|action...`)来标明数据和方法等。注意`observable`作为基础数据，必须先以class property的形式定义好。



```js
// ./store/index.js
class TodoStore {
  todos = [];
  pendingRequests = 0;
  constructor() {
    makeObservable(this, {
      todos: observable,
      pendingRequests: observable,
      completedTodosCount: computed,
      report: computed,
      addTodo: action,
    });
    autorun(() => console.log(this.report));
  }

  get completedTodosCount() {
    return this.todos.filter((todo) => todo.completed === true).length;
  }

  get report() {
    if (this.todos.length === 0) return "<none>";
    const nextTodo = this.todos.find((todo) => todo.completed === false);
    return (
      `Next todo: "${nextTodo ? nextTodo.task : "<none>"}". ` +
      `Progress: ${this.completedTodosCount}/${this.todos.length}`
    );
  }

  addTodo(task) {
    this.todos.push({
      task: task,
      completed: false,
      assignee: null
    });
  }
}
```

之后将`todoStore`实例化，生成的observable对象就可以用于管理数据变化了。如果在代码中触发action，则mobx会自动计算出computed结果，以及执行reaction

```js
observableTodoStore.addTodo("read MobX tutorial");
observableTodoStore.addTodo("try MobX");
observableTodoStore.todos[0].completed = true;
observableTodoStore.todos[1].task = "try MobX in own project";
observableTodoStore.todos[0].task = "grok MobX tutorial";
```

```
Next todo: "read MobX tutorial". Progress: 0/1
Next todo: "read MobX tutorial". Progress: 0/2
Next todo: "try MobX". Progress: 1/2
Next todo: "try MobX in own project". Progress: 1/2
```



#### `makeAutoObservable(target, overrides?, options?)`

Inference rules:

- All *own* properties become `observable`.
- All `get`ters become `computed`.
- All `set`ters become `action`.
- All *functions on prototype* become `autoAction`.
- All *generator functions on prototype* become `flow`. (Note that generator functions are not detectable in some transpiler configurations, if flow doesn't work as expected, make sure to specify `flow` explicitly.)
- Members marked with `false` in the `overrides` argument will not be annotated. For example, using it for read only fields such as identifiers.

####  `observable(source, overrides?, options?)`

The `source` object will be cloned and all members will be made observable, similar to how it would be done by `makeAutoObservable`



### action： 改变state

Usage:

- `action` *(annotation)*
- `action(fn)`
- `action(name, fn)`
- `runInAction(fn)` 立即执行

### computed

get定义computed

### reaction

`autorun()`

`reaction()`

`(async) when()`



### 在组件中订阅变化

Mobx提供了`mobx-react`和`mobx-react-lite`包，能够用高阶组件封装react组件，使其能够订阅observable变化。

用`observer`封装react组件，即可订阅observable变化，无论observable以何种形式被组件引用，比如：props，全局变量或context注入。

`observer(component)`

```jsx
// component/todoList.js
const { observer } = require("mobx-react");

const TodoList = observer(({ store }) => {
  const onNewTodo = () => {
    store.addTodo(prompt("Enter a new todo:", "coffee plz"));
  };

  return (
    <div>
      {store.report}
      <ul>
        {store.todos.map((todo, idx) => (
          <TodoView todo={todo} key={idx} />
        ))}
      </ul>
      {store.pendingRequests > 0 ? <marquee>Loading...</marquee> : null}
      <button onClick={onNewTodo}>New Todo</button>
      <small> (double-click a todo to edit)</small>
    </div>
  );
});

const TodoView = observer(({ todo }) => {
  const onToggleCompleted = () => {
    todo.completed = !todo.completed;
  };

  const onRename = () => {
    todo.task = prompt("Task name", todo.task) || todo.task;
  };

  return (
    <li onDoubleClick={onRename}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={onToggleCompleted}
      />
      {todo.task}
      {todo.assignee ? <small>{todo.assignee.name}</small> : null}
    </li>
  );
});

export default TodoList;
```



```jsx
// App.js

<TodoList store={todoStore}></TodoList>
```

### Using local observable state in `observer` components