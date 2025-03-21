## Fetch Join, Paging

JPA를 사용할 때 Pageable 인터페이스를 통해 페이징을 쉽게 구현할 수 있다. 

하지만 JPA N+1 문제에서 해결 방법 중 하나인 fetch join과 paging을 동시에 처리하면 문제가 발생하는데, 알아보자

## 예제(Post, Reply)

- Post : Reply= 1 : N

### PostEntity

```kotlin
@Builder
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Entity
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private String title;
    private String content;

    @OneToMany(mappedBy = "post")
    private List<Reply> replies;

    public void addReply(Reply reply) {
        if (replies == null) {
            replies = new ArrayList<>();
        }
        replies.add(reply);
    }
}
```

### ReplyEntity

```kotlin
@Builder
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Entity
public class Reply {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private String content;

    @ManyToOne
    @JoinColumn(name = "post_id")
    private Post post;
}
```

### PostRepository

```kotlin
public interface PostRepository extends JpaRepository<Post, Long> {

	
		/*
			SELECT DISTINCT p 
			FROM Post p
				JOIN Reply r ON p.id = r.post_id
			WHERE p.content LIKE '%content%'
		*/
		/*
			SELECT COUNT(DISTINCT p) 
			FROM Post p
				JOIN Reply r ON p.id = r.post_id
			WHERE p.content LIKE '%content%'
		*/
    @Query(value = "SELECT DISTINCT p FROM Post p JOIN FETCH p.replies WHERE p.content LIKE %:content%",
            countQuery = "SELECT COUNT(DISTINCT p) FROM Post p INNER JOIN p.replies WHERE p.content LIKE %:content%")
    Page<Post> findByContentWithRepliesFetchJoin(String content, Pageable pageable);
}
```

### TestCode

```sql
    @Test
    public void findByContentWithRepliesFetchJoinTest() {

        Pageable pageable = PageRequest.of(1, 5);
        Page<Post> postPage = postRepository.findByContentLikeFetchJoin("post", pageable);

        List<Post> posts = postPage.getContent();
        Set<Reply> replies = posts.stream()
                .map(post -> post.getReplies())
                .flatMap(repliesStream -> repliesStream.stream())
                .collect(Collectors.toSet());

        assertThat(replies.size()).isEqualTo(50);
    }
```

### 쿼리 결과와 경고 메시지

- 쿼리 결과
    
    ```sql
    select distinct post0_.id         as id1_0_0_,
                    replies1_.id      as id1_1_1_,
                    post0_.content    as content2_0_0_,
                    post0_.title      as title3_0_0_,
                    replies1_.content as content2_1_1_,
                    replies1_.post_id as post_id3_1_1_,
                    replies1_.post_id as post_id3_1_0__,
                    replies1_.id      as id1_1_0__
    from post post0_ inner join reply replies1_ on post0_.id = replies1_.post_id
    where post0_.content like ?
    ```
    
    > limit, offset 이나 rownum 같은 top n 쿼리가 날라가지 않고 데이터가 조회된 모습
    > 
    
- 경고 메시지
    
    ```kotlin
    2025-03-21 08:49:17.440  WARN 39536 --- [           main] o.h.h.internal.ast.QueryTranslatorImpl   : HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
    ```
    
    > 보여지는 내용은 페이징 처리가 된 모습이지만, 실제로는 모든 쿼리 결과를 메모리에 올려놓은 후에 결과를 잘라오게 된다.
    > 
    
    > 이는 OOM(Out Of Memory) 이라는 치명적인 문제를 유발할 수 있다.
    > 

## 해결 방법

### 1.  join 제거, default_batch_fetch_size 설정

`@OneToMany` 로 관계가 연관관계가 맺어져있는 경우에 조인과 페이징을 동시에 처리하기 어렵다.(N+1문제)

따라서 join 을 제거하여 일반적인 페이징 처리로 바꾸고(Pageable ), N+1 문제는 `default_batch_fetch_size` 를 설정하여 해결한다.

### application.yml

```sql
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000
```

### PostRepository

```kotlin
public interface PostRepository extends JpaRepository<Post, Long> {

	
		/*
			SELECT p 
			FROM Post p
			WHERE p.content LIKE '%content%'
		*/
		/*
			SELECT COUNT(p) 
			FROM Post p
			WHERE p.content LIKE '%content%'
		*/
    @Query(value = "SELECT p FROM Post p WHERE p.content LIKE %:content%",
            countQuery = "SELECT COUNT(p) FROM Post p WHERE p.content LIKE %:content%")
    Page<Post> findByContentLike(String content, Pageable pageable);
}
```

### TestCode

```sql
    @Test
    public void findByContentWithRepliesFetchJoinTest() {

        Pageable pageable = PageRequest.of(1, 5);
        Page<Post> postPage = postRepository.findByContentLike("post", pageable);

        List<Post> posts = postPage.getContent();
        Set<Reply> replies = posts.stream()
                .map(post -> post.getReplies())
                .flatMap(repliesStream -> repliesStream.stream())
                .collect(Collectors.toSet());

        assertThat(replies.size()).isEqualTo(50);
    }
```

### 쿼리 결과

```sql
select post0_.id as id1_0_, post0_.content as content2_0_, post0_.title as title3_0_
from post post0_
where post0_.content like ?
limit ? offset ?
```

> limit, offset 으로 페이징 처리가 된다
> 

```sql
select count(post0_.id) as col_0_0_
from post post0_
where post0_.content like ?
```

> count quer
> 

```sql
select replies0_.post_id as post_id3_1_1_,
       replies0_.id      as id1_1_1_,
       replies0_.id      as id1_1_0_,
       replies0_.content as content2_1_0_,
       replies0_.post_id as post_id3_1_0_
from reply replies0_
where replies0_.post_id in (?, ?, ?, ?, ?)
```

> default_batch_fetch_size 설정을 통해 IN 쿼리 수행하여 N+1 해결
>
