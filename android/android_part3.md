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

