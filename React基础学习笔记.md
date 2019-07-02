# React 笔记

## JSX

大写开头的标签React自动识别为用户的自定义组件

小写开头的标签React自动识别为系统的默认组件

JSX允许在大括号中嵌入任何表达式

组件标签后面的属性都为传入该组件的Props

`<A />`实际上是`<A></A>`的简写，标签内的内容实际也是Props的一部分，通过`props.children`调用

## State 与 Props

State是组件私有参数，相当于Vue中的data，但是Vue是双向绑定，而React是单向绑定（module-》View），view改变时需要依靠Props传递进行手动setState()操作

数据的双向绑定是value的单向绑定（M-》V）与事件监听（V-》M）实现

React与Vue的value单向绑定实现不同，Vue基于数据劫持

Prop是父子组件传递参数

State可以依靠Props构建

State的更新可能是异步的，这是由于React性能优化策略，React会将多个setState()的参数对象合并成一个执行，这是因为React在其中添加了函数节流操作，一般情况下不会有问题出现，除非操作间隔小于节流时间间隔

但当setState的参数是函数时，将会立即执行

构造函数是第一次和唯一一次对state直接注册监听的地方（注册get方法），只有在该处可以进行state的直接赋值，其他地方只能进行setState操作进行数据监听的注册与更改并触发render回调

## 事件处理

React的事件命名严格采用小驼峰法

事件的默认行为阻止不能采用返回false，而应严格使用event.preventDefault（）实现

在class里对方法使用箭头函数声明是实验性功能`public class fileds`

在render（）中使用箭头函数在jsx中回调，相当于在render函数中声明封装这个箭头函数再调用

## 条件渲染

在jsx中可以逻辑运算符进行条件渲染
![逻辑算符](/img/logic.png)

### 列表&key

列表中的元素需要标识一个独有的key用来给React辨识

元素的 key 只有放在就近的数组上下文中才有意义，数组的单独元素标识key是错误的用法

key的独有仅限在兄弟节点间

## 表单

此处进行事件监听实现数据的双向绑定

## 状态提升

Vue的子传父的React实现，通过父组件统一管理变量参数，依靠子组件定义的属性来触发父组件的事件方法来修改父组件的参数

## 组合&继承

JSX可以作为props传递给其他组件

FaceBook的人说，React只要组合就够了，他们没试过继承（（（

## React哲学

为了正确地构建应用，你首先需要找出应用所需的 state 的最小表示，并根据需要计算出其他所有数据。其中的关键正是 DRY: Don’t Repeat Yourself。只保留应用所需的可变 state 的最小集合，其他数据均由它们计算产生。比如，你要编写一个任务清单应用，你只需要保存一个包含所有事项的数组，而无需额外保存一个单独的 state 变量（用于存储任务个数）。当你需要展示任务个数时，只需要利用该数组的 length 属性即可。

## 代码拆分

### 动态import

```js
//静态（提前声明）
import { add } from './math';

console.log(add(16, 26));

//动态（指逻辑中调用）
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

### React.lazy

该方法将动态import转化为静态操作

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <OtherComponent />
    </div>
  );
}
```

### Suspense

用于动态import加载完成前的后备内容

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

fallback prop（属性） 接受在等待加载组件时要渲染的任何 React 元素。 您可以将 Suspense 组件放在惰性组件上方的任何位置。 您甚至可以使用一个 Suspense 组件包装多个惰性组件

## 上下文（context）

指全局的Props（state），避免过多的props传递

它依靠React.createContext()生成，为一个函数，其Provider属性为自定义组件，该组件内的子组件可以直接获取该组件的props

### 没有context的解决办法

将使用传递参数的组件作为被传递的主体

```js
<Page user={user} avatarSize={avatarSize} />
// ... which renders ...
<PageLayout user={user} avatarSize={avatarSize} />
// ... which renders ...
<NavigationBar user={user} avatarSize={avatarSize} />
// ... which renders ...
<Link href={user.permalink}>
  <Avatar user={user} size={avatarSize} />
</Link>


function Page(props) {
  const user = props.user;
  const userLink = (
    <Link href={user.permalink}>
      <Avatar user={user} size={props.avatarSize} />
    </Link>
  );
  return <PageLayout userLink={userLink} />;
}

// Now, we have:
<Page user={user} />
// ... which renders ...
<PageLayout userLink={...} />
// ... which renders ...
<NavigationBar userLink={...} />
// ... which renders ...
{props.userLink}
```

```js
function Page(props) {
  const user = props.user;
  const content = <Feed user={user} />;
  const topBar = (
    <NavigationBar>
      <Link href={user.permalink}>
        <Avatar user={user} size={props.avatarSize} />
      </Link>
    </NavigationBar>
  );
  return (
    <PageLayout
      topBar={topBar}
      content={content}
    />
  );
}
```

### API详解

`const MyContext = React.createContext(defaultValue);`

Provider（父）=-》Consumer（子）

创建一个 Context 对象对。当 React 渲染订阅这 个Context 对象的组件时，它将从组件树中匹配最接近的 Provider 中读取当前的 context 值。

defaultValue 参数 仅 当 consumer(使用者) 在树中没有匹配的 Provider(提供者) 时使用它。这有助于在不封装它们的情况下对组件进行测试。注意：将 undefined 作为 Provider(提供者) 值传递不会导致 consumer(使用者) 组件使用 defaultValue 。

`Context.Provider`

Context对象附带的React组件

`Class.contextType`

在类组件中声明绑定的context对象，声明后即可直接跨组件调用该context对象的Value，如果该context的provider组件不是该类组件的上游，则调用的value为context对象创建时的初始值

`Context.Consumer`

```js
<MyContext.Consumer>
  {value => /* render something based on the context value */}
</MyContext.Consumer>
```

一个可以订阅 context 变化的 React 组件。 这允许您订阅 函数式组件 中的 context 。

需要接收一个 函数作为子节点。 该函数接收当前 context 值并返回一个 React 节点。 传递给函数的 value 参数将等于组件树中上层这个 context 最接近的 Provider 的 value 属性。 如果上层没有提供这个 context 的 Provider ，value参数将等于传递给 createContext() 的 defaultValue 。

