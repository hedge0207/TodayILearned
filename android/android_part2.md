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

