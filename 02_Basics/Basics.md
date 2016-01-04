# Basics

## Actions

actionはJavascriptオブジェクトで、typeプロパティを持つ。
typeは直接文字列でもよいが、const変数のほうが便利になる。

### Action Creators

actionを作る関数

```javascript
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}

dispatch(addTodo(text))
dispatch(completeTodo(index))
```

bound action creatorは自動で発行する

```javascript
const boundAddTodo = (text) => dispatch(addTodo(text))
const boundCompleteTodo = (index) => dispatch(completeTodo(index))

boundAddTodo(text)
boundCompleteTodo(index)
```

`store.dispatch()`に直接アクセスるよりも、react-reduxの`connect()`ヘルパーのようなものを使うだろう。
`bindActionCreators()`もaction creatorを`dispath()`関数にバインドするのに使える。

## Reducers

### Designing the State Shape

### Handling Actions

reducerでしてはいけないこと：

- 引数オブジェクトを操作し変更する
- APIコールやルーティングトランジションといった副作用を起こす
- 純粋でない関数を実行する。`Date.now()`や`Math.random()`など。

reducerは純粋関数である必要がある。

Reduxは`state`を`undefined`にして最初のreducerを実行する。ここで初期状態を定義する。

```javascript
import { VisibilityFilters } from './actions'

const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
}

function todoApp(state, action) {
  if (typeof state === 'undefined') {
    return initialState
  }

  // For now, don’t handle any actions
  // and just return the state given to us.
  return state
}
```


ES6のデフォルト引数シンタックスでよりコンパクトに書ける。

```javascript
function todoApp(state = initialState, action) {
  // For now, don’t handle any actions
  // and just return the state given to us.
  return state
}
```

`SET_VISIBILITY_FILTER`アクションを扱う。

```javascript
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```

1. `state` オブジェクトを操作してはいけない。必ず新しいオブジェクトを作って返す。
2. `default`の場合（不明なアクションの場合）は`state`をそのまま返す。

### Handling More Actions

TODOの残りのActionは以下のようになる。

```javascript
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      })
    case COMPLETE_TODO:
      return Object.assign({}, state, {
        todos: [
          ...state.todos.slice(0, action.index),
          Object.assign({}, state.todos[action.index], {
            completed: true
          }),
          ...state.todos.slice(action.index + 1)
        ]
    })
    default:
      return state
  }
}
```

`COMPLETE_TODO`のsliceする操作をよく書くようであれば、react-addons-update, updeep, Immutable.jsといったものを使うのも良い。

### Splitting Reducers

`todos`と`visibilityFilter`の状態は独立しているので、これらを分割してreducerを理解しやすくする。

```javascript
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
    case COMPLETE_TODO:
      return [
        ...state.slice(0, action.index),
        Object.assign({}, state[action.index], {
          completed: true
        }),
        ...state.slice(action.index + 1)
      ]
    default:
      return state
  }
}

function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    case ADD_TODO:
    case COMPLETE_TODO:
      return Object.assign({}, state, {
        todos: todos(state.todos, action)
      })
    default:
      return state
  }
}
```

`todos`の引数にも`state`があるが、これはただの配列になっている。状態の一部を分割したreducerに渡している。
これはreducer compositionと呼ばれ、Reduxアプリケーションを構築する基礎的なパターンである。

`visibilityFilter`のみを扱うreducerを作成する。

```javascript
function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
  case SET_VISIBILITY_FILTER:
    return action.filter
  default:
    return state
  }
}
```

これで、状態の部分を扱うreducerとそれを呼ぶメインのreducerに書き換え、一つのオブジェクトを返すように結合する。
これで初期状態の全体を知る必要はなく、子reducerがそれぞれの初期状態を`state`が`undefined`のときに返すようになっていれば良い。

```javascript
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
    case COMPLETE_TODO:
      return [
        ...state.slice(0, action.index),
        Object.assign({}, state[action.index], {
          completed: true
        }),
        ...state.slice(action.index + 1)
      ]
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

`combineReducers`を使うと以下のように書き換えられる。

```javascript
import { combineReducers } from 'redux'

