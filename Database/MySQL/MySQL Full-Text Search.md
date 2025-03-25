## INTRO

게시글이나 컨텐츠 등 텍스트 데이터를 검색할 때 LIKE 키워드를 사용하여 패턴 일치 검색을 하곤 한다. 하지만 이 경우 인덱스를 통한 검색이 불가능하다.

MySQL에서 Full-Test Search를 사용하면 텍스트 데이터의 검색 성능을 최적화하기 위해 인덱스를 이용한 검색을 할 수 있다. MySQL은 FULLTEXT 인덱스를 제공하며, 이를 이용하면 대량의 텍스트 데이터에 대해 빠르고 효율적으로 검색할 수 있다.

- Full-Text Search는 MySQL 5.6 이상에서 기본적으로 지원이 된다.
- ngram parser는 MySQL 5.7 이상, InnoDB 엔진을 사용할 때 지원되며 8.0 이상에서는 플러그인 방식으로 사용할 수 있다.
- 중국어(C),일본어(J),한글(K)을 지원한다. (CJK)

## Full-Text Search Function

[`MATCH (***col1***,***col2***,...) AGAINST (***expr*** [***search_modifier***])`](https://dev.mysql.com/doc/refman/8.4/en/fulltext-search.html#function_match)

Full-Text 검색은 `MATCH AGAINST` 구문을 통해 가능하다.

```sql
search_modifier:
  {
       IN NATURAL LANGUAGE MODE
     | IN NATURAL LANGUAGE MODE WITH QUERY EXPANSION
     | IN BOOLEAN MODE
     | WITH QUERY EXPANSION
  }
```

> `MATCH`는 검색할 열 지정, (쉼표로 구분 가능)
> 

> `AGAINST` 는 검색할 문자열과 수행할 검색 유형을 나타내는 `search_modifier`를 사용한다.
> 

```sql
SELECT * FROM article WHERE MATCH(title) AGAINST(’develop’ in boolean mode);
```

## FULLTEXT INDEX

`FULLTEXT` 는 MySQL에서 제공하는 full-text index이며 다음과 같이 생성한다.

```sql
# 1. ALTER TABLE
ALTER TABLE articles ADD FULLTEXT INDEX fulltext_index (title,content) WITH PARSER ngram;

# 2. CREATE INDEX
CREATE FULLTEXT INDEX fulltext_index ON articles (title,content) WITH PARSER ngram;
```

> PARSER는 ngram으로 지정한 예제
> 

## Seacrh Types

- **`Natural Language Full-Text Searches`**
- **`Boolean Full-Text Searches`**
- **`Full-Text Searches with Query Expansion`**

## 1. Natural Language Full-Text Searches

`Natural Language` 검색은 검색 문자열을 token(단어의 단위)로 분리한 후, 해당 단어 중 하나라도 포함되는 행을 찾는다.

자연어  검색은 기본 검색 타입이므로 `MATCH .. AGAINST` 구문에 별도의 옵션을 지정하지 않으면 자연어 검색 모드로 검색하게 된다. (명시해도 됨)

```sql
mysql>  SELECT * FROM articles
        WHERE MATCH (title,content)
        AGAINST ('database' IN NATURAL LANGUAGE MODE);
+----+-------------------+------------------------------------------+
| id | title             | content                                  |
+----+-------------------+------------------------------------------+
|  1 | MySQL Tutorial    | DBMS stands for DataBase ...             |
|  5 | MySQL vs YourSQL  | In the following database comparison ... |
+----+-------------------+------------------------------------------+
2 rows in set (0.00 sec)
```

✅ **대소문자 구분 안 함**

Natural Language Full-Text Searches 방식은 기본적으로 대소문자를 구분하지 않는다.

대소문자를 구분하는 전체 텍스트 검색을 수행하려면 `binary collation` 을 사용할 수 있다.

- 예시
    
    `Collation` → 데이터베이스에서 문자열을 정렬하고 비교하는 방법을 정의 
    
    ### 1. **`utf8_bin`을 사용한 테이블 생성**
    
    ```sql
    sql
    
    CREATE TABLE articles_utf8 (
        id INT AUTO_INCREMENT PRIMARY KEY,
        title VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_bin,  -- utf8_bin 사용
        content TEXT CHARACTER SET utf8 COLLATE utf8_bin,        -- utf8_bin 사용
        FULLTEXT(title, content)                                 -- FULLTEXT 인덱스
    );
    
    ```
    
    > - `utf8_bin`을 사용하여 **`UTF-8` 문자셋**을 사용하는 테이블 생성
    - `utf8_bin`을 Collation으로 사용하여 **대소문자 구분** 활성화
    > 
    
    ```sql
    
    SELECT * FROM articles_utf8 
    WHERE MATCH(title, content) 
    AGAINST('MySQL' IN NATURAL LANGUAGE MODE);
    ```
    
    > 이제 `MATCH .. AGAINST`를 사용하여 대소문자를 구분하는 검색을 수행할 수 있다.
    > 

✅ 무시되는 검색어

길이가 너무 짧거나`(short words)`  특정 단어`(stop words)`는 full-text 검색에서 무시된다.

- 예시
    
    ### 1. 너무 짧은 단어(Short Words)
    
    단어의 기본 길이로 지정된 길이보다 짧을 경우 무시된다.
    
    기본 길이는 다음과 같이 확인할 수 있다.
    
    ```sql
    mysql> show variables like '%ft_min%';
    +--------------------------+-------+
    | Variable_name            | Value |
    +--------------------------+-------+
    | ft_min_word_len          | 5     |
    | innodb_ft_min_token_size | 4     |
    +--------------------------+-------+
    2 rows in set (0.02 sec)
    
    ```
    
    > `ft_min_word_len` 의 기본 값으로 5가 설정되어 있다. 따라서 5자 이하의 단어는 검색에서 제외된다.
    > 
    > - 기본 값 변경 방법
    >     
    >     ```sql
    >     SET GLOBAL ft_min_word_len = 1;
    >     SET GLOBAL innodb_ft_min_token_size = 1;
    >     ```
    >     
    >     > 한글 기준으로 한 글자 단위로도 검색 가능하게 1로 수정하는게 보통임
    >     > 
    >     
    >     ```sql
    >     $ sudo systemctl restart mysql
    >     ```
    >     
    
    ### 1. 특정 단어(Stop Words)
    
    `Stop Word`는 MySQL에 내장되어 있는 의미가 없는 단어들이다. (on, this, is, a 등)
    
    다음과 같이 확인할 수 있다.
    
    ```sql
    mysql> select * from INFORMATION_SCHEMA.INNODB_FT_DEFAULT_STOPWORD;
    +-------+
    | value |
    +-------+
    | on    |
    | this  |
    | is    |
    | a     |
    | for   |
    | an    |
    | of    |
    | at    |
    | and   |
    +-------+
    ```
    
    > 영어가 아닌 언어는 MySQL에 내장된 stop word가 없다
    > 

✅ 매치율

검색 결과는 가장 높은 관련성을 가진 결과부터 자동 정렬된다.

입력된 검색어의 키워드가 얼마나 더 많이 포함되어 있는지에 따라 매치율이 결정되는데 전체 테이블의 50% 이상의 레코드가 검색된 키워드를 가지고 있다면 그 키워드는 검색어로서 의미가 없다고 판단되어 검색 결과에서 제외된다.

## 2. Boolean Full-Text Searches

`Boolean Searches` 는 문자열을 단어 단위로 분리한 후, 추가적인 검색 규칙을 적용하여 단어가 포함되는 행을 찾는다.

`IN BOOLEAN MODE` 를 지정해서 사용한다.

```sql
mysql> SELECT * FROM articles WHERE MATCH (title,content)
    AGAINST ('+MySQL -YourSQL' IN BOOLEAN MODE);
```

> “MySQL” 은 추가하고 “YourSQL”은 제외하라는 검색 규칙을 적용
> 
- 참고
    
    ```
    InnoDB 엔진을 사용하면 MATCH() 식의 모든 열에 FULLTEXT 인덱스가 필요하지만,MyISAM 검색 인덱스에 대한 부울 쿼리는 FULLTEXT 인덱스가 없어도 작동할 수 있습니다.하지만 실행되는 검색은 상당히 느립니다.
    ```
    
    > 출처:https://gngsn.tistory.com/162
    > 
    
    | Operator | Description |
    | --- | --- |
    | + | AND, 반드시 포함하는 단어 |
    | - | NOT, 반드시 제외하는 단어 |
    | > | 포함하며, 검색 순위를 높일 단어+mysql >tutorial: mysql과 tutorial가 포함하는 행을 찾을 때, tutorial이 포함되면 검색 랭킹이 높아짐
     |
    | < | 포함하되,검색 순위를 낮출 단어+mysql <training: mysql과 training가 포함하는 행을 찾지만, training이 포함되면 검색 랭킹이 낮아짐 |
    | 0 | 
    하위 표현식으로 그룹화 (포함, 제외, 순위 지정 등)+mysql +(>tutorial <training): mysql AND tutorial, mysql AND training 이지만, tutorial의 우선순위가 더욱 높게 지정
     |
    | ~ | Negate. '-' 연산자와 비슷하지만 제외 시키지는 않고 검색 조건을 낮춤 |
    | * | Wildcard. 와일드카드my*: mysql, mybatis 등 my 뒤의 와일드 카드로 붙음 |
    | ** | 구문 정의 |

## 3. Full-Text Searches with Query Expansion

`Query Expansion` 은 `Natural Full-Text Searches` 를 확장한 내용으로, 사용자가 어떤 단어를 검색했을 때, 그 단어와 연관이 있는 텍스트들이 같이 검색된다. 

Natural Search 결과에 매칭된 행을 기반으로 검색 문자열을 재구성하여 검색을 수행하는 두 단계로 이루어진다.

 `IN NATURAL LANGUAGE MODE WITH QUERY EXPANSION` 또는 `WITH QUERY EXPANSION` 을 지정해서 사용한다.

```sql
mysql> SELECT * FROM articles
    WHERE MATCH (title,content)
    AGAINST ('database' WITH QUERY EXPANSION);
+----+-----------------------+------------------------------------------+
| id | title                 | content                                  |
+----+-----------------------+------------------------------------------+
|  5 | MySQL vs. YourSQL     | In the following database comparison ... |
|  1 | MySQL Tutorial        | DBMS stands for DataBase ...             |
|  3 | Optimizing MySQL      | In this tutorial we show ...             |
|  6 | MySQL Security        | When configured properly, MySQL ...      |
|  2 | How To Use MySQL Well | After you went through a ...             |
|  4 | 1001 MySQL Tricks     | 1. Never run mysqld as root. 2. ...      |
+----+-----------------------+------------------------------------------+
6 rows in set (0.00 sec)
```

`Query Expansion` 은 일반적으로 검색 구문이 아주 짧을 때 유용하다. 사용자가 입력한 검색어에 대해 연관된 단어를 자동으로 찾아 확장하여 검색을 수행하는데, 개발자가 별도로 관련 단어들을 설정하지 않아도 MySQL DBMS가 자동으로 처리해준다.

예를 들어, 사용자가 `computer`를 검색하면 MySQL은 내부적으로 `computer`와 관련된 다른 단어들( `laptop`, `desktop`, `PC`)을 찾아 검색에 포함시켜 보다 넓은 범위의 검색 결과를 반환한다.

***reference*** : 

https://dev.mysql.com/doc/refman/8.4/en/fulltext-search.html

https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-%ED%92%80%ED%85%8D%EC%8A%A4%ED%8A%B8-%EC%9D%B8%EB%8D%B1%EC%8A%A4Full-Text-Index-%EC%82%AC%EC%9A%A9%EB%B2%95

https://www.essential2189.dev/db-performance-fts
