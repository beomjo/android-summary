# Shared Element Transition

## Shared Element Transition란 ?
머티어리얼 디자인 앱의 Activity, Fragment Transition은 공통 요소 간의 모션 및 변환을 통해 서로 다른 상태 간에 시각적 연결을 제공한다. 
쉽게 이야기하면 Activity나 Fragment에서 쓰였던 뷰를 재활용하여 좀 더 동적인 움직임을 보여주는 방법 중 하나이다.
Material Design 으로 동작하기 때문에 API 21 미만은 사용할 수 없다.


## 예시
![ezgif com-gif-maker](https://user-images.githubusercontent.com/57612082/112417159-53fbe900-8d6a-11eb-87d2-e26058452b5e.gif)
![shared-element-transition](https://user-images.githubusercontent.com/39984656/112491754-f4cac280-8dc3-11eb-8e6d-f08246ecbe1f.gif)
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
    ViewCompat.setTransitionName(someView, "item1")
}
```
window의 requestFeature() 함수를 사용하여 transition 사용함을 설정
Activity A에서는 `setExitSharedElementCallback` 즉 돌아 올때 실행될 transition의 callback을 설정한다.
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

#### Activity B 설정
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
Fragment에서도 Activity와 동일하게 먼저 각 공유할 View에 고유 한 전환 이름을 지정하여보기가 한 Fragment에서 다음 Fragment로 매핑 될 수 있도록해야한다.

#### Fragment A 설정
```kotlin
class FragmentA : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        ...
        val itemImageView = view.findViewById<ImageView>(R.id.item_image)
        ViewCompat.setTransitionName(itemImageView, “item_image”)
    }
}
```

```kotlin
parentFragmentManager.beginTransaction()
        .setReorderingAllowed(true)
        .addSharedElement(itemImageView, "item_image")
        .replace(
            R.id.fragment_container_layout,
            FragmentB.newInstance(item)
        )
        .addToBackStack(null)
        .commit()
```
transaction을 호출할때 `addSharedElement()`로 SharedElement로 사용할 view와 transitionName을 전달한다.
`setReorderingAllowed`을 지정하는것은 transition을 연기하기 위함이다. 
아래의 `postponeEnterTransition()`, `startPostponedEnterTransition()`을 사용하기 위해 설정해야 한다. 
[[Transition 연기]](https://developer.android.com/guide/fragments/animate#postpone)

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    postponeEnterTransition()
    view.doOnPreDraw { startPostponedEnterTransition() }
}
```
그리고 onCreateView에
`postponeEnterTransition()`와 `view.doOnPreDraw { startPostponedEnterTransition() }`를 설정한다
이는 exitTransition을 위한 지연과, view가 onPreDraw일때 exitTransition을 실행하기 위함이다.


#### Fragment B 설정
```kotlin
class FragmentB : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        ...
        val heroImageView = view.findViewById<ImageView>(R.id.hero_image)
        ViewCompat.setTransitionName(heroImageView, "item_image")
        sharedElementEnterTransition = getMaterialTransitionSet()
        sharedElementReturnTransition = getMaterialTransitionSet()
    }
}
```
Fragment B에서도 Fragment A에서 설정한 것과 동일한 transitionName을 설정한다
그리고 `sharedElementEnterTransition`, `sharedElementReturnTransition` 속성에 custom Transition을 설정한다.

```kotlin
private fun getMaterialTransitionSet(): TransitionSet {
    return TransitionSet().apply {
        addTransition(ChangeImageTransform())
        addTransition(ChangeBounds())
        addTransition(ChangeTransform())
        addTransition(getContentTransform())
    }
}

private fun getContentTransform(): MaterialContainerTransform {
    return MaterialContainerTransform().apply {
        duration = 400
        pathMotion = MaterialArcMotion()
        isElevationShadowEnabled = true
        startElevation = 9f
        endElevation = 9f
        scrimColor = Color.TRANSPARENT
    }
}
```

```
changeBounds - 타겟 보기의 레이아웃 경계에서 변경사항을 애니메이션으로 보여줌
changeClipBounds - 타겟 보기의 경사 제한 경계에 있는 변경사항을 애니메이션으로 보여줌
changeTransform - 타겟 보기의 배율 및 회전 변경사항을 애니메이션으로 보여줌
changeImageTransform - 타겟 이미지의 크기 및 배율 변경사항을 애니메이션으로 보여줌
```
TransitionSet에 위와 같은 SharedElementTransition을 적용할 수 있고
Custom MaterialContainerTransform도 지정할 수 있다.