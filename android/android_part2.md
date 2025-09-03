# 뷰 배치하기

- LinearLayout 배치

  - 뷰를 가로나 세로 방향으로 나여하는 레이아웃 클래스이다.
    - `orientation` 속성에 `horizontal`이나 `vertical` 값으로 방향을 지정한다.
    - 설정한 방향대로 뷰가 순서대로 나열된다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
  </LinearLayout>
  ```

  - LinearLayout 역시 뷰이므로 LinearLayout 내에 위치시켜 중첩이 가능하다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="horizontal">
      <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:orientation="vertical">
      </LinearLayout>
  </LinearLayout>
  ```



- `layout_weight` 속성

  - 화면을 배치하다 보면 생기는 여백을 `layout_weight` 속성으로 채우는 것이 가능하다.
    - `layout_weight` 속성을 사용하면 수치를 따로 계산하지 않아도 각 뷰에 설정한 가중치로 여백을 채울 수 있다.
    - 예를 들어 아래와 같이 버튼 두 개가 가로로 표현되는 화면이 있다고 할때, 하나의 뷰에 `layout_weight`을 설정하면 해당 버튼이 여백을 모두 채우는 크기가 된다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="horizontal">
      <Button
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="button1"
          android:layout_weight="1"/>
      <Button
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="button2"/>
  </LinearLayout>
  ```

  - 아래와 같이 복수의 뷰로 여백을 채우는 것도 가능하다.
    - 아래와 같이 설정하면 button1이 1/4만큼, button2가 3/4만큼 여백을 채우게 된다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="horizontal">
      <Button
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="button1"
          android:layout_weight="1"/>
      <Button
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="button2"
          android:layout_weight="3"/>
  </LinearLayout>
  ```

  - 중첩된 레이아웃에서 여백 채우기
    - `layout_weight` 속성은 같은 영역에 있는 뷰끼리만 여백을 나누어 차지한다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
      <LinearLayout
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:orientation="horizontal">
          <Button
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="button1"
              android:layout_weight="1"/>
          <Button
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="button2"
              android:layout_weight="3"/>
      </LinearLayout>
      <Button
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="button3"
          android:layout_weight="1"/>
      <Button
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="button4"/>
  </LinearLayout>
  ```

  - 여백 채우기로 뷰의 크기 설정하기
    - `layout_weight`는 여백을 채우는 속성이지만 뷰의 크기를 설정하는 데 사용하기도 한다.
    - 기본적으로 뷰의 크기가 0이면 아무 것도 나오지 않으므로, 화면 전체가 여백이 된다.
    - 이 때 `layout_weight`를 설정하면 그 여백을 채우게 되어 크기가 자동으로 설정된다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
      <Button
          android:layout_width="match_parent"
          android:layout_height="0dp"
          android:text="button1"
          android:layout_weight="1"/>
      <Button
          android:layout_width="match_parent"
          android:layout_height="0dp"
          android:text="button2"
          android:layout_weight="1"/>
      <Button
          android:layout_width="match_parent"
          android:layout_height="0dp"
          android:text="button3"
          android:layout_weight="1"/>
  </LinearLayout>
  ```




- 뷰의 정렬을 위한 속성

  - `gravity`, `layout_gravity` 속성을 이용하여 뷰를 정렬할 수 있다.
    - 만약 이 속성을 사용하지 않을 경우 기본 값은 left/top으로 왼쪽 위를 기준으로 정렬한다.
    - `gravity`는 뷰의 컨텐츠를 정렬하는 속성이고, `layout_gravity`는 뷰 자체를 정렬하는 속성이다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
      <TextView
          android:layout_width="150dp"
          android:layout_height="150dp"
          android:background="#FF0000"
          android:textSize="15sp"
          android:textStyle="bold"
          android:textColor="#FFFFFF"
          android:text="Hello World!"
          android:gravity="right|bottom"
          android:layout_gravity="center_horizontal"/>
  </LinearLayout>
  ```

  - 레이아웃에 `gravity` 속성 적용하기
    - 레이아웃은 뷰를 배치하는 레이아웃이므로 레이아웃의 `orientation` 속성에 설정한 방향과 같은 방향으로는 `layout_gravity` 속성이 적용되지 않는다.
    - 따라서 레이아웃 내의 뷰의 위치를 조정하기 위해서는 `layout_gravity`를 설정하는 것이 아니라 `gravity`를 설정해야한다.
    - 레이아웃에 속한 뷰는 레이아웃의 컨텐츠이기 때문이다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical"
      android:gravity="center">
      <TextView
          android:layout_width="150dp"
          android:layout_height="150dp"
          android:background="#FF0000"
          android:textSize="15sp"
          android:textStyle="bold"
          android:textColor="#FFFFFF"
          android:text="Hello World!"
          android:gravity="right|bottom"/>
  </LinearLayout>
  ```



