# dva应用的前置科技树

## React 类组件、函数式组件

### 类组件、纯类组件

`React.Component`与`React.PureComponet`

```js
// Component
class Welcome extends React.Component {
    render() {
        return <h1>Hello, {this.props.name}</h1>;
    }
}
```

```js
// PureComponent
class Welcome extends React.PureComponent {
    render() {
        return <h1>Hello, {this.props.name}</h1>;
    }
}
```

React生命周期函数`shouldComponentUpdate`，这个函数返回一个布尔值，如果返回true，那么当props或state改变的时候进行更新；如果返回false，当props或state改变的时候不更新，默认返回true。这里的更新不更新，其实说的是执不执行render函数，如果不执行render函数，那该组件和其子组件都不会重新渲染。

组件继承PureComponent时，不可override `shouldComponentUpdate`函数，且`PureComponent`针对`shouldComponentUpdate`进行了优化，只对props和state进行浅比较（引用类型的地址比较）来确定触发，因此内部值发生改变时不会触发更新，除非进行增删或改地址操作

### 函数式组件 

函数即组件，返回为Dom结构

* 没有生命周期，但会被更新和挂载
* 没有this（组件实例）
* 没有内部状态（state）

```js
// functional component
function Welcome(props) {
    return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

函数式组件因为没有大量无用代码和显式this声明，也无生命周期和状态，因此它的性能比类组件好

## React Hooks

让函数式组件通过Hooks挂载状态机，简化了类组件中存在的生命周期函数操作和this指向问题，主要是解决之前业务发生更改时，函数式要重写成类、代码不能复用的问题

```js
    import React, {useState} from 'react';
    const ButtonWithHooks = props => {
      const [active, setActive] = useState(false); // ***
      return (
        <div className={active ? "active-button" : "default-button"} //*
          onClick={() => {
            setActive(!active); // ***
            props.onClick();
          }}
        >
          {props.text}
        </div>
      )
    };
```

我们只在React组件函数逻辑最顶层使用Hooks，不能在循环、条件、嵌套和一般js函数中调用Hook函数

React通过Hooks的调用顺序来确定调用中state对应的useState，React在执行时会维护一个组件内的Hooks栈，Hooks函数的执行返回的结果会一一顺次匹配hooks栈中的Hooks，若匹配失败，则整个组件匹配失败的hooks会失效，故只能逻辑最顶层使用Hooks。

### Hooks API

#### useState()

输入为初始state，Hooks函数内部维护一个state栈。Hooks函数的返回为[state，stateSetFuc]，Hooks的stateSet是替换式，类组件的setState是合并式。

#### useEffect()

该Hooks函数自动绑定`componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个生命周期。输入为渲染执行后需要进行的操作函数（副作用）与执行条件（所依赖的值数组），只有执行条件发生变动才会执行。

与生命周期函数不同，使用该Hooks函数式异步的，不会阻塞渲染的进行

当作为参数传入的函数，如果函数的返回为一个函数，则React在自动清除副作用时会调用该返回函数

该Hooks函数主要是解决类组件里业务逻辑的分割进不同生命周期内的问题

同时该函数解决了类组件中遗忘在`componentDidUpdate`进行的业务函数的逻辑更新问题

#### useContext()

#### 自定义Hook

自定义Hook命名开头统一使用`use`，参数可自定

## Redux、React-Router、Redux-saga

### Redux

#### reducer

形式为`(state,action) = >{newState}`的纯函数，单纯执行计算操作

```js
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1;
  case 'DECREMENT':
    return state - 1;
  default:
    return state;
  }
}
```

```js
function todos(state = [], action) {
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          })
        }
        return todo
      })
    default:
      return state
  }
}

function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter
    default:
      return state
  }
}

function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  }
}
```

#### Store

存储所有state的对象，通过createStore(reducer)创建

一个应用里只能有一个store，因此要组合reducer

返回API {subscribe, dispatch, getState}

action通过dispatch(action)触发reducer触发更新

通过getState()获取reducer所有State的现状

通过subscribe(listener)来注册监听dispatch触发时所要进行的操作

通过subscribe(listener)返回的函数来注销监听器

#### action

它是一个对象，必须有一个名为`type`的字符串属性来表示将要执行的动作

## dva及其API详解

### 核心概念

* State：一个对象，保存整个应用状态
* View：React 组件构成的视图层
* Action：一个对象，描述事件
* connect 方法：一个函数，绑定 State 到 View
* dispatch 方法：一个函数，触发 Action 变更 State

### Connect()

connect((states)=>{react states-props map})(link component)

connect函数编写柯里化，第一次传参映射函数返回一个可继续传参的函数式组件，第二次传参需要连接的组件，将需要连接的组件包含在函数式组件中，构成一个含有state的组件

#### 柯里化

```js
let add = function(x){
    return function(y){
        return x+y;
    }
}
add(3)(4); //7
```

#### @connect()

语法糖，对该文件内的所有组件进行链接

### reducer()

与redux中的reducer相同功能，不过api用法不同，action不再作为参数传入reducer中，reducer只接受纯state

### dispatch()

redux中Store.dispatch的简化，被connect的组件会自动在props中拥有该方法

与redux中的用法相同，在model外调用时type为‘model-namespace/reducer’，payload为调用时需要传递的参数

### Effect

为Action处理迭代器，处理异步action

```js
effects: {
    *query({ payload }, { select, call, put }) {
      yield put({ type: 'showLoading' });
      const { data } = yield call(query);
      if (data) {
        yield put({
          type: 'querySuccess',
          payload: {
            list: data.data,
            total: data.page.total,
            current: data.page.current
          }
        });
      }
    },
  },
```

其中 call 和 put 是 dva 提供的方便操作 effects 的函数，来自redux-saga，简单理解 call 是调用执行一个函数，而 put 则是相当于 dispatch 执行一个 action，而 select 则可以用来访问其它 model

#### call(fn, ...args)

操作同对象的call方法

#### put(action)

操作同dispatch

#### select(selector,...args)

创建Effect，用来让中间件在当前Store的state上调用指定的选择器，先返回selector(getState(), ...args)的结果（指全局state）在函数内部

selector：Function是一个(state,...args) =>{数据操作} 的函数，调用全局State返回指定的state的当前状态

args：Array，传递进selector函数中


