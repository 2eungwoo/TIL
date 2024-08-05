# Test Guide

스프링에서 다양한 테스트 전략을 제공하고 있는 만큼 테스트 코드를 통일성 있게 관리하는 것이 중요하다. 더 안전하고 통일성 있게 테스트를 진행하는 방법에 대한 가이드를 따라보자

# 목차

- [Test Guide](#Test_Guide)
- [목차](#목차)
- [테스트 전략](#테스트_전략)
- [통합테스트](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [장점](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [단점](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [Code](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
        - [IntegrationTest](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
        - [Test Code](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
- [서비스 테스트](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [장점](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [단점](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [Code](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
        - [MockTest](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
        - [Test Code](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
- [Mock API 테스트](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [장점](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [단점](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [Code](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
- [Repository Test](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [장점](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [단점](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [Code](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
        - [RepositoryTest](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
        - [Test Code](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
- [POJO 테스트](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [설명](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [장점](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [단점](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
    - [Code](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
        - [Embeddable](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)
        - [Test Code](https://www.notion.so/Test-Guide-8e9adaa99a3f44cd8ea9653fbbc895be?pvs=21)

- [Test Guide](#test-guide)
- [목차](#목차)
- [테스트 전략](#테스트-전략)
- [통합 테스트](#통합-테스트)
    - [장점](#장점)
    - [단점](#단점)
    - [Code](#code)
        - [IntegrationTest](#integrationtest)
        - [Test Code](#test-code)
- [서비스 테스트](#서비스-테스트)
    - [장점](#장점-1)
    - [단점](#단점-1)
    - [Code](#code-1)
        - [MockTest](#mocktest)
        - [Test Code](#test-code-1)
- [Mock API 테스트](#mock-api-테스트)
    - [장점](#장점-2)
    - [단점](#단점-2)
    - [Code](#code-2)
- [Repository Test](#repository-test)
    - [장점](#장점-3)
    - [단점](#단점-3)
    - [Code](#code-3)
        - [RepositoryTest](#repositorytest)
        - [Test Code](#test-code-2)
- [POJO 테스트](#pojo-테스트)
    - [설명](#설명)
    - [장점](#장점-4)
    - [단점](#단점-4)
    - [Code](#code-4)
        - [Embeddable](#embeddable)
        - [Test Code](#test-code-3)
# 테스트 전략

| 어노테이션 | 설명 | 부모 클래스 | Bean |
| --- | --- | --- | --- |
| @SpringBootTest | 통합 테스트, 전체 | IntegrationTest | Bean 전체 |
| @WebMvcTest | 단위 테스트, Mvc 테스트 | MockApiTest | MVC 관련 Bean |
| @DataJpaTest | 단위 테스트, Jpa 테스트 | RepositoryTest | JPA 관련 Bean |
| None | 단위 테스트, Service 테스트 | MockTest | None |
| None | POJO, Domain 테스트 | None | None |

# 통합 테스트

## 장점

- 모든 Bean을 올리고 테스트를 진행하기 때문에 쉽고, 운영환경과 가장 유사하게 테스트 가능
- Api를 테스트할 경우 요청~응답 전체적인 테스트 가능

## 단점

- 모든 Bean을 올리고 테스트하기 때문에 시간이 오래 걸림
- 테스트의 단위가 크기 때문에 테스트 실패 시 디버깅이 어려움
- 외부 Api call 같은 rollback 처리가 안되는 테스트 진행이 어려움

## Code

### IntegrationTest

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = ApiApp.class)
@AutoConfigureMockMvc
@ActiveProfiles(TestProfile.TEST)
@Transactional
@Ignore
public class IntegrationTest {
    @Autowired protected MockMvc mvc;
    @Autowired protected ObjectMapper objectMapper;
    ...
}

```

- 통합 테스트의 Base 클래스. Base 클래스를 통해서 테스트 전략을 통일성 있게 가져갈 수 있다.
- 통합 테스트는 컨트롤러 테스트가 주 이며 요청부터 응답까지의 전체 플로우를 테스트한다.
- `@ActiveProfiles(TestProfile.TEST)` 설정으로 테스트에 profile을 지정. 환경별로 properties 파일을 관리하듯이 test도 반드시 별도의 properties 파일로 관리하는 것이 바람직하다.
- 인터페이스나 enum 클래스를 통해서 profile을 관리한다. 오타 실수를 줄일 수 있으며 전체적인 profile이 몇 개 있는지 한 번에 확인할 수 있다.
- `@Transactional` 트랜잭션 어노테이션을 추가하면 테스트코드의 데이터베이스 정보가 자동으로 rollback 된다. Base 클래스에 이 속성을 추가 해야 실수 없이 진행할 수 있다.
- `@Transactional`을 추가하면 자연스럽게 데이터베이스 상태 의존적인 테스트를 자연스럽게 하지 않을 수 있게 된다.
- 통합 테스트 시 필요한 기능들을 `protected`로 제공해줄 수 있습니다. Api 테스트를 주로 하게 되니 ObjectMapper 등을 제공해줄 수 있다. 유틸성 메서드들도 `protected`로 제공해주면 중복 코드 및 테스트 코드의 편의성이 높아진다.
- 실제로 동작할 필요가 없으니 `@Ignore` 어노테이션을 추가한다.

### Test Code

```java
public class MemberApiTest extends IntegrationTest {

    @Autowired
    private MemberSetup memberSetup;

    @Test
    public void 회원가입_성공() throws Exception {
        //given
        final Member member = MemberBuilder.build();
        final Email email = member.getEmail();
        final Name name = member.getName();
        final SignUpRequest dto = SignUpRequestBuilder.build(email, name);

        //when
        final ResultActions resultActions = requestSignUp(dto);

        //then
        resultActions
                .andExpect(status().isOk())
                .andExpect(jsonPath("email.value").value(email.getValue()))
                .andExpect(jsonPath("email.host").value(email.getHost()))
                .andExpect(jsonPath("email.id").value(email.getId()))
                .andExpect(jsonPath("name.first").value(name.getFirst()))
                .andExpect(jsonPath("name.middle").value(name.getMiddle()))
                .andExpect(jsonPath("name.last").value(name.getLast()))
                .andExpect(jsonPath("name.fullName").value(name.getFullName()))
        ;
    }

    @Test
    public void 회원조회() throws Exception {
        //given
        final Member member = memberSetup.save();

        //when
        final ResultActions resultActions = requestGetMember(member.getId());

        //then
        resultActions
                .andExpect(status().isOk())
                .andExpect(jsonPath("email.value").value(member.getEmail().getValue()))
                .andExpect(jsonPath("email.host").value(member.getEmail().getHost())
                .andExpect(jsonPath("email.id").value(member.getEmail().getId()))
                .andExpect(jsonPath("name.first").value(member.getName().getFirst()))
                .andExpect(jsonPath("name.middle").value(member.getName().getMiddle()))
                .andExpect(jsonPath("name.last").value(member.getName().getLast()))
                .andExpect(jsonPath("name.fullName").value(member.getName().getFullName()))
        ;
    }

    private ResultActions requestSignUp(SignUpRequest dto) throws Exception {
        return mvc.perform(post("/members")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(objectMapper.writeValueAsString(dto)))
                .andDo(print());
    }
    ...
}

```

- `IntegrationTest` 클래스를 상속받는다. 이 상속을 통해서 MemberApiTest에서 테스트를 위한 어노테이션이 생략되며 어떤 통합 테스트라도 항상 통일성을 가질 수 있다.
- `given`, `when`, `then` 키워드로 테스트 흐름을 알려준다. 다른 사람의 테스트 코드의 가독성이 높아지기 때문에 해당 키워드로 적절하게 표시하는 것을 권장되는 방법이다.
- 요청에 대한 메서드를 `requestSignUp(...)`으로 분리해서 재사용성을 높인다. 해당 메서드로 validate 실패에 대한 케이스도 작성한다. `andDo(print())` 메서드를 추가해서 디버깅에 유용하게끔 해당 요청에 대한 결과를 출력한다.
- 모든 response에 대한 `andExpect`를 작성한다.
    - **response에 하나라도 빠지거나 변경되면 API 변경이 이루어진 것이고 그 변경에 맞게 테스트 코드도 변경되어야 한다.**
- `회원 조회` 테스트 같은 경우 `memberSetup.save();` 메서드로 테스트전에 데이터베이스에 insert 한다.
    - 데이터베이스에 미리 있는 값을 검증하는 것은 데이터베이스 상태에 의존한 코드가 되며 누군가가 회원 정보를 변경하게 되면 테스트 코드가 실패하게 된다.
    - 테스트 전에 데이터를 insert하지 않는다면 테스트 코드 구동 전에 `.sql` 으로 미리 데이터베이스를 준비한다. ApplicationRunner를 이용해서 데이터베이스를 준비시키는 방법도 있다.
    - **중요한 것은 데이터베이스 상태에 너무 의존적인 테스트는 향후 로직의 문제가 없더라도 테스트가 실패하는 상황이 자주 만나게 된다.**

# 서비스 테스트

## 장점

- 진행하고자 하는 테스트에만 집중할 수 있으며 테스트 속도가 빠르다.
- 테스트 진행시 중요 관점이 아닌 것들은 Mocking 처리해서 외부 의존성들을 줄일 수 있다.
    - 예를 들어 주문 할인 로직이 제대로 동작하는지에 대한 테스트만 진행할지 이게 실제로 데이터베이스에 insert 되는지는 해당 테스트의 관심사가 아니다.

## 단점

- 의존성 있는 객체를 Mocking 하기 때문에 문제가 완결된 테스트는 아니다.
- Mocking 작업이 많이 귀찮으며 Mocking 라이브러리에 대한 학습 비용이 발생한다.

## Code

### MockTest

```java
@RunWith(MockitoJUnitRunner.class)
@ActiveProfiles(TestProfile.TEST)
@Ignore
public class MockTest {

}

```

- 주로 Service 영역을 테스트 한다.
- `MockitoJUnitRunner`을 통해서 Mock 테스트를 진행한다.

### Test Code

```java
public class MemberSignUpServiceTest extends MockTest {

    @InjectMocks
    private MemberSignUpService memberSignUpService;

    @Mock
    private MemberRepository memberRepository;
    private Member member;

    @Before
    public void setUp() throws Exception {
        member = MemberBuilder.build();
    }

    @Test
    public void 회원가입_성공() {
        //given
        final Email email = member.getEmail();
        final Name name = member.getName();
        final SignUpRequest dto = SignUpRequestBuilder.build(email, name);

        given(memberRepository.existsByEmail(any())).willReturn(false);
        given(memberRepository.save(any())).willReturn(member);

        //when
        final Member signUpMember = memberSignUpService.doSignUp(dto);

        //then
        assertThat(signUpMember).isNotNull();
        assertThat(signUpMember.getEmail().getValue()).isEqualTo(member.getEmail().getValue());
        assertThat(signUpMember.getName().getFullName()).isEqualTo(member.getName().getFullName());
    }

    @Test(expected = EmailDuplicateException.class)
    public void 회원가입_이메일중복_경우() {
        //given
        final Email email = member.getEmail();
        final Name name = member.getName();
        final SignUpRequest dto = SignUpRequestBuilder.build(email, name);

        given(memberRepository.existsByEmail(any())).willReturn(true);

        //when
        memberSignUpService.doSignUp(dto);
    }
}

```

- `MockTest` 객체를 상속받아 테스트의 일관성을 갖는다.
- `회원가입_성공` 테스트는 오직 회원 가입에대한 단위 테스트만 진행한다.
    - `existsByEmail`을 모킹해서 해당 이메일이 중복되지 않았다고 가정한다.
    - `then` 에서는 회원 객체가 해당 비지니스 요구사항에 맞게 생성됐는지를 검사한다.
    - 실제 데이터베이스에 Insert 됐는지 여부는 해당 테스트의 관심사가 아니다.
- `회원가입_이메일중복_경우` 테스트는 회원가입시 이메일이 중복인지 여부를 확인한다.
    - `existsByEmail`을 모킹해서 이메일이 중복됐다고 가정한다.
    - `@Test(expected = ...)`로 이메일이 중복되었을 경우 `EmailDuplicateException` 예외가 발생하는지 확인한다.
    - 해당 이메일이 데이터베이스에 실제로 있어서 예외가 발생하는지는 관심사가 아니다. 작성한 코드가 제대로 동작 하는지(알맞게 예외가 터지는지)가 해당 테스트의 관심사이다.
- 오직 테스트의 관심사만 테스트를 진행하기 때문에 예외 발생시 디버깅 작업도 명확해진다.
- 외부 의존도가 낮기 때문에 테스트 하고자하는 부분만 명확하게 테스트가 가능하다.
    - 이것은 단점이기도 하다. 해당 테스트만 진행하지 외부 의존을 갖는 코드까지 테스트하지 않으니 실제 환경에서 제대로 동작하지 않을 가능성이 있다. 외부 의존에 대한 테스트는 통합 테스트에서 진행한다.

# Mock Api 테스트

## 장점

- Mock 테스트와 장점은 거의 같습니다.
- `WebApplication` 관련된 Bean들만 등록하기 때문에 통합 테스트보다 빠르게 테스트할 수 있다.
- 통합 테스트를 진행하기 어려운 테스트를 진행한다.
    - 주로 외부 Api 같은 rollback 처리가 힘들거나 불가능한 테스트를 한다.
    - 예를 들어 외부 결제 모듈 Api를 콜하면 안 되는 케이스에서 주로 사용 할 수 있다.
    - 이런 문제는 통합 테스트에서 해당 객체를 Mock 객체로 변경해서 테스트를 변경해서 테스트할 수도 있다.

## 단점

- Mock 테스트와 단점은 거의 같다.
- 요청부터 응답까지 모든 테스트를 Mock기반으로 테스트하기 때문에 실제 환경에서는 제대로 동작하지 않을 가능성이 매우 크다.

## Code

```java
@WebMvcTest(MemberApi.class)
public class MemberMockApiTest extends MockApiTest {
    @MockBean private MemberSignUpService memberSignUpService;
    @MockBean private MemberHelperService memberHelperService;
    ...

    @Test
    public void 회원가입_유효하지않은_입력값() throws Exception {
        //given
        final Email email = Email.of("asdasd@d"); // 이메일 형식이 유효하지 않음
        final Name name = Name.builder().build();
        final SignUpRequest dto = SignUpRequestBuilder.build(email, name);
        final Member member = MemberBuilder.build();

        given(memberSignUpService.doSignUp(any())).willReturn(member);

        //when
        final ResultActions resultActions = requestSignUp(dto);

        //then
        resultActions
                .andExpect(status().isBadRequest())
        ;

    }

```

- `@WebMvcTest(MemberApi.class)` 어노테이션을 통해서 하고자 하는 `MemberApi`의 테스트를 진행한다.
- `@MockBean` 으로 객체를 주입받아 Mocking 작업을 진행한다.
- 테스트의 관심사는 오직 Request와 그에 따른 Response 이다.

# Repository Test

## 장점

- `Repository` 관련된 Bean들만 등록하기 때문에 통합 테스트에 비해서 빠르다.
- `Repository`에 대한 관심사만 갖기 때문에 테스트 범위가 작다.

## 단점

- 테스트 범위가 작기 때문에 실제 환경과 차이가 발생한다.

## Code

### RepositoryTest

```java
@RunWith(SpringRunner.class)
@DataJpaTest
@ActiveProfiles(TestProfile.TEST)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Ignore
public class RepositoryTest {
}

```

- `@DataJpaTest` 어노테이션을 통해서 `Repository`에 대한 Bean만 등록한다.
- `@DataJpaTest`는 기본적으로 메모리 데이터베이스에 대한 테스트를 진행한다. `@AutoConfigureTestDatabase` 어노테이션을 통해서 profile에 등록된 데이터베이스 정보로 대체할 수 있다.
- `JpaRepository`에서 기본적으로 기본적으로 제공해주는 `findById`, `findByAll`, `deleteById`등은 테스트를 하지 않는다.
    - 기본적으로 `save()` null 제약 조건 등의 테스트는 진행해도 좋다고 한다.
    - 주로 커스텀하게 작성한 쿼리 메서드, `@Query`으로 작성된 JPQL등의 커스텀하게 추가된 메서드를 테스트한다.

### Test Code

```java
public class MemberRepositoryTest extends RepositoryTest {

    @Autowired
    private MemberRepository memberRepository;

    private Member saveMember;
    private Email email;

    @Before
    public void setUp() throws Exception {
        final String value = "2eungwoo@github.com";
        email = EmailBuilder.build(value);
        final Name name = NameBuilder.build();
        saveMember = memberRepository.save(MemberBuilder.build(email, name));
    }

    ...
    @Test
    public void existsByEmail_존재하는경우_true() {
        final boolean existsByEmail = memberRepository.existsByEmail(email);
        assertThat(existsByEmail).isTrue();
    }

    @Test
    public void existsByEmail_존재하지않은_경우_false() {
        final boolean existsByEmail = memberRepository.existsByEmail(Email.of("ehdgoanfrhkqortntksdls@asd.com"));
        assertThat(existsByEmail).isFalse();
    }
}

```

- `setUp()` 메서드를 통해서 Member를 데이터베이스에 insert 한다.
    - `setUp()` 메서드는 메번 테스트 코드가 실행되기전에 실행된다. 즉, 테스트 코드를 실행할 때마다 insert -> rollback이 자동으로 이루어진니다.
- 추가 작성한 쿼리메서드 `existsByEmail`을 테스트 진행한다.
    - 실제로 작성된 쿼리가 어떻게 출력되는지 `show-sql` 옵션을 통해서 확인 합니다. ORM은 SQL을 직접 작성하지 않으니 실제 쿼리가 어떻게 출력되는지 확인하는 습관을 반드시 가져야한다.

# POJO 테스트

## 설명

각 엔티티(Embeddable, Entity, 일반 POJO, 모든 객체) 객체들의 기능이 풍부해야 다. 객체 본인의 책임을 충분히 다하지 않고 있으면 다른 영역으로 그 객체의 책임이 넘어 가게된다. 예를 들어 `Name` 객체가`getFullName()` 메서드를 제공해주지 않는다면 `getFullName()` 메서드를 만족시키는 메서드들은 다른 계층에서 구현하게 되고 어느 계층에서 어떻게 사용되고 있는지 모르기 때문에 누군가는 중복코드를 만들게 된다.

객체지향에서 본인의 책임(기능)은 본인 스스로가 제공해야 한다. 특히 엔티티 객체들은 가장 핵심적인 객체이고 이 객체를 사용하는 계층들은 다양하게 분포되기 때문에 반드시 테스트 코드를 작성해야한다.

## 장점

- POJO 객체이므로 테스트하기 편하다. 외부에서 주입 받을 의존성도 없고 Mocking할 대상도 없다.
- 엔티티 객체는 사용하는 계층이 많으므로 테스트의 효율성이 높다.

## 단점

- POJO를 테스트 하므로 테스트 속도 및 난도가 낮지만 높은 안전성을 갖게 된다.

## Code

### Embeddable

```java
@Embeddable
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"first", "middle", "last"})
public class Name {

    @NotEmpty
    @Column(name = "first_name", length = 50)
    private String first;

    @Column(name = "middle_name", length = 50)
    private String middle;

    @NotEmpty
    @Column(name = "last_name", length = 50)
    private String last;

    @Builder
    public Name(final String first, final String middle, final String last) {
        this.first = first;
        this.middle = StringUtils.isEmpty(middle) ? null : middle;
        this.last = last;
    }

    public String getFullName() {
        if (this.middle == null) {
            return String.format("%s %s", this.first, this.last);
        }
        return String.format("%s %s %s", this.first, this.middle, this.last);
    }
}

```

- `Name` 객체는 `Member` 객체에서 사용되고 있다. 이처럼 Name 이라는 객체를 `Embeddable`로 별도로 가지고 있으면 데이터의 응집력 재사용성이 높아진다.
    - 예를 들어 주문 시 주문자 정보를 받아야 된다면 `Order` 라는 객체에도 동일하게 `Name` 객체를 사용하면 재사용성이 높아진다.
- `Embeddable` 객체에서도 다른 객체와 마찬가지로 `Name` 관련된 기능을 충분히 제공해야 한다. `getFullName()` 메서드 처럼 `first`, `last`, `middle`의 이름을 적절하게 조합해서 제공해준다.

### Test Code

```java
public class NameTest {

    @Test
    public void getFullName_isFullName_ReturnFullName() {
        final Name name = Name.builder()
                .first("first")
                .middle("middle")
                .last("last")
                .build();
        final String fullName = name.getFullName();
        assertThat(fullName, is("first middle last"));
    }

    @Test
    public void getFullName_WithoutMiddle_ReturnMiddleNameIsNull() {
        final Name name = Name.builder()
                .first("first")
                .middle("")
                .last("last")
                .build();
        final String fullName = name.getFullName();
        assertThat(fullName, is("first last"));
        assertThat(name.getMiddle(), is(nullValue()));
    }

    @Test
    public void getFullName_MiddleNameIsNull_ReturnMiddleNameIsNull() {
        final Name name = Name.builder()
                .first("first")
                .middle("")
                .last("last")
                .build();
        final String fullName = name.getFullName();
        assertThat(fullName, is("first last"));
        assertThat(name.getMiddle(), is(nullValue()));
    }

}

```

- `entity`, `Embeddable` 객체 등의 객체들도 반드시 테스트 코드를 작성해야한다.
- middle값이 비어있을 경우 null로 잘 들어가는지, `getFullName()` 메서드가 잘 동작하는지 테스트한다.

  *reference* : https://github.com/2eungwoo/spring-guide?tab=readme-ov-file
