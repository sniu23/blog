
# [React in patterns 中文版 摘录](https://github.com/SangKa/react-in-patterns-cn)

# 属性

- `propTypes` ：定义每个属性的类型
- `defaultProps` ：为组件的属性设置默认值

```jsx
// Title.jsx
function Title(props) {
  return <h1>{ props.text }</h1>;
} 
Title.propTypes = {
  text: PropTypes.string
};
Title.defaultProps = {
  text: 'Hello world'
};

// App.jsx
function App() {
  return <Title text='Hello React' />;
}
```

- `属性`可以是一个组件

```jsx
function SomethingElse({ answer }) {
  return <div>The answer is { answer }</div>;
}
function Answer() {
  return <span>42</span>;
}

<SomethingElse answer={ <Answer /> } />
```

- `props.children` 属性，它可以让我们访问父组件标签内的子元素

```jsx
function Title({ text, children }) {
  return (
    <h1>
      { text }
      { children }
    </h1>
  );
}
function App() {
  return (
    <Title text='Hello React'>
      <span>community</span>
    </Title>
  );
}
```

# 事件

- 传入的`属性`可以是任何东西，包括`函数`，我们可以使用它来发送数据或触发操作。

**important**
```jsx
// <NameField />，它接受用户的输入并能将结果发送出去。
function NameField({ valueUpdated }) {
  return (
    <input
      onChange={ event => valueUpdated(event.target.value) } />
  );
};
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = { name: '' };
  }
  render() {
    return (
      <div>
        <NameField
          valueUpdated={ name => this.setState({ name }) } />
        Name: { this.state.name }
      </div>
    );
  }
};
```

