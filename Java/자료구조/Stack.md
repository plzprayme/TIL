# Stack

`java.util`에 위치한 클래스이다.

`Vector`를 상속한다.

## Methods

### 삽입

#### `E push(E item)`

아이템을 스택의 가장 상단에 추가한다.

내부적으로는 `Vector.addElement()`를 호출한다.

`Vector`에는 `E[]` 형태로 아이템을 관리하고 있다.



### 제거

#### `synchronized E pop()`

스택의 가장 상단에 있는 아이템을 제거하고 리턴한다.



### 조회

#### `synchronized E peek()`

스택의 가장 상단에 있는 아이템을 리턴한다. 

만약 스택이 비어 있다면 `EmptyStackException`을 발생시킨다.



#### `synchronized int search(Object o)`

파라미터로 받은 엘리먼트가 있는지 확인한다.

없다면 `-1` 을 리턴한다. 있다면 인덱스를 리턴하는데 top 부터 1이다.



### 유틸

#### `boolean empty()`

스택이 비어있는지 여부를 리턴한다.