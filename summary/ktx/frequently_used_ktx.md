# 자주 사용하는 KTX

## TextView Extensions
앱에 EditText가 있고 사용자 입력을 통하여 작업을 수행할때
사용하지 않는 interface도 구현을 해주어야 하여 쓸모없는 코드가 늘어난다.
```kotlin
searchEditText.addTextChangedListener(object : TextWatcher {
  override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {}

  override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {}

  override fun afterTextChanged(p0: Editable?) {
    // perform a search
  }
})
```
사용자가 입력을 마친 시점에만 관심이 있을경우 아래와 같이 사용할 수 있다. 
```kotlin
searchEditText.doAfterTextChanged {
  // perform a search
}
```


## SharedPreference Extensions
SharedPreferences에 값을 저장할때는 기존에 다음과 같이 사용한다
```kotlin
sharedPreferences
  .edit()
  .putBoolean("key", value)
  .apply()
```
KTX를 사용하면 다음과같이 간결하게 코드 작성이 가능하다
```kotlin
sharedPreferences.edit { putBoolean("key", value) }
```


## File Extensions
문자열 또는 파일을 Uri로 변환할 수 있는 편리한 Extension을 제공한다.

기존 사용방식
```kotlin
Uri.parse(uriString)
Uri.fromFile(this)
```

Extensions을 사용하면 아래와 같이 사용 가능하다.
```kotlin
uriString.toUri()
file.toUri()
```


## Fragment Extensions
fragment는 다루기 쉽지않으므로 KTX에 포함된 모든 Extensions들은 사용하면 유익하다.

먼저 아래와 같이 의존성을 추가한다.
```
dependencies {
  implementation "androidx.fragment:fragment-ktx:1.2.5"
}
```

fragment에서 ViewModel에 쉽게 Access하는 방법
```kotlin
class MyFragment : Fragment() {
  val viewmodel: MyViewModel by activityViewModels()
}
```

fragmentManager의 `commit`, `add`, `replace` transaction Extensions
```kotlin
supportFragmentManager.commit {
  addToBackStack(BACK_STACK_ROOT)
  add(fragment, fragment::class.java.simpleName)
}
```

### LiveDate Extensions
LiveData는 수명주기를 인식하기 대문에 주로 UI계층에서 사용되는 관창가능한 데이터이다.
LiveData Extensions를 사용하면 Coroutine Flow와 LiveDate를 자유롭게 변환가능하다.

먼저 아래와같이 의존성을 추가한다.
```
dependencies {
  implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.2.0"
}
```

```kotlin
private val _showProgress = MutableStateFlow(false)
val showProgress = _showProgress.asLiveData()
```

### Lifecycle Extensions
Coroutine은 Android 개발에 중요한 부분이 되었다.
일반적으로 Coroutine을 사용하는 시나리오중 하나는 Fragment에서 Coroutine을 시작하는것이며 많은 코드 작성이 필요하다.

```kotlin
class TestFragment : Fragment(R.layout.fragment_test), CoroutineScope {
  private val job = Job()
  override val coroutineContext = job + Dispatchers.Main

  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    launch {
      // do some stuff
    }
  }

  override fun onDestroyView() {
    super.onDestroyView()
    job.cancel() // cancel all the coroutines
  }
}
```
CoroutineScope를 구현하고 Context를 제공하여야하며, 필요하지않은때 Coroutine을 지워야한다.
다행이 Lifecycle Extensions를 통해 lifecycleOwner 범위에 연결된 CoroutineScope 인스턴스를 제공하는 lifecycleScope가 있다.

사용하려면 아래와 같이 의존성을 추가한다
```
dependencies {
  implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0"
}
```

이제 viewLifecycleOwner의 lifecycleScope를 사용하고, Fragment가 파괴되면 Coroutine이 자동으로 취소된다.
```kotlin
class TestFragment : Fragment(R.layout.fragment_test) {
  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    viewLifecycleOwner.lifecycleScope.launch {
      // do some stuff
    }
  }
}
```
그 외에도 scope lifecycle에 따라 Coroutine을 실행할 시기를 관리할 수 있다.
```kotlin
class TestFragment : Fragment(R.layout.fragment_test) {
  ...

  fun testLifecycleScope() {
    viewLifecycleOwner.lifecycleScope.launch {
      whenResumed {
        // do some stuff only when the scope is in the Resumed state
      }
    }
  }
}
```
`whenResumed()` 이외에도 `whenCreated()`, `whenStarted()`, `whenStateAtLeast()`등이 있다.


## ViewModel Lifecycle Extensions
아래와 같이 의존성을 추가하면 lifecycleScope와 유사하게 viewModelScope를 사용할 수 있다.
```
dependencies {
  implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0"
}
```

viewModelScope는 각 ViewModel에 대해 정의되며 ViewModel이 제거될때 Coroutine이 자동으로 취소된다.

```kotin
class MyViewModel: ViewModel() {
  init {
    viewModelScope.launch {
      // Coroutine that will be canceled when the ViewModel is cleared.
    }
  }
}
```