- **受控组件** ：避免使用`refs`来让浏览器来处理用户的输入(这种方式称为`非受控输入`）

**important**
```jsx
// 受控组件
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
    this._change = this._handleInputChange.bind(this);
  }
  render() {
    return (
      <input
        type='text'
        value={ this.state.value }
        onChange={ this._change } />
    );
  }
  _handleInputChange(e) {
    this.setState({ value: e.target.value });
  }
};
```

```jsx
class Form extends React.Component {
  constructor(props) {
    super(props);
    this._onNameChanged = this._onFieldChange.bind(this, 'name');
    this._onPasswordChanged =
      this._onFieldChange.bind(this, 'password');
  }
  render() {
    return (
      <form>
        <input onChange={ this._onNameChanged } />
        <input onChange={ this._onPasswordChanged } />
      </form>
    );
  }
  _onFieldChange(field, event) {
    console.log(`${ field } changed to ${ event.target.value }`);
  }
};
```

# 组合

- 优先组合，避免层层嵌套的方式。

```jsx
// 层次嵌套: 依赖紧密，难以单独测试
// app.jsx
import Header from './Header.jsx';

export default function App() {
  return <Header />;
}

// Header.jsx
import Navigation from './Navigation.jsx';

export default function Header() {
  return <header><Navigation /></header>;
}

// Navigation.jsx
export default function Navigation() {
  return (<nav> ... </nav>);
}
```

1. 使用 `children`

**important**
```jsx
function TodoList({ todos, children }) {
  return (
    <section className='main-section'>
      <ul className='todo-list'>{
        todos.map((todo, i) => (
          <li key={ i }>{ children(todo) }</li>
        ))
      }</ul>
    </section>
  );
}

function App() {
  const todos = [
    { label: 'Write tests', status: 'done' },
    { label: 'Sent report', status: 'progress' },
    { label: 'Answer emails', status: 'done' }
  ];
  const isCompleted = todo => todo.status === 'done';
  return (
    <TodoList todos={ todos }>
      {
        todo => isCompleted(todo) ?
          <b>{ todo.label }</b> :
          todo.label
      }
    </TodoList>
  );
}
```

2. 使用 `render`

```jsx
function TodoList({ todos, render }) {
  return (
    <section className='main-section'>
      <ul className='todo-list'>{
        todos.map((todo, i) => (
          <li key={ i }>{ render(todo) }</li>
        ))
      }</ul>
    </section>
  );
}

return (
  <TodoList
    todos={ todos }
    render={
      todo => isCompleted(todo) ?
        <b>{ todo.label }</b> : todo.label
    } />
);
```

**高阶函数**： 看上去与 装饰器模式 十分相似,它接收原始组件并返回原始组件的增强/填充版本。

```jsx
var config = require('path/to/configuration');

var enhanceComponent = (Component) =>
  class Enhance extends React.Component {
    render() {
      return (
        <Component
          {...this.props}
          title={ config.appTitle }
        />
      )
    }
  };

var OriginalTitle  = ({ title }) => <h1>{ title }</h1>;
var EnhancedTitle = enhanceComponent(OriginalTitle);
```

# 展示组件和容器组件

```jsx
// 容器型组件
// Clock/index.js
import Clock from './Clock.jsx'; // <-- 展示型组件

export default class ClockContainer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { time: props.time };
    this._update = this._updateTime.bind(this);
  }
  render() {
    return <Clock { ...this._extract(this.state.time) }/>;
  }
  componentDidMount() {
    this._interval = setInterval(this._update, 1000);
  }
  componentWillUnmount() {
    clearInterval(this._interval);
  }
  _extract(time) {
    return {
      hours: time.getHours(),
      minutes: time.getMinutes(),
      seconds: time.getSeconds()
    };
  }
  _updateTime() {
    this.setState({
      time: new Date(this.state.time.getTime() + 1000)
    });
  }
};

// 展示型组件
// Clock/Clock.jsx
export default function Clock(props) {
  var [ hours, minutes, seconds ] = [
    props.hours,
    props.minutes,
    props.seconds
  ].map(num => num < 10 ? '0' + num : num);

  return <h1>{ hours } : { minutes } : { seconds }</h1>;
};
```

**好处**： 
将组件分成容器型组件和展示型组件可以增加组件的可复用性。测试也将变得更容易，因为组件承担的职责更少。

容器型组件容器型组件知道数据及其结构，以及数据的来源。它们知道是如何运转的，或所谓的业务逻辑。它们接收信息并对其进行处理，以方便展示型组件使用。它们可以搭配不同的展示型组件使用，因为它们不参与任何展示相关的工作。

展示型组件只是纯粹地负责展示，它可以很好地预测出渲染后的 HTML 标记。

# 样式

1. 经典 CSS 类: `className`。 (JSX 语法，使用定义在外部 .css 文件中的 CSS 类来处理样式。）

```jsx
<h1 className='title'>Styling</h1>
```

2. 内联样式： `style`。（内联样式、必须得是一个对象，使用 JavaScript 编写样式, 比普通的 CSS 更具灵活性）

```jsx
const inlineStyles = {
  color: 'red',
  fontSize: '10px',
  marginTop: '2em',
  'border-top': 'solid 1px #000'
};

<h2 style={ inlineStyles }>Inline styling</h2>

const theme = {
  fontFamily: 'Georgia',
  color: 'blue'
};
const paragraphText = {
  ...theme,
  fontSize: '20px'
};
```

3. `css-modules`: 如果你不喜欢 JavaScript 用法来写 CSS ，那么可以使用 CSS 模块，它可以让我们继续编写普通的 CSS 。通常，这个库是在打包阶段发挥作用的。

```jsx
/* style.css */
.title {
  color: green;
}

// App.jsx
import styles from "./style.css";

function App() {
  return <h1 style={ styles.title }>Hello world</h1>;
}
```

# 单向数据流

它的主要思想是： 
- 组件不会改变接收的数据。它们只会监听数据的变化，当数据发生变化时它们会使用接收到的新值，而不是去修改已有的值。当组件的更新机制触发后，它们只是使用新值进行重新渲染而已。
- 它消除了在多个地方同时管理状态的情况，它只会在一个地方 (通常就是 store) 进行状态管理。

```js
var Store = {
  _handlers: [],
  _flag: '',
  subscribe: function(handler) {
    this._handlers.push(handler);
  },
  set: function(value) {
    this._flag = value;
    this._handlers.forEach(handler => handler(value))
  },
  get: function() {
    return this._flag;
  }
};

class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = { value: Store.get() };
    Store.subscribe(value => this.setState({ value }));
  }
  render() {
    return (
      <div>
        <Switcher
          value={ this.state.value }
          onChange={ Store.set.bind(Store) } />
      </div>
    );
  }
};

function Switcher({ value, onChange }) {
  return (
    <button onClick={ e => onChange(!value) }>
      { value ? 'lights on' : 'lights off' }
    </button>
  );
};

<Switcher
  value={ Store.get() }
  onChange={ Store.set.bind(Store) } />

```

# Redux

由视图组件 (React) 来派发（`dispatch`）动作。动作（`action`）直接派发到 `store` 中（在 Redux 中只有一个 store）。决定数据如何改变的逻辑以纯函数 ( pure functions ) 的形式存在，我们称之为 reducers 。一旦 store 接收到动作，它会将当前状态和给定动作发送给 `reducer` 并要求其返回一个新的状态。然后，在数据不可变的方式下， reducer 需要返回新的状态。再然后， store 更新自身的内部状态。最后，与 store 连接的 React 组件会重新渲染。

