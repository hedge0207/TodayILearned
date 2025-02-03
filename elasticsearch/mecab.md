# Mecab 사용해보기

- Mecab 설치하기

  - mecab-ko 설치

  > [링크](https://www.google.com/url?q=https://bitbucket.org/eunjeon/mecab-ko/downloads/mecab-0.996-ko-0.9.1.tar.gz&sa=D&source=editors&ust=1734069486784087&usg=AOvVaw0yTlT3vbWJ_05IaXAArkmt)에서 압축 파일을 다운 받는다.

  ```bash
  $ tar zxfv mecab-0.996-ko-0.9.0.tar.gz
  $ cd mecab-0.996-ko-0.9.0
  $ ./configure
  $ make
  $ make check
  $ sudo make install
  ```

  - mecab-ko-dic 설치

  > [링크](https://bitbucket.org/eunjeon/mecab-ko-dic/downloads/)에서 최신 버전의 mecab-ko-dic을 다운 받는다.

  ```bash
  $ tar zxfv mecab-ko-dic-<version>.tar.gz
  $ cd mecab-ko-dic-<version>
  $ ./configure
  $ make
  $ sudo make install
  ```

  - mecab-ko-dic 실행해보기

  ```bash
  $ echo "설치가 완료되었습니다." | mecab -d /usr/local/lib/mecab/dic/mecab-ko-dic
  
  # 결과가 아래 순서로 출력된다.
  표현층 품사,의미부류,받침유무,발음,type,시작형태소,끝형태소,원형표현,색인표현
  ```



- 사전에 단어 추가하기

  - 원래 "이체수수료"라는 단어는 아래와 같이 분석된다.
    - "이체수수료"로 분석되길 원하기에, 해당 단어를 사전에 추가한다.

  ```bash
  echo "이체수수료" | mecab -d /usr/local/lib/mecab/dic/mecab-ko-dic
  이	MM,~명사,F,이,*,*,*,*
  체수	NNG,*,F,체수,*,*,*,*
  수료	NNG,행위,F,수료,*,*,*,*
  ```

  - 사전 파일 작성하기
    - 임의의 위치에 아래와 같이 사전 파일을 생성 후 CSV 형식으로 단어를 추가한다.

  ```
  # my-dict_in.csv
  이체수수료,,,,NNG,*,F,이체수수료,*,*,*,*,*
  ```

  - 위에서 작성한 사전 파일을 읽어서 좌우 문맥 ID와 낱말 비용을 생성한다.
    - `-m` 에는 model.def 파일의 경로를 입력한다.
    - `-d` 에는 mecab-ko-dict의 경로를 입력한다.
    - `-u`에는 결과 파일의 경로를 입력한다.
    - `-f`, `-t`에는 encoding 방식을 입력한다.
    - `-a`에는 위에서 작성한 사전 파일의 경로를 입력한다.

  ```bash
  $ /usr/local/libexec/mecab/mecab-dict-index \
  -m </path/to/mecab-ko-dic>/model.def \
  -d </path/to/mecab-ko-dic> \
  -u </path/to/mecab-ko-dic>/my-dict.csv \
  -f utf-8 \
  -t utf-8 \
  -a ./my-dict_in.csv
  ```

  - 결과 확인
    - 좌우 문맥 ID와 낱말 비용이 계산되어 추가된 것을 볼 수 있다.

  ```bash
  $ cat ./my-dict.csv
  
  # output
  이체수수료,1780,3533,2639,NNG,*,F,이체수수료,*,*,*,*,*
  ```

  - 적용 여부 확인하기
    - 아직 적용되지 않은 것을 볼 수 있다.
    - 위에서 좌우 문맥 ID와 낱말 비용만 계산했을 뿐, 아직 해당 내용을 사전에 적용하진 않았기 때문이다.

  ```bash
  $ echo "이체수수료" | mecab -d /usr/local/lib/mecab/dic/mecab-ko-dic
  이	MM,~명사,F,이,*,*,*,*
  체수	NNG,*,F,체수,*,*,*,*
  수료	NNG,행위,F,수료,*,*,*,*
  ```

  - 추가한 사전을 적용하기 위해서는 사전을 컴파일하고 설치해야한다.
    - 만약 이 때 위에서 사전 생성을 위해 생성한 `my-dict_in.csv`파일이 mecab-ko-dic 경로에 있다면, 다른 곳으로 옮기거나 삭제해야한다.
    - 이는 해당 폴더에 있는 모든 `.csv` 파일을 읽으면서 컴파일과 설치가 진행되기 때문이다.

  ```bash
  $ make clean
  $ make
  $ sudo make install
  ```

  - 확인
    - 사전이 등록된 것을 확인할 수 있다.

  ```bash
  $ echo "이체수수료" | mecab -d /usr/local/lib/mecab/dic/mecab-ko-dic
  이체수수료	NNG,*,F,이체수수료,*,*,*,*,*
  ```
