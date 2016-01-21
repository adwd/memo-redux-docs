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

Reduxは3つの基礎的な原則によって表現できる。

### Single source of truth

単一のstoreのオブジェクトツリーにアプリケーションのすべての状態を保存する。

これによりサーバーからの状態はそのためのコードを必要とせずクライアントにシリアライズされ、
伝達されるのであらゆるアプリケーションを作るのを簡単にする。
一つの状態のツリーはアプリケーションの分析とデバッグを容易にする。
さらにはアプリケーションの素早い開発サイクルを保つことも可能にする。
状態が一つのツリーに保存されていれば、伝統的に難しいとされてきた幾つかの機能、例えばUndo/Redoは
簡単に実装することができる。


```javascript
console.log(store.getState())

/* Prints
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    }, 
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
*/
```

### State is read-only

状態を変える唯一の方法はactionを発行することで、これは何が起こったかを記述しているオブジェクトである。

これはビューやネットワークのコールバックのどちらも状態を直接書き換えないことを保証する。
かわりに、状態変化の意図(intent)を述べるようになる。
全ての状態変化が集中管理され、厳格な順序でひとつずつ起こるため、繊細なレースコンディションを気にかける必要がない。
actionはただのプレーンなオブジェクトなので、ログ、シリアライズ、保存、リプレイをデバッグやテストなどで行える。

```javascript
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1
})

store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED'
})
```

### Changes are made with pure functions

アクションによってどのように胴体のツリーが変化するかを指定するには、純粋な関数であるreducerを使う。

reducerはひとつ前の状態とアクションを引数に取り、次の状態を返す単なる純粋関数である。
前の状態を変更するのではなく新しい状態オブジェクトを返すことに注意する。
はじめは一つのreducerでよいが、アプリケーションが大きくなるにつれ状態ツリーの部分に応じて小さなreducerに分割していく。
reducerはただの関数なので、呼ばれる順番の変更や引数の追加、ページネーションといった共有のタスクではreducerを再利用することができる。

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

以上でReduxの全てがわかったはずだ。

## Prior Art

Reduxは様々な技術の合成だ。いくつかのパターンや技術に似ているが、それらとは重要な点で異なる。
いくつかの類似点と相違点を以下で述べる。

### Flux

ReduxはFluxの実装と考えてよいか？

YesでありNoである。

（Fluxの開発者はReduxを認めている。）

ReduxはFluxの幾つかの重要な要素からインスパイアされた。
Fluxのように、Reduxはアプリケーションの特定のレイヤー（Fluxではstore、Reduxではreducer）で
モデルの更新ロジックに集中することを規定する。
アプリケーションコードに直接データを変更させる代わりに、状態の変化をactionと呼ばれるプレーンなオブジェクトで記述することを求める。

Fluxとは違い、ReduxはDispatcherの概念を持っていない。これはイベントエミッターの替わりに純粋な関数に依存するからで、
純粋関数は合成が用意でそれらを管理する余分なエンティティを必要としない。
Fluxをどのように見ているかに依存するが、これはFluxからの逸脱であるがもしくは実装の詳細のどちらかに見えるだろう。
Fluxはよく（(staet, action) => state）のように表現される。
この意味ではReduxは完全にFluxアーキテクチャであると言えるが、純粋関数のおかげでよりシンプルになっている。

Fluxとのもう一つの重要な違いは、Reduxはあなたがデータを変更しないことを仮定している。
プレーンなオブジェクトと配列を状態として使うが、reducerの中でそれを変更することは強く禁止する。
常に新しいオブジェクトを返すべきで、これはES7やBabelに実装されているobject spread syntaxや、Immutable.jsで容易になる。

パフォーマンス上のコーナーケースのためデータを変更する純粋でないreducerを書くことは可能ではあるが、これをすることを勧めない。
トラベル、記録・再生、ホットリローディングといった開発用の機能が動かなくなる。
それ以上にイミュータブルであることがパフォーマンスの問題になることはほとんどのアプリケーションでは無いことのように思われる。
これはOmが実証したように、オブジェクトの生成でのデメリットはreducerが純粋関数で何が変わったかが明白なためコストの高い再描画、再計算を行われないことによるメリットが上回る。

### Elm

### Immutable

### Baobab

### Rx

## Ecosystem

## Examples