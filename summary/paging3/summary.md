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
- **기존 Paging2의 DataSource들을 통합, 심플한 DataSource 인터페이스 제공**
   -  `DatPageKeyedDataSource`, `PositionalDataSource`, `ItemKeyedDataSource` 통합
- **Header, Footer 지원**


## 핵심개념
- `PagingData`: 페이지로 나눈 데이터의 컨테이너. 데이터를 새로고침할 때마다 상응하는 PagingData가 별도로 생성된다
- `PagingSource`: `PagingSource`는 데이터의 스냅샷을 PagingData의 스트림으로 로드하기 위한 기본 클래스
- `Pager.flow`: 구현된 `PagingSource`의 구성 방법을 정의하는 함수와 PagingConfig를 기반으로 `Flow<PagingData>`를 빌드
- `PagingDataAdapter`: `RecyclerView`에 `PagingData`를 표시하는 `RecyclerView.Adapter`. `PagingDataAdapter`는 `Kotlin Flow`, `LiveData`, `RxJava` `Flowable` 또는 `RxJava Observable`에 연결할 수 있다. `PagingDataAdapter`는 페이지가 로드될 때 내부 `PagingData` 로딩 이벤트를 수신 대기하고 업데이트된 콘텐츠가 새로운 `PagingData` 객체 형태로 수신될 때 백그라운드 스레드에서 DiffUtil를 사용하여 세분화된 업데이트를 계산한다
- `RemoteMediator`: 네트워크 및 데이터베이스에서 페이지로 나누기를 구현하는 데 유용

### `PagingSource`
PagingSource를 정의하려면 아래의 항목을 정의해야한다.  
- **페이징 키의 유형**: 현재 로드한 데이터의 페이지 정보 데이터의 타입, 예시에서 검색 API에서 페이지에 1을 기반으로 하는 색인 번호를 사용하므로 유형은 Int
- **로드된 데이터의 유형**: 응답 모델 타입 `Document`
- **데이터를 가져오는 위치**: Retrofit에서 가져오므로 `SearchApi`
```kotlin
class SearchPagingSource(
        private val query: String,
        private val api: SearchApi
) : PagingSource<Int, Document>() {
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Document> {
        TODO("Not yet implemented")
    }
   override fun getRefreshKey(state: PagingState<Int, Document>): Int? {
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
PagingConfig를 기반으로 `Flow<PagingData<T>>`를 전달
```kotlin
fun getSearchResultStream(query: String): Flow<PagingData<Repo>> {
        return Pager(
            config = PagingConfig(
                pageSize = NETWORK_PAGE_SIZE,
                enablePlaceholders = false,
            ),
            pagingSourceFactory = { SearchPagingSource(query, api) }
        ).flow
    }
```

### PagingDataAdapter
`PagingData`를 `RecyclerView`에 바인딩하려면 `PagingDataAdapter`를 사용하면된다.  
`PagingData` 콘텐츠가 로드될 때마다 `PagingDataAdapter`에서 알림을 받은 다음 `RecyclerView`에 업데이트하라는 신호를 보낸다.  
```kotlin
PagingDataAdapter<T : Any, VH : RecyclerView.ViewHolder>
```
T는 `PagingData`의 타입 이다.  

```kotlin
class SearchPagingAdapter : PagingDataAdapter<SearchItem, RecyclerView.ViewHolder>(SearchDiffUtil()) {
    // body is unchanged
}
```


## LoadStateAdapter를 사용하여 Header, Footer 추가하기
`LoadStateAdapter`는 로드 상태가 변경되면 자동으로 알림을 받는다.
상태는 `LoadState`값으로 `onCreateViewHolder`, `onBindViewHolder` 함수로 넘어오게 된다.

```kotlin
class SearchPagingLoadStateAdapter(private val retry: () -> Unit) : LoadStateAdapter<SearchLoadStateViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, loadState: LoadState): SearchLoadStateViewHolder {
        return SearchLoadStateViewHolder.create(parent, retry)
    }
    
    override fun onBindViewHolder(holder: SearchLoadStateViewHolder, loadState: LoadState) {
        holder.bind(loadState)
    }
}