const todoApp = combineReducers({
  visibilityFilter,
  todos
})

export default todoApp
```

## Store

”何が起きたか”を表現するactionsと、actionsに基づき状態を更新するreducerを定義した。
storeはそれらをくっつける。storeの責任範囲は：

- アプリケーションの状態を保持する
- `getState()`で状態へのアクセスを可能にする
- `dipatch(actions)`で状態の更新を可能にする
- `subscribe(listener)`でリスナーを登録する

Reduxでは状態は一つのオブジェクトなので、ロジックを分割したいときはstoreを複数持つのではなくreducer compositionを使う。

reducerからstoreを作成する。

```javascript
import { createStore } from 'redux'
import todoApp from './reducers'
let store = createStore(todoApp)
```

## Data Flow

Reduxは厳格な単方向データフローのアーキテクチャ

データライフサイクルは4つのステップからなる。

### 1. `store.dispatch()`を実行する

アプリケーションのどこからでも`store.dispatch(action)`を呼べる。
XHRコールバックや、あらかじめスケジューリングされた一定間隔でもよい。

### 2. `store`は`reducer`を実行する

現在の状態とアクションを引数にして、`store`は`reducer`を実行する。

```javascript
// The current application state (list of todos and chosen filter)
let previousState = {
 visibleTodoFilter: 'SHOW_ALL',
 todos: [ 
   {
     text: 'Read the docs.',
     complete: false
   }
 ]
}

// The action being performed (adding a todo)
let action = {
 type: 'ADD_TODO',
 text: 'Understand the flow.'
}

// Your reducer returns the next application state
let nextState = todoApp(previousState, action)
```
 
`reducer`は純粋な関数であり、副作用はない。APIコールといった副作用は、actionをdispatchするまえに行う。
 
### 3. ルートのreducerが複数のreducerからの出力を一つの状態ツリーに結合する。

```javascript
function todos(state = [], action) {
   // Somehow calculate it...
   return nextState
 }

 function visibleTodoFilter(state = 'SHOW_ALL', action) {
   // Somehow calculate it...
   return nextState
 }

 let todoApp = combineReducers({
   todos,
   visibleTodoFilter
 })
 ```

### 4. `store`はルート`reducer`が返した状態のツリー全体を保存する

`store.subscribe(listener)`で登録された全てのリスナーが呼び出される。リスナーは`store.getState()`を実行し現在の状態を得る。

## Usage with React

ReduxはReactにかぎらずAngular, Ember, jQueryなどでも使えるが、ReactやDekuといった状態からUIを返す関数をもつフレームワークとは特によく動く。

### Installing React Redux

`react-redux`はReactバインディングを提供する。

### Container and Presentational Components

ReduxのReactバインディングは、コンテナとプレゼンテーショナルコンポーネントの分離のアイディアを含む。
アプリケーションのトップレベルのコンポーネント（ルートハンドラーなど）だけがReduxに関連するほうがよく、
その下のコンポーネントは全てのデータをプロパティで受け取る。

|                 コンテナコンポーネント        | プレゼンテーショナルコンポーネント
-------------- | -------------------------- | --------------------------------
場所  　　　　　　| トップレベル、ルートハンドラー  | 中間や末端のコンポーネント
Reduxを使う 　　 | Yes                        | No
データを読み取る  | Reduxのstateをsubscribeする | プロパティからデータを読み取る
データを変更する  | Reduxのactionをdispatchする | プロパティからコールバックを実行する

このtodoアプリの例では一つのコンテナコンポーネントがビュー階層のトップにあるだけになる。
より複雑なアプリケーションでは、複数の階層とコンポーネントを持つことになる。
コンポーネントをネストする場合は、可能な限りプロパティを渡していくことを推奨する。

### Designing Component Hierarchy

状態を表すオブジェクトツリーをUI階層に合わせて設計する。[Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html)はこのプロセスを説明するとても良いチュートリアルだ。

TODOを新たにテキストで追加し、完了・未完了をマークでき、ページ下部に表示するTODOを全て・完了のみ・未完了のみにフィルタできるTODOアプリケーションを考える。

この概要から以下のコンポーネントとプロパティが思い浮かぶ：
- `AddTodo`はボタンのある`input`フィールド
  - `onAddClick(text: string)`はボタンが押された時のコールバック
- `TodoList`は表示されているTODOのリスト
  - `todos: Array`はTODOアイテムの配列で、`{ text, completed }`からなる
  - `onTodoClick(index: number)`はTODOがクリックされた時のコールバック
- `Todo`は一つのTODOアイテム
  - `text: string` is the text to show.
  - `completed: boolean` is whether todo should appear crossed out.
  - `onClick()` is a callback to invoke when a todo is clicked.
- `Footer` is a component where we let user change visible todo filter.
  - `filter: string` is the current filter: 'SHOW_ALL', 'SHOW_COMPLETED' or 'SHOW_ACTIVE'.
  - `onFilterChange(nextFilter: string)`: Callback to invoke when user chooses a different filter.

これらはすべてプレゼンテーショナルコンポーネントである。データがどこから来るかや変更されるかは関知しない。
与えられたデータを描画するのみである。

他の何かへReduxから移行する場合、これらのコンポーネントをそのまま使うことができる。これらはReduxに一切依存しない。

これらを書いてみる。Reduxへのバインディングはまだ考える必要はない。実際に描画するまではフェイクのデータを渡せば良い。

### Presentational Components

これらはただのReactコンポーネントなので、詳細は省く：

```javascript
// components/AddTodo.js
import React, { Component, PropTypes } from 'react'

