
## 1. Redux

> Redux 是 JavaScript 状态容器，提供可预测化的状态管理。

## 2. 三条原则

- **应用程序状态存储在单个对象（Store）中。**
- **应用程序状态不可变，只能通过描述状态更改的操作（Action）彻底替换。**
- **缩减程序（Reducer）根据当前状态和某个操作来创建下一个状态。**

一、Redux 通过一个 JavaScript 对象管理状态，该对象称为数据存储，包含应用程序的所有状态。将状态集中保存在一个对象中，这使得在阅读代码时推断应用程序数据变得更容易。另外，当所有数据都在一个位置时，应用程序更容易调试和测试。

二、对于 Redux，您不能修改应用程序状态。而需要使用新状态替换现有状态。新状态由操作指定，操作也是不可变的 JavaScript 对象，用于描述状态更改。将状态更改封装在不可变的对象中有许多优点，其中一个优点是能够实现无限撤销和重做 — 实际上类似于时光机。操作也按严格的顺序执行，所以不会发生竞争条件。

三、缩减程序是纯 JavaScript 函数，它们：
- 根据当前状态和某个操作来创建一个新状态
- 集中化数据变化
- 可处理所有或部分状态
- 可组合和重用

因为缩减程序是纯函数，缩减程序没有副作用 — 所以它们很容易读取、测试和调试。您可以构造缩减程序，轻松实现仅关注整体应用程序状态的一部分的简单缩减程序。

因为应用程序`状态不可变`，而且因为缩减程序是没有副作用的`纯函数`，所以 Redux 是一个`可预测的`状态容器：给定任何状态和任何操作，您就可以非常肯定地预测应用程序的下一个状态。进而使实现无限撤销/重做和实时编辑时间旅行成为可能。

## 3. 概念

![redux](redux.png)

### 3.1 Store

- 保存数据的地方，你可以把它看成一个容器。
- 一个应用程序中只能有一个。
- Redux提供`createStore`这个函数，用来生成 Store。
- Store对象包含所有数据。如果想得到某个时点的数据`Store.getState()`，就要对 Store 生成快照。这种时点的数据集合，就叫做 `State`。
- 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。

### 3.2 Action

- 一个形式为 `{type: 'ACTION_NAME', payload: something}` 的普通 JavaScript 对象。
- Action 就是 View 发出的通知，描述 State 应该要发生变化了。（State 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。）
- 要想更新 state 中的数据，你需要发起一个 Action。它会运送数据到 Store。
- type属性：是必须的，表示 Action 的名称。payload属性：表示携带的信息，非必须。

### 3.3 Reducer

- 形式为 `(state, action) => state` 的纯函数。
- Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。

## 4. 不可变和纯函数

```js
/*
更新数组b也会同时改变数组a。
因为对象和数组是引用数据类型 → 这意味着这样的数据类型实际上并不保存值，而是存储指向存储单元的指针。
将a赋值给b，其实我们只是创建了第二个指向同一存储单元的指针。
*/
let a = [1, 2, 3]
let b = a
b.push(8)
b // [1, 2, 3, 8]
a // [1, 2, 3, 8]
```

```js
/*
要解决这个问题，我们需要将引用的值复制到一个新的存储单元。
1. 使用 Object.assign() 创建了一个新的副本，由数组b指向。
2. 也可以使用操作符(...)执行不可变操作
*/
a = [1,2,3]
b = Object.assign([],a)
b.push(8)
b // [ 1, 2, 3, 8 ]
a // [ 1, 2, 3 ]

a = [1,2,3]
b = [...a, 4, 5, 6]
b // [ 1, 2, 3, 4, 5, 6 ]
a // [ 1, 2, 3 ]
```

## 5. 数据流

![dispatch](dispatch.png)

## 6. 安装 

```bash
npm install --save redux
npm install --save react-redux
npm install --save-dev redux-devtools
```

## 7. API 

- React
  - Redux
    - `Object createStore(reducer, initialState)` - 创建 Redux 存储
      - `Object getState()` 返回应用程序的当前状态
      - `void dispatch(Objectaction)` 分派一个操作，触发一次状态更改
      - `subscribe(Functioncallback)` 导致 Redux 调用每次分派的回调方法
      - `replaceReducer(nextReducer)` 替换状态树的缩减程序
    - `Object combineReducers(reducers)` - 将多个缩减程序组合为一个 
      - rootReducer
    - `Object bindActionCreators(actionCreators, dispatch)` - 将多个操作创建器绑定到分派函数
    - `void applyMiddleware(...middlewares)` - 应用 Redux 中间件
    - `Object compose(...functions)` - 从左向右构造函数
  - react-redux
    - `Provider` 组件允许 Provider 组件中包含的组件访问 Redux 存储
    - `void connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])`  - 将一个 React 表示组件连接到 Redux 存储

### React-Redux

Redux 是独立的，它与React没有任何关系。React-Redux是官方提供的一个库，用来结合redux和react的模块。

React-Redux提供了两个接口Provider、connect。

将抓取和显示数据的操作混合在一个组件中，这会让该组件很难测试。相反，可通过一个容器组件来`分离关注点`，该组件抓取数据并传递给关联的表示组件。表示组件是一个无状态组件，仅显示数据且容易测试。此方法实质上实现了 `被动视图设计模式`。


## 参考

[使用 Redux 管理状态](https://www.ibm.com/developerworks/cn/web/wa-manage-state-with-redux-p1-david-geary/index.html)