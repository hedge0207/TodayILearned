# 앱 실행 및 출시

- 앱 실행하기
  - Android Studio를 실행하고 [Phone and Tablet]에서 [Empty Views Activity]를 선택해 프로젝트를 생성한다.
  - 가상 기기에서 실행하기
    - 안드로이드 가상 기기는 AVD(Android Virtual Device)라고 하며, 흔히 에뮬레이터라고 부른다.
    - Android Studio 오른쪽 위에 있는 [Device Manager]를 클릭하면 디바이스 매니저 창이 나타나는데, 이 창에는 안도르이드 개발 환경을 구축하며 자동으로 설정된 AVD 목록을 확인할 수 있다.
    - 새로 추가하는 것도 가능하며, 새로 추가할 경우 하드웨어와 시스템 이미지를 선택해야 하는데, 시스템 이미지는 하드웨어에 실행할 안드로이드 운영체제의 버전을 선택하는 것이다.
    - [Recommended] 탭에는 Play Store 같은 구글의 기본 앱치 설치된 버전이고 [x86 images] 탭에는 나머지 버전이 모두 표시된다.
    - 이 두 탭에 있는 AVD는 인텔의 하드웨어 가속 기능인 HAXM(Hardware Accelerated Execution Manager)을 사용한다.
    - 만약 HAXM을 설치할 수 없는 환경이라면 [Other images]에 있는 시스템 이미지를 사용해야 한다.
  - 제스처 내비게이션
    - AVD를 실행하면 안드로이드 휴대폰에서 흔히 보던 내비게이션 바(이전 화면으로 돌아가기, 홈으로 가기 등)가 보이지 않을 수도 있다.
    - 이 버튼이 보이지 않는 이유는 디바이스 환경 설정에서 시스템 내비게이션 설정이 제스처 네비게이션으로 설정되어 있기 때문으로, 내비게이션 버튼을 손가락 제스처로 대체하는 것이다.
    - 네비게이션 바를 표시하고 싶다면 AVD 화면 내에서[Settings 앱 실행 - System - Navigation mode]에서 설정할 수 있다.
  - 실제 스마트폰에서 실행하기
    - 앱을 개발할 때는 대부분 에뮬레이터에서 테스트할 수 있지만, 최종 배포하기 전에는 실제 기기에서 테스트해 봐야 한다.
    - 또한 에뮬레이터를 실행하기 어려운 환경이거나 성능이 까다로운 앱은 실제 기기에서 테스트하기도 한다.
    - 앱일 실제 스마트폰에서 실행하려면 먼저 컴퓨터와 연결해야한다.
    - macOS에서는 컴퓨터가 스마트폰을 자동으로 인식하지만, Windows에서는 USB 드라이버를 설치해야하며, 이는 스마트폰 제조사 홈페이지에서 설치가 가능하다.
    - 스마트폰에서 USB 디버깅을 허용해줘야 하는데, 이는 스마트폰마다 설정이 다를 수 있다.
  - 앱 실행하기
    - AVD에서 실행하든 실제 스마트폰에서 실행하든 앱을 실행하는 방법은 동일하다.
    - Android Studio 우측 상단에 초록색 우측 화살표 버튼을 누르면 앱이 실행된다.



- 앱 출시 준비

  - 배포 파일
    - 앱을 출시하려면 프로젝트의 여러 파일을 하나로 묶어서 배포해야한다.
    - 사용자가 자신의 안드로이드 스마트폰에 앱을 설치하려면 구글 Play Store나 기타 앱 마켓에서 내려받아야 한다.
    - 이때 사용자는 개발자가 등록한 배포 파일을 내려받게 된다.
    - 안드로이드 앱의 배포 파일은 APK(Andorid Appication Package)와 AAB(Adnroid App Bundle)가 있다.
    - APK는 안드로이드의 전통적인 배포 파일로, 컴파일된 코드와 리소스를 묶어서 키로 서명한 것이다.
    - AAB는 2018년 Google IO에서 발표한 새로운 안드로이드 앱 배포 파일이다.
    - AAB 파일은 Play Store에 올리면 사용자 기기에 맞게 최적화된 APK를 대신 만들어 준다.
    - 즉 사용자 기기에 내려받아 앱을 설치하는 파일은 똑같이 APK지만, 개발자가 직접 APK 파일을 만들지 않고 Play Store에 최적화된 파일을 만들어주는 것이다.
    - 이 때 앱을 내려받는 기기에 맞는 파일만 포함하므로 앱의 크기가 줄어드는 효과가 있다.
    - 즉 AAB 방식으로 등록한 앱을 내려 받는 사용자는 APK 방식과 달리 사용자 기기에 필요한 파일만 내려받게 된다.
  - 앱 서명
    - 안드로이드 배포 파일을 만들려면 키를 만들어 앱에 서명해야 한다.
    - 앱을 서명하는 방법은 크게 두 가지로 나뉘는데 하나는 서명 키를 개발자가 직접 만들어 관리하는 방법이고, 다른 하나는 Google Play에서 관리하는 방법이다.
    - 개발자가 서명 키를 직접 관리하는 방법은 출시용으로 앱을 빌드할 때 개발자가 만든 키로 앱을 서명하고 Play Store에 등록하여 사용자에게 전달한다.
    - 즉 개발자가 만든 서명 키 1개로 앱을 관리하는데, 개발자가 키를 분실하거나 도용될 때 대처할 방법이 없다.
    - 앱을 업데이트 하려면 이전 버전과 똑같은 키로 서명해야 하는데 서명할 수가 없어 업데이트를 하지 못하며, 새로운 키를 만들어 서명하면 완전히 다른 새로운 앱으로 등록된다.
    - 이러한 문제를 해결하고자 Google에서는 Play 앱 서명이라는 서비스를 만들었다.
    - 구글 Play 앱 서명 방식은 키를 2개로 구분한다.
    - 개발자가 만들고 관리하는 업로드 키와 구글 Play가 만드는 앱 서명 키이다.
    - 개발자가 키를 만들어 앱을 서명하지만 이 키는 앱을 구글 Play에 등록할 때만 사용하는 업로드 키이다.
    - 구글 Play는 업로드된 앱을 다시 자체적으로 만든 키로 서명해서 사용자에게 전달한다.
    - 구글 Play의 앱 서명 키는 구글에서 관리하며 개발자가 직접 제어할 수 없다.
    - 이처럼 키가 구분되어 있어 개발자가 키를 분실하거나 도용되더라도 앱을 서명한 키는 안전하므로 앱을 계속 업데이트 할 수 있따.
    - 개발자가 관리하는 키는 다시 만들어 Play Store에 등록하면 된다.

  - Play Store에 앱을 배포하기 전에 준비해야 할 파일은 아래와 같다.
    - AAB 파일: 완성한 앱을 서명한 배포 파일.
    - 앱 아이콘 이미지: Play Store에 표시할 앱 이미지로, 512 * 512px 크기에 1MB 미만으로 JPEG나 32bit PNG 파일을 준비해야한다.
    - 그래픽 이미지: Play Store에서 앱을 프로모션할 때 이용할 이미지로, 1024 * 500px 크기에 15MB 미만으로 JPEG나 24bit PNG 파일을 준비해야한다.
    - 휴대전화 스크린샷: 스마트폰에서 앱을 실행한 스크린샷으로, 230~3840px 크기(비율은 16:9 또는 9:16)에 1개당 8MB 미만 이미지를 2~8개 JPEG나 24비트 PNG 파일로 준비해야 한다.
    - 7/10인치 테블릿 스크린샷: 7인치나 10인치 태블릿에서 앱을 실행한 스크린샷이다.
    - 320~3840px 크기(비율은 16:9 또는 9:16)에 1개당 8MB 미만 이미지를 최대 8개 JPEG나 24비트 PNG 파일로 준비해야 한다.