```
`LoadState`가 `LoadState.Loading`일때는 ProgressBar표시, `LoadState.Error`일때는 Retry뷰를 표시해주면 된다.  
재시도 매커니즘은 `PagingDataAdapter`의 retry함수를 사용한다.   
`PagingDataAdapter`에 헤더와 푸터는 아래처럼 등록한다.  
```kotlin
private fun initAdapter() {
    binding.list.adapter = adapter.withLoadStateHeaderAndFooter(
            header = SearchPagingLoadStateAdapter { adapter.retry() },
            footer = SearchPagingLoadStateAdapter { adapter.retry() }
    )
}
```


## 구분자(Separator) 추가하기
구분자를 추가하면 목록의 가독성을 개선할 수도 있다.
예를들어 날짜별로 정렬한다고 할때 월 별로 구분하는 구분자를 추가할 수 있겠다.

PagingData의 모델 변경
```kotlin
 sealed class SearchUiItem {
        data class DocumentItem(val document: Document) : SearchUiItem()
        data class SeparatorItem(val description: String) : SearchUiItem()
}
``` 

```kotlin
 private fun getPager(requestParam: SearchPagingParam): LiveData<PagingData<SearchUiItem>> {
        return getSearchPagingData(requestParam)
            .map { pagingData -> pagingData.map { document -> SearchUiItem.DocumentItem(document) } }
            .map { pagingData ->
                when (sort) {
                    SortType.TITLE -> insertTitleSeparator(pagingData)
                    else -> insertDateSeparator(pagingData)
                }
            }
            .cachedIn(viewModelScope)
            .asLiveData()
    }

    private fun insertTitleSeparator(it: PagingData<SearchUiItem.DocumentItem>): PagingData<SearchUiItem> {
        return it.insertSeparators { before, after ->
            if (after == null) return@insertSeparators null
            if (before == null) return@insertSeparators SearchUiItem.SeparatorItem("${after.document.title.first()}")

            val beforeFirstWord = before.document.title.first()
            val afterFirstWord = after.document.title.first()
            return@insertSeparators when (beforeFirstWord != afterFirstWord) {
                true -> SearchUiItem.SeparatorItem("${after.document.title.first()}")
                else -> null
            }
        }
    }

    private fun insertDateSeparator(it: PagingData<SearchUiItem.DocumentItem>): PagingData<SearchUiItem> {
        return it.insertSeparators { before, after ->
            if (after == null) return@insertSeparators null
            if (before == null) return@insertSeparators SearchUiItem.SeparatorItem(
                dateHelper.convert(after.document.date)
            )

            val beforeDate = before.document.date
            val afterDate = after.document.date

            val beforeDateString = dateHelper.convert(beforeDate, R.string.date_month)
            val afterDateString = dateHelper.convert(afterDate)
            return@insertSeparators when (beforeDateString != afterDateString) {
                true -> SearchUiItem.SeparatorItem(description = afterDateString)
                else -> null
            }
        }
    }