- `RelativeLayout`

  - 상대 뷰의 위치를 기준으로 정렬하는 레이아웃 클래스이다.
    - 화면에 이미 출력된 특정 뷰를 기준으로 방향을 지정하여 배치한다.
    - `LinearLayout`과는 달리 자동으로 뷰들을 배치하지 않으며, 별도의 속성을 설정하지 않을 경우 뷰들이 같은 위치에 겹쳐있게 된다.
    - 예를 들어 아래와 같을 경우 Button1과 Button2가 겹쳐져 Button1은 보이지 않게 된다.

  ```xml
  <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_height="match_parent"
      android:layout_width="match_parent">
      <Button
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="Button1"/>
      <Button
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="Button2"/>
  </RelativeLayout>
  ```

  - 아래 속성들을 설정 가능하며, 속성에 입력하는 값은 기준이 되는 뷰의 ID이다.
    - `layout_above`: 기준 뷰의 위쪽에 배치.
    - `layout_below`: 기준 뷰의 아래쪽에 배치.
    - `layout_toLeftOf`: 기준 뷰의 왼쪽에 배치.
    - `layout_toRightOf`: 기준 뷰의 오른쪽에 배치.

  ```xml
  <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_height="match_parent"
      android:layout_width="match_parent">
      <Button
          android:id="@+id/testButton"
          android:layout_width="wrap_content"
          android:layout_height="100dp"
          android:text="Button1"/>
      <Button
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="Button2"
          android:layout_toRightOf="@+id/testButton"/>
  </RelativeLayout>
  ```

  - `align`을 사용하면 상대 뷰의 어느 위치를 기준으로 배치할지를 설정할 수 있다.
    - `layout_alignTop`: 기준 뷰의 위쪽에 맞춤.
    - `layout_alignBottom`: 기준 뷰의 아래쪽에 맞춤.
    - `layout_alignLeft`: 기준 뷰의 왼쪽에 맞춤.
    - `layout_alignRight`: 기준 뷰의 오른쪽에 맞춤.
    - `layout_alignBaseLine`: 기준 뷰의 텍스트 기준선에 맞춤

  ```xml
  <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_height="match_parent"
      android:layout_width="match_parent">
      <Button
          android:id="@+id/testButton"
          android:layout_width="wrap_content"
          android:layout_height="100dp"
          android:text="Button1"/>
      <Button
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="Button2"
          android:layout_toRightOf="@+id/testButton"
          android:layout_alignBottom="@+id/testButton"/>
  </RelativeLayout>
  ```

  - 상위 레이아웃을 기준으로 정렬하게 해주는 속성들도 있다.
    - `layout_alignParentTop`: 부모의 위쪽에 맞춤.
    - `layout_alignParentBottom`: 부모의 아래쪽에 맞춤.
    - `layout_alignParentLeft`: 부모의 왼쪽에 맞춤.
    - `layout_alignParentRight`: 부모의 오른쪽에 맞춤.
    - `layout_centerHorizontal`: 부모의 가로 방향 중앙에 맞춤.
    - `layout_centerVertical`: 부모의 세로 방향 중앙에 맞춤.
    - `layout_centerInParent`: 부모의 가로, 세로 중앙에 맞춤

  ```xml
  <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_height="match_parent"
      android:layout_width="match_parent">
      <Button
          android:id="@+id/testButton"
          android:layout_width="wrap_content"
          android:layout_height="100dp"
          android:text="Button1"/>
      <Button
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="Button2"
          android:layout_alignBottom="@+id/testButton"
          android:layout_alignParentRight="true"/>
  </RelativeLayout>
  ```



