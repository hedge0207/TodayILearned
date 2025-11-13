# 퍼미션

- 퍼미션 설정하기

  - 퍼미션
    - 앱의 특정 기능에 부여하는 접근 권한을 의미한다.
    - 앱이 다른 앱이나 안드로이드 시스템에서 보호하는 특정 기능을 이용할 때 퍼미션 사용을 설정해야한다.
    - 마찬가지로 직접 개발한 기능을 다른 앱에서 사용할 수 없도록 보호하고 권한을 얻은 앱에만 허용하고 싶을 때 퍼미션을 설정할 수도 있다.
  - 퍼미션과 관련된 태그
    - 퍼미션과 관련된 태그는 아래와 같으며, manifest 파일에 설정해줘야한다.
    - `<permission>`: 기능을 보호하려는 앱의 manifest 파일에 설정한다.
    - `<uses-permission>`: 보호된 기능을 사용하려는 앱의 manifest 파일에 설정한다.
  - 태그 예시
    - A 앱과  B 앱이 있고, A 앱의 컴포넌트를 B 앱에서 사용하려고 한다고 가정해보자.
    - 만약 A 앱의 컴포넌트에서 퍼미션을 설정했다면, B 앱에서 연동할 때 문제가 발생한다.
    - A 앱의 개발자가 manifest 파일에 `<permission>` 태그로 퍼미션을 설정하면 이를 이용하는 B 앱에서는 이를 사용할 수 없다.
    - B 앱에서 이를 사용하기 위해서는 B 앱의 manifest 파일에 `<uses-permission>` 태그로 해당 퍼미션을 이용하겠다고 설정해야한다.
  - `<permission>` 태그
    - `name`: 퍼미션의 이름으로 퍼미션을 구별하는 식별자 역할을 한다.
    - `label`, `description`: 퍼미션에 관한 설명으로 이 퍼미션을 이용하는 외부 앱에서 권한 인증 화면에 출력할 퍼미션의 정보이다.
    - `protectionLevel`: 보호 수준을 의미한다.

  ```xml
  <permission android:name-"com.example.permission.TEST_PERMISSION"
      android:label="Test Permission"
      android:description="@string/permission_desc"
      android:protectionLevel="dangerous"/>
  ```

  - `protectionLevel`에는 아래와 같은 값을 지정할 수 있다.
    - `normal`: 낮은 수준의 보호. 사용자에게 권한 허용을 요청하지 않아도 된다.
    - `dangerous`: 높은 수준의 보호. 사용자에게 권한 허용을 요청해야한다.
    - `signature`: 같은 키로 인증한 앱만 실행한다.
    - `signatureOrSystem`: 안드로이드 시스템 앱이거나 같은 키로 인증한 앱만 실행한다.
  - `<uses-permission>`
    - `ACCESS_NETWORK_STATE`와 `ACCESS_FINE_LOCATION`은 시스템에 선언된 퍼미션으로 각각 네트워크에 접근하고 사용자 위치를 추적하는데 필요하다.
    - `ACCESS_NETWORK_STATE`는 `protectionLevel`이 `normal`이고, `ACCESS_FINE_LOCATION`은  `dangerous`이다.
    - 단, 아래와 같이 두 개로 설정했지만 앱의 권한 화면을 보면 보호 수준이 `dangerous`로 설정된 퍼미션만 노출된다.

  ```xml
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  ```

  - `permission` 설정
    - manifest 파일에 `<permission>`을 설정했다고 해서 컴포넌트가 보호되지는 않ㄴ느다.
    - `<permission>`을 설정한 다음 이 퍼미션으로 보호하려는 컴포넌트에 적용해야 한다.
    - 퍼미션을 컴포넌트에 적용할 때는 `android:permission` 속성을 이용한다.

  ```xml
  <activity android:name=".OneActivity" android:permission="com.example.TEST_PERMISSION">
      <intent-filter>
          <action android:name="android.intent.action.PICK" />
      </intent-filter>
  </activity>
  ```

  - 위 컴포넌트를 이용하는 곳에서는 manifest 파일에 `<uses-permission>`을 선언해야한다.

  ```xml
  <uses-permission android:name="com.example.permission.TEST_PERMISSION" />
  ```

  - 안드로이드 시스템에서 보호하는 기능알 사용할 때도 manifest 파일에 퍼미션 사용 설정을 해야 하며, 시스템이 보호하는 기능은 대표적으로 아래와 같은 것들이 있다.
    - `ACCESS_FINE_LOCATION`: 위치 정보 접근
    - `ACCESS_NETWORK_STATE`: 네트워크 정보 접근
    - `ACCESS_WIFI_STATE`: 와이파이 네트워크 정보 접근
    - `BATTERY_STATS`: 배터리 정보 접근
    - `BLUETOOTH`: 블루투스 장치에 연결
    - `BLUETOOTH_ADMIN`: 블루투스 장치를 검색하고 페어링
    - `CAMERA`: 카메라 장치에 접근
    - `INTERNET`: 네트워크 연결
    - `READ_EXTERNAL_STORAGE`: 외부 저장소에서 파일 읽기
    - `WRITE_EXTERNAL_STORAGE`: 외부 저장소에 파일 쓰기
    - `READ_PHONE_STATE`: 전화기 정보 접근
    - `SEND_SMS`: 문자 메시지 발신
    - `RECEIVE_SMS`: 문자 메시지 수신
    - `RECEIVE_BOOT_COMPLETED`: 부팅 완료 시 실행
    - `VIBRATE`: 진동 울리기



