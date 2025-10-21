# CRUD 실행하기

## Insert

- Insert 실행하기

  - `insert()` method를 실행하여 새로운 data를 삽입하는 것이 가능하다.
    - `insert()` method는 `Insert`의 instance를 생성하며, `Insert`는 SQL의 INSERT문을 표현하는 class이다.

  ```python
  from sqlalchemy import String, insert, Table, Column, Integer, String, MetaData
  
  
  metadata = MetaData()
  
  user_table = Table(
      "user_account",
      metadata,
      Column("id", Integer, primary_key=True),
      Column("name", String(30)),
      Column("fullname", String),
  )
  
  # insert()는 Insert class의 instance를 반환하며 Insert instance의 values() method를 사용해 값을 설정한다.
  stmt = insert(user_table).values(name="spongebob", fullname="Spongebob Squarepants")
  print(stmt)		# INSERT INTO user_account (name, fullname) VALUES (:name, :fullname)
  ```

  - `compile()` method를 사용하여 문자열화 된 형식을 얻을 수 있다.

  ```python
  compiled = stmt.compile()
  print(compiled.params)		# {'name': 'spongebob', 'fullname': 'Spongebob Squarepants'}
  ```

  - SQL문 실행하기
    - INSERT문은 row를 반환하지 않지만, RETURNING 절을 지원하는 DB의 경우 값을 반환하기도 한다.

  ```python
  # table 생성
  user_table.metadata.create_all(engine)
  
  # SQL문 실행
  with engine.connect() as conn:
      result = conn.execute(stmt)
      conn.commit()
      print(result.inserted_primary_key)		# (1,)
  ```

  - INSERT는 보통 VALUES 절을 자동으로 생성한다.
    - 위 예시에서는 `Insert.values()`를 사용하여 값을 설정해줬다.
    - 값을 설정하지 않더라도 VALUES 절 자체는 추가가 된다.

  ```python
  # VALUES 절이 자동으로 추가된 것을 확인할 수 있다.
  print(insert(user_table))		# INSERT INTO user_account (id, name, fullname) VALUES (:id, :name, :fullname)
  ```

  - 이를 이용하여 아래와 같이 SQL문을 실행할 때 값을 설정하는 것이 가능하다.
    - 사실 이 방식이 보다 일반적인 방식이다.
    - `execute()`의 첫 번째 인자로 `Insert`의 instance를 넘기고, 두 번째 인자로 삽입할 값을 넘기면, SQLAlchemy가 이들을 조합하여 SQL문을 만든다.

  ```python
  with engine.connect() as conn:
      result = conn.execute(
          insert(user_table),
          [
              {"name": "sandy", "fullname": "Sandy Cheeks"},
              {"name": "patrick", "fullname": "Patrick Star"},
          ],
      )
      conn.commit()
  ```



- INSERT...RETURNING

  - RETURNING 절을 지원하는 DB를 대상으로 사용할 때는, RETUNING 절이 자동으로 포함된다.
    - RETURNING 절을 지원하는 DB들은 RETURNING 절에서 기본 값으로 반환하는 값(일반적으로 마지막으로 insert된 data의 primary key)을 가지고 있다.
    - SQLite는 RETURNING절을 지원하며, RETURNING 절이 있을 떄 기본 값으로 마지막으로 insert된 data의 primary key를 반환하도록 되어 있다.
    - SQLite를 사용한 위 예시에서 RETURNING절이 자동으로 포함되었고, RETURNING절이 반환하는 기본 값인 마지막으로 insert된 data의 primary key가 반환되었다.
  - 아래와 같이 명시적으로 지정하는 것도 가능하다.
    - `Insert.returning()` method를 사용하면 된다.	

  ```python
  insert_stmt = insert(user_table).returning(
      user_table.c.id, user_table.c.name
  )
  print(insert_stmt)		
  # INSERT INTO user_account (id, name, fullname) VALUES (:id, :name, :fullname) RETURNING user_account.id, user_account.name
  ```

  - INSERT..FROM SELECT와 조합하는 것도 가능하다.

  ```python
  select_stmt = select(user_table.c.id, user_table.c.name)
  insert_stmt = insert(user_table).from_select(
      ["id", "name"], select_stmt
  )
  print(insert_stmt.returning(user_table.c.id, user_table.c.name))
  """
  INSERT INTO user_account (id, name) SELECT user_account.id, user_account.name 
  FROM user_account RETURNING user_account.id, user_account.name
  """ 
  ```