- AAB 배포 파일 만들기

  - 고유한 패키지명으로 바꾸기
    - 안드로이드 앱은 개발자가 지정한 고유한 패키지명으로 식별된다.
    - 그런데 안드로이드 스튜디오에서 프로젝트를 만들 때 기본인 `com.example`로 시작하는 패키지명은 Play Store에 등록할 수 없다.
    - 따라서 출시용 앱으로 빌드하기 전에 고유한 패키지명으로 변경해야 한다.
    - 패키지명을 변경하려면 모듈 수준의 build.gradle.kts 파일을 수정해야한다.
    - Android Studio의 프로젝트 창을 보면 [Gradle Scripts] 아래 build.gradle.kts(Module:app)이라는 파일이 있는데, 여기서 app 사용자가 지정한 프로젝트명과 모듈명이다.
    - 아 파일을 열어 `applicationId` 항목을 고유한 이름으로 수정하면 된다.
    - 변경한 후에는 편집창 위쪽에 동기화 메뉴가 나타나는데, [Sync now]를 클릭하면 수정한 내용이 적용된다.

  ```groovy
  android {
      # ...
  
      defaultConfig {
          applicationId = "com.cha.firstandroidapp"
          # ...
      }
      # ...
  }
  ```

  - AAB 파일 생성 시작하기
    - [Build - Generate Signed Bundle / APK]를 선택한다/
    - 배포 파일 형식을 선택하는 창이 나오면 [Android App Bundle]을 선택한다.
  - 키 저장소와 서명 키(업로드 키) 만들기
    - 키 저장소는 일종의 인증서라고 생각하면 된다.
    - 키 저장소로 내가 만든 앱에 서명해야 앱을 출시할 수 있다.
    - 이전에 만들어 놓은 키 저장소가 있다면 [Choose existing]을 통해 사용하면 되고, 새로 만들어야 한다면 [Create new]를 통해 생성할 수 있다.
    - 키 저장소 생성 창이 열리면 경로 부분에 디렉터리 아이콘을 눌러 저장할 위치를 선택하고 이름을 입력한다.
    - 이어서 키 저장소 경로 아래 비밀번호를 입력한다.
    - 그리고 키 영역에서 alias와 비밀번호, 기타 키 정보를 입력한 OK를 클릭하면 확장자가 jks인 키 저장소 파일이 만들어진다.
  - 앱 서명하기
    - 서명키를 만든 후에 앱 서명 창으로 다시 돌아가게 되는데, 이 창에서 방금 생성한 키 저장소 비밀번호와 키 alias, 비밀번호를 입력한다.
    - 만일 [Export encrypted key]가 체크되어 있다면 해제하고 Next를 클릭한다.
    - 만약 이미 Play Store에 등록된 앱을 Play 앱 서명으로 다시 등록하려면 [Export encrypted key]를 체크하고 경로를 지정한다.
    - 그러면 서명 키로 암호화된 pepk 파일로 저장되는데, 이 파일로 기존 앱을 Play 앱 서명으로 등록할 수 있다.
  - 릴리즈용 빌드하기
    - AAB 파일을 디버그 용으로 만들 것인지 릴리즈 용으로 만들것인지를 선택한다.
    - [release]를 선택하고 Create를 클릭하면 서명된 AAB 파일로 빌드를 시작한다.
  - 빌드된 AAB 파일 확인하기
    - Android Studio가 빌드를 마치면 프로젝트 루트 디렉터리에 app/release/app-release.aab] 파일이 만들어진다.
    - 파일명에서 app은 프로젝트의 모듈명에 해당한다.



