# Race Condition

> 은빈

### 경쟁 조건

```
            cnt = 1        cnt = -1
             cnt++          cnt--
            ------>        <------
Execution A          Data          Execution B
            <------        ------>
			cnt = 0        cnt = 0
```

하나의 데이터에 여러 주체가 동시에 접근하는 것을 Race Condition이라고 한다.

위의 코드와 같이 cnt = 0인 데이터를 Execution A와 B가 각각 가져간 뒤, A는 cnt++을 실행하고, B는 cnt--를 실행해 각각 cnt = 1과 cnt = -1을 만든 상황이 있다고 가정하자. 이 때, 하나의 데이터를 가져와 각자 연산한 뒤 데이터에 각자의 결과를 적용하면, 맨 마지막에 연산된 결과만 반영된다. 이 경우엔 0 + 1 - 1 = 0의 값이 아닌, 0 - 1 = -1의 값으로 데이터가 수정될 수 있다.

공유 메모리를 사용하는 프로세스들, 커널 내부 데이터에 접근하는 루틴들 간에 이러한 간섭 현상이 생길 수 있다.

### 발생 상황과 해결 방법

1. 커널 수행 중 인터럽트 발생
	- 커널 수행 중엔 인터럽트 처리를 하지 않게 만든다.
	- 커널 수행이 모두 끝나면 인터럽트 처리를 하게끔 한다.
2. 프로세스가 시스템 콜을 해 커널 수행중일 때 문맥 교환이 일어남
	- 커널 수행 중엔 CPU를 선점하지 않게 한다.
	- 커널 모드에서 사용자 모드로 돌아갈 때 선점하게끔 한다.
3. 멀티프로세서에서 공유 메모리 내 커널 데이터에 접근
	- 하나의 프로세서(CPU)가 커널 데이터에 접근할 때 데이터에 대해 lock을 걸어야 한다.

즉, 중요한 데이터 코드에 복수의 프로세스, 인터럽트, CPU 등의 간섭이 발생할 시 하나의 수행만 할 수 있도록 해야 한다는 것. 그렇다면 중요한 데이터 코드가 어디인지 알아야 그 부분만 경쟁 조건에 대해 해결해놓으면 될 것이다. 그 중요한 코드를 Critical Section(임계 영역)이라고 한다.

이 임계 영역에 프로세스 하나가 들어가 있다면, 다른 프로세스는 접근할수 없게(상호 배제, Mutual Exclusion)하고, 어떤 프로세스도 임계 영역을 수행중이지 않을 때, 어떤 프로세스가 임계 영역에 들어가고자 한다면 임계 영역을 수행하게(Progress) 해 주고, 각 프로세스들이 임계 영역에 들어갈 수 있는 횟수에 제한을 두어 기아 현상을 막아야(Bounded Waiting)한다.

<br><br><br>

> 선경

### 경쟁 조건
**명령어의 실행 순서에 따라 결과가 달라지는 상황**을 의미한다. 멀티 쓰레드가 같은 코드를 실행할 때 경쟁 조건이 발생하는데 이러한 코드 부분을 **임계 영역(critical section)** 이라고 부른다. 공유 자원을 접근하고 여러 쓰레드에서 동시에 실행되면 안되는 코드가 여기에 해당한다. 결국 경쟁 조건은 여러 쓰레드가 거의 동시에 임계 영역을 실행하려고 할 때 발생한다. 

어떤 경우에 여러 쓰레드가 동시에 임계 영역에 접근하게 될까? 멀티 프로세서 환경 혹은 단일 프로세서 환경에서도 인터럽트가 적절하지 않은 시점에 호출되는 경우(e.g. 프로세스가 A가 임계 영역에 진입한 상황에서 같은 임계 영역에 진입하는 프로세스 B로 전환한 경우)가 여기에 해당된다. 

경쟁 조건에 처한 경우 실행할 때마다 다른 결과가 나오는데 이처럼 결과가 어떨 지 알지 못하는 경우를 **비결정적인 결과**라고 부른다.

경쟁 조건이 나타나는 것을 방지하려면 하나의 쓰레드가 임계 영역 내의 코드를 실행하고 있을 때는 다른 쓰레드는 실행할 수 없도록 보장하는 **상호 배제(mutual exclusion)** 가 필요하다. 
