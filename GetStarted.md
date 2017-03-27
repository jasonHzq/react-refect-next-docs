# 起步

## 核心概念

### refect

refect 是对 redux 的一个扩展。refect 完全遵循 redux 三大原则。

refect 是一个分形框架。一个 refect 组件包含了该组件的行为（reducer、actions）和View。并以他们的组合形态让父级使用。

一个 refect 组件里可以包含 refect 组件。

### react-refect

react-refect 是 refect 对 View 层为 React 的情况做得一个支持。

### react-refect-next

react-refect-next 是在 react-refect 的基础上做了 ts 支持。因此 react-refect-next 中的 API 、refect 组件的数据获取，在支持 ts 的编辑器中都有自动提示和类型校验，自动提示中也包含了 API 的文档和参数规定。

不熟悉 ts 的同学可参考 [ts中文文档](https://www.tslang.cn/docs/handbook/basic-types.html)

本文档主要介绍 react-refect-next，一下简称 next 。

## State

相当于 redux 中的 initialState。next 以 class 的方式，在声明初始值也定义了类型。

```ts
class State {
    count = 0
}
```

## Reducer

Reducer 类中的每个方法，都是 reducer 对应每个 action 的处理逻辑。

Reducer 的方法也可以包含一个或多个参数，它们会以数组的形式放到 action 的 payload 中。

#### 代码-1.1

```ts
import { BaseReducer } from 'react-refect-next';

class Reducer extends BaseReducer<State> {
    addCount() {
        return {
            ...this.state,
            count: this.state.count + 1,
        };
    }
}
```

代码-1.1 等同于 代码-1.2:

```js
function reducer(state, action) {
  const { type } = action;

  switch(type) {
    case 'addOne': {
      return {
        ...state,
        count: state.count + 1,
      };
    }
    default: {
      return state;
    }
  }
}
```

## Tasks

Tasks 负责异步处理。可以理解为 redux-thunk。

在 Tasks 方法中，可以通过 this.get() 获取 State 数据，也可以通过this.get('a.b') 直接获取深层数据，但是这种做法不能享受State数据结构提示和类型校验；

可以通过 this.actions.someAction(...args) 来 dispatch 这个 'someAction'。

可以通过 this.dispatch 方法  dispatch 外部 action，例如 redux-react-router 中定义的 action

Reducer 和 Tasks 中的方法都会被统计到 this.actions 中。

Tasks 的方法支持 async await，前提是你的应用有 generator-runtime。

Tasks 的方法也支持 redux-saga，前提是你的应用的 store 要使用 refect-saga-middleware。

```ts
import { BaseTasks } from 'react-refect-next';

class Tasks extends BaseTasks<Tasks & Reducer, State, any> {
    addIfOdd() {
        if (this.get().count % 2) {
            this.actions.addCount();
        }
    }
}
```

## Effects

Tasks 可以借助 effects 来处理异步逻辑。

可以使用的 Effects 可以由顶层传入，也可以由该 Refect 组件插入。

参看 [@ali/refect-fetch-effect](http://gitlab.alibaba-inc.com/dt-npm/refect-fetch-effect)

## View

```tsx
class Props extends State {
  actions: Tasks & Reducer
}

class View extends React.Component<Props, any> {
  render() {
    return (
      <div>
        num: {num}
        <button onClick={actions.addOne}>addOne</button>
        <button onClick={actions.addIfOdd}>addIfOdd</button>
      </div>
    );
  }
}
```

## refect

you can use refectLocal to create Root view and refect to create children view;

#### refectLocal

refectLocal will create redux store for you.

```jsx
import { refectLocal } from 'react-refect';

const LocalCounterRex = refectLocal({
  view: Counter,
  tasks,
  reducer,
  initialState: { num: 0 },
});

ReactDOM.render(<LocalCounterRex />, document.getElementById('app'));
```

## refect

使用 refect 函数生成一个 Component。

```ts
import refect from 'react-refect-next';

const Counter = refect({
  State,
  Reducer,
  Tasks,
  View,
  defaultNamespace: 'counter',
});

export default Counter;

```

defaultNamespace 为该 Refect 组件的默认 namespace ，父级也可以取一个新的 namespace 来替换。例如在同一个 View 中使用同一个 Refect 组件多次时，为了让 每个 Refect 组件都有一个自己的 state，需要有不同的 namespace：

```tsx
function ParentView() {
  return (
    <div>
      <Counter />
      <Counter namespace="counter2" />
      <Counter namespace="counter3" />
    </div>
  );
}
```


## refectLocal

与 refect 函数相同，但 refectLoal 会提供一个 store 。一般可在顶层使用。