- Google Play에 앱 등록하기
  - 자신의 구글 계정을 개발자 계정으로 등록해야한다.
    - 구글의 개발자 계정으로 등록하려면 25달러의 비용이 든다(구독형이 아닌 최초 1회만 결제하면 된다).
    - [구글 플에이 콘솔](https://play.google.com/console/about/)에 접속한 후 [Play Console로 이동]을 클릭한다.
    - 개발자 계정 만들기 화면으로 이동되며, 개발자 계정을 만드는 데 필요한 정보를 입력하고 [계정 생성 및 결제]를 클릭한다.
  - Play console에서 앱 만들기
    - 앱 만들기 화면에서 각종 설정을 완료한 후 앱을 만든다.
    - 앱 이름, 언어, 유료 여부 등을 설정한다.
  - Play console에서 앱 설정하기
    - 앱을 만든 후 대시보드에서 앱을 게시하는 데 필요한 각종 정보와 설정, 파일 등을 등록할 수 있다.
    - 구글에 안내하는 절차에 따라 앱에 대한 정보를 입력한다.
  - Google Play에 앱 게시하기
    - 프로덕션 화면에서 [새 버전 만들기]를 클릭한다.
    - 서명 관련 설정을 완료화고 준비된 AAB 파일을 업로드하여 마무리한다.
    - 이후 구글에서 심사를 거쳐 Google Play에 등록된다.





 

# 안드로이드 앱의 기본 구조

- 안드로이드(Android)

  - 리눅스 커널을 기반으로 구글에서 제작한 모바일 운영체제.

    - 2008년 버전 1이 출시되었다.
    - 전 셰계 모바일 플랫폼 시장의 70~80%를 차지하고 있다.

  - 특징

    - 공개 운영체제인 리눅스를 기반으로 한다.
    - 안드로이드에서 실행되는 앱은 자바나 코틀린을 이용해 개발한다.
    - 안드로이드 운영체제의 주요 부분과 라이브러리, 구글에서 만든 앱 등의 코드는 대부분 공개되어 있다.
    - 안드로이드 플랫폼에서는 모든 애플리케이션이 평등하다는 사상을 바탕으로, 모바일에 기본으로 탑재된 앱ㄱ과 개발자가 만든 앱이 똑같은 환경에서 똑같은 API를 이용한다.

  - 안드로이드 운영체제의 구조

    - 리눅스 커널
    - 하드웨어 추상화 레이어(Hardware Abstraction Layer, HAL): Java API 프레임워크에서 하드웨어 기능을 이용할 수 있게 표준 인터페이스를 제공한다.
    - 안드로이드 런타임(ART): 안드로이드 앱은 DEX 파일로 빌드되는데, 이 파일을 해석해서 앱을 실행하는 역할을 한다.

    - 네이티브 C/C++ 라이브러리: 안드로이드 앱은 대부분 Java 프레임워크로 개발하지만, 네이티브 C/C++ 라이브러리를 사용할 수도 있는데, 이를 안드로이드 NDK(Native Development Kit)라 한다.
    - Java API 프레임워크: 앱을 개발할 때 사용하는 Java API.



- 컴포넌트 기반 개발
  - 안드로이드 앱 개발의 핵심은 컴포넌트이다.
    - 컴포넌트란 애플리케이션의 구성 요소를 말하는데, 하나의 애플리케이션은 여러 컴포넌트로 구성된다.
  - 안드로이드에서는 클래스로 컴포넌트를 개발한다.
    - 즉 하나의 클래스가 하나의 컴포넌트가 된다.
    - 그러나 애플리케이션을 구성하는 모든 클래스가 컴포넌트라는 것은 아니다.
    - 앱은 여러 클래스로 구성되는데 크게 컴포넌트 클래스와 일반 클래스로 구분된다.
    - 두 종류의 클래스는 런타임 때 생명주기를 누가 관리하느가에 따라 구분된다.
    - 앱이 실행될 때 클래스의 객체 생성부터 소멸까지 생명 주기를 개발자 코드에서 한다면 일반 클래스이다.
    - 반면에 개발자가 만들기는 했지만 생명주기를 안드로이드 시스템에서 관리한다면 컴포넌트 클래스이다.

  - 컴포넌트의 종류
    - 액티비티: 화면을 구성하는 컴포넌트로, 앱의 화면을 안드로이드폰에 출력하려면 액티비티를 만들어야하며, 앱이 실행되면 액티비티에서 출력한 내용이 안드로이드폰에 나온다.
    - 서비스: 백그라운드 작업을 하는 컴포넌트로, 화면 출력 기능이 없어 서비스기 실행되더라도 화면에는 출력되지 않는다.
    - 콘텐츠 프로바이더: 앱의 데이터를 공유하는 컴포넌트로, 앱 간에 공유하는 데이터를 주고 받게 해주는 역할을 한다.
    - 브로드캐스트 리시버: 시스템 이벤트가 발생할 때 실행되게 하는 컴포넌트로, 여기서 이벤트는 사용자 이벤트가 아니라 시스템에서 발생하는 특정 상황을 의미한다(e.g. 부팅 완료, 배터리 방전 등).
  - 4가지 컴포넌트를 구분하는 방법
    - 컴포넌트는 앱이 실행될 때 안드로이드 시스템에서 생명주기를 관리하는 클래스지만 개발자가 만들어야 하는 클래스이다.
    - 개발자가 컴포넌트 클래스를 만들 때는 지정된 클래스를 상속받아야 하는데 이 상위 클래스를 보고 구분할 수 있다.
    - 액티비티는 `Activity` 클래스를 상속 받아 만들고, 서비스는 `Service`, 컨텐츠 프로바이더는 `ContentProvider`, 브로드캐스트 리시버는 `BroadcastReceiver` 클래스를 상속받아서 만든다.
  - 앱을 개발할 때 컴포넌트를 어떻게 구성해야 하는가
    - 컴포넌트는 개발자가 만들고자 하는 앱의 기능과 화면 등을 고려해 필요한 만큼 구성하면 된다.
    - 앱을 만들 때 어떤 컴포넌트를 어떻게 구성하는지는 설계에 따라 달라지며 정해진 규칙은 없다.
    - 심지어 액티비티가 없어 사용자에게 화면을 제공하지 않는 앱도 개발할 수 있다.
  - 컴포넌트는 앱 안에서 독립된 실행 단위다.
    - 독립된 실행 단위란 컴포넌트끼리 서로 종속되지 않아서 코드 결합이 발생하지 않는다는 의미이다.
    - 채팅 앱을 개발하면서 채팅방 목록 화면을 `ListActivity`, 채팅 화면을 `ChatActivity`라는 클래스명으로 작성했다고 가정해보자.
    - `ListActivity`에서 `ChatActivity`를 실행해야 하므로 `ListActivity`에서 `ChatActivity` 객체를 생성하여 실행하면 될 것 같지만, 안드로이드에서 이 방법은 불가능하다.
    - 컴포넌트의 생명 주기를 안드로이드 시스템에서 관리하므로 코드에서 직접 객체를 생성해 실행할 수 없기 때문이다.
    - `ListActivity`에서 `ChatActivity`를 실행해야한다면 안드로이드 시스템에 의뢰해서 시스템이 `ChatActivity`를 실행해야 한다.
    - 즉 `ListActivity`와 `ChatActivity`를 결합해서 직접 실행하는 것이 아니라 안드로이드 시스템에 의뢰해 두 클래스가 서로 종속되지 않고 독립해서 실행되게 해야 한다.
  - 앱 실행 시점이 다양하다.
    - 컴포넌트가 앱 내에서 독립해서 실행되는 특징 덕분에 앱의 실행 시점이 다양할 수 있다.
    - 앱의 첫 화면이 `ListActivity`라고 가정하면 앱을 실행하면 `ListActivity`부터 실행되어 화면에 출력되며, 이후 채팅방을 클릭하여 채팅 화면으로 넘어간다.
    - 하지만 앱은 사용자가 직접 실행하지 않아도 실행될 수 있다.
    - 예를 들어 사용자가 알림 창에서 메시지 수신 알림을 터치하면 채팅 리스트를 거치지 않고 바로 채팅 화면이 열린다.
    - 이처럼 앱의 실행 시점은 다양할 수 있다.
    - 이 때문에 안드로이드 앱에는 메인 함수 개념이 없다고 말한다.
    - 메인 함수란 앱의 단일 시작점을 의미하는데 안드로이드 앱은 실행 시점이 다양해서 메인 함수 개념이 없다고 표현한다.
  - 애플리케이션 라이브러리를 사용할 수 있다.
    - 애플리케이션 라이브러리란 다른 애플리케이션을 라이브러리처럼 이용하는 것을 말한다.
    - 슬랙을 예로 들면 채팅 화면에서 갤러리 앱을 사용해 채팅에 공유할 수 있다.
    - 이는 슬랙이 갤러리 앱을 라이브러리처럼 이용한 것이다.



- 리소스 활용 개발

  - 안드로이드 앱 개발의 또 다른 특징은 리소스를 많이 활용한다는 점이다.
    - 리소스란 코드에서 정적인 값을 분리한 것이다.
    - 앱에서 발생하는 데이터나 사용자 이벤트에 따른 동적인 값이 아니라 항상 똑같은 값이라면 굳이 코드에 담지 않고 분리해서 개발한다.
    - 이를 통해 코드가 짧아져 개발 생산성과 유지 보수성이 향상된다.
  - 대표적인 예가 문자열을 리소스로 이용하는 것이다.
    - 만약 화면에 "Hello World!"를 출력한다면 아래와 같이 코드를 작성할 것이다.

  ```html
  // 일반 텍스트
  textView.text = "Hello World!"
  
  // 리소스로 등록하기
  <string name="mytxt">Hello World!</string>
  
  // 리소스 사용하기
  text.View.text = resources.getString(R.string.mytxt)
  ```





## 모듈의 폴더 구성

- Gradle 빌드 설정 파일

  - `build.gradle.kts`이 gradle 빌드 설정 파일이다.

    - 안드로이드 스튜디오로 프로젝트를 생성하면 두 개의 파일이 생성된 것을 확인할 수 있는데, 하나는 프로젝트 수준의 파일이고, 하나는 모듈 수준의 파일이다.
    - 모듈은 앱을 의미하므로 대부분의 빌드 설정은 모듈 수준의 gradle 파일에 작성한다.

  - 설정 파일 예시

    - `compileSdk`에는 앱을 컴파일하거나 빌드할 때 적용할 android SDK 버전을 의미한다.
    - `applicationId`는 앱의 식별자를 의미하며, 만약 구글 플레이스토어에 동일한 식별자를 이미 사용하고 있는 앱이 있다면 등록되지 않으며, 스마트폰에 동일한 식별자를 사용하는 앱이 설치되어 있다면 설치되지 않는다.
    - `targetSdk`는 개발할 때 적용되는 SDK 버전이다.
    - `minSdk`는 이 앱을 설치할 수 있는 기기의 최소 SDK 버전을 의미한다. 

    - `versionCode`, `versionName`는 앱의 버전을 의미한다.

  ```groovy
  // 플러그인 선언
  plugins {
      alias(libs.plugins.android.application)
      alias(libs.plugins.kotlin.android)
  }
  
  android {
      namespace = "com.example.firstandroidapp"
      compileSdk = 34		// 컴파일 버전 설정
  
      defaultConfig {
          applicationId = "com.example.androidlab"	// 앱의 식별자
          minSdk = 24
          targetSdk = 34
          versionCode = 1
          versionName = "1.0"
  
          // ...
      }
  
      // ...
      // 개발 언어의 버전을 설정한다.
      compileOptions {
          sourceCompatibility = JavaVersion.VERSION_11
          targetCompatibility = JavaVersion.VERSION_11
      }
      kotlinOptions {
          jvmTarget = "11"
      }
  }
  
  // 앱에서 사용하는 라이브러리의 버전을 설정한다.
  dependencies {
      implementation(libs.androidx.core.ktx)
      implementation(libs.androidx.appcompat)
      implementation(libs.material)
      implementation(libs.androidx.activity)
      implementation(libs.androidx.constraintlayout)
      testImplementation(libs.junit)
      androidTestImplementation(libs.androidx.junit)
      androidTestImplementation(libs.androidx.espresso.core)
  }
  ```



- API 레벨 호환성

  - `build.gradle` 파일에 섲렁하는 `targetSdk`와 `minSdk`는 API 레벨 호환성과 관련된 설정이다.
    - 아래와 같이 설정했다면 34버전의  API로 앱을 개발(`targetSdk`)한다는 의미이며, 24 버전 기기부터 설치할 수 있다(`minSdk`)는 의미이다.
    - 따라서 34 버전의 API로 개발하지만, 24 버전에서도 오류 없이 동작해야한다.
    - 따라서 앱을 개발할 때  `minSdk` 설정값보다 상위 버전에서 제공하는 API를 사용한다면 호환성을 고려해야한다.
    - 안드로이드 API 문서를 보면 `Added in API level n`과 같은 문구를 볼 수 있는데, 이는 버전 호환성을 위한 정보로, 만약 명시된 버전 보다 낮은 버전에서 사용할 경우 에러가 발생한다.

  ```groovy
  minSdk 24
  targetSdk 34
  ```

  - API 호환성에 문제가 있는 API를 사용할 때는 `@`로 시작하는 어노테이션을 추가해 에러를 해결할 수 있다.
    - API 레벨 호환성이 문제가 있는 API를 사용한 함수나 클래스 선언부 위에 `@RequirsesApi` 어노테이션응 추가하면 안드로이드 스튜디오에서 에러가 발생하지 않는다.
    - `@RequiresApi` 대신 `@TargetApi`를 사용해도 된다.
    - 단, 이는 안드로이드 스튜디오에서 에러를 무시하는 설정일 뿐이며, 앱이 실행될 때 API 레벨 호환성 문제를 막으려면 직접 코드로 처리해야한다.

  ```kotlin
  @RequiredApi(Build.VERSION_CODES.S)
  fun foo() {
      // ...
  }
  ```



- 메인 환경 파일

  - `AndroidManifest.xml`은 안드로이드 앱의 메인 환경 파일이다.
    - 안드로이드 시스템은 이 파일에 설정한 대로 사용자의 폰에서 앱을 실행한다.
    - 즉, 매니페스트 파일은 개발부터 실행까지 중요한 역할을 한다.
  - 네임스페이스 선언
    - `<manifest>`는 매니페스트 파일의 루트 태그이다.
    - `xmlns`는 XML의 네임스페이스 선언이며 `http://schemas.android.com/apk/res/android`는 안드로이드 표준 네임스페이스이다.

  ```xml
  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:tools="http://schemas.android.com/tools">
  ```

  - 애플리케이션 태그
    - `<application>` 태그는 앱 전체를 대상으로 하는 설정이다.
    - 앱의 아이콘을 설정하는 `icon` 속성이 있는데 이곳에 지정한 이미지가 앱을 설치한 사용자의 폰에 보이는 실행 아이콘이다.
    - `label` 속성에는 앱의 이름을 등록한다.
    - `theme` 속성은 앱에 적용할 테마를 설정한다.
    - XML의 속성값이 @로 시작하면 리소스를 의미한다.

  ```xml
  <application
          android:allowBackup="true"
          android:dataExtractionRules="@xml/data_extraction_rules"
          android:fullBackupContent="@xml/backup_rules"
          android:icon="@mipmap/ic_launcher"
          android:label="@string/app_name"
          android:roundIcon="@mipmap/ic_launcher_round"
          android:supportsRtl="true"
          android:theme="@style/Theme.FirstAndroidApp"
          tools:targetApi="31">
  ```

  - 액티비티 선언
    - 안드로이드 컴포넌트는 시스템에서 생명주기를 관리하는데, 시스템은 매니페스트 파일에 있는 대로 앱을 실행한다.
    - 결국 컴포넌트는 매니페스느 파일에 등록해야 시스템이 인지한다.
    - 액티비티를 등록할 때 `name`은 필수 속성이며, 클래스 이름을 등록하면 된다.

  ```xml
  <activity
      android:name=".MainActivity"
      android:exported="true">
      <intent-filter>
          <action android:name="android.intent.action.MAIN" />
  
          <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
  </activity>
  ```

  - 각 컴포넌트별 태그
    - 액티비티: `<activity>`
    - 서비스: `<service>`
    - 컨텐츠 프로바이더: `<provider>`
    - 리시버: `<receiver>`



- 리소스 폴더
  - res 폴더는 앱의 리소스를 등록하는 목적으로 사용한다.
    - 앱이 만들어지면 res 폴더 아래 아래와 같은 폴더가 기본으로 생성된다.
    - `drawable`: 이미지 리소스
    - `layout`: UI 구성에 필요한 XML 리소스
    - `mipmap`: 앱 아이콘 이미지
    - `values`: 문자열 등의 값으로 이용되는 리소스
  - `res` 폴더 아래에 리소스를 만들면 자동으로 R.java 파일에 상수 변수로 리소스가 등록된다.
    - 코드에서는 이 상수 변수로 리소스를 이용한다.
    - 이 파일은 개발자가 만드는 파일이 아니라 res 폴더에 있는 리소스를 보고 자동으로 만들어진다.
    - 최신 버전의 안드로이드 스튜디오에서는 R.java 파일을 보여주지 않는다.
    - R.java 파일에 각 폴더의 하위 클래스가 만들어지고 그 안에 파일명을 기준으로 int 형 변수가 자동으로 생성된다.
  - 안드로이드 리소스 파일이 R.java 파일에 상수 변수로 등록되어 이용되면서 다음과 같은 규칙이 생긴다.
    - res 하위의 폴더명은 지정된 폴더명을 사용해야 한다.
    - 각 리소스 폴더에 다시 하위 폴더를 정의할 수 없다.
    - 리소스 파일명은 자바의 이름 규칙을 위배할 수 없다.
    - 리소스 파일명에는 알파벳 대문자를 사용할 수 없다.



- 레이아웃 XML 파일

  - `res/layout` 폴더 아래에 기본으로 만들어지는 `activity_main.xml` 파일은 화면을 구성하는 레이아웃 XML 파일이다.

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      tools:context=".MainActivity">
  
      <TextView
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="Hello World!"
          app:layout_constraintBottom_toBottomOf="parent"
          app:layout_constraintEnd_toEndOf="parent"
          app:layout_constraintStart_toStartOf="parent"
          app:layout_constraintTop_toTopOf="parent" />
  
  </androidx.constraintlayout.widget.ConstraintLayout>
  ```

  - `<TextView>`는 화면에 문자열을 출력하는 역할을 한다.







# 뷰를 이용한 화면 구성

## 화면을 구성하는 방법

- 액티비티 뷰 구조
  - 안드로이드 앱의 기본 구조는 컴포넌트를 기반으로 한다.
    - 즉 안드로이드 앱은 액티비티, 서비스, 브로드캐스트 리시버, 콘텐츠 프로바이더와 같은 컴포넌트를 조합해서 만든다.
    - 이 중에서 화면을 출력하는 컴포넌트는 액티비티뿐이므로, 앱에서 화면을 출력하고 싶다면 액티비티를 만들어야 한다.
  - 뷰 클래스
    - 액티비티는 화면을 출력하는 컴포넌트일 뿐이지 그 자체가 화면은 아니다.
    - 별도로 화면 구성을 하지 않고 단순히 액티비티만 실행하면 텅 빈 흰색 화면만 보인다.
    - 화면에 내용을 표시하려면 뷰 클래스를 이용해 구성해야한다.
    - 예를 들어 화면에 문자열을 출력하려면 TextView 클래스를 사용하고, 이미지를 출력하려면 ImageView 클래스를 이용한다.
    - 이런 클래스들을 뷰 클래스라 한다.
  - 액티비티에서 뷰로 화면을 구성하는 방법은 2가지이다.
    - 액티비티 코드로 작성하는 방법과 레이아웃 XML 파일로 작성하는 방법이다.
    - 액티비티 코드로 작성하는 방법은 화면을 구성하는 뷰 클래스를 액티비티 코드애서 직접 생성한다.



- 뷰 클래스의 기본 구조
  - 뷰 객체의 계층 구조
    - 액티비티 화면을 구성할 때 사용하는 클래스는 모두 `View`의 하위 클래스이다.
    - 이 때문에 화면 구성과 과련된 클래스를 통칭하여 뷰 클래스라고 부른다.
  - `ViewGroup`
    - `View`의 하위 클래스지만 자체 UI는 없어서 화면에 출력해도 아무것도 나오지 않는다.
    - 다른 뷰 여러 개를 묶어서 제어할 목적으로 사용하며, 일반적으로 컨테이너 기능을 담당한다고 이야기한다.
    - 직접 사용하지는 않고, `ViewGroup`을 상속 받는 레이아웃 클래스(`LinearLayout`, `RelativeLayour` 등)를 사용한다.



- 레이아웃 클래스

  - 아래와 같이 작성한 레이아웃 XML 파일을 액티비티 화면에 출력해도 아무것도 나오지 않는다.
    - `ViewGroup` 클래스의 자식 클래스들은 화면에 출력할 용도가 아니라 다른 뷰 객체 여러 개를 담아서 한꺼번에 제어할 목적으로 사용하는 것이기 때문이다.
    - 레이아웃을 이용해 뷰 객체를 계층으로 묶으면 한거번에 출력하거나 정렬하는 등 편하게 제어할 수 있다.

  ```xml
  <LinearLayout xmlns:android="http;//schemas.android.com/apk/res/android"
        android:layout_widht="match_parent"
        android:layout_height="match_parent"
        android:orientation="verical">
  </LinearLayout>
  ```

  - 아래와 같이 레이아웃 클래스에 다른 뷰를 포함시켜 화면을 구성한다.
    - 버튼 객체를 생성하고 이 객체를 `LinearLayout`에 추가한다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
      <Button
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="확인"/>
  </LinearLayout>
  ```

  - 레이아웃 중첩
    - 레이아웃 객체를 중첩해서 복잡하게 구성할 수도 있다.
    - 이처럼 객체를 계층 구조로 만들어 이용하는 패턴을 컴포지트 패턴 또는 문서 객체 모델(document object model)이라 한다.

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
      <Button
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:layout_marginTop="16dp"
          android:text="버튼1"/>
      <LinearLayout
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:orientation="horizontal">
          <Button
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="버튼2"/>
  	</LinearLayout>
  </LinearLayout>
  ```



- 레이아웃 XML의 뷰를 코드에서 사용하기

  - 화면 구성을 위와 같이 레이아웃 XML 파일에 작성하고, 액티비티에서 `setContetView()` 함수로 XML 파일을 지정하면 화면을 출력한다.

  ```kotlin
  class MainActivity: AppCompatActivity() {
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_main)
      }
  }
  ```

  - 식별자 부여하기
    - 때로는 XML에 선언한 객체를 코드에서 사용해야 할 때가 있는데, 이를 위해서는 각 개체에 식별자를 부여하고, 코드에서 그 식별자로 객체를 얻어 와야 한다.
    - 식별자를 부여하기 위해 사용하는 속성이 `id`로,  `android:id="@+id/<identifier>"` 형태로 추가하면 된다.
    - 식별자는 앱에서 유일해야 한다.
    - XML 속성 값이 `@`로 시작하면 R.java 파일을 의미하는 것으로, 식별자를 설정하면 R.java 파일에 식별자가 상수 변수로 추가된다.
    - 식별자의 이름은 일반적으로 `<type>_<purpose>` 형태로 명명한다.

  ```xml
  <TextView
  	android:id="@+id/text_title"
      android:layour_witdh="wrap_content"
      android:layout_height="wrap_content"
      android:text="Hello World!" />
  ```

  - 코드에서 식별자로 객체 가져오기
    - R.java 파일의 상수 변수를 통해 객체를 가져온다.
    - `findViewById()`함수를 사용하면 된다.

  ```kotlin
  setContentView(R.layout.activity_main)
  val textView: TextView = findViewById(R.id.text_title)
  ```

  - 뷰 객체의 타입을 제네릭으로 명시하는 것도 가능하다.

  ```kotlin
  setContentView(R.layout.activity_main)
  val textView = findViewById<TextView>(R.id.text_title)
  ```



- 뷰의 크기 설정

  - 뷰의 크기를 설정하는 속성은 두 가지가 있다.
    - `layout_width`: 뷰의 가로 길이
    - `layout_height`: 뷰의 세로 길이
  - 이 속성들에는 아래 세 가지 중 하나를 설정할 수 있다.
    - 수치: `100px`처럼 수치로 지정할 수 있다. 수치는 px, dp 등의 단위를 사용하고, 수치를 생략하는 것은 불가능하다.
    - `match_parent`: 부모의 크기 전체를 의미한다.
    - `wrap_content`: 자신의 컨텐츠를 화면에 출력할 수 있는 적절한 크기를 의미한다.

  ```xml
  <TextView
      android:layour_witdh="100px"
      android:layout_height="wrap_content"
      android:text="Hello World!" />
  ```



- 뷰의 간격 설정

  - 뷰의 간격은 아래 두 속성으로 설정한다.
    - `margin`: 뷰와 뷰 사이의 간격을 설정한다.
    - `padding`: 뷰와 컨텐츠 테두리 사이의 간격을 설정한다.
    - 모든 방향에 동일한 간격이 설정된다.
  - 만약 각 방향 별로 다른 간격을 적용하고 싶다면 아래 속성을 설정해야한다.
    - `margin`: `layout_marginTop`, `layout_marginBottom`, `layout_marginLeft`, `layout_marginRight`
    - `padding`: `paddingTop`, `paddingBottom`, `paddingLeft`, `paddingRight`

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="horizontal"
      android:padding="16dp">
      <Button
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:layout_marginTop="16dp"
          android:layout_paddingBottom="50dp"
          android:text="확인"/>
      <Button
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:layout_marginLeft="16dp"
          android:layout_paddingBottom="50dp"
          android:text="취소"/>
  </LinearLayout>
  ```