export default class AddTodo extends Component {
  render() {
    return (
      <div>
        <input type='text' ref='input' />
        <button onClick={e => this.handleClick(e)}>
          Add
        </button>
      </div>
    )
  }

  handleClick(e) {
    const node = this.refs.input
    const text = node.value.trim()
    this.props.onAddClick(text)
    node.value = ''
  }
}

AddTodo.propTypes = {
  onAddClick: PropTypes.func.isRequired
}
```

```javascript
// components/Todo.js
import React, { Component, PropTypes } from 'react'

export default class Todo extends Component {
  render() {
    return (
      <li
        onClick={this.props.onClick}
        style={{
          textDecoration: this.props.completed ? 'line-through' : 'none',
          cursor: this.props.completed ? 'default' : 'pointer'
        }}>
        {this.props.text}
      </li>
    )
  }
}

Todo.propTypes = {
  onClick: PropTypes.func.isRequired,
  text: PropTypes.string.isRequired,
  completed: PropTypes.bool.isRequired
}
```

```javascript
// components/TodoList.js
import React, { Component, PropTypes } from 'react'
import Todo from './Todo'

export default class TodoList extends Component {
  render() {
    return (
      <ul>
        {this.props.todos.map((todo, index) =>
          <Todo {...todo}
                key={index}
                onClick={() => this.props.onTodoClick(index)} />
        )}
      </ul>
    )
  }
}

TodoList.propTypes = {
  onTodoClick: PropTypes.func.isRequired,
  todos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  }).isRequired).isRequired
}
```

```javascript
// components/Footer.js
import React, { Component, PropTypes } from 'react'

export default class Footer extends Component {
  renderFilter(filter, name) {
    if (filter === this.props.filter) {
      return name
    }

    return (
      <a href='#' onClick={e => {
        e.preventDefault()
        this.props.onFilterChange(filter)
      }}>
        {name}
      </a>
    )
  }

  render() {
    return (
      <p>
        Show:
        {' '}
        {this.renderFilter('SHOW_ALL', 'All')}
        {', '}
        {this.renderFilter('SHOW_COMPLETED', 'Completed')}
        {', '}
        {this.renderFilter('SHOW_ACTIVE', 'Active')}
        .
      </p>
    )
  }
}

Footer.propTypes = {
  onFilterChange: PropTypes.func.isRequired,
  filter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
}
```

```javascript
// containers/App.js
import React, { Component } from 'react'
import AddTodo from '../components/AddTodo'
import TodoList from '../components/TodoList'
import Footer from '../components/Footer'