```


## 네트워크 및 데이터베이스에서 페이징(RemoteMediator)
로컬 데이터베이스에 데이터를 저장하여 앱에 오프라인 지원을 추가할 수 있다.   
이렇게 하면 데이터베이스가 앱의 정보 소스가 되고 항상 데이터베이스에서 데이터가 로드 된다.   
더 이상 데이터가 없을 때마다 네트워크에 더 많은 데이터를 요청한 다음 데이터베이스에 저장한다.  
데이터베이스가 정보 소스이므로 더 많은 데이터가 저장되면 UI가 자동으로 업데이트 된다.  

### Room 설정

#### Entity 추가
목록을 로걸 DB에 저장하므로 아래와같이 Entity(테이블)를 작성한다.
```kotlin
@Entity(tableName = "document_table")
internal data class DocumentTable(
    @field:SerializedName("type")
    val type: DocumentType,
    @PrimaryKey
    @field:SerializedName("url")
    val url: String,
    @field:SerializedName("thumbnail")
    val thumbnail: String,
    @field:SerializedName("title")
    val title: String,
    @field:SerializedName("content")
    val content: String,
    @field:SerializedName("date")
    val date: Date?,
)
```

`RemoteMediator`의
`PagingState`에서 마지막으로 로드된 항목을 가져오면 항목이 속한 페이지의 색인을 알 수 없으므로,  
이 문제를 해결하기 위해 `remote_key`라고 하는 다음 및 이전 페이지 키를 저장하는 다른 Entity(테이블)을 추가 한다.  
```kotlin
@Entity(tableName = "remote_key")
data class RemoteKeyTable(
    @PrimaryKey
    @field:SerializedName("position")
    val position: Int? = -1,
    @field:SerializedName("prev_key")
    val prevKey: Int?,
    @field:SerializedName("next_key")
    val nextKey: Int?
)
```

#### Dao 추가
Entity(테이블)에 접근하기위한 인터페이스 추가,
Title 또는 Date로 정렬하여 테이블에서 가져오도록 쿼리를 작성한다.
```kotlin
@Dao
internal interface DocumentDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertDocuments(documents: List<DocumentTable>)

    @Query(
        "SELECT * FROM document_table " +
                "WHERE title Like '%'||:query||'%' or content Like '%'||:query||'%' " +
                "ORDER BY date DESC"
    )
    fun getDocumentByDate(query: String): PagingSource<Int, DocumentTable>

    @Query(
        "SELECT * FROM document_table " +
                "WHERE title Like '%'||:query||'%' or content Like '%'||:query||'%' " +
                "ORDER BY title ASC"
    )
    fun getDocumentByTitle(query: String): PagingSource<Int, DocumentTable>

    @Query("DELETE from document_table")
    suspend fun clearAllDocuments()
}
```

다음 및 이전 페이지 키를 저장하는 다른 Entity(테이블)도 접근할 수 있는 인터페이스를 정의.  
```kotlin
@Dao
interface RemoteKeyDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertKey(remoteKey: RemoteKeyTable)

    @Query("SELECT * FROM remote_key")
    suspend fun getRemoteKeyTable(): Array<RemoteKeyTable>?

    @Query("DELETE FROM remote_key")
    suspend fun clearRemoteKeys()
}
```

#### Database 정의
```kotlin
@Database(
    entities = [DocumentTable::class, RemoteKeyTable::class],
    version = 1,
    exportSchema = false
)
@TypeConverters(DateConverter::class)
internal abstract class AppDatabase : RoomDatabase() {

    abstract fun documentDao(): DocumentDao

    abstract fun remoteKeyDao(): RemoteKeyDao

    companion object {

        private const val NAME = "app_database"

        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getInstance(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                INSTANCE ?: Room.databaseBuilder(
                    context,
                    AppDatabase::class.java,
                    NAME
                ).build()
            }
        }
    }
}
```  

### RemoteMediator 구현
`RemoteMediator`는 `load` 함수를 필수로 구현해야 한다.   

```kotlin
public abstract suspend fun load(
        loadType: LoadType,
        state: PagingState<Key, Value>
    ): MediatorResult
