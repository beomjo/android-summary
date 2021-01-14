# Context

## Dispatchers
- Dispatcher.Default : CoroutineContext가 지정되지 않았을때 지정되는 기본 Dispatcher
    - launch, async과 같은 표준 Builder에서 사용하는 기본 Dispatcher
    - JVM의 공유된 백그라운드 스레드의 commonPool에서 동작
- Dispatcher.IO : I/O 작업을 위한 Dispatcher
- Dispatcher.Main : 메인스레드(UI)에 국한된 Dispatcher
- Dispatcher.Unconfined : 첫번째 중단 함수 호출을 만날때 까지 호출 스레드에서 Coroutine을 시작하여 중단 이후에  재개(resume)될 때는 중단 함수를 재개한 스레드에서 수행된다. 
    - Coroutine이 CPU 시간을 소모하지 않거나 공유되는 데이터(UI)를 업데이트 하지 않는 경우처럼 특정 스레드에 국한된 작업이 아닌 경우 적절하다.


## Scope

#### GlobalScope
어플리케이션이 동작하는동안 별도의 생명주기를 관리하지않고 사용할 수 있는 Scope로
안드로이드 앱이 처음 시작할때부터 종료할 때 까지 하나의 CoroutineContext안에서 동작한다.  
Dispatchers.Default의 스레드를 사용한다.

#### CoroutineScope
특정한 목적의 Dispatcher를 지정하여 새로운 Coroutine의 범위를 정의한다.  
모든 Coroutine Builder(launch, async)의 확장이다.  

#### ProducerScope


## Builder
- launch 
    - launch{}의 실행으로 Job을 반환한다
    - 결과가 포함되어있지않다
    - 그자리에서 바로 예외를 발생시킨다
    - join()을 만나면 현재 Coroutine이 완료댈때까지 기다린다 (suspend 된다)
    - CoroutineScope를 사용한다
- async
    - Deffered를 반환한다
    - 결과를가 포함되어있다
    - await()를 만나면 현재 Coroutine이 완료댈때까지 기다린다 (suspend 된다)
    - CoroutineScope를 사용한다
- produce 
    - ReceiveChannel을 반환한다
    - producer-consumer패턴을 사용할 수 있도록 하는 빌더
    - ProducerScope를 사용한다
- runBlocking
    - Coroutine을 생성한 후 Coroutine이 완료되어 결과를 반환할때까지 현재 Thread를 Block한다
    - runBlocking의 기본 Dispatcher는 시작된 Threa이다
    - CoroutineScope를 사용한다


## Job
- 수명주기를 가지고 있으며 완료로 끝난다. 그리고 취소할 수 있다.  
- 상위-하위 계층구조를 가질 수 있으며, 상위 Job을 취소하면 하위Job이 즉시 취소된다.


#### State
Job은 Coroutine의 6가지 상태를 가지고있다.
Job으로 Coroutine의 상태를 확인할 수 있고 제어할 수 있다.
|state|isActive|isCompleted|isCancelled|
|-------|-------|-------|-------|
|New(optional init state)|false|false|false|
|Active(default init state)|true|false|false|
|Completing(transient state)|true|false|false|
|Cancelling(transient state)|false|false|true|
|Cancelled(final state)|false|true|true|
|Completed(final state)|false|true|false|  