export default class App extends Component {
  render() {
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            console.log('add todo', text)
          } />
        <TodoList
          todos={
            [
              {
                text: 'Use Redux',
                completed: true
              },
              {
                text: 'Learn to connect it to React',
                completed: false
              }
            ]
          }
          onTodoClick={index =>
            console.log('todo clicked', index)
          } />
        <Footer
          filter='SHOW_ALL'
          onFilterChange={filter =>
            console.log('filter change', filter)
          } />
      </div>
    )
  }
}
```

### Connecting to Redux

`App`コンポーネントをReduxに接続し、actionをdispatchし状態をRedux storeから読み出すには、`App`コンポーネントに2つの変更を加える。

まず、`react-redux`から`Provider`をインポートする。そしてルートコンポーネントを`<Provider>`でラップする。

```javascript
// index.js
import React from 'react'
import { render } from 'react-dom'
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import App from './containers/App'
import todoApp from './reducers'

let store = createStore(todoApp)

let rootElement = document.getElementById('root')
render(
  <Provider store={store}>
    <App />
  </Provider>,
  rootElement
)
```

これでコンポーネントにstoreインスタンスの使用を可能にする。（内部的には、Reactのcontext機能で行われている。）
それから、Reduxに繋げたいコンポーネントを`connect()`関数でラップする。
トップレベルのコンポーネントかもしくはルートハンドラにのみ行うようにする。
どのコンポーネントもReduxのstoreと`connect()`することはできるが、これはデータフローを追うことが困難になるのであまり深いところでやらないほうが良い。

`connect()`でラップされた任意のコンポーネントは、`dispatch`関数とグローバルな状態から必要な任意の状態をプロパティとして受け取る。
殆どの場合では`connect()`に一つの引数を渡すだけで良く、これは`selector`と呼ばれる関数である。
この関数はReduxのstoreのグローバルな状態を受け、コンポーネントに必要なプロパティを返す。
最も単純なケースでは、与えられた`state`をそのまま返す（純粋な`identity`関数）が、まずは変換する場合を見る。

合成可能な`selector`で高性能でメモ化された変換処理を作るには、[reselect](https://github.com/rackt/reselect)を見ておく。
この例ではそれを使わないが、大きなアプリケーションでは非常によく動く。

```javascript
// containers/App.js
import React, { Component, PropTypes } from 'react'
import { connect } from 'react-redux'
import { addTodo, completeTodo, setVisibilityFilter, VisibilityFilters } from '../actions'
import AddTodo from '../components/AddTodo'
import TodoList from '../components/TodoList'
import Footer from '../components/Footer'

class App extends Component {
  render() {
    // Injected by connect() call:
    const { dispatch, visibleTodos, visibilityFilter } = this.props
    return (
      <div>
        <AddTodo
          onAddClick={text =>
            dispatch(addTodo(text))
          } />
        <TodoList
          todos={visibleTodos}
          onTodoClick={index =>
            dispatch(completeTodo(index))
          } />
        <Footer
          filter={visibilityFilter}
          onFilterChange={nextFilter =>
            dispatch(setVisibilityFilter(nextFilter))
          } />
      </div>
    )
  }
}

App.propTypes = {
  visibleTodos: PropTypes.arrayOf(PropTypes.shape({
    text: PropTypes.string.isRequired,
    completed: PropTypes.bool.isRequired
  }).isRequired).isRequired,
  visibilityFilter: PropTypes.oneOf([
    'SHOW_ALL',
    'SHOW_COMPLETED',
    'SHOW_ACTIVE'
  ]).isRequired
}

function selectTodos(todos, filter) {
  switch (filter) {
    case VisibilityFilters.SHOW_ALL:
      return todos
    case VisibilityFilters.SHOW_COMPLETED:
      return todos.filter(todo => todo.completed)
    case VisibilityFilters.SHOW_ACTIVE:
      return todos.filter(todo => !todo.completed)
  }
}

// Which props do we want to inject, given the global state?
// Note: use https://github.com/faassen/reselect for better performance.
function select(state) {
  return {
    visibleTodos: selectTodos(state.todos, state.visibilityFilter),
    visibilityFilter: state.visibilityFilter
  }
}

// Wrap the component to inject dispatch and state into it
export default connect(select)(App)
```
