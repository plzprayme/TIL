ref: https://vuex.vuejs.org/guide/getters.html

# Getters
우리는 store의 데이터 기반의 computed 값이 필요할 때가 있다. 아래 코드 조각을 보자.
```js
computed: {
  doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```

일정한 조건에 맞는 데이터만 필터링하는 코드이다. 그런데 하나 이상의 component가 위의 값을 원할 수 있다. 그럼 우리는 코드를 복붙하거나 미리 선언해둔 helper 함수를 import해서 사용할 것이다.  
위의 안티패턴을 방지하기 위해서 vuex는 getters를 지원한다. 아래 코드 조각처럼 사용하면 된다.

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

## Property-Style Access
`getters`는 `store.getters` 의 형태로 expose 된다. 그래서 이렇게 사용할 수 있다. 

`store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]`

그리고 `getters`는  또 다른 `getters`를 두번 째 인자로 받을 수 있다.

```js
getters: {
  // ...
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length
  }
}

...
...

store.getters.doneTodosCount // -> 1
```

이제 어느 component에서나 손쉽게 사용이 가능하다.

```js
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```

## Method-Style Access
```js
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}

...

store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```

## mapGettes Style
```js
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
    // mix the getters into computed with object spread operator
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}

...

...mapGetters({
  // map `this.doneCount` to `this.$store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```

