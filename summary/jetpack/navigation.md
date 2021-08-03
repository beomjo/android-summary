# Navigation Component

## Navigation Component란?
Navigating은 다른 화면으로 이동한다는 것을 뜻한다.  
Android 개발에 있어서 필수적인 기본 요소일 것이다.  
Jetpack과 함께 소개된 Navigation Component는 이러한 App 내의 화면이동 흐름을 그래프로 지정하여   
마치 네비게이션 처럼 동작한다.


## Navigation Component 장점
- 일반적인 안드로이드 화면이동에 대한 단순한 설정
    - 하단 탐색 등
- 백 스택 처리 위임
- 트랜잭션 처리 위임
- 화면 간 Type Safe 한 Arguments 전달
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
xml 파일 형태로 화면 탐색에 관련된 정보를 포함하며, 중심화  한다.   

![image](https://user-images.githubusercontent.com/39984656/127829853-2d762215-91df-466f-b001-3af352190da3.png)  
그래프내에서 시각적으로 표시되는 화면들은 목적지(Destination)이라 불린다. 
Destination을 클릭하면 딥링크 URL 등을 볼 수 있다.  
그래프내의 화살표는 액션이라 부른다. 앱을 통하여 사용자가 이동할 수 있는 다양한 경로를 나타낸다.  
그래프 내의 액션을 클릭하면 arguments, transaction, animation, backstack 조정 등 모든 정보를 볼 수 있다.

```xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:app="http://schemas.android.com/apk/res-auto"
            xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@+id/home_dest">

    <!-- ...tags for fragments and activities here -->

</navigation>
```  
- `<navigation>`은 모든 탐색 그래프의 루트 노드이다
- `<navigation>`에는 `<activity>` 또는 `<fragment>` 요소로 표시된 대상이 하나 이상 포함된다
- `app:startDestination`은 사용자가 앱을 처음 열 때 기본적으로 실행되는 대상을 지정하는 속성이다  

`<navigation>` 안의 `<fragment>` 요소에 대하여 살펴보자
```xml
<fragment
    android:id="@+id/flow_step_one_dest"
    android:name="com.sample.navigation.FlowStepFragment"
    tools:layout="@layout/flow_step_one_fragment">
    <argument
        .../>

    <action
        android:id="@+id/next_action"
        app:destination="@+id/flow_step_two_dest">
    </action>
</fragment>
```
- `android:id`는 이 XML 및 코드의 다른 위치에서 대상을 참조하는 데 사용할 수 있는 프래그먼트의 ID를 정의한다
- `android:name`은 대상으로 이동할 때 인스턴스화할 프래그먼트의 정규화된 클래스 이름을 선언한다
- `tools:layout`은 그래픽 편집기에서 표시해야 하는 레이아웃을 지정한다

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
NavController는 Graph내에 정의된 정보를 바탕으로
Kotlin코드 내에서 화면 이동을 처리한다.
```kotlin
findNavController().navigate(R.id.win_action)
```


## Navigation UI
- Options Menus
- Bottom Navigation
- Navigation View
- Navigation Drawer
- ActionBar
- Toolbar
- Collapsing ToolBar


## SafeArgs
Navigation Component에는 대상 및 작업에 대해 지정된 인수에 대한 유형 안전 액세스를 위한 간단한 개체 및 빌더 클래스를 생성하는 safe args 라는 Gradle 플러그인이 있다.  

**build.gradle(:project)**  
``` 
dependencies {
        classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$navigationVersion"
    //...
    }
```


**build.gradle(:app)**  
```
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'androidx.navigation.safeargs.kotlin'

android { 
   //...
}
```

NavGraph에서 argument 설정하기  
```xml
<fragment
    android:id="@+id/flow_step_one_dest"
    android:name="com.sample.navigation.FlowStepFragment"
    tools:layout="@layout/flow_step_one_fragment">
    <argument
        android:name="flowStepNumber"
        app:argType="integer"
        android:defaultValue="1"/>

    <action...>
    </action>
</fragment>
```
**/app/build/generated/debug/com/sample/navigation/** 경로에 Args 코드가 생성된다.  
![image](https://user-images.githubusercontent.com/39984656/127954371-1e25cf94-186c-4cd9-b8c1-1155972c8eaf.png)

FlowStepFragment 화면에서 아래와 같이 넘겨받은 args를 사용할 수 있다.
```kotlin
val safeArgs: FlowStepFragmentArgs by navArgs()
```

FlowStepFragment화면으로 args를 넘길 때는 아래와 같이 사용한다.  
```kotlin
view.findViewById<Button>(R.id.navigate_action_button)?.setOnClickListener {
    val flowStepNumberArg = 1
    val action = HomeFragmentDirections.nextAction(flowStepNumberArg)
    findNavController().navigate(action)
}
```


## Destination으로 Deeplinking
Navigation Component를 사용하여 딥 링크를 처리하는 것의 한 가지 이점은 사용자가 앱 위젯, 알림 또는 웹 링크와 같은 다른 진입점에서 적절한 백 스택을 사용하여 올바른 대상에서 시작할 수 있다는 것이다.  

### 위젯에서 딥링크를 통해 Destination으로 이동
**DeepLinkAppWidgetProvider.kt**
```kotlin
class DeepLinkAppWidgetProvider : AppWidgetProvider() {
    override fun onUpdate(
        context: Context,
        appWidgetManager: AppWidgetManager,
        appWidgetIds: IntArray
    ) {
        val remoteViews = RemoteViews(
            context.packageName,
            R.layout.deep_link_appwidget
        )

        val args = Bundle()
        args.putString("myarg", "From Widget")
        val pendingIntent = NavDeepLinkBuilder(context)
                .setGraph(R.navigation.mobile_navigation)
                .setDestination(R.id.deeplink_dest)
                .setArguments(args)
                .createPendingIntent()

        remoteViews.setOnClickPendingIntent(R.id.deep_link_button, pendingIntent)
        appWidgetManager.updateAppWidget(appWidgetIds, remoteViews)
    }
}
```
- `setGraph` 탐색 그래프를 포함한다
- `setDestination` 링크가 어디로 이동하는지 지정한다
- `setArguments` 딥 링크에 전달하려는 모든 인수를 포함한다

**AndroidManifest.xml**
```xml
<receiver android:name=".DeepLinkAppWidgetProvider">
            <intent-filter>
                <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
            </intent-filter>

            <meta-data
                android:name="android.appwidget.provider"
                android:resource="@xml/deep_link_appwidget_info" />
        </receiver>
```

**nav_graph.xml**
```xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    app:startDestination="@+id/home_dest">
    
    ...
```
딥 링크를 통해 진입한경우 Desination으로 `app:startDestination` 속성을 사용하여 백스택을 통해 돌아갈 화면을 지정할 수 있다.


## WebLink 
딥 링크의 가장 일반적인 용도 중 하나는 웹 링크가 앱에서 활동을 열 수 있도록 하는 것이다.  
전통적으로 안드로이드는 manifest파일에 `<intent-filter>`을 사용 하고 URL을 열려는 활동과 연결하였다.  
Navigation Component를 사용하면 이 작업이 매우 간단해지고 탐색 그래프의 대상에 URL을 직접 매핑할 수 있다.  

직접적인 URL 일치 외에도 아래와 같은 기능이 제공된다.  
- scheme이 없는 URI는 http 및 https로 간주된다 
    - 예를 들어, `www.example.com`은 `http://www.example.com` , `https://www.example.com` 와 일치한다
-  형식의 자리 표시자를 사용 {placeholder_name}하여 하나 이상의 문자를 일치 시킬 수 있다
    - 예를 들어, `http://www.example.com/users/4`는 `http://www.example.com/users/{id}`
- `.*` 와일드카드를 사용하여 0개 이상의 문자를 일치 시킬 수 있다
- NavController는 자동으로 ACTION_VIEW intent를 처리 하고 일치하는 딥 링크를 찾는다

### <deeplink> 를 사용하여 graph에 URI기반 딥 링크 추가하기
nav_graph.xml의 이동할 대상화면에 아래와같이 `<deeplink>`를 추가한다.  
```xml
<fragment
    android:id="@+id/deeplink_dest"
    android:name="com.sample.navigation.DeepLinkFragment"
    android:label="@string/deeplink"
    tools:layout="@layout/deeplink_fragment">

    <argument
        android:name="myarg"
        android:defaultValue="Android!"/>

    <deepLink app:uri="www.example.com/{myarg}" />
</fragment>
```

**AndroidManifest.xml**
`<nav-graph>`를 manifest에 추가해준다.
manifest에 선언된 `<nav-graph>`는 Graph에 작성된 내용을 기반으로 intent-filter, action, category, data 등을 자동으로 생성한다.
```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>

    <nav-graph android:value="@navigation/nav_graph" />
</activity>
```

생성되는 내용은 아래와 같다
```xml
 <activity
            android:name="com.example.android.codelabs.navigation.MainActivity">

            <intent-filter>

                <action
                    android:name="android.intent.action.MAIN" />

                <category
                    android:name="android.intent.category.DEFAULT" />

                <category
                    android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

            <intent-filter>

                <action
                    android:name="android.intent.action.VIEW" />

                <category
                    android:name="android.intent.category.DEFAULT" />

                <category
                    android:name="android.intent.category.BROWSABLE" />

                <data
                    android:scheme="http" />

                <data
                    android:scheme="https" />

                <data
                    android:host="www.example.com" />

                <data
                    android:pathPrefix="/" />
            </intent-filter>
        </activity>
```


## 참고
- https://developer.android.com/codelabs/android-navigation#0