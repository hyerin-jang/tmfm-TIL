# 11.5. Method Security
- 2.0 버전 이후 스프링 시큐리티는 서비스 레이어 메소드 보호를 위한 기능을 대폭 개선
- 프레임워크의 기존 @Secured 어노테이션 외에 JSR-250 어노테이션도 지원
- 3.0 부터는 새로운 표현식 기반 어노테이션도 사용할 수 있음
- 원하는 빈을 선언한 곳을 intercept-methods 요소로 장식하면 단일 빈을 보호할 수도 있고, AspectJ 스타일 포인트 컷으로 서비스 레이어 전체에 걸친 빈 보호 가능

## 11.5.1. EnableGlobalMethodSecurity
- @Configuration 인스턴스 중 아무곳에나 @EnableGlobalMethodSecurity 어노테이션을 붙이면 어노테이션 기반 보안 기능을 할성화 할 수 있음
````
@EnableGlobalMethodSecurity(securedEnabled = true)
public class MethodSecurityConfig {
// ...
}
````
- 클래스나 인터페이스에 있는 메소드에 어노테이션을 달면 해당 메소드의 조건에 따라 접근을 제한
- 스프링 시큐리티의 네이티브 어노테이션은 메소드의 속성 셋을 정의
- 이 값은 실제 결정을 내리는 AccessDecisionManager로 전달
````
public interface BankService {

@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account readAccount(Long id);

@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account[] findAccounts();

@Secured("ROLE_TELLER")
public Account post(Account account, double amount);
}
````
- JSR-250 어노테이션은 다음과 같이 활성화 할 수 있음
````
@EnableGlobalMethodSecurity(jsr250Enabled = true)
public class MethodSecurityConfig {
// ...
}
````
- 이는 표준을 따르며 간단한 role 기반 제약 조건을 적용할 수는 있지만 스프링 시큐리티의 네이티브 어노테이션 같은 기능은 없음
- 새로운 표현식 기반 문법을 사용하려면 다음과 같이 작성해야 함
````
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig {
// ...
}
````
````
public interface BankService {

@PreAuthorize("isAnonymous()")
public Account readAccount(Long id);

@PreAuthorize("isAnonymous()")
public Account[] findAccounts();

@PreAuthorize("hasAuthority('ROLE_TELLER')")
public Account post(Account account, double amount);
}
````

## 11.5.2. GlobalMethodSecurityConfiguration
- @EnableGlobalMethodSecurity 어노테이션이 지원하는 것보다 더 복잡한 작업이 필요한 인스턴스는 
GlobalMethodSecurityConfiguration을 확장해서 하위클래스에도 @EnableGlobalMethodSecurity 어노테이션을 달아주면 됨
````
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
    @Override
    protected MethodSecurityExpressionHandler createExpressionHandler() {
        // ... create and return custom MethodSecurityExpressionHandler ...
        return expressionHandler;
    }
}
````

## 11.5.3.The <global-method-security> Element
- 이 요소는 어노테이션 기반 보안을 활성화하며 어플리케이션 컨텍스트 전역에 적용할 포인트컷 선언을 함께 묶을 때도 사용
- <global-method-security /> 요소 하나만 선언하면 됨
````
<global-method-security secured-annotations="enabled" />
````
- 클래스나 인터페이스에 있는 어노테이션을 달면 이제 해당 메소드의 조건에 따라 접근을 제한
- 스프링 시큐리티의 네이티브 어노테이션은 메소드의 속성 셋을 정의
- 이 값은 실제 결정을 내리는 AccessDecisionManager로 전달
````
public interface BankService {

@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account readAccount(Long id);

@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account[] findAccounts();

@Secured("ROLE_TELLER")
public Account post(Account account, double amount);
}
````
- JSR-250 어노테이션은 다음과 같이 활성화 할 수 있음
````
<global-method-security jsr250-annotations="enabled" />
````
- 이는 표준을 따르며 간단한 role 기반 제약조건을 적용할 순 있지만 스프링 시큐리티의 네이티브 어노테이션같은 기능은 없음
- 새로운 표현식 기반 문법을 사용하려면 아래와 같이 작성해야 함
````
<global-method-security pre-post-annotations="enabled" />
````
````
// in java
public interface BankService {

@PreAuthorize("isAnonymous()")
public Account readAccount(Long id);

@PreAuthorize("isAnonymous()")
public Account[] findAccounts();

@PreAuthorize("hasAuthority('ROLE_TELLER')")
public Account post(Account account, double amount);
}
````
- 사용자 권한 리스트로 role 이름을 확인하는 것 외에 다른 규칙을 간단하게 정의해야 한다면 표현식 기반 어노테이션을 사용하는게 좋음


- 어노테이션을 붙인 메소드는 스브링빈으로 정의한 인스턴스일때만 보안이 적용
- 스프링이 생성하지 않는 인스턴스를 보호하고 싶다면 AspectJ를 사용해야 함


- 어플리케이션 내에선 어노테이션 유형을 여러개 활성화해도 되지만 특정 인터페이스나 클래스에는 한가지 유형만 적용해야함
- 그렇지 않으면 의도한대로 정의되지 않음
- 메소드 하나에 두 어노테이션을 발견하면 둘 중 하나만 적용

## 11.5.4. Adding Security Pointcuts using protect-pointcut
- protect-pointcut은 좀 더 강력한 기능을 제공하는데 간단한 선언문 하나만으로도 많은 빈 보호 가능
````
<global-method-security>
<protect-pointcut expression="execution(* com.mycompany.*Service.*(..))"
    access="ROLE_USER"/>
</global-method-security>
````
- 여기선 어플리케이션 컨텍스트에 선언한 빈 중 com.mycompany 패키지에 있으며 Service로 끝나는 클래스의 모든 메소드를 보호
- ROLE_USER role이 있는 사용자만 이 메소드를 실핼할 수 있음
- URL 매칭과 마찬가지로 첫번째로 매칭되는 표현식을 사용하기 때문에 가장 구체적인 조건이 앞에 있어야 함
- 시큐리티 어노테이션이 포인트컷보다 우선

# 11.6. Domain Object Security (ACLs)
## 11.6.1. Overview
## 11.6.2. Key Concepts
## 11.6.3. Getting Started