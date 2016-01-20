# Redux

## The Gist

一つのstoreの中にオブジェクトツリーとしてアプリの状態が保存される。
状態を変える唯一の方法はactionを発行することで、これは何が起こったかを記述しているオブジェクト。
アクションを状態のツリーに変換するのにreducerを使う。

```
import { createStore } from 'redux'

/**
 * これはreducerで、(state, action) => stateのシグニチャを持つ純粋な関数。
 * アクションが状態を次の状態に変化させる処理を記述する。
 * 
 * 状態の型は任意で、プリミティブでも、配列でも、オブジェクトでも、Immutabke.jsのデータ構造でも良い。
 * 唯一重要なのは状態オブジェクトはミュータブルでないことで、新しいオブジェクトを次の状態として返す。
 * この例では、switchステートメントと文字列を使っているが、mapsといったヘルパー関数を使うこともできる。
 */
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// アプリケーションの状態を保持するRedux storeを作成する。
// storeのAPIは{ subscribe, dispatch, getState }。
let store = createStore(counter)

// 主導で更新に対してsubscribeすることができ、もしくはViewレイヤーに対しバインディングすることもできる。
store.subscribe(() =>
  console.log(store.getState())
)

// The only way to mutate the internal state is to dispatch an action.
// The actions can be serialized, logged or stored and later replayed.
// 状態を更新する唯一の方法はactionを発行すること。
// アクションはシリアライズすることができ、ログしたり保存して後でリプレイしたりできる。
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

Fluxと異なるのは、ReduxはDispatcherと複数のstoreを持たない。かわりに一つのstoreとreducerを持つ。
