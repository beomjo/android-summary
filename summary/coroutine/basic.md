# Kotlin Coroutine Basic

## Coroutine이란?  
먼저 Coroutine의 단어의 뜻을 알아보면 
coroutine은 co 와 routine의 합성어이다.  
**co** 는 cooperation, "협력하다" 라는 뜻이고, **routine**는 함수 또는 하나의 테스크를 의미한다.  
**간단하게 해석하자면 협력하는 함수**라고 말할 수 있다.  

## Coroutine의 특징   
- 협력형 멀티테스킹
- 동시성 프로그래밍 지원


## 협력형 멀티테스킹  
#### Sub Routine과 Coroutine의 차이  
일반적인 함수를 **Sub routine**이라고도하며 Sub routine은 return을 만나면 탈출하여 Main Routine으로 돌아가게 된다.  
즉 일반적인 Sub routine은 return문을 만나거나 닫는괄호`}`를 만나기전까지는 탈출하지 않으므로 **진입점이 1개, 탈출점도 1개**이다.  
반면, **Coroutine**은 **진입점과 탈출점이 여러개**이다. Coroutine block을 생성하고 그안에서 suspend 키워드를 가진 A함수를 만나면
더 이상 아래 코드를 실행하지 않고 멈춘다(suspend) 그리고 Coroutine block을 탈출한다.  
탈출한뒤 다른 코드들이 실행되며 A함수는 어디선가 실행되고 있다.  
다른 코드들이 실행되다가도 A함수가 끝나게되면 다시 Coroutine block으로 진입하여 A함수 다음부분이 실행된다.  
이를 **협력형 멀티테스킹**이라 한다.  


## 동시성 프로그래밍  
함수를 중간에 빠져나왔다가, 다른 함수에 진입하고, 다시 원점으로 돌아와 멈추었던 부분부터 다시 시작하는 이 특성은
동시성 프로그래밍을 가능하게 한다.  
Coroutine을 생성하여서 동시성 프로그래밍을하는 특징때문에 가볍고 효율적인 성능으로 동작이 가능하다.  
만약 스레드로 Coroutine이 아닌 Thread두개로 동시성 프로그래밍을 한다면
CPU가 매번 Thread를 점우했다가 놓아주기를 반복하여 Context-Switching이 발생한다 (비용이 크다)

#### 동시성(Concurrency) vs 병렬(Parallelism)
|동시성|병렬|
|------|---|
|동시에 실행되는 것 같이 보이는 것|실제로 동시에 여러 작업이 처리되는 것|   
|![image](https://user-images.githubusercontent.com/39984656/104463005-954a7c80-55f4-11eb-8f54-f6779f6a3578.png)|![image](https://user-images.githubusercontent.com/39984656/104463017-9a0f3080-55f4-11eb-949f-54da05ebf518.png)|



#### Context-Switching
스레드 실행 혹은 종료시 스레드의 상태를 저장하고 복구하는 프로세스  
= 두개의 스레드에서 작업 시 CPU가 매번 스레드를 점유했다가 놓아주는 것을 반복할때 비용 발생


#### Thread vs Coroutine
코루틴이 하나의 실행-종료되어야 하는 일(Job)이라면 스레드는 그 일이 실행되는 곳이다.  
- 코루틴과 스레드는 모두 동시성을 보장하기 위한 기술이다.  
- 하나의 스레드에 여러개의 코루틴이 동시에(concurrency) 실행될 수 있다.
- 레드는 여러스레드가 있다면 병렬로(Parallelism) 실행된다.
- 코루틴의 경우 스레드풀을 사용하는 dispatcher로 코루틴들을 실행하면 스레드처럼 병렬프로그래밍을 할 수 있다.


## Coroutine의 장점
- 경량 : 코루틴을 실행중인 스레드를 차단하지 않는 정지를 지원하므로 단일스레드에서 많은 코루틴을 실행할 수 있다.  
- 메모리 누수 감소 : 구조화된 동시실행을 사용하여 범위 내에서 작업을 실행한다.  
- 처리 도중 취소 가능 : 실행중인 코루틴의 계층구조를 통해 자동으로 취소가 전달된다.  
  - 다른 처리를 중단(blocking)시키지 않고 중지(suspend)하는 형태로 가볍게 동작  