- `FrameLayout`

  - 여러 뷰를 겹쳐서 출력하는 레이아웃 클래스이다.
    - 뷰를 추가한 순서대로 겹쳐서 출력한다.
    - 단순히 겹쳐서 출력하는 레이아웃이므로 위치를 조정하는 특별한 속성이 없다.

  ```xml
  <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent">
      <Button
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="BUTTON1" />
      <ImageView
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:src="@mipmap/ic_launcher"/>
  </FrameLayout>
  ```

  - 주로 같은 위치에 여러 뷰를 겹쳐 놓고, 어떤 순간에 하나의 뷰만 출력할 때 사용한다.
    - 따라서 대부분 뷰의 표시 여부를 설정하는 `visibility` 속성을 함께 사용한다.

  ```xml
  <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent">
      <Button
          android:id="@+id/button"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="BUTTON1" />
      <ImageView
          android:id="@+id/image"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:src="@mipmap/ic_launcher"
          android:visibility="invisible"
          android:clickable="true"/>
  </FrameLayout>
  ```

  - 위와 같이 처음에는 `ImageView`를 보이지 않게 설정하고 액티비티 코드에서 원하는 순간에 뷰의 `visibility` 속성을 변경한다.

  ```kotlin
  class MainActivity : AppCompatActivity() {
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          val binding = ActivityMainBinding.inflate(layoutInflater)
          setContentView(binding.root)
          binding.button.setOnClickListener {
              binding.button.visibility = View.INVISIBLE
              binding.image.visibility = View.VISIBLE
          }
          binding.image.setOnClickListener {
              binding.button.visibility = View.VISIBLE
              binding.image.visibility = View.INVISIBLE
          }
      }
  }
  ```

  - 위 코드를 실행하기 위해서는 `build.gradle.kts` 파일에 아래 내용을 추가해야한다.
    - 이후에 `Sync Project with Gradle Files`를 실행한다.

  ```groovy
  android {
      buildFeatures{
          viewBinding = true
      }
  }
  ```



- `GridLayout`

  - 행과 열로 구성된 테이블 화면을 만드는 레이아웃 클래스이다.

    - `orientation` 속성으로 가로나 세로 방향으로 뷰를 나열하는데 줄바꿈을 자동으로 해준다.

    - `rowCount` 속성으로 행의 개수를 설정하고, `columnCount` 속성으로 열의 개수를 설정한다.
    - `orientation`값이 horizontal이면 `columnCount` 속성으로, vertical이면 `rowCount` 속성으로 줄바꿈을 한다.
    - `GridLayout`에 추가한 뷰의 크기는 기본으로 `wrap_content`로 지정되므로 `layout_width`, `layout_height` 속성을 설정하지 않아도 오류가 발생하지 않는다.

  ```xml
  <GridLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="horizontal"
      android:columnCount="3">
      <Button android:text="A" />
      <Button android:text="B" />
      <Button android:text="C" />
      <Button android:text="D" />
      <Button android:text="E" />
  </GridLayout>
  ```

  - `layout_row`, `layout_column`
    - `layout_row`는 뷰가 위치하는 새로 방향 인덱스를, `layout_column`는 뷰가 위치하는 가로 방향 인덱스를 설정한다.

  ```xml
  <GridLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="horizontal"
      android:columnCount="3">
      <Button android:text="A" />
      <Button android:text="B" />
      <Button android:text="C" />
      <Button android:text="D" />
      <Button android:text="E" />
  </GridLayout>
  ```

  - `layout_gravity`
    - `layout_gravity`를 사용하여 크기를 확장할 수 있다.
    - 만약 공간이 충분하다면 여백에 다음 뷰를 넣어 한 칸에 뷰를 2개 표시할 수도 있다.

  ```xml
  <GridLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="horizontal"
      android:columnCount="3">
      <Button android:text="A" />
      <Button android:text="BBBBBBBBBBBBBBBBBBB" />
      <Button android:text="C" />
      <Button android:text="D" />
      <Button android:text="E"
          android:layout_gravity="fill_horizontal"/>
      <Button android:text="F" />
  </GridLayout>
  
  
  <!-- 한 칸에 뷰 2개 표시하기 -->
  <GridLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="horizontal"
      android:columnCount="3">
      <Button android:text="A" />
      <Button android:text="BBBBBBBBBBBBBBBBBBB" />
      <Button android:text="C" />
      <Button android:text="D" />
      <Button android:text="E" />
      <Button android:text="F"
          android:layout_row="1"
          android:layout_column="1"
          android:layout_gravity="right"/>
  </GridLayout>
  ```

  - `layout_columnSpan`, `layout_rowSpan`
    - `layout_columnSpan`은 가로로 열 병합에 사용하고, `layout_rowSpan`은 세로로 행 병합에 사용한다.

  ```xml
  <GridLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="horizontal"
      android:columnCount="3">
      <Button android:text="A"
          android:layout_columnSpan="2"
          android:layout_rowSpan="2"
          android:layout_gravity="fill"/>
      <Button android:text="B" />
      <Button android:text="C" />
      <Button android:text="D" />
      <Button android:text="E" />
      <Button android:text="F" />
  </GridLayout>
  ```




