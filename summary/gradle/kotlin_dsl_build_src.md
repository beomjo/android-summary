# Kotlin Dsl + buildSrc 으로 의존성 관리

## Kotlin Dsl이란?
DSL이란 <sup>Domain Specific Language</sup>의 약어로 특정 분야에 최적화된 프로그래밍 언어를 뜻한다.   
상용구 코드를 최소화 하기 위해 명령형 코드 대신 선언적 코드 형식을 따른다.  
Kotlin DSL은 코틀린의 언어적인 특징으로 가독성이 좋고 간략한 코드를 사용하여 Gradle 스크립팅을 하는 것을 목적으로 하는 DSL이다.  

## Kotlin Dsl 장, 단점
### 장점
- IDE 지원 향상된 편집환경
    - Code highlighting
    - 자동완성 지원
    - 코드 탐색
    - 오류 코드 강조
    - 변수 리펙토링 가능
- 익숙한 Kotlin언어 사용
    - 러닝 커브 적음
- 멀티 모듈사용시 중복 의존성 선언 필요없어짐

### 단점 
- 빌드 캐시가 Invalidation 되거나 클린 빌드시에 Groovy DSL보다 느림
- 새로운 라이브러리 버전 Inspection 기능 미지원
- Java8이상에서 동작


## Kotlin Dsl 사용
기존 *.gradle 파일을 *.gradle.kts 로 변경한다.
```
ex) build.gradle -> build.gradle.kts
```  

아래 링크를 참고하여 kts파일로 변환하자    
- [KTS로 이전하기](https://developer.android.com/studio/build/migrate-to-kts)  
- [Google 도움말 레시피](https://developer.android.com/studio/build/gradle-tips)
- [Gradle Kotlin Dsl 입문서](https://docs.gradle.org/current/userguide/kotlin_dsl.html)

## buildSrc
의존성 관리와 IDE 자동완성 지원을 위해 kotlin 코드를 가지는 buildSrc 모듈을 만들 수 있다. 

Gradle 문서를 보면  
> Gradle이 수행되면 buildSrc 디렉토리가 존재하는지 체크한다.   
> 이 경우 Gradle은 자동적으로 코드를 컴파일하고 테스트한 뒤 당신의 빌드 스크립트의 classpath에 넣는다.
> 이 방법은 유지 보수, 리팩토링 및 코드 테스트가 더 쉽다.


 


  