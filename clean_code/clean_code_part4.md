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

