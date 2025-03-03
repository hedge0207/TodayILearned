# Index File Formats

> https://lucene.apache.org/core/9_10_0/core/org/apache/lucene/codecs/lucene99/package-summary.html#package.description

- Lucene의 index에는 아래와 같은 것들이 저장된다.

  - Segment Info
    - Segement에 대한 metadata를 저장한다.
    - 문서의 개수, 사용되는 파일, segment가 저장된 방식 등이 저장되어 있다.
  - Field names
    - Index에서 사용하는 field들이 metadata가 저장되어 있다.
  - Stored Field values
    - 각 문서에 대한 field name-value 쌍의 목록이 저장되어 있다.
    - 저장된 쌍들은 검색시에 결과로 반환된다.
  - Term dictionary
    - 색인된 문서에 있는 term들에 대한 정보를 저장한다.
    - 각 Term을 가지고 있는 document의 개수와 term의 frequency 등에 대한 정보도 저장되어 있다.
  - Term Frequency data
    - Term dictionary에 있는 각각의 term을 포함하고 있는 document의 개수나 document 내에서 term이 등장하는 빈도 등이 저장된다.
  - Term Proximity data
    - Term dictionary에 있는 각각의 term이 document 내에서 어느 위치에 있는지를 저장한다.
  - Normalization factors
    - Document 내부의 각 field에 대해 검색시에 부여되는 가중치가 저장되어 있다.
  - Term Vectors
    - Document 내부의 각 field에 대해 term vector(==document vector)가 저장되어 있다.
    - Term vector는 term text와 term frequency로 구성된다.
  - Per-document values
    - Stored value와 유사하지만, per-document values의 경우 빠른 접근을 위해 main memory에 load된다.
    - Stored value는 일반적으로 검색 결과를 요약하기 위해 사용되지만, per-document values는 scoring에 사용된다.
  - Live documents
    - 어떤 document가 삭제되지 않고 남아있는지를 저장한다.

  - Point values
    - 큰 숫자 값(BigInteger, BigDecimal)이나 지리적 데이터를 다루기 위해 여러 차원으로 색인된 field을 저장한다.
  - Vector values
    - 고차원의 최근접 이웃 검색을 위해 random access와 연산에 최적화 된 vector 값들을 저장한다.



- Lucene에서 사용하는 파일들

  - 만약 Compound File format을 사용하면 아래 file들을 다 사용하지는 않고, 아래 파일들이 하나의 `.cfs` 파일로 합쳐진다.
  - Compound File format을 사용하더라도 Segment info file, Lock file, Deleted documents file등은 합쳐지지 않는다.

  | Name                | Extension              | Description                                                  |
  | ------------------- | ---------------------- | ------------------------------------------------------------ |
  | Segments File       | segments_N             | Commit point에 대한 정보를 저장한다.                         |
  | Lock File           | write.lock             | 여러 IndexWriter가 같은 파일에 쓰는 것을 방지하기 위한 lock file이다. |
  | Segment Info        | .si                    | Segment에 대한 metadata를 저장한다.                          |
  | Compound Fike       | .cfs, .cfe             | 다른 모든 index file들을 합쳐서 만드는 파일이다.             |
  | Fields              | .fnm                   | Field들에 대한 정보를 저장한다.                              |
  | Field Index         | .fdx                   | Field data에 대한 pointer를 저장한다.                        |
  | Field data          | .fdt                   | Stored field를 저장한다.                                     |
  | Term Dictionary     | .tim                   | Term dictionary를 저장한다.                                  |
  | Term index          | .tip                   | Term dictionary의 색인을 저장한다.                           |
  | Frequencies         | .doc                   | Term이 등장하는 document들의 목록을 frequency와 함께 저장한다. |
  | Positions           | .pos                   | Term이 index의 어느 위치에 등장하는지에 대한 정보를 저장한다. |
  | Payloads            | .pay                   | 추가적인 per-position metadata를 저장한다.                   |
  | Norms               | .nvd, .nvm             | Documents와 fields에 대한 길이 정보와 boost factor를 인코딩하여 저장한다. |
  | Per-document Values | .dvd, .dvm             | 추가적인 scoring factor와 per-document 정보를 인코딩하여 저장한다. |
  | Term Vector Index   | .tvx                   | Offset을 저장한다.                                           |
  | Term Vector Data    | .tvd                   | Term vector를 저장한다.                                      |
  | Live Documents      | .liv                   | 각 문서들의 삭제 여부를 저장한다.                            |
  | Point values        | .dii, dim              | 색인된 point들을 저장한다.                                   |
  | Vector values       | .vec, .vem, .veq, .vex | 색인된 vector 정보를 저장한다.                               |



