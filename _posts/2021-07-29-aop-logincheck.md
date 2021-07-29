---
layout: post
title: AOP를 통해 로그인체크를 진행해보자
categories: [dev]
tags: [dev, spring framework]
description: 사이드 프로젝트 리팩토링
fullview: false
comments: true
---
사이드 프로젝트를 진행하면서 컨트롤러 단에서 매번 로그인여부를 체크하는 코드를 중복되어 작성하였었다.

```java
if (!isSignIn(httpSession)) {
	throw new UserNotSignInException();
}
```

이러한 코드가 핸들러마다 작성되어 있으니 개선할 수 있는 방법이 없을까?하다가 AOP를 통해서 로그인 체크를 할 수 있도록 코드를 개선해보기로 하였다.

### 로그인 체크 Aspect 만들기

```java
@Component
@Aspect
public class CheckLoginAspect {

    @Before(value = "@annotation(com.flab.kidsafer.aop.CheckLogin)") // 이어서 생성할 어노테이션
    public void loginCheck() throws HttpClientErrorException {
        HttpSession session = ((ServletRequestAttributes) (RequestContextHolder
            .currentRequestAttributes())).getRequest().getSession();
        int userId = SessionUtil.getLoginUserId(session);
        if (userId == -1) {
            throw new UserNotSignInException();
        }
    }
}
```
`@Component` 어노테이션을 통해 해당 클래스가 빈으로 등록되도록 설정해주었고,  해당 클래스가 횡단관심사(부가기능) Class임을 명시하기 위해 `@Aspect` 를 작성해주었다.  
AOP를 통해 등록할 수 있는 어드바이스는 총 5가지가 있는데(`@Before`, `@Around`, `@AfterReturning`, `@AfterThrowing`, `@After`), 필자의 경우 핸들러 진입 전에 로그인 체크를 해주어야 하기 때문에 `@Before`를 사용하였다.  
세션에서 userId를 가져와서, 없을 경우(Primitive type으로, 없으면 -1을 반환하도록 했었다) 예외를 발생시키도록 작성했다. 세션단에 `userId`가 있을 경우, 이 메소드는 성공적으로 처리가 되고 이어서 핸들러의 메소드를 수행할 것이다.


### 로그인 체크에 사용할 어노테이션 만들기

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface CheckLogin {
}
```

CheckLogin이라는 인터페이스를 만들었다. 이 어노테이션은 주로 메소드에 붙겠지만, 클래스 단에도 붙을 수 있어 `ElementType.TYPE` 또한 추가하였다.  
물론 포인트컷으로 특정 메소드에 한해 사용할 수 있도록 지정할 수 있지만, 아무래도 포인트 컷은 메소드가 추가될 때마다 작성을 해주어야 하기도 하고, type-safety하지 않기 때문에 해당 어노테이션이 붙은 메소드에만 해당 메소드를 수행하도록 하였다.
### 로그인 체크가 필요한 핸들러에 어노테이션 추가하기

```java
@CheckLogin
@PostMapping("/signOut")
public void signOut(HttpSession httpSession) {
	SessionUtil.logoutUser(httpSession);
}
```

많은 핸들러(컨트롤러)가 있지만 그 중에 하나를 예시로 들어보겠다.
로그아웃을 처리하는 핸들러인데, 로그아웃을 하기 위해서는 사전에 로그인이 되어있어야만 가능하다. 그렇기 때문에 해당 핸들러에 `@CheckLogin` 이라는 어노테이션을 추가했고, 기존에 중복적으로 나타나던 코드는 삭제하였다.


### AspectJ Proxy를 생성할 수 있도록 최상단 Application에 셋팅해주기

```java
@EnableAspectJAutoProxy
@SpringBootApplication
public class KidSaferApplication {
	public static void main(String[] args) {
		SpringApplication.run(KidSaferApplication.class, args);
	}
}
```

마지막으로 Application을 구동하는 클래스에서 `@EnableAspectJAutoProxy` 라는 어노테이션을 추가해주었다. 이 어노테이션은 Aspect어노테이션으로 등록한 빈을 다이나믹 프록시 빈으로 생성할 수 있도록 한다.

#### @EnableAspectJAutoProxy

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {
	....
}
```

* @EnableAspectJAutoProxy를 사용하면, `AspectJAutoProxyRegistarar.claass`를 임포트한다. 
* BeanPostProcessor에서 `AnnotationAwareAspectJAutoProxyCreator`를 등록하고, 이를 통해 프록시 빈으로 등록할 수 있도록 해준다.
