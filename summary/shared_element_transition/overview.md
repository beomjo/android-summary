# Shared Element Transition

## Shared Element Transition란 ?
머티어리얼 디자인 앱의 Activity, Fragment Transition은 공통 요소 간의 모션 및 변환을 통해 서로 다른 상태 간에 시각적 연결을 제공한다. 
쉽게 이야기하면 Activity나 Fragment에서 쓰였던 뷰를 재활용하여 좀 더 동적인 움직임을 보여주는 방법 중 하나이다.
Material Design 으로 동작하기 때문에 API 21 미만은 사용할 수 없다.


## 예시
![ezgif com-gif-maker](https://user-images.githubusercontent.com/57612082/112417159-53fbe900-8d6a-11eb-87d2-e26058452b5e.gif)
![sample](https://user-images.githubusercontent.com/57612082/112416975-0da68a00-8d6a-11eb-8bab-8520e7bbe2c3.gif)


## Activity
xml로 transition을 설정하는 방법도 있지만
여기선 코드로 설정하는 방법을 설명한다.

#### Activity A 설정
```kotlin
// Activity A 
override fun onCreate(savedInstanceState: Bundle?) {
    window.requestFeature(Window.FEATURE_ACTIVITY_TRANSITIONS)
    setExitSharedElementCallback(MaterialContainerTransformSharedElementCallback())
    window.sharedElementsUseOverlay = false
    super.onCreate(savedInstanceState)
}
```
window의 requestFeature() 함수를 사용하여 transition 사용함을 설정
Activity A에서는 `setExitSharedElementCallback` 즉 돌아 올때 실행될 transition의 callback을 설정한다.

```kotlin
// Activity A
ViewCompat.setTransitionName(someView, "item1")
```
Activity B로 공유할 SharedElement에 transitionName을 설정한다.

```kotlin
const val TRANSITION_NAME = "item1"

context?.let {
    val activity = it as Activity
    val options = ActivityOptions.makeSceneTransitionAnimation(
        activity,
        Pair(startView, TRANSITION_NAME),
    )
    val intent = Intent(context, ActivityB::class.java).
    activity.startActivity(intent, options.toBundle())
}
```
`ActivityOptions.makeSceneTransitionAnimation()` 을 호출하여
sharedElements를 설정한다.
그리고 ActivityB로 이동한다.


#### Activity B
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    window.requestFeature(Window.FEATURE_ACTIVITY_TRANSITIONS)

    rootView.transitionName = transitionName

    setEnterSharedElementCallback(MaterialContainerTransformSharedElementCallback())
    setExitSharedElementCallback(MaterialContainerTransformSharedElementCallback())

    window.sharedElementEnterTransition = getContentTransform(rootView)
    window.sharedElementReturnTransition = getContentTransform(rootView)
        
    super.onCreate(savedInstanceState)
    }
```
Activity B 에서도 window의 `requestFeature()` 함수를 호출하여 transition을 사용함을 설정한다.
또한 Activity B에도 Activity A에서 설정한 것과 동일한 transitionName을 설정한다.
`setEnterSharedElementCallback()`, `setExitSharedElementCallback()`를 호출하여
진입, 되돌아갈때의 sharedElementCallback을 설정한다.

그리고 window의 `sharedElementEnterTransition`, `sharedElementReturnTransition` 속성에 custom Transition을 설정한다.

```kotlin
private fun getContentTransform(rootView: View): MaterialContainerTransform {
    return MaterialContainerTransform().apply {
        addTarget(rootView)
        duration = 400
        pathMotion = MaterialArcMotion()
        isElevationShadowEnabled = true
        startElevation = 9f
        endElevation = 9f
        setAllContainerColors(MaterialColors.getColor(rootView, R.attr.colorSurface))
    }
}
```


## Fragment
Fragment에서 Fragment로 이동할 때도 sharedElementTransition을 설정할 수 있다.