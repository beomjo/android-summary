# ViewPager2

## Index
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->
- [ViewPager2](#viewpager2)
- [ViewPager2](#viewpager2)
  - [Index](#index)
  - [개요](#개요)
  - [ViewPager2 새로운 기능](#viewpager2-새로운-기능)
  - [설정](#설정)
  - [어뎁터](#어뎁터)
    - [View를 통해 페이징하는 경우 RecyclerView.Adapter 사용](#view를-통해-페이징하는-경우-recyclerviewadapter-사용)
    - [Fragment를 통해 페이징하는 경우 FragmentStateAdapter 사용](#fragment를-통해-페이징하는-경우-fragmentstateadapter-사용)
  - [TabLayout과 함께 사용](#tablayout과-함께-사용)
  - [중첩 스크롤 지원](#중첩-스크롤-지원)
  - [가짜 드래그(Fake Drag)지원](#가짜-드래그fake-drag지원)

<!-- /code_chunk_output -->


## 개요
기존의 ViewPager에서는 좌우 스크롤링만 가능했었다. 
상하 스크롤링기능을 추가하고 싶다면 새로운 모듈을 만들어서 사용해야하는 번거로움이 생길 뿐더러 
몇몇 기기에서는 스크롤이 버벅거리는 현상이 발생했다.
ViewPager 2에서 위와같은 문제를 개선하여 사용된다.


## ViewPager2 새로운 기능
- RecyclerView기반으로 사용(RecyclerView API 사용 가능)
- 수직스크롤링 지원
- notifyDataSetChanged기능
- 페이지 변경 에니메이션 제어 기능 향상
- 사용하기 편해진 페이지 변경 리스너
- 페이저 아이템(View 또는 Fragment)을 가변적으로 추가 또는 삭제가능


## 설정
```
dependencies {
    implementation "androidx.viewpager2:viewpager2:{version}"
}
```

레이아웃파일 
```xml
<androidx.viewpager2.widget.ViewPager2
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/pager"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```


## 어뎁터

### View를 통해 페이징하는 경우 RecyclerView.Adapter 사용
기존 ViewPager에서 PagerAdapter를 사용한 경우
ViewPager2에서는 RecyclerView Adapter를 사용한다.
```kotlin
class CardViewAdapter : RecyclerView.Adapter<CardViewHolder>() {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CardViewHolder {
        return CardViewHolder(CardView(LayoutInflater.from(parent.context), parent))
    }

    override fun onBindViewHolder(holder: CardViewHolder, position: Int) {
        holder.bind(Card.DECK[position])
    }

    override fun getItemCount(): Int {
        return Card.DECK.size
    }
}

class CardViewHolder internal constructor(private val cardView: CardView) :
    RecyclerView.ViewHolder(cardView.view) {
    internal fun bind(card: Card) {
        cardView.bind(card)
    }
}

```
```kotlin
viewPager.adapter = CardViewAdapter()
```

### Fragment를 통해 페이징하는 경우 FragmentStateAdapter 사용
기존 ViewPager에서 FragmentPagerAdapter 또는 FragmentStatePagerAdapter를 사용한 경우
ViewPager2에서는 FragmentStateAdapter를 사용한다.
```kotlin
 viewPager.adapter = object : FragmentStateAdapter(this) {
            override fun createFragment(position: Int): Fragment {
                return CardFragment.create(Card.DECK[position])
            }

            override fun getItemCount(): Int {
                return Card.DECK.size
            }
        }
```


## TabLayout과 함께 사용
기존 ViewPager에서는 TabLayout이 ViewPager의 하위 요소로 선언되지만
ViewPager 2에서는 TabLayout이 ViewPager와 동등한 수준으로 선언된다.
```xml
 <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <com.google.android.material.tabs.TabLayout
            android:id="@+id/tab_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

        <androidx.viewpager2.widget.ViewPager2
            android:id="@+id/pager"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1" />

</LinearLayout>
```

TabLayout 객체를 ViewPager 객체에 첨부하는 코드를 작성한다. 
TabLayout은 ViewPager와 통합하는 데는 자체 `setupWithViewPager()` 메서드를 사용하지만 
ViewPager2와 통합하는 데는 `TabLayoutMediator` 인스턴스가 있어야 한다.
`TabLayoutMediator` 객체는 TabLayout 객체의 페이지 제목을 생성하는 작업도 처리한다. 따라서 어댑터 클래스가 `getPageTitle()`를 재정의할 필요가 없다.  

```kotlin
tabLayout = findViewById(R.id.tabs)
TabLayoutMediator(tabLayout, viewPager) { tab, position ->
    tab.text = Card.DECK[position].toString()
}.attach()
```


## 중첩 스크롤 지원
NestedScrollableHost를 사용하면
viewPager2와 그 내부 요소의 스크롤 방향이 같을 때, 스크롤이 혼선되는 경우가 있다.
이 때 자식 뷰가 우선적으로 스크롤을 인식할 수 있도록 할 수 있다.

```xml
<!-- item_nested_recyclerviews.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical">

    <TextView
        android:id="@+id/page_title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="@dimen/spacer_large"
        android:layout_marginRight="@dimen/spacer_large"
        android:layout_marginTop="@dimen/spacer_large"
        android:gravity="center"
        android:textAppearance="@style/TextAppearance.AppCompat.Headline"
        tools:text="Page 1" />

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="20dp"
        android:layout_marginRight="20dp"
        android:layout_marginTop="32dp"
        android:text="@string/first_rv"
        android:textStyle="bold" />

    <androidx.viewpager2.integration.testapp.NestedScrollableHost
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp">
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/first_rv"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="#FFFFFF" />
    </androidx.viewpager2.integration.testapp.NestedScrollableHost>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="20dp"
        android:layout_marginRight="20dp"
        android:layout_marginTop="32dp"
        android:text="@string/second_rv"
        android:textStyle="bold" />

    <androidx.viewpager2.integration.testapp.NestedScrollableHost
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginLeft="20dp"
        android:layout_marginRight="20dp"
        android:layout_marginTop="8dp"
        android:layout_weight="1">
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/second_rv"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="#FFFFFF" />
    </androidx.viewpager2.integration.testapp.NestedScrollableHost>

</LinearLayout>
```

```kotlin
class ParallelNestedScrollingActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val viewPager = ViewPager2(this).apply {
            layoutParams = matchParent()
            orientation = ORIENTATION_HORIZONTAL
            adapter = VpAdapter()
        }
        setContentView(viewPager)
    }

    class VpAdapter : RecyclerView.Adapter<VpAdapter.ViewHolder>() {
        override fun getItemCount(): Int {
            return 4
        }

        @SuppressLint("ResourceType")
        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val inflater = LayoutInflater.from(parent.context)
            val root = inflater.inflate(R.layout.item_nested_recyclerviews, parent, false)
            return ViewHolder(root).apply {
                rv1.setUpRecyclerView(RecyclerView.HORIZONTAL)
                rv2.setUpRecyclerView(RecyclerView.VERTICAL)
            }
        }

        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            with(holder) {
                title.text = title.context.getString(R.string.page_position, adapterPosition)
                itemView.setBackgroundResource(PAGE_COLORS[position % PAGE_COLORS.size])
            }
        }

        private fun RecyclerView.setUpRecyclerView(orientation: Int) {
            layoutManager = LinearLayoutManager(context, orientation, false)
            adapter = RvAdapter(orientation)
        }

        class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
            val title: TextView = itemView.findViewById(R.id.page_title)
            val rv1: RecyclerView = itemView.findViewById(R.id.first_rv)
            val rv2: RecyclerView = itemView.findViewById(R.id.second_rv)
        }
    }

    class RvAdapter(private val orientation: Int) : RecyclerView.Adapter<RvAdapter.ViewHolder>() {
        override fun getItemCount(): Int {
            return 40
        }

        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val tv = TextView(parent.context)
            tv.layoutParams = matchParent().apply {
                if (orientation == RecyclerView.HORIZONTAL) {
                    width = WRAP_CONTENT
                } else {
                    height = WRAP_CONTENT
                }
            }
            tv.textSize = 20f
            tv.gravity = Gravity.CENTER
            tv.setPadding(20, 55, 20, 55)
            return ViewHolder(tv)
        }

        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            with(holder) {
                tv.text = tv.context.getString(R.string.item_position, adapterPosition)
                tv.setBackgroundResource(CELL_COLORS[position % CELL_COLORS.size])
            }
        }

        class ViewHolder(val tv: TextView) : RecyclerView.ViewHolder(tv)
    }
}
```


## 가짜 드래그(Fake Drag)지원
다른 구성요소에서 드래그 이벤트를 감지하고 이를 ViewPager2에 위임할 수 있다

```kotlin
class FakeDragActivity : FragmentActivity() {

    private lateinit var viewPager: ViewPager2
    private var landscape = false
    private var lastValue: Float = 0f

    private val isRtl = TextUtilsCompat.getLayoutDirectionFromLocale(Locale.getDefault()) ==
            ViewCompat.LAYOUT_DIRECTION_RTL

    private val ViewPager2.isHorizontal: Boolean
        get() {
            return orientation == ViewPager2.ORIENTATION_HORIZONTAL
        }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_fakedrag)
        landscape = resources.configuration.orientation == Configuration.ORIENTATION_LANDSCAPE

        viewPager = findViewById(R.id.viewPager)
        viewPager.adapter = CardViewAdapter()
        viewPager.isUserInputEnabled = false
        UserInputController(viewPager, findViewById(R.id.disable_user_input_checkbox)).setUp()
        OrientationController(viewPager, findViewById(R.id.orientation_spinner)).setUp()

        findViewById<View>(R.id.touchpad).setOnTouchListener { _, event ->
            handleOnTouchEvent(event)
        }
    }

    private fun mirrorInRtl(f: Float): Float {
        return if (isRtl) -f else f
    }

    private fun getValue(event: MotionEvent): Float {
        return if (landscape) event.y else mirrorInRtl(event.x)
    }

    private fun handleOnTouchEvent(event: MotionEvent): Boolean {
        when (event.action) {
            MotionEvent.ACTION_DOWN -> {
                lastValue = getValue(event)
                viewPager.beginFakeDrag()
            }

            MotionEvent.ACTION_MOVE -> {
                val value = getValue(event)
                val delta = value - lastValue
                viewPager.fakeDragBy(if (viewPager.isHorizontal) mirrorInRtl(delta) else delta)
                lastValue = value
            }

            MotionEvent.ACTION_CANCEL, MotionEvent.ACTION_UP -> {
                viewPager.endFakeDrag()
            }
        }
        return true
    }
}
```

가짜 드래그를 시작한 후 fakeDragBy(float)를사용하여 x 축을 따라 주어진 픽셀 수만큼 ViewPager를 드래그 할 수 있다(음수 값은 왼쪽으로, 양수 값은 오른쪽으로)
