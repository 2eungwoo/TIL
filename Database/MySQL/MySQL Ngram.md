## INTRO

MySQL의 Full-Text Search에서 단어를 파싱하여 검색하는 과정에서 필요한 paser 중 하나이다.

이전에 프로젝트에서 사용해본 적이 있는데, 당시 완성에만 너무 급급해서 진득하게 문서를 읽고 정리하지 않았기 때문에 제대로 이해하기 위해 정리해보자

## Ngram

- ngram은 MySQL의 Built-in Parser이다. (5.7이상)
- InnoDB, MyISAM 엔진을 지원하고 중국어(C), 일본어(J), 한국어(K) 를 지원한다(CJK)
- ngram parser는 MySQL 5.7 이상, InnoDB 엔진을 사용할 때 지원되며 8.0 이상에서는 플러그인 방식으로 사용할 수 있다.

## Tokenize

`ngram parser` 는 문자열을 n개의 문자로 구성된 연속된 시퀀스로 `tokenize` 한다.

```sql
string = "abcd"

n=1: 'a','b','c','d'
n=2: 'ab','bc','cd'
n=3: 'abc','bcd'
n=4: 'abcd'
```

한글은 어떻게 할까?

```sql
# if token size = 2 
string = "빵은 커피랑 먹으면 맛있다"

["빵은","커피","피랑","먹으","으면","맛있","있다"]
```

> 토큰화 할 때 띄어쓰기(공백)은 무시된다.
> 
> - 예시
>     
>     ```sql
>     CREATE TABLE IF NOT EXISTS `bread` (
>       `id` INT(11) UNSIGNED NOT NULL AUTO_INCREMENT,
>       `phrase` VARCHAR(100) NOT NULL,
>       `author` VARCHAR(350) NULL DEFAULT NULL,
>       `reg_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
>       PRIMARY KEY (`id`), 
>       FULLTEXT INDEX `ft_idx_phrase` (`phrase`)  WITH PARSER `ngram`
>     );
>     ```
>     
>     > `phrase` 컬럼에 `ngram parser` 를 사용한 `full-text index` 를 걸어줌
>     > 
>     
>     ```sql
>     mysql> INSERT INTO bread(phrase, author) VALUES('빵은 커피랑 먹으면 맛있다','swlee');
>     ```
>     
>     ```sql
>     
>     mysql> INSERT INTO bread(phrase, author) VALUES('빵은 커피랑 먹으면 맛있다','swlee');
>     +----+----------------------------+---------+
>     | id | phrase                     | author  |
>     +----+----------------------------+---------+
>     |  1 | 빵은 커피랑 먹으면 맛있다    | swlee   |
>     +----+----------------------------+---------+
>     1 row in set (0.00 sec)
>     ```
>     
>     ```sql
>     mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_CACHE;
>     +--------+--------------+-------------+-----------+--------+----------+
>     | WORD   | FIRST_DOC_ID | LAST_DOC_ID | DOC_COUNT | DOC_ID | POSITION |
>     +--------+--------------+-------------+-----------+--------+----------+
>     | 맛있   |            1 |           1 |         1 |      1 |        27 |
>     | 먹으   |            1 |           1 |         1 |      1 |       17 |
>     | 빵은   |            1 |           1 |         1 |      1 |        0  |
>     | 으면   |            1 |           1 |         1 |      1 |       22 |
>     | 있다   |            1 |           1 |         1 |      1 |       32 |
>     | 커피   |            1 |           1 |         1 |      1 |        6  |
>     | 피랑   |            1 |           1 |         1 |      1 |       12 |
>     +--------+--------------+-------------+-----------+--------+----------+
>     7 rows in set (0.00 sec) 
>     ```
>     
>     > 해보니까 꽤 재밌다
>     > 

## Token Size

`ngram parser`의 기본 토큰 사이즈는 `2(bigram)`이다.

- 변경 방법
    
    ```sql
    SET GLOBAL ngram_token_size = 1;
    ```
    
    > 이렇게 하는거 일줄 알았는데 아님 이렇게하면 안된다
    > 
    
    방법1. MySQL 설정 파일(my.cnf)에서 설정
    
    ```sql
    [mysqld]
    ngram_token_size=1
    ```
    
    ```sql
    $ sudo systemctl restart mysql
    ```
    
    예를 들어 Docker 컨테이너에서 실행할 때
    
    ```sql
    $ docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:latest --ngram_token_size=1
    ```
    
    방법2. MySQL 실행할 때 옵션 추가(명령어)
    
    ```sql
    $ mysqld --ngram_token_size=1
    ```
    
    일반적으로 `ngram_token_size` 는 검색에 뜨게 하려는 길이로 설정하면 된다.
    
    한 글자만 검색해도 뜨게 하려면 1로 설정하면 된다. 이 때, 토큰 크기가 작을수록 전체 텍스트 검색 영역이 작아지고 검색 속도가 빨라진다고 한다.
    

ngram parser를 사용하는 FULLTEXT 인덱스의 경우 innodb_ft_min_token_size, innodb_ft_max_token_size, ft_min_word_len, ft_max_word_len 의 구성 옵션이 모두 무시되고

토큰의 제어를 위해 `ngram_token_size` 가 사용된다.

### Creating a FULLTEXT Index that Uses the ngram Parser

```sql
mysql> USE test;

