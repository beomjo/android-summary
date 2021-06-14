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

![image](https://user-images.githubusercontent.com/57612082/121766924-303d8780-cb90-11eb-8b7b-964bbd3bd144.png)


## Annotation, Annotation Processor 만들기

### 설정  
1. annotation module 추가하기
<img width="746" alt="스크린샷 2021-06-13 오후 8 37 36" src="https://user-images.githubusercontent.com/57612082/121805781-9953f680-cc87-11eb-9c13-028c24a7fd65.png">  
  
  
2. annotation_processor module 추가하기  
<img width="745" alt="스크린샷 2021-06-13 오후 8 38 03" src="https://user-images.githubusercontent.com/57612082/121805783-9a852380-cc87-11eb-91d2-04a40fe1547e.png">  


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

### Custom Annotation 선언

