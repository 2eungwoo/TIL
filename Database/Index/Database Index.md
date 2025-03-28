## INTRO

인덱스에 대해서 여러 글도 읽어 보고 직접 다뤄보기도 하면서 꽤 많이 접해봤지만, 따로 정리해놓은 곳이 없어서 그 필요성을 느끼고 기록하게 되었다.

데이터베이스의 주요 개념 중 하나인 인덱스에 대해 알아보자

## INDEX

 `인덱스` 는 테이블에서 인덱스 데이터 구조를 유지하는 데 필요한 추가적인 쓰기 작업과 저장 공간을 대가로 검색 속도를 향상시켜주는 데이터 구조이다.

특정 컬럼에 인덱스를 생성하면 해당 컬럼의 데이터들을 정렬하여 별도의 메모리 공간에 데이터의 물리적 주소와 함께 저장한다.

이 후 조건절에 인덱스를 사용할 경우 옵티마이저에서 판단 하에 검색을 진행하는데, 인덱스를 사용한 검색을 하게 될 경우 인덱스에 저장되어 있는 데이터의 물리 주소로 이동해 데이터를 가져오는 방식으로 검색 속도가 향상되는 것이다.

- 빠른 검색
    
    대규모의 데이터베이스에서 검색 시간이 선형 시간으로 가면 비효율적이다.
    
    특정 데이터를 찾기 위해 모든 레코드를 조건에 일치하는 행이 나타날 때까지 조회하려면 O(N)의 시간을 소모하지만, 인덱스를 통해 O(logN) 이하의 검색 성능을 기대할 수 있다.
    

- 제약 조건 통제
    
    인덱스는 UNIQUE, PRIMARY KEY, FOREIGN KEY와 같은 제약 조건을 감시하는 데에도 쓰인다.
    
    인덱스 생성은 기본 테이블에 암묵적으로 제약 조건을 생성하게 하며 인덱스는 UNIQUE 제약 조건으로도 선언 가능하다.
    
    즉, 제약 조건(UNIQUE, PRIMARY KEY, FOREIGN KEY 등)을 지키도록 보장하는 역할을 하며 인덱스가 이러한 제약을 제대로 강제하고 있는지 확인하는 작업을 포함한다는 뜻이다.
    

## INDEX 구조

## B-Tree

일반적으로 인덱스는 `B-Tree` 구조를 갖는다.(Balanced Tree)

이는 정렬을 통한 구조로 검색이 항상 동일 시간을 갖는다는 특징이 있다.

`B-Tree` 는 범위를 기반으로 나뉘어진 정렬된 리스트이다.

![image](https://github.com/user-attachments/assets/205d9cae-9efd-43fb-985a-39bb4f838c8a)

- `Branch Blocks` : 검색을 위한 블록
- `Leaf Blocks` : 값을 저장하는 블록

### Branch Blocks

상위 블록에는 하위 블록을 가리키는 인덱스 데이터를 포함한다.

그림의 블록들을 잘 보면 범위를 가리키는 인덱스 데이터를 포함하고 있다.

root 블록에는 하위 블록들의 인덱스 데이터가, 하위 블록들은 leaf block으로의 인덱스 데이터를 갖고 있는 모습이다.

### Leaf Blocks

모든 인덱스의 데이터 값과 실제 행을 찾을 때 사용되는 `ROWID` 를 포함한다.

그림을 잘 보면 `(key,rowid)` 의 형태로 블록에 저장되어 있다.

그래서 인덱싱된 값만 접근하려는 경우(인덱스를 사용하여 특정 값을 찾을 때) 인덱스 블록에만 접근하여 빠르게 값을 찾아낼 수 있다.

- Ex) 테이블에 `이름` 필드가 있고 인덱스를 사용하여 `홍길동` 을 찾는 경우 인덱스에서 바로 찾아냄
    
    

만약 인덱스에서 찾은 값만으로 실제 행 데이터를 찾을 수 없다면 `rowid` 를 사용하여 실제 데이터(행)을 찾아야 한다.

`rowid` 는 데이터베이스 테이블에서 각 행을 고유하게 식별하는 ID이다.(PK랑 다른거임)

- Ex) `홍길동` 이라는 이름만으로는 다른 정보(나이, 주소, 성별 등)를 알 수 없으므로 `홍길동`의 `rowid` 를 사용해 해당 행을 테이블에서 찾아야 한다.
- 여기서 생각해보면,
    - 단순히 인덱스를 통해 검색할 때, 인덱스 블록에만 접근한다 → 실제 데이터는 필요 없다.
    - 실제 데이터를 가져온다 → `rowid` 를 이용해 테이블 행을 찾는다.
    
    라는 말이 된다.
    