- 퍼미션 허용 확인

  - 퍼미션은  API 레벨 1버전부터 있었던 내용이지만  API 레벨 23(Android 6) 버전부터 정책이 변경되었다.
    - API 레벨 23 이전에는 앞에서 살펴본 것 처럼 개발자가 manifest 파일에 `<uses-permission>`으로 선언만 하면 보호받는 기능을 앱에서 이용할 수 있었다.
    - 사용자는 앱의 권한 화면에서 이 앱이 어떤 기능을 이용하는지 확인만 할 수 있었기 때문이다.
    - 결국 개발자가 "이 퍼미션을 이용한다."라고 선언만 하면 되는 일종의 신고제였다.
    - 그러나 API 레벨 23 버전부터 허가제로 변경되어, 사용자가 권한 화면에서 이를 거부할 수 있게 되었다.
    - 만약 사용자가 권한 설정에서 특정 퍼미션을 거부하면 `<uses-permission>`을 선언하지 않은 것과 같으며, 앱에서 해당 기능을 사용할 수 없다.
    - 따라서 API 레벨 23 버전부터는 앱을 실행할 때 사용자가 퍼미션을 거부했는지 확인하고, 만약 거부했으면 퍼미션을 허용해달라고 요청해야 한다.
  - 사용자가 퍼미션을 허용했는지 확인하려면 `checkSelfPermission()` 함수를 이용한다.
    - 두 번째 매개변수가 퍼미션을 구분하는 이름이며 결과값은 아래 두 개의 상수 중 하나로 전달된다.
    - `PackageManager.PERMISSION_GRANTED`: 권한을 허용한 경우.
    - `PackageManager.PERMISSION_DENIED`: 권한을 거부한 경우.

  ```kotlin
  open static fun checkSelfPermission(
  	@NonNull context: Context,
      @NonNull permission: String
  ): Int
  ```

  - 퍼미션 허용 여부를 확인하는 예시

  ```kotlin
  val status = ContextCompat.checkSelfPermission(this, "android.permission.ACCESS_FINE_LOCATION")
  if (status == PackageManager.PERMISSION_GRANTED) {
      Log.d("foo", "permission granted")
  } else {
      Log.d("foo", "permission denied")
  }
  ```

  - 사용자에게 퍼미션을 요청할 때는 `ActivityResultLauncher`를 이용한다.
    - 만약 퍼미션을 거부한 상태라면 사용자에게 해당 퍼미션을 허용해 달라고 요청해야한다.
    - `ActivityResultLauncher` 클래스는 액티비티에서 결과를 돌려받아야 할 때 사용하며 대표적으로 퍼미션 허용 요청과 다른 액티비티를 실행하고 결과를 돌려 받을 때이다.
    - `ActivityResultLauncher` 객체는  `registerForActivityResult()` 함수를 호출하여 생성한다.
    - 첫 번째 매개변수는 어떤 요청인지를 나타내는 객체이며, 다양한 요청에 대응하는 서브 클래스들이 있다(퍼미션 허용을 요청할 때는 `RequestPermission` 서브 클래스를 사용한다).
    - 두 번째 매개변수는 결과를 받았을 때 호출되는 콜백이다.

  ```kotlin
  public final <I, O> ActivityResultLauncher<I> registerForActivityResult(
      @NonNull ActivityResultContract<I, O> contract,
      @NonNull ActivityResultCallback<O> callback
  )
  ```

  - 퍼미션 허용 요청 예시
    - `registerForActivityResult` 함수를 사용하여 `ActivityResultLauncher` 객체를 생성하고, `launch()` 함수를 호출하여 요청을 실행한다.
    - 요청 결과는  `registerForActivityResult()` 함수의 두 번째 매개변수로 등록한 콜백으로 전달된다.

  ```kotlin
  val requestPermissionLauncher = registerForActivityResult(
      ActivityResultContracts.RequestPermission()
  ) { isGranted -> 
      if (isGranted) {
          Log.d("foo", "callback, granted")
      } else {
          Log.d("foo", "callback, denied")
      }
  }
  
  requestPermissionLauncher.launch("android.permission.ACCESS_FINE_LOCATION")
  ```







