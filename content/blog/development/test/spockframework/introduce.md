---
title: Spock Framework 소개
date: 2022-03-29 15:03:73
category: test
draft: false
---
![spock logo](./resources/spock-main-logo.png)
# 소개

* BDD(Behaviour-Driven Development) 스타일의 테스트 및 명세 프레임워크
    * BDD 와 TDD 논란(차이점 모호함)
    * TDD 보다 비지니스 요구사항에 집중
* Groovy 로 작성
* Java 로 작성되는 테스트 코드 대비 좋은 가독성

# 특징

## 메소드 명

Groovy 의 특징으로 메소드 명을 문자열로 사용할 수 있습니다.
그로인해 테스트의 이름에 스페이스를 사용할 수 있습니다.

### JUnit

``` java
    @Test
    public void 추천된_목록_가져오기() {
        //...
    }
```

### Spock Framework

``` groovy
    def "추천된 목록 가져오기"() {
        //...
    }
```

## 블록

### 종류

| Block | 의미 |
| --- | --- |
| given: | 테스트 준비 단계 |
| when: | 테스트 실행 단계 |
| then: | 테스트 검증 단계 |
| expect: | when: + then: 을 한번에 처리 |
| cleanup: | 테스트 완료후 후처리(파일시스템 정리 등등의 목적) |
| where: | 데이터 기반의 테스트 시 데이터를 정의 |

### given:

테스트 준비 단계

``` groovy
def "테스트 예제"() {
    given:
    def a = 1
    def b = 2
}
```

### when:

테스트 실행 단계

``` groovy
def "테스트 예제"() {
    given:
    def a = 1
    def b = 2

    when:
    def c = a + b
}
```

### then:

테스트 검증 단계

``` groovy
def "테스트 예제"() {
    given:
    def a = 1
    def b = 2

    when:
    def c = a + b

    then:
    c == 3
    c > 2
    c < 4
}
```

### expect:

when: + then: 을 한번에 처리

``` groovy
def "테스트 예제"() {
    given:
    def a = 1
    def b = 2

    expect:
    a + b == 3
}
```

### cleanup:

테스트 완료후 후처리(파일시스템 정리 등등의 목적)

``` groovy
def "테스트 예제"() {
    given:
    def file = new File("/some/path")
    file.createNewFile()

    when:
    // ...

    then:
    // ...

    cleanup:
    file.delete()
}
```

### where:

데이터 기반의 테스트 시 데이터를 정의

``` groovy
def "데이터 기반 테스트 예제"(def a, def b, def c) {
    expect:
    a + b == c

    where:
    a | b | c
    1 | 2 | 3
    2 | 3 | 5
    3 | 4 | 7
}
```

## 예제

### 예외

* 예외 발생

``` groovy
def "예외가 발생하는 테스트 예제"() {
    given:
    def stack = new Stack()

    when:
    stack.pop()

    then:
    thrown(EmptyStackException)
    stack.empty
}
```

* 예외 발생 안함

``` groovy
def "예외가 발생하지 않는 테스트 예제"() {
    given:
    def map = new HashMap()

    when:
    map.put(null, "elem")

    then:
    notThrown(NullPointerException)
    map.get(null) == "elem"
}
```

### 인터랙션

Mock을 생성하여 호출 횟수를 테스트

``` groovy
interface TestService {

    void callTwice()

    void callOnce()

    void callNever()
}

def "호출 회수 테스트 예제"() {
    given:
    def service = Mock(TestService)

    when:
    service.callOnce()
    service.callTwice()
    service.callTwice()

    then:
    0 * service.callNever()
    1 * service.callOnce()
    2 * service.callTwice()
}
```

### 데이터 테이블

where 에 정의한 행 만큼 반복적으로 실행

``` groovy
def "데이터 기반 테스트 예제"(def a, def b, def c) {
    expect:
    a + b == c

    where:
    a | b | c
    1 | 2 | 3
    2 | 3 | 5
    3 | 4 | 7
}
```

# Spring 과 함께

일반적인 JUnit 사용과 동일하게 클래스 선언부에 `@SpringBootTest` 어노테이션을 지정하여 사용하며
종속성 주입 또한 동일하게 `@Autowired` 와 같은 어노테이션을 사용합니다.