- INSERT...FROM SELECT

  - `Insert.from_select()` method를 사용하여 SELECT문으로 조회한 문서를 바로 삽입할 수 있다.

  ```python
  # SELECT문을 작성하고
  select_stmt = select(user_table.c.id, user_table.c.name + "@aol.com")
  # SELECT문으로 조회한 row들을 삽입한다.
  insert_stmt = insert(address_table).from_select(
      ["user_id", "email_address"], select_stmt
  )
  print(insert_stmt)
  """
  INSERT INTO address (user_id, email_address)
  SELECT user_account.id, user_account.name || :name_1 AS anon_1
  FROM user_account
  """
  ```

  - 일반적으로 table을 copy할 때 사용한다.



## Select

- SELECT문 실행하기

  - `select()` 함수는 `Select`의 instance를 생성한다.
    - `Select` instance는 SELECT query를 실행하는 데 사용된다.
    - `Select`의 instance를 core의 `Connection.execute()`에 인자로 전달하거나 ORM의 `Session.execute()`의 인자로 전달하여 사용하며, 이 method들의 결과로 반환되는 `Result` instance를 통해 조회 결과를 확인할 수 있다.

  ```python
  from sqlalchemy import select
  
  
  metadata = MetaData()
  
  user_table = Table(
      "user_account",
      metadata,
      Column("id", Integer, primary_key=True),
      Column("name", String(30)),
      Column("fullname", String),
  )
  
  user_table.metadata.create_all(engine)
  
  stmt = select(user_table)
  print(type(stmt))		# <class 'sqlalchemy.sql.selectable.Select'>
  print(stmt)
  
  """
  SELECT user_account.id, user_account.name, user_account.fullname 
  FROM user_account
  """
  ```

  - WHERE절과 함께 사용하기
    - `Select.where()`method를 사용하여 WHERE절을 구성할 수 있다.

  ```python
  from sqlalchemy import select
  stmt = select(user_table).where(user_table.c.name == "spongebob")
  print(stmt)
  
  """
  SELECT user_account.id, user_account.name, user_account.fullname
  FROM user_account
  WHERE user_account.name = :name_1
  """
  ```

  - SELECT문을 실행한 결과로 `CursorResult`의 instance가 반환되며, `CursorResult`는 table의 row를 나타내는 `Row` 객체들을 가지고 있는 iterator이다.

  ```python
  with Session(engine) as session:
      session.execute(insert(user_table).values(name="spongebob", fullname="Spongebob Squarepants"))
      result = session.execute(select(user_table).where(user_table.c.name == "spongebob"))
      print(type(result))		# <class 'sqlalchemy.engine.cursor.CursorResult'>
      for row in result:
          print(row)			# (1, 'spongebob', 'Spongebob Squarepants')
  ```

  - Label도 사용할 수 있다.

  ```python
  from sqlalchemy import func, cast
  stmt = select(
      ("Username: " + user_table.c.name).label("username"),
  ).order_by(user_table.c.name)
  with engine.connect() as conn:
      for row in conn.execute(stmt):
          print(f"{row.username}")
          
  """
  Username: patrick
  Username: sandy
  Username: spongebob
  """
  ```

  - Textual column expression을 사용하는 것도 가능하다.

  ```python
  from sqlalchemy import text
  stmt = select(text("'some phrase'"), user_table.c.name).order_by(user_table.c.name)
  with engine.connect() as conn:
      print(conn.execute(stmt).all())		
  
  # [('some phrase', 'patrick'), ('some phrase', 'sandy'), ('some phrase', 'spongebob')]
  ```



