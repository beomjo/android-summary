# ViewModel

## Jetpack ViewModel
ViewModel은 Jetpack 구성요소중 하나로, 본래 ViewModel이란 이름은  
소프트웨어 개발 디자인 패턴 중 하나인 MVVM 디자인 패턴으로부터 파생되었다.  
MVVM 패턴 관점에서 부르는 ViewModel과 구분하기위해  
AAC<sup><Android Architecture Component</sup> ViewModel이라고 부르기도한다.  


## 특징
- 라이프 사이클을 고려하여 UI관련된 데이터를 저장 관리하기 위해 설계
- 화면전환과 같이 구성이 변경되는 상황에서도 Data유지
- Activity가 끝날때 사라지지않고, View의 생명주기와 별개로 흘러감
- ViewModel은 Activity생명주기 외부에 존재하므로, Context를 저장한다면 Memory Leak이 발생할 수 있다


## 생명주기
ViewModel 객체의 범위는 ViewModel을 생성할때 ViewModelProvider에 전달되는 Lifecycle로 지정된다.  
ViewModel은 범위가 지정된 Activity에서 Activity가 종료될때까지,  
범위가 지정된 Fragment에서는 Fragment가 분리될 때 까지 메모리에 남아있다.  
![image](https://user-images.githubusercontent.com/39984656/109841004-f81ed280-7c8b-11eb-9ad6-4df9da8734b3.png)  

안드로이드에서 화면이 회전할 경우 onCreate가 여러번 호출될 수 있다.  
보통의 경우 onCreate에서 ViewModel을 생성하기 때문에 화면이 회전하는경우 ViewModel을 다시 생성하여 데이터가 보존되지 않을 수 있다.  


## ViewModel 생성하는 방법

### 1. 파라미터가 없는 ViewModel, Lifecycle Extensions사용
lifecycle-extensions  디펜던시 추가  
```
dependencies {
    // ...
    implementation "androidx.lifecycle:lifecycle-extensions:$lifecycle_version"
}
``` 

```kotlin
lateinit var viewModel: MyViewModel
viewModel = ViewModelProvider(this).get(MyViewModel::class.java)
```  

### 2. 파라미터가 없는 ViewModel, ViewModelProvider.NewInstanceFactory사용
NewInstanceFactory는 안드로이드에서 기본적으로 제공해주는 팩토리 클래스이며, ViewModelProvider.Factory 인터페이스를 구현한다.  
ViewModel 클래스가 파라미터를 필요로 하지 않거나, 특별히 팩토리를 커스텀 할 필요가 없는 상황에서   
lifecycle-extensions 디펜던시를 추가하지 않고 사용할 수 있는 방법이다.  
```kotlin
lateinit var viewModel: MyViewModel
viewModel = ViewModelProvider(this, ViewModelProvider.NewInstanceFactory())
            .get(MyViewModel::class.java)
```

### 3. 파라미터가 없는 ViewModel, ViewModelProvider.Factory 구현
ViewModelProvider.Factory 인터페이스를 직접 구현하는 방법

```kotlin
class ViewModelFactory : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>) =
            with(modelClass) {
                when {
                    isAssignableFrom(MyViewModel::class.java) -> MyViewModel()
                    else ->
                        throw IllegalArgumentException("Unknown ViewModel class: ${modelClass.name}")
                }
            } as T
}
```

### 4. 파라미터가 있는 ViewModel
```kotlin
class ViewModelFactory(private val param: String) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>) =
            with(modelClass) {
                when {
                    isAssignableFrom(MyViewModel::class.java) -> MyViewModel(param)
                    else ->
                        throw IllegalArgumentException("Unknown ViewModel class: ${modelClass.name}")
                }
            } as T
}
```

```kotlin
lateinit var viewModel: MyViewModel

val param = "fff"

viewModel = ViewModelProvider(this, ViewModelFactory(param)).get(MyViewModel::class.java)
```

### 5. 파라미터가 없는 ViewModel, Lifecycle ViewModel KTX 사용
```kotlin
private val viewModel : MyViewModel by viewModels()
```

### 6. 파라미터가 있는 ViewModel, Lifecycle ViewModel KTX 사용
```kotlin
class ViewModelFactory(private val param: String) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>) =
            with(modelClass) {
                when {
                    isAssignableFrom(MyViewModel::class.java) -> MyViewModel(param)
                    else ->
                        throw IllegalArgumentException("Unknown ViewModel class: ${modelClass.name}")
                }
            } as T
}
```

```kotlin
private val param = "fff"
private val viewModel : MyViewModel by viewModels { ViewModelFactory(param)}
```

#### 7. 파라미터가 있는 ViewModel, Dagger 사용
- TBD


## ViewModel이 사라지는 시점
ViewModel은 소유자 Owner의 생명주기가 종료되는 시점에  
리소스 정리를위해 `onCleared()`함수가 호출된다.


## Fragment간의 데이터 공유
하나의 Activity에 두개의 Fragment가 있을 때  
Fragment간의 ViewModel을 공유하는 방법에 대하여 알아보자. 

### 1. Activity에서 ViewModel을 선언하여 Fragment로 넘겨주기
Activity에서 [ViewModel 생성하는 방법](#viewmodel-생성하는-방법)으로 생성한 후 프래그먼트로 넘겨준다.

### 2. Fragment에서 선언하기

#### 2-1 activity를 형변환 하기
activity를 FragmentActivity나 ViewModelStoreOwner로 형변환 하여 넘겨준다.  
둘 다 동일한 표현이다.
```kotlin
lateinit var viewModel: MyViewModel
viewModel = ViewModelProvider(activity as FragmentActivity).get(MyViewModel::class.java)
```
```kotlin
lateinit var viewModel: MyViewModel
viewModel = ViewModelProvider(activity as ViewModelStoreOwner).get(MyViewModel::class.java)
```

#### 2-2 requiredActivity 사용
fragment에 자체적으로 정의되어있는 `requireActivity()`를 사용한다.  
`requireActivity()`가 null 인경우 `IllegalStateException`이 발생하므로 주의 필요  
```kotlin
lateinit var viewModel: MyViewModel
viewModel = ViewModelProvider(requireActivity()).get(MyViewModel::class.java)
```

### 3. Fragment KTX사용
fragment-ktx를 사용하면 `lateinit`나 `lazy` 없이 한 줄로 초기화가 가능하다.  
`activityViewModels()`를 사용한다.  
```
dependency {
    implementation "androidx.core:core-ktx:$ktx_core_version"
}
```

```kotlin
class SharedViewModel : ViewModel() {
    val selected = MutableLiveData<Item>()

    fun select(item: Item) {
        selected.value = item
    }
}

class MasterFragment : Fragment() {

    private lateinit var itemSelector: Selector

    private val model: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        itemSelector.setOnClickListener { item ->
            // Update the UI
        }
    }
}

class DetailFragment : Fragment() {

    private val model: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        model.selected.observe(viewLifecycleOwner, Observer<Item> { item ->
            // Update the UI
        })
    }
}
```

ViewModel에 동일하게 파라미터를 전달하는 경우에는  
Factory를 생성하여 사용한다
```kotlin
private val viewModel: MyViewModel by activityViewModels { ViewModelFactory(param) }
```