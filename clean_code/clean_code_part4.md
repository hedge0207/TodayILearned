# 점진적인 개선

- 깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 한다. 
  - 프로그래밍은 과학 보다는 공예(craft)에 가깝다.
  - 대다수 신참 프로그래머는 이 충고를 충실히 따르지 않는다.
    - 그들은 무조건 돌아가는 프로그램을 목표로 잡는다.
    - 일단 프로그램이 돌아가면 다음 업무로 넘어간다.
    - 돌아가는 프로그램은 그 상태가 어떻든 그대로 버려둔다.
  - 처음에 적당히 깔끔하게 짰다고 생각하더라도 이후에 기능이 추가되면서 지저분한 코드가 되기도 한다.
    - 기능이 추가되면서 유지보수가 적당히 수월했던 코드는 버그와 결함이 숨어있을지 모르는 의심스러운 코드가 되기도 한다.
    - 만약 코드가 의심스럽다면 코드 구조를 유지보수하기 좋은 상태로 변경해야한다.
  - 프로그램을 망치는 가장 좋은 방법 중 하나는 개선이라는 이름 아래 구조를 크게 뒤집는 행위다.
    - 어떤 프로그램은 그런 '개선'에서 결코 회복하지 못한다.
    - '개선' 전과 똑같이 프로그램을 돌리기가 아주 어렵기 때문이다.



- 점진적 개선에는 테스트 코드가 필요하다.
  - 변경 전후에 시스템이 똑같이 돌아간다는 사실을 확인하려면 언제든 실행이 가능한 자동화된 테스트 슈트가 필요하다.
  - 개선 대상 코드에 대한 테스트를 작성하고, 개선 이후에 모든 테스트가 통과하는지 확인해야한다.