#### Job LifeCycle
![image](https://user-images.githubusercontent.com/39984656/104475773-d21d7000-5602-11eb-9c5e-f889cb959539.png)


#### Functions
- start : 아직 시작되지 않은 Job이 있는경우 관련한 Coroutine을 시작한다
    - 동작중인경우 true, 준비 또는 완료상태이면 false를 return한다
- join : 현재 동작중인 Coroutine 동작이 끝날대 까지 대기한다
    - async{} await와 비슷한 개념  
- cancel : Job을 취소
- cancelAndJoin : 현재 Job을 취소하고 취소된 작업이 완료될때까지 대기한다.
- cancelChildren : 현재 Coroutine의 하위 Job을 모두 취소한다
- ensureActive : 현재 Job이 활성 상태인지 확인
- invokeOnCompletion : Job에 핸들러를 등록하고 완료 또는 취소될때 핸들러 호출


#### Inheritors
- CompletableJob : complete()를 사용하여 Job을 완료처리할 수 있다.
- SupervisorJob : children Job들중 하나가 실패하더라도 다른 children Job에 영향을 주지 않는다.
- Deferred : async{}에서 수행된 결과가 있는 Job
- NonCancellable : 항상 활성상태인 취소 불가능한 Job


## Channel
Channel은 개념적으로 [BlockingQueue](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/BlockingQueue.html)와 유사하다.
BlockingQueue에서 put을 할때 가득 차있다면 Block하여 대기하고, take를 할때 비어있다면 Block하여 대기한다
Channel은 비슷하게 send와 receive를 사용한다.
따라서 여러 coroutine 사이에 순서를 보장받으며 공유하여 사용할 수 있다.
또한 Channel은 Queue와 달리 더 이상 사용하지 않을때 close 시킬 수 있다.


#### Functions
- send : Element를 채널로 전송. 채널의 버퍼가 가득차 있다면 Block하여 대기
- receive : Element를 전달받아 사용. 채널이 비어있다면 Block하여 대기
- cancel : 채널의 나머지 Element 수신을 취소한다. 채널을 닫고 버퍼링된 모든 Element들을 제거한다
- close : 채널을 닫지만, 버퍼링된 요소들은 사용가능하다
- invokeOnClose : 채널이 닫히거나 취소될때 호출되는 핸들러를 등록한다
- offer : 채널에 Element를 추가하지만 coroutine을 suspend하지않는다
- poll :  채널에서 Element를 받지만 coroutine을 suspend하지않는다

#### Capacity Policy
- RENDEVOUS
    - 기본용량정책
    - 버퍼가 없음
![image](https://user-images.githubusercontent.com/39984656/104595869-8a582080-56b6-11eb-9f08-1eaf176fbd74.png)  
- BUFFERED
    - 채널에 용량을 부여
    - 버퍼의 용량이 가득차게되면 
![image](https://user-images.githubusercontent.com/39984656/104595947-a360d180-56b6-11eb-91ef-bb4ac75da032.png)
- UNLIMITED
    - 채널에 무제한 용량을 부여 (메모리 고갈시 까지)
    - Coroutine을 suspend 하지않고 모든 Element를 채널에 넣을 수 있다
![image](https://user-images.githubusercontent.com/39984656/104596041-c5f2ea80-56b6-11eb-8a63-faa006d3fd0f.png)
- CONFLATED
    - send시 suspend하지않는다
    - receive시에 가장 최신의 Element만 가져온다
![image](https://user-images.githubusercontent.com/39984656/104596084-d905ba80-56b6-11eb-9027-1c55b2e7b5a0.png)


## Top-Level Suspending Functions
- delay : 스레드를 차단하지않고 Coroutine을 주어진 시간동안 지연한뒤 다시 시작
- yeild : 현재 Coroutine의 Dispatcher의 Thread를 가능한경우 다른 Coroutine에 양보한다
- withContext : 지정된 CoroutineContext를 다른 CoroutineContext로 변경한다
    - CoroutineContext의 새 Dispatcher가 지정된 경우에는 block실행을 다른 스레드로 이동하고 완료될시에 원래 Dispatcher로 돌아간다
- withTimeout : 지정된 제한시간내에 block내의 Coroutine이 완료되지않을경우 TimeoutCancellationException을 throw한다
- withTimoeoutOrNull : 지정된 제한시간내에 block내의 Coroutine이 완료되지않을경우 null을 반환한다
- awaitAll : 주어진 모든 작업의 완료를 기다린다 
    - 주어진 모든 작업이 완료될때까지 현재 Coroutine을 suspend한다
    - e.g. async{}의 완료 대기
- joinAll : 주어진 모든 작업이 완료될때까지 기다린다 
    - 주어진 모든 작업이 완료될때까지 현재 Coroutine을 suspend한다
    - e.g. launch{}의 완료 대기