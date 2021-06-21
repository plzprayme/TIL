# Queue

`java.util` 패키지에 위치한 인터페이스이다.



인터페이스이기 때문에 `java.util.LinkedList`로 인스턴스화한다.

`Queue<Integer> q = new LinkedList<>();`



## Methods

### 삽입

#### `boolean add(E e)`

삽입에 성공하면 `true`를 반환한다.

삽입에 실패하면 아래의 예외를 발생시킨다.

1. `IllegalStateException`
2. `ClassCastException` 
3. `NullPointerException`
4. `IllegalArgumentException`

#### `boolean offer(E e)`

`add`와 같은 동작을 한다.

용량이 제한된 Queue를 사용할 때 `add` 대신 `offer`를 사용하는 것이 바람직하다.

삽입에 실패하면 아래의 예외를 발생시킨다.

1. `ClassCastException` 
2. `NullPointerException`
3. `IllegalArgumentException`



### 제거

#### `E remove()`

가장 먼저 삽입된 엘리먼트를 리턴하고 제거한다.

만약 Queue가 비어 있다면 `NoSuchElementException` 예외를 발생시킨다.



#### `E poll()`

`remove`와 같은 동작을 한다.

다른 점은 만약 Queue가 비어 있다면 `null`을 반환한다.



### 조회

#### `E element()`

가장 먼저 삽입된 엘리먼트를 리턴하고 제거하지 않는다.

만약 Queue가 비어 있다면 `NoSuchElementException` 예외를 발생시킨다.



#### `E peek()`

`element` 와 같은 동작을 한다.

다른 점은 만약 Queue가 비어 있다면 `null`을 반환한다.