- `ConstraintLayout`

  - 안드로이드 플랫폼이 아니라 androidx에서 제공하는 라이브러리이다.
    - 사용을 위해서는 `build.gradle.kts` 파일의 dependencies에 아래와 같이 선언해야한다.
    - 프로젝트를 만들 때 자동으로 만들어지는 레이아웃 XML 파일을 보면 기본 레이아웃이  `ConstraintLayout`으로 설정된다.

  ```groovy
  dependencies {
      implementation(libs.androidx.constraintlayout)
  }
  ```

  - 레이아웃 편집기에서 레이아웃 구성하기
    - `ConstraintLayout`은 뷰를 상대 위치로 배치하는 `RelativeLayout`과 비슷하지만 더 많은 속성을 제공한다.
    - 그런데 그 많은 속성을  XML에 직접 작성하기는 번거로우므로 안드로이드 스튜디오는 GUI로 레이아웃을 구성할 수 있도록 레이아웃 편집기를 제공한다.
    - 안드로이드 스튜디오에서 레이아웃 XML 파일을 디자인 모드로 열면 레이아웃 편집기가 실행된다.
    - 레이아웃 편집기는 팔레트에서 뷰를 선택해 마우스로 드래그해서 작업 창에 놓는 방식으로 화면을 구성하며, 어트리뷰트 창에서 뷰의 세부 속성을 설정할 수 있다.
    - `ConstraintLayout`에 배치할 뷰를 팔레트에서 드래그 앤 드랍 한 후 component tree에서 둘 사이의 관계를 연결해줘야한다.
    - 그럼 `Attributes` 탭의  `Layout`에 `Constraints Widget`이 나타나는데, 이걸로 제약 조건을 지정할 수 있다.
    - 이 제약 조건으로 레이아웃의 어디에 어느 정도의 여백으로 출력할 것인지, 또는 다른 뷰를 기준으로 어디에 위치시킬지 등을 설정할 수 있다.





# 사용자 이벤트 처리하기

- 터치 이벤트

  - 터치 이벤트를 처리하고 싶다면 액티비티 클래스에 터치 이벤트의 콜백 함수인 `onTouchEvent()`를 선언하면 된다.
    - 콜백 함수란 어떤 이벤트가 발생하거나 특정 시점에 도달했을 때 시스템에서 자동으로 호출하는 함수를 말한다.
    - 액티비티에  `onTouchEvent()` 함수를 오버라이드만 하면 사용자가 이 액티비티 화면을 터치하는 순간 자동으로 호출된다.
    - `onTouchEvent()` 함수에 전달되는 매개변수는 `MotionEvent` 객체이며, 이 객체에 터치의 종류와 발생 지점(좌표)이 담긴다.

  ```kotlin
  class MainActivity : AppCompatActivity() {
      override fun onTouchEvent(event: MotionEvent?): Boolean {
          return super.onTouchEvent(event)
      }
  }
  ```

  - 터치 이벤트의 종류
    - `ACTION_DOWN`: 화면이 눌린 순간의 이벤트
    - `ACTION_UP`: 화면에서 떼어진 순간의 이벤트
    - `ACTION_MOVE`: 화면에 터치한채로 이동하는 순간의 이벤트

  ```kotlin
  class MainActivity : AppCompatActivity() {
      override fun onTouchEvent(event: MotionEvent?): Boolean {
          when (event?.action) {
              MotionEvent.ACTION_DOWN -> {
                  Log.d("foo","Touch down event")
              }
              MotionEvent.ACTION_UP -> {
                  Log.d("foo","Touch up event")
              }
          }
          return super.onTouchEvent(event)
      }
  }
  ```

  - 터치 이벤트 발생 좌표 얻기
    - 터치 이벤트를 처리할 때는 이벤트의 종류 뿐만 아니라 이벤트가 발생한 지점을 알아야 하는 경우도 있다.
    - 이 좌표는 `onTouchEvent()`의 매개 변수인 `MotionEvent` 객체로 얻는다.
    - `x`, `y`: 이벤트가 발생한 뷰 내에서의  x, y 좌표.
    - `rawX`, `rawY`: 전체 화면에서의 x, y 좌표.

  ```kotlin
  class MainActivity : AppCompatActivity() {
      override fun onTouchEvent(event: MotionEvent?): Boolean {
          when (event?.action) {
              MotionEvent.ACTION_DOWN -> {
                  Log.d("foo","Touch down event. x: ${event.x}, rawX:${event.rawX}")
              }
          }
          return super.onTouchEvent(event)
      }
  }
  ```




