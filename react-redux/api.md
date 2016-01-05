# API

## `<Provider store>`

`connect()`の呼び出しで、コンポーネント階層に対してRedux storeを使用可能にする。
普通は、`<Provider>`にルートのコンポーネントをラップせずに`connect()`を使うことができる。

もしも本当にそうする必要がある場合、`connect()`されたコンポーネントすべてに`store`をpropとして手動で渡すことができるが、
ユニットテストで`store`をスタブするためやコードベースがReact以外にも使う場合のために推奨しない。
普通は、単純に`<Provider>`を使う。

## Props

- `store` (Redux store): アプリケーション内の単一のRedux store
- `children` (ReactElement): コンポーネント階層のルート

## Example

### Vanilla React

```javascript
ReactDOM.render(
  <Provider store={store}>
    <MyRootComponent />
  </Provider>,
  rootEl
)
```

### React Router 0.13

```javascript
Router.run(routes, Router.HistoryLocation, (Handler, routerState) => { // note "routerState" here
  ReactDOM.render(
    <Provider store={store}>
      {/* note "routerState" here: important to pass it down */}
      <Handler routerState={routerState} />
    </Provider>,
    document.getElementById('root')
  )
})
```

### React Router 1.0

```javascript
ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>...</Router>
  </Provider>,
  targetEl
)
```

## `connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])`

ReactコンポーネントとRedux storeを接続する。

渡されたコンポーネントクラスを変更することはしない代わりに、新しい接続されたコンポーネントクラスを返す。

### Arguments

- [`mapStateToProps(state, [ownProps]): stateProps`] (Function): 指定された場合、コンポーネントはRedux storeの更新をサブスクライブする。更新があった時に`mapStateToProps`が呼び出される。返り値は単なるオブジェクトである必要があり、そのオブジェクトはコンポーネントのpropsにマージされる。省略した場合、コンポーネントはRedux storeの更新にサブスクライブしない。`ownProps`が2番めの引数として指定された場合、その値は渡したコンポーネントのpropsになり、`mapStateToProps`がコンポーネントが新しいpropsを受け取るたびに再度呼び出される。
- [`mapDispatchToProps(dispatch, [ownProps]): dispatchProps`] (Object or Function): オブジェクトが渡された場合、その内部にある関数がRedux action creatorと仮定される。関数と同じ名前でオブジェクトで、Redux storeにバインドされているものについては、コンポーネントのpropsにマージされる。関数が渡された場合、引数に`dispatch`が与えられる。オブジェクトを返すか独自のやりかたで`dispatch`をaction creatorにバインド（その場合Reduxの`bindActionCreators()`ヘルパーを使うだろう）するかは自由だ。省略した場合、デフォルトの実装がコンポーネントのpropsに`dispatch`を注入する。`ownProps`が2番めの引数として指定された場合、その値は渡したコンポーネントのpropsになり、`mapStateToProps`がコンポーネントが新しいpropsを受け取るたびに再度呼び出される。
- [`mergeProps(stateProps, dispatchProps, ownProps): props`] (Function): 指定された場合、`mapStateToProps()`と`mapDispatchToProps()`の結果と、親のpropsが渡される。この関数の返り値のオブジェクトはラップされたコンポーネントのpropsに渡される。propsに基づき状態を選択する場合や、特定のpropsにaction creatorsをバインドするときにこれを指定するだろう。省略した場合、`Object.assign({}, ownProps, stateProps, dispatchProps)`がデフォルトで使用される。
- [`options`] (Object) 指定された場合、コネクターの振る舞いをさらにカスタマイズする。
  - [`pure = true`] (Boolean): trueの場合、`shouldComponentUpdate`を実装し、`mergeProps`の結果を浅く比較し、不必要な更新を防ぎ、コンポーネントが純粋なコンポーネントであることといかなる入力・状態、propsや選択されたRedux storeの状態に依存しないことを仮定する。デフォルトではtrue。
  - [`withRef = false`] (Boolean): trueの場合、ラップされたコンポーネントのインスタンスへのrefを保存し、`getWrappedInstance()`メソッドで使用可能にする。デフォルトでは`false`

