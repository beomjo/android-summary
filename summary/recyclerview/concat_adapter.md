# ConcatAdapter

## ConcatAdapter란?
`MergeAdapter`라는 이름으로 처음 나왔지만, `ConcatAdapter`로 이름이 변경되었다.  
`RecyclerView`는 하나의 Adapter만을 가지며, 반복되는 뷰 리스트를 그리는 위젯이다.  
`RecyclerView`에서는 Header 혹은 Footer 등을 표현하기 위해서는 포지션으로 구분하고, 각 뷰 타입의 로직을 포지션 별로 구분하여 넣어주어야한다.   
이는 한 클래스에서 2가지 역할을 가지는 방법이며, 뷰가 복잡해질수록 코드가 비대해지는 안 좋은 방법이다.  
이러한 문제점을 `ConcatAdapter`로 해결할 수 있다.  

## ConcatAdapter특징
`ConcatAdapter`의 Adapter들은 기본적으로 각자의 ViewHolder Pool을 가지고 있어 서로의 ViewHolder를 재사용할 수 없다.

## 의존성 추가
recyclerview 1.2.0 이상 버전에서 `ConcatAdapter`를 사용할 수 있다.
```
implementation "androidx.recyclerview:recyclerview:{version}"
```

## ConcatAdapter.Config
`ConcatAdapter.Config`로 ConcatAdapter에 설정을 전달할 수 있다

```kotlin
ConcatAdapter(
    ConcatAdapter.Config.Builder()
                    .setIsolateViewTypes(true)
                    .setStableIdMode(StableIdMode.NO_STABLE_IDS)
                    .build(),
    adapter1,
    adapter2,
)
```

`setIsolateViewType`이 true인 경우 Adapter들은 기본적으로 각자의 ViewHolder Pool을 가지고 있어 서로의 ViewHolder를 재사용할 수 없지만, false인 경우 Adapter는 동일한 ViewType을 사용하도록 ViewHolder Pool을 공유한다.  

`setStableIdMode` 설정으로 `ConcatAdapter`가 stableIds를 지원해야 하는지 설정할 수 있다.  
- `NO_STABLE_IDS`: ConcatAdapter하위 어댑터가 보고한 stableIds를 무시한다 (**기본설정**)
- `ISOLATED_STABLE_IDS`: `ConcatAdapter`는 `RecyclerView.Adapter.hasStableIds()`에 대해 true를 리턴하며, 추가된 모든 어댑터들은 stable ids를 가지게된다. 두개의 다른 어댑터는 서로 알지 못하기 때문에 같은 stable ids를 리턴할 수 있으므로, `ConcatAdapter`는 각 어댑터의 id pool 을 서로 분리하여 RecyclerView에 보내지기 전 보고된 stable id를 덮어쓴다. 이 모드에서 `RecyclerView.ViewHolder.getItemId()`의 리턴값은 `RecyclerView.Adapter.getItemId(int)`의 리턴값과 다를 수 있다
- `SHARED_STABLE_IDS`: `ConcatAdapter`는 `RecyclerView.Adapter.hasStableIds()`에 대해 true를 리턴하며, 추가된 모든 어댑터들은 stable ids를 가지게된다. 그러나 `ISOLATED_STABLE_IDS` 모드와는 달리, 리턴된 item ids를 재정의하지 않는다. 자식 어댑터들은 서로를 인지하고, 어댑터 간 item을 이동하지 않는 한 절대 같은 id를 리턴해서는 안된다


위 Config를 설정하지 않으면 기본은 `setIsolateViewTypes(true)`, `setStableIdMode(StableIdMode.NO_STABLE_IDS)` 이다.
```java
@NonNull
public static final Config DEFAULT = new Config(true, NO_STABLE_IDS);
```


## 사용예
```kotlin
adapter = ConcatAdapter(
              ConcatAdapter.Config.DEFAULT,
              SearchControlMenuAdapter(),
              SearchPagingAdapter()
          )
```

