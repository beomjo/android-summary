# MockK

## 목차
- [MockK](#mockk)
  - [목차](#목차)
  - [MockK](#mockk-1)
  - [장점](#장점)
  - [설정](#설정)
  - [동작 검증](#동작-검증)
  - [어노테이션사용](#어노테이션사용)
  - [Spy](#spy)
  - [Relaxed mock](#relaxed-mock)
  - [Unit을 반환하는 함수에 대한 Mock relaxed](#unit을-반환하는-함수에-대한-mock-relaxed)
  - [Object mocks](#object-mocks)
  - [Enumeration(열거형) mocks](#enumeration열거형-mocks)
  - [부분적인 Argument Matcher 적용](#부분적인-argument-matcher-적용)
  - [연속 호출(Chained calls)](#연속-호출chained-calls)
  - [계층적 mocking](#계층적-mocking)
  - [Capture](#capture)
  - [최소(atLeast), 최대(atMost) 또는 정확하게(expactly)](#최소atleast-최대atmost-또는-정확하게expactly)
  - [순서 확인](#순서-확인)
  - [모든 호출 확인](#모든-호출-확인)
  - [verify 시간초과](#verify-시간초과)
  - [Unit 반환](#unit-반환)
  - [Coroutine](#coroutine)
  - [확장함수](#확장함수)
  - [private 함수 mocking, 동적 호출](#private-함수-mocking-동적-호출)


## MockK
MockK는 코틀린을 위한 Mock 프레임워크이다.  
자바에서 주로 사용하는 Mockito 유사해서 약간만 노력하면 쉽게 적응할 수 있다.  
간단한 사용법을 정리하며, 자세한 모든 사용법은 [mockk.io](https://mockk.io/)에서 확인하자.  


## 장점
- kotlin 기본 예약어와 겹치지않게 사용가능
- kotlin 언어 스타일을 살린 테스트 작성 (람다, 중위함수등)
- open 클래스가 아니여도 Mocking가능
- object Mocking가능
- enum Mocking가능
- 계층적 Mocking가능
- 확장함수 Mocking가능
- private 함수 Mocking, Call 가능
- LiveData 테스트 
- Coroutine 테스트


## 설정
종속성 추가하기
```
dependencies {
    testImplementation "io.mockk:mockk:{version}"
}
```


## 동작 검증
```kotlin
//given
val car = mockk<Car>()
every { car.drive(Direction.NORTH) } returns OutCome.OK

//when
car.drive(Direction.NORTH)

//then
verify { car.drive(Directino.NORTH) }
confirmVerified(car)
```


## 어노테이션사용
어노테이션을 사용하여 mock객체 생성을 단순화 할 수 있다.  
```kotlin
class TrafficSystem {
  lateinit var car1: Car
  
  lateinit var car2: Car
  
  lateinit var car3: Car
}
```

```kotlin
class CarTest {
  @MockK
  lateinit var car1: Car

  @RelaxedMockK
  lateinit var car2: Car

  @MockK(relaxUnitFun = true)
  lateinit var car3: Car

  @SpyK
  var car4 = Car()
  
  @InjectMockKs
  var trafficSystem = TrafficSystem()
  
  @Before
  fun setUp() = MockKAnnotations.init(this, relaxUnitFun = true) // turn relaxUnitFun on for all mocks

  @Test
  fun calculateAddsValues1() {
      // ... use car1, car2, car3 and car4
  }
}
```


## Spy
```kotlin
//given
val car = spyk(Car()) // or spyk<Car>() to call default constructor


//when
car.drive(Direction.NORTH) // returns whatever real function of Car returns

//then
verify { car.drive(Direction.NORTH) }
confirmVerified(car)
```


## Relaxed mock
`relaxed mock`는 PrimitiveType을 반환하는 모든 함수에 대해 기본값을 반환하는 mock 객체이다.  
이를 통해 각 경우에 대한 Stubbing(when)을 건너뛰고 필요한 것을 stub할 수 있다.  
`relaxed mock`의 메소드를 호출하면 `0`, `false`, `""`등과 같은 기본값을 반환한다.  
ReferenceType을 반환하는 메소드는 Stubbing해주어야한다.  
```kotlin
//given
val car = mockk<Car>(relaxed = true)

//when
car.drive(Direction.NORTH) // returns null


//then
verify { car.drive(Direction.NORTH) }
confirmVerified(car)
```

```kotlin
 // given
val service = mockk<TestableService>(relaxed = true)
 
// when
val result = service.getDataFromDb("Any Param")
 
// then
assertEquals("", result)
```


## Unit을 반환하는 함수에 대한 Mock relaxed 
만약 Unit을 반환하는 함수가 있다면 따로 Stubbing 하지않고 아래와같이 사용할 수 있다.  

```kotlin
// mockk 함수의 인자로 전달
mockk<MockCls>(releaxUnitFun = ture)
```

```kotlin
// Annotation으로 Mock을 생성하는경우 Annotation 인자로 전달
@MockK(relaxUnitFun = true)
lateinit var mock1 : RufMockCls

init {
    MockKAnnotations.init(this)
}
```

```kotlin
// MockKAnnotations.init 함수 호출시 인자로 전달
@MockK
lateinit var mock2 : RufMockCls

init {
    MockKAnnotations.init(this, relaxUnitFun = true)
}
```


## Object mocks
Object도 아래와같이 mock으로 쓸 수 있다.
```kotlin
object MockObj {
    fun add(a: Int, b: Int) = a + b
}
```

```kotlin
mockkObject(MockObj)

assertEquals(3, MockObj.add(1, 2))

every { MockObj.add(1, 2) } returns 55

assertEquals(55, MockObj.add(1, 2))
```

Kotlin 언어 제한에도 불구하고 테스트 로직에서 필요한 경우 새 객체 인스턴스를 만들 수 있다.
```kotlin
val newObjectMock = mockk<MockObj>()
```


## Enumeration(열거형) mocks
Enum은 `mockObject`를 통해 mock객체로 만들 수 있다.
```kotlin
enum class Enumeration(val goodInt: Int) {
    CONSTANT(35),
    OTHER_CONSTANT(45);
}
```

```kotlin
//given
mockkObject(Enumeration.CONSTANT)
every { Enumeration.CONSTANT.goodInt } returns 42

//when
val result = Enumeration.CONSTANT.goodInt

//then
assertEquals(42, result)
```


## 부분적인 Argument Matcher 적용
일반 인수와 매쳐를 모두 혼합하여 사용 가능하다.
```kotlin
// given
val car = mockk<Car>()
every { 
  car.recordTelemetry(
    speed = more(50),
    direction = Direction.NORTH, // here eq() is used
    lat = any(),
    long = any()
  )
} returns Outcome.RECORDED

// when
obj.recordTelemetry(60, Direction.NORTH, 51.1377382, 17.0257142)

// then
verify { obj.recordTelemetry(60, Direction.NORTH, 51.1377382, 17.0257142) }
confirmVerified(obj)
```


## 연속 호출(Chained calls)
연속해서 호출되는 경우도 쉽게 한번의 Stubbing으로 가능하다.
```kotlin
// given
val car = mockk<Car>()
every { car.door(DoorType.FRONT_LEFT).windowState() } returns WindowState.UP

// when
car.door(DoorType.FRONT_LEFT) // returns chained mock for Door
car.door(DoorType.FRONT_LEFT).windowState() // returns WindowState.UP

// then
verify { car.door(DoorType.FRONT_LEFT).windowState() }
confirmVerified(car)
```


## 계층적 mocking
```kotlin
interface AddressBook {
    val contacts: List<Contact>
}

interface Contact {
    val name: String
    val telephone: String
    val address: Address
}

interface Address {
    val city: String
    val zip: String
}

val addressBook = mockk<AddressBook> {
    every { contacts } returns listOf(
        mockk {
            every { name } returns "John"
            every { telephone } returns "123-456-789"
            every { address.city } returns "New-York"
            every { address.zip } returns "123-45"
        },
        mockk {
            every { name } returns "Alex"
            every { telephone } returns "789-456-123"
            every { address } returns mockk {
                every { city } returns "Wroclaw"
                every { zip } returns "543-21"
            }
        }
    )
}
```


## Capture
`CapturingSlot` 또는 `MutableList` 에 대한 인수를 캡처할 수 있다.
```kotlin
// given
val car = mockk<Car>()
val slot = slot<Double>()
val list = mutableListOf<Double>()
every {
  obj.recordTelemetry(
    speed = capture(slot), // makes mock match call with any value for `speed` and record it in a slot
    direction = Direction.NORTH // makes mock and capturing only match calls with specific `direction`. Use `any()` to match calls with any `direction`
  )
} answers {
  println(slot.captured)

  Outcome.RECORDED
}
every {
  obj.recordTelemetry(
    speed = capture(list),
    direction = Direction.SOUTH
  )
} answers {
  println(list)

  Outcome.RECORDED
}


// when
obj.recordTelemetry(speed = 15, direction = Direction.NORTH) // prints 15
obj.recordTelemetry(speed = 16, direction = Direction.SOUTH) // prints 16


// then
verify(exactly = 2) { obj.recordTelemetry(speed = or(15, 16), direction = any()) }
confirmVerified(obj)
```


## 최소(atLeast), 최대(atMost) 또는 정확하게(expactly)
`atLeast`, `atMost`, `exactly` 매개변수를 사용하여 호출 수를 확인할 수 있다.
```kotlin
// given
val car = mockk<Car>(relaxed = true)

// when
car.accelerate(fromSpeed = 10, toSpeed = 20)
car.accelerate(fromSpeed = 10, toSpeed = 30)
car.accelerate(fromSpeed = 20, toSpeed = 30)

// then
// all pass
verify(atLeast = 3) { car.accelerate(allAny()) }
verify(atMost  = 2) { car.accelerate(fromSpeed = 10, toSpeed = or(20, 30)) }
verify(exactly = 1) { car.accelerate(fromSpeed = 10, toSpeed = 20) }
verify(exactly = 0) { car.accelerate(fromSpeed = 30, toSpeed = 10) } // means no calls were performed
confirmVerified(car)
```


## 순서 확인
- `verifyAll` : 순서를 확인하지 않고 모든 호출이 발생했는지 확인
- `verifySequence` : 내부적인 호출이 지정된 순서대로 발생했는지 확인
- `verifyOrder` : 호출이 { }블럭안에 명시된 순서대로 발생했는지 확인
- `wasNot Called` : 호출이 전혀 되지않았는지 확인
```kotlin
lass MockedClass {
    fun sum(a: Int, b: Int) = a + b
}

val obj = mockk<MockedClass>()
val slot = slot<Int>()

every {
    obj.sum(any(), capture(slot))
} answers {
    1 + firstArg<Int>() + slot.captured
}



obj.sum(1, 2) // returns 4
obj.sum(1, 3) // returns 5
obj.sum(2, 2) // returns 5



verifyAll {
    obj.sum(1, 3)
    obj.sum(1, 2)
    obj.sum(2, 2)
}

verifySequence {
    obj.sum(1, 2)
    obj.sum(1, 3)
    obj.sum(2, 2)
}

verifyOrder {
    obj.sum(1, 2)
    obj.sum(2, 2)
}

val obj2 = mockk<MockedClass>()
val obj3 = mockk<MockedClass>()
verify {
    listOf(obj2, obj3) wasNot Called
}

confirmVerified(obj)
```


## 모든 호출 확인
모든 호출이 `verify`구문에 의해 확인되었는지 다시 확인하려면 `confirmVerified`를 사용할 수 있다.  
확인 없이 일부 호출이 남아있는경우 예외가 발생한다.  
```kotlin
confirmVerified(mock1, mock2)
```


## verify 시간초과
`verify`가 통과되거나 시간 초과에 도달할 때까지 기다린다.  
```kotlin
mockk<MockCls> {
    every { sum(1, 2) } returns 4

    Thread {
        Thread.sleep(2000)
        sum(1, 2)
    }.start()

    verify(timeout = 3000) { sum(1, 2) }
}
```


## Unit 반환
함수가 Unit을 반환한다면 `justRun` 구문을 사용할 수 있다.
```kotlin
class MockedClass {
    fun sum(a: Int, b: Int): Unit {
        println(a + b)
    }
}
```

```kotlin
// given
val obj = mockk<MockedClass>()
justRun { obj.sum(any(), 3) }


// when
obj.sum(1, 1)
obj.sum(1, 2)
obj.sum(1, 3)


// then
verify {
    obj.sum(1, 1)
    obj.sum(1, 2)
    obj.sum(1, 3)
}
```

 `justRun { obj.sum(any(), 3) }`은 다음과 같이 작성할 수도 있다.

```kotlin
every { obj.sum(any(), 3) } just Runs
every { obj.sum(any(), 3) } returns Unit
every { obj.sum(any(), 3) } answers { Unit }
```


## Coroutine
`coEvery`, `coVerify`, `coMatch`, `coAssert`, `coRun`, `coAnswers`또는 `coInvoke`를 사용할 수 있다.
```kotlin
// given
val car = mockk<Car>()
coEvery { car.drive(Direction.NORTH) } returns Outcome.OK

// when
car.drive(Direction.NORTH) // returns OK

// then
coVerify { car.drive(Direction.NORTH) }
```


## LiveData
LiveData는 Observer를 mocking하여 호출되었는지 verify하는 형식으로 테스트 가능하다.
```kotlin
// given
val mockIntent = mockk<Intent>()
val isLoginSuccess = true
val loginSuccessObserver = mockk<Observer<Boolean>> {
    every { onChanged(isLoginSuccess) } just Runs
}
viewModel.loginSuccess.observeForever(loginSuccessObserver)

// when
 viewModel.processGoogleLogin(mockIntent)

// verify
verify(loginSuccessObserver.onChanged(eq(isLoginSuccess)))
```


## 확장함수
확장함수도 mocking 할 수 있다.  
객체나 클래스의 경우 단지 `mockk`로 생성하는 것만으로도 확장함수를 mocking 할 수 있다.
```kotlin
data class Obj(val value: Int)

class Ext {
    fun Obj.extensionFunc() = value + 5
}

with(mockk<Ext>()) {
    every {
        Obj(5).extensionFunc()
    } returns 11

    assertEquals(11, Obj(5).extensionFunc())

    verify {
        Obj(5).extensionFunc()
    }
}
```

모듈 전체 확장 함수를 mocking 하려면 `mockkStatic(...)`을 사용하여 모듈 클래스 이름을 인수로 빌드해야한다.
```kotlin
data class Obj(val value: Int)

// declared in File.kt ("com.example.extensions" package) 
fun Obj.extensionFunc() = value + 5

mockkStatic("com.example.extensions.FileKt")

every {
    Obj(5).extensionFunc()
} returns 11

assertEquals(11, Obj(5).extensionFunc())

verify {
    Obj(5).extensionFunc()
}
```

jvm 환경에서 클래스 이름을 함수 참조로 바꿀 수도 있다.
```kotlin
mockkStatic(Obj::extensionFunc)
```


## private 함수 mocking, 동적 호출
private 함수를 mocking해야하는 경우, 호출해야하는 경우 함수명을 통해 할 수 있다.
```kotlin
class Car {
    fun drive() = accelerate()

    private fun accelerate() = "going faster"
}
```

```kotlin
// given
val mock = spyk<Car>(recordPrivateCalls = true)
every { mock["accelerate"]() } returns "going not so fast"

// when
val result = mock.drive()


// then
assertEquals("going not so fast", result)
verifySequence {
    mock.drive()
    mock["accelerate"]()
}
```

private 함수 호출을 확인하려면 `spyk` 와 `recordPrivateCalls = true`로 설정해야 한다.  
또한 아래 문법을 사용하여도 private 함수를 검증할 수 있다.  
```kotlin
val mock = spyk(Team(), recordPrivateCalls = true)

every { mock getProperty "speed" } returns 33
every { mock setProperty "acceleration" value less(5) } just runs
justRun { mock invokeNoArgs "privateMethod" }
every { mock invoke "openDoor" withArguments listOf("left", "rear") } returns "OK"

verify { mock getProperty "speed" }
verify { mock setProperty "acceleration" value less(5) }
verify { mock invoke "openDoor" withArguments listOf("left", "rear") }
```