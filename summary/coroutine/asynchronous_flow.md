<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
# Asynchronous Flow  

<!-- code_chunk_output -->

- [Asynchronous Flow](#asynchronous-flow)
  - [여러 값을 표시하는 방법](#여러-값을-표시하는-방법)
      - [Collection](#collection)
      - [Sequences](#sequences)
      - [Suspending functions](#suspending-functions)
    - [Flows](#flows)
  - [플로우는 차갑다(Flows are cold)](#플로우는-차갑다flows-are-cold)
  - [플로우의 취소(Flow cancellation basics)](#플로우의-취소flow-cancellation-basics)
  - [플로우 빌더(Flow Builders)](#플로우-빌더flow-builders)
  - [중간 연산자(Intermedicate flow operators)](#중간-연산자intermedicate-flow-operators)
    - [변환 연산자(Transform operator)](#변환-연산자transform-operator)
    - [크기 제한 연산자(Size-limiting operators)](#크기-제한-연산자size-limiting-operators)
  - [플로우 종단 연산자(Terminal flow operator)](#플로우-종단-연산자terminal-flow-operator)
  - [플로우는 순차적이다(Flow are sequential)](#플로우는-순차적이다flow-are-sequential)
  - [플롱우 컨텍스트(Flow Context)](#플롱우-컨텍스트flow-context)
    - [withContext를 통한 잘못 된 방출(Wrong emission withContext)](#withcontext를-통한-잘못-된-방출wrong-emission-withcontext)
    - [flowOn 연산자(flowOn operator)](#flowon-연산자flowon-operator)
  - [버퍼링(Buffering)](#버퍼링buffering)
    - [병합(Conflation)](#병합conflation)
    - [최신 값 처리(Processing the latest value)](#최신-값-처리processing-the-latest-value)
  - [다중 플로우 합성(Composing multiple flows)](#다중-플로우-합성composing-multiple-flows)
    - [Zip](#zip)
    - [Combine](#combine)
  - [플로우 플래트닝(Flattening flows)](#플로우-플래트닝flattening-flows)
    - [flatMapConcat](#flatmapconcat)
    - [flatMapMerge](#flatmapmerge)
    - [flatMapLatest](#flatmaplatest)
  - [플로우 예외(Flow Exception)](#플로우-예외flow-exception)
    - [수집기의 try and catch(Collector try and catch)](#수집기의-try-and-catchcollector-try-and-catch)
    - [모든 예외처리(Everthing is cautch)](#모든-예외처리everthing-is-cautch)
  - [예외 투명성(Exception transparency)](#예외-투명성exception-transparency)
    - [catch 예외 투명성(Transparent catch)](#catch-예외-투명성transparent-catch)
    - [선억적인 에러 캐치(Catching declaratively)](#선억적인-에러-캐치catching-declaratively)
  - [플로우 완료(Flow completion)](#플로우-완료flow-completion)
    - [선언적인 처리(Declarative handling)](#선언적인-처리declarative-handling)
  - [플로우 실행(Launching flow)](#플로우-실행launching-flow)
  - [플로우 취소 확인(Flow cancellation checks)](#플로우-취소-확인flow-cancellation-checks)
    - [바쁜 flow를 취소 가능하게 만들기(Making busy flow cancellable)](#바쁜-flow를-취소-가능하게-만들기making-busy-flow-cancellable)

<!-- /code_chunk_output -->


## 여러 값을 표시하는 방법   

#### Collection
모든 연산을 수행한 수 한번에 `List<Int>`를 반환 한다
```
fun simple(): List<Int> = listOf(1, 2, 3)

fun main() {
    simple().forEach { value -> println(value) } 
}
```

#### Sequences 
중간에 리스트를 만들지 않고 값을 순차적으로 계산한다
```
fun simple(): Sequence<Int> = sequence { // sequence builder
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it
        yield(i) // yield next value
    }
}

fun main() {
    simple().forEach { value -> println(value) } 
}
```
```
listOf(1, 2, 3, 4)
    .asSequence()
    .map { it * it }
    .find { it > 3 }

// list(1) -> map(1*1) -> find(1>3) -> list(2) -> map(2*2) -> find(4>3) -> 종료
```
[Kotlin-Sequences](https://kotlinlang.org/docs/reference/sequences.html)

#### Suspending functions
```
suspend fun simple(): List<Int> {
    delay(1000) // pretend we are doing something asynchronous here
    return listOf(1, 2, 3)
}

fun main() = runBlocking<Unit> {
    simple().forEach { value -> println(value) } 
}
```

### Flows
List가 모든 연산을 수행한 후 한번에 모든값을 반환하는것과 반대로  
Flow는 Sequnce와 같이 순차적으로 값을 내보내고 정상적으로 완료 또는 예외가 발생하는 비동기 스트림이다.
```
fun simple(): Flow<Int> = flow { // flow builder
    for (i in 1..3) {
        delay(100) // pretend we are doing something useful here
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    // Launch a concurrent coroutine to check if the main thread is blocked
    launch {
        for (k in 1..3) {
            println("I'm not blocked $k")
            delay(100)
        }
    }
    // Collect the flow
    simple().collect { value -> println(value) } 
}
```
위 코드는 
- Flow 타입을 생성은 `flow { }` 빌더를 이용한다
- `flow { ... }` 블록 안의 코드는 중단 가능하다
- simple() 함수는 더이상 `suspend` 로 선언하지 않는다
- 결과 값들은 flow 에서 `emit()` 함수를 이용하여 방출된다
- flow 에서 방출된 값들은 `collect` 함수를 이용하여 수집된다
- main thread를 차단하지 않고 `println(value)`하기 전에 100ms를 대기한다

```
I'm not blocked 1
1
I'm not blocked 2
2
I'm not blocked 3
3
```


## 플로우는 차갑다(Flows are cold)
flow는 sequnce와 비슷하게 cold 스트림이다.  
`flow{ }` 빌더가 `collect()` 를 호출할 때 까지 실행되지 않는다.  
이 때문에 `suspend` 로 선언하지 않아도 되는것이다.  


## 플로우의 취소(Flow cancellation basics)
flow의 취소는 coroutine의 일반적인 협력적인 취소를 지킵니다.
```
fun simple(): Flow<Int> = flow { 
    for (i in 1..3) {
        delay(100)          
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    withTimeoutOrNull(250) { // Timeout after 250ms 
        simple().collect { value -> println(value) } 
    }
    println("Done")
}
```
flow가 `withTimeoutOrNull{ }` 에서 실행될때 
```
Emitting 1 
1 
Emitting 2 
2 
Done
```

## 플로우 빌더(Flow Builders)
- flow {  }
    - 기본적인 flow Builder
- [flowOf(...)](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-of.html)
    - 고정된 값을 방출하는 flow 정의
- [awFlow()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/kotlin.-function0/as-flow.html)
    - 다양한 Collection, Sequnce를 확장함수를 통하여 Flow로 변환


## 중간 연산자(Intermedicate flow operators)
Collection이나 Sequnce와 동일하게 연산자로 변환할 수 있다.  
하지만 중요한 차이점은 연산자로 수행되는 코드블럭에서 `suspend` 함수를 호출할 수 있다.  
익숙한 `map()`, `filter()` 등이 대표적인 예시이다.  
```
suspend fun performRequest(request: Int): String {
    delay(1000) // imitate long-running asynchronous work
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow() // a flow of requests
        .map { request -> performRequest(request) }
        .collect { response -> println(response) }
```

### 변환 연산자(Transform operator)
플로우 변환 연산자들 중에서 가장 일반적인 것은 `transform` 연산자다.  
이 연산자는 `map` 이나 `filter()` 같은 단순한 변환이나 혹은 복잡한 다른 변환들을 구현하기 위해 된다.  
`transform()` 연산자를 사용하여 우리는 임의의 횟수로 임의의 값들을 방출할 수 있습니다.  

```
suspend fun performRequest(request: Int): String {
    delay(1000) // imitate long-running asynchronous work
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow() // a flow of requests
    .transform { request ->
        emit("Making request $request") 
        emit(performRequest(request)) 
    }
    .collect { response -> println(response) }
}
```
예를 들어, `transform` 연산자를 사용하여 오래 걸리는 비동기 요청을 수행하기 전에 기본 문자열을 먼저 방출하고  
요청에 대한 응답이 도착하면 그 결과를 방출할 수 있다.  
```
Making request 1 
response 1 
Making request 2 
response 2 
Making request 3 
response 3
```

### 크기 제한 연산자(Size-limiting operators)
take같은 크기 제한 중간 연산자는 정의된 제한치에 도달하면 실행을 취소한다.  
코루틴에서 취소는 언제나 예외를 발생시키는 방식으로 수행 되며,   
이를 통해 `try { ... } finally { ... }` 로 예외 처리등이 가능하다.  
```
fun numbers(): Flow<Int> = flow {
    try {                          
        emit(1)
        emit(2) 
        println("This line will not execute")
        emit(3)    
    } finally {
        println("Finally in numbers")
    }
}

fun main() = runBlocking<Unit> {
    numbers() 
        .take(2) // take only the first two
        .collect { value -> println(value) }
}  
```
```
1 
2 
Finally in numbers
```


## 플로우 종단 연산자(Terminal flow operator)
 플로우 수집을 시작하는 중단 함수이다.
 - `collect()`
 - `toList()`
 - `toSet()`
 - `reduce()`
 - `fold()`

 ```
 val sum = (1..5).asFlow()
    .map { it * it } // squares of numbers from 1 to 5                           
    .reduce { a, b -> a + b } // sum them (terminal operator)
println(sum)
 ```


## 플로우는 순차적이다(Flow are sequential)
Flow는 Sequnce처럼 기본적으로 `collect()` 등의 종단 연산자가 호출될 때 순차적으로 연산된다.
```
(1..5).asFlow()
    .filter {
        println("Filter $it")
        it % 2 == 0              
    }              
    .map { 
        println("Map $it")
        "string $it"
    }.collect { 
        println("Collect $it")
    }    
```

```
Filter 1
Filter 2
Map 2
Collect string 2
Filter 3
Filter 4
Map 4
Collect string 4
Filter 5
```

## 플롱우 컨텍스트(Flow Context)
flow의 수집(종단함수 호출)은 항상 CoroutineContext안에서 수행된다.  
이를 **컨텍스트 보존(context preservation)** 이라 한다.  

```
fun simple(): Flow<Int> = flow {
    log("Started simple flow")
    for (i in 1..3) {
        emit(i)
    }
}  

fun main() = runBlocking<Unit> {
    simple().collect { value -> log("Collected $value") } 
}      
```
```
[main @coroutine#1] Started simple flow
[main @coroutine#1] Collected 1
[main @coroutine#1] Collected 2
[main @coroutine#1] Collected 3
```

### withContext를 통한 잘못 된 방출(Wrong emission withContext)
`flow{ }` 블럭 내에서 `withContext` 로 CoroutineContext를 변경하면 안된다.  
```
fun simple(): Flow<Int> = flow {
    kotlinx.coroutines.withContext(Dispatchers.Default) {
        for (i in 1..3) {
            Thread.sleep(100)
            emit(i) 
        }
    }
}

fun main() = runBlocking<Unit> {
    simple().collect { value -> println(value) } 
}   
```
다음과 같은 에러 발생
```
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
        Flow was collected in [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@5511c7f8, BlockingEventLoop@2eac3323],
        but emission happened in [CoroutineId(1), "coroutine#1":DispatchedCoroutine{Active}@2dae0000, Dispatchers.Default].
        Please refer to 'flow' documentation or use 'flowOn' instead
    at ...
```

### flowOn 연산자(flowOn operator)
flow에서 CoroutineContext를 변경하려면 `flowOn()` 연산자를 사용해야한다.  
```
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        Thread.sleep(100) 
        log("Emitting $i")
        emit(i) / emit next value
    }
}.flowOn(Dispatchers.Default) 

fun main() = runBlocking<Unit> {
    simple().collect { value ->
        log("Collected $value") 
    } 
}       
```
`flow {}` 은 Background에서 동작하며,  
그 이후 `collect()` 는 MainThread에서 실행된다.
```
[DefaultDispatcher-worker-1 @coroutine#2] Emitting 1
[main @coroutine#1] Collected 1
[DefaultDispatcher-worker-1 @coroutine#2] Emitting 2
[main @coroutine#1] Collected 2
[DefaultDispatcher-worker-1 @coroutine#2] Emitting 3
[main @coroutine#1] Collected 3
```
flowOn 연산자가 CoroutineDispatcher를 변경할 buffering 매커니즘을 사용하게되어  
`flow{ }` 의 기본적인 특성인 순차성을 잃어버리게 될 수 있다.


## 버퍼링(Buffering)
비동기 작업의 경우에 buffer를 사용하였을 때 시간이 단축되는 경우도 있다.
```
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().collect { value -> 
            delay(300)
            println(value) 
        } 
    }   
    println("Collected in $time ms")
}
```
`(100ms + 300ms) * 3 = 1200ms` 의 시간이 걸린다
```
1
2
3
Collected in 1220 ms
```
아래와 같이 buffer를 사용해보면
```
val time = measureTimeMillis {
    simple()
        .buffer() // buffer emissions, don't wait
        .collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
}   
println("Collected in $time ms")
```
첫 번째 수를 위해서 100ms를 기다린 뒤 
각각의 수처리를 위해서 300ms를 기다리게 된다
`100ms + (300ms * 3) = 1000ms`
```
1
2
3
Collected in 1034 ms
```

또한 flowOn 연산자가 
flowOn 연산자가 CoroutineDispatcher를 변경할 경우 동일한 버퍼링 매커니즘을 사용한다.

### 병합(Conflation)
flow가 연산의 일부분이나, 상태의 업데이트만을 처리해야 할 경우 `conflate()`를 병합을 사용할 수 있다.  
`conflate()` 을 사용하여 `collect()` 의 처리가 너무 느릴 경우 방출된 중간 값을 스킵할 수 있다.  

```
val time = measureTimeMillis {
    simple()
        .conflate() // conflate emissions, don't process each one
        .collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
}   
println("Collected in $time ms")
```
두 번째 수를 스킵하고 가장 최근 값인 세 번째 수가 `collect()` 로 전달된다.  
```
1 
3 
Collected in 758 ms
```

### 최신 값 처리(Processing the latest value)
중간값을 모두 삭제하여 최신의 값만을 처리하여 속도를 높이는 방법도 있다.  
`collectLatest()` 를 사용하여 
새로운 값이 emit될 때 마다 기존의 `collect` 작업을 취소하고 재시작 한다.  
```
val time = measureTimeMillis {
    simple()
        .collectLatest { value -> // cancel & restart on the latest value
            println("Collecting $value") 
            delay(300) // pretend we are processing it for 300 ms
            println("Done $value") 
        } 
}   
println("Collected in $time ms")
```
마지막 값 3에 대해서만 `collect` 를 끝까지 수행한다.  
약 `(100ms * 3) + 300ms = 600ms` 의 시간이 소요된다.  
```
Collecting 1
Collecting 2
Collecting 3
Done 3
Collected in 677 ms
```


## 다중 플로우 합성(Composing multiple flows)
여러가지 flow를 합성하는 여러가지 방법이 있다.

### Zip 
```
val nums = (1..3).asFlow() 
val strs = flowOf("one", "two", "three")
nums.zip(strs) { a, b -> "$a -> $b" }
    .collect { println(it) } 
```

### Combine
두 flow에서 방출이 일어날때마다 다른 flow의 최신값을 가지고 병합하여 출력.
```
val nums = (1..3).asFlow().onEach { delay(300) } 
val strs = flowOf("one", "two", "three").onEach { delay(400) } 

val startTime = System.currentTimeMillis() 
nums.combine(strs) { a, b -> "$a -> $b" } 
    .collect { value -> // collect and print 
        println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
    } 
```
```
1 -> one at 452 ms from start 
2 -> one at 651 ms from start 
2 -> two at 854 ms from start 
3 -> two at 952 ms from start 
3 -> three at 1256 ms from start
```


## 플로우 플래트닝(Flattening flows)
`flow{ flow{ } }` 처럼 flow가 중첩되는 경우가 있다. ( `Flow<Flow<String>>`)
예를들어 
```
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}
```
```
(1..3).asFlow().map { requestFlow(it) }
```
이러한 경우 flattening이 연산자로 flattning이 필요하다

### flatMapConcat
`flatMapConcat()`은 현재 flow가 완료된 후 그 다음 flow를 순차적으로 처리하고 결과로 Flow를 반환한다
```
val startTime = System.currentTimeMillis() 
(1..3).asFlow().onEach { delay(100) 
    .flatMapConcat { requestFlow(it) }              
    .collect { value -> // collect and print 
        println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
    } 
```
```
1: First at 121 ms from start 
1: Second at 622 ms from start 
2: First at 727 ms from start 
2: Second at 1227 ms from start 
3: First at 1328 ms from start 
3: Second at 1829 ms from start
```

### flatMapMerge
`flatMapMerge()`는 두 flow를 동시에 수집하고 가능한 빨리 값을 방출하도록 한다.
```
val startTime = System.currentTimeMillis() 
(1..3).asFlow().onEach { delay(100) }
    .flatMapMerge { requestFlow(it) }
    .collect { value -> // collect and print 
        println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
    } 
```
```
1: First at 136 ms from start
2: First at 231 ms from start
3: First at 333 ms from start
1: Second at 639 ms from start
2: Second at 732 ms from start
3: Second at 833 ms from start
```

### flatMapLatest
`flatMapLatest()`는 새로운 flow가 방출될 때마다 직전 flow를 취소한다.
```
val startTime = System.currentTimeMillis() // remember the start time 
(1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
    .flatMapLatest { requestFlow(it) }
    .collect { value -> // collect and print 
        println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
    } 
```
새 값이 방출되면 현재 진행중인 `{ requestFlow(it) }` 를 취소한다.
```
1: First at 142 ms from start
2: First at 322 ms from start
3: First at 425 ms from start
3: Second at 931 ms from start
```

## 플로우 예외(Flow Exception)
flow는 블럭 안에서 코드가 예외를 발생시키면 예외 발생 상태로 종료된다.  
예외처리에 대하여 알아보자

### 수집기의 try and catch(Collector try and catch)
collector에서 `try{  } catch{ }` 를 사용
```
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value ->         
            println(value)
            check(value <= 1) { "Collected $value" }
        }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}   
```
```
Emitting 1
1
Emitting 2
2
Caught java.lang.IllegalStateException: Collected 2
```

### 모든 예외처리(Everthing is cautch)
`flow{ }` 이나, 중간연산자, 종단연산자 등에서 발생하는 모든 에러도 `try{ } catch{ }` 로 예외처리 가능  
```
fun simple(): Flow<String> = 
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "Crashed on $value" }                 
        "string $value"
    }

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}     
```
```
Emitting 1 
string 1 
Emitting 2 
Caught java.lang.IllegalStateException: Crashed on 2
```


## 예외 투명성(Exception transparency)
위의 `try{ } catch{  }` 를 사용하는것은 예외투명성을 위반하는 것이다.  
예외 투명성을 보존하기 위한 방법으로 아래와 같은 방법이 있다.  
- `throw` 연산자를 통한 예외 다시 던지기  
- `catch()` 로직에서 `emit()` 을 사용하여 값 타입으로 방출  
- 다른 코드를 통한 예외 무시, 로깅, 기타 처리  

### catch 예외 투명성(Transparent catch)
`catch()`는 업스트림에서 발생한 예외만을 처리한다.
```
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> println("Caught $e") } // does not catch downstream exceptions
        .collect { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
}     
```
```
Emitting 1
1
Emitting 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
 at FileKt$main$1$invokeSuspend$$inlined$collect$1.emit (Collect.kt:136) 
 ....
```

### 선억적인 에러 캐치(Catching declaratively)
```
simple()
    .onEach { value ->
        check(value <= 1) { "Collected $value" }                 
        println(value) 
    }
    .catch { e -> println("Caught $e") }
    .collect()
```
```
Emitting 1
1
Emitting 2
Caught java.lang.IllegalStateException: Collected 2
```


## 플로우 완료(Flow completion)
flow의 수집이 종료(정상종료 or 예외발생)되엇을 때 그 이후 동작을 처리해야 할 때 가 있다

### 선언적인 처리(Declarative handling)
`onCompletion()` 중간 연산자를 추가하여  
flow가 완전히 수집되었을때 실행할 로직을 정의할 수 있다.

```
simple()
    .onCompletion { println("Done") }
    .collect { value -> println(value) }
```

`onCompletion`을 사용함으로써 얻을 수 있는 최대의 이점은  
람다에 nullable로 정의되는 Throwable 파라미터를 이용해 수집이 성공적으로 종료되었는지 알 수 있다는 점이다.
```
fun simple(): Flow<Int> = flow {
    emit(1)
    throw RuntimeException()
}

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { cause -> if (cause != null) println("Flow completed exceptionally") }
        .catch { cause -> println("Caught exception") }
        .collect { value -> println(value) }
}            
```
`onCompletion()`연산자는 `catch()`와 달리 예외를 처리하지 않는다.
예외는 다운스트림으로 계속 전달된다.
```
1
Flow completed exceptionally
Caught exception
```


다운스트림에서 예외가 발생할 시 Throwable은 null이다.
```
fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { cause -> println("Flow completed with $cause") }
        .collect { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
}
```
```
1
Flow completed with 
java.lang.IllegalStateException: Collected 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
```

## 플로우 실행(Launching flow)
flow에서 발생하는 이벤트들에 대응하는 처리를 각각 해야한다면
중간 연산자 `onEach()`를 사용한다.

```
fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .collect() // <--- Collecting the flow waits
    println("Done")
}       
```
```
Event: 1
Event: 2
Event: 3
Done
```
중간 연사자 이므로 `collect()`를 호출하지 않으면 수집되지않는다.  
  
하지만 이때 `launchIn()`을 사용하면   
flow의 수집을 다른 Coroutine에서 수행할 수 있으며   
이를 통해 이후 작성된 코드들이 곧바로 실행되도록 할 수 있다.  
```
fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .launchIn(this) // <--- Launching the flow in a separate coroutine
    println("Done")
} 
```
```
Done
Event: 1
Event: 2
Event: 3
```

`launchIn()`에 반드시 필요한 인자는 CoroutineScope이다.  
또한 `launchIn()`은 Job을 반환한다.


## 플로우 취소 확인(Flow cancellation checks)
`flow { }` 는 내보낸 각 값에 대하여 자체적으로 `ensureActive` 검사를 수행한다.
```
fun foo(): Flow<Int> = flow { 
    for (i in 1..5) {
        println("Emitting $i") 
        emit(i) 
    }
}

fun main() = runBlocking<Unit> {
    foo().collect { value -> 
        if (value == 3) cancel()  
        println(value)
    } 
}
```
3까지 숫자를 방출하였고 4번째를 방출하고 난뒤에는 `collect` 를 실행할 수 없으므로 
에러가 발생한다.  
```
Emitting 1
1
Emitting 2
2
Emitting 3
3
Emitting 4
Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job="coroutine#1":BlockingCoroutine{Cancelled}@6d7b4f4c
```


하지만 `asFlow()` 와 같은 대부분의 다른 연산자들은  
성능상의 이유로 자체적으로 추가 취소 확인을 수행하지 않는다.
```
          
fun main() = runBlocking<Unit> {
    (1..5).asFlow().collect { value -> 
        if (value == 3) cancel()  
        println(value)
    } 
}
```
1~5까지 모든 숫자가 수집되고 `runBlocking` 이 반환되기 전에 취소가 감지된다
```
1
2
3
4
5
Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job="coroutine#1":BlockingCoroutine{Cancelled}@3327bd23
```

### 바쁜 flow를 취소 가능하게 만들기(Making busy flow cancellable)
`cancellable()` 을 사용하면 `.onEach{ currentCoroutineContext().ensureActive() }`를 수행하여
1~3까지 숫자만 수집된다
```
fun main() = runBlocking<Unit> {
    (1..5).asFlow().cancellable().collect { value -> 
        if (value == 3) cancel()  
        println(value)
    } 
}
```
```
1
2
3
Exception in thread "main" kotlinx.coroutines.JobCancellationException: BlockingCoroutine was cancelled; job="coroutine#1":BlockingCoroutine{Cancelled}@5ec0a365
```