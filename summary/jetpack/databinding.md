# DataBinding


## 목차
- [DataBinding](#databinding)
  - [목차](#목차)
  - [DataBinding이란 ?](#databinding이란)
  - [설정](#설정)
  - [레이아웃 선언](#레이아웃-선언)
  - [바인딩하기](#바인딩하기)
    - [즉시 바인딩하기](#즉시-바인딩하기)
  - [표현식 언어](#표현식-언어)
    - [xml 레이아웃 표현식언어](#xml-레이아웃-표현식언어)
    - [사용할 수 없는 연산자](#사용할-수-없는-연산자)
    - [Null 병합 연산자](#null-병합-연산자)
    - [null 포인터 예외 방지](#null-포인터-예외-방지)
    - [컬렉션](#컬렉션)
    - [문자열 리터럴](#문자열-리터럴)
    - [리소스](#리소스)
    - [리스너 바인딩](#리스너-바인딩)
    - [메소드 참조](#메소드-참조)
    - [import](#import)
      - [alias](#alias)
      - [다른 클래스 import](#다른-클래스-import)
    - [include](#include)
  - [BindingAdapter](#bindingadapter)
  - [Observable data object](#observable-data-object)
    - [ObservableField](#observablefield)
    - [ObservableCollection](#observablecollection)
    - [ObservableObject](#observableobject)
    - [LiveData](#livedata)
      - [LiveCyclerOwner 등록하기](#livecyclerowner-등록하기)
      - [LiveData 바인딩](#livedata-바인딩)
  - [단방향, 양방향 바인딩](#단방향-양방향-바인딩)
    - [커스텀 속성을 통한 양방향 바인딩](#커스텀-속성을-통한-양방향-바인딩)
  - [리싸이클러뷰 바인딩](#리싸이클러뷰-바인딩)


## DataBinding이란 ?
xml 에 데이터를 바인딩하여 불필요한 코드를 줄이는 방법으로, 보통 MVVM 패턴을 구현 할 때 사용된다.  
즉, 데이터바인딩은 애플리케이션 로직과 레이아웃을 binding하는 데 필요한 불필요한 코드를 최소화한다.  


## 설정
```
// Android Studio 4.0 
android { 
    buildFeatures { 
        dataBinding = true 
    } 
}
```


## 레이아웃 선언
표현식 언어로 레이아웃의 뷰와 변수를 연결하는 표현식을 작성할 수 있다.  
DataBinding Library는 레이아웃의 뷰를 데이터 개체와 결합하는 데 필요한 클래스를 자동으로 생성한다.   
DataBinding Library는 import, variable 및 includes과 같이 레이아웃에서 사용할 수 있는 기능을 제공한다.  

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:app="http://schemas.android.com/apk/res-auto">
        <data>
            <variable
                name="viewmodel"
                type="com.myapp.data.ViewModel" />
        </data>
        <ConstraintLayout... /> <!-- UI layout's root element -->
</layout>
```

레이아웃 내의 표현식은 `'@{}'` 구문을 사용하여 특성 속성에 작성된다.
```xml
<TextView 
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{viewmodel.getName()}" />
```


## 바인딩하기
layout 태그로 xml을 감싸면 각 레이아웃의 Binding Class가 생성된다.  
기본적으로 클래스 이름은 레이아웃 파일 이름을 기반으로 하여 파스칼 표기법으로 변환하고 Binding 접미사를 추가한다.  

```
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding: ActivityMainBinding = DataBindingUtil.setContentView(
            this, R.layout.activity_main)
    // 위의 레이아웃 파일 이름은 activity_main.xml이므로 생성되는 클래스는 ActivityMainBinding이다

    binding.viewmodel = MainViewModel()
}
```

LayoutInflator를 사용할 수도 있다.  
```kotlin
val binding: ActivityMainBinding = ActivityMainBinding.inflate(getLayoutInflater())
```

또한 `Fragment`, `RecyclerView` 어댑터 내에서 데이터 결합 항목을 사용하고 있다면 다음 코드 예에서와 같이 결합 클래스 또는 BindingClass의 타입을 미리 알 수 없는경우 `DataBindingUtil` 클래스의 `inflate()` 메서드를 사용할 수도 있다.  
```kotlin
val listItemBinding = ListItemBinding.inflate(layoutInflater, viewGroup, false)
// or
val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)
```

### 즉시 바인딩하기
구체적인 결합 클래스를 알 수 없는 때도 있다.   
예를 들어 임의의 레이아웃에 작동하는 RecyclerView.Adapter는 특정 결합 클래스를 인식하지 못한다.   
따라서 이 어댑터는 `onBindViewHolder()` 메서드를 호출하는 동안에도 계속해서 결합 값을 할당해야 한다.  
```kotlin
override fun onBindViewHolder(holder: BindingHolder, position: Int) {
    item: T = items.get(position)
    holder.binding.setVariable(BR.item, item);
    holder.binding.executePendingBindings();
}
```

## 표현식 언어
### xml 레이아웃 표현식언어
- 산술 `+ - / * %`
- 문자열 연결 `+`
- 논리 `&& ||`
- 바이너리 `& | ^`
- 단항 `+ - ! ~`
- 전환 `>> >>> <<`
- 비교 `== > < >= <=(<는 &lt;으로 이스케이프 처리해야 함)`
- `instanceof` 
- 리터럴 - 문자, 문자열, 숫자, `null`
- 변환
- 메서드 호출
- 필드 액세스
- 배열 액세스 `[]`
- 삼항 연산자 `?:`

```xml
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age > 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```

### 사용할 수 없는 연산자
- `this`
- `super`
- `new`
- 명시적 제네릭(Generic) 호출

### Null 병합 연산자
null 병합 연산자(`??`)는 왼쪽 피연산자가 `null`이 아니면 왼쪽 피연산자를 선택하고 `null`이면 오른쪽 피연산자를 선택한다.  
```xml
android:text="@{user.displayName ?? user.lastName}"
```

### null 포인터 예외 방지
생성된 데이터 결합 코드는 자동으로 `null` 값을 확인하고 `null` 포인터 예외를 방지한다.  
 예를 들어 `@{user.name}`표현식에서 `user`가 null이면 `user.name`에 null이 기본값으로 할당됩니다. age의 유형이 `int`인 `user.age`를 참조하면 데이터 결합은 0의 기본값을 사용한다. 

### 컬렉션
`Array`, `List`, `Map`과 같은 일반 컬렉션에는 편의상 `[]` 연산자를 사용하여 액세스할 수 있다.  
```
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String>"/>
    <variable name="sparse" type="SparseArray&lt;String>"/>
    <variable name="map" type="Map&lt;String, String>"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
    </data>
    …
    android:text="@{list[index]}"
    …
    android:text="@{sparse[index]}"
    …
    android:text="@{map[key]}"
```

### 문자열 리터럴
다음 예에서와 같이 작은따옴표를 사용하여 속성 값을 묶어 표현식에 큰따옴표를 사용할 수 있다.  
```xml
android:text='@{map["firstName"]}'
```
또한 큰따옴표를 사용하여 속성 값을 묶을 수도 있다.
```xml
android:text="@{map[`firstName`]}"
```

### 리소스
표현식은 다음 구문을 사용하여 앱 리소스를 참조할 수 있다.
```xml
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```

복수형 매개변수도 전달하려 리소스를 참조 가능하다.
```xml
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
```

### 리스너 바인딩
리스너도 바인딩 가능하다.
이벤트 속성 이름은 몇 가지 예외를 제외하고 리스너 메서드의 이름에 따라 결정된다.  
setOnXXXListener에서 set을 빼고 O를 소문자 o로 바꿔쓴다.  
```xml
<!-- setOnNavigationSelectListener-->
app:NavigationItemSelectedListener="@{viewModel::NavigationItemSelectedListener}"

<!-- onRangeSelectListener -->
app:onRangeSelectedListener="@{calendarDialogViewModel::onRangeSelectedListener}"


<!-- setOnRefreshListener -->
 app:onRefreshListener="@{() -> homeVM.onRefresh()}"
```

### 메소드 참조
```kotlin
class MyHandlers {
    fun onClickFriend(view: View) { ... }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
       <data>
           <variable name="handlers" type="com.example.MyHandlers"/>
           <variable name="user" type="com.example.User"/>
       </data>
       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.firstName}"
               android:onClick="@{handlers::onClickFriend}"/>
       </LinearLayout>
    </layout>
```

매개변수도 전달 가능하다
```kotlin
class Presenter {
    fun onSaveClick(task: Task){}
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="task" type="com.android.example.Task" />
        <variable name="presenter" type="com.android.example.Presenter" />
    </data>

    <LinearLayout 
        android:layout_width="match_parent" 
        android:layout_height="match_parent">
            <Button 
                android:layout_width="wrap_content" 
                android:layout_height="wrap_content"
                android:onClick="@{() -> presenter.onSaveClick(task)}" />
        </LinearLayout>
</layout>
```
다음과 같이 둘 이상의 매개변수와 함께 람다 표현식을 사용할 수 있다.
```kotlin
class Presenter {
    fun onCompletedChanged(task: Task, completed: Boolean){}
}
```

```xml
<CheckBox 
    android:layout_width="wrap_content" 
    android:layout_height="wrap_content"
    android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)"/>
```

### import
import를 사용하면 레이아웃 파일 내에서 클래스를 쉽게 참조할 수 있다.
```xml
<data>
    <import type="android.view.View"/>
</data>

...

<TextView
       android:text="@{user.lastName}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```

#### alias
클래스 이름 충돌이 발생하면 클래스 중 하나의 이름을 별칭으로 바꿀 수 있다.  
다음 예는 `com.example.real.estate` 패키지의 View 클래스 이름을 Vista로 바꾼다.  
```xml
<import type="android.view.View"/>
<import type="com.example.real.estate.View" alias="Vista"/>
```

#### 다른 클래스 import
```xml
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List&lt;User>"/>
</data>
```

또한 가져온 유형을 사용하여 표현식의 일부를 변환할 수도 있다. 다음 예는 connection 속성을 User 유형으로 변환  
```xml
<TextView
    android:text="@{((User)(user.connection)).lastName}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

표현식에서 정적 필드 및 메서드를 참조할 때 가져온 유형을 사용할 수도 있다.   
다음 코드는 `MyStringUtils` 클래스를 가져와서 `capitalize` 메서드를 참조한다.  
```xml
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
    …
    <TextView
       android:text="@{MyStringUtils.capitalize(user.lastName)}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
```

### include
속성에 앱 네임스페이스 및 변수 이름을 사용함으로써 포함하는 레이아웃에서 포함된 레이아웃의 결합으로 변수를 전달할 수 있다.  
```xml
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:bind="http://schemas.android.com/apk/res-auto">
       <data>
           <variable name="user" type="com.example.User"/>
       </data>

       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
           <include layout="@layout/name" bind:user="@{user}"/>
           <include layout="@layout/contact" bind:user="@{user}"/>
       </LinearLayout>
</layout>
```


## BindingAdapter
BindingAdapter는 적절한 프레임워크를 호출하여 값을 설정하는 작업을 담당한다.  
일부 속성에는 CustomBindingLogic이 필요하다  
예를 들어, `android:paddingLeft` 속성에는 연결된 setter가 없다. 대신 `setPadding(left, top, right, bottom)` 메서드가 제공되니 BindingAdapter 어노테이션이 있는 static BindingAdapter 메서드를 사용하면 속성의 setter가 호출되는 방식을 맞춤설정할 수 있다.  
```kotlin
@BindingAdapter("android:paddingLeft")
fun setPaddingLeft(view: View, padding: Int) {
    view.setPadding(padding,
                view.getPaddingTop(),
                view.getPaddingRight(),
                view.getPaddingBottom())
}

@BindingAdapter("app:goneUnless")
fun goneUnless(view: View, visible: Boolean) {
    view.visibility = if (visible) View.VISIBLE else View.GONE
}

@BindingAdapter("loadImage")
fun bindLoadImageFromUri(view: ImageView, uri: Uri?) {
    uri?.let {
        Glide.with(view.context)
            .load(it.toString())
            .into(view)
    }
}
```
매개변수 유형은 중요하다.  
첫 번째 매개변수는 속성과 연결된 뷰의 유형을 결정한다.  
두 번째 매개변수는 지정된 속성의 결합 표현식에서 허용되는 유형을 결정한다.  
  
개발자가 정의하는 BindingAdapter는 충돌이 발생하면 Android 프레임워크에서 제공하는 기본 어댑터보다 우선 적용된다.  
  
또한 여러가지 속성을 받는 어뎁터도 정의할 수 있다.  
```kotlin
@BindingAdapter("imageUrl", "error")
fun loadImage(view: ImageView, url: String, error: Drawable) {
    Picasso.get().load(url).error(error).into(view)
}
```

```xml
<ImageView 
    app:imageUrl="@{venue.imageUrl}" 
    app:error="@{@drawable/venueError}"/>
```

하나라도 설정될 때 어댑터가 호출되도록 하려면 다음 예시와 같이 어댑터의 선택적 `requireAll` 플래그를 `false`로 설정할 수 있다.  

```kotlin
@BindingAdapter(value = ["imageUrl", "placeholder"], requireAll = false)
fun setImageUrl(imageView: ImageView, url: String?, placeHolder: Drawable?) {
    if (url == null) {
        imageView.setImageDrawable(placeholder);
    } else {
        MyImageLoader.loadInto(imageView, url, placeholder);
    }
}
```

BindingAdapter 메서드는 선택적으로 핸들러의 이전 값을 사용할 수 있다.  
이전 값과 새 값을 사용하는 메서드는 아래 예와 같이 속성의 모든 이전 값을 먼저 선언한 후 새 값을 선언해야 한다.  
```kotlin
@BindingAdapter("android:paddingLeft")
fun setPaddingLeft(view: View, oldPadding: Int, newPadding: Int) {
    if (oldPadding != newPadding) {
        view.setPadding(padding,
                    view.getPaddingTop(),
                    view.getPaddingRight(),
                    view.getPaddingBottom())
    }
}

```

## Observable data object
간단한 기존 객체를 데이터 결합에 사용할 수는 있지만 객체를 수정해도 UI가 자동으로 업데이트되지는 않는다.  
Observable data를 사용하면 데이터 변경 시 리스너라는 다른 객체에 알리는 기능을 데이터 객체에 제공할 수 있다.  
Observable data object에는 세 가지 유형, 즉 객체(object), 필드(field) 및 컬렉션(collection), LiveData, Flow등이 있다.

### ObservableField
`Observable` 클래스 및 다음과 같은 프리미티브(Primitive) 관련 클래스를 사용하여 필드를 식별 가능하게 만들 수 있다.
- [ObservableField](https://developer.android.com/reference/android/databinding/ObservableField)
- [ObservableBoolean](https://developer.android.com/reference/android/databinding/ObservableBoolean)
- [ObservableByte](https://developer.android.com/reference/android/databinding/ObservableByte)
- [ObservableChar](https://developer.android.com/reference/android/databinding/ObservableChar)
- [ObservableShort](https://developer.android.com/reference/android/databinding/ObservableShort)
- [ObservableInt](https://developer.android.com/reference/android/databinding/ObservableInt)
- [ObservableLong](https://developer.android.com/reference/android/databinding/ObservableLong)
- [ObservableFloat](https://developer.android.com/reference/android/databinding/ObservableFloat)
- [ObservableDouble](https://developer.android.com/reference/android/databinding/ObservableDouble)
- [ObservableParcelable](https://developer.android.com/reference/android/databinding/ObservableParcelable)

```kotlin
val firstName = ObservableField<String>()
val lastName = ObservableField<String>()
val age = ObservableInt()
```
다음과 같이 field 값에 액세스한다.
```kotlin
user.firstName = "Google"
val age = user.age
```

### ObservableCollection
일부 앱은 동적 구조를 사용하여 데이터를 보유합니다.  
ObservableCollection 통해 키를 사용하여 이러한 구조에 액세스할 수 있다.  
다음 예와 같이 키가 String과 같은 참조 유형일 때는 `ObservableArrayMap` 클래스가 유용하다.  
```kotlin
 ObservableArrayMap<String, Any>().apply {
        put("firstName", "Google")
        put("lastName", "Inc.")
        put("age", 17)
    }
```

```xml
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap<String, Object>"/>
</data>
    …
<TextView
    android:text="@{user.lastName}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
<TextView
    android:text="@{String.valueOf(1 + (Integer)user.age)}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
 ```

 또 다른 예시
 ```kotlin
  ObservableArrayList<Any>().apply {
        add("Google")
        add("Inc.")
        add(17)
    }
```

```xml
<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList<Object>"/>
</data>
    …
<TextView
    android:text='@{user[Fields.LAST_NAME]}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
<TextView
    android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

### ObservableObject
[Observable](https://developer.android.com/reference/android/databinding/Observable) 인터페이스를 구현하는 클래스를 사용하면 식별 가능한 객체의 속성 변경에 관해 알림을 받으려는 리스너를 등록할 수 있다.  
  
[Observable](https://developer.android.com/reference/android/databinding/Observable) 인터페이스에 리스너를 추가 및 삭제하는 메커니즘이 있지만 알림이 전송되는 시점은 개발자가 직접 결정해야 한다.   
더 쉽게 개발할 수 있도록 데이터 결합 라이브러리는 리스너 등록 메커니즘을 구현하는 [BaseObservable](https://developer.android.com/reference/android/databinding/BaseObservable) 클래스를 제공.  
BaseObservable을 구현하는 데이터 클래스는 속성이 변경될 때 알리는 역할을 한다.   
다음 예와 같이 Bindable 주석을 getter에 할당하고 setter의 `notifyPropertyChanged()` 메서드를 호출함으로써 이 작업을 완료한다.  

```kotlin
class User : BaseObservable() {

    @get:Bindable
    var firstName: String = ""
        set(value) {
            field = value
            notifyPropertyChanged(BR.firstName)
        }

    @get:Bindable
    var lastName: String = ""
        set(value) {
            field = value
            notifyPropertyChanged(BR.lastName)
        }
    }
```    

### LiveData
[ObservableField](#observablefield)와 같이 `Observable`을 구현하는 객체와 달리 LiveData 객체는 데이터 변경을 구독하는 관찰자의 수명 주기를 알고 있다.  
이 수명 주기를 알면 LiveData 사용의 이점에 설명된 많은 이점을 활용할 수 있다.  
  
#### LiveCyclerOwner 등록하기
LiveData 객체를 사용하려면 수명 주기 소유자를 지정하여 LiveData 객체의 범위를 정의해야 한다.  
다음 예에서는 결합 클래스를 인스턴스화한 후 활동을 수명 주기 소유자로 지정한다.  
```kotlin
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // Inflate view and obtain an instance of the binding class.
        val binding: UserBinding = DataBindingUtil.setContentView(this, R.layout.user)

        // Specify the current activity as the lifecycle owner.
        binding.setLifecycleOwner(this)
    }
}
```

#### LiveData 바인딩
```kotlin
private val _homeCategories = MutableLiveData<List<HomeCategory>>()
val homeCategories: LiveData<List<HomeCategory>>
    get() = _homeCategories
```

```
<layout>
    <data>
        <variable
            name="viewmodel"
            type="com.example.main.MainViewModel"/>
        <variable
            name="adapter"
            type="com.example.main.MainAdapter" /> 
    </data>

    ...
     <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/recyclerview"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                app:adapter="@{adapter}"
                app:adapterHomeCategories="@{viewmodel.homeCategories}"
                app:layout_constraintBottom_toBottomOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:spanCount="2"
                tools:listitem="@layout/item_home_category" />
    ...
</layout>
```
```kotlin
object RecyclerViewBindingAdapter {
    @JvmStatic
    @BindingAdapter("adapterHomeCategories")
    fun bindAdapterHomeItemList(recyclerView: RecyclerView, homeCategories: List<HomeCategory>?) {
        homeCategories?.let { (recyclerView.adapter as MainAdapter).addItems(homeCategories) }
    }
}
```

## 단방향, 양방향 바인딩
단방향 DataBinding을 사용하면 아래와같이 사용하게된다.  
속성에 값을 설정하고 이 속성의 변경에 반응하는 리스너를 설정한다.  
```xml
 <CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@{viewmodel.rememberMe}"
    android:onCheckedChanged="@{viewmodel.rememberMeChanged}"/>
```

이것을 양방향 DataBinding으로 변경하면 아래와 같다
```xml
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@={viewmodel.rememberMe}"/>
```
'=' 기호가 포함된 `@={}` 표기법은 속성과 관련된 데이터 변경사항을 받는 동시에 사용자 업데이트를 수신 한다.  

### 커스텀 속성을 통한 양방향 바인딩
초기 값을 설정하고 값이 변경될 때 업데이트하는 메서드에 `@BindingAdapter`를 사용하여 어노테이션을 추가  
```kotlin
@BindingAdapter("time")
@JvmStatic fun setTime(view: MyView, newValue: Time) {
    // Important to break potential infinite loops.
    if (view.time != newValue) {
        view.time = newValue
    }
}
```

뷰에서 값을 읽는 메서드에 `@InverseBindingAdapter`를 사용하여 어노테이션을 추가
```kotlin
@InverseBindingAdapter("time")
@JvmStatic fun getTime(view: MyView) : Time {
    return view.getTime()
} 
```

이때 데이터 결합은 데이터 변경 시 진행할 작업(`@BindingAdapter`를 사용하여 어노테이션을 추가한 메서드를 호출함)  과 뷰 속성 변경 시 호출할 항목(InverseBindingListener를 호출함)을 파악하게 돤다. 하지만 속성이 언제 또는 어떻게 변경되는지는 인식하지 못한다.  
  
속성의 변경 시기 또는 방식을 알기 위해서는 뷰에 리스너를 설정해야 한다.   
리스너는 맞춤 뷰와 연결된 맞춤 리스너이거나 포커스 상실 또는 텍스트 변경과 같은 일반 이벤트일 수 있다.  

```kotlin
@BindingAdapter("app:timeAttrChanged")
@JvmStatic fun setListeners(
        view: MyView,
        attrChange: InverseBindingListener
) {
    // Set a listener for click, focus, touch, etc.
}
```
리스너에는 InverseBindingListener가 매개변수로 포함한다.  
InverseBindingListener를 사용하면 데이터 결합 시스템에 속성이 변경되었음을 알릴 수 있다.  
그러면 시스템은 `@InverseBindingAdapter`를 사용하여 어노테이션이 추가된 메서드 호출을 시작할 수 있다.  


## 리싸이클러뷰 바인딩
리싸이클러뷰 각 아이템에도 DataBinding을 사용할 수 있다.  
```kotlin
// HomeAdapter.kt
class HomeAdapter(
    private val clickListener: HomeItemViewHolder.OnClickListener
) : BindingRecyclerViewAdapter<HomeAdapter.HomeItemViewHolder>() {

    private val items = mutableListOf<HomeCategory>()

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): HomeItemViewHolder {
        val binding = ItemHomeCategoryBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return HomeItemViewHolder(binding)
    }

    override fun onBindViewHolder(holder: HomeItemViewHolder, position: Int) {
        holder.bind(items[position], clickListener)
        holder.binding.executePendingBindings()
    }

    override fun getItemCount(): Int = items.size

    fun addItems(items: List<HomeCategory>) {
        this.items.clear()
        this.items.addAll(items)
        notifyDataSetChanged()
    }

    class HomeItemViewHolder(val binding: ItemHomeCategoryBinding) : RecyclerView.ViewHolder(binding.root) {

        interface OnClickListener {
            fun onItemClick(item: HomeCategory)
        }

        fun bind(item: HomeCategory, clickListener: OnClickListener) {
            binding.item = item
            binding.clickListener = clickListener
        }
    }
}
```

```kotlin
// HomFragment.kt
override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        super.onCreateView(inflater, container, savedInstanceState)
        return binding {
            homeVM = homeViewModel
            adapter = HomeAdapter(object : HomeAdapter.HomeItemViewHolder.OnClickListener {
                override fun onItemClick(item: HomeCategory) {
                    startActivity(Intent(context, CategoryActivity::class.java))
                }
            })
        }.root
    }
```

```xml
<!-- home_fragment.xml-->
<com.beomjo.whitenoise.ui.view.GridRecyclerView
            android:id="@+id/recyclerview"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:adapter="@{adapter}"
            app:adapterHomeCategories="@{homeVM.homeCategories}"
            app:layoutManager="androidx.recyclerview.widget.GridLayoutManager"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:spanCount="2"
            tools:listitem="@layout/item_home_category" />
```

```xml
<!-- item_home_category.xml-->
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="item"
            type="com.beomjo.whitenoise.model.HomeCategory" />

        <variable
            name="clickListener"
            type="com.beomjo.whitenoise.ui.adapters.HomeAdapter.HomeItemViewHolder.OnClickListener" />
    </data>

    <androidx.cardview.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:cardCornerRadius="24dp"
        app:cardElevation="0dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">

        <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="180dp"
            android:background="@{item.getPrimaryColor()}"
            android:onClick="@{()->clickListener.onItemClick(item)}"
            android:padding="30dp"
            tools:background="@android:color/holo_blue_bright">

            <ImageView
                android:layout_width="36dp"
                android:layout_height="36dp"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:loadImage="@{item.iconUrl}" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:gravity="center"
                android:text="@{item.name}"
                android:textColor="@color/white"
                android:textSize="24sp"
                app:layout_constraintBottom_toBottomOf="parent"
                app:layout_constraintStart_toStartOf="parent"
                tools:text="afaf" />
        </androidx.constraintlayout.widget.ConstraintLayout>
    </androidx.cardview.widget.CardView>
</layout>
```

```kotlin
object RecyclerViewBindingAdapter {
    @JvmStatic
    @BindingAdapter("adapterHomeCategories")
    fun bindAdapterHomeItemList(recyclerView: RecyclerView, homeCategories: List<HomeCategory>?) {
        homeCategories?.let { (recyclerView.adapter as HomeAdapter).addItems(homeCategories) }
    }
}
```
