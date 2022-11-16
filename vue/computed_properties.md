ref: https://vuejs.org/guide/essentials/computed.html

# Computed Properties
in-template expression은 아주 편리하지만 너무 간단한 작업만 가능한 operation이다.  
그리고 template에 너무 많은 로직을 넣는 것은 유지보수하기에 좋지 않을 것이다. 다음의 코드 조각을 보자.  
```js
export default {
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  }
}
```
여기에서 이런 로직을 추가한다고 가정하자. 

> 이미 책을 쓴 author 와 그렇지 않은 경우를 구분하여 다른 message를 출력

그리고 아래는 그 로직을 수행하는 코드 조각이다.
```js
<p>Has published books:</p>
<span>{{ author.books.length > 0 ? 'Yes' : 'No' }}</span>
```

template가 조금 어수선해졌다. 그리고 `author.books.length`에 의존하고 있는 것에 주목해야한다.  
게다가 다른 template에서 같은 메세지가 필요하다면 우리는 DRY 원칙을 어기게 된다.

이런 경우에 `computed` 를 유용하게 사용할 수 있다. 아래 코드 조각을 보자.

```js
export default {
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  },
  computed: {
    // a computed getter
    publishedBooksMessage() {
      // `this` points to the component instance
      return this.author.books.length > 0 ? 'Yes' : 'No'
    }
  }
}

...
...

<p>Has published books:</p>
<span>{{ publishedBooksMessage }}</span>
```

`publishedBooksMessage` 라는 이름의 `computed property` 를 선언했다.  
물론 런타임에 변경되는 `books.length` 값에 맞게 `publishedBooksMessage` 의 값도 변한다.  
template에서 사용할 때 함수를 호출하는 것이 아니라 `normal property` 처럼 호출 할 수 있다.  