- 점진적 개선 코드

  - 전반적으로 깔끔한 구조에 잘 짜인 코드를 살펴볼 것이다.
  - `Args` 클래스
    - 아래 코드는 여기저기 뒤적일 필요 없이 위에서 아래로 코드가 읽힌다.

  ```java
  import java.util.*;
  
  import static com.cleancoder.args.ArgsException.ErrorCode.*;
  
  public class Args {
    private Map<Character, ArgumentMarshaler> marshalers;
    private Set<Character> argsFound;
    private ListIterator<String> currentArgument;
  
    public Args(String schema, String[] args) throws ArgsException {
      marshalers = new HashMap<Character, ArgumentMarshaler>();
      argsFound = new HashSet<Character>();
  
      parseSchema(schema);
      parseArgumentStrings(Arrays.asList(args));
    }
  
    private void parseSchema(String schema) throws ArgsException {
      for (String element : schema.split(","))
        if (element.length() > 0)
          parseSchemaElement(element.trim());
    }
  
    private void parseSchemaElement(String element) throws ArgsException {
      char elementId = element.charAt(0);
      String elementTail = element.substring(1);
      validateSchemaElementId(elementId);
      if (elementTail.length() == 0)
        marshalers.put(elementId, new BooleanArgumentMarshaler());
      else if (elementTail.equals("*"))
        marshalers.put(elementId, new StringArgumentMarshaler());
      else if (elementTail.equals("#"))
        marshalers.put(elementId, new IntegerArgumentMarshaler());
      else if (elementTail.equals("##"))
        marshalers.put(elementId, new DoubleArgumentMarshaler());
      else if (elementTail.equals("[*]"))
        marshalers.put(elementId, new StringArrayArgumentMarshaler());
      else if (elementTail.equals("&"))
        marshalers.put(elementId, new MapArgumentMarshaler());
      else
        throw new ArgsException(INVALID_ARGUMENT_FORMAT, elementId, elementTail);
    }
  
    private void validateSchemaElementId(char elementId) throws ArgsException {
      if (!Character.isLetter(elementId))
        throw new ArgsException(INVALID_ARGUMENT_NAME, elementId, null);
    }
  
    private void parseArgumentStrings(List<String> argsList) throws ArgsException {
      for (currentArgument = argsList.listIterator(); currentArgument.hasNext();) {
        String argString = currentArgument.next();
        if (argString.startsWith("-")) {
          parseArgumentCharacters(argString.substring(1));
        } else {
          currentArgument.previous();
          break;
        }
      }
    }
  
    private void parseArgumentCharacters(String argChars) throws ArgsException {
      for (int i = 0; i < argChars.length(); i++)
        parseArgumentCharacter(argChars.charAt(i));
    }
  
    private void parseArgumentCharacter(char argChar) throws ArgsException {
      ArgumentMarshaler m = marshalers.get(argChar);
      if (m == null) {
        throw new ArgsException(UNEXPECTED_ARGUMENT, argChar, null);
      } else {
        argsFound.add(argChar);
        try {
          m.set(currentArgument);
        } catch (ArgsException e) {
          e.setErrorArgumentId(argChar);
          throw e;
        }
      }
    }
  
    public boolean has(char arg) {
      return argsFound.contains(arg);
    }
  
    public int nextArgument() {
      return currentArgument.nextIndex();
    }
  
    public boolean getBoolean(char arg) {
      return BooleanArgumentMarshaler.getValue(marshalers.get(arg));
    }
  
    public String getString(char arg) {
      return StringArgumentMarshaler.getValue(marshalers.get(arg));
    }
  
    public int getInt(char arg) {
      return IntegerArgumentMarshaler.getValue(marshalers.get(arg));
    }
  
    public double getDouble(char arg) {
      return DoubleArgumentMarshaler.getValue(marshalers.get(arg));
    }
  
    public String[] getStringArray(char arg) {
      return StringArrayArgumentMarshaler.getValue(marshalers.get(arg));
    }
  
    public Map<String, String> getMap(char arg) {
      return MapArgumentMarshaler.getValue(marshalers.get(arg));
    }
  }
  ```

  - `ArgumentMarshaler` 인터페이스

  ```java
  import java.util.Iterator;
  
  public interface ArgumentMarshaler {
    void set(Iterator<String> currentArgument) throws ArgsException;
  }
  ```

  - `BooleanArgumentMarshaler`

  ```java
  import java.util.Iterator;
  
  public class BooleanArgumentMarshaler implements ArgumentMarshaler {
    private boolean booleanValue = false;
  
    public void set(Iterator<String> currentArgument) throws ArgsException {
      booleanValue = true;
    }
  
    public static boolean getValue(ArgumentMarshaler am) {
      if (am != null && am instanceof BooleanArgumentMarshaler)
        return ((BooleanArgumentMarshaler) am).booleanValue;
      else
        return false;
    }
  }
  ```

  - `StringArgumentMarshaler`

  ```java
  import java.util.Iterator;
  import java.util.NoSuchElementException;
  
  import static com.cleancoder.args.ArgsException.ErrorCode.MISSING_STRING;
  
  public class StringArgumentMarshaler implements ArgumentMarshaler {
    private String stringValue = "";
  
    public void set(Iterator<String> currentArgument) throws ArgsException {
      try {
        stringValue = currentArgument.next();
      } catch (NoSuchElementException e) {
        throw new ArgsException(MISSING_STRING);
      }
    }
  
    public static String getValue(ArgumentMarshaler am) {
      if (am != null && am instanceof StringArgumentMarshaler)
        return ((StringArgumentMarshaler) am).stringValue;
      else
        return "";
    }
  }
  ```

  - `IntegerArgumentMarshaler`

  ```java
  package com.cleancoder.args;
  
  import static com.cleancoder.args.ArgsException.ErrorCode.*;
  
  import java.util.*;
  
  public class IntegerArgumentMarshaler implements ArgumentMarshaler {
    private int intValue = 0;
  
    public void set(Iterator<String> currentArgument) throws ArgsException {
      String parameter = null;
      try {
        parameter = currentArgument.next();
        intValue = Integer.parseInt(parameter);
      } catch (NoSuchElementException e) {
        throw new ArgsException(MISSING_INTEGER);
      } catch (NumberFormatException e) {
        throw new ArgsException(INVALID_INTEGER, parameter);
      }
    }
  
    public static int getValue(ArgumentMarshaler am) {
      if (am != null && am instanceof IntegerArgumentMarshaler)
        return ((IntegerArgumentMarshaler) am).intValue;
      else
        return 0;
    }
  }
  ```

  - `DoubleArgumentMarshaler`

  ```java
  import static com.cleancoder.args.ArgsException.ErrorCode.*;
  
  import java.util.*;
  
  public class DoubleArgumentMarshaler implements ArgumentMarshaler {
    private double doubleValue = 0;
  
    public void set(Iterator<String> currentArgument) throws ArgsException {
      String parameter = null;
      try {
        parameter = currentArgument.next();
        doubleValue = Double.parseDouble(parameter);
      } catch (NoSuchElementException e) {
        throw new ArgsException(MISSING_DOUBLE);
      } catch (NumberFormatException e) {
        throw new ArgsException(INVALID_DOUBLE, parameter);
      }
    }
  
    public static double getValue(ArgumentMarshaler am) {
      if (am != null && am instanceof DoubleArgumentMarshaler)
        return ((DoubleArgumentMarshaler) am).doubleValue;
      else
        return 0.0;
    }
  }
  ```

  - `StringArrayArgumentMarshaler`역시 위와 같은 패턴이다.

  ```java
  import static com.cleancoder.args.ArgsException.ErrorCode.*;
  
  import java.util.*;
  
  public class StringArrayArgumentMarshaler implements ArgumentMarshaler {
    private List<String> strings = new ArrayList<String>();
  
    public void set(Iterator<String> currentArgument) throws ArgsException {
      try {
        strings.add(currentArgument.next());
      } catch (NoSuchElementException e) {
        throw new ArgsException(MISSING_STRING);
      }
    }
  
    public static String[] getValue(ArgumentMarshaler am) {
      if (am != null && am instanceof StringArrayArgumentMarshaler)
        return ((StringArrayArgumentMarshaler) am).strings.toArray(new String[0]);
      else
        return new String[0];
    }
  }
  ```

  - `ArgsException`

  ```java
  package com.cleancoder.args;
  
  import static com.cleancoder.args.ArgsException.ErrorCode.*;
  
  public class ArgsException extends Exception {
    private char errorArgumentId = '\0';
    private String errorParameter = null;
    private ErrorCode errorCode = OK;
  
    public ArgsException() {}
  
    public ArgsException(String message) {super(message);}
  
    public ArgsException(ErrorCode errorCode) {
      this.errorCode = errorCode;
    }
  
    public ArgsException(ErrorCode errorCode, String errorParameter) {
      this.errorCode = errorCode;
      this.errorParameter = errorParameter;
    }
  
    public ArgsException(ErrorCode errorCode, char errorArgumentId, String errorParameter) {
      this.errorCode = errorCode;
      this.errorParameter = errorParameter;
      this.errorArgumentId = errorArgumentId;
    }
  
    public char getErrorArgumentId() {
      return errorArgumentId;
    }
  
    public void setErrorArgumentId(char errorArgumentId) {
      this.errorArgumentId = errorArgumentId;
    }
  
    public String getErrorParameter() {
      return errorParameter;
    }
  
    public void setErrorParameter(String errorParameter) {
      this.errorParameter = errorParameter;
    }
  
    public ErrorCode getErrorCode() {
      return errorCode;
    }
  
    public void setErrorCode(ErrorCode errorCode) {
      this.errorCode = errorCode;
    }
  
    public String errorMessage() {
      switch (errorCode) {
        case OK:
          return "TILT: Should not get here.";
        case UNEXPECTED_ARGUMENT:
          return String.format("Argument -%c unexpected.", errorArgumentId);
        case MISSING_STRING:
          return String.format("Could not find string parameter for -%c.", errorArgumentId);
        case INVALID_INTEGER:
          return String.format("Argument -%c expects an integer but was '%s'.", errorArgumentId, errorParameter);
        case MISSING_INTEGER:
          return String.format("Could not find integer parameter for -%c.", errorArgumentId);
        case INVALID_DOUBLE:
          return String.format("Argument -%c expects a double but was '%s'.", errorArgumentId, errorParameter);
        case MISSING_DOUBLE:
          return String.format("Could not find double parameter for -%c.", errorArgumentId);
        case INVALID_ARGUMENT_NAME:
          return String.format("'%c' is not a valid argument name.", errorArgumentId);
        case INVALID_ARGUMENT_FORMAT:
          return String.format("'%s' is not a valid argument format.", errorParameter);
      }
      return "";
    }
  
    public enum ErrorCode {
      OK, INVALID_ARGUMENT_FORMAT, UNEXPECTED_ARGUMENT, INVALID_ARGUMENT_NAME,
      MISSING_STRING,
      MISSING_INTEGER, INVALID_INTEGER,
      MISSING_DOUBLE, INVALID_DOUBLE,
      MALFORMED_MAP, MISSING_MAP
    }
  }
  ```





