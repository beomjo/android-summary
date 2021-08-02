# Navigation Component

## Navigation Component란?
Navigating은 다른화면으로 이동한다는것을 뜻한다.  
Android개발에 있어서 필수적인 기본 요소일 것 이다.  
Jetpack과 함께 소개된 Navigation Component는 이러한 App내의 화면이동 흐름을 그래포르 지정하여   
마치 네비게이션 처럼 동작한다.


## Navigation Component 장점
- 일반적인 안드로이드 화면이동에 대한 단순한 설정
    - 하단 탐색등
- 백스택 처리 위임
- 트랜잭션 처리 위임
- 화면간 Type Safe한 Arguments 전달
- Transition animation 처리
- 간단한 딥 링킹
- 모든 Navigation 정보를 모으고, 시각화된 그래프로 볼 수 있음 


## Navigation Component 특징
Navigaion Component의 특징중 하나는 세 자기 중요한 부분이 조화롭게 작동된다는 것
- Navigation graph
- NavHostFragment
- NavController


### Navigation graph
Navigation graph는 새로운 Android resource 유형으로,  
xml 파일 형태로 화면 탐색에 관련된 정보를 포함하며, 중심화 한다.   
![image](https://user-images.githubusercontent.com/39984656/127829853-2d762215-91df-466f-b001-3af352190da3.png)  
그래프내에서 시각적으로 표시되는 화면들은 목적지(Destination)이라 불린다. 
Destination을 클릭하면 딥링크 URL등을 볼 수 있다
그래프내의 화살표는 액션이라 부른다. 앱을 통하여 사용자가 이동할 수 있는 다양한 경로를 나타낸다
그래프 내의 액션을 클릭하면 arguments, transaction, animation, backstack 조정 등 모든 정보를 볼 수 있다.

### NavHostFragment
NavHostFragment는 Fragment를 상속하고 있으며, NavHost 인터페이스를 구현한다.  
NavHostFragment는 보통 FragmentContainerView와 사용하며, 
화면이동이 발생하도록 레이아웃 내에 화면을 보여주는 영역을 제공한다.
NavHostFragment는 개별적으로 NavController를 가진다.
```xml
<androidx.fragment.app.FragmentContainerView
android:layout_width="match_parent"
android:layout_height="match_parent"
android:id="@+id/my_nav_host_fragment"
android:name="androidx.navigation.fragment.NavHostFragment"
app:navGraph="@navigation/nav_sample"
app:defaultNavHost="true" />
```

### NavController
NavController는 화면이 이동되는 Navigation을 처리한다.