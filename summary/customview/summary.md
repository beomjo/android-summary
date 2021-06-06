# CustomView

## CustomView란?
안드로이드 기본 위젯 위젯 또는 레이아웃이 요구사항에 맞지않는다면 View 서브클래스를 직접만들 수 있다.  
기존 위젯 또는 레이아웃을 약간만 조정해야 하는 경우 위젯 또는 레이아웃을 서브클래스로 만들고 그 메소드를 재정의 할 수도 있다.

## CustomView 생성하기
1. View or Layout을 확장하는 클래스를 생성한다
2. XML에서 속성과 매개변수를 가져올 수 있는 생성자를 만든다
3. 슈퍼클래스의 일부 메소드를 재정의한다
    - onDraw()
    - onMeasure()
    - onLayout()
    - onSizeChanged()
    - 필요한 다른 on.. 메소드 재정의
4. 새로 작성한 확장 클래스를 사용한다


## CustomView를 만들기 위한 ViewLifeCycle 알아보기  
View가 생성되고, ParentView에 addView가 호출되면 `onAttachToWindow()` 가 호출되고, 크기를 결정하고, 레이아웃을 그리고, Canvas위에 그리는 `onDraw()`까지 호출된다.  

![image](https://user-images.githubusercontent.com/39984656/120372413-20cd6b80-c352-11eb-89b1-b0c5cd654d69.png)


## CustomView 작성하기

### Constructor
모든 CustomView는 생성자에서 출발한다.  
생성자에서 초기화하고, default값 등을 설정한다.  
View는 초기 설정을 쉽게 세팅하기위해 AttributeSet이라는 인터페이스를 지원한다.  
attrs.xml파일(res/valeus/attrs.xml)을 만들어 이것을 부름으로서 뷰의 설정값을 쉽게 설정할 수 있다.  

attrs.xml 파일에 아래와 같이 리소스 작성  
![image](https://user-images.githubusercontent.com/39984656/120375570-eb2a8180-c355-11eb-96a0-7872b3c81c50.png)  
CustomView에 속성을 전달하면 생성자로 전달한다  
![image](https://user-images.githubusercontent.com/39984656/120375722-0f865e00-c356-11eb-916b-617e3d2e1f82.png)   
생성자에서는 아래와 같이 설정한 속성을 불러온다  
```kotlin
 constructor(context: Context, attrs: AttributeSet?) : this(context, attrs, 0){
        val strokeWidth = context.obtainStyledAttributes(attrs, R.styleable.NewAttr)
            .getDimensionPixelSize(
                R.styleable.NewAttr_strokeWidth,
                context.resources.getDimensionPixelSize(R.dimen._2dp)
            )

        val color = context.obtainStyledAttributes(attrs, R.styleable.NewAttr)
            .getColor(R.styleable.NewAttr_strokeColor, Color.WHITE)
    }
```

### onAttachToWindow() 
Parent View가 `addView(childView)`를 호출하고 나서 호출

### onMeasure() - 뷰의 크기 측정
View는 layout안에서 각각 자신의 width나 height을 가진다.  
자신의 width, height를 widthMeasureSpec(부모컨테이너에서 정한 가로), heightMeasureSpec(부모컨테이너에서 정한 세로)라 한다.

- `measure(widthMeasureSpec: Int, heightMeasureSpec: Int)` 호출
    - 부모노드에서 자식노드를 경유하며 실행되며, View의 크기를 알아내기위해 호출
    - 실제 View의 크기를 측정하는것은 아니며, 실제크기는 `onMeasure(int,int)`에서 측정한다
- `onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int)` 를 호출하여 View의 크기를 알아낸다

#### onMeasure 함수 안에서
- View가 원하는 사이즈를 계산
- MeasureSpec 에 따라 mode 를 가져온다
- MeasureSpec의 mode를 체크하여 뷰의 크기를 적용

```kotlin
 override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)

        val widthMode = MeasureSpec.getMode(widthMeasureSpec)
        val widthSize = MeasureSpec.getSize(widthMeasureSpec)
        val heightMode = MeasureSpec.getMode(heightMeasureSpec)
        val heightSize = MeasureSpec.getSize(heightMeasureSpec)

        val width = when (widthMode) {
            MeasureSpec.EXACTLY -> widthSize
            MeasureSpec.AT_MOST -> (paddingLeft + paddingRight + suggestedMinimumWidth).coerceAtMost(widthSize)
            else -> widthMeasureSpec
        }

        val height = when (heightMode) {
            MeasureSpec.EXACTLY -> heightSize
            MeasureSpec.AT_MOST -> (paddingTop + paddingBottom + suggestedMinimumHeight).coerceAtMost(heightSize)
            else -> heightMeasureSpec
        }

        setMeasuredDimension(width, height)
    }
```

**MeasureSpec.AT_MOST** : wrap_content 에 매핑되며 뷰 내부의 크기에 따라 크기가 달라진다  
**MeasureSpec.EXACTLY** : fill_parent, match_parent 로 외부에서 미리 크기가 지정된다  
**MeasureSpec.UNSPECIFIED** : Mode 가 설정되지 않았을 경우. 소스상에서 사이즈를 직접 넣었을 때 사용  

### onLayout() - 뷰의 위치와 크기를 할당
- `layout(changed: Boolean, left: Int, top: Int, right: Int, bottom: Int)` 호출
    - 부모에서 자식노드를 경유할때 실행되며, View와 Child View들의 크기와 위치를 할당한다
    - `measure(int,int)`에 의해 각 뷰에 저장된 크기를 사용하여 위치를 지정한다
    - CustomView가 ParentView일때 주로 쓰인다.
    - 넘어오는 파라미터는 어플리케이션 전체를 기준으로 넘어온다 (**주의**)

### onDraw() - 뷰 그리기
실제로 뷰를 그리는 단계이다.  
  
`onDraw()` 함수는 개발자가 원하는대로 구현할 수 있는 Canvas를 제공한다.  
`onDraw()` 를 오버라이드하고, Canvas에 그리고싶은 애용을 그리면 된다.  
  
Scroll 또는 Swipe를 할때 `onDraw()`가 다시 호출되는데 
`onDraw()`함수는 호출 비용이 크니, 한번 생성한 객체를 재활용하는 로직을 추가해주는것이 좋다.  

```kotlin
 override fun onDraw(canvas: Canvas?) {
        super.onDraw(canvas)

        val width = measuredWidth + 0.0f
        val height = measuredHeight + 0.0f

        val circle = Paint()
        circle.color = this.lineColor
        circle.strokeWidth = 10f
        circle.isAntiAlias = false
        circle.style = Paint.Style.STROKE

        canvas?.drawArc(
            RectF(
                10f, 10f, width - 10f, height - 10f
            ), -90f,
            (this.curValue + 0.0f) / (this.maxValue + 0.0f) * 360, false, circle
        )

        val textp = Paint()
        textp.color = Color.BLACK
        textp.textSize = 30f
        textp.textAlign = Paint.Align.CENTER


        if (System.currentTimeMillis() / 1000 % 2 == 0L) {
            canvas?.drawText(
                "${this.curValue} / ${this.maxValue}",
                (width / 2),
                (height / 2),
                textp
            )
        }
    }
```

### invalidate()
View의 text 또는 color변경 등 단순히 View를 다시 그릴때나,
touch interaction등이 발생할 때 `onDraw()`를 호출하며 View를 다시 그린다.

### requestLayout()
`measure()`부터 호출하여 다시 View를 그린다.  
뷰의 사이즈가 변경되었을때, 그것을 다시 측정해야될 때 호출한다.