- 뷰의 표시 여부 설정

  - `visibility` 속성은 뷰가 화면에 출력되어야 하는지 여부를 설정한다.
    - `visible`: 뷰가 화면에 출력된다(기본값).
    - `invisible`: 뷰가 화면에 출력되지 않지만 자리는 차지한다.
    - `gone`: 뷰가 화면에 출력되지 않으며 자리도 차지하지 않는다.
    - `visibility`를 `invisible`이나 `gone`으로 설정하는 것은 주로 특정 조건일 때만 보이게 처리하기 위함이다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="horizontal"
      android:padding="16dp">
      <Button
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="확인",
          android:visibility="visible"/>
      <Button
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="invisible"
          android:visibility="invisible"/>
      <Button
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="gone"
          android:visibility="gone"/>
  </LinearLayout>
  ```

  - XML이 아닌 코드에서 뷰의 visibility 속성을 조정하려면 뷰의 visibility 속성 값을 설정해주면 된다.
    - `View.VISIBLE`, `View.INVISIBLE` 중 하나로 설정한다.

  ```kotlin
  visibleBtn.setOnClickListener {
      targetView.visibility = View.VISIBLE
  }
  
  invisibleBtn.setOnClickListener {
      targetView.visibility = View.INVISIBLE
  }
  ```





## 기본적인 뷰의 종류

- `TextView`

  - 문자열을 화면에 출력하는 뷰이다.

  ```xml
  <TextView
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:text="hello world"
      android:textColor="#CFCFCE"
      android:textSize="20sp"
      android:textStyle="bold" />
  ```

  - `android:text`
    - `TextView`에 출력할 문자열을 지정한다.
    - 문자열을 대입해도 되고, `android:text="@string/hello"` 처럼 문자열 리소스를 지정해도 된다.
  - `android:textColor`
    - 문자열의 색상을 지정한다.
    - 값은 16진수 RGB 형식을 사용한다.
  - `android:textSize`
    - 문자열의 크기를 지정한다.
    - 값은 숫자를 사용하고, 단위는 px, dp, sp 등을 사용하며 생략이 불가능하다.
  - `android:textStyle`
    - 문자열의 스타일을 지정한다.
    - 값은 `bold`, `italic`, `normal` 중 하나를 사용한다.

  - `android:autoLink`
    - `TextView`에 출력할 문자열을 분석해 특정 형태의 문자열에 자동 링크를 추가한다.
    - `web`, `phone`, `email` 등을 사용할 수 있으며 여러 개를 함께 설정하려면 `|` 기호로 연결한다.

  ```xml
  <TextView
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:text="https://github.com/hedge0207"
      android:autoLink="web" /> 
  ```

  - `android:maxLines`
    - `TextView`는 긴 문자열을 자동으로 줄바꿈하는데, 때로는 문자열이 특정 줄까지만 나오도록 해야 할 때가 있다.
    - 이 때 사용하는 속성으로, 설정된 숫자 만큼의 행까지만 출력된다.

  - `android:ellipsize`
    - `maxLines` 속성을 사용할 때 출력되지 않은 문자열이 더 있다는 것을 표시하기 위해 줄임표를 넣을 때 사용한다.
    - `end`, `middle`, `start` 중 하나의 값을 설정하면 된다.
    - `start`, `middle`의 경우 `singleLine="true"`를 통해 문자열으 한 줄로 출력하도록 설정했을 때만 적용된다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
      <TextView
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="@string/long_text"
          android:singleLine="true"
          android:ellipsize="middle"/>
      <TextView
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="@string/long_text"
          android:maxLines="3"
          android:ellipsize="end"/>
  </LinearLayout>
  ```