- Lucene 색인 파일 생성해보기

  - 아래 코드를 실행한다.
    - Lucene에서는 `IndexWriter`를 사용하여 문서를 색인한다.
    - 문서를 색인한 후에 `commit`을 해준다.

  ```java
  package org.apache.lucene;
  
  import org.apache.lucene.document.Document;
  import org.apache.lucene.document.Field;
  import org.apache.lucene.document.StringField;
  import org.apache.lucene.document.TextField;
  import org.apache.lucene.index.*;
  import org.apache.lucene.search.*;
  import org.apache.lucene.store.Directory;
  import org.apache.lucene.store.NIOFSDirectory;
  
  import java.io.IOException;
  import java.nio.file.Paths;
  
  public class MyTest {
      public static void main(String[] args) throws IOException {
          String index_dir = "/path/to/create/index/files";
          Directory index = new NIOFSDirectory(Paths.get(index_dir));
          IndexWriterConfig config = new IndexWriterConfig();
          IndexWriter writer = new IndexWriter(index, config);
  
          // create a document
          Document doc = new Document();
          doc.add(new TextField("title", "Lucene - IndexWriter", Field.Store.YES));
          doc.add(new StringField("author", "aliyun", Field.Store.YES));
  
          //index a document
          writer.addDocument(doc);
          writer.commit();
      }
  }
  ```

  - 설정한 경로에 아래 파일들이 생성된다.
    - `.cfs`, `.cfe`, `.si`, `segments_`, `write.lock`
    - Segment의 크기가 작을 경우 Compound File format이 기본값이므로 `.cfs`, `.cfe` 파일이 생성된다.





# Lucene Anaylzer 동작 과정

- 사전 지식

  - `Tokenizer`와 `TokenFilter`는 `TokenStream`을 상속 받는다.

  ```java
  public abstract class Tokenizer extends TokenStream {
    	// ...
  }
  
  public abstract class TokenFilter extends TokenStream implements Unwrappable<TokenStream> {
      // ...
  }
  ```

  - Token filter들은 `TokenFilter`를 상속 받으며, `TokenFilter`는 `TokenStream`을 상속받는다.

  ```java
  public abstract class TokenFilter extends TokenStream implements Unwrappable<TokenStream> {
      // ...
  }
  
  public class LowerCaseFilter extends TokenFilter {
      // ...
  }
  ```

  - `TokenStream`은 `AttributeSource`를 상속 받는다.

  ```java
  public abstract class TokenStream extends AttributeSource implements Closeable {
      // ...
  }
  ```

  - `TokenStream`과 `TokenStreamComponents`의 관계
    - `TokenStream`은 `TokenStreamComponents`의 구성 요소이다.
    - `TokenStreamComponents`는 하나의 `Tokenizer`와 `TokenStream`을 포함한다.
    - `TokenStreamComponents`는 `Tokenizer`를 통해 생성한 개별 token들을 대상으로 `TokenStream`을 통해 token filter 등의 후속 처리를 실행한다.