- COLUMNS와 FROM절 설정하기

  - `select()`에 table만 넣을 경우 해당 table의 모든 column을 조회한다.

  ```python
  print(select(user_table))
  """
  SELECT user_account.id, user_account.name, user_account.fullname
  FROM user_account
  """
  ```

  - 그러나 아래와 같이 조회할 column을 지정하는 것도 가능하다.
    - `Table.c`를 통해 접근할 수 있는 `Column` object를 인자로 넘기면 된다.
    - 혹은 `Table.c["column1", "column2"]`와 같이 접근하여 인자로 전달하면 된다.

  ```python
  print(select(user_table.c.name, user_table.c.fullname))
  """
  SELECT user_account.name, user_account.fullname
  FROM user_account
  """
  
  print(select(user_table.c["name", "fullname"]))
  """
  SELECT user_account.name, user_account.fullname
  FROM user_account
  """
  ```

  - ORM mapped class 사용하기
    - 위에서는` Table`의 instance를 인자로 넘겼지만, ORM mapped class 자체를 넘겨도 된다.

  ```python
  from typing import Optional
  
  from sqlalchemy.orm import Mapped, mapped_column, DeclarativeBase
  from sqlalchemy import String, select
  
  
  class Base(DeclarativeBase):
      pass
  
  class User(Base):
      __tablename__ = "user_account"
  
      id: Mapped[int] = mapped_column(primary_key=True)
      name: Mapped[str] = mapped_column(String(30))
      fullname: Mapped[Optional[str]]
  
  
  print(select(User))
  """
  SELECT user_account.id, user_account.name, user_account.fullname
  FROM user_account
  """
  ```

  - ORM mapped class를 사용하여 조회하기
    - `Session.execute()`를 통해 실행할 수 있다.
    - 각 low의 단일 element를 가지는 tuple로 반환된다.

  ```python
  with Session(engine) as session:
      session.execute(insert(User).values(name="spongebob", fullname="Spongebob Squarepants"))
      session.commit()
      rows = session.execute(select(User))
      for row in rows:
          print(row[0])
  ```

  - `Session.scalars()`를 사용하면 보다 간편하게 조회가 가능하다.
    - 각 row가 tuple로 반환되지 않는다.

  ```python
  with Session(engine) as session:
      session.execute(insert(User).values(name="spongebob", fullname="Spongebob Squarepants"))
      session.commit()
      rows = session.scalars(select(User))
      for row in rows:
          print(row)
  ```

  - ORM mapped class도 마찬가지로 column을 지정해서 조회할 수 있다.
    - 이 때, `execute()`를 사용하여 조회할 경우 각 row는 ORM mapped class의 인스턴스가 아니라 각 column 값들의 tuple이 된다.

  ```python
  print(select(User.name, User.fullname))
  """
  SELECT user_account.name, user_account.fullname 
  FROM user_account
  """
  
  with Session(engine) as session:
      session.execute(insert(User).values(name="spongebob", fullname="Spongebob Squarepants"))
      session.commit()
      rows = session.execute(select(User.name, User.fullname))
      for row in rows:
          print(row)		# ('spongebob', 'Spongebob Squarepants')
  ```



- WHERE 절

  - SQLAlchemy는 Python operation을 SQL expression으로 변환해준다.

  ```python
  print(user_table.c.name == "squidward")	# user_account.name = :name_1
  print(address_table.c.user_id > 10)		# address.user_id > :user_id_1
  
  # 아래와 같이 where절을 생성할 때 사용 가능하다.
  print(select(user_table).where(user_table.c.name == "squidward"))
  """
  SELECT user_account.id, user_account.name, user_account.fullname
  FROM user_account
  WHERE user_account.name = :name_1
  """
  ```

  - AND를 통해 여러 조건을 설정하려면 `.where()` 메서드를 여러 번 사용하거나, 하나의 `.where()` 메서드 안에 여러 조건들을 넣으면 된다.

  ```python
  print(
      select(address_table.c.email_address)
      .where(user_table.c.name == "squidward")
      .where(address_table.c.user_id == user_table.c.id)
  )
  
  print(
      select(address_table.c.email_address).where(
          user_table.c.name == "squidward",
          address_table.c.user_id == user_table.c.id,
      )
  )
  
  # 결과는 같다.
  """
  SELECT address.email_address
  FROM address, user_account
  WHERE user_account.name = :name_1 AND address.user_id = user_account.id
  """
  ```

  - `and_()`와 `or_()` 메서드를 사용하여 AND와 OR를 표현할 수 있다.

  ```python
  from sqlalchemy import and_, or_
  print(
      select(Address.email_address).where(
          and_(
              or_(User.name == "squidward", User.name == "sandy"),
              Address.user_id == User.id,
          )
      )
  )
  
  """
  SELECT address.email_address
  FROM address, user_account
  WHERE (user_account.name = :name_1 OR user_account.name = :name_2)
  AND address.user_id = user_account.id
  """
  ```

  - `Select.filter_by()`를 사용해도 AND를 표현할 수 있다.

  ```python
  print(select(User).filter_by(name="spongebob", fullname="Spongebob Squarepants"))
  
  """
  SELECT user_account.id, user_account.name, user_account.fullname
  FROM user_account
  WHERE user_account.name = :name_1 AND user_account.fullname = :fullname_1
  """
  ```



