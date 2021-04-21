# Room

## Room 이란 ?
Room은 안드로이드 앱에서 SQLite 데이터베이스를 쉽고 편리하게 사용할 수 있도록 한다. 
Entity, DAO, Room Database 총 세 개의 구성 요소를 통해 Room 데이터베이스 라이브러리가 구성한다


## 구성요소
- Entity : Room으로 작업할 때 테이블을 설명하는 어노테이션이 달린 클래스
- RoomDataBase : SQLite 데이터베이스에 대한 액세스 포인트 역할, 데이터베이스의 전체적인 소유자 역할을 하며 DB를 새롭게 생성하거나 버전을 관리
- DAO<sup>Data Access Object</sup> : 데이터베이스에 접근하여 수행할 작업을 메소드 형태로 정의, SQL 쿼리를 함수에 매핑한다. (select, insert, delete, join...)
![image](https://user-images.githubusercontent.com/39984656/115577616-8d8b2a00-a2ff-11eb-9c84-7d90a1ed1a63.png)


## Room vs SQLite
|| Room | SQLite |
|----|----|----|
|컴파일타임에 쿼리 유효성 검사 | ✅ | ❌ | 
|Schema가 변경될 경우 SQL 쿼리 업데이트| ✅(자동업데이트) | ✅(수동업데이트)|
|보일러플레이트| ❌(상용구 없이 ORM라이브러리를 통해 매핑) | ✅(많은 상용구 필요)|
|LiveData, RxJava, Coroutine 지원| ✅ | ❌|


## 설정
```
dependencies { 
    // Room components
    implementation "androidx.room:room-ktx:$rootProject.roomVersion"
    kapt "androidx.room:room-compiler:$rootProject.roomVersion"
    androidTestImplementation "androidx.room:room-testing:$rootProject.roomVersion"
}
```


## Entity 생성
```kotlin
@Entity(tableName = "word_table")
data class Word(
    @PrimaryKey @ColumnInfo(name = "word") val word: String

)
```

```kotlin
@PrimaryKey*(autoGenerate = true) val id : Int,

@ColumnInfo(name = "word") val word: String
```

- `@Entity(tableName = "word_table")` 
    - `@Entity` 어노테이션이 달린 클래스는 SQLite 테이블을 나타낸다. 
    - 클래스이름과 다른 이름을 원하는 경우 `tableName` 속성을 지정한다
- `@PrimaryKey` 모든 엔티티에는 기본키가 필요하다.
- `@ColumnInfo(name= "word")` 테이블 컬럼이 멤버 변수의 이름과 다른 이름을 원한다면 설정할 수 있다
- PrimaryKey(기본키)에 autoGenerate 속성을 사용하여 고유키를 자동 생성할 수 있다


## DAO
DAO(Data Access Object)에서 SQL 쿼리를 지정하고 메서드 호출로 연결한다
DAO는 인터페이스 또는 추상클래스여야 한다.
기본적으로 쿼리는 별도의 스레드에서 실행되어야 한다.

```kotlin
@Dao
interface WordDao {

    @Query("SELECT * FROM word_table ORDER BY word ASC")
    fun getAlphabetizedWord(): List<Word>

    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun insert(word: Word)

    @Query("DELETE FROM word_table")
    suspend fun deleteAll()
}
```
- `@Query("SELECT * FROM word_table ORDER BY word ASC")` 쿼리 설정
- `@Insert` 어노테이션은 SQL을 제공할 필요가 없다
- `onConflict = OnConflictStrategy.IGNORE` 이미 목록에 있는 단어와 정확히 동일한 경우 새 단어를 무시하는 충돌 전략을 설정한다

[Room DAO 자세히 알아보기](https://developer.android.com/training/data-storage/room/accessing-data.html)


## Room Database 구현
Room Database 클래스는 추상적이고 확장되어야한다.
일반적으로 전체 앱에대해 Room Database의 인스턴스는 하나만 필요하다.

```kotlin
@Database(entities = [Word::class], version = 1, exportSchema = false)
abstract class WordRoomDataBase : RoomDatabase() {

    abstract fun wordDao(): WordDao

    companion object {

        @Volatile
        private var INSTANCE: WordRoomDataBase? = null

        fun getDatabase(context: Context): WordRoomDataBase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    WordRoomDataBase::class.java,
                    "word_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

- WordRoomDataBase 클래스에 데이터베이스가 되도록 `@Database` 어노테이션을 설정한다. 데이터베이스에 속한 엔티티를 선언하고, 버전번호를 설정한다. 
- 마이그레이션을 사용하지 않는다면 빌드 경고를 피하기 위해 `exportSchema`를 false로 설정한다
    - [Room Database 마이그레이션 이해하기](https://medium.com/androiddevelopers/understanding-migrations-with-room-f01e04b07929)
- 데이터베이스는 각 @Dao에 대한 추상메소드를 통해 dao를 노출한다
- 데이터베이스의 인스턴스는 1개만 피요하므로 다중인스턴스를 방지한다 `object`, `synchronized`을 사용하여 싱글톤구현
- `Room.databaseBuilder().build()`를 사용하여 데이터베이스 인스턴스를 생성한다