# 다양한 다이얼로그

- Dialog
  - 사용자와 상호작용하는 대화 상자를 의미한다.
  - 대표적인 다이얼로그들은 아래와 같다.
    - 토스트
    - 날ㄷ짜 또는 시간 입력
    - 알림 창
  - 개발자가 직접 커스텀 다이얼로그를 구성하는 것도 가능하다.



- 토스트 메시지 띄우기

  - 토스트(Toast)는 화면 아래쪽에 잠깐 보였다가 사라지는 문자열을 의미한다.
    - 사용자에게 간단한 메시지로 특정한 상황을 알릴 때 사용한다.
    - 토스트를 사용하는 대표적인 예로 사용자가 폰의 뒤로 가기 버튼을 눌러 앱을 종료될 때 "종료하려면 한 번 더 누르세요." 메시지가 출력되는 것을 들 수 있다.
  - 토스트는 `Toast.makeText()` 함수로 만든다.
    - 두 번째 매개변수는 출력할 문자열이며, 세 번째 매개 변수는 토스트가 화면에 출력되는 시간이다.
    - 세 번째 매개 변수는 일반적으로 `Toast.LENGTH_SHORT`, `Toast.LEGNTH_LONG` 상수를 사용하며, 각기 3초, 5초이다.

  ```kotlin
  open static fun makeText(context: Context!, text: CharSequence!, duration: Int): Toast!
  
  open static fun makeText(context: Context!, resId: Int, duration: Int): Toast!
  ```

  - 토스트의 출력은 `show()` 함수를 사용하여 화면에 출력한다.

  ```kotlin
  val toast = Toast.makeText(this, "종료하려면 한 번 더 누르세요.", Toast.LENGTH_SHORT)
  toast.show()
  ```

  - 토스트는 `makeText()` 함수 이외에도 아래의 세터 함수로 만들 수도 있다.

  ```kotlin
  // 화면에 출력될 시간 설정
  open fun setDuration(duration: Int): Unit
  // 화면에 출력될 위치 설정
  open fun setGravity(gravity: Int, xOffset: Int, yOffset: Int): Unit
  // 화면에 출력될 위치 설정
  open fun setMargin(horizontalMargin: Float, verticalMargin: Float): Unit
  // 화면에 출력될 문자열 설정
  open fun setText(resId: Int): Unit
  ```

  - 토스트가 화면에 출력되거나 사라지는 순간을 콜백으로 감지해 특정 로직을 수행하게 할 수도 있다.
    - 이 콜백 기능은 API 레벨 30 버전에서 추가되었다.
    - 토스트의 콜백을 등록하려면 `Toast.Callback` 타입의 객체를 토스트 객체의 `addCallback()` 함수로 등록하면 된다.
    - 토스트가 화면에 출력될 때 `Toast.Callback` 객체의  `onToastShown()` 함수가, 화면에서 사라질 때 `onToastHidden()` 함수가 자동으로 호출된다.

  ```kotlin
  @RequiresApi(Build.VERSION_CODES.R)
  fun showToast() {
      val toast = Toast.makeText(this, "종료하려면 한 번 더 누르세요.", Toast.LENGTH_SHORT)
      toast.addCallback(
          object: Toast.Callback() {
              override fun onToastHidden() {
                  super.onToastHidden()
                  Log.d("foo", "toast hidden")
              }
              
              override fun onToastShown() {
                  super.onToastShown()
                  Log.d("foo", "toast shown")
              }
          })
      toast.show()
  }
  ```