- 키 이벤트

  - 사용자가 스마트폰의 키를 누르는 순간에 발생한다.
    - 액티비티에서 키 이벤트를 처리하라면 아래와 같은 콜백 함수를 오버라이드해야 한다.
    - `onKeyDown`: 키를 누른 순간의 이벤트.
    - `onKeyUp`: 키를 떼는 순간의 이벤트.
    - `onKeyLongPress`: 키를 오래 누르는 순간의 이벤트.

  ```kotlin
  class MainActivity : AppCompatActivity() {
      override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
          Log.d("foo", "onKeyDown")
          return super.onKeyDown(keyCode, event)
      }
  
      override fun onKeyUp(keyCode: Int, event: KeyEvent?): Boolean {
          Log.d("foo", "onKeyUp")
          return super.onKeyUp(keyCode, event)
      }
  }
  ```

  - 키 이벤트 함수의 첫 번째 매개변수는 키의 코드이며, 이 값으로 사용자가 어떤 키를 눌렀는지 식별할 수 있다.

  ```kotlin
  class MainActivity : AppCompatActivity() {
      override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
          when (keyCode) {
              KeyEvent.KEYCODE_0 -> Log.d("foo", "0")
              KeyEvent.KEYCODE_A -> Log.d("foo", "A")
          }
          return super.onKeyDown(keyCode, event)
      }
  }
  ```

  - 소프트 키보드의 키는 키 이벤트로 처리할 수 없다.
    - 소프트 키보드는 앱에서 글을 입력할 때 화면 아래에서 올라오는 키보드를 의미한다.
    - 오직 하드웨어 키보드에서 발생한 입력만 키 이벤트로 처리할 수 있다.
    - 안드로이드 시스템 버튼(폰 하단에 위치하는 세 개의 버튼) 중 뒤로가기 버튼은 키로 취급되어 처리할 수 있다.
    - 또한 볼륨 조절 버튼 역시 키로 취급해 이벤트를 처리할 수 있다.
    - 그러나 전원, 홈, 오버뷰 버튼은 키로 취급하지 않는다.

  ```kotlin
  class MainActivity : AppCompatActivity() {
      override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
          when (keyCode) {
              KeyEvent.KEYCODE_BACK -> Log.d("foo", "back")
              KeyEvent.KEYCODE_VOLUME_UP -> Log.d("foo", "volume up")
          }
          return super.onKeyDown(keyCode, event)
      }
  }
  ```

  - 뒤로 가기 버튼의 경우 `onKeyDown()`, `onKeyUp()` 함수뿐 아니라  `onBackPressed()` 함수를 사용하여 이벤트를 처리할 수도 있었다.
    - 그러나 이는 Android 13(API 33) 버전부터 depreacted 되었다.

  ```kotlin
  class MainActivity : AppCompatActivity() {
      override fun onBackPressed() {
          Log.d("foo", "back")
      }
  }
  ```

  - 현재는 `androidx.activity.OnBackPressedCallBack()` 함수 사용을 권장하고 있다.

  ```kotlin
  onBackPressDispatcher.addCallback(this, object: OnBackPressedCallback(true){
      override fun handleOnBackPressed() {
          
      }
  })
  ```



