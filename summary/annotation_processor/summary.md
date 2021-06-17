# Annotation Processor

## Annotation이란?
Annotation은 자바 소스코드에 추가할 수 있는 메타 데이터의 한 형태이다.  
클래스, 인터페이스, 메소드, 변수, 매개변수 등에 추가할 수 있다.  
Annotation은 소스 파일에서 읽을 수도 있고, 컴파일러에 의해 생성된 클래스 파일에 내장되어 읽힐 수도 있으며,   
Runtime에 Java VM에 의해 유지되어 리플렉션에 의해 읽어낼 수도 있다.  


## Annotation Processor란?
Annotation Processor는 Java 컴파일러의 플러그인의 일종으로 일반적으로 코드베이스를 검사, 수정 또는 생성하는데 사용된다.  


## Annotation Processor 장점
- 모든 처리가 런타임이 아닌 컴파일타임에 발생하여 빠르다
- 리플렉션을 사용하지 않는다
- 보일러플레이트 코드를 생성해준다


## Annotation Processor 동작
1. 자바 컴파일러가 컴파일러를 수행한다
2. 실행되지않은 Annotation Processor를 수행한다
3. 프로세서 내부에서 Annotation이 달린 Element(변수, 메소드, 클래스)등을 처리한다
4. 컴파일러가 모든 Annotation Processor가 실행되었는지 확인하고, 그렇지않다면 반복해서 위 작업을 실행한다

<img src="https://user-images.githubusercontent.com/57612082/121766924-303d8780-cb90-11eb-8b7b-964bbd3bd144.png" width="600" height="300">


## Annotation, Annotation Processor 만들기

