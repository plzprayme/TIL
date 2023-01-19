# Golang Goroutine Scheduling
ref: https://ssup2.github.io/theory_analysis/Golang_Goroutine_Scheduling/  


## Goroutine Scheduling
Golang 은 OS Tread 보다 경량화된 Thread인 Goroutine 을 제공한다.  
Goroutine 은 Golang Runtime에 퐇삼된 Golang Scheduler 가 수행하는 Thread Scheduling을 통해서 실행된다.  
그러니까 다수의 Goroutine이 소수의 Thread 위에서 동작하게 된다.  
  
고루틴은 세 개의 큐가 있다
* Net Poller: Network IO
* GRQ (Global Run Queue)
* LRQ (Local Run Queue)

### Goroutine State
고루틴의 상태를 간단하게 세 가지로 나타낼 수 있다. (실제로는 더 많다)
* Waiting: 외부 Event를 대기하고 있는 상태. I/O, Lock 해제 등 Goroutine이 실행 가능하다는 것을 알려주는 OS Event를 말한다.
* Runnable: Goroutine 이 실행 가능한 상태
* Executing: Goroutine이 샐행되고 있는 상태

최대 고루틴 병렬 수행 처리 수 는 `GOMAXPROCS` 환경 변수 값으로 결정한다. default 값은 CPU Core 개수이다.

### Run Queue
Processor는 LRQ를 가지고 있다. LRQ에서 하나씩 고루틴을 꺼내서 수행한다.  
LRQ를 운영함으로써 GRQ에서 발생하는 Race Condition을 줄인다.  
Processor 라는 개념이 도입된 이유는 LRQ 숫자를 줄이기 위함이다. LRQ 개수가 많아지면 Overhead가 커진다.  

LRQ는 FIFO, LIFO 가 결합된 형태이다. LIFO 부분은 사이즈가 1이라 하나의 Goroutine만 저장된다.  
LRQ는 LIFO 부터 리턴하고 FIFO를 리턴한다.  

Goroutine의 Locality 를 부여하기 위한 설계 방법이다. Goroutine이 새로운 Goroutine을 생성하고 그 고루틴이 종료되길 기다린다면, 새로 생성된 고루틴이 빠르게 실행되고 종료되어야 성능상 이득이 있다.  
따라서 새로 생성된 Goroutine은 Goroutine을 생성한 Processor의 LRQ에 저장된다. LIFO 에서 수행되고 있다면 FIFO에 쌓이니까 동일한 Processor에서 빠르게 수행할 수 있다.   

Executing 고루틴은 한번에 최대 10ms 동작한다. 10ms 이후에는 Wating 상태가 되어 GRQ로 이동된다.LRQ가 가득차면 GRQ에 저장된다.  

### System Call
만약 Sync System Call이 발생하면 Thread가 Blocking 되고 Processor가 가진 LRQ의 모든 고루틴이 묶여있게 된다.  
이를 방지 하기 위해서 Sync System Call이 발생하면 Processor는 다른 스레드에 할당되고 LRQ에 남은 고루틴을 실행한다.  
Sync System Call이 완료되면 수행되던 Goroutine은 LRQ에 다시 들어가고 스레드는 소멸되지 않고 남아서 대기한다.  

Aysnc System Call 호출 시 Net Poller에 들어가서 완료 Event를 대기한다. Network Poller는 Background Thread에서 동작하며 Linux에서는 epoll() 함수가 Event를 수신한다.

### Work Stealing
LRQ가 비어있으면 다른 곳에서 고루틴을 가져온다.  
다른 LRQ의 절반을 가져오고, 만약 없다면 GRQ에서 가져오고 없다면 Net Poller에서 가져온다.  
단, 새로 생성된 고루틴은 3ms 동안 Stealing 되지 않는다.

### Fairness
공평하게 스케쥴링 되어야 한다.
* Thread: 고루틴이 이용중인 스레드를 반환하지 않고 계속이용할 수 있다. 스레드 독점을 막기 위해 10ms timeout이 있다.
* LRQ: 두개의 고루틴이 번갈아가면서 저장되고 실행된다면 LIFO 에 의해서 FIFO가 수행안될 수 있다. 따라서 하나의 고루틴이 LIFO를 10ms이상 점유할 수 없다.
* GRQ: LRQ가 항상 꽉 차 있으면 GRQ가 수행안될 수 있다. 61번 째 스케쥴링마다 GRQ 고루틴을 먼저 수행하도록 만들었다
* Network Poller: 독립적인 Background Thread이다. OS Scheduler에 따라 동작한다.