- 뷰 이벤트

  - `TextView`, `EditText`, `ImageView`, `Button` 등의 뷰를 사용자가 터치했을 때는 터치 이벤트를 사용하지 않는다.
    - 터치 이벤트를 사용하지 못 하는 것이 아니라 사용하지 않는 것이다.
    - 이러한 뷰의 이벤트를 터치 이벤트로 처리하지 않는 이유는 개발의 편의성을 위함이다.
    - 만약 화면에 버튼과 체크박스가 있을 때 사용자가 체크박스를 터치했다고 가정해보자.
    - 화면에 여러 개의 뷰가 있으므로 사용자가 어느 뷰를 터치했는지를 알아야 한다.
    - 즉 `onTouchEvent()`의 매개 변수를 통해서 사용자가 터치한 지점의 좌표를 얻어서 터치된 뷰가 버튼인지 체크박스인지를 알아내야한다.
    - 만약 뷰가 더 많아진다면 훨씬 더 복잡해질 것이다.
  - 뷰 이벤트의 처리 구조
    - 뷰 이벤트는 `onTouchEvent()` 콜백 함수만 선언하면 처리 되는 터치 이벤트, `onKeyDown()` 콜백 함수만 선언하면 처리 되는 키 이벤트 등과 달리 이벤트 콜백 함수만 선언해서는 처리할 수 없다.
    - 뷰 이벤트는 이벤트 소스와 이벤트 헨들러로 역할이 나뉘며 이 둘을 리스터로 연결해야 이벤트를 처리할 수 있다.
    - 이벤트 소스는 이벤트가 발생한 객체를 의미한다.
    - 이밴트 핸들러는 이벤트 발생 시 실행할 로직이 구현된 객체를 의미한다.
    - 리스너는 이벤트 소스와 이벤트 헨들러를 연결해 주는 함수이다.
    - 즉, 이벤트 소스에서 리스터로 이벤트 헨들러를 등록해 놓으면 이벤트가 발생할 때 실행되는 구조이다.

  ```kotlin
  // checkbox는 이벤트 소스, setOnCheckedChangeListener는 리스너, object은 이벤트 헨들러이다.
  binding.checkbox.setOnCheckedChangeListener(object: CompoundButton.OnCheckedChangeListener {
      override fun onCheckedChanged(buttonView: CompoundButton?, isChecked: Boolean) {
          Log.d("foo", "체크박스 클릭")
      }
  }) 
  ```

  - 이벤트 핸들러
    - 대부분의 이벤트 핸들러는 이름의 형식이 `On*Listener`(e.g. `OnClickListener`, `OnLongClickListener` 등)인 인터페이스를 구현해서 만든다.
    - 위 예시에서는 인터페이스를 구현한 `object` 클래스를 이벤트 핸들러로 만들었지만, 액티비티 자체에서 인터페이스를 구현할 수도 있다.
    - 또한 이벤트 핸들러를 별도의 클래스로 만들어 처리할 수도 있으며, 코틀린의 SAM(Single Abstract Method) 기법을 사용할 수도 있다.
    - 각 예시는 아래와 같다.

  ```kotlin
  // 액티비티에서 인터페이스 구현
  class MainActivity : AppCompatActivity(), CompoundButton.OnCheckedChangeListener {
      override fun onCreate(savedInstanceState: Bundle?){
          super.onCreate(savedInstanceState)
          val binding = ActivityMainBinding.inflate(layoutInflater)
          setContentView(binding.root)
          binding.checkbox.OnCheckedChangeListener(this)
      }
      override fun onCheckedChanged(buttonView: CompoundButton?, isChecked: Boolean) {
          Log.d("foo", "체크박스 클릭")
      }
  }
  
  // 이벤트 헨들러를 별도의 클래스로 구현
  class MyEventHandler : CompoundButton.OnCheckedChangeListener {
      override fun onCheckedChanged(buttonView: CompoundButton?, isChecked: Boolean) {
          Log.d("foo", "체크박스 클릭")
      }
  }
  
  class MainActivity : AppCompatActivity() {
      override fun onCreate(savedInstanceState: Bundle?){
          super.onCreate(savedInstanceState)
          val binding = ActivityMainBinding.inflate(layoutInflater)
          setContentView(binding.root)
          binding.checkbox.OnCheckedChangeListener(MyEventHandler())
      }
  }
  
  // SAM 기법으로 구현
  class MainActivity : AppCompatActivity() {
      override fun onCreate(savedInstanceState: Bundle?){
          super.onCreate(savedInstanceState)
          val binding = ActivityMainBinding.inflate(layoutInflater)
          setContentView(binding.root)
          binding.checkbox.OnCheckedChangeListener {
                  buttonView, isChecked ->
              Log.d("foo", "체크박스 클릭")
          }
      }
  }
  ```

