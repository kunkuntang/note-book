本文主要讲述的是自己上手Jotai过程中遇到的问题和记录下需要留意的点，很多内容都可以从[官网](https://jotai.org/)中自行查阅到，不会涉及Jotai的一些高阶用法。由于官网是全英编写，暂无中文，所以本文对于英文能力较差但是想尝试Jotai的人来说，也会有所帮助。

## 安装
`npm install jotai` or `yarn add jotai`

没什么好注意的，用自己喜欢的包管理工具安装到项目中即可

## 理念（Concepts）
一个工具被设计出来，肯定是需要解决现阶段出现的某些问题。简单翻译一下Jotai在官网中理念里的描述：

> Jotai是为了解决在使用React的过程中遇到的额外重复渲染问题。额外重复渲染问题指的是渲染进程多次渲染出同样界面。通常解决这个问题会用到 `React` 的 `Context`（useContext + useState）， 但是这样会导致应用存在很多 `context` 和引起另外的问题，比如大量的 Provider 嵌套。

此外在理念章节中我们需要认识的地方是，Jotai和传统的**自上而下**的状态管理工具（如：[use-context-selector](https://github.com/dai-shi/use-context-selector)）不一样， 借鉴来自[Recoil](https://recoiljs.org/)的灵感，它使用原子类模型（Atomic Model）实现了自下而上的状态管理。Jotai有两个原则：

- Primitive：它的接口很像`useState`
- Flexible：Atoms可以被拆解也可以被组合，也可以使用`useReducer`的方式处理副作用

## 基本概念（Primitives）
Jotai里的State其实就是atoms的组合，其实一个atom是state的一个部分。**这里需要注意的是，Jotai的 atoms 是不绑定到指定的react组件上的，这跟 react的useState不太一样。**

#### 创建atom
简单地使用 jotai 里的 atom 方法便可以创建一个 atom，在创建的时候你需要传入一个初始值。
```javascript
import { atom } from 'jotai'

const priceAtom = atom(10)

const messageAtom = atom('hello')

const productAtom = atom({ id: 12, name: 'good stuff' })
```

atom可以派生多种不同类型的atom：

- 只读atom
- 只写atom
- 读写atom

要创建这些atom，只需要传入对应的 读方法（read function） 和 写方法（write function） 即可

```javascript
const readOnlyAtom = atom((get) => get(priceAtom) * 2)

const writeOnlyAtom = atom(

 null, // it's a convention to pass `null` for the first argument

 (get, set, update) => {

 // `update` is any single value we receive for updating this atom

 set(priceAtom, get(priceAtom) - update.discount)

 }

)

const readWriteAtom = atom(

 (get) => get(priceAtom) * 2,

 (get, set, newPrice) => {

 set(priceAtom, newPrice / 2)

 // you can set as many atoms as you want at the same time
 // 你可以一次性地修改多个atoms

 }
)
```

- get 它是用来读取atom里的值的，在只读atom中，get返回的值是可响应（reactive）的，但在写atom中即不是，get只能读取同步的值，对于异步的情况应该使用 Async Atom。
- set 是用来更新 atom 的，它可以一次性地修改多个atoms。

atom可以在应用的任何地方被创建，也支持动态创建，但是请注意保留 atom 的引用。如果要在render函数中创建atom，应该使用 useMemo 或者 useRef 来保存 atom 的引用。

```javascript
const Component = ({ value }) => {

 const valueAtom = useMemo(() => atom({ value }), [value])

 // ...

}
```

#### 使用atom

useAtom 是Jotai 中读取 atom 值的一个 hook。

> Jotai 的 state 是由 atom config 和 atom value 组成的 WeakMap。

像React 的 useState 一样，useAtom 返回一个包含 atom 值和 修改 atom 值的元组（tuple）。

```javascript
const [value, updateValue] = useAtom(anAtom)
```

updateValue 只接收一个参数，这个参数会被传递到 write 方法的第三个参数，至于这个参数应该传什么结构完全处决于怎么写这个atom的write方法。

> Jotai的 state 在初始的时候是没有值的，当遇到有 useAtom 的时候，它就会把传入 useAtom 的atom 的值存到 state 中。如果 atom 是派生 atom ，会执行读方法（read function）并把计算出的值存到 state 中。当一个 atom 不再使用时，也就是所有使用它的组件都被卸载了，存在 state 中的值会被标记为垃圾等待回收。

##### onMount
在创建 atom 的时候可以配置一个 onMount 的函数，onMount 函数会在 atom 首次被 provider 使用的时候调用，在 onMount 函数可以使用 setAtom 来改变 atom 的值，并且 onMount 函数执行完后会返回一个 onUnmount 函数，onUnmount 函数会在该 atom 不再使用的时候调用。

```javascript
const anAtom = atom(1)

anAtom.onMount = (setAtom) => {

 console.log('atom is mounted in provider')

 setAtom(c => c + 1) // increment count on mount

 return () => { ... } // return optional onUnmount function

}
```

setAtom 会调用 atom 的 write 方法。同样的，调用 setAtom 里传递的参数取决于该 atom 定义的 write 方法：

```javascript
const countAtom = atom(1)

const derivedAtom = atom(

 (get) => get(countAtom),

 (get, set, action) => {

 if (action.type === 'init') {

 set(countAtom, 10)

 } else if (action.type === 'inc') {

 set(countAtom, (c) => c + 1)

 }

 }

)

derivedAtom.onMount = (setAtom) => {

 setAtom({ type: 'init' })

}
```
### Provider

Provider 是为一个子树组件（component sub tree，即这个组件也包含很多子孙，但却不是根组件）提供 state 的，多个 Provider 为多个子树组件提供 state，哪怕它们是嵌套关系，就像 React Context 一样。

```javascript
const SubTree = () => (

 <Provider>

	 <Child />

 </Provider>

)
```

**当在使用 atom 的时候没有设置 Provider ，那么会使用 Jotai 的默认 Provider。**

Providers 的用处有：

- 为每个子树提供不同的 state。
- Provider 可以提供一些 debug 信息
- Provider 可以为 atom 提供初始值

## 注意⚠️
Jotai state is within React component tree. Zustand state is in the store outside React.
Jotai 的状态（state）是存储在 React 组件树中的，这意味着只能在 React 中修改 state 中的值。如果有需要在 React 外修改 state 的情况，使用 Zustand 或者 valtio 会更适合。当然也可以通过 Jotai 的工具包来适配一起使用。