```
`load`함수는 `LoadType`과, `PagingState`를 전달하며,
반환값으로 `MediatorResult`를 반환해야한다.

**LoadType**
- `LoadType.REFRESH`: `PagingSource`가 무효화되었고, `PagingData`를 새로고침할때, 초기화 신호
- `LoadType.PREPEND`: `PagingSource`의 현재 데이터의 첫번째 페이지에 도달하였을경우 
- `LoadType.APPEND`: `PagingSource`의 현재 데이터의 마지막 페이지에 도달하였을경우 

**PagingState** : 이전에 로드된 페이지, 목록에서 가장 최근에 액세스한 index, 페이징 스트림을 초기화할 때 정의한 `PagingConfig` 정보를 담고 있다. 

**MediatorResult**
- `MediatorResult.Success`: 네트워크에서 데이터를 가져온 경우, 여기에서 더 많은 데이터를 로드할 수 있는지 여부(`endOfPaginationReached`로)를 전달한다
- `MediatorResult.Error`: 네트워크에 데이터를 요청하는 동안 오류가 발생한 경우 반환


#### load 함수 구현
load 함수에서 제공되는 LoadType을 활용하여 데이터 추가로드 여부에 대한 로직을 구현한 뒤, 결과를 load함수의 반환형인 `MediatorResult`에 `endOfPaginationReached` 파라미터에 넘겨주어 데이터 로드를 끝마칠지 판단한다.
```kotlin
  override suspend fun load(
        loadType: LoadType,
        state: PagingState<Int, DocumentTable>
    ): MediatorResult {
        try {
            val position: Int = when (loadType) {
                LoadType.REFRESH -> STARTING_POSITION
                LoadType.PREPEND -> return MediatorResult.Success(endOfPaginationReached = true)
                LoadType.APPEND -> {
                    val key = database.remoteKeyDao().getRemoteKeyTable()?.lastOrNull()
                    val nextKey = key?.nextKey
                        ?: return MediatorResult.Success(endOfPaginationReached = key != null)
                    nextKey
                }
            }

            database.withTransaction {
                if (loadType == LoadType.REFRESH) {
                    database.documentDao().clearAllDocuments()
                    database.remoteKeyDao().clearRemoteKeys()
                }
            }

            val documentList = fetchDocumentList(position)

            database.withTransaction {
                val prevKey = if (position == STARTING_POSITION) null else position - 1
                val nextKey = if (documentList.hasMore) position + 1 else null
                val keys = RemoteKeyTable(position = position, prevKey = prevKey, nextKey = nextKey)

                database.documentDao().insertDocuments(documentList.documents.map { it.toTable() })
                database.remoteKeyDao().insertKey(keys)
            }

            return MediatorResult.Success(endOfPaginationReached = !documentList.hasMore)
        } catch (exception: IOException) {
            return MediatorResult.Error(exception)
        } catch (exception: HttpException) {
            return MediatorResult.Error(exception)
        }
    }
```

### Pager 빌더로 PageData flow 생성
`Pager`에 remoteMediator와 pagingSourceFactory를 전달한다.  
```kotlin
@ExperimentalPagingApi constructor(
    config: PagingConfig,
    initialKey: Key? = null,
    remoteMediator: RemoteMediator<Key, Value>?,
    pagingSourceFactory: () -> PagingSource<Key, Value>
) {
    ...
```

pagingSourceFactory는 정렬 타입에 따라 다르게 전달되도록 구현.

```kotlin
@Singleton
internal class DocumentRepositoryImpl @Inject constructor(
    private val searchRemoteMediatorFactory: SearchRemoteMediatorFactory,
    private val documentDao: DocumentDao
) : DocumentRepository {
    override fun fetchDocumentPagingData(param: SearchPagingParam): Flow<PagingData<Document>> {
        @OptIn(ExperimentalPagingApi::class)
        return Pager(
            config = PagingConfig(
                pageSize = SearchRemoteMediator.PER_PAGE_SIZE,
                prefetchDistance = 3

            ),
            remoteMediator = searchRemoteMediatorFactory.create(param)
        ) {
            when (param.sortType) {
                SortType.TITLE -> documentDao.getDocumentByTitle(param.query)
                else -> documentDao.getDocumentByDate(param.query)
            }
        }.flow.map {
            it.map { documentTable ->
                documentTable.toEntity()
            }
        }
    }
}
```

생성한 PagingData는 PagingDataAdapter로 전달한다.