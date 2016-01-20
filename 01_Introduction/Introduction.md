# Introduction

## Motivation

複雑化するSPAの開発において、状態と非同期性を分離することでこれらを扱う。
Reduxは状態の変化を予測可能にする。

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