``` groovy
@SpringBootTest
class WorkServiceSpec extends Specification {
    //...
    @Autowired
    WorkService workService
}
```

그외에 Mock 을 만들어서 빈으로 등록하는 부분은 아래의 문서를 참조 바랍니다.
[Spock Framework - Spring Module](https://spockframework.org/spock/docs/2.0/modules.html#_spring_module)

# 설정

## Dependency 설정

build.gradle

``` groovy
dependencies {
    ...
    // spock framework 기본 종속성
    testImplementation 'org.spockframework:spock-core:2.0-groovy-3.0'
    // spock framework spring module 종속성 (버전을 동일하게 사용해야함)
    testImplementation 'org.spockframework:spock-spring:2.0-groovy-3.0'
    // groovy 종속성 버전 지정을 위함(spock 버전에 따라 조정 필요함)
    testImplementation 'org.codehaus.groovy:groovy:3.0.8'
}
```

## Groovy 설정

build.gradle

``` groovy
plugins {
    //...
    id 'groovy'
}
```

# JUnit 과 비교

## 라이프사이클

| Spock | JUnit |
| --- | --- |
| Specification | Test class |
| `setup()` | `@Before` |
| `cleanup()` | `@After` |
| `setupSpec()` | `@BeforeClass` |
| `cleanupSpec()` | `@AfterClass` |
| Feature | Test |
| Feature method | Test method |
| Data-driven feature | Theory |
| Condition | Assertion |
| Exception condition | `@Test(expected=…)` |
| Interaction | Mock expectation (e.g. in Mockito) |

[출처](https://spockframework.org/spock/docs/2.0/spock_primer.html#_comparison_to_junit)

## 실제 코드 비교

### JUnit

``` java

@ExtendWith(SpringExtension.class)
@SpringBootTest
@ActiveProfiles({"test"})
@Transactional
class WorkServiceTest {
    @PersistenceContext
    private EntityManager em;
    @Autowired
    private WorkService workService;

    @Test
    public void 업무생성() {
        WorkCreateReqDto req = new WorkCreateReqDto().setName("작업1").setWorkerId(1);
        WorkCreateRespDto work = workService.createWork(req);
        assertTrue(work.isSuccess() == true);
        assertTrue(work.getId() > 0);

        // 중복 생성 시 중복 생성 예외 발생
        Exception exception = assertThrows(AlreadyExistException.class, () -> {
            workService.createWork(req);
        });
        assertEquals(exception.getClass(), AlreadyExistException.class);
    }

    @Test
    public void 상태변경() {
        WorkCreateReqDto req = new WorkCreateReqDto().setName("작업1").setWorkerId(1);
        WorkCreateRespDto work = workService.createWork(req);

        boolean b = workService.changeWorkStatus(work.getId(), WorkStatusType.WORKING);
        assertTrue(b);

        em.clear();
        Optional<WorkRespDto> work1 = workService.getWork(work.getId());
        assertEquals(work1.get().getWorkStatus(), WorkStatusType.WORKING);
    }
    
}
```

### Spock Framework

``` groovy

@SpringBootTest
@ActiveProfiles("test")
@Transactional
class WorkServiceSpec extends Specification {

    @PersistenceContext
    EntityManager em

    @Autowired
    WorkService workService

    static def createRequest = new WorkCreateReqDto(
            name: "작업1",
            workerId: 1
    )

    void "업무생성"() {
        given:

        when:
        def work = workService.createWork(createRequest)

        then:
        work.success == true
        work.id > 0
    }

    void "중복된 업무생성시 생성예외 발생"() {
        given:
        workService.createWork(createRequest)
        when:
        workService.createWork(createRequest)


        then:
        thrown(AlreadyExistException)
    }

    void "상태변경"() {
        given:
        def work = workService.createWork(createRequest)

        when:
        def changeResult = workService.changeWorkStatus(work.id, WorkStatusType.WORKING)
        em.clear()
        def changedWork = workService.getWork(work.id).get()

        then:
        changeResult == true
        changedWork.workStatus == WorkStatusType.WORKING
    }

}
```

위와 같이 Groovy의 문법적 특징과 Spock Framework의 Block을 활용하여 좀 더 가독성 있고 간결한 테스트를 작성할 수 있습니다
