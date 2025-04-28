*repository* : [TDD-Practice](https://github.com/2eungwoo/TDD-practice)

## Layer 구성

클라이언트의 요청이 Controller → Service → Repository 순으로 진행된다.

각각 계층 단위로 Member를 생성, 조회하는 간단 로직을 테스트해보려고 한다. 테스트코드를 작성해본다.

## Repository Test

***MemberRepository.class***

```java
public interface MemberRepository extends JpaRepository<Member,Long> {}
```

***MemberRepositoryTest.class***

```java
package study.tddpractice.Repository;

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class MemberRepositoryTest {

    @Autowired
    private MemberRepository memberRepository;

    @AfterEach

    @Test
    @DisplayName(" 멤버 저장 ")
    void saveMember(){
        // given
        Member newMember = Member.builder()
                .memberName("test new member")
                .build();

        // when
        Member testSaveMember = memberRepository.save(newMember);

        // then
        assertThat("test new member").isEqualTo(testSaveMember.getMemberName());
    }

    @Test
    @DisplayName(" 멤버 조회 by ID ")
    void findMemberById(){
        // given
        Member newMember = Member.builder()
                .memberName("test member")
                .build();

        Member savedMember = memberRepository.save(newMember);

        // when
        Member foundMember = memberRepository.findById(savedMember.getId()).get();

        // then
        assertThat(foundMember)
                .isNotNull()
                .isEqualTo(savedMember);

        assertThat(savedMember.getId()).isEqualTo(foundMember.getId());
        assertThat(savedMember.getMemberName()).isEqualTo(foundMember.getMemberName());
    }
}
```

테스트 코드를 보면 `given` `when` `then` 패턴으로 작성되어있다.

각 단계가 의미하는 바는 다음과 같다.

`given` : 테스트 시나리오에 필요한 값 설정

`when` : 테스트 행동과 목적 명시

`then` : 코드를 통해 도출된 결과를 검증

 `@DataJpaTest` 를 사용하면 JPA 관련 테스트 설정만 로드하며, 실제 데이터베이스를 사용하지 않고 테스트를 진행할 수 있다. 

또한 `@Transactional` 이 설정되어 있어 `@DataJpaTest` 로 실행한 테스트는 `@Transactional` 이 자동으로 설정된 상태로 실행된다. 

아래와 같은 JPA 영속성 컨텍스트의 쿼리 실행 방식에 의해 테스트 코드에서 실제 쿼리가 나가지 않게 할 수 있다.

### @DataJpaTest

**JPA 영속성 컨텍스트의 쿼리 실행 방식**

- 영속성 컨텍스트는 `flush()`가 실행되기 전까지 실행될 쿼리를 가지고 있다가 한번에 쿼리를 실행하도록 설계되어있습니다.
- `flush()`가 실행되기 전까지 쿼리가 실행되지 않는다는 것은 `@Transactional`안에서 Rollback이 되어야 하는 상황이라면 애초에 쿼리가 실행조차 되지 않을수 있다는 것입니다.

### **JUnit에서의 @Transactional**

- JUnit을 통한 Test가 실행된다면 `@Transactional`이 적용되어있는 로직이 실행된다면 SELECT를 제외한 모든 쿼리는 Rollback 대상으로 취급합니다.

*reference :* https://cobbybb.tistory.com/23

## Service Test

***MemberServiceTest.class***

```java
package study.tddpractice.service;

@ExtendWith(MockitoExtension.class)
class MemberServiceTest {

    @InjectMocks
    private MemberService memberService;

    @Mock
    private MemberRepository memberRepository;

    @Test
    @DisplayName("회원 생성")
    void saveMember() {
        // given
        Member newMember = Member.builder()
                .id(333L)
                .memberName("Tester")
                .build();

        MemberDto.Request request = MemberDto.Request.builder()
                .memberName("Tester")
                .build();

        given(memberRepository.save(any(Member.class))).willReturn(newMember); // mocking

        // when
        MemberDto.Response savedMember = memberService.saveMember(request);

        // then
        assertThat(savedMember.getMemberName()).isEqualTo("Tester");
    }

    @Test
    @DisplayName("회원 조회 by id")
    void findMember() {
        // given
        Long fakeMemberId = 333L;
        Member newMember = Member.builder()
                .id(fakeMemberId)
                .memberName("Tester")
                .build();

        given(memberRepository.findById(fakeMemberId)).willReturn(Optional.ofNullable(newMember)); // mocking

        // when
        MemberDto.Response foundMember = memberService.findMemberById(fakeMemberId);

        // then
        assert newMember != null;
        assertThat(newMember.getId()).isEqualTo(foundMember.getMemberId());
        assertThat(newMember.getMemberName()).isEqualTo(foundMember.getMemberName());

    }
}
```

service계층 test에 사용된 Mock 개념

`Mock` : 가짜 객체를 만들어 사용하는 방법

실제 객체를 사용하기에 시간, 비용이 높거나 객체 간 의존성이 강해 구현이 곤란할 경우 가짜 객체를 만들어 사용한다. 이러한 가짜(Mock) 객체를 만들어 사용할 수 있도록 `Mockito` 라는 테스트 프레임워크가 지원한다.

RepositoryTest와 비슷한 흐름으로 테스트 코드가 작성되어 있으며 ServiceTest에서는 **`given().willReturn();`** 코드가 새롭게 추가되었다. 이것이 Mock 개념을 사용한 코드인데, 그중에서도 앞서 살펴본 Stub의 개념이 적용되어 있는 코드이다.

memberRepository는 `@Mock` 어노테이션이 붙은 걸로 보았을 때, 내부 구현이 없는 상태라는 것을 유추할 수 있다. `memberRepository.findById(fakeMemberI)` 가 호출이 되면, member를 반환하는 행동을 미리 지정해둔 것이고, `given().willReturn(newMember);` 로 사용되는 모습을 볼 수 있다.

- **`@ExtendWith` :** 단위 테스트에 공통적으로 사용할 확장 기능을 선언해 주는 역할
    
    - `SpringExtension.class` 또는 `MockitoExtension.class`를 많이 사용한다. 
    
    - SpringExtension을 이용하게되면 Spring TestContext Framework와 Junit5와 통합하여 사용하게 되고, MockitoExtension을 이용하게되면 Mokito와 관련된 MockContext기반에서 조금은 가볍게 진행이가능하다.
- **`@Mock` : 객체를 생성한다. 실제로 메서드는 갖고 있지만 내부 구현이 없는 상태이다.
- **`@Spy` :** 모든 기능을 가지고 있는 완전한 객체이다. Stub 하지 않은 메서드들은 원본 메서드 그대로 사용한다. 즉, 테스트 대상의 일부분만 Mocking 하는 것이다. 대체로 @Spy보다는 @Mock을 쓰는 것을 추천하지만, 외부 라이브러리를 이용한 테스트에는 @Spy를 사용하는 것을 추천한다.
- **`@InjectMocks` :** @Mock 또는 @Spy로 생성된 가짜 객체를 자동으로 주입시켜 주는 객체. @InjectMocks 객체에서 사용할 객체를 @Mock으로 만들어 쓰면 된다. 만약 Service를 테스트하는 클래스를 생성했다면, Service 객체를 @InjectMocks 어노테이션을 사용해 생성하고, Service단에서 사용할 Repository와 같은 객체들은 @Mock 어노테이션을 사용해 생성하면 된다.
    
    ```java
        class MemberServiceTest {
        
    		    @InjectMocks
    		    private MemberService memberService;
    
    		    @Mock
    		    private MemberRepository memberRepository;
        }
    ```
    
- **`Stub` :** 다른 객체 대신에 가짜 객체(Mock Object)를 주입하여 어떤 결과를 반환하라고 정해진 답변을 준비시킨다. Mock 객체의 메소드를 호출해도 실제로 코드를 실행하지 않기 때문에, 메소드의 행동을 미리 정해두어야 한다. `when()`, `thenReturn()`, `thenThrow()` 등을 사용해 리턴 값 또는 예외 발생을 정할 수 있다. 같은 조건으로 다시 stub 할 경우 이전의 행동을 덮어 씌운다

## Controller Test

***MemberControllerTest.class***

```java
package study.tddpractice.controller;

@WebMvcTest(MemberController.class)
@MockBean(JpaMetamodelMappingContext.class)
class MemberControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    ObjectMapper objectMapper;

    @MockBean
    private MemberService memberService;

    @Test
    @DisplayName(" 회원 저장 ")
    void saveMember() throws Exception {
        // given
        MemberDto.Request memberJoinRequest = MemberDto.Request.builder()
                .memberId(100L)
                .memberName("TTT")
                .build();

        Member mem = memberJoinRequest.toEntity();
        given(memberService.saveMember(/*memberJoinRequest*/any())).willReturn(new MemberDto.Response(mem));

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY);
        String memberJson = objectMapper.writeValueAsString(memberJoinRequest);

        // when
        final ResultActions result = mockMvc.perform(
                MockMvcRequestBuilders.post("/join")
                        .content(memberJson)
                        .accept(MediaType.APPLICATION_JSON)
                        .contentType(MediaType.APPLICATION_JSON)
        );

        // then
        result
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.data.memberId").value(100L))
                .andExpect(jsonPath("$.data.memberName").value("TTT"))
                .andDo(print());

    }

    @Test
    @DisplayName(" 회원 조회 by ID ")
    void findMemberById() throws Exception {
        // given
        Long mockMemberId = 3L;
        Member mem = Member.builder()
                .id(mockMemberId)
                .memberName("TE")
                .build();

        given(memberService.findMemberById(mockMemberId)).willReturn(new MemberDto.Response(mem));

        // when
        Long memberId = 3L;
        final ResultActions result = mockMvc.perform(
                MockMvcRequestBuilders.get("/members/{memberId}", memberId)
                        .accept(MediaType.APPLICATION_JSON)
                        .contentType(MediaType.APPLICATION_JSON)
        );

        // then
        result
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.data.memberId").value(3))
                .andExpect(jsonPath("$.data.memberName").value("TE"))
                .andDo(print());

    }

}
```

Controller를 테스트하기 위해서는 HTTP 호출이 필요하다. 일반적인 방법으로는 HTTP 호출이 불가능하므로 스프링에서는 이를 위한 MockMvc를 제공하고 있다.

*reference* :

[[jackson, issue] 이슈 해결 - No serializer found for class](https://steady-hello.tistory.com/90)

[[Spring] Mock 객체의 리턴값이 왜 null이 나올까?](https://dd-developer.tistory.com/108)
