# Spring Data JPA


아직 Spring & MyBatis를 사용한다고 하지만 이젠 생산성이 좋은 JPA를 많이 쓴다고 하니 해보자.

<!--more-->

## Entity

~~~java
@NoArgsConstructor
@Getter
@Entity
public class Students extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String studentId;
    private String password;
    private String name;
    private String birthDay;
    private String email;
    private String phone;

    @Builder
    public Students(String studentId, String password, String name,
                    String birthDay, String email, String phone){
        this.studentId = studentId;
        this.password = password;
        this.name = name;
        this.birthDay = birthDay;
        this.email = email;
        this.phone = phone;
    }

    public void updateStudent(UpdateStudentReq dto) {
        this.password = dto.getPassword();
        this.birthDay = dto.getBirthDay();
        this.phone = dto.getPhone();
        this.email = dto.getEmail();
    }
}
~~~

여기서 Student 클래스는 실제 DB의 테이블과 Mapping될 클래스이고 Entity 클래스라고 한다.

* @Entity

1. 테이블과 연결될 클래스이다.
2. `_`으로 이름을 매칭한다.  (ex. JaejinTest -> Jaejin_Test)

* @Id

1. 해당 테이블의 PK필드를 나타낸다.

* @GeneratedValue

1. PK의 생성 규칙
2. default는 AUTO이고 auto_increment와 같이 자동 증가하는 정수 값이 된다.
3. spring2.x버전 부터는 조심 ! [참고 blog](https://jojoldu.tistory.com/295)

* @Column

1. 테이블의 컬럼 굳이 사용안해도 모두 컬럼이 된다. 다음과 같이 모두 그냥 써도 컬럼으로 인식한다.
~~~java
private String studentId;
private String password;
private String name;
~~~
2. 기본값 외에 추가가 필요한 옵션이 있으면 사용한다. 

JPA는 [lombok 라이브러리](https://projectlombok.org/)하고 같이 사용하니 알아두도록하자.

lombok은 단순히 의존성 추가 뿐만아니라 IDE에 맞게 셋팅이 필요하다.

## Repository

이제 Entity를 Repository와 연결 시켜 보자.

~~~java
@Repository
public interface StudentsRepository extends JpaRepository<Students, Long> {
}
~~~

JpaRepository를 상속 하고 있는데 이렇게 되면 CRUD를 따로 작성 할 필요없이 사용가능하다.

~~~java
private final StudentsRepository studentsRepository;

public Students create(StudentsDto.RegistStudentReq dto) {
    return studentsRepository.save(dto.toEntity());
}
~~~

이렇게 JpaRepository를 상속받은 Repository객체를 이용해서 save 메소드를 바로 호출 할 수 있다.

## Queries

CRUD는 대충 알것 같은데 내가 원하는 Query를 작성하고 싶다면 어떻게 해야할까?

~~~java
@Repository
public interface StudentsRepository extends JpaRepository<Students, Long> {
    List<Student> findByEmailAddressAndName(String emailAddress, String name);
}
~~~

이렇게 한 추가 해보면 다음과 같이 Query로 인식한다.

~~~sql
select s from Students s where s.emailAddress = ?1 and s.name = ?2
~~~

더 많은 Sample을 확인하고 싶다면 [JPA 문서](https://docs.spring.io/spring-data/jpa/docs/1.3.0.RELEASE/reference/html/jpa.repositories.html) Table 2.2. Supported keywords inside method names 를 확인하자.

물론 @Query로 직접 Query를 작성할 수도 있다.

~~~java
@Repository
public interface StudentsRepository extends JpaRepository<Students, Long> {
    @Query("select s from Students s where s.emailAddress = ?1 and s.name = ?2")
    List<Student> findByEmailAddressAndName(String emailAddress, String name);
}
~~~

참고로 Update를 해야하는 Query에는 `@Modifying`을 추가해야한다.

~~~java
@Modifying
@Query("UPDATE Posts p SET p.author = :author, p.content = :content, p.title = :title, p.modifiedDate = :modifiedDate WHERE p.id = :id")
void editPost(@Param("author") String author,
                @Param("content") String content,
                @Param("title") String title,
                @Param("modifiedDate") LocalDateTime modifiedDate,
                @Param("id") Long id);
~~~

---

**`참고`** - https://jojoldu.tistory.com/295
