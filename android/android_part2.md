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

