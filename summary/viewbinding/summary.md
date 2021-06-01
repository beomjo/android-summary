# ViewBinding

## ViewBinding이란?
ViewBinding을 사용하여 뷰와 상호작용하는 코드를 쉽게 작성 가능하다.  
ViewBinding은 XML 레이아웃 파일의 결합 클래스를 생성한다.  
바인딩 클래스의 인스턴스에는 레이아웃에 ID가 있는 모든 뷰의 직접 참조가 포함된다.  
ViewBinding을 사용하여 `findViewById`를 대체할 수 있다.  


## 설정
build.gradle(module)
```
// Android Studio 3.6 이상
android {
        ...
        viewBinding {
            enabled = true
        }
    }
    
```
```
// Android Studio 4.0 이상
android { 
	buildFeatures { 
		viewBinding = true 
	} 
}
```


## findViewById와 차이점
- Null Safety 
    - 뷰 결합은 뷰의 직접 참조를 생성하므로 유효하지 않은 뷰 ID로 인해 null 포인터 예외가 발생할 위험이 없다  
    - 레이아웃의 일부 구성에만 뷰가 있는 경우 결합 클래스에서 참조를 포함하는 필드가 @Nullable로 표시된다
- Type Safe 
    - 각 바인딩 클래스에 있는 필드의 유형이 XML 파일에서 참조하는 뷰와 일치한다(클래스 변환 예외가 발생할 위험이 없음)


## DataBinding과 비교  
ViewBinding과 DataBinding은 모두 뷰를 직접 참조할 수 있는 Binding 클래스를 생성하는 공통점이 있다.  
ViewBinding은 DataBinding과 비교하여 아래와 같은 이점이 있다.  

### DataBinding과 비교하였을 때 ViewBinding의 이점
- 더 빠른 컴파일
    - ViewBinding에는 Annotation Process가 필요하지 않으므로 컴파일 시간이 더 짧다
- 사용 편의성
    -  ViewBinding에는 특별히 태그된 XML 레이아웃 파일이 필요하지 않으므로 앱에서 더 신속하게 채택할 수 있다
    -  모듈에서 뷰 결합을 사용 설정하면 모듈의 모든 레이아웃에 ViewBinding이 자동으로 적용된다


### DataBinding과 비교하였을 때 ViewBinding의 단점
- ViewBinding은 [레이아웃변수 또는 레이아웃 표현식](https://developer.android.com/topic/libraries/data-binding/expressions?hl=ko)을 지원하지 않는다
- ViewBinding은 [양방향 데이터 결합](https://developer.android.com/topic/libraries/data-binding/two-way?hl=ko)을 지원하지 않는다


## 사용
result_profile.xml 이라는 레이아웃 파일을 뷰바인딩으로 설정해보자.  
```xml
<LinearLayout ... >
        <TextView android:id="@+id/name" />
        <ImageView android:cropToPadding="true" />
        <Button android:id="@+id/button"
            android:background="@drawable/rounded_button" />
</LinearLayout>
```

### Activity
```kotlin
private lateinit var binding: ResultProfileBinding

override fun onCreate(savedInstanceState: Bundle) {
    super.onCreate(savedInstanceState)
    binding = ResultProfileBinding.inflate(layoutInflater)
    setContentView(binding.root)
}   
```
위와같이 설정한 후 binding 클래스 인스턴스를 통하여 뷰를 참조할 수 있다.  
```kotlin
binding.name.text = viewModel.name
binding.button.setOnClickListener { viewModel.userClicked() }
```

### Fragment
```kotlin
  private var _binding: ResultProfileBinding? = null
    // onCreateView와 onDestroyView 생명주기 사이에서만 사용할 수 있다
    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        _binding = ResultProfileBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
```

`inflate` 메소드를 사용하려면 layoutInflater를 전달해야한다.
이미 레이아웃이 inflate된 상태라면 binding 클래스의 정적 메소드 `bind()`를 호출하면 된다.
```kotlin
private var fragmentBlankBinding: ResultProfileBinding? = null

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    fragmentBlankBinding = ResultProfileBinding.bind(view)
    fragmentBlankBinding.textViewFragment.text = getString(string.hello_from_vb_bindfragment)
}

override fun onDestroyView() {
    fragmentBlankBinding = null
    super.onDestroyView()
}
```
프래그먼트에서 바인딩 변수는 필요하지 않다면 필드에서 제거해주는것이 좋다.