mysql> CREATE TABLE articles (
      id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
      title VARCHAR(200),
      body TEXT,
      FULLTEXT (title,body) WITH PARSER ngram
    ) ENGINE=InnoDB CHARACTER SET utf8mb4;

# or if already existing table

CREATE TABLE articles (
      id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
      title VARCHAR(200),
      body TEXT
     ) ENGINE=InnoDB CHARACTER SET utf8mb4;

ALTER TABLE articles ADD FULLTEXT INDEX ft_index (title,body) WITH PARSER ngram;

# or

CREATE FULLTEXT INDEX ft_index ON articles (title,body) WITH PARSER ngram;
```

### Use

```sql
# MATCH() in SELECT list...
SELECT MATCH (a) AGAINST ('abc') FROM t GROUP BY a WITH ROLLUP;
SELECT 1 FROM t GROUP BY a, MATCH (a) AGAINST ('abc') WITH ROLLUP;

# ...in HAVING clause...
SELECT 1 FROM t GROUP BY a WITH ROLLUP HAVING MATCH (a) AGAINST ('abc');

# ...and in ORDER BY clause
SELECT 1 FROM t GROUP BY a WITH ROLLUP ORDER BY MATCH (a) AGAINST ('abc');
```

> Full-Text Search 방법으로 그냥 쿼리 날리면 된다.
> 

## 알아둘 점

### Stopword Handling

`ngram` 에서는 `stopword` 가 처리가 조금 다르다.

일반적으로 tokenized된 것과 완전히 일치하는 단어가 stopwords 테이블에 있다면 그 단어는 full-text search 인덱스에 추가되지 않는다. 

하지만 `ngram parser` 는 토큰화된 단어에 stopwords가 포함되어 있는지 확인하고 포함된 경우에는 토큰을 제외한다.

- 예시
    
    ```sql
    # when ngram_token_size=2,
    string = "ab,cd"
    
    tokens = ["ab","b,",",c","cd"]
    ```
    
    > 만약 `','`가 stopwords로 정의된 경우에 ngram을 안썼으면 완전히 `쉼표` 와 동일한 경우이어야 제외되지만 ngram을 사용하면 “b,’ , “,c” 는 `쉼표`를 포함하므로 검색에서 제외된다.
    > 

### Space Handling

`ngram` 에서 공백은 항상 제외된다.

Stopwords에 공백이 포함되어 있다고 보면 된다.

- 예시
    
    ```sql
    # when ngram_token_size=2,
    string = "ab cd"
    
    tokens = ["ab","cd"]
    ```
    
    > 공백이 포함된 `“b “`, `“ c”` 는 제외된 모습
    > 
    
    ```sql
    #when ngram_token_size=2,
    string = "나 또 빵 먹는다"
    
    tokens = ["먹는","는다"]
    ```
    
    ```sql
    mysql> INSERT INTO books(title, author) VALUES('나 또 빵 먹는다', '이승우');
    
    mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_FT_INDEX_CACHE;
    +--------+--------------+-------------+-----------+--------+----------+
    | WORD   | FIRST_DOC_ID | LAST_DOC_ID | DOC_COUNT | DOC_ID | POSITION |
    +--------+--------------+-------------+-----------+--------+----------+
    | 먹는    |            1 |           1 |         1 |      1 |        7 |
    | 는다    |            1 |           1 |         1 |      1 |       14 |
    +--------+--------------+-------------+-----------+--------+----------+
    ```
    
    > 공백이 포함된 애들은 다 제외됨
    > 

### Term Search

`자연어 모드(Natural Language Mode)` 의 경우 단어들의 `조합(union)` 검색으로 변환되고

`불린 모드(Boolean Mode` 의 경우 `구문(phrase` 검색으로 변환된다.

- 예시
    
    ```sql
    # when ngram_token_size=2,
    string = "빵을 굉장히 잘 먹는 사람"
    
    검색어 : "굉장히"
    -> tokens = ["굉장","장히"]
    ```
    
    > `자연어 모드` : tokens[]에 일치하는 게 있으면 출력
    > 
    
    > `불린 모드` : 정확하게 “굉장히” 가 일치해야 출력 ( +굉장+장히 ) 연산으로 들어감
    > 

### Wildcard Search

`Wildcard(*)` 검색의 접두사 용어가 ngram 토큰 크기보다 짧으면, 쿼리는 접두사 용어로 시작하는 ngram 토큰을 포함하는 모든 인덱스 행을 반환한다.

- 예시
    
    ```sql
    # when ngram_token_size = 2
    
    검색어 : "a*"
    (접두용어:a)
    ```
    
    > `a` 로 시작하는 모든 행 반환
    > 

`Wildcard(*)*` 검색의 접두사 용어가 ngram 토큰 크기보다 길면, 접두사 용어가 ngram 구문으로 변환되고 와일드카드 연산자는 무시된다.

- 예시
    
    ```sql
    # when ngram_token_size=2,
    
    검색어 : "abc*" 
    (접두용어:abc)
    -> token= ["ab,bc"]
    ```
    
    > 와일드카드 연산자 무시되고 그냥 원래 하던대로 바뀜
    > 
    
    ```sql
    mysql> SELECT title FROM books WHERE MATCH(title) AGAINST ('SQ*' IN BOOLEAN MODE);
    +--------+
    | title  |
    +--------+
    | MY SQL |
    | MYSQL  |
    | SQL    |
    +--------+
    
    mysql> SELECT title FROM books WHERE MATCH(title) AGAINST ('SQL*' IN BOOLEAN MODE);
    +--------+
    | title  |
    +--------+
    | MY SQL |
    | MYSQL  |
    +--------+
    ```
    
    > 출처: https://gngsn.tistory.com/163 [ENFJ.dev:티스토리]
    > 

### Phrase Search

구문 검색은 ngram 검색으로 변환된다.

- 예제
    
    ```sql
    # when ngram_token_size=2
    검색어 : "생크림 케이크" 일 때
    "생크림 케이크", "생크", "크림", "케이", "이크" 를 포함하는 결과를 반환,
    "생크림케이크" 는 반환X
    ```
    
    ```sql
    mysql> SELECT keyword FROM example_strs;
    +--------------------------+
    | keyword                  |
    +--------------------------+
    | 스트로베리초코생크림케이크 |
    | 생크 림 케 이크           |
    | 생크림 케이크             |
    | 생크림케이크              |
    +--------------------------+
    ```
    
    ```sql
    mysql> SELECT keyword FROM example_strs 
    WHERE MATCH(keyword) AGAINST ('생크림 케이크' IN BOOLEAN MODE);
    +--------------------------+
    | keyword                  |
    +--------------------------+
    | 스트로베리초코생크림케이크 |
    | 생크 림 케 이크           |
    | 생크림 케이크             |
    +--------------------------+
    
    ```
    
    > `생크림케이크` 는 검색 제외되고 `["생크림 케이크", "생크", "크림", "케이", "이크"]` 중 포함되는 단어가 존재하는 나머지 결과들은 검색 됨
    > 

***reference*** : <br/>
[MYSQL::MySQL 8.4 Reference Manual :: 14.9.8 ngram Full-Text Parser](https://dev.mysql.com/doc/refman/8.4/en/fulltext-search-ngram.html) <br/>
[gngsn MySQL Ngram, 제대로 이해하기](https://gngsn.tistory.com/163) <br/>
[Full Text Search를 이용한 DB 성능 개선 일지](https://www.essential2189.dev/db-performance-fts) <br/>
