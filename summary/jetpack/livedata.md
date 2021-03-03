# LiveData

## LiveData란?
 LiveData는 데이터를 저장하고 변화를 관찰할 수 있는 Data Holder 클래스이다.
 LiveData는 또한 Activity, Fragment, Service등의 안드로이드 컴포넌트의 LifeCycle(생명주기)를 인식한다. 
 그리고 생명주기가 Active(활성)일때만 데이터를 업데이트한다.
 
 ![image](https://user-images.githubusercontent.com/39984656/109813128-47efa080-7c70-11eb-8479-1b9521523b29.png)

Activity를 예로 보면 
`onStart()`, `onResume()`같이 Active(활성)상태 일때만 변경을 처리하고,
`onStop()`같이 다른 화면으로 넘어가고있는 상태라면 데이터 변경을 처리하지 않는다.
그리고 다시 원래 화면으로 돌아와 `onResume()`이 호출되면 마지막에 변경된 최신 데이터를 실행한다.


## 장점
- **UI가 데이터 상태와 일치하는지 확인**
    - Observer 패턴으로 데이터가 변경될때마다 UI갱신 가능
- **메모리 누수 없음, 더 이상 수동 수명주기 처리 필요하지 않음**
    - LiveData는 LifeCycleOwner가 파괴될때 스스로 정리된다
- **활동 중지로 인한 충돌 없읍**
    - 수명주기가 비활성 상태일때 LiveData 이벤트 받지않음
- **항상 최신 데이터**
    - 수명주기가 비활성 상태에서 다시 활성화 될때 최신 데이터를 받는다
- **적절한 구성 변경**
    - 장치회전과 같은 구성 변경으로 인해 Activity 또는 Fragment이 다시 create되었을때 최신 데이터가 즉시 수신됨
- **자원 공유**
    - LiveData를 상속하여 자신만의 LiveData클래스를 구현가능
    - 싱글톤 패턴을 이용하여 시스템 서비스를 둘러싸면(Wrap) 앱 어디에서나 자원 공유 가능


## LiveData 확장
LiveData를 상속하여 자신만의 LiveData클래스를 구현할 수 있다

LiveData의 메소드
- observe() : LifeCycleOwner수명 내의 관찰자 목록에 지정된 관찰자 추가
- observeForever() : 관찰자 목록에 지정된 관찰자를 추가
- removeObserver() : 관찰자 목록에 지정된 관찰자 제거
- removeObservers() : 모든 관찰자 제거
- postValue() :  주어진 값을 설정하기 위헤 작업을 메인스레드로 게시
- setValue() : 값을 설정
- getValue() : 현재 값 반환
- onActive() : 활성 관찰자수가 0에서 1로 변경될 때 호출
- onInactive() : 활성 관찰자수가 1에서 0으로 변경될 때 호출
- hasObservers() : LiveData에 옵저버가 있다면 true반환
- hasActiveObservers() : LiveData에 활성 관찰자가 있으면 true반환
```kotlin
class MyLiveData<T> : LiveData<T>() {

    override fun observe(owner: LifecycleOwner, observer: Observer<in T>) {
        super.observe(owner, observer)
    }

    override fun observeForever(observer: Observer<in T>) {
        super.observeForever(observer)
    }

    override fun removeObserver(observer: Observer<in T>) {
        super.removeObserver(observer)
    }

    override fun removeObservers(owner: LifecycleOwner) {
        super.removeObservers(owner)
    }

    override fun postValue(value: T) {
        super.postValue(value)
    }

    override fun setValue(value: T) {
        super.setValue(value)
    }

    override fun getValue(): T? {
        return super.getValue()
    }

    override fun onActive() {
        super.onActive()
    }

    override fun onInactive() {
        super.onInactive()
    }

    override fun hasObservers(): Boolean {
        return super.hasObservers()
    }

    override fun hasActiveObservers(): Boolean {
        return super.hasActiveObservers()
    }

}
```

LiveData를 상속하여 만든 예시
```kotlin
open class VolatileLiveData<T> : MutableLiveData<T>() {
    private val lastValueSeq = AtomicInteger(0)
    private val wrappers = HashMap<Observer<in T>, Observer<T>>()

    @MainThread
    override fun setValue(value: T) {
        lastValueSeq.incrementAndGet()
        super.setValue(value)
    }

    @MainThread
    override fun observe(owner: LifecycleOwner, observer: Observer<in T>) {
        val observerWrapper = ObserverWrapper(lastValueSeq, observer)
        wrappers[observer] = observerWrapper
        super.observe(owner, observerWrapper)
    }

    @MainThread
    override fun observeForever(observer: Observer<in T>) {
        val observerWrapper = ObserverWrapper(lastValueSeq, observer)
        wrappers[observer] = observerWrapper
        super.observeForever(observerWrapper)
    }

    @MainThread
    override fun removeObserver(observer: Observer<in T>) {
        val observerWrapper = wrappers[observer]
        observerWrapper?.let {
            wrappers.remove(observerWrapper)
            super.removeObserver(it)
        }
    }

    private class ObserverWrapper<T>(private var currentSeq: AtomicInteger, private val observer: Observer<in T>) :Observer<T> {
        private val initialSeq = currentSeq.get()
        private var _observer: Observer<in T> = Observer {
            if (currentSeq.get() != initialSeq) {
                _observer = observer
                _observer.onChanged(it)
            }
        }

        override fun onChanged(value: T) {
            _observer.onChanged(value)
        }
    }
}
```


## LiveData와 데이터바인딩
Databinding과 LiveData를 함께 쓰면 편리하다.
```xml
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <variable name="vm" type="MainViewModel" />
    </data>
.........

</layout>
```
```xml
<TextView
    .....
     android:text="@{vm.post.title}"
     
</TextView>
```
title이 변경될 때마다 TextView UI는 알아서 변경된다.


## LiveData Transformation 

### [Map](https://developer.android.com/reference/androidx/lifecycle/Transformations#map(androidx.lifecycle.LiveData%3CX%3E,%20androidx.arch.core.util.Function%3CX,%20Y%3E))
LiveData의 데이터를 변경한 뒤, 변경된 데이터를 발행

```kotlin
val userLiveData: LiveData<User> = UserLiveData()
val userName: LiveData<String> = Transformations.map(userLiveData) {
    user -> "${user.name} ${user.lastName}"
}
```

### [SwitchMap](https://developer.android.com/reference/androidx/lifecycle/Transformations#switchMap(androidx.lifecycle.LiveData%3CX%3E,%20androidx.arch.core.util.Function%3CX,%20androidx.lifecycle.LiveData%3CY%3E%3E))
```kotlin
switchMap(LiveData<X> source, Function<X, LiveData<Y>> switchMapFunction)
```
- switchMap의 첫 번째 인자로 LiveData source를 넘겨준다.
넘겨준 LiveData source 가 변경될 때마다 switchMap이 반환하는 새로운 LiveData의 value역시 새롭게 갱신된다
- 두 번째 인자로는 함수를 넘겨준다. 함수의 파라미터 타입은 source로 넘겨준 LiveData의 value Type(<X>)이며 함수의 **return값은 LiveData이어야만 한다**

```kotlin
 val userIdLiveData: MutableLiveData<Int> = MutableLiveData<Int>().apply { value = 1 };
 val userLiveData: LiveData<User> = Transformations.switchMap(userIdLiveData) { id ->
     repository.getUserById(id)
 }

 fun setUserId(userId: Int) {
      userIdLiveData.setValue(userId);
 }
```

### [MediatorLiveData](https://developer.android.com/reference/androidx/lifecycle/MediatorLiveData)
Mediator는 중개자 라는 뜻으로
MediatorLiveData는 여러 데이터소스를 한곳에서 Observe할때 사용한다
서, 각각의 데이터 소스들이 변경되는 것을 따로 따로 관찰하는 것이 아니라 어떤 소스에서 변경이 일어나든 한번에 관찰하려고 하는 것이다.
Rx의 merge와 비슷하다고 보면 된다.

```kotlin
 LiveData liveData1 = ...;
 LiveData liveData2 = ...;

 MediatorLiveData liveDataMerger = new MediatorLiveData<>();
 liveDataMerger.addSource(liveData1, value -> liveDataMerger.setValue(value));
 liveDataMerger.addSource(liveData2, value -> liveDataMerger.setValue(value));
 ```

 ```kotlin
  liveDataMerger.addSource(liveData1, new Observer() {
      private int count = 1;

      @Override public void onChanged(@Nullable Integer s) {
          count++;
          liveDataMerger.setValue(s);
          if (count > 10) {
              liveDataMerger.removeSource(liveData1);
          }
      }
 });
 ```