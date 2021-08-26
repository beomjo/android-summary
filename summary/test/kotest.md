# Kotest

## Index

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Kotest](#kotest)
  - [Index](#index)
  - [KoTest란](#kotest란)
  - [테스트 스타일](#테스트-스타일)
    - [Fun Spec](#fun-spec)
    - [String Spec](#string-spec)
    - [Should Spec](#should-spec)
    - [Describe Spec](#describe-spec)
    - [Behavior Spec](#behavior-spec)
    - [Word Spec](#word-spec)
    - [Free Spec](#free-spec)
    - [Feature Spec](#feature-spec)
    - [Expect Spec](#expect-spec)
    - [Annotation Spec](#annotation-spec)
  - [조건부로 테스트 실행하기](#조건부로-테스트-실행하기)
    - [Config](#config)
    - [Focus](#focus)
    - [Bang](#bang)
    - [X-Method](#x-method)
    - [@Ignored](#ignored)
    - [@EnabledIf](#enabledif)
    - [Gradle에서 필터링하기](#gradle에서-필터링하기)
  - [Isolation Modes](#isolation-modes)
    - [SingleInstance](#singleinstance)
    - [InstancePerTest](#instancepertest)
    - [InstancePerLeaf](#instanceperleaf)
    - [Global Isolation Mode](#global-isolation-mode)
  - [Assertions](#assertions)
    - [Style](#style)
    - [Core Matchers](#core-matchers)
      - [범용](#범용)
      - [타입](#타입)
      - [비교](#비교)
      - [Iterator](#iterator)
      - [Map](#map)
      - [String](#string)
      - [Integer, Long](#integer-long)
      - [Collection](#collection)
      - [Coroutine Channel](#coroutine-channel)
    - [Soft Assertions](#soft-assertions)

<!-- /code_chunk_output -->


## KoTest란
- Kotest DSL을 사용
- 여러 TestStyle을 사용하여 간단하고 아름다운 테스트 작성
- [데이터 기반 테스트](https://kotest.io/docs/framework/datatesting/data-driven-testing.html)로 많은 양의 매개변수도 테스트 가능
- 모든 테스트에 대해 호출수, 병렬처리, 시간제한, 테스트그룹화, 조건부비활성등의 미세 조정 테스트 가능
- 중첩 테스트기능 제공
- 동적테스트 제공 (런타임에 조건부로 테스트를 추가 가능)
- 테스트수명주기에 맞는 다양한 콜백을 제공

## 테스트 스타일
| 테스트 스타일 | 영감받은것 |
| --------- | --------- |
| [Fun Spec](https://kotest.io/docs/framework/testing-styles.html#fun-spec) | ScalaTest |
| [Describe Spec](https://kotest.io/docs/framework/testing-styles.html#describe-spec) | Javascript frameworks a nd RSpec |
| [Should Spec](https://kotest.io/docs/framework/testing-styles.html#should-spec) | A Kotest original |
| [String Spec](https://kotest.io/docs/framework/testing-styles.html#string-spec) | A Kotest original |
| [Behavior Spec](https://kotest.io/docs/framework/testing-styles.html#behavior-spec) | BDD frameworks |
| [Free Spec](https://kotest.io/docs/framework/testing-styles.html#free-spec) | ScalaTest |
| [Word Spec](https://kotest.io/docs/framework/testing-styles.html#word-spec) | ScalaTest |
| [Feature Spec](https://kotest.io/docs/framework/testing-styles.html#feature-spec) | Cucumber |
| [Expect Spec](https://kotest.io/docs/framework/testing-styles.html#expect-spec) |A Kotest original |
| [Annotation Spec](https://kotest.io/docs/framework/testing-styles.html#annotation-spec) |JUnit |

### Fun Spec
`FunSpec` 테스트는 test를 설명하기위해 문자열인수로 호출된 함수를 호출한다음   
테스트 자체를 람다로 호출하여 테스트를 생성할 수 있다.
```kotlin
class MyTests : FunSpec({
    test("String length should return the length of the string") {
        "sammy".length shouldBe 5
        "".length shouldBe 0
    }
})
```

### String Spec
`StringSpec` 테스트는 구문을 절대 최소값으로 줄인다.  
테스트 코드와 함께 문자열 다음에 람다 표현식을 작성하기만 하면 된다.
```kotlin
class MyTests : StringSpec({
    "strings.length should return size of string" {
        "hello".length shouldBe 5
    }
})
```

Config 추가  
```kotlin
class MyTests : StringSpec({
    "strings.length should return size of string".config(enabled = false, invocations = 3) {
        "hello".length shouldBe 5
    }
})
```

### Should Spec
`ShouldSpec`은 `FunSpec`과 비슷하지만 `test`키워드 대신 `should` 키워드를 사용한다.  
```kotlin
class MyTests : ShouldSpec({
    should("return the length of the string") {
        "sammy".length shouldBe 5
        "".length shouldBe 0
    }
})
```

`context` 블록과도 중첩이 가능하다.  
```kotlin
class MyTests : ShouldSpec({
    context("String.length") {
        should("return the length of the string") {
            "sammy".length shouldBe 5
            "".length shouldBe 0
        }
    }
})
```
`xshould`, `xcontext` 등을 사용하여 테스트를 비활성화 할 수 있다. 
```kotlin
class MyTests : ShouldSpec({
    context("this outer block is enabled") {
        xshould("this test is disabled") {
            // test here
        }
    }
    xcontext("this block is disabled") {
        should("disabled by inheritance from the parent") {
            // test here
        }
    }
})
```

### Describe Spec
`DescribeSpec`의 테스트 스타일은 `describe / it` 키워드를 사용한다. (Ruby, JS 스타일)  
테스트는 하나 이상의 `describe` 블록에 중첩되어야한다.  
```kotlin
class MyTests : DescribeSpec({
    describe("score") {
        it("start as zero") {
            // test here
        }
        describe("with a strike") {
            it("adds ten") {
                // test here
            }
            it("carries strike to the next frame") {
                // test here
            }
        }

        describe("for the opposite team") {
            it("Should negate one score") {
                // test here
            }
        }
    }
})
```

`xdescribe`, `xit`을 사용하여 테스트를 비활성화 할 수 있다.  
```kotlin
class MyTests : DescribeSpec({
    describe("this outer block is enabled") {
        xit("this test is disabled") {
            // test here
        }
    }
    xdescribe("this block is disabled") {
        it("disabled by inheritance from the parent") {
            // test here
        }
    }
})
```

### Behavior Spec
BDD<sup>Behavior Driven Development</sup> 스타일의 테스트 작성방법으로  
`given`, `when`, `then`을 사용

`when`이 코틀린 예약어와 겹치므로 역따옴표로 묶어야 한다.   
역따옴표 사용이 마음에 들지 않는 경우 사용할 수 있는 `Given`, `When`, `Then` 으로 사용할 수도 있다. 
```kotlin
class MyTests : BehaviorSpec({
    given("a broomstick") {
        `when`("I sit on it") {
            then("I should be able to fly") {
                // test code
            }
        }
        `when`("I throw it away") {
            then("it should come back") {
                // test code
            }
        }
    }
})
```

`and`를 사용하여 `given`과 `when`에 depth를 추가할 수 있다.  
```kotlin
class MyTests : BehaviorSpec({
    given("a broomstick") {
        and("a witch") {
            `when`("The witch sits on it") {
                and("she laughs hysterically") {
                    then("She should be able to fly") {
                        // test code
                    }
                }
            }
        }
    }
})
```

`xgiven`, `xwhen`, `xthen`을 사용하여 테스트를 비활성화 할 수 있다.
```kotlin
class MyTests : DescribeSpec({
    xgiven("this is disabled") {
        When("disabled by inheritance from the parent") {
            then("disabled by inheritance from its grandparent") {
                // disabled test
            }
        }
    }
    given("this is active") {
        When("this is active too") {
            xthen("this is disabled") {
               // disabled test
            }
        }
    }
})
```

### Word Spec
`WordSpec`은 context 문자열뒤에 테스트를 키워드 `should`를 사용하여 테스트를 작성한다. 
```kotlin
class MyTests : WordSpec({
    "String.length" should {
        "return the length of the string" {
            "sammy".length shouldBe 5
            "".length shouldBe 0
        }
    }
})
```
또한 다른 수준의 중첩을 추가할 수 있는 `When` 키워드를 지원한다.
`when`는 Kotlin의 키워드 이므로 역따옴표 또는 대문자 변형을 사용해야 한다.

```kotlin
class MyTests : WordSpec({
    "Hello" When {
        "asked for length" should {
            "return 5" {
                "Hello".length shouldBe 5
            }
        }
        "appended to Bob" should {
            "return Hello Bob" {
                "Hello " + "Bob" shouldBe "Hello Bob"
            }
        }
    }
})
```

### Free Spec
`FreeSpec` 키워드 `-`(빼기)를 사용하여 임의의 깊이 수준을 중첩할 수 있다.
```kotlin
class MyTests : FreeSpec({
    "String.length" - {
        "should return the length of the string" {
            "sammy".length shouldBe 5
            "".length shouldBe 0
        }
    }
    "containers can be nested as deep as you want" - {
        "and so we nest another container" - {
            "yet another container" - {
                "finally a real test" {
                    1 + 1 shouldBe 2
                }
            }
        }
    }
})
```

### Feature Spec
`FeatureSpec`은  `feature`, `scenario`를 사용한다. ([cucumber](https://cucumber.io/docs/gherkin/reference/)와 비슷하다)
```kotlin
class MyTests : FeatureSpec({
    feature("the can of coke") {
        scenario("should be fizzy when I shake it") {
            // test here
        }
        scenario("and should be tasty") {
            // test here
        }
    }
})
```
`xsceanario`, `xfeature`를 사용하여 테스트를 비활성화 할 수 있다.
```kotlin
class MyTests : FeatureSpec({
    feature("this outer block is enabled") {
        xscenario("this test is disabled") {
            // test here
        }
    }
    xfeature("this block is disabled") {
        scenario("disabled by inheritance from the parent") {
            // test here
        }
    }
})
```

### Expect Spec
`ExpectSpec`은 `FunSpec`, ``ShouldSpec`과 비슷하며, `expect` 키워드를 사용한다.  
```kotlin
class MyTests : ExpectSpec({
    expect("my test") {
        // test here
    }
})
```
`context` 블럭에 하나 이상 중첩하여 작성할 수 있다.  
```kotlin
class MyTests : ExpectSpec({
    context("a calculator") {
        expect("simple addition") {
            // test here
        }
        expect("integer overflow") {
            // test here
        }
    }
})
```
`xexpect`, `xcontext`를 사용하여 테스트를 비활성화 할 수 있다.  
```kotlin
class MyTests : DescribeSpec({
    context("this outer block is enabled") {
        xexpect("this test is disabled") {
            // test here
        }
    }
    xcontext("this block is disabled") {
        expect("disabled by inheritance from the parent") {
            // test here
        }
    }
})
```

### Annotation Spec
JUnit에서 마이그레이션하는 경우 `AnnotationSpecJUnit`을 사용할 수 있다.(`Junit 4/5와 동일한 주석을 사용`)    
정의한 함수에 `@Test ` 주석을 추가하기만 하면 된다.

JUnit과 유사하게 테스트 이전과 테스트 이후등에 무언가를 실행하기 위해 Annotation을 추가할 수도 있다.
```kotlin
@BeforeAll / @BeforeClass
@BeforeEach / @Before
@AfterAll / @AfterClass
@AfterEach / @After

```

JUnit 자체를 사용하는 것보다 이점을 제공하지 않지만 일반적으로 기존 형식을 그대로 가져올 수 있어 더 빠르게 마이그레이션할 수 있다.

```kotlin
class  AnnotationSpecExample : AnnotationSpec () { 

    @BeforeEach fun beforeTest () {
         println ( " 각 테스트 전 " ) 
    } 
    @Test fun test1 () {
         1 shouldBe 1 
    } 
    @Test fun test2 () {
         3 shouldBe 3 
    } 
}
```
테스트를 비활성화 하려면 `@Ignore`를 사용한다.  


## 조건부로 테스트 실행하기
테스트를 비활성화하는 방법에는 여러 가지가 있다. 
테스트에서 하드코딩하거나, 런타임에 조건부로 비활성화 할 수도 있다. 

### Config
`config` 매개변수의 `enabled`를 `false`로 설정하여 테스트 케이스를 비활성화할 수 있다.
```kotlin
"should do something".config(enabled = false) {
  ...
}
```
`SystemUtils.IS_OS_LINUX`등을 사용하여 리눅스환경에서만 테스트를 실행하는등으로 활용할 수도 있다.  
```kotlin
"should do something".config(enabled = IS_OS_LINUX) {
  ...
}
```

값이 아닌 테스트를 기반으로 비활성화를 사용하려면 `enabledIf`를 사용할 수 있다.
아래는 리눅스환경이거나, 테스트이름에 "danger"가 들어가지않는 테스트만 실행한다.  ㄴ
```kotlin
val disableDangerOnWindows: EnabledIf = { !it.name.startsWith("danger") || IS_OS_LINUX }

"danger will robinson".config(enabledIf = disableDangerOnWindows) {
  // test here
}

"very safe will".config(enabledIf = disableDangerOnWindows) {
 // test here
}
```

### Focus
최상위 레벨 테스트 이름 앞에으로 `f:`를 붙여주면 해당 테스트 및 해당범위내에 정의된 모든 하위 테스트들만 실행되고   
나머지는 테스트들은 건너뒨다.
```kotlin
class FocusExample : StringSpec({
    "test 1" {
     // this will be skipped
    }

    "f:test 2" {
     // this will be executed
    }

    "test 3" {
     // this will be skipped
    }
})
```
중첩 테스트는 상위 테스트가 실행된 후에만 검색되기 때문에 포커스모드는 중첩 테스트에 대해서는 작동 하지 않음.

### Bang
Focus의 반대기능으로 테스트이름에 접두어 `!`를 붙힌 테스트만 건너뛴다.
```kotlin
class BangExample : StringSpec({

  "!test 1" {
    // this will be ignored
  }

  "test 2" {
    // this will run
  }

  "test 3" {
    // this will run too
  }
})
```

### X-Method
kotest의 spec들은 테스트함수가 x로 시작하게되면 테스트를 비활성화한다.(JS Style임)  
```kotlin
class XMethodsExample : DescribeSpec({

  xdescribe("this block and it's children are now disabled") {
    it("will not run") {
      // disabled test
    }
  }

})
```

### @Ignored
모든 테스트를 비활성화하려면 `@Ignored` 어노테이션을 사용할 수 있다.   
Spec을 건너뛰고 인스턴스화하지도 않는다.  
```kotlin
@Ignored
class IgnoredSpec : FunSpec() {
  init {
    error("boom") // spec will not be created so this error will not happen
  }
}
```

### @EnabledIf
`@Ignored`와 유사하게 Spec을 인스턴스화해야 하는지 여부를 결정할 수 있다.
```kotlin
class LinuxOnlyCondition : EnabledCondition() {
   override fun enabled(specKlass: KClass<out Spec>): Boolean =
      if (specKlass.simpleName?.contains("Linux") == true) IS_OS_LINUX else true
}
```
적용
```kotlin
@EnabledIf(LinuxOnlyCondition::class)
class MyLinuxTest1 : FunSpec() {
  ..
}

@EnabledIf(LinuxOnlyCondition::class)
class MyLinuxTest2 : DescribeSpec() {
  ..
}
```

### Gradle에서 필터링하기
gradle을 통해 JUnit 플랫폼 러너를 통해 Kotest를 실행할 때 Kotest는 테스트 필터링을 위한 표준 gradle 구문을 지원한다.   
빌드 스크립트에서 또는 `--tests` 명령줄 옵션을 통해 필터링을 활성화할 수 있습니다.
```kotlin
tasks.test {
    filter {
        //include all tests from package
        includeTestsMatching("com.sksamuel.somepackage.*")
    }
}
```

```cmd
$ ./gradlew test --tests 'com.sksamuel.somepackage*'
```
클래스 수준만 필터링 가능하다(개별 메소드 XX)


## Isolation Modes
격리모드(Isolation Modes)를 사용하면 테스트 엔진이 테스트 케이스에 대한 사양 인스턴스를 생성하는 방법을 제어할 수 있다.
`IsolationMode`라는 enum으로 제어할 수 있으며 `SingleInstance`, `InstancePerLeaf`, `InstancePerTest`값이 있다. 

`isolationMode`에 값을 할당하거나, `fun isolationMode()`을 오버라이드 하여 설정할 수 있다.
```kotlin
class MyTestClass : WordSpec({
 isolationMode = IsolationMode.SingleInstance
 // tests here
})
```

```kotlin
class MyTestClass : WordSpec() {
  override fun isolationMode() = IsolationMode.SingleInstance
  init {
    // tests here
  }
}
```

### SingleInstance
기본 격리모드는 `SingleInstance`으로 Spec클래스의 하나의 인스턴스가 생성된 다음   
모든 테스트가 완료될 때까지 각 테스트 케이스가 차례로 실행되는 방식이다.  

```kotlin
// 모든 테스트에 동일한 인스턴스가 사용되므로 동일한 ID가 세 번 인쇄된다.
class SingleInstanceExample : WordSpec({
   val id = UUID.randomUUID()
   "a" should {
      println(id)
      "b" {
         println(id)
      }
      "c" {
         println(id)
      }
   }
})
```

### InstancePerTest
`IsolationMode.InstancePerTest`모드는 내부 컨텍스트를 포함하여 모든 테스트 케이스에 대해 새 Spec이 생성된다.  
즉, 외부 컨텍스트는 Spec의 자체 인스턴스에서 "독립 실행형" 테스트로 실행된다.  
```kotlin
class InstancePerTestExample : WordSpec() {

  override fun isolationMode(): IsolationMode = IsolationMode.InstancePerTest

  init {
    "a" should {
      println("Hello")
      "b" {
        println("From")
      }
      "c" {
        println("Sam")
      }
    }
  }
}
```

출력값은 아래와 같다
```
Hello
Hello
From
Hello
Sam
```
외부 컨텍스트(테스트 "a")가 먼저 실행되고, 그 다음 (테스트 "b")에 대해 다시 실행되고, (테스트 "c")에 대해 다시 실행된다.   
Spec 클래스의 깨끗한 인스턴스에서 매번, 변수를 재사용하고자 할 때 매우 유용하다.

또 다른 예시
```kotlin
class InstancePerTestExample : WordSpec() {

  override fun isolationMode(): IsolationMode = IsolationMode.InstancePerTest

  val counter = AtomicInteger(0)

  init {
    "a" should {
      println("a=" + counter.getAndIncrement())
      "b" {
        println("b=" + counter.getAndIncrement())
      }
      "c" {
        println("c=" + counter.getAndIncrement())
      }
    }
  }
}
```
출력값
```
a=0 a=0 b=1 a=0 c=1
```

### InstancePerLeaf
`IsolationMode.InstancePerLeaf`는 하위 테스트만 새 인스턴스가 생성된다.  
```kotlin
class InstancePerLeafExample : WordSpec() {

  override fun isolationMode(): IsolationMode = IsolationMode.InstancePerLeaf

  init {
    "a" should {
      println("Hello")
      "b" {
        println("From")
      }
      "c" {
        println("Sam")
      }
    }
  }
}
```
"a" 테스트가 먼저 실행되고 동일한 인스턴스에서 "b" 테스트가 실행된다.   
그런 다음 새 사양이 생성되고 테스트 "a"가 다시 실행된 다음 테스트 "c"가 실행된다. 
```
Hello
From
Hello
Sam
```

### Global Isolation Mode
시스템 속성을 통해 전역적으로 격리모드를 설정할 수도 있다.  

```kotlin
class ProjectConfig: AbstractProjectConfig() {
   override val isolationMode = IsolationMode.InstancePerLeaf
}
```

## Assertions
Kotest는 현재 여러모듈에 걸쳐 약 325개의 matcher를 가지고 있다.  
Kotest matcher를는 프레임워크에 구애받지않고 Kotest 프레임워크 또는 다른 프레임워크와 함께 사용할 수 있다.  

### Style
matcher는 두 가지 스타일을 제공한다.  

확장함수 스타일
```kotlin
a.shouldStartWith("foo")
```

중위함수 스타일
```kotlin
a should startWith("foo")
```

### Core Matchers
`kotest-assertions-core`모듈에서 제공하는 matchers  

#### 범용
- `obj.shouldBe(other)` :  주어진 obj와 other가 모두 같다는 범용 assertion
- `expr.shouldBeTrue()` :  표현식이 `true`인지 확인
- `expr.shouldBeFalse()` :  표현식이 `false`인지 확인
- `shouldThrow<T> { block }` : 블록이 `T` `Throwable` 또는 하위 유형을 throw하는지 확인
- `shouldThrowExactly<T> { block }` : 블록이 정확히 예외를 throw하는지 확인
- `shouldThrowAny { block }` : 블록이 모든 유형의 throwable을 throw하는지 확인
- `shouldThrowMessage(message) { block }` : 코드 블록이 주어진 메시지와 함께 모든 throwable을 throw하는지 확인

#### 타입 
- `obj.shouldBeSameInstanceAs(other)` :	개체를 ID로 비교하여 정확히 동일한 참조인지 확인
- `obj.shouldBeTypeOf<T>()`	: 주어진 Type이 정확히 `T` 유형이라고 확인, 하위클래스는 실패한다
- `obj.shouldBeInstanceOf<T>` :	주어진 참조가 `T` 유형 또는 `T`의 하위 클래스임을 확인
- `obj.shouldHaveAnnotation(annotationClass)` :	주어진 인스턴스에에 지정된 유형의 Annotation이 있는지 확인
- `obj.shouldBeNull()` : 지정된 참조가 null임을 확인

#### 비교
- `comp.shouldBeLessThan(other)` : `compareTo`를 사용하여 comp가 other보다 미만인지 확인
- `comp.shouldBeLessThanOrEqualTo(other)` : `compareTo`를 사용하여 comp가 other보다 작거나 같은지 확인
- `comp.shouldBeEqualComparingTo(other)` : `compareTo`를 사용하여 comp가 other보다 같은지 확인
- `comp.shouldBeEqualComparingTo(other, comparator)` : `comparator.compare`를 사용하여 comp와 other이 같은지 확인
- `comp.shouldBeGreaterThan(other)` : `compareTo`를 사용하여 comp가 other보다 큰지 확인
- `comp.shouldBeGreaterThanOrEqualTo(other)` : `compareTo`를 사용하여 comp가 other보다 크거나 같은지 확인

#### Iterator
- `iterator.shouldBeEmpty()` : iterator에 다음값이 없는지 검증
- `iterator.shouldHaveNext()` : iterator에 다음값이 있는지 검증

#### Map
- `map.shouldContain("key", "value")` : map에 "key"에 대해 매핑되는 "value"가 있는지 확인
- `map.shouldContainAll(other)`	: map과 other이 동일한 쌍을 포함하는지 확인
- `map.shouldContainExactly(other)` : map과 other이 정확하게 동일한 쌍을 포함하는지(다른 쌍은 없어야함) 확인	
- `map.shouldContainKey(key)` : map에 "key"의 값이 포함되어있는지 확인
- `map.shouldContainKeys(keys)` : map에 주어진 "keys"에 모든 값이 매핑되어있는지 확인
- `map.shouldContainValue(value)` : map에 주어진 "value"가 하나이상 매핑되어있는지 확인
- `map.shouldContainValues(values)` : map에 주어진 "values"가 모두 포함하고 있는지 확인
- `map.shouldBeEmpty()` : map이 비어있는지 확인

#### String
- `str.shouldBeBlank()` : 문자열이 nullOrEmpty인지 확인
- `str.shouldBeEmpty()`	: 문자열의 길이가 0임을 확인
- `str.shouldBeLowerCase()`	: 문자열이 모두 소문자인지 확인
- `str.shouldBeUpperCase()`	: 문자열이 모두 대문자임을 확인
- `str.shouldContain("substr")`	: 문자열에 지정된 하위 문자열이 포함되어 있는지 확인. 하위 문자열은 문자열과 같을 수 있고, 대 소문자를 구분한다
- `str.shouldContain(regex)` : 문자열에 주어진 정규 표현식이 포함되어 있는지 확인
- `str.shouldContainADigit()` : 문자열에 최소한 하나의 숫자가 포함되어 있는지 확인
- `str.shouldContainIgnoringCase(substring)` : 문자열에 대소문자를 무시하고 하위 문자열이 포함되어 있는지 확인
- `str.shouldContainOnlyDigits()` : 문자열에 숫자만 포함되어 있거나 비어 있는지 확인
- `str.shouldBeInteger([radix])` : 문자열에 정수가 포함되어 있는지 확인하고 반환
- `str.shouldContainOnlyOnce(substring)` : 문자열에 부분 문자열이 정확히 한 번 포함되어 있는지 확인
- `str.shouldEndWith("suffix")`	: 문자열이 지정된 접미사로 끝나는 것을 확인. 접미사는 문자열과 같을 수 있고, 대소문자를 구분한다
- `str.shouldHaveLength(length)` : 문자열이 주어진 길이를 가지고 있는지 확인
`str.shouldHaveLineCount(count)` : 문자열에 주어진 수의 행이 포함되어 있는지 확인. `tr.split("\n").length.shouldBe(n)`
- `str.shouldHaveMaxLength(max)` : 문자열이 주어진 최대 길이보다 길지 않은지 확인
- `str.shouldHaveMinLength(min)` : 문자열이 주어진 최소 길이보다 짧지 않은지 확인
- `str.shouldHaveSameLengthAs(length)` : 문자열의 길이가 다른 문자열과 동일한지 확인
- `str.shouldMatch(regex)` : 문자열이 주어진 정규식과 완전히 일치하는지 확인
- `str.shouldStartWith("prefix")` : 문자열이 주어진 접두사로 시작하는지 확인. 접두사는 문자열과 같을 수 있고, 대소문자를 구분한다
- `str.shouldBeEqualIgnoringCase(other)` : 문자열이 대소문자를 무시하고 다른 문자열과 같은지 확인

#### Integer, Long
`num.shouldBeBetween(x, y)` : x와 y 사이에 있고 x와 y를 모두 포함하는지 확인
`num.shouldBeLessThan(n)` : num이 주어진 값 n보다 작다고 주장
`num.shouldBeLessThanOrEqual(n)`: num이 주어진 값 n보다 작거나 같은지 확인
`num.shouldBeGreaterThan(n)` : num이 주어진 값 n보다 큰지 확인
`num.shouldBeGreaterThanOrEqual(n)` : num이 주어진 값 n보다 크거나 같은지 확인
`num.shouldBeEven()` : num이 짝수임인지 확인
`num.shouldBeOdd()` : num이 홀수임을 확인
`num.shouldBeInRange(range)` : num이 지정된 범위에 포함되어 있는지 확인
`num.shouldBeZero()` : num이 0임을 확인

#### Collection 
- `collection.shouldBeEmpty()` : 컬렉션에 element가 없음을 확인
- `collection.shouldBeUnique()`	: 컬렉션의 모든 element가 중복이 없는지 확인
- `collection.shouldContain(element)` : 컬렉션에 지정된 element가 포함되어 있는지 확인
- `collection.shouldContainAll(e1, e2, ..., en)` : 컬렉션에 순서가 중요하지 않은 나열된 모든 element가 포함되어 있는지 확인
- `collection.shouldContainDuplicates()` : 컬렉션에 하나이상의 중복 element가 포함되어 있는지 확인
- `collection.shouldContainExactly(e1, e2, ..., en)` : 컬렉션이 정확히 주어진 값을 포함하고 다른것은 순서대로 포함하지 않음을 확인
- `collection.shouldContainExactlyInAnyOrder(e1, e2, ..., en)` : 컬렉션이 순서에 상관없이 정확히 주어진 값을 포함하고 다른 것은 포함하지 않음을 확인 
- `collection.shouldContainNull()` : 컬렉션에 하나 이상의 null element가 포함되어 있는지 확인
- `collection.shouldContainOnlyNulls()`	: 컬렉션이 null elements만 포함하거나 비어 있는지 확인
- `collection.shouldHaveSingleElement(element)`	: 컬렉션이 하나의 element만 포함하는지 확인
- `collection.shouldHaveSingleElement { block }` : 컬렉션이 주어진 술어에의해 하나의 element를 포함하는지 확인
- `collection.shouldHaveSize(length)` : 컬렉션이 정확히 주어진 길이인지 확인
- `collection.shouldBeSingleton()` : 컬렉션에 element가 하나만 포함되어 있는지 확인
- `collection.shouldBeSingleton { block }` : 컬렉션이 단 하나의 element인지 확인한 후 이 element를 사용하여 block실행
- `collection.shouldHaveLowerBound(element)` : 주어진 element가 컬렉션의 모든 element보다 작거나 같은지 확인. (Comparable을 구현하는 element에 대해서만 작동)
- `collection.shouldHaveUpperBound(element)` : 주어진 element가 컬렉션의 모든 element보다 크거나 같은지 확인. (Comparable을 구현하는 요소에 대해서만 작동)
- `collection.shouldBeSmallerThan(col)` : 컬렉션이 다른 컬렉션보다 작은지 확인
- `collection.shouldBeLargerThan(col)` : 컬렉션이 다른 컬렉션보다 큰지 확인
- `collection.shouldBeSameSizeAs(col)` : 컬렉션의 크기가 다른 컬렉션과 동일한지 확인
- `collection.shouldHaveAtLeastSize(n)` : 컬렉션의 크기가 n 이상인지 확인
- `collection.shouldHaveAtMostSize(n)` : 컬렉션의 크기가 최대 n인지 확인
- `list.shouldBeSorted()` : 목록이 정렬되었는지 확인
- `list.shouldContainInOrder(other)` : 이 목록에 주어진 목록이 순서대로 포함되어 있는지 확인
- `list.shouldExistInOrder({ element }, ...)` :	이 목록에 술어와 순서대로 일치하는 element가 포함되어 있는지 확인. 다른 element는 술어와 일치하는 element 주변이나, element 사이에 나타날 수 있음
- `list.shouldHaveElementAt(index, element)` : 이 목록이 주어진 index에 주어진 element를 포함하는지 확인
- `list.shouldStartWith(lst)` : 이 목록이 주어진 목록의 element로 순서대로 시작하는지 확인
- `list.shouldEndWith(lst)` : 이 목록이 주어진 목록의 element로 순서대로 끝나는 것을 확인
- `value.shouldBeOneOf(collection)` :특정 인스턴스가 컬렉션에 포함되어 있는지 확인
- `collection.shouldContainAnyOf(collection)` : 컬렉션에 있는 element 중 적어도 하나가 있음을 확인
- `value.shouldBeIn(collection)` : object가 컬렉션에 포함되어 있는지 확인. (참조가 아닌 값으로 확인함)

#### Coroutine Channel
`channel.shouldReceiveWithin(duration)`	: 채널이 기간내에 수신수신하는지 확인
`channel.shouldReceiveNoElementsWithin(duration)` : 채널이 기간 내에 어떤 요소도 수신하지 않는지 확인
`channel.shouldHaveSize(n)`	: 닫기 전에 채널이 정확히 n개의 요소를 수신하는지 확인
`channel.shouldReceiveAtLeast(n)` : 채널이 >= n개의 요소를 수신하는지 확인
`channel.shouldReceiveAtMost(n)` : 채널이 닫기 전에 <=n 요소를 수신하는지 확인
`channel.shouldBeClosed()` : 채널이 닫혀 있는지 확인
`channel.shouldBeOpen()` : 채널이 열려 있는지 확인
`channel.shouldBeEmpty() : `채널이 비어 있음을 확인


### Soft Assertions
때때로 테스트에서 여러 어설션을 수행하고 실패한 모든 어설션을 보고 싶을 수 있다. 
Kotest는 이를 위한 assertSoftly기능을 제공한다.  
```kotlin
assertSoftly {
  foo shouldBe bar
  foo should contain(baz)
}
```
블록 내의 어설션이 실패하면 테스트가 계속 실행되며. 모든 실패는 블록 끝에서 보고된다.  