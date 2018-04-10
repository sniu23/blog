# React 用于构建用户界面的 JavaScript 库

- 声明式视图
- 组件化：拥有各自的状态
- 学习一次，到处使用

## 一、安装

```bash
npm install -g create-react-app
create-react-app my-app
# npx create-react-app my-app

cd my-app
npm start
```
## 二、JSX

- 扩展语法糖
- 推荐使用来描述 UI 信息（类似模板语法）
- 具有 JavaScrip 的全部能力

## 三、渲染

1. 条件渲染
  - `if、else`
  - `&&`
  - `?: 三元符`
2. 列表渲染
  - `map`
  - 需要指定一个在同辈元素中必须是唯一的`key`

## 四、表单

VUE的解释： modal, 双向数据绑定。

REACT的解释： 
  天生内部状态，通过`“受控”`来完成：交互处理和访问。

`在HTML当中，像<input>,<textarea>, 和 <select>这类表单元素会维持自身状态，并根据用户输入进行更新。但在React中，可变的状态通常保存在组件的状态属性中，并且只能用 setState() 方法进行更新。`

```jsx
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

## 五、事件处理

REACT 区别 DOM 元素：
1. React事件绑定`属性`的命名采用`驼峰式写法`，而不是小写。
2. 如果采用 JSX 的语法你需要传入一个`函数`作为事件处理函数，而不是一个字符串(DOM元素的写法)
3. 不能使用返回 false 的方式阻止默认行为。你必须明确的使用 `preventDefault`
4. 必须谨慎对待 JSX 回调函数中的 `this`，类的方法默认是不会绑定 this 的。*（通常情况下，如果你没有在方法后面添加 () ，例如 onClick={this.handleClick}，你应该为这个方法绑定 this。）*
    + 使用`属性初始化器` `babel-plugin-transform-class-properties` 来正确的绑定回调函数
    + 使用`箭头函数`传递参数： <button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
    + 使用`Function.prototype.bind`传递参数： <button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>


## 六、组件

**组件命名必须以大写开头。**

`封装`： 父组件或子组件都不能知道某个组件是有状态还是无状态，并且它们不应该关心某组件是被定义为一个函数还是一个类。

`Props`： 
1. 是`只读的`，不能改变。

`State`:
1. 使用`setState()`改变状态，而不是直接更新
2. this.setState((`prevState`, props) => {}) 使用以前的状态作为参数
3. 状态更新可能是`异步`的， 调用 setState() 独立地更新其中的变量，交给 React 去做状态更新合并

## 七、组件的封装、继承 和 组合

**重要**：`数据自顶向下单向流动`。
  - `Props`：`外部`传入的数据。
  - `State`：`内部`持有的`私有`数据。

**诀窍**：父子组件`继承`：
  - 一些组件在设计前无法获知自己要使用什么子组件?
    - 通过使用特殊的 `{props.children}` 预定义，透过 props 直接传递子组件/子元素来渲染。
  - 数据放在父组件还是子组件呢？
    - `状态提升 lifting-state-up`： 如果同一个数据的变化需要几个不同的组件来反映，建议提升共享的状态到它们最近的祖先组件中。
  - 父子组件是怎么交互的呢？
    - 借用 VUE 的解释： `Props down, Events up.` (属性向下传递，事件向上冒泡)

**我的理解：**
- UI处于静态： 数据自上向下传递（父/子组件除了可以各自持有私有状态外，父组件能够通过子组件属性将数据传递给子组件）
- 当用户在子组件UI层交互时：事件自下而上反馈，交由父组件处理数据变更。
  - 子组件事件处理逻辑：接收交互事件并将其暴露给属性; 
  - 父组件渲染函数中描述子组件时：已经指定了接收子组件事件属性的处理方法;
  - 父组件处理方法的逻辑：变更父组件状态数据。
  - 变化后的数据又自上向下传播，由此带来视图的变化;


**建议**：使用`组合`而不是`继承`以实现代码的重用。


## 八、组件构建步骤

1. 构思/获得： UI草样 和 API接口数据JSON格式;
2. 利用单一职责原则，拆解UI，获得有层次结构的各组件（命名和功能）定义;
3. 实现静态版本（能够使用数据渲染视图即可，暂不考虑影响视图变化的时间和状态）;
4. 为了让 UI 可以交互，需要触发变化的数据模型。此时，需要确定 UI state(状态) 的`最小（但完整）`表示;
  - 让我们仔细分析每一个数据，弄清楚哪一个是 state(状态) 。请简单地提出有关每个数据的 3 个问题：
    1. 是否通过 props(属性) 从父级传入？ 如果是这样，它可能不是 state(状态) 。
    2. 是否永远不会发生变化？ 如果是这样，它可能不是 state(状态)。
    3. 是否可以由组件中其他的 state(状态) 或 props(属性) 计算得出？如果是这样，则它不是 state(状态)。
5. 确定 state状态 的位置（参考`状态提升`）;
6. props属性 和 state状态 已经可以沿着层次结构自上向下传递了，最后一步： 添加反向数据流（自下而上的事件冒泡）。

## 九、组件生命周期

**Mounting(装载)**： 当组件实例被创建并将其插入 DOM 时。
- `constructor()`
- `componentWillMount()`
- `render()`
- `componentDidMount()`

**Updating(更新)**： 改变 props 或 state 可以触发更新事件。 在重新渲染组件时。
- `componentWillReceiveProps()`
- `shouldComponentUpdate()`
- `componentWillUpdate()`
- `render()`
- `componentDidUpdate()`

**Unmounting(卸载)**： 当一个组件从 DOM 中删除时。
- `componentWillUnmount()`