### returns

指定されたオプションに従ってコンポーネントに状態とaction creatorを注入するReactコンポーネント

#### Static Properties

- `WrappedComponent` (Component): `connect()`に渡されたオリジナルのコンポーネント

#### Static Methods

コンポーネントの全ての静的メソッドは巻き上げられる。

#### Instance Methods

##### `getWrappedInstance(): ReactComponent`

ラップされたコンポーネントインスタンスを返す。`connect()`の4番目の`options`引数として`{ withRef: true }`を渡した時のみ有効。

### Remarks

- 2度実行される必要がある。最初は上記の引数で、2度目はcomponentを引数に取る：`connect(mapStateToProps, mapDispatchToProps, mergeProps)(MyComponent)`
- 渡されたReactコンポーネントを変更しない。新しい接続されたコンポーネントを返す。
- `mapStateToProps`関数は一つの引数にRedux store全体の状態を取り、propsとして渡されるオブジェクトを返す。これはよくselectorと呼ばれる。[reselect](https://github.com/rackt/reselect)を効率的にselectorを合成するのとcompute derived dataのために使う。

### Examples

#### `dispatch`を注入しstoreをリッスンしない

```javascript
export default connect()(TodoApp)
```

#### `dispatch`とグローバルな状態の全てのフィールドを注入する

> これをやってはいけない。これは全てのアクションで`TodoApp`が再描画されるのでパフィーマンス最適化をダメにしてしまう。 
> 状態の関連する部分だけをリッスンするようにView階層のコンポーネントごとに粒度を細かくして`connect()`したほうが良い。

```javascript
export default connect(state => state)(TodoApp)
```

#### `todos`と全てのaction creator(`addTodo`, `completeTodo`, ...)を`actions`として注入する

```javascript
import * as actionCreators from './actionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return { actions: bindActionCreators(actionCreators, dispatch) }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)

// こっちのほうがわかりやすい？
export dafault connect(
  (state) => { todos: state.todos },
  (dispatch) => { actions: bindActionCreators(actionCreators, dispatch)}
)
```

#### `todos`と指定したaction creator(`addTodo`)を注入する

```javascript
import { addTodo } from './actionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators({ addTodo }, dispatch)
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

#### `todos`と、todoActionCreatorsを`todoActions`として、counterActionCreatorsを`counterActions`として注入する

```javascript
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return {
    todoActions: bindActionCreators(todoActionCreators, dispatch),
    counterActions: bindActionCreators(counterActionCreators, dispatch)
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

#### `todos`と、todoActionCreatorsとcounterActionCreatorsを一緒に`actions`として注入する

```javascript
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return {
    actions: bindActionCreators(Object.assign({}, todoActionCreators, counterActionCreators), dispatch)
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

#### `todos`と、todoActionCreatorsとcounterActionCreatorsをpropsに直接注入する

```javascript
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators(Object.assign({}, todoActionCreators, counterActionCreators), dispatch)
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

#### propsによってそのユーザー特定の`todos`を注入する

```javascript
import * as actionCreators from './actionCreators'

function mapStateToProps(state, ownProps) {
  return { todos: state.todos[ownProps.userId] }
}

export default connect(mapStateToProps)(TodoApp)
```

#### propsによってそのユーザー特定の`todos`と、actionに`props.userId`を注入する

```javascript
import * as actionCreators from './actionCreators'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mergeProps(stateProps, dispatchProps, ownProps) {
  return Object.assign({}, ownProps, {
    todos: stateProps.todos[ownProps.userId],
    addTodo: (text) => dispatchProps.addTodo(ownProps.userId, text)
  })
}

export default connect(mapStateToProps, actionCreators, mergeProps)(TodoApp)
```
