# 반복문안에 if문 제거하기

넥스트 스텝 강의 내용 구현, 자바 자료구조 구현을 하면서 depth를 1로 유지하려고 노력하고 있다.



그런데 항상 반복문안에 if문이 들어가는 패턴이 반복된다.

`stream`을 사용하지 않고 어떻게 객체지향적으로 해결할 수 있을까? 

아직 잘 모르겠다.



## 문제

예를들어,

```java
private boolean search(T isData) {
    Node<T> node = head;
    while (!node.nextIsNull()) {
        if (node.sameWith(isData)) {  // 결과에 따라 함수가 끝이 나야한다.
            return true;
        }
        node = node.next;
    }
    return false;
}
```



위의 코드에서 주석처럼 결과에 따라 함수가 끝이 나야하는 로직이 계속 등장한다.

이 때문에 객체에 메시지를 보낸다고 해도 원하는대로 동작하지 않는다.



예를들어,

```java
private boolean search(T isData) {
    Node<T> node = head;
    boolean flag = false;
    while (!node.nextIsNull()) {
        flag = node.sameWith(isData) // true -> true -> false?
        node = node.next;
    }
    return flag;
}
```

이런 경우에는 100번의 반복 중 99번을 `sameWith`가 `true`를 반환해도 마지막 한번이 `false`라면 결과 값은 `false`이다.

그런데 내가 원하는 동작은 `true`를 만나는 순간 함수가 끝나는 동작이다.

이것을 어떻게 해결해야 할지 잘 떠오르지 않는다. 고통스럽지만 넘어야한다. 

이번 주 안에 해답을 찾고 이어서 답변을 작성하고 싶다.



## 해결