### 설정  
1. annotation module 추가하기  
![annotation module](https://user-images.githubusercontent.com/57612082/122064421-1f10a700-ce2c-11eb-87b0-61a94569fa7b.png)   
    
2. annotation_processor module 추가하기  
![annotation processor module](https://user-images.githubusercontent.com/57612082/122064443-22a42e00-ce2c-11eb-83cb-49bd29b92cb8.png)  

3. 의존성 설정  
app/build.gradle  
```
dependencies {
    implementation project(':annotation')
    kapt project(':annotation_processor')
}
```  
  
annotation_processor/build.gradle  
```
dependencies {
    implementation project(':annotation')
}
```  

### Annotation, Annotation Processor 만들기

#### 1. Custom Annotation 선언

```kotlin
@MustBeDocumented
@Target(AnnotationTarget.FUNCTION)
@Retention(AnnotationRetention.BINARY)
annotation class PrintFunction
```
- `annotation class` : 애노테이션 클래스 선언
- `@Target` : 애노테이션 타겟 지정  
    ```java
    public enum class AnnotationTarget {
        CLASS, //class, interface, enum 등에 지정할 때
        ANNOTATION_CLASS, //애노테이션 클래스에 지정할 때
        TYPE_PARAMETER, // Generic type parameter에 지정할 때
        PROPERTY, // 프로퍼티에 지정할 때
        FIELD, // property의 backing field에 지정할 때
        LOCAL_VARIABLE, // 로컬 변수에 애노테이션을 지정할 때
        VALUE_PARAMETER, // 함수 또는 생성자의 값 매개 변수에 지정할 때
        CONSTRUCTOR, // 생성자에 애노테이션을 지정할 때
        FUNCTION, // 함수에 애노테이션을 지정할 때
        PROPERTY_GETTER, // 프로퍼티 게터에 지정할 때
        PROPERTY_SETTER, // 프로퍼티 세터에 지정할 때
        TYPE, 
        EXPRESSION, // 임의의 expression에 지정할 때 (ex, @Suppress(“UNCHECKED_CAST”))
        FILE, //top-level function 이나 property등등에 지정할 때
        @SinceKotlin("1.1")
        TYPEALIAS
    }
    ```
- `@Retention` : 사용자 정의 애노테이션이 저장되는 타입을 내며 3가지 타입이 있다
    - `AnnotationRetention.BINARY` 
        - CompileTime 과 Binary 에도 포함되지만 Reflection 을 통해 접근할 수는 없다
    - `AnnotationRetention.SOURCE`
        -  CompileTime 에만 유용하며 빌드된 Binary 에는 포함되지 않는다. 
        - 개발중에 warning 이 뜨는 걸 보이지 않도록 하는 `@suppress` 와 같이 개발 중에만 유용하다 
        - Binary 에 포함될 필요는 없는 경우에 한다
    - `AnnotationRetention.RUNTIME` 
        - CompileTime과 Binary 에도 포함된다
        - Reflection 을 통해 접근 가능하다
        - `@Retention`을 표시해주지 않을경우, 디폴트로 RUNTIME으로 지정된다
- `@MustBeDocumented`
    - Generated Documentation 에 해당 Annotation 도 포함될 수 있는지를 나타낸다
    - 주로 Library 를 만들때 사용한다  

#### 2. AnnotationProcessor 생성
애노테이션가 달린 메소드의 클래스이름, 메소드이름을 간략하게 출력하는 Annotation Processor 생성
```kotlin
@SupportedSourceVersion(SourceVersion.RELEASE_8)
class PrintFunctionProcessor : AbstractProcessor() {
    override fun init(processingEnv: ProcessingEnvironment?) {
        super.init(processingEnv)
        //프로세싱에 필요한 기본적인 정보들을 processingEnvironment 부터 가져온다
    }

    override fun process(p0: MutableSet<out TypeElement>?, p1: RoundEnvironment?): Boolean {
        val elements = p1?.getElementsAnnotatedWith(PrintFunction::class.java)
        elements?.forEach {
            StringBuilder()
                .append("---------------\n")
                .append("enclosedElements: ${it.enclosedElements}\n")
                .append("enclosingElement: ${it.enclosingElement}\n")
                .append("kind: ${it.kind}\n")
                .append("modifiers: ${it.modifiers}\n")
                .append("simpleName: ${it.simpleName}\n")
                .let(::println)
        }
        return false
    }

    override fun getSupportedAnnotationTypes(): MutableSet<String> {
        return hashSetOf(PrintFunction::class.java.canonicalName) // 어떤 애노테이션을 처리할 지 Set에 추가
    }

    override fun getSupportedSourceVersion(): SourceVersion {
        return super.getSupportedSourceVersion() //지원되는 소스 버전을 리턴
    }
}
```
- init() : 파일을 생성하기 위해 필요한 Filer나 디버깅에 필요한 Messager, 각종 유틸클래스들을 이곳에서 받을 수 있다
- process() : 프로세서의 핵심 으로 이곳에서 클래스, 메소드, 필드 등에 추가한 애노테이션을 처리하고 처리에 대한 결과로 자바 파일을 생성할 수 있다
- getSupportedAnnotationType() : 어떤 애노테이션들을 처리할 지 Set형태로 리턴하게 된다 
- getSupportedSourceVersion() : 일반적으로 최신의 자바 버전을 리턴

#### 3. AnnotationProcessor 등록
![image](https://user-images.githubusercontent.com/57612082/122088144-73258680-ce40-11eb-87f7-792bf2f185d1.png)  
`annotation_processor/src/main/resources/META-INF/services` 위치에 `javax.annotation.processing.Processor`을 생성하고  
파일을 열어 애노테이션 프로세서의 패키지명을 포함하고 있는 CanonicalName을 적는다.  
`com.beomjo.sample.annotation_processor.PrintFunctionProcessor`


## 예제  

### AnnotationRetention.RUNTIME 사용
TBD  

### AnnotationRetention.BINARY 사용
Annotation 선언   
```kotlin
@MustBeDocumented
@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.BINARY)
annotation class GenerateWith
```  

Annotation Processor 구현  
```kotlin
@SupportedSourceVersion(SourceVersion.RELEASE_8)
class ClassGeneratorProcessor : AbstractProcessor() {

    private val kaptKotlinGeneratedOption = "kapt.kotlin.generated"  
    private lateinit var kaptKotlinGenerated: File

    private val sampleAnnotation = GenerateWith::class.java.canonicalName

    override fun getSupportedAnnotationTypes() = setOf(sampleAnnotation)

    override fun init(processingEnv: ProcessingEnvironment) {
        super.init(processingEnv)
        kaptKotlinGenerated = File(processingEnv.options[kaptKotlinGeneratedOption])
    }

    override fun process(annotations: Set<TypeElement>, roundEnv: RoundEnvironment): Boolean {
        val annotation = annotations.firstOrNull { it.toString() == sampleAnnotation } ?: return false
        for (element in roundEnv.getElementsAnnotatedWith(annotation)) {
            val className = element.simpleName.toString()
            val `package` = processingEnv.elementUtils.getPackageOf(element).toString()
            generateClass(className, `package`)
        }
        return true
    }

    private fun generateClass(className: String, `package`: String) {
        val source = generateSourceSource(`package`, className)
        val relativePath = `package`.replace('.', File.separatorChar)
        val folder = File(kaptKotlinGenerated, relativePath).apply { mkdirs() }
        File(folder, "Generated$className.kt").writeText(source)
    }

    private fun generateSourceSource(`package`: String, className: String) =
        """
        package $`package`
        
        class Generated$className {
            fun printInfo() {
                println("The annotated class was $`package`.$className")
            }
        }
        """.trimIndent()
}
```

생성한 Annotation 추가   
```kotlin
@GenerateWith
class SampleClass
```  

`app/build/generated/source/kaptKotlin/....`에 Generated 파일 추가됨  
![image](https://user-images.githubusercontent.com/39984656/122099228-bb4aa600-ce4c-11eb-9364-ea4e58402aad.png)  

하드코딩 하지않고 [kotlin-poet](https://square.github.io/kotlinpoet/) 라이브러리를 사용하여도 좋을것 같다.


### AnnotationRetention.SOURCE 사용
[PrintFunction 어노테이션](#custom-annotation-선언) 참고