- 날짜 또는 시간 입력 받기

  - Picker 다이얼로그
    - 앱에서 사용자에게 날짜나 시간을 입력받는 데 사용하는 다이얼로그를 피커 다이얼로그라고 한다.
    - 날짜를 입력 받을 때는 `DatePickerDialog`를, 시간을 입력 받을 때는 `TimePickerDialog`를 사용한다.
  - 날짜를 입력받는 데이트 피커 다이얼로그의 생성자는 아래와 같다.
    - 두 번째 매개변수로 `DatePickerDialog.OnDateSetListener` 구현 객체를 등록하면 다이얼로그에서 사용자가 설정한 날짜를 콜백 함수로 얻을 수 있다.
    - 나머지 Int 타입의 매개 변수는 처음에 보이는 날짜이다.
    - `month` 값은 0부터 11까지 지정되며, 0은 1월을 의미한다.

  ```kotlin
  DatePickerDialog(context: Context, listener: DatePickerDialog.OnDateSetListener?, year: Int, month: Int, dayOfMont: Int)
  ```

  - 데이트 피커 다이얼로그 사용 예시

  ```kotlin
  DatePickerDialog(this, object: DatePickerDialog.OnDateSetListener {
      override fun onDateSet(datePicker: DatePicker?, year: Int, month: Int, dayOfMonth: Int) {
          Log.d("foo", "yaer: $year, month: ${month+1}, dayOfMont: $dayOfMonth")
      }
  }, 2025, 8, 16).show()
  ```

  - 시간을 입력 받는 타임 피커 다이얼로그의 생성자는 아래와 같다.
    - 두 번째 매개변수로 `TimePickerDialog.OnTimeSetListener`를 구현한 객체를 지정하면 사용자가 다이얼로그에서 설정한 시간을 얻을 수 있으며 처음에 보일 시간을  Int 타입으로 설정할 수 있다.
    - 마지막 매개 변수로 시간을 24시간과 12시간 형태 중 어떤 것으로 출력할 것인지를 지정한다.
    - false로 지정해 12시간 형태로 출력하면 오전/오후를 선택하는 부분이 출력된다.

  ```kotlin
  TimePickerDialog(context: Context!, listenr: TimePickerDialog.OnTimeSetListener!, hourOfDay: Int, minute: Int, is24HourView: Boolean)
  ```

  - 타임 피커 다이얼로그 사용 예시

  ```kotlin
  TimePickerDialog(this, object: TimePickerDialog.OnTimeSetListener {
      override fun onTimeSet(timePicker: TimePicker?, hourOfDay: Int, minute: Int) {
          Log.d("foo", "time: $hourOfDay, minute: $minute")
      }
  }, 15, 0, true).show()
  ```