# JUnit 들여다보기

> 필요할 경우 추가.





# SerialDate 리팩터링

> 필요할 경우 추가.







# 냄새와 휴리스틱

- 일반

  - 한 소스 파일에 하나의 언어만 사용해라.

    - 오늘날 프로그래밍 환경은 한 소스 파일 내에서 다양한 언어를 지원한다.
    - 예를 들어 어떤 자바 소스 파일은 XML, HTML, YAML, Javadoc, 영어 등을 포함한다.
    - 이는 좋게 말하면 혼란스럽고, 나쁘게 말하면 조잡하다.
    - 이상적으로는 소스 파일 하나에 언어 하나만 사용하는 방식이 가장 좋다.
    - 현실적으로는 여러 언어가 불가피하지만 노력을 기울여 소스 파일에서 언어 수와 범위를 최대한으로 줄여라.

  - 당연하게 여겨지는 동작은 당연히 구현되어야한다.

    - 최소 놀람의 법칙(The Principle of Least Suprise)에 의거해 함수나 클래스는 다른 프로그래머가 당연하게 여길 만한 동작과 기능을 제공해야 한다.
    - 당연한 동작을 구현하지 않으면 코드를 읽거나 사용하는 사람이 더 이상 함수 이름만으로 함수 기능을 직관적으로 예상하기 어렵다.
    - 저자를 신뢰하지 못하므로 코드를 일일이 살펴야 한다.

  - 경계를 올바로 처리해야한다.

    - 코드는 올바로 동작해야한다.
    - 그런데 우리는 올바른 동작이 아주 복잡하다는 사실을 자주 간과한다.
    - 흔히 개발자들은 머릿속에서 코드를 돌려보고 끝낸다.
    - 자신의 직관에 의존할 뿐 모든 경계와 구석진 곳에서 코드를 증명하려 애쓰지 않는다.
    - 부지런함을 대신할 지름길은 없다.
    - 모든 경계 조건, 모든 구석진 곳, 모든 기벽, 모든 예외는 우아하고 직관적인 알고리즘을 좌초시킬 암초다.
    - 스스로의 직관에 의존하지 마라.
    - 모든 경계 조건을 찾아내고, 모든 경계 조건을 테스트하는 테스트 케이스를 작성하라.

  - 안전 절차 무시

    - 안전 절차를 무시하면 위험하다.
    - 컴파일러 경고 일부를 꺼버리면 빌드가 쉬워질지 모르지만 자칫하면 끝없는 디버깅에 시달린다.
    - 실패하는 테스트 케이스를 일단 제껴두고 나중으로 미루는 태도는 옳지 않다.

  - 중복

    - 데이비드 토머스와 앤디 헌트는 이를 DRY(Don't Repeat Yourself) 원칙이라 부른다.
    - 켄트 벡은 익스트림 프로그래밍의 핵심 규칙 중 하나로 선언한 후 "한 번, 단 한 번만(Once, and only once)"라 명명했다.ㄴ
    - 론 제프리스는 이 규칙을 모든 테스트를 통과한다는 규칙 다음으로 중요하게 꼽았다.
    - 코드에서 중복을 발견할 때마다 추상화할 기회로 간주하라.
    - 중복된 코드를 하위 루틴이나 다른 클래스로 분리하라.
    - 이렇듯 추상화로 중복을 정리하면 설계 언어의 어휘가 늘어난다.
    - 다른 프로그래머들이 그만큼 어휘를 사용하기 쉬워진다.
    - 추상화 수준을 높였으므로 구현이 빨라지고 오류가 적어진다.
    - 사실 최근 15년 동안 나온 디자인 패턴은 대다수가 중복을 제거하는 잘 알려진 방법에 불과하다.

    - 어디서든 중복을 발견하면 없애라.
  - 추상화 수준이 올바르지 못하다.
    - 추상화는 저차원 상세 개념에서 고차원 일반 개념을 분리한다.
    - 때로 우리는 고차원 개념을 표현하는 추상 클래스와 저차원 개념을 표현하는 파생 클래스를 생성해 추상화를 수행한다.
    - 추상화로 개념을 분리할 때는 철저해야 한다.
    - 모든 저차원 개념은 파생 클래스에 넣고, 모든 고차원 개념은 기초 클래스에 넣는다.
    - 예를 들어 세부 구현과 관련한 상수, 변수 ,유틸리티 함수는 기초 클래스에 넣으면 안 된다.
    - 기초 클래스는 구현 정보에 무지해야한다.
    - 소스 파일, 컴포넌트, 모듈도 마찬가지다.
    - 우수한 소프트웨어 설계자는 개념을 다양한 창원으로 분리해 다른 컨테이너에 넣는다.
    - 때로는 기초 클래스와 파생 클래스로 분리하고, 때로는 소스 파일과 모듈과 컴포넌트로 분리한다.
    - 고차원 개념과 저차원 개념을 섞어서는 안 된다.
    - 예를 들어 아래 클래스에서 `percentFull` 메서드는 추상화 수준이 올바르지 못하다.
    - Stack을 구현하는 방법은 다양하며, 어떤 구현에서는 `percentFull`이 표현하고자 하는 "꽉 찬 정도"라는 개념이 타당할 수 있다.
    - 그러나 모든 Stack의 파생 클래스가 이러한 개념을 필요로 하는 것은 아니므로, `BoundedStack`과 같은 파생 인터페이스에 넣는 것이 마땅하다.

  ```java
  public interface Stack {
      Object pop() throws EmptyException;
      void push(Object o) throws FullException;
      double percentFull();
      class EmptyException extends Exception {}
      class FullException extends Exception {}
  }
  ```

  - 기초 클래스가 파생 클래스에 의존해선 안 된다.
    - 기초 클래스와 파생 클래스를 나누는 가장 일반적인 이유는 고차원 기초 클래스 개념을 저차원 파생 클래스 개념으로부터 분리해 독립성을 보장하기 위해서다.
    - 그러므로 기초 클래스가 파생 클래스를 사용한다면 뭔가 문제가 있다는 말이다.
    - 일반적으로 기초 클래스는 파생 클래스를 아예 몰라야 마땅하다.
    - 물론 예외는 있으며, 간혹 파생 클래스 개수가 확실히 고정되어 있다면 기초 클래스에 파생 클래스를 선택하는 코드가 들어간다.
  - 과도한 정보를 제공해선 안 된다.
    - 잘 정의된 모듈은 인터페이스가 아주 작다.
    - 하지만 작은 인터페이스로도 많은 동작이 가능하다.
    - 부실하게 정의된 모듈은 인터페이스가 구질구질하다.
    - 그래서 간단한 동작 하나에도 온갖 인터페이스가 필요하다.
    - 잘 정의된 인터페이스는 많은 함수를 제공하지 않으므로 결합도가 낮다.
    - 불실하게 정의된 인터페이스는 반드시 호출해야 하는 온갖 함수를 제공하므로 결합도가 높다.
    - 클래스나 모듈 인터페이스에 노출할 함수를 제한할 줄 알아야 한다.
    - 클래스가 제공하는 메서드는 적을수록 좋고, 함수가 아는 변수도 적을수록 좋으며, 클래스의 인스턴스 변수도 적을수록 좋다.
    - 자료, 유틸리티 함수, 상수와 임시 변수를 숨겨라.
    - 하위 클래스에서 필요하다는 이유로 protected 변수나 함수를 마구 생성하지 마라.
    - 인터페이스를 매우 작게 그리고 매우 깐깐하게 만들어라.
  - 죽은 코드는 제거하라
    - 죽은 코드란 실행되지 않는 코드를 가리킨다.
    - 불가능한 조건을 확인하는 if문과, throw문이 없는 try문에서 catch 블록이 좋은 예다.
    - 아무도 호출하지 않는 유틸리티 함수와 switch 문에서 불가능한 case 조건도 또 다른 예다.
    - 죽은 코드는 설계가 변해도 제대로 수정되지 않기에 죽은 지 오래될수록 코드의 악취는 강해진다.
  - 수직 분리
    - 변수와 함수는 사용되는 위치에 가깝게 정의한다.
    - 지역 변수는 처음으로 사용하기 직전에 선언하며 수직으로 가까운 곳에 위치해야한다.
    - 비공개 함수는 처음으로 호출한 직후에 정의한다.
    - 비공개 함수는 전체 클래스 scope에 소하지만 그래도 정의하는 위치와 호출하는 위치를 가깝게 유지한다.
    - 비공개 함수는 처음으로 호출되는 위치를 찾은 후 조금만 아래로 내려가면 찾을 수 있어야한다.
  - 일관성이 있어야한다.
    - 어떤 개념을 특정 방식으로 구현했다면 유사한 개념도 같은 방식으로 구현한다.
    - 이는 최소 놀람의 원칙에도 부합하며, 표기법은 신중하게 선택하고, 일단 선택한 표기법은 신중하게 따른다.
    - 한 함수에서 response라는 변수에 HttpResponse 인스턴스를 저장했다면, HttpResponse 객체를 사용하는 다른 함수에서도 일관성 있게 동일한 변수 이름을 사용한다.
    - 한 메서드를 processVerificationRequest라고 명명했다면, 유사한 요청을 처리하는 다른 메서드도 processDeletionRequest처럼 유사한 이름을 사용한다.
  - 잡동사니를 없애라.
    - 비어 있는 기본 생성자, 아무도 사용하지 않는 변수, 아무도 호출하지 않는 함수, 정보를 제공하지 못하는 주석 등은 모드 필요 없는 잡동사니다.
    - 소스 파일은 언제나 깔끔하게 정리하라.
  - 인위적 결합을 피하라.
    - 서로 무관한 개념을 인위적으로 결합하면 안 된다.
    - 일반적인 enum은 특정 클래스에 속할 이유가 없으며, enum이 클래스에 속한다면 enum을 사용하는 코드가 특정 클래스를 알아야만 한다.
    - 범용 static 함수도 마찬가지로 특정 클래스에 속할 이유가 없다.
    - 일반적으로 인위적인 결합은 직접적인 상호작용이 없는 두 모듈 사이에서 일어난다.
    - 뚜렷한 목적 없이 변수, 상수, 함수를 잘못된 위치에 넣어버린 겨로가다.
  - 기능 욕심을 버려라.
    - 클래스 메서드는 자기 클래스의 변수와 함수에 관심을 가져야지 다른 클래스의 변수와 함수에 관심을 가져서는 안 된다.
    - 메서드가 다른 객체의 참조자와 변경자를 사용해 그 객체 내용을 조작한다면 메서드가 그 객체 클래스의 범위를 욕심내는 탓이다.
    - 예를 들어 아래 `HourlyPayCalculator` 클래스는 `HourlyEmployee`의 인스턴스에서 온갖 정보를 가져온다.
    - 기능 욕심은 한 클래스의 속사정을 다른 클래스에 노출하므로, 별다른 문제가 없다면 제거하는 편이 좋다.
    - 하지만 때로는 어쩔 수 없는 경우도 생긴다.
    - 예를 들어 아래 `HourlyEmployeeReport.reportHours` 메서드 역시 `HourlyEmployee`의 범위를 욕심내지만 그렇다고 `HourlyEmployee`가 보고서 형식을 알 필요는 없으며, `HourlyEmployeeReport.reportHours` 메서드를 `HourlyEmployee`클래스로 옮기면 SRP와 OCP를 위반한다(`HourlyEmployee`가 보고서 형식과 결합되므로 보고서 형식이 바뀌면 클래스도 바뀐다).

  ```java
  public class HourlyPayCalculator {
      public Money calculateWeeklyPay(HourlyEmployee e) {
          int tenthRate = e.getTenthRate().getPennies();
          int tenthsWorked = e.getTenthsWorked();
          // ...
      }
  }
  
  
  // 어쩔 수 없는 경우
  public class HourlyEmployeeReport {
      private HourlyEmployee employee;
      
      public HourlyEmployeeReport(HourlyEmployee e) {
          this.employee = e;
      }
      
      String reportHours() {
          return String.format(
              "Name:%s\tHours:%d.%1d\n", 
              emplyee.getName(), 
              employee.getTenthsWorkded()/10,
              employee.getTenthsWorkded()%10);
      }
  }
  ```

  - 선택자 인수를 피해라.
    - 함수 호출 끝에 달리는 false 인수만큼이나 이상한 코드도 없다.
    - 선택자 인수는 목적을 기억하기 어려울 뿐 아니라 각 선택자 인수가 여러 함수를 하나로 조합한다.
    - 선택자 인수는 큰 함수를 작은 함수 여럿으로 쪼개지 않으려는 게으름의 소산이다.
    - 예를 들어 `calculateWeeklyPay` 메서드는 `overtime`이라는 boolean 값을 인수로 받아 그 값에 따라 각기 다른 처리를 한다.
    - 차라리 두 개의 메서드로 분리하는 것이 낫다.
    - 이는 비단 boolean만의 문제가 아니며, enum, int 등 함수 동작을 제어하려는 인수는 하나같이 바람직하지 않다.
    - 일반적으로, 인수를 넘겨 동작을 선택하는 대신 새로운 함수를 만드는 것이 좋다.

  ```java
  // 아래와 같이 하나의 함수에서 boolean 값에 따라 다른 처리를 하기 보다는
  public int calculateWeeklyPay(boolean overtime) {
      // ...
      double overtimeRate = overtime ? 1.5 : 1.0 * tenthRate;
      // ...
  }
  
  
  // 아래와 같이 두 개의 함수로 분리하는 것이 낫다.
  public int straightPay() {
      // ...
  }
  
  public int overTimePay() {
      // ...
  }
  ```

  - 모호한 의도
    - 코드를 짤 때는 의도를 최대한 분명히 밝힌다.
    - 행을 바꾸지 않고 표현한 수식, 헝가리식 표기법, 매직 넘버 등은 모두 저자의 의도를 흐린다.
  - 잘못 지운 책임
    - 소프트웨어 개발자가 내리는 가장 중요한 결정 중 하나가 코드를 배치하는 위치다.
    - 여기서도 최소 놀람의 원칙을 적용한다.
    - 코드는 독자가 자연스럽게 기대할 위치에 배치한다.
    - 때로는 독자에게 직관적인 위치가 아니라 개발자에게 편한 함수에 배치해도 된다.
    - 결정을 내리는 한 가지 방법으로 함수 이름을 살펴보는 방법이 있다.
    - 더 적합한 이름을 가진 함수에 더 적합한 코드를 배치한다.
  - 부적절한 static 함수
    - static 메서드를 소유하는 객체에서 가져오는 정보를 사용하지 않고, 인수를 통해 모든 정보를 받는다면 좋은 static 메서드일 수도 있다. 
    - 그러나 간혹 우리는 static으로 정의하면 안 되는 함수를 static으로 정의한다.
    - 예를 들어 만약 static 메서드를 서브 클래스에 따라 다르게 구현해야하는 상황이 오거나, static 메서드를 빈번하게 재정의해야 하는 경우 static 메서드로 정의해선 안 된다.
    - 일반적으로 static 메서드 보다는 일반 메서드가 더 나으며, 조금이라도 의심스럽다면 일반 메서드로 정의한다.
  - 서술적 변수는 많이 써도 괜찮다.
    - 프로그램 가독성을 높이는 가장 효과적인 방법 중 하나가 계산을 여러 단계로 나누고 중간 값으로 서술적인 변수 이름을 사용하는 방법이다.
    - 서술적인 변수 이름은 많이 써도 무방하며, 일반적으로는 많을수록 더 좋다.
    - 계산을 몇 단계로 나누고 중간값에 좋은 변수 이름만 붙여도 해독하기 어렵던 모듈이 순식간에 읽기 쉬운 모듈로 탈바꿈한다.
    - 예를 들어 아래 코드는 첫 번째로 일치하는 그룹이 key이고, 두 번째로 일치하는 그룹이 value라는 것을 서술적 변수를 통해 명확하게 드러내어 가독성을 높였다.

  ```java
  Matcher match = headerParttern.matcher(line);
  if (match.find()) {
      String key = match.group(1);
      String value = match.group(2);
      headers.put(key.toLowerCase(), value);
  }
  ```

  - 이름과 기능이 일치하는 함수.
    - 함수의 이름은 함수의 역할을 분명히 드러내야한다.
    - 이름만으로 역할이 분명하지 않기에 구현을 살피거나 문서를 뒤적여야 한다면 더 좋은 이름으로 바꾸거나 더 좋은 이름을 붙이기 쉽도록 기능을 정리해야한다.
  - 알고리즘을 이해해야한다.
    - 구현이 끝났다고 선언하기 전에 자신이 구현한 코드가 돌아가는 방식을 확실히 이해하는지 확인하라.
    - 단순히 자신이 작성한 테스트 케이스를 모두 통과한다는 사실 만으로는 부족하다.
    - 코드가 돌아간다는 사실을 아는 것과 돌아가기 위한 알고리즘이 올바르다는 것을 아는 것은 다르다.
    - 알고리즘이 올바르다는 사실을 확인하고 이해하려면 기능이 뻔히 보일 정도로 함수를 깔끔하고 명확하게 재구성하는 방법이 최고다.
  - 논리적 의존성은 물리적으로 드러내라
    - 한 모듈이 다른 모듈에 의존한다면 물리적인 의존성이 있어야한다.
    - 의존하는 모듈이 상대 모듈에 대해 뭔가를 가정하면(논리적으로 의존하면) 안 된다.
    - 의존하는 모든 정보를 명시적으로 요청하는 편이 좋다.
    - 예를 들어 아래 코드에서 `HourlyReporter`는 정보를 모아 `HourlyReportFormatter`에 적당한 형태로 넘기고, `HourlyReportFormatter`는 이를 출력한다.
    - 이 코드는 `PAGE_SIZE`라는 논리적인 의존성이 존재한다.
    - 이 상수는 `HourlyReportFormatter`가 책임질 정보이며, `HourlyReporter`가 이를 알아야 할 필요는 없다.
    - `HourlyReporter`는 `HourlyReportFormatter`가 페이지 크기 55를 처리할 줄 안다는 사실에 의존한다.
    - 만약 `HourlyReportFormatter`의 구현 중 하나가 페이지 크기 55를 제대로 처리하지 못한다면 오류가 생긴다.
    - `HourlyReportFormatter`에 `getMaxPageSize()`라는 메서드를 추가하면 논리적인 의존성이 물리적인 의존성으로 변한다.

  ```java
  public class HourlyReporter {
      private HourlyReportFormatter formatter;
      private List<LineItem> page;
      private final int PAGE_SIZE = 55;
      
      public HourlyReporter(HourlyReportFormatter formatter) {
          this.formatter = formatter;
          page = new ArrayList<LineItem>();
      }
      
      public void generateReport(List<HourlyEmployee> employees) {
          for (HourlyEmployee e : employees) {
              addLineItemToPage(e);
              if (page.size() == PAGE_SIZE)
                  printAndClearItemList();
          }
          if (page.size() > 0)
              printAndClearItemList();
      }
      
      private void printAndClearItemList() {
          formatter.format(page);
          page.clear();
      }
      
      private void addLineItemToPAge(HourlyEmployee e) {
          LineItem item = new LineItem();
          // ...
          page.add(item);
      }
  }
  ```

  - if/else 혹은 switch/case문보다 다형성을 사용하라.
    - 대다수 개발자가 switch문을 사용하는 이유는 그 상황에서 가장 올바른 선택이기보다는 당장 손쉬운 선택이기 때문이다.
    - 그러므로 switch를 선택하기 전에 다형성을 먼저 고려해야한다.
    - 또한 대개의 경우 새로운 요구사항이 발생하면 새로운 타입을 추가하는 것이 기존 함수를 변경하는 것 보다 더 쉽다.
    - 그러므로 모든 switch문을 의심해야한다.
  - 표준 표기법을 따르라
    - 팀은 업계 표준에 기반한 구현 표준을 따라야한다.
    - 구현 표준은 인스턴스 변수 이름을 선언하는 위치, 이름을 정하는 방법, 괄호를 넣는 위치 등을 명시해야한다.
    - 표준을 설명하는 문서는 코드 자체로 충분해야 하며 별도 문서를 만들 필요는 없어야한다.
  - 매직 넘버는 명명된 상수로 교체하라
    - 숫자는 명명된 상수 뒤로 숨겨야한다.
    - 어떤 상수는 이해하기 쉬우므로 코ㅡ 자체가 자명하다면, 상수 뒤로 숨길 필요가 없을 수 있다.
    - 매직 넘버는 단지 숫자만을 의미하지는 않으며, 의미가 분명하지 않은 모든 하드코딩된 값을 의미한다.
  - 정확하라
    - 코드에서 뭔가를 결정할 때는 정확히 결정해야한다.
    - 결정을 내리는 이유와 예외를 처리할 방법을 분명히 알아야 한다.
    - 코드에서 모호성과 부정확은 이견차나 게으름의 결과이므로 어느쪽이든 제거해야 마땅하다.
  - 관례보다 구조를 사용하라
    - 설계 결정을 강제할 때는 규칙보다 관례를 사용한다.
    - 명명 관례도 좋지만 구조 자체로 강제하면 더 좋다.
    - 멋진 enum 변수를 사용한 switch/case문 보다는 추상 메서드가 있는 기초  클래스가 더 좋다.
    - switch/case문을 매번 똑같이 구현하게 강제하기는 어렵지만, 파생 클래스는 추상 메서드를 모두 구현하지 않으면 안 되기 때문이다.

  - 조건을 캡슐화하라
    - 부울 논리는 이해하기 어려우므로, 조건의 의도를 분명히 밝히는 함수로 표현하라.

  ```java
  // bad
  if (timer.hasExpired() && !timer.isRecurrent())
  
  // good
  if (shouldBeDeleted(timer))
  ```

  - 부정 조건은 피하라
    - 부정 조건은 긍정 조건보다 이해하기 어렵다.
    - 가능하면 긍정 조건으로 표현한다.

  ```java
  // bad
  if (!buffer.shouldNotCompact())
      
  // good
  if (buffer.shouldCompact())
  ```

  - 함수는 한 가지만 해야 한다.
    - 함수를 짜다보면 한 함수 안에 여러 단락을 이어, 일련의 작업을 수행하고픈 유혹에 빠진다.
    - 이런 함수는 한 가지만 수행하는 함수가 아니므로, 한 가지만 수행하는 좀 더 작은 함수로 여럿으로 나눠야 마땅하다.

  ```java
  // bad
  public void pay() {
      for (Employee e : employees) {
          if (e.isPayday()) {
              Money pay = e.calculatePay();
              e.deliverPay(pay);
          }
      }
  }
  
  // good
  public void pay() {
      for (Employee e : employees) {
          payIfNeccessary(e);
      }
  }
  
  private void payIfNeccessary(Employee e) {
      if (e.isPayday()) {
          calculateAndDeliverPay(e);
      }
  }
  
  private void calculateAndDeliverPay(Employee e) {
      Money pay = e.calculatePay();
      e.deliverPay(pay);
  }
  ```

  - 시간적 결합은 숨겨서는 안 된다.
    - 때로는 시간적인 결합이 필요할 수 있다.
    - 하지만 시간적인 결합을 숨겨서는 안 되나.
    - 실행되는 순서가 중요하다면 시간적인 결합을 노출하고, 순서대로 호출될 수 있도록 강제해야한다.

  ```java
  // bad
  public class MooqDiver {
      Gradient gradient;
      List<Spline> splines;
      
      public void dive(String reason) {
          saturateGradient();
          reticulateSplines();
          diveForMooq(reason);
      }
  }
  
  // good
  public class MooqDiver {
      Gradient gradient;
      List<Spline> splines;
      
      public void dive(String reason) {
          gradient = saturateGradient();
          List<Spline> splines = reticulateSplines();
          diveForMooq(splines, reason);
      }
  }
  ```

  - 일관성을 유지하라
    - 코드 구조를 잡을 때는 이유를 고민해야하며, 그 이유를 코드 구조로 명백히 표현해야한다.
    - 구조에 일관성이 없어 보인다면 남들이 맘대로 바꿔도 괜찮다고 생각할 것이다.
  - 경계 조건을 캡슐화하라
    - 경계 조건은 빼먹거나 놓치기 십상이다.
    - 경계 조건은 한 곳에서 별도로 처리해야하며, 코드 여기저기에서 처리하면 안 된다.

  ```java
  // bad
  if (level + 1 < tags.length) {
      parts = new Parse(tags, level + 1);
  }
  
  // good
  int nextLevel = level + 1;
  if (nextLevel < tags.length) {
      parts = new Parse(tags, nextLevel);
  }
  ```

  - 함수는 추상화 수준을 한 단계만 내려가야 한다.
    - 함수 내 모든 문장은 추상화 수준이 동일해야 하며, 그 추상화 수준은 함수 이름이 의미하는 작업보다 한 단계만 낮아야 한다.
    - 예를 들어 아래 1번 코드는 추상화 수준이 최소한 두 개가 섞여 있는데, 첫 번째는 수평선에 크기가 있다는 개념이고, 두 번째는 HR 태그 자체의 문법이다.
    - 2번 코드는 1번 코드의 뒤섞인 추상화 수준을 분리한 것이다.
    - `render` 함수는 HR 태그만 생성하며, HR 태그의 문법은 `HtmlTag`가 알아서 처리하게 한다.
    - 추상화 수준 분리는 리팩터링을 수행하는 가장 중요한 이유 중 하나지만, 제대로 하기에 가장 어려운 작업이기도 하다.

  ```java
  // 1
  public String render() throws Exception {
      StringBuffer html = new StringBuffer("<hr");
      if (size > 0) {
          html.append(" size=\"").append(szie + 1).append("\"");
      }
      html.append(">");
     
      return html.toString();
  }
  
  
  // 2
  public String render() throws Exception {
      HtmlTag hr = new HtmlTag("hr");
      if (extraDashes > 0) {
          hr.addattribute("size", hrSize(extraDashes));
      }
      return hr.html();
      
      private String hrSize(int height) {
          int hrSize = height + 1;
          return String.format("%d", hrSize);
      }
  }
  ```

  - 설정 정보는 최상위 단계에 둬라
    - 추상화 최상위 단계에 둬야 할 기본값 상수나 설정 관련 상수를 저차원 함수에 숨겨서는 안 된다.
    - 대신 고차원 함수에서 저차원 함수를 호출할 때 인자로 넘긴다.
    - 설정 관련 상수는 최상위 단계에 둬야 변경이 쉽다.
  - 추이적 탐색을 피하라
    - 일반적으로 한 모듈은 주변 모듈을 모를수록 좋다.
    - 좀 더 구체적으로 A가 B를 사용하고, B가 C를 사용한다고해서, A가 C를 알아야 할 필요는 없다.
    - 이를 디미터의 법칙이라 부른다.
    - 내가 아는 모듈이 연이어 자신이 아는 모듈을 따라가며 시스템 전체를 휘젓게 둬서는 안 된다.
    - 예를 들어 `a.getB().getC()`와 같은 코드는 바람직하지 않다.
    - 이 형태는 설계를 변경해야 할 때 B와 C 사이에 Q를 넣기 힘들게 만든다.
    - 내가 사용하는 모듈이 내게 필요한 서비스를 모두 제공해야 하며, 원하는 메서드를 찾느라 객체 그래프를 따라 시스템을 탐색할 필요가 없어야한다.



- 주석
  - 부적절한 정보
    - 다른 시스템에(소스 코드 관리 시스템, 버그 추적 시스템, 이슈 추적 시스템, 기타 디록 관리 시스템 등)저장할 정보는 주석으로 적절하지 못하다.
    - 예를 들어 변경 이력은 소스 코드만 번잡하게 만든다.
    - 일반적으로 작성자, 최종 수정일, SPR(Software Problem Report) 번호 등과 같은 메타 정보만 주석으로 넣는다.
    - 주석은 코드와 설계에 기술적인 설명을 부연하는 수단이다.
  - 쓸모 없는 주석
    - 오래된 주석, 엉뚱한 주석, 잘못된 주석은 더 이상 쓸모가 없다.
    - 주석은 빨리 낡으므로 쓸모 없어질 주석은 아예 달지 않는 편이 가장 좋다.
    - 쓸모 없어진 주석은 재빨리 삭제하는 편이 가장 좋다.
    - 쓸모 없는 주석은 일단 들어가고 나면 코드에서 쉽게 멀어진다.
    - 코드와 무관하게 따로 놀며 코드를 그릇된 방향으로 이끈다.
  - 중복된 주석
    - 코드만으로 충분한데 구구절절 설명하는 주석이 중복된 주석이다.
    - 함수 시그니처만 기술하는 Javadoc 역시 쓸모없는 주석이다.
  - 성의 없는 주석
    - 작성할 가치가 있는 주석은 잘 작성해야한다.
    - 주석을 달아야 한다면 시간을 들여 최대한 멋지게 작성한다.
    - 단어를 신중하게 선택하고 문법과 구두점을 올바로 사용한다.
    - 주절대지 않고, 당연한 소리를 반복하지 않으며, 간결하고 명료하게 작성한다.
  - 주석 처리된 코드
    - 주석 처리된 코드는 얼마나 오래된 코드인지, 중요한 코드인지 아닌지 알 길이 없으며 아무도 삭제하지 않는다.
    - 그렇게 자신이 포함된 모듈을 오염시키며 읽는 사람을 헷갈리게 만든다.
    - 주석으로 처리된 코드를 발견하면 즉각 지워버려라.



- 환경
  - 빌드는 간단해야 한다.
    - 빌드는 간단히 한 단계로 끝나야 한다.
    - 소스 코드 관리 시스템에서 이것 저것 따로 체크아웃할 필요가 없어야한다.
    - 불가해한 명렬ㅇ이나 스크립트를 잇달아 실행해 각 요소를 따로 빌드할 필요가 없어야 한다.
    - 한 명령으로 전체를 체크아웃해서 한 명령으로 빌드할 수 있어야 한다.
  - 여러 단계로 테스트해야 한다.
    - 모든 단위 테스트는 한 명령으로 돌려야 한다.
    - 모든 테스트를 한 번에 실행하는 능력은 아주 근본적이고 아주 중요하다.



- 함수
  - 너무 많은 인수
    - 함수에서 인수 개수는 적을 수록 좋다.
    - 넷 이상은 그 가치가 아주 의심스러우므로 최대한 피한다.
  - 출력 인수
    - 출력 인수는 작관을 정면으로 위배한다.
    - 일반적으로 독자는 인수를 출력이 아닌 입력으로 간주한다.
    - 함수에서 뭔가의 상태를 변경해야 한다면 출력 인수를 사용할 게 아니라 함수가 속한 객체의 상태를 변경한다.
  - 플래그 인수
    - boolean 인수는 함수가 여러 기능을 수행한다는 명백한 증거다.
    - 플래그 인수는 혼란을 초래하므로 피해야 마땅하다.
  - 죽은 함수
    - 아무도 호출하지 않는 함수는 삭제한다.
    - 죽은 코드는 낭비다.



- Java
  - 긴 import 목록을 피하고 와일드카드를 사용하라.
    - 긴 import 목록은 읽기에 부담스럽다.
    - 사용하는 패키지를 간단히 명시하면 충분하다.
    - 와일드카드 import 문은 때로 이름 충돌이나 모호성을 초래한다.
    - 이름이 같으나 패키지가 다른 클래스는 명시적인 import문을 사용하거나 아니면 코드에서 클래스를 사용할 때 전체 경로를 명시한다.
  - 상수는 상속하지 않는다.
    - 상수를 인터페이스에 선언하고 그 상수를 상속해 해당 상수를 사용해선 안 된다.
    - 인터페이스는 행위의 집합이지 상수를 공유하기 위한 계층이 아니다.
    - 대신 static import를 사용하라.
  - 상수 대 enum
    - 상수는 코드에서 의미를 잃어버리기도 하지만 enum은 그렇지 않다.
    - Enum은 강력한 서술적 도구이다.



- 이름
  - 서술적인 이름을 사용하라
    - 소프트웨어 가독성의 대부분은 이름이 결정한다.
    - 시간을 들여 현명한 이름을 선택하고 유효한 상태로 유지한다.
    - 유효한 상태로 유지하라는 말의 의미는 코드의 변경에 따라 더 이상 적절한 이름이 아니게 될 경우 이름을 수정하라는 의미다.
  - 적절한 추상화 수준에서 이름을 선택하라.
    - 구현을 드러내는 이름은 피하라.
    - 작업 대상 클래스나 함수가 위치하는 추상화 수준을 반영하는 이름을 선택하라.
    - 코드를 살펴볼 때마다 추상화 수준이 맞지 않는 이름을 찾아 변경하라.
  - 가능한한 표준 명명법을 사용하라
    - 특정 디자인 패턴을 사용한다면 해당 디자인 패턴의 이름에 맞는 이름을 명명하라.
    - 표준 명명법이 없을 때는 관례를 따라라.
  - 명확한 이름
    - 함수나 변수의 목적을 명확히 밝히는 이름을 선택하라.
    - 이름이 길다는 단점은 서술성이 충분히 메꾼다.
  - 긴 범위는 긴 이름을 사용하라.
    - 이름 길이는 범위 길이에 비례해야 한다.
    - 범위가 작으면 아주 짧은 이름을 사용해도 괜찮지만, 범위가 길어지면 긴 이름을 사용한다.
    - 이름이 짧은 변수나 함수는 범위가 길어지면 의미를 잃는다.
  - 인코딩을 피하라
    - 이름에 타입 정보나 범위 정보를 넣어서는 안 된다.
    - 헝가리안 표기법의 오염에서 이름을 보호하라.
  - 이름으로 부수 효과를 설명하라
    - 명명 대상의 역할을 모두 기술하는 이름을 사용한다.
    - 이름에 부수 효과를 숨겨서는 안된다.
    - 만약 뭔가를 생성고 반환한다면 `create`가 아닌 `createAndReturn`과 같이 지어야한다.



- 테스트

  - 테스트는 충분해야한다.
    - 테스트 케이스는 잠재적으로 문제가 생길만한 부분을 모두 테스트해야 한다.
    - 테스트 케이스가 확인하지 않는 조건이나 검증하지 않는 조건이 있다면 그 테스트는 불완전하다.

  - 커버리지 도구를 사용하라
    - 커버리지 도구는 테스트가 빠뜨리는 공백을 알려준다.
    - 커버리지 도구는 테스트가 불충분한 부분을 쉽게 찾을 수 있게 해준다.
  - 사소한 테스트를 건너뛰지 마라.
    - 사소한 테스트는 짜기 쉽지만, 그 가치는 구현에 드는 비용을 능가한다.
  - 무시한 테스트는 모호함을 뜻한다.
    - 불분명한 요구사항은 테스트 케이스를 주석으로 처리하거나 테스트가 실행될 때 제외시킨다.
    - 선택 기준은 모호함이 존재하는 테스트 케이스가 컴파일이 가능한지 여부에 달려있다.
  - 경계 조건을 테스트하라
    - 경계 조건은 각별히 신경 써서 테스트한다.
    - 알고리즘의 경계 조건에서 실수하는 경우가 흔하다.
  - 버그 주변은 철저히 테스트하라.
    - 버그는 서로 모이는 경향이 있다.
    - 한 함수에서 버그를 발견했다면, 그 함수를 철저히 테스트하는 편이 좋다.
  - 실패 패턴을 살펴라.
    - 테스트 케이스가 실패하는 패턴으로 문제를 진단할 수 있다.
    - 테스트 케이스를 최대한 꼼꼼히 짜야하는 이유도 이 때문이다.
    - 합리적인 순서로 정렬된 꼼꼼한 테스트 케이스는 실패 패턴을 드러낸다.
  - 테스트 커버리지 패턴을 살펴라.
    - 통과하는 테스트가 실행하거나 실행하지 않는 코드를 살펴보면 실패하는 테스트 케이스의 실패 원인이 드러난다.
  - 테스트는 빨라야한다.
    - 테스트가 느리면 실행하지 않게 된다.
    - 테스트 케이스가 빨리 돌아가게 최대한 노력해야한다.
