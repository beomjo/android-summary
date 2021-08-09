# Paging3

## Paging3란?
로컬 저장소에서나 네트워크를 통해 대규모 데이터 세트의 데이터 페이지를 로드하고 표시하여 네트워크 대역폭과 시스템 리소스를 모두 더 효율적으로 사용할 수 있도록 도와주는 라이브러리이다.  
쉽게 말하면 안드로이드에서 로컬 데이터베이스 또는 네트워크(Remote)의 데이터를 페이지 단위로 UI에 쉽게 표현할 수 있도록 도와주는 라이브러리다.  


## 기존 Paging 구현 방식
기존에 페이징을 구현하는 방식은 RecyclerView와 같은 리스트 UI가 상단 또는 하단에 도달했는지 판단하는 코드를 작성한 뒤,   
하단에 도달하였을때 다음 페이지를 로드(or Refresh)하는 코드를 실행하는 방식이 주를 이루었다.   
페이징이 필요한 모든 화면에 동일한 코드를 작성해야만 했고 네트워크 오류, 스크롤 오류등의 예외 처리 코드가 많이 필요하였다.  
Paging 라이브러리를 사용할 경우 위와 같은 과정없이 쉽게 페이징을 구현할 수 있다.  


## Paging3 기능 및 장점
Paging 3은 이전 Paging 라이브러리 버전과 크게 달라졌다.   

- **페이징된 데이터의 메모리 내 캐싱 지원**
    - 앱이 페이징 데이터로 작업하는 동안 시스템 리소스를 효율적으로 사용할 있음
- **요청 중복 제거 기능이 기본으로 제공**
   - 앱에서 네트워크 대역폭과 시스템 리소스를 효율적으로 사용할 수 있음
- **사용자가 로드된 데이터의 끝까지 스크롤할 때 구성 가능한 RecyclerView 어댑터가 자동으로 데이터를 요청**
- **Kotlin Coroutine 및 Flow뿐만 아니라 LiveData 및 RxJava 최고 수준 지원**
- **새로고침 및 재시도 기능을 포함하여 오류 처리 기본으로 지원**
- **기존 Paging2의 DataSource들을 통합, 심플한 DataSource 인터페이스 제공**
   - `DatPageKeyedDataSource`, `PositionalDataSource`, `ItemKeyedDataSource` 통합
- **Header, Footer 지원**


## 핵심개념
- `PagingData`: 페이지로 나눈 데이터의 컨테이너. 데이터를 새로고침할 때마다 상응하는 PagingData가 별도로 생성된다
- `PagingSource`: `PagingSource`는 데이터의 스냅샷을 PagingData의 스트림으로 로드하기 위한 기본 클래스
- `Pager.flow`: 구현된 `PagingSource`의 구성 방법을 정의하는 함수와 PagingConfig를 기반으로 `Flow<PagingData>`를 빌드
- `PagingDataAdapter`: `RecyclerView`에 `PagingData`를 표시하는 `RecyclerView.Adapter`. `PagingDataAdapter`는 `Kotlin Flow`, `LiveData`, `RxJava` `Flowable` 또는 `RxJava Observable`에 연결할 수 있다. `PagingDataAdapter`는 페이지가 로드될 때 내부 `PagingData` 로딩 이벤트를 수신 대기하고 업데이트된 콘텐츠가 새로운 `PagingData` 객체 형태로 수신될 때 백그라운드 스레드에서 DiffUtil를 사용하여 세분화된 업데이트를 계산한다
- `RemoteMediator`: 네트워크 및 데이터베이스에서 페이지로 나누기를 구현하는 데 유용

### `PagingSource`
PagingSource를 정의하려면 아래의 항목을 정의해야한다.  
- **페이징 키의 유형**: 현재 로드한 데이터의 페이지 정보 데이터의 타입, 예시에서 GitHub는 API에서 페이지에 1을 기반으로 하는 색인 번호를 사용하므로 유형은 Int
- **로드된 데이터의 유형**: 응답 모델 타입 `Repo`
- **데이터를 가져오는 위치**: Retrofit에서 가져오므로 `GithubService`
```kotlin
class GithubPagingSource(
        private val service: GithubService,
        private val query: String
) : PagingSource<Int, Repo>() {
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Repo> {
        TODO("Not yet implemented")
    }
   override fun getRefreshKey(state: PagingState<Int, Repo>): Int? {
        TODO("Not yet implemented")
    }
}
```