그림을 자세히 보면 `Leaf Block` 끼리 화살표가 이어져있는데 이는 수평적 탐색을 위한 장치로, 조건절을 만족하는 데이터를 모두 찾거나 rowid를 얻기 위해서 사용된다고 한다.

## INDEX 종류

인덱스는 자동으로 생성되기도 하고 직접 설정도 가능하다.

자동으로 생성되는 경우에는

- PK 설정 : `Clustered Index`
- UNIQUE 설정 : `Non-Clustered Index`

가 자동 생성된다.

### Clustered Index

`Clustered Index` 는 물리적으로 데이터들을 정렬한다.

데이터 삽입 순서와 상관없이 인덱스의 행을 기준으로 정렬되며 이 정렬 기준은 인덱스와 같기 때문에 테이블 당 하나의 `Clustered Index` 를 포함할 수 있다.

레코드를 삽입,수정할 때마다 순서 유지가 보장되기 때문에 조회 성능에 아주 도움이 되지만 바꿔 말하면 삽입,수정이 일어날 때마다 부담이 된다는 것이다.

`Clustered Index` 는 데이터를 `Leaf Blocks` 가 직접 가지고 있다. 

인덱스 페이지를 key와 데이터 페이지의 번호로 구성하고, 검색하고자 하는 데이터의 key로 페이지 번호를 알아내어 데이터를 찾는다.

### Non-Clustered Index

`Non-Clustered Index` 는 물리적으로 정렬되지 않은 상태로 데이터가 저장된다.

그래서 `Clustered Index` 에 비해 조회는 느리지만 삽입,수정에는 더 우수하다고 볼 수 있다.

`Non-Clustered Index` 는 데이터를 직접 가지고 있지 않고 데이터의 위치를 가리킨다.

이 때 두 가지 경우가 있는데,

- `Clustered Index` 를 가리킴
- `실제 데이터` 를 저장한 위치를 가리킴

## INDEX 사용 쿼리

`인덱스` 를 사용하기에 앞서 조회 성능 향상을 목적으로 무작정 생성하는 것은 바람직하지 않다.

> [인덱스 효과적으로 설정하기](https://www.notion.so/1bf1f679d4cb80349c40e42abb8ecbc4?pvs=21)
> 

인덱스를 설정해준 컬럼에 대해 효과적인 쿼리를 작성해볼 수 있을까?

### Covering Index Query : `Clustered Index`

`Clustered Index`는 테이블 데이터를 물리적으로 정렬하여 저장한다.

`Clustered Index`를 활용한 커버링 인덱스는 주로 PK를 사용하여 데이터를 효율적으로 조회한다.

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,       -- Clustered Index
    name VARCHAR(100),
    age INT,
);
```

```sql
SELECT id, name FROM employees WHERE id = 5;
```

> `Clustered Index` 를 활용한 쿼리
> 

여기서 `id` 는 `Clustered Index` 의 key 값으로, `id` 와 `name` 만 인덱스로 조회할 수 있어 커버링 인덱스 쿼리가 된다. 

인덱스 내에서 데이터가 바로 제공되므로 테이블을 추가로 조회하지 않는다.

### Covering Index Query : `Non-Clustered Index`

`Non-Clustered Index`는 인덱스가 별도로 존재하고, 필요한 컬럼들을 인덱스에 포함시켜 커버링 인덱스를 활용할 수 있다.

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),       -- Non-Clustered Index
    age INT,
);

-- name 컬럼에 인덱스 생성
CREATE INDEX idx_name ON employees(name);
```

```sql
SELECT name, age FROM employees WHERE name = 'Hong GilDong';
```

> `Non-Clustered Index` 를 활용한 쿼리
> 

이 쿼리는 `name` 컬럼에 대한 `Non-Clustered Index` 를 사용하여 `name` , `age` 를 모두 인덱스에서 가져올 수 있기 때문에 테이블을 조회할 필요 없이 커버링 인덱스가 적용된다.

*reference*
https://en.wikipedia.org/wiki/Database_index#Bitmap_index<br/>
https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html<br/>
https://docs.oracle.com/cd/E11882_01/server.112/e40540/indexiot.htm#CNCPT1170<br/>
https://choicode.tistory.com/27<br/>
https://gngsn.tistory.com/88?category=851218