- 명시적인 FROM과 JOIN

  - FROM은 명시적으로 설정하지 않아도 자동으로 설정된다.

  ```py
  print(select(user_table.c.name))
  
  """
  SELECT user_account.name
  FROM user_account
  """
  ```

  - `Select.select_from()`을 사용하면 FROM을 명시적으로 설정하는 것이 가능하다.

  ```python
  from sqlalchemy import func
  print(select(func.count("*")).select_from(user_table))
  
  """
  SELECT count(:count_2) AS count_1
  FROM user_account
  """
  ```

  - `Select.join_from()` 메서드를 사용하여 join이 가능하다.
    - 좌측과 우측을 모두 parameter로 받는다.

  ```python
  print(
      select(user_table.c.name, address_table.c.email_address).join_from(
          user_table, address_table
      )
  )
  
  """
  SELECT user_account.name, address.email_address
  FROM user_account JOIN address ON user_account.id = address.user_id
  """
  ```

  - `Select.join()` 메서드를 사용하는 것도 가능하다.
    - 이 경우 우측만 받으며, 좌측은 자동으로 설정된다.

  ```python
  print(select(user_table.c.name, address_table.c.email_address).join(address_table))
  
  """
  SELECT user_account.name, address.email_address
  FROM user_account JOIN address ON user_account.id = address.user_id
  """
  ```

  - `Select.select_from()` 메서드와 함께 사용하려면 아래와 같이 하면 된다.

  ```python
  print(select(address_table.c.email_address).select_from(user_table).join(address_table))
  
  """
  SELECT address.email_address
  FROM user_account JOIN address ON user_account.id = address.user_id
  """
  ```

  - ON 절 설정하기
    - `Select.join()`와 `Select.join_from()` 메서드는 join 대상 table외에 추가적으로 ON 절에 사용할 표현식을 받는다.

  ```python
  print(
      select(address_table.c.email_address)
      .select_from(user_table)
      .join(address_table, user_table.c.id == address_table.c.user_id)
  )
  
  """
  SELECT address.email_address
  FROM user_account JOIN address ON user_account.id = address.user_id
  """
  ```

  - OUTER와 FULL join
    - `Select.join()`와 `Select.join_from()` 메서드는 `isouter`, `full`이라는 keyword argument를 받는다.
    - 각 값을 True로 주면 각각 OUTER join과 FULL join을 실행한다.

  ```python
  print(select(user_table).join(address_table, isouter=True))
  """
  SELECT user_account.id, user_account.name, user_account.fullname
  FROM user_account LEFT OUTER JOIN address ON user_account.id = address.user_id
  """
  
  print(select(user_table).join(address_table, full=True))
  """
  SELECT user_account.id, user_account.name, user_account.fullname
  FROM user_account FULL OUTER JOIN address ON user_account.id = address.user_id
  """
  ```



