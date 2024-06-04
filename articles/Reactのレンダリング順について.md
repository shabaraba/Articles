# Reactのレンダリング順について

Created: 2022年5月26日 10:00
Updated: 2023年2月13日 9:19
Published_Time: 2022年5月26日
Slug: react-rendering-sequence
Tags: React
Published: Yes
URL: https://blog.shaba.dev/posts/react-rendering-sequence

## useEffect系はコンポネント順ではない

考えてみると当たり前だが、順番的には子コンポーネントのレンダリング準備が整ってから親コンポーネントのレンダリングに取り掛かる

なので、親のuseEffectやuseLayoutEffectは子コンポーネントのそれらが終えてから実行される

[https://codepen.io/shabaraba/pen/jOZPPzz](https://codepen.io/shabaraba/pen/jOZPPzz)

```jsx
import React, {useState, useLayoutEffect, createContext, useContext} from "https://cdn.skypack.dev/react@17.0.1";
import * as ReactDOM from "https://cdn.skypack.dev/react-dom@17.0.1";

const StateContext = createContext({})
const App = () => {
  const [state, setState] = useState([])
  useLayoutEffect(() => {
    console.log("parent layout effect")
    setState((currentState) => [...currentState, "parent"])
  }, [])
  
  const stateContextProps = {
    state: state,
    setState: setState
  }
  console.log("parent")
  console.log(state)
  return (
    <StateContext.Provider value={stateContextProps}>
      <ChildComponent name="child1"/>
      <ChildComponent name="child2"/>
      <ChildComponent name="child3"/>
    </StateContext.Provider>
  )
}

const ChildComponent = ({name}) => {
  const stateContext = useContext(StateContext)
  useLayoutEffect(() => {
    console.log(name + " layout effect")
    stateContext.setState((currentState) => [...currentState, name])
  }, [])

  console.log(name)

  return (
    <div> {name} </div>
  )
}

ReactDOM.render(<App />, document.querySelector('#app'))
```

↓ 実行ログ

```
"parent"

// [object Array] (0)
[]

"child1"

"child2"

"child3"

"child1 layout effect"

"child2 layout effect"

"child3 layout effect"

"parent layout effect"

"parent"

// [object Array] (4)
["child1","child2","child3","parent"]

"child1"

"child2"

"child3"
```

実行順は親→子コンポーネント（親のreturn内で初めて子コンポーネントが呼ばれるので）
にもかかわらず、`useLayoutEffect`の順は子→親。

## 親は子の最後にuseEffectされる点に注意

useContextでsetStateを渡して子コンポーネント内で親のstateを操作するような場合、親の操作するタイミングには注意する必要があるようです。

例えば親→子の順にstateに値を追加していきたい場合は、親が最後に呼ばれるので、下記のように親を先頭に追加するよう記載する必要があります。

```jsx
  useLayoutEffect(() => {
    console.log("parent layout effect")
    setState((currentState) => ["parent", ...currentState])
  }, [])
```

↓ 実行ログ

```
// [object Array] (4)
["parent","child1","child2","child3"]
```