- 알림 창 띄우기

  - 안드로이드 다이얼로그의 기본은 `AlertDialog`이다.
    - 알림 창은 단순히 메시지만 출력할 수도 있고 다양한 화면을 출력할 수도 있다.
    - 앞에서 살펴본 데이트 피커와 타임 피커도 `AlertDialog`의 하위 클래스이며 각각의 화면에 데이트 피커와 타임 피커를 출력한 다이얼로그이다.
  - 알림 창은 크게 3가지 영역으로 구분된다.
    - 제목, 내용, 버튼 영역으로 구분된다.
    - 그런데 이 세 영역이 항상 모두 보이는 것은 아니다.
    - 알림 창을 설정할 때 제목과 버튼 정보를 지정하지 않았다면 내용 영역만 출력된다.
  - `AlertDialog.Builder`
    - 알림창의 생성자는 접근 제한자가  protected로 선언되어 객체를 직접 생성할 수 없다.
    - 그 대신 `AlertDialog.Builder`를 제공하므로 이 빌더를 이용해 알림 창을 만든다.
    - 먼저  `AlertDialog.Builder` 객체를 생성하고, 빌더의 세터 함수로 알림 창의 정보를 지정한다.

  ```kotlin
  AlertDialog.Builder(context: Context!)
  ```

  - 아래 함수들은 알림 창의 아이콘, 제목, 내용을 지정하는 함수이다.

  ```kotlin
  open fun setIcon(iconId: Int): AlertDialog.Builder!
  
  open fun setTitle(title: CharSequence!): AlertDialog.Builder!
  
  open fun setMessage(message: CharSequence!): AlertDialog.Builder!
  ```

  - 아래 함수들은 알림 창에 버튼을 지정하는 함수이다.
    - 각 함수의 첫 번째 매개변수는 버튼의 문자열이며, 두 번째 매개변수는 사용자가 버튼을 클릭했을 때 처리할 이벤트 핸들러이다.
    - 만약 버튼을 클릭했을 때 처리할 내용이 없다면 두 번째 매개변수에 null을 대입하며, null을 대입하더라도 버튼을 클릭하면 창이 닫힌다.
    - 알림 창의 버튼은 최대 3개까지만 추가할 수 있으며, 만약 같은 함수를 여러 번 사용하면 버튼은 중복되어 하나만 나타난다.

  ```kotlin
  open fun setPositiveButton(text: CharSequence!, listener: DialogInterface.OnclickListener!): AlertDialog.Builder!
  
  open fun setNegativeButton(text: CharSequence!, listener: DialogInterface.OnclickListener!): AlertDialog.Builder!
  
  open fun setNeutralButton(text: CharSequence!, listener: DialogInterface.OnclickListener!): AlertDialog.Builder!
  ```

  - 버튼 함수를 위와 같이 세 개로 구분하는 이유는 이벤트 핸들러에서 어떤 버튼이 클릭되었는지 구분하기 위해서다.
    - 각 이벤트에 해당하는 이벤트 핸들러를 따로 만들 수도 있지만, 한 알림 창의 버튼 이벤트를 하나의 이벤트 핸들러에서 모두 처리할 수도 있다.
    - 이때 어떤 버튼이 클릭되었는지를 구분해야 하는데, 셋 중 어떤 함수를 사용했는지에 따라 이벤트 핸들러에 전달되는 매개변수 값이 달라서 그 값으로 구분한다.
    - 알림 창을 클릭했을 때 호출되는  `onClick()` 함수의 두 번째 매개변수로 이벤트가 발생한 버튼을 알 수 있다.
    - `setPositiveButton()` 함수로 만든 버튼은 이벤트 구분자가 `DialogInterface.BUTTON_POSITIVE`로 지정된다.
    - `setNegativeButton()` 함수로 만든 버튼은 이벤트 구분자가 `DialogInterface.BUTTON_NEGATIVE`로 지정된다.

  ```kotlin
  val eventHandler = object: DialogInterface.OnClickListener {
      override fun onClick(p0: DialogInterface?, p1: Int) {
          if (p1 == DialogInterface.BUTTON_POSITIVE) {
              Log.d("foo", "positive button click")
          } else if (p1 == DialogInterface.BUTTON_NEGATIVE) {
              Log.d("foo", "negative button click")
          }
      }
  }
  
  // ...
  setPositiveButton("OK", eventHandler)
  setNegativeButton("Cancel", eventHandler)
  ```

  - 알림 창의 내용 영역에는 문자열을 출력하는  `setMessage()`말고도 다영한 함수를 사용이 가능하다.
    - 목록을 제공하고, 목록 중 하나를 선택하는 알림 창을 만들고자 한다면 아래 함수를 이용하면 된다.

  ```kotlin
  open fun setItems(items: Array<CharSequence!>!, listener: DialogInterface.OnClickListener!): AlertDialog.Builder!
  
  open fun setMultiChoiceItems(items: Array<CharSequence!>!, checkItems: BooleanArray!, listener: DialogInterface.OnMultiChoiceClickListener!): AlertDialog.Builder!
  
  open fun setSingleChoiceItems(items: Array<CharSequence!>!, checkedItem: Int, listener: DialogInterface.OnClickListener!): AlertDialog.Builder!
  ```

  - 목록을 출력하는 알림창 예시
    - `setItems()` 함수의 두 번째 매개변수는 항목을 선택할 때의 이벤트 핸들러이며 사용자가 항목을 선택하면  `OnClick()` 함수가 자동으로 호출된다.
    - 사용자가 선택한 항목의 index는 `onClick()` 함수의 두 번째 매개변수로 전달된다.

  ```kotlin
  val items = arrayOf<String>("foo", "bar", "baz", "qux")
  AlertDialog.Builder(this).run {
      setTitle("items test")
      setIcon(android.R.drawable.ic_dialog_info)
      setItems(items, object: DialogInterface.OnClickListener {
          override fun onClick(p: DialogInterface?, p1: Int) {
              Log.d("foo", "선택된 문자열: ${items[p1]}")
          }
      })
      setPositiveButton("닫기", null)
      show()
  }
  ```

  - 체크박스 사용하기
    - `setMultiChoiceItems()` 함수는 다중 선택을 위한 체크박스가 함께 출력되는 항목을 만들어준다.
    - 두 번째 매개변수로 처음 체크 상태를 지정한다.
    - 세 번째 매개변수가 항목을 선택할 때의 이벤트 핸들러이며 사용자가 항목을 선택하는 순간 `onClick()` 함수가 자동으로 호출된다.
    - `onClick()` 함수의 두 번째 매개변수로 선택한 항목의 인덱스가 전달되며 세 번째 매개변수로 체크 상태가 전달된다.

  ```kotlin
  setMultiChoiceItems(items, booleanArrayOf(true, false, true, false), object: DialogInterface.OnMultiChoiceClickListener{
      override fun onClick(p0: DialogInterface?, p1: Int, p2: Boolean) {
          Log.d("foo ${items[p1]}이 ${if(p2) "선택되었습니다." else "해제되었습니다."}")
      }
  })
  ```

  - 라디오 버튼 사용하기
    - `setSingleChoiceItems()` 함수는 하나만 선택할 수 있는 라디오 버튼으로 구성된 항목을 만들어준다.
    - 두 번째 매개변수로 처음 선택할 항목을 지정한다.

  ```kotlin
  setSingleChoiceItems(items, 1, object: DialogInterface.OnClickListener {
      override fun onClick(p0: DialogInterface?, p1: Int) {
          Log.d("foo", "${items[p1]} 이 선택되었습니다.")
      }
  })
  ```

  - 알림 창의 속성을 설정하는 함수들
    - 두 함수 모두 사용자의 행동에 따라 알림 창을 닫을 것인지를 설정한다.
    - `setCancelable()` 함수는 사용자가 기기의 뒤로 가기 버튼을 눌렀을 때 창을 닫는다.
    - `setCanceledOnTouchOutside()` 함수는 알림 창의 바깥 영역을 터치했을 때 매개 변수가  true면 닫고, false면 닫지 않는다(기본 값은  true).
    - `setCancelable()`는 `AlertDialog.Builder` 클래스의 메서드이고, `setCanceledOnTouchOutside()`는 `Dialog` 클래스의 메서드이다.

  ```kotlin
  open fun setCancelable(cancelable: Boolean): AlertDialog.Builder!
  
  open fun setCanceledOnTouchOutside(cancel: Boolean): Unit
  ```

  - 알림 창 속성 예시

  ```kotlin
  AlertDialog.Builder(this).run {
      setTitle("items test")
      setIcon(android.R.drawable.ic_dialog_info)
      setItems(items, object: DialogInterface.OnClickListener {
          override fun onClick(p: DialogInterface?, p1: Int) {
              Log.d("foo", "선택된 문자열: ${items[p1]}")
          }
      })
      setCancelable(false)
      setPositiveButton("닫기", null)
      show()
  }.setCanceledOnTouchOutside(false)
  ```




