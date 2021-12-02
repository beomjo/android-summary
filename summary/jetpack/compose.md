# Compose

## Compose란?
Jetpack Compose는 네이티브 UI를 빌드하기 위한 Android의 최신 도구 키트이다.   
Jetpack Compose는 Android에서 UI 개발을 간소화하고 가속화한다.   
적은 수의 코드, 강력한 도구 및 직관적인 Kotlin API를 사용하여 앱을 빠르고 생동감 있게 구현가능하다.  


## 왜 안드로이드에는 새로운 UI 툴킷이 필요했는가?
- 기존 안드로이드 UI툴킷(View)은 10년도 더 전에 설계된 것이기때문에 한정된 하드웨어에 맞춰 설계되어있음
- 기존 안드로이드 View의 한계를 넘어설려면 플랫폼과의 강한 결합을 끊어야만 했음
- 기존과 같이 compat( 호환성 라이브러리)를 계속 제공하는 것보다는 완전히 분리된 플랫폼을 제공하기 위함
- 개발 언어의 변경 (Compose는 Java가 아닌 Kotlin을 개발언어로 사용)
    

## Compose 특징
- Declarative UI (선언적인 UI)
- 기존에 있던 View와 상호 운용 가능
- 라이프사이클안에서 어떻게 동작하는가에대해서(타이밍에대해서) Compose 를 사용하게되면 고민할 필요가 없음
- 코드감소, 빠른개발과정, 강력한 성능


## Composable 함수
- `@Composable` 어노테이션으로 지정한다
- Composable 함수는 매개변수를 받을 수 있으며, 이 매개변수를 통해 앱 로직이 UI를 형성할 수 있다
- Composable 함수는 아무것도 반환하지 않는다
- 함수는 멱등원이며 부작용이 없다
- 함수는 동일한 인수로 여러번 호출될 때 동일한 방식으로 작동하며, 함수는 속성 또는 전역변수 수정과 같은 부작용 없이 UI를 형성한다

## ReComposition
명령형방식에서는 위젯을 변경하려면 위젯의 setter를 호출하여 내부 상태를 변경한다.  
Compose에서는 새 데이터를 사용하여 Composable함수를 호출한다. 이렇게 하면 함수가 ReComposition 되며 필요한 경우 함수에서 내보낸 위젯이 새데이터로 다시 그려진다.   
Compose Framework는 변경된 구성요소들만 지능적으로 재구성 할 수 있다.  


## Compose Layout



## Compose Architecture



## Compose Animation



## Compose Navigation