- `ImageView`

  - 이미지를 화면에 출력하는 뷰이다.
  - `android:src`
    - `ImageView`에 출력할 이미지를 설정한다.
    - 리소스 이미지, 파일 이미지, 네트워크 이미지 등을 출력할 수 있다.

  ```xml
  <ImageView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:src="@drawable/test"/>
  ```

  - `android:maxWidth`, `android:maxHeight`, `android:adjustViewBounds`
    - `ImageView`가 출력하는 이미지의 최대 크기를 지정한다.
    - 뷰의 크기는 `layout_widht`, `layout_height`로 설정하지만 이 속성은 크기가 고정되어 있어 뷰에 넣을 이미지 크기가 다양하다면 이미지와 뷰의 크기가 맞지 않는 상황이 발생할 수 있따.
    - 또한 이미지가 클 때 위 두 속성의 값을 `wrap_content`로 설정하면 뷰의 크기가 지나치게 커지는 문제가 있다.
    - 이럴 때 이 속성들을 이용해 크기를 조절할 수 있다.

  ```xml
  <ImageView
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:maxWidth="100dp"
      android:maxHeight="100dp"
      android:adjustViewBounds="true"
      android:src="@drawable/test"/>
  ```

  - `android:backgound`
    - 전체 뷰 중 이미지가 차지하지 않은 공간의 색을 설정한다.
    - 값은 16진수 RGB 형식을 사용한다.



