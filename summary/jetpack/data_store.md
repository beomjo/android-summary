# DataStore

## DataStore 란 ?
[Protocol Buffer](https://developers.google.com/protocol-buffers)를 사용하여 키-값 쌍 또는 Type(유형)이 지정된 객체를 저장할 수 있는 데이터 저장소이다.
DataStore는 Kotlin Coroutine 및 Flow를 사용하여 비동기적이고 일관된 트랜잭션 방식으로 데이터를 저장한다.
DataStore는 소규모 단순 데이터 셋에 적합하며, 부분업데이트나 참조 무결정은 지원하지 않는다.
복잡한 대규모 데이터 셋, 부분 업데이트, 참조 무결성 등을 지원해야 하는 경우에는 DataStore대신 Room을 사용하는것이 좋다.

## DataStore 종류
- Preferences DataStore : 키를 사용하여 데이터를 저장하고 데이터에 엑세스한다. 타입 안정성을 제공하지 않으며 사전 정의된 스키마가 필요하지 않다.
- Proto DataStore : 맞춤 데이터 타입(Type)의 인스턴스로 데이터를 저장한다. 타입 안정성을 제공하며 [Protocol Buffer](https://developers.google.com/protocol-buffers)를 사용하여 스키마를 정의해야 한다.


## SharedPreference vs PreferenceDataStore vs PhotoDataStore
||SharedPreference|PreferencesDataStore|ProtoDataStore|
|-------------|----------------|----------------|----------------|
|Async(비동기)처리| ✅ ([Listener](https://developer.android.com/reference/android/content/SharedPreferences.OnSharedPreferenceChangeListener)를 통해 변경된 값을 읽는 경우만 가능) | ✅ (Coroutine Flow, RxJava2, RxJava3 사용가능)| ✅ (Coroutine Flow, RxJava2, RxJava3 사용가능)|
|동기처리| ✅ (그러나 UI Thread에서 호출하기에 안전하지 않음) | ❌ | ❌ |
|UI Thread에서 호출이 안전|❌ (1)|✅ (Dispatchers.IO 에서 작업)|✅ (Dispatchers.IO 에서 작업)|
|오류가 발생했는지 알 수 있음|❌|✅|✅|
|런타임 Exception에서 안전함|❌|✅|✅|
|강력한 일관성을 보장하는 Transaction API가 있음|❌|✅|✅|
|데이터 마이그레이션 가능여부|❌|✅|✅|
|Type 안정성(Type보장)|❌|❌|✅|
(1) SharedPreferences는 UI Thread에서 안전하게 호출할 수 있지만 실제로는 Disk I/O 작업을 수행하는 동기 API가 있다.
또한  `apply()`에서 UI Thread를 `fsync()`에서 차단한다.
보류중인 `fsync()` 호출은 응용프로그램의 모든 위치에서 SharedPreferences의 작업이 `apply()`start, stop될 때 마다 트리거된다. 따라서 종종 `apply()`에 의해 예약된 보류중인 `fsync()`호출시에 ANR의 원인이 된다.


## PreferencesDataStore

## 특징
- Transaction 방식으로 데이터 업데이트 처리
- 데이터의 현재 상태를 노출한다
- `apply()` `commit()` 메소드가 없다
- 내부 상태에 대한 변경가능한 참조를 반환하지 않음

### Setting
Preferences DataStore
```
// .gradle(module)
// Preferences DataStore (SharedPreferences like APIs)
dependencies {
  implementation "androidx.datastore:datastore-preferences:1.0.0-alpha07"

  // optional - RxJava2 support
  implementation "androidx.datastore:datastore-preferences-rxjava2:1.0.0-alpha07"

  // optional - RxJava3 support
  implementation "androidx.datastore:datastore-preferences-rxjava3:1.0.0-alpha07"
}
// Alternatively - use the following artifact without an Android dependency.
dependencies {
  implementation "androidx.datastore:datastore-preferences-core:1.0.0-alpha07"
}
```

### Preferences DataStore 만들기
```kotlin
private const val USER_PREFERENCES_NAME = "user_preferences"

val Context.dataStore by preferencesDataStore(name = USER_PREFERENCES_NAME)
```

### Preferences DataStore 읽기
```kotlin
enum class SortOrder {
    NONE,
    BY_DEADLINE,
    BY_PRIORITY,
    BY_DEADLINE_AND_PRIORITY
}

object PreferencesKeys {
    val SHOW_COMPLETED = booleanPreferencesKey("show_completed ")
    val SORT_ORDER = stringPreferencesKey("sort_order")
}
```
```kotlin
val userPreferencesFlow: Flow<UserPreferences> = dataStore.data
    .catch { exception ->
        if (exception is IOException) {
            emit(emptyPreferences())
        } else {
            throw exception
        }
    }
    .map { preferences ->
        val sortOrder = 
            SortOrder.valueOf(preferences[PreferencesKeys.SORT_ORDER] ?: SortOrder.NONE.name)
        val showCompleted = preferences[PreferencesKeys.SHOW_COMPLETED] ?: false
        UserPreferences(
            showCompleted = showCompleted,
            sortOrder = sortOrder,
        )
    }
```

### Preferences DataStore에 쓰기
```kotlin
suspend fun enableSortByDeadline(enable: Boolean) {
    dataStore.edit { preferences ->
        val currentOrder = SortOrder.valueOf(
            preferences[PreferencesKeys.SORT_ORDER] ?: SortOrder.NONE.name
        )

        val newSortOrder =
            if (enable) {
                if (currentOrder == SortOrder.BY_PRIORITY) {
                    SortOrder.BY_DEADLINE_AND_PRIORITY
                } else {
                    SortOrder.BY_DEADLINE
                }
            } else {
                if (currentOrder == SortOrder.BY_DEADLINE_AND_PRIORITY) {
                    SortOrder.BY_PRIORITY
                } else {
                    SortOrder.NONE
                }
            }
        preferences[PreferencesKeys.SORT_ORDER] = newSortOrder.name
    }
}
```

## ProtoDataStore

### 특징
[Protocol Buffer](https://developers.google.com/protocol-buffers)를 사용하여 스키마를 정의한다.
Protobufs를 사용하면 강력한 Type 데이터를 유지할 수 있다.  
XML 및 기타 유사한 데이터 형식보다 더 빠르고, 작고, 단순하며, 모호하지 않다. 
Proto DataStore에서는 새로운 직렬화 메커니즘을 배워야하지만 Proto DataStore가 제공하는 강력한 유형의 이점은 그만한 가치가 있다.

### Setting
Proto DataStore
```
// .gradle(module)
// Typed DataStore (Typed API surface, such as Proto)
dependencies {
  implementation "androidx.datastore:datastore:1.0.0-alpha07"

  // optional - RxJava2 support
  implementation "androidx.datastore:datastore-rxjava2:1.0.0-alpha07"

  // optional - RxJava3 support
  implementation "androidx.datastore:datastore-rxjava3:1.0.0-alpha07"
}
// Alternatively - use the following artifact without an Android dependency.
dependencies {
  implementation "androidx.datastore:datastore-core:1.0.0-alpha07"
}
```