- ORDER BY, GROUP BY, HAVING

  - ORDER BY
    - `Select.order_by()`를 통해 설정이 가능하다.
    - 오름차순, 내림차순 설정은 `ColumnElement`의 `.asc()`, `.desc()` 메서드를 사용하면 된다.

  ```python
  print(select(user_table).order_by(user_table.c.name))
  
  """
  SELECT user_account.id, user_account.name, user_account.fullname
  FROM user_account ORDER BY user_account.name
  """
  
  print(select(User).order_by(User.fullname.desc()))
  """
  SELECT user_account.id, user_account.name, user_account.fullname
  FROM user_account ORDER BY user_account.fullname DESC
  """
  ```

  - 집계 함수 사용하기
    - SQLAlchemy에서는 `func`을 통해 SQL의 집계 함수를 사용할 수 있다.
    - `func`은 새로운 `Function` 인스턴스를 생성하는 constructor 객체이다.

  ```python
  from sqlalchemy import func
  count_fn = func.count(user_table.c.id)
  print(count_fn)
  
  """
  count(user_account.id)
  """
  ```

  - GROUP BY와 HAVING 사용하기
    - 각각 `Select`의 `group_by()`, `having()` 메서드를 사용한다.

  ```python
  print(
  	select(User.name, func.count(Address.id).label("count"))
      .join(Address)
      .group_by(User.name)
      .having(func.count(Address.id) > 1)
  )
  
  """
  SELECT "user".name, count(address.id) AS count 
  FROM "user" JOIN address ON "user".id = address.user_id GROUP BY "user".name 
  HAVING count(address.id) > :count_
  """
  ```

  - Label 활용하기
    - Label을 사용하여 정렬과 grouping이 가능하다.

  ```python
  from sqlalchemy import func, desc
  stmt = (
      select(Address.user_id, func.count(Address.id).label("num_addresses"))
      .group_by("user_id")
      .order_by("user_id", desc("num_addresses"))
  )
  print(stmt)
  
  """
  SELECT address.user_id, count(address.id) AS num_addresses
  FROM address GROUP BY address.user_id ORDER BY address.user_id, num_addresses DESC
  """
  ```



- ALIAS 사용하기

  - 아래와 같이 alias를 설정할 수 있다.
    - `.alias()` 메서드는 `Alias` object를 생성하며, `Alias`는 `Table`과 마찬가지로 `Alias.c`를 통해 column들의 collection에 접근할 수 있다.

  ```python
  foo = user_table.alias()
  bar = user_table.alias()
  print(
      select(foo.c.name, bar.c.name).join_from(
          foo, bar, foo.c.id > bar.c.id
      )
  )
  
  """
  SELECT user_account_1.name, user_account_2.name AS name_1
  FROM user_account AS user_account_1
  JOIN user_account AS user_account_2 ON user_account_1.id > user_account_2.id
  """
  ```

  - ORM entitiy는 `aliased()`를 사용하여 alias를 설정할 수 있다.
    - `aliased()` 함수는 `Alias` object를 생성한다.

  ```python
  from sqlalchemy.orm import aliased
  foo = aliased(Address)
  bar = aliased(Address)
  print(
      select(User)
      .join_from(User, foo)
      .where(foo.email_address == "patrick@aol.com")
      .join_from(User, bar)
      .where(bar.email_address == "patrick@gmail.com")
  )
  """
  SELECT user_account.id, user_account.name, user_account.fullname
  FROM user_account
  JOIN address AS address_1 ON user_account.id = address_1.user_id
  JOIN address AS address_2 ON user_account.id = address_2.user_id
  WHERE address_1.email_address = :email_address_1
  AND address_2.email_address = :email_address_2
  """
  ```





## SQL function

- SQL function은 반환 타입이 있다.

  - 각 SQL function은 SQLAlchmey에 정의된 type들을 반환한다.

  ```python
  print(func.now().type)		# DateTime()
  ```

  - 내장 함수에는 이미 정의된 반환 타입이 있다.
    - `count`, `max`, `min` 같은 집계 함수는 적절한 반환 타입이 설정되어 있다.

  ```python
  m1 = func.max(Column("some_int", Integer))
  print(m1.type)		# Integer()
  
  m2 = func.max(Column("some_str", String))
  print(m2.type)		# String()
  ```



- 다양한 SQL function을 사용할 수 있다.

  - `.count()`

  ```python
  print(select(func.count()).select_from(user_table))
  
  """
  SELECT count(*) AS count_1
  FROM user_account
  """
  ```