- 버튼, 체크박스, 라디오 버튼

  - `Button`
    - 사용자 이벤트를 처리한다.

  ```xml
  <Button
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:text="확인"/>
  ```

  - `CheckBox`
    - 다중 선택을 제공한다.
    - 한 화면에 여러 개개 나오더라도 다중 선택을 제공하므로 서로 영향을 미치지 않는다.
    - 즉 하나의 체크박스가 선택되었어도 다른 체크박스를 선택할 수 있다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
      <CheckBox
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="check1"/>
      <CheckBox
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="check2"/>
  </LinearLayout>
  ```

  - `RadioGroup`, `RadioButton`
    - 단일 선택을 제공한다.
    - 화면에 여러 개가 나오면 하나만 선택할 수 있으므로 여러 개를 묶어서 처리해야 한다.
    - 이를 위해 `RadioGroup`과 함께 사용하며, 그룹으로 묶은 라디오 버튼 중 하나만 선택이 가능하다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
      <RadioGroup
          android:layout_width="wrap_content"
          android:layout_height="wrap_content">
          <RadioButton
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="radio1"/>
          <RadioButton
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="radio1"/>
      </RadioGroup>
  </LinearLayout>
  ```



- `EditText`

  - 사용자가 글을 입력할 수 있는 뷰
  - `android:lines`, `android:maxLines` 속성
    - `EditText`는 한 줄 입력 크기로 출력되었다가 사용자가 엔터를 누르면 여러 줄 입력 크기가 된다.
    - `android:lines`는 처음부터 여러 줄 입력 크기로 나오게 하는 속성이며, 설정된 것에서 더 늘어나지 않는다.
    - `android:maxLines`는 처음에는 한 줄 입력 크기로 출력되다, 사용자가 엔터를 입력하면 최대 설정한 크기까지 늘어난다.
  - `android:inputType`
    - 글을 입력할 때 올라오는 키보드를 지정하는 속성이다.
    - 예를 들어 키보드로 한 줄 입력을 강제하고 싶거나 키보드를 전화번호 입력 모드로 지정하고 싶을 때 사용한다.
    - 따로 설정하지 않으면 기본적으로 문자 입력 모드로 키보드가 올라온다.

  | 속성값               | 설명                                                         |
  | -------------------- | ------------------------------------------------------------ |
  | none                 | 모든 문자 입력이 가능하며, 줄바꿈이 가능하다.                |
  | text                 | 문자열 한 줄 입력.                                           |
  | textCapCharacters    | 대문자 입력 모드.                                            |
  | textCapWords         | 각 단어의 첫 글자 입력 시 키보드가 자동으로 대문자 입력 모드. |
  | TextCapSentences     | 각 문단의 첫 글자 입력 시 키보드가 자동으로 대문자 입력 모드. |
  | textMultiLine        | 여러 줄 입력 가능.                                           |
  | textNoSuggestions    | 단어 입력 시 키보드의 추천 단어를 보여주지 않음.             |
  | textUri              | URI 입력 모드.                                               |
  | textEmailAddress     | 이메일 주소 입력 모드                                        |
  | textPassword         | 비밀번호 입력 모드로 입력한 문자를 점으로 표시. 키보드는 영문자와 숫자, 특수 키만 표시 |
  | textVisibilePassword | textPassword와 같으며, 입력한 문자를 표시해준다는 차이가 있다. |
  | number               | 숫자 입력 모드.                                              |
  | numberSigned         | number와 같으며 부호 키인 마이너스 입력 가능.                |
  | numberDecimal        | number와 같으며 소숫점 입력 가능.                            |
  | numberPassword       | 숫자 키만 입력 가능하며, 점으로 표시된다.                    |
  | phone                | 전화번호 입력 모드                                           |





