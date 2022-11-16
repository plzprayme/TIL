ref: [https://vuex.vuejs.org/guide/getters.html](https://v3.vuex.vuejs.org/). 
ver: vuex 3.6.2

# Vuex
Vuex는 vue를 위한 statement management pattern + library 이다. 이걸 사용하면 application의 모든 component들에게 중앙집중형 store를 제공할 수 있다.  
vuex가 제공하는 store를 규칙을 가지고 있어서 예측 가능한 범위 안에서만 변경이 발생하도록 강제한다.  
잘 사용하고 싶다면 [devtools](https://github.com/vuejs/devtools) 같은 vue echosystem을 살펴보자  


## State Management Pattern
간단한 vue app 코드 조각이다.
```js
new Vue({
  // state
  data () {
    return {
      count: 0
    }
  },
  // view
  template: `
    <div>{{ count }}</div>
  `,
  // actions
  methods: {
    increment () {
      this.count++
    }
  }
})
```

`state, vuew, actions` 으로 이루어진 self-contained app이다.  
위의 앱은 data flow가 one-way이다. `(state -> view -> actions -> state)` 이런 구조는 component가 많아지면서 state를 공유하게되면 곧바로 무너지게 된다. 구체적인 상황을 살펴보자  

1. 여러 view가 동일한 state를 참조한다.
2. 서로 다른 view의 actions가 동일한 state를 mutate 한다.

그리고 문제도 있다.

1.  깊숙한 nested component에 props를 넘겨주는 것이 귀찮다. 게다가 sibling component는 사용도 못한다.
2.  위 문제를 해결하기 위해 부모, 자식 instance에 직접 접근하거나 mutate하는 경우가 생긴다.
3.  이벤트를 통해 여러개의 state copy의 동기화를 시도한다.


그래서 shared state를 global singletone 으로 관리하고자 한다. 그러면 component 들이 하나의 view로 간주할 수 있고 다시 one-way data flow를 유지할 수 있다. 

## When we use
작은 규모의 SPA면 [simple statement management](https://vuejs.org/guide/scaling-up/state-management.html#what-is-state-management) 를 사용하자.  
middle ~ large SPA면 vuex를 사용해라