`PagingSource`에서는 두 가지 함수를 구현해야한다.  
- `public abstract suspend fun load(params: LoadParams<Key>): LoadResult<Key, Value>`
- `public abstract fun getRefreshKey(state: PagingState<Key, Value>): Key?`

#### load 함수
load 함수는 `LoadResult`를 반환해야한다.   
- `LoadResult.Page`: 로드에 성공한 경우
- `LoadResult.Error`: 오류가 발생한 경우

`LoadResult.Page`를 구성할 때 상응하는 방향으로 목록을 로드할 수 없는 경우 `nextKey` 또는 `prevKey`에 null을 전달한다.   
예를 들어 네트워크 응답에 성공했지만 목록이 비어 있는 경우에는 로드할 데이터가 없는 것으로 간주할 수 있습니다. 따라서 nextKey가 null일 수 있다.  

#### getRefreshKey 함수
새로고침키는 `PagingSource.load()`의 후속 새로고침 호출에 사용 된다.  
첫 번째 호출은 Pager에 의해 제공되는 initialKey를 사용한다.  
새로고침은 스와이프하여 새로고침하거나 데이터베이스 업데이트, 구성 변경, 프로세스 중단 등으로 인해 무효화되어   
Paging 라이브러리가 현재 목록을 대체할 새 데이터를 로드하려고 할 때마다 발생한다.   
일반적으로 후속 새로고침 호출은 가장 최근에 액세스한 인덱스를 나타내는 `PagingState.anchorPosition` 주변 데이터의 로드를 다시 시작한다.  

### PagingData
`PagingData`는 `PagingData`를 다른 레이어로 전달할 API를 결정해야한다.  
- **Kotlin Flow**: `Pager.flow` 사용
- **LiveData**: `Pager.liveData` 사용
- **RxJava Flowable**: `Pager.flowable` 사용
- **RxJava Observable**: `Pager.observable` 사용

또한 `PageData`는 아래의 매개변수를 전달해야 한다.
- **PagingConfig**
    - 로드 대기 시간, 초기 로드의 크기 요청 등 PagingSource에서 콘텐츠를 로드하는 방법에 관한 옵션 설정
    - 정의해야 하는 유일한 필수 매개변수는 각 페이지에 로드해야 하는 항목 수를 가리키는 *페이지 크기*이다
    - 기본적으로 Paging은 로드하는 모든 페이지를 메모리에 유지한다. 사용자가 스크롤할 때 메모리를 낭비하지 않으려면 `PagingConfig`에서 `maxSize` 매개변수를 설정해야한다. 기본적으로 Paging은 로드되지 않은 항목을 집계할 수 있고 `enablePlaceholders` 구성 플래그가 true인 경우 아직 로드되지 않은 콘텐츠의 자리표시자로 null 항목을 반환한다
    - `PagingConfig.pageSize`는 여러 화면의 항목이 포함될 만큼 충분히 커야한다. 페이지가 너무 작으면 페이지의 콘텐츠가 전체 화면을 가리지 않기 때문에 목록이 깜박일 수 있다. 페이지 크기가 클수록 로드 효율이 좋지만 목록이 업데이트될 때 지연 시간이 늘어날 수 있다
    - 기본적으로 `PagingConfig.maxSize`는 무제한이므로 페이지가 삭제되지 않는다. 페이지를 삭제하려면 사용자가 스크롤 방향을 변경할 때 네트워크 요청이 너무 많이 발생하지 않도록 `maxSize`를 충분히 큰 수로 유지해야 한다. 최솟값은 `pageSize + prefetchDistance * 2`이다
- **PagingSource를 만드는 방법을 정의하는 함수**


### Paging.flow
PagingConfig를 기반으로 `Flow<PagingData<T>>`를 빌드  
```kotlin
fun getSearchResultStream(query: String): Flow<PagingData<Repo>> {
        return Pager(
            config = PagingConfig(
                pageSize = NETWORK_PAGE_SIZE,
                enablePlaceholders = false,
            ),
            pagingSourceFactory = { GithubPagingSource(query, service) }
        ).flow
    }
```

### PagingAdapter
`PagingData`를 `RecyclerView`에 바인딩하려면 `PagingDataAdapter`를 사용하면된다.  
`PagingData` 콘텐츠가 로드될 때마다 `PagingDataAdapter`에서 알림을 받은 다음 `RecyclerView`에 업데이트하라는 신호를 보낸다


### 