## 뷰 바인딩

- 뷰 바인딩(view binding)

  - 레이아웃 XML 파일에 선언한 뷰 객체를 코드에서 쉽게 이용하는 방법이다.

    - 안드로이드는 UI를 구성할 때 대부분 레이아웃 XML 파일을 이용한다.
    - 레이아웃 XML에 등록한 뷰는 `findViewByID()` 함수를 통해 사용할 수 있다.
    - 그런데, 모든 뷰를 이렇게 관리하면 번거로울 수 있다.
    - 따라서 안드로이드는 액티비티에서 `findViewById()` 함수를 이용하지 않고 레이아웃 XML 파일에 드록된 뷰 객체를 사용할 수 있는 기능을 제공한다.

    - 초기에는 butterknife라는 라이브러리를 사용해야 했으나, 이제 라이브러리 없이도 가능하다.
    - Kotlin Android Extensions도 같은 목적을 가진 다른 기법이나, 2021년 지원이 종료됐다.

  - 예를 들어 아래와 같이 작성한 레이아웃 XML이 있다고 가정해보자.

    - 아래에서 선언한 뷰 3개를 코드에서 id값으로 얻어서 사용할 수도 있다.
    - 그런데 뷰 바인딩 기법을 사용하면 코드에서 훨씬 간편하게 뷰 객체를 이용할 수 있다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:orientation="vertical">
      <Button
          android:id="@+id/visibileBtn"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="visibile"/>
      <TextView
          android:id="@+id/targetView"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="@string/main_desc"
          android:textSize="17dp"
           />
      <TextView
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="hello world"
          android:background="@FF0000"
          android:textColor="#FFFFFF"/>
      <Button
          android:id="@id+id/invisibileBtn"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="invisible"/>
  </LinearLayout>
  ```

  - 뷰 바인딩을 사용하려면 build.gradle 파일에 아래와 같이 선언해야한다.
    - 아래와 같이 설정하면 레이아웃 XML 파일에 등록된 뷰 객체를 포함하는 클래스가 자동으로 생성된다.
    - 즉 코드에서 뷰를 선언하고 `findViewById()` 함수를 호출하지 않아도 이를 구현한 클래스가 자동으로 생성되므로 이 클래스를 이용해 뷰를 사용하기만 하면 된다.

  ```groovy
  android {
      viewBinding.isEnabled = true
  }
  ```

  - 자동으로 만들어지는 클래스의 이름은 레이아웃 XML 파일명을 따른다.
    - 첫 글자를 대문자로 하고 밑줄은 빼고 뒤에 오는 단어를 대문자로 만든 후 Binding을 추가한다.
    - e.g. `activity_main.xml` - `ActivitiyMainBinding`
  - 자동으로 만들어진 클래스의 `inflate()` 함수를 호출하면 바인딩 객체를 얻을 수 있다.
    - 이때 인자로 `layoutInflater`를 전달한다.
    - 그리고 바인딩 객체의 root 프로퍼티에는 XML의 루트 태그 객체가 자동으로 등록되므로 액티비티 화면 출력은 `setContentView()` 함수에 `binding.root`를 전달하면 된다.
    - 바인딩 객체에 등록된 뷰 객체명은 XML 파일에 등록한 뷰의 id 값을 따른다.

  ```kotlin
  class MainActivity : AppCompatActivity() {
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceSTate)
          
          // 바인딩 객체 획득
          val binding = ActivityMainBinding.inflate(layoutInflater)
          
          // 액티비티 화면 출력
          setContentView(binding.root)
          
          // 뷰 객체 사용
          binding.visibleBtn.setOnClickListener {
              binding.targetView.visibility = View.VISIBLE
          }
          binding.invisibleBtn.setOnClickListener {
              binding.targetView.visibility = View.INVISIBLE
          }
      }
  }
  ```

  - `tools:viewBindingIgnore`
    - 뷰 바인딩을 사용하면 레이아웃 XML 하나당 바인딩 클래스가 자동으로 만들어진다.
    - 이 때 어떤 레이아웃 XML 파일은 바인딩 클래스를 만들 필요가 없을 수도 있따.
    - 이때는 XML 파일의 루트 태그에 `tools:viewBindingIgnore=true` 속성을 추가하면 XML 파일을 위한 바인딩 클래스를 만들지 않는다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      tools:viewBindingIgnore=true>
  ```