- 소리와 진동 알림

  - 소리 알림

    - 사용자에게 짧은 소리로 특정한 상황을 알릴 때 사용하는 알림이다.

    - 알림은 자체 녹은한 음원을 쓸 수도 있지만 안드로이드 시스템에 등록된 소리를 사용할 수도 있다.

  - 안드로이드 시스템에 등록된 소리를 이용하는 방법.

    - 안드로이드 시스템은 알림, 알람, 벨소리 등의 소리를 제공하며 이 소리는  `RingtoneManager`로 얻을 수 있다.
    - `RingtoneManager.getDefaultUri()` 메서드를 사용해 소리의 식별값을 얻는다.
    - 소리의 식별값은  `Uri` 객체이며, 이 값을 `RingToneManager.getRingtone()` 메서드의 두 번째 파라미터로 전달하면 소리를 재생하는 `RingTone` 객체를 얻을 수 있다.
    - `RingTone.play()` 메서드를 호출하면 소리가 재생된다.

  ```kotlin
  val notification: Uri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION)
  val ringtone = RingToneManager.getRingtone(applicationContext, notification)
  ringtone.play()
  ```

  - 자체적으로 준비한 소리를 이용하는 방법.
    - 음원 파일은 리소스로 등록해서 사용해야 하며, 리소스 디렉터리는 `res/raw`이다.
    - 음원을 재생하는 클래스는  `MediaPlayer`이며,  `MediaPlayer.start()` 메서드를 호출하면 음원이 재생된다.

  ```kotlin
  val player: MediaPlayer = MediaPlayer.create(this, R.raw.fallbackring)
  player.start()
  ```

  - 진동 알림
    - 앱에서 진동을 울리게 하려면 먼저 manifest 파일에  `<uses-permission>`으로 퍼미션을 얻어야 한다.

  ```xml
  <uses-permission android:name="android.permission.VIBRATE" />
  ```

  - 진동은  `Vibrator` 클래스를 이용한다.
    - `Vibrator` 객체를 얻는 방법이 API 레벨 31(Android 12)부터 변경되었다.
    - 이전 버전에서는 `VIBRATOR_SERVICE`로 식별되는 시스템 서비스를 이용했지만, 31버전부터는 `VIBRATOR_MANAGER_SERVICE`로 식별되는 `VibratorManager`라는 시스템 서비스를 얻고 이 서비스에서  `Vibrator`를 이용해야 한다.

  ```kotlin
  val vibrator = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
      val vibratorManager = this.getSystemService(Context.VIBRATOR_MANAGER_SERVICE) as VibratorManager
      vibratorManager.defaultVibrator;
  } else {
      getSystemService(VIBRATOR_SERVICE) as Vibrator
  }
  ```

  - 시간과 패턴을 지정해서 진동 울리기(API 레벨 1부터 제공하는 함수)
    - API 레벨 1부터 제공한 진동 알림 함수가 26 버전(Android 8)에서  deprecated 되었다.
    - 26 버전에서 새로운 함수를 제공하지만 이전 버전의 기기 사용자를 위해 여전히 사용해야한다.
    - API 레벨 1부터 제공했다가  deprecated 된 진동 알림 함수는 아래와 같다.
    - 첫 번째 함수의 매개 변수는 진동이 울리는 시간을 의미한다.
    - 두 번째 함수는 진동을 반복해서 울리는 함수로 첫 번째 매개변수에는 진동 패턴을 배열로 지정하고, 두 번째 매개변수에는 이 패턴을 얼마나 반복할지를 지정한다.

  ```kotlin
  open fun vibrate(milliseconds: Long): Unit
  
  // 만약 pattern 파라미터에 500, 1000, 500, 2000을 넘기면 0.5초 쉬고 1초 울리고, 0.5초 쉬고 2초 울린다.
  // 두 번째 매개변수를 -1로 지정하면 반복하지 않고 패턴대로 한 번만 진동이 울리고, 0으로 지정하면 코드에서 cancel() 함수로 진동 알림을 끄지 않는 한 패턴대로 계속 울린다.
  open fun vibrate(pattern: LongArray!, repeat: Int): Unit
  ```

  - 진동의 세기까지 지정해 진동 울리기(API 레벨 26부터 제공하는 함수)
    - API 레벨 26부터는 진동 정보를 `VibrationEffect` 객체로 지정할 수 있는 `vibrate()` 함수를 제공한다.
    - `VibrationEffect` 객체로는 진동이 울리는 시간 외에도 진동의 세기까지 제어할 수 있다.

  ```kotlin
  open fun vibrate(vibe: VibrationEffect!): Unit
  ```

  - `createOneShot()` 함수로 `VibrationEffect` 객체를 생성할 수 있다.
    - 이 함수로 만든 `VibrationEffect` 객체를 `vibrate`의 파라미터로 넘기면 `createOneShot()` 함수의 첫 번째 매개변수로 설정한 시간 만큼 진동이 울린다.
    - 두 번째 매개변수를 이용해 진동의 세기를 지정할 수 있으며, 진동의 세기는 0~255 사이의 숫자로 표현한다.
    - 0으로 지정하면 진동이 울리지 않고, 255는 기기에서 지원하는 가장 강한 강도로 울린다.
    - 숫자를 직접 대입해도 되고,  `VibrationEffect.DEFAULT_AMPLITUDE`와 같은 상수를 지정해 기기가 정의한 기본 세기로 진동이 울리게 할 수도 있다.

  ```kotlin
  open static fun createOneShot(milliseconds: Long, amplitude: Int): VibrationEffect!
  ```

  - 진동 예시

  ```kotlin
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
      vibrator.vibrate(
      	VibrationEffect.createOneShot(500, VibrationEffect.DEFAULT_AMPLITUDE))
  } else {
      vibrator.vibrate(500)
  }
  ```

  - `createWaveform()` 함수를 사용하면 반복해서 진동을 울릴 수 있다.
    - 첫 번째 매개변수는 마찬가지로 진동이 울리는 시간 패턴의 배열이다
    - 두 번째 매개변수는 진동 세기 패턴의 배열이다.
    - 세 번째 매개변수는 이 패턴의 반복 횟수이다.

  ```kotlin
  // createWaveform 함수
  open static fun createWaveform(timings: LongArray!, amplitudes: IntArray!, repeat: Int): VibrationEffect!
  
  
  // 예시, 처음 0.5초 동안 진동이 울리지 않고, 1초간 50만큼의 세기로 울리고, 다시 0.5초간 울리지 않고, 다시 2초간 200만큼의 세기로 울린다.
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
      vibrator.vibrate(
      	VibrationEffect.createWaveform(longArrayOf(500, 1000, 500 2000), intArrayOf(0, 50, 0, 200), 		-1))
  } else {
      vibrator.vibrate(longArrayOf(500, 1000, 500, 2000), -1)
  }
  ```

