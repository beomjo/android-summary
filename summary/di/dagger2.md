# Dagger 2
Annotation을 이용하여 컴파일 단계에서 의존성을 주입하는 구현체를 생성해주고 실행하는 라이브러리
 
## 장점
- 보일러플레이트 코드를 줄인다
- 리플렉션을 사용하지않고 컴파일 타임에 코드를 생성하여 빠르다
- 재사용성 증가
- 스코프를 이용한 객체 관리


## 핵심 키워드
- Inject
- Component
- Module
- Scope
- SubComponent
- Qualifier
- Named

### @Inject
`@Inject` 으로 의존성을 요청한다.
`@Inject`는 필드. 생성자, 메소드에 붙여 컴포넌트로부터 객체를 주입받을 수 있게 한다.


### @Component
Component는 Module과 Inject 사이의 브릿지 역할을 수행한다.
Component가 Module로 부터 객체를 생성하여 Inject를 요청한 쪽으로 전달한다.
`@Component`는 `interface` 또는 `abstract` 클래스에 붙힐 수 있다.
`@Component` Annotation이 달린 `interface`,`abstract` 클래스에는 적어도 하나의 추상 컴포넌트 메소드가 있어야한다.

#### Component의 메소드
- Provision Method : 파라미터가 없고, injection 시킬 객체를 리턴
- Member Injection Method : 멤버 파라미터로 의존성 주입 시킬 객체를 받음
```
@Component(modules = CoffeeMakerModule.class)
public interface CoffeeComponent {
    //provision method
    CoffeeMaker make();
​
    //member-injection method
    void inject(CoffeeMaker coffeeMaker);
}
```

#### @BindInstance
`@BindsInstance`는 
- Component 빌더내의 메소드에 추가하거나 Component 팩토리내의 파라미터로 추가하여 객체를 Component가 가지고 있는 특정 키에 바인딩한다.
- 빌더나 팩토리에게 파라미터로 넘겨줌으로써 컴포넌트가 이 객체들을 관리하고, 요청시 객체를 넘겨준다
```
@Component.Builder
interface Builder {
    @BindsInstance  
    fun foo(foo:Foo): Builder
    @BindsInstance
    fun bar(@Blue bar:Bar): Builder
    ...
}

// or

@Component.Factory
interface Factory {
    fun newMyComponent(
        @BindsInstance foo:Foo,
        @BindsInstance @Blue bar:Bar
    ): MyComponent
}
```

### @Module
Component에 연결하여 의존성 객체를 생성한다.
- @Binds
    - Module내에서 abstract 메소드 앞에 붙여 바인딩을 위임하는 어노테이션
    - 객체를 생성하는 대신 컴포넌트 내에 있는 객체를 파라미터로 받아 바인딩함으로 좀 더 효울적임
- @Provides
    - 주입될 인스턴스를 직접 생성 하여 주입
    - 프로젝트내에서 코드에 접근 불가능한 서드파티등에 사용
```
@Binds 
abstract fun  bindRandom(secureRandom : SecureRandom): Random
```
```
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

  @Provides
  @Singleton
  fun provideOkHttpClient(): OkHttpClient {
    return OkHttpClient.Builder()
      .addInterceptor(HttpRequestInterceptor())
      .build()
  }

  @Provides
  @Singleton
  fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
    return Retrofit.Builder()
      .client(okHttpClient)
      .baseUrl("https://pokeapi.co/api/v2/")
      .addConverterFactory(MoshiConverterFactory.create())
      .addCallAdapterFactory(CoroutinesResponseCallAdapterFactory())
      .build()
  }
}
```

### @Scope
의존성 관련 객체의 범위를 지정. 
같은 범위에서 주입 요청시 동일한 객체를 반환한다.
```
// 커스텀 스코프 생성 예시
@Scope
@MustBeDocumented
@Retention(value = AnnotationRetention.RUNTIME)
annotation class LoggedUserScope
```

### @SubComponent
Dagger는 SubComponent를 지원한다.
Component와 부모 자식 관계를 맺을 수 있다.

### @Qualifier , @Named
 Component에서 같은 타입을 바인딩하여 리턴하는 경우가 있을시 구분하기위하여 한정자 `@Qualifier`, `@Named`를 사용한다.

 ```
 // @Qualifier를 사용하여 커스텀 한정자 생성, @Named대신 사용한다
 @Qualifier
 @Retention(AnnotationRetention.RUNTIME)
 annotation class Hello
 ```
 ```
 // @Named를 사용하여 같은 리턴타입의 의존성 식별
 @Provides @Named("hello") fun provideHello() = "Hello"
 @Provides @Named("world") fun provideHello() = "World"

 // 의존성 주입을 요청하는 곳에서도 @Named를 사용해야한다
 @Inject @Named("hello") String strHello;
@Inject @Named("world") String strWorld;
 ```


## 의존성을 주입하는 순서 (Flow)
![image](https://user-images.githubusercontent.com/39984656/107518083-f2385300-6bf1-11eb-8041-15ba7ca1dc43.png)

1. @Inject가 선언된 클래스의 생성자, 변수, 메소드에서 주입 요청
2. SubComponent 
3. Module
4. Scope에 있으면 해당 객체 리턴, 없으면 생성
5. SubComponent에서 맞는 타입을 못찾으면 상위 Component로
6. Module
7. Scope 있으면 해당 객체 리턴, 없으면 생성



## 그래프
![image](https://user-images.githubusercontent.com/39984656/107519253-632c3a80-6bf3-11eb-89c4-db75f2e4e971.png)
의존성 주입 요청시, 모듈로부터 생성된 객체를, 컴포넌트를 주입받는다.
위와 같이 모듈과 컴포넌트 그리고 의존성주입을 받는 대상과의 계약관계를 Dagger에서 **그래프**라 한다.
