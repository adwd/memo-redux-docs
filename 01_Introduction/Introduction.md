# Introduction

## Motivation

複雑化するSPAの開発において、状態と非同期性を分離することでこれらを扱う。
Reduxは状態の変化を予測可能にする。

JavascriptのSPAに対する要求はますます複雑になり、以前より一層多くのアプリケーション上の状態を管理しなければならなくなった。
この状態はサーバーのレスポンスやキャッシュ、ローカルで作成されたサーバーに送信していない編集中のデータ等が含まれる。
UIの状態も同様に、表示しているルーティング、選択されているタブ、スピナーを表示するかどうか、ページネーションを表示するかなど複雑になっている。

このような変化し続ける状態を管理するのは難しい。あるモデルが別のモデルを更新する場合、
あるビューがモデルを更新するとそれがまた別のモデルを更新し、これが順々に続くと別のビューが更新されるかもしれない。
ある時点で、アプリケーションの状態がいつ、なぜ、どのように変わるかを理解できなくなる。
システムが不明瞭で非決定的になると、バグを再現することや新しい機能を追加することが難しくなる。

この上、フロントエンド開発では新しい要求が一般的になっていることを考えてみる。
開発者として、楽観的な更新やサーバーサイドレンダリング、ルーティングの遷移の前にデータを取得することなどが期待されている。
以前に経験したことのないような複雑さを管理しなければならなくなったことに気づき、
必然的にこのように問いたくなる：ギブアップする時が来たのだろうか？答えはNoだ。

この複雑さは、状態の変化と非同期性という人が扱うには難しい2つの概念を混同して扱っていることからくる。
メントスコーラみたいなものだ。これらは分離されていれば素晴らしいが、一緒になると混乱を招く。
Reactのようなライブラリはビューのレイヤーにおいて非同期性と直接のDOM操作を取り除くことでこの問題を解決しようとしている。
しかし、状態の管理は未だに残されたままだ。Reduxはそこに焦点を置いている。

Flux、CQRS、イベントソーシングという段階を経て、Reduxは更新がいつ、どのように起こるかについて
明白な制約を課すことで状態の変化を予測可能にしようとしている。
その制約はReduxの3つの原則に反映されている。

## Three Principles

### Single source of truth

単一のstoreのオブジェクトツリーにアプリケーションのすべての状態を保存する。

### State is read-only

状態を変える唯一の方法はactionを発行することで、これは何が起こったかを記述しているオブジェクト。

### Changes are made with pure functions

```javascript
function visibilityFilter(state = 'SHOW_ALL', action) {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
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

import { combineReducers, createStore } from 'redux'
let reducer = combineReducers({ visibilityFilter, todos })
let store = createStore(reducer)
```

## Prior Art