## 실습 - 비밀번호 화면 만들기

- 새 모듈 만들기

  - 문자열 리소스를 등록한다.
    - `res/values/strings.xml`파일에 아래와 같이 추가한다.

  ```xml
  <resources>
      <string name="app_name">PasswordPractice</string>
      <string name="main_desc">
          현재 비밀번호를 확인해주세요.
      </string>
      <string name="password_txt">비밀번호를 잊으셨나요?</string>
  </resources>
  ```

  - 레이아웃 XML 파일(res/layout/activity_main.xml)을 작성한다.

  ```xml
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      tools:context=".MainActivity"
      android:orientation="vertical"
      android:padding="16dp">
  
      <TextView
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="@string/main_desc"
          android:textSize="17dp"
           />
      <TextView
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:text="kkang104@gmail.com"
          android:layout_marginTop="10dp"
          android:textColor="#CFCFCE"/>
  
      <View
          android:layout_width="match_parent"
          android:layout_height="1dp"
          android:layout_marginTop="10dp"
          android:background="#D4D4D3"/>
  
      <EditText
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:hint="비밀번호"
          android:inputType="textPassword"/>
      <TextView
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:layout_marginTop="10dp"
          android:text="@string/password_txt"/>
  
      <Button
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:layout_marginTop="16dp"
          android:text="확인"/>
  </LinearLayout>
  ```

  - Android studio에서 run을 실행한다.



