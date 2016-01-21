# Basics

## Actions

actionはアプリケーションからstoreに送信する情報のペイロードである。
storeへ送る唯一の情報源である。
`store.dispatch()`を使ってstoreにデータを送る。

```javascript
const ADD_TODO = 'ADD_TODO'
```

```javascript
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

actionはただのJavascriptオブジェクトである。actionは`type`プロパティを持たなければならず、
これは実行されたactionのタイプを示している。
タイプは一般的に文字列定数で定義される。
アプリケーションが大きくなると、こういった定数を別のモジュールに分離したくなるだろう。

```javascript
import { ADD_TODO, REMOVE_TODO } from '../actionTypes'
```

> ボイラープレートについて
> ファイルを分けてactionタイプの定数を定義する必要はなく、もしくは定義しなければならないわけでもない。
> 小さなプロジェクトにおいてはactionタイプにただの文字列リテラルを使うほうがわかりやすいだろう。
> しかし、大きなコードベースでは明示的に定数を宣言するメリットが有る。
> (Reducing Boilerplate)[http://rackt.org/redux/docs/recipes/ReducingBoilerplate.html]に
> コードベースを綺麗に保つ実用的なTIPSがある。

`type`の他には、actionオブジェクトの構造をどうするかは自由だ。
関心があるなら(Flux Standard Action)[https://github.com/acdlite/flux-standard-action]で
actionをどのように構築するか推奨されているかを見るといい。

actionタイプにユーザーがTODOを一つ完了したことを示すものを追加する。
TODOを配列で保存しているので、完了したTODOを指定するのに`index`プロパティを指定する。
実際のアプリケーションでは、何かを生成するたびにユニークなIDを発行するのが賢い。
ーションが大きくなると、こういった定数を別のモジュールに分離したくなるだろう。

```javascript
{
  type: COMPLETE_TODO,
  index: 5
}
```

可能な限り小さなデータを渡す方が良い。例えば、TODOアイテム全体を渡すよりも`index`だけを渡す方が良い。

最後に、TODOの表示を切り替えるactionタイプを追加する。

```javascript
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

### Action Creators

action creatorはそのままactionを作る関数だ。
actionとaction creatorを一緒にするのは簡単なので、チームにあった方を選ぼう。

(Flux)[http://facebook.github.io/flux/]の伝統的な方法では、
action creatorは実行された時dispatchを実行する。

```javascript
function addTodoWithDispatch(text) {
  const action = {
    type: ADD_TODO,
    text
  }
  dispatch(action)
}
```

それとは対照に、Reduxのaction creatorは単純にactionを返すだけだ。

```javascript
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

このため再利用とテストが容易になる。実際にはdispatchを実行するため、戻り値をdispatchに渡す。

```javascript
dispatch(addTodo(text))
dispatch(completeTodo(index))
```

あるいは、自動でdispatchを実行するbound action creatorを作ることもできる。

```javascript
const boundAddTodo = (text) => dispatch(addTodo(text))
const boundCompleteTodo = (index) => dispatch(completeTodo(index))
```

これで直接呼ぶだけでdispatchが実行される。

```javascript
boundAddTodo(text)
boundCompleteTodo(index)
```

`store.dispatch()`に直接アクセスするよりも、react-reduxの`connect()`ヘルパーのようなものを使うだろう。
`bindActionCreators()`もaction creatorを`dispath()`関数にバインドするのに使える。

### Source code

#### `action.js`

```javascript
/*
 * action types
 */

export const ADD_TODO = 'ADD_TODO'
export const COMPLETE_TODO = 'COMPLETE_TODO'
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER'

/*
 * other constants
 */

export const VisibilityFilters = {
  SHOW_ALL: 'SHOW_ALL',
  SHOW_COMPLETED: 'SHOW_COMPLETED',
  SHOW_ACTIVE: 'SHOW_ACTIVE'
}

/*
 * action creators
 */

export function addTodo(text) {
  return { type: ADD_TODO, text }
}

export function completeTodo(index) {
  return { type: COMPLETE_TODO, index }
}

export function setVisibilityFilter(filter) {
  return { type: SET_VISIBILITY_FILTER, filter }
}
```

### Next Steps

次はactionが実行された時にどのように状態が更新されるかを記述するreducerを定義する。

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

`createStore`の第2引数には任意で初期状態を指定できる。
これはサーバー上で動作するReduxアプリケーションの状態をクライアントでも一致させるときに有用だ。

```javascript
let store = createStore(todoApp, window.STATE_FROM_SERVER)
```

## Dispatching Actions

storeを作成したので、プログラムを動作させて確認する。UIなしでも、ロジックをテストすることができる。

```javascript
import { addTodo, completeTodo, setVisibilityFilter, VisibilityFilters } from './actions'

// Log the initial state
console.log(store.getState())

// Every time the state changes, log it
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
)

// Dispatch some actions
store.dispatch(addTodo('Learn about actions'))
store.dispatch(addTodo('Learn about reducers'))
store.dispatch(addTodo('Learn about store'))
store.dispatch(completeTodo(0))
store.dispatch(completeTodo(1))
store.dispatch(setVisibilityFilter(VisibilityFilters.SHOW_COMPLETED))

// Stop listening to state updates
unsubscribe()
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