- 우선 단순한 `WhitespaceAnalyzer`를 기준으로 Lucene analyzer의 동작 과정을 살펴본다.

  - 실행 코드

  ```java
  import org.apache.lucene.analysis.Analyzer;
  import org.apache.lucene.analysis.TokenStream;
  import org.apache.lucene.analysis.core.WhitespaceAnalyzer;
  import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
  
  import java.io.IOException;
  
  
  public class Main {
      public static void main(String[] args) {
          // analyzer 생성
          Analyzer analyzer = new WhitespaceAnalyzer();
          // token stream 생성
          try (TokenStream tokenStream = analyzer.tokenStream("Hello World!")) {
              // token stream 초기화
              tokenStream.reset();
              // token 생성
              while (tokenStream.incrementToken()) {
                  CharTermAttribute termAttr = tokenStream.getAttribute(CharTermAttribute.class);
                  System.out.println(termAttr.toString());
              }
              // token stream 종료
              tokenStream.end();
          } catch (IOException e) {
              throw new RuntimeException(e);
          }
          // anayzer 닫기
          analyzer.close();
      }
  }
  
  /* 실행 결과
  Hello
  World!
  */
  ```

  - `WhitespaceTokenizer`

  ```java
  public final class WhitespaceTokenizer extends CharTokenizer {
  
    public WhitespaceTokenizer() {}
  
    public WhitespaceTokenizer(AttributeFactory factory) {
      super(factory);
    }
  
    public WhitespaceTokenizer(int maxTokenLen) {
      super(TokenStream.DEFAULT_TOKEN_ATTRIBUTE_FACTORY, maxTokenLen);
    }
  
    public WhitespaceTokenizer(AttributeFactory factory, int maxTokenLen) {
      super(factory, maxTokenLen);
    }
  
    @Override
    protected boolean isTokenChar(int c) {
      return !Character.isWhitespace(c);
    }
  }
  ```

  - `WhitespaceAnalyzer`
    - `WhitespaceAnalyzer`는 `Analyzer`를 상속 받는다.

  ```java
  package org.apache.lucene.analysis.core;
  
  import org.apache.lucene.analysis.Analyzer;
  
  
  public final class WhitespaceAnalyzer extends Analyzer {
  
    private final int maxTokenLength;
  
    public WhitespaceAnalyzer() {
      this(WhitespaceTokenizer.DEFAULT_MAX_WORD_LEN);
    }
  
    public WhitespaceAnalyzer(int maxTokenLength) {
      this.maxTokenLength = maxTokenLength;
    }
  
    @Override
    protected TokenStreamComponents createComponents(final String fieldName) {
      return new TokenStreamComponents(new WhitespaceTokenizer(maxTokenLength));
    }
  }
  ```

  - `TokenStreamComponents` 생성
    - `Analyzer.tokenStream()` method가 호출되면 `TokenStreamComponent`와 `TokenStreamComponents`의 `TokenStream`이 생성된다.
    - 성능 향상을 위해 특정 field를 대상으로 이전에 생성한 `TokenStreamComponents`가 있으면 그걸 재사용하고, 없을 경우에만 새로 생성한다.
    - 이 경우 기존에 생성한 `TokenStreamComponents`가 없을 것이므로 `WhitespaceAnalyzer.createComponents()` method가 실행된다.
    - `TokenStreamComponents`는 생성될 때, `Tokenizer`와 `TokenStream`을 받는데, `WhitespaceAnalyzer`의 경우 추가적인 token filter는 없으므로, `WhitespaceTokenizer`만을 받아 생성된다.
    - `TokenStreamComponents`는 생성될 때 `sink`라는 변수에 `TokenStream`을 저장하는데, `TokenStreamComponents.getTokenStream()`는 `sink`를 반환한다.
    - `WhitespaceAnalyzer`의 경우  `TokenStreamComponents`를 생성할 때, tokenizer만 넘겼으므로, `sink`에는 tokenizer가 저장된다.

  ```java
  // Analyzer.tokenStream()
  public final TokenStream tokenStream(final String fieldName, final String text) {
      TokenStreamComponents components = reuseStrategy.getReusableComponents(this, fieldName);
      @SuppressWarnings("resource")
      final ReusableStringReader strReader =
          (components == null || components.reusableStringReader == null)
              ? new ReusableStringReader()
              : components.reusableStringReader;
      strReader.setValue(text);
      final Reader r = initReader(fieldName, strReader);
      // 만약 기존에 parameter로 받은 field를 대상으로 생성했던 TokenStreamComponents가 없으면
      if (components == null) {
        // TokenStreamComponents를 생성한다.
        components = createComponents(fieldName);
          // 해당 field를 대상으로 생성한 TokenStreamComponents를 등록한다/
        reuseStrategy.setReusableComponents(this, fieldName, components);
      }
  
      components.setReader(r);
      components.reusableStringReader = strReader;
      // 생성한 TokenStreamComponents의 TokenStream을 반환한다.
      return components.getTokenStream();
  }
  ```

  - `TokenStream.reset()`
    - Tokenizing에 필요한 사전 준비를 하는 method로, `TokenStream`은 이전 상태를 유지하기 때문에, 이전 상태를 초기화하기 위해 사용한다.
    - 즉 내부에서 사용된 상태나 입력 스트림을 초기화하여, 새로운 입력 텍스트를 처리할 수 있도록 만든다.
    - `TokenStream`에 abstract method로 선언되어 있으며, `TokenStream`을 상속 받는 `Tokenizer`를 상속 받는 class마다 override하여 사용한다.
    - `WhitespaceTokenizer`의 경우 따로 override하지는 않고, `Tokenizer.reset()`을 그대로 상속 받아 사용한다.

  ```java
  public abstract class Tokenizer extends TokenStream {
    // ...
    @Override
    public void reset() throws IOException {
      super.reset();
      input = inputPending;
      inputPending = ILLEGAL_STATE_READER;
    }
  }
  ```

  - `TokenStream.incrementToken()`
    - `incrementToken()` method는 token을 생성하는 역할을 한다.
    - `TokenStream`에 abstract method로 선언되어 있으며, `TokenStream`을 상속 받는 `Tokenizer`를 상속 받는 class들에 구현되어 있다.
    - `WhitespaceTokenizer`의 경우 따로 구현되어 있지는 않고, 자신이 상속 받는 `CharTokenizer.incrementToken()`을 사용한다.
    - 다만 각 문자에 대해 토큰인지 여부를 판단하는 `isTokenChar()` method의 경우 `WhitespaceTokenizer`에서 override한 method를 사용한다.
    - `buffer`는 `termAtt.termBuffer`를 참조하므로 `buffer`의 변경사항은 `termAtt.termBuffer`에도 반영된다.
    - `buffer`에 새로운 token의 character들을 덮어쓰는 방식으로 동작하므로, 이전 token의 정보를 알 수 있는 방법은 제공하지 않는다.

  ```java
  public abstract class CharTokenizer extends Tokenizer {
  
    //...
    @Override
    public final boolean incrementToken() throws IOException {
      clearAttributes();
      int length = 0;		// 현재 token의 길이
      int start = -1;		// 현재 token의 시작 offsest
      int end = -1;		// 현재 token의 마지막 offset
      
      // 현재 token을 저장하기 위한 배열
      char[] buffer = termAtt.buffer();
      while (true) {
        // bufferIndex와 dataLen은 instance 생성시에 0으로 초기화된다.
        if (bufferIndex >= dataLen) {
          offset += dataLen;
          CharacterUtils.fill(ioBuffer, input); // input에서 data를 읽어서 ioBuffer에 저장한다.
          if (ioBuffer.getLength() == 0) {	// ioBuffer.getLength는 아직 읽지 않은 문자 배열의 길이를 반환한다.
            dataLen = 0;
            if (length > 0) {
              break;
            } else {
              // 더 이상 읽지 않은 문자가 없고, 현재 token의 길이가 0이면 false를 반환한다.
              finalOffset = correctOffset(offset);
              return false;
            }
          }
          dataLen = ioBuffer.getLength();
          bufferIndex = 0;
        }
        final int c = Character.codePointAt(ioBuffer.getBuffer(), bufferIndex, ioBuffer.getLength());
        final int charCount = Character.charCount(c);
        bufferIndex += charCount;
  
        if (isTokenChar(c)) {		// 유효한 token일 경우
          if (length == 0) {
            assert start == -1;
            start = offset + bufferIndex - charCount;
            end = start;
          } else if (length >= buffer.length - 1) {
            buffer = termAtt.resizeBuffer(2 + length);
          }
          end += charCount;
          length += Character.toChars(c, buffer, length);	// buffer에 character를 추가하고, length를 증가시킨다.
          if (length >= maxTokenLen) {
            break;
          }
        } else if (length > 0) {
          break;
        }
      }
  	
      termAtt.setLength(length);		
      assert start != -1;
      offsetAtt.setOffset(correctOffset(start), finalOffset = correctOffset(end));
      return true;
    }
  }
  ```

  - 동작과정
    - `input`에 "Hello World!"가 저장된다.
    - `CharacterUtils.fill(ioBuffer, input);`을 통해 `ioBuffer`에 "Hello World!"가 char 배열로 저장된다.
    - `final int c = Character.codePointAt(ioBuffer.getBuffer(), bufferIndex, ioBuffer.getLength());`를 통해 character에 해당하는 유니코드 코드 포인트를 가져온다.
    - `isTokenChar(c)`를 통해 유효한 character인지 확인한다.
    - `Character.toChars(c, buffer, length);`를 통해 character를 `buffer`에 추가하고, `length`를 증가시킨다.
    - 첫 반복을 돌았을 때, `buffer`의 상태는 `[H,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`와 같다.
    - 이후 유효하지 않은 character가 나오거나 더 이상 읽어올 character가 없을 때까지 반복을 거듭하면서 `buffer`는 아래와 같이 변화한다.
    - `[H, e,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`
    - `[H, e, l,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`
    - `[H, e, l, l,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`
    - `[H, e, l, l, o,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`
    - 이후 유효하지 않은 character(whitespace)를 만나게 되고, `length`의 길이가 0보다 크므로 반복문이 종료된다.
    - `termAtt.setLength(length);`를 통해 `termAtt.termBuffer`의 길이가 설정된다.
    - 아직 token이 남아있다는 의미로 true가 반환된다.
    - `termAttr.toString()`은 `termAttr.termBuffer` 배열에서 `termAtt.setLength(length)`를 통해 설정된 길이 만큼을 잘라서 String으로 변환 후 반환한다.
    - 다시 `incrementToken()` method가 호출될 때, buffer의 상태는 `[H, e, l, l, o,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`와 같다.
    - 이후 다시 유효하지 않은 character를 만나거나, 더 이상 읽어올 character가 없을 때 까지 반복하면서 아래와 같이 변화한다.
    - `[W, e, l, l, o,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`
    - `[W, o, l, l, o,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`
    - `[W, o, r, l, o,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`
    - `[W, o, r, l, d,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`
    - `[W, o, r, l, d, !,  ,  ,  ,  ,  ,  ,  ,  ,  ,  ]`
    - 더 이상 읽어올 character가 없고, `length`가 0보다 크므로 반복문이 종료된다.
    - `termAtt.setLength(length);`를 통해 `termAtt.termBuffer`의 길이가 설정된다.
    - 아직 token이 남아있다는 의미로 true가 반환된다.
    - 다시 `incrementToken()` method가 호출되고, 더 이상 읽어올 character가 없고, `length`가 0이므로 false가 반환된다.

