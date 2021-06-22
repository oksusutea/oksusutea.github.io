---
layout: post
title: 모델 바인딩과 검증
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: 모델 바인딩 검증 방법
fullview: false
comments: true
---

* 컨트롤러 메소드에 @ModelAttribute가 지정된 파라미터를 @Controller 메소드에 추가하면 크게 세가지 작업이 자동으로 진행된다.
	* 파라미터 타입의 오브젝트를 만든다.
		* 이를 위해 디폴트 생성자가 반드시 필요하다.
		* @SessionAttributes에 의해 세션에 저장된 모델 오브젝트가 있다면, 새로운 오브젝트 생성 대신 세션에 저장되어 있는 오브젝트를 가져온다.
	* 준비된 모델 오브젝트의 프로퍼티에 웹 파라미터를 바인딩 해준다.
		* 기본 프로퍼티 에디터를 이용해 HTTP 파라미터 값을 모델의 프로퍼티 타입에 맞게 전환한다.
		* 전환이 불가능한 경우라면 BindingResult 오브젝트 안에 바인딩 오류를 저장해 컨트롤러에 넘겨주거나 예외를 발생시킨다.
	* 모델의 값을 검증한다.
		* 바인딩 단계에서 타입에 대한 검증은 끝났지만, 그 외 검증할 내용이 있다면 적절한 검증기를 등록해 모델의 내용을 검증한다.

## 프로퍼티 바인딩
* 스프링에서 말하는 바인딩은 오브젝트의 프로퍼티에 값을 넣는 것을 말한다.
* 프로퍼티 바인딩은 프로퍼티의 타입에 맞게 주어진 값을 적절히 변환하고, 프로퍼티의 수정자 메소드를 호출해 값을 넣는 두 가지 작업이 필요하다.
* 스프링은 바인딩 과정에서 필요한 변환 작업을 위해 기본적으로 두가지 종류의 API를 제공한다.

### PropertyEditor
* 스프링이 기본적으로 제공하는 바인딩용 타입 변환 API는 PropertyEditor다.
* 해당 API는 자바 빈 표준에 정의된 인터페이스다.

### 디폴트 프로퍼티 에디터
* 프로퍼티 에디터는 XML의 value 애트리뷰트 뿐 아니라 @Controller의 파라미터에도 동일하게 적용된다.
* 바인딩 과정에서는 변환할 파라미터 또는 모델 프로퍼티의 타입에 맞는 프로퍼티 에디터가 자동으로 선정돼서 사용된다.

### 커스텀 프로퍼티 에디터
* 지원하지 않는 커스텀 타입을 @RequestParam, 혹은 @ModelAttributes 파라미터로 넣을 경우 ConversionNotSupportedException 예외로 HTTP 500 에러를 만나게 된다.
* 커스텀 프로퍼티 에디터를 만들기 위해서는 PropertyEditor를 구현해야 하며, 그 과정에서 사용되는 메소드는 총 네 가지가 있다.
	* 먼저 서블릿에서 스트링 타입으로 문자열을 가져온다.
	* setAsText() 메소드를 이용해 스트링 타입의 문자열을 넣는다.
	* getValue()로 변환된 오브젝트를 가져온다.
* 오브젝트를 문자열로 바꿀 때에는 반대의 순서로 진행한다 : 
	* setValue()로 오브젝트를 넣고
	* getAsText()로 변환된 문자열을 가져온다.
* 프로퍼티 에디터를 만들 때에는 ProPertyEditor 인터페이스를 직접 구현하기 보다는 기본 구현이 되어있는 PropertyEditorSupport 클래스를 상속해 필요한 메소드만 오버라이드 하는 편이 낫다.

```java
public class LevelPropertyEditor extends PropertyEditorSupport { 
	public String getAsText() {
		return String.valueOf(((Level)this.getValue()).intValue());
	}
	public void setAsText(String text) throws IllegalArgumentException {
		this.setValue(Level.valueOf(Integer.parseInt(text.trim())));
	}
}
```

### @InitBinder
* 프로퍼티 에디터를 통해 문자열과 오브젝트 사이에 타입 변환이 어떻게 일어나는지 확인했다.
	* 문자열 -> 오브젝트 : setAsText() [오브젝트 타입의 PropertyEditor] -> getValue() [오브젝트]
	* 오브젝트 -> 문자열 : setValue() [오브젝트] -> getAsText() [오브젝트 타입의 PropertyEditor]
* 스프링 MVC는 디폴트 프로퍼티 에디터만 등록되어 있기 때문에, 커스텀 에디를 추가해 특정 타입의 변환이 필요할 때 사용되도록 만들어야 한다.
* 컨트롤러 메소드에서 바인딩이 어떻게 일어날까?
	* @Controller를 호출해줄 책임이 있는 AnnotationMethodHandlerAdapter는 @RequestParam, @ModelAttribute등 HTTP 요청을 파라미터 변수에 바인딩해주는 작업이 필요한 애노테이션을 만나면 WebDataBinder를 만든다.
	* WebDataBinder는 여러가지 기능을 제공하지만, 그 중에 HTTP요청으로부터 가져온 문자열을 파라미터 타입의 오브젝트로 변환하는 기능도 포함되어 있다. 이 변환 갖업은 프로퍼티 에디터를 이용한다.
	* 따라서 메소드 파라미터 바인딩에 커스텀 프로퍼티 에디터를 적용하려면 WebDataBinder에 프로퍼티 에디터를 직접 등록해야 한다.
	* WebDataBinder 초기화 메소드를 이용해 프로퍼티 에디터를 등록할 수 있다.

```java
@InitBinder
public void initBinder(WebDataBinder dataBinder) {
	dataBinder.registerCustomEditor(Level.class, new LevelPropertyEditor());
}
```

* @InitBinder가 붙은 메소드는 메소드 파라미터를 바인딩 하기 전에 자동으로 호출된다.
* @InitBinder에 의해 등록된 커스텀 에디터는 같은 컨트롤러 메소드에서 HTTP 요청을 바인딩하는 모든 작업에 적용된다.  
* 적용 대상 : @RequestParam, @CookieValue, @RequestHeader, @PathVarible, @ModelAttribute

#### WebDataInitBinder에 커스텀 프로퍼티 에디터를 등록하는 방법

#### 특정 타입에 무조건 적용되는 프로터피 에디터 등록
* 위 코드처럼 적용 타입과 프로퍼티 에디터 두 개를 파라미터로 받는 registerCustomEditor()을 사용해 프로퍼티 에디터로 등록했다면, 해당 타입을 가진 바인딩 대상이 나오면 항상 프로퍼티 에디터가 적용된다.

#### 특정 이름의 프로퍼티에만 적용되는 프로퍼티 에디터 등록
* 같은 타입이지만, 프로퍼티 이름이 일치하지 않는 경우 등록한 커스텀 프로퍼티 에디터가 적용되지 않게 설정할 수 있다.
* 프로퍼티 이름이 필요하므로 @RequestParam과 같은 단일 파라미터 바인딩에는 적용되지 않는다. @ModelAttribute로 지정된 모델 오브젝트의 프로퍼티 바인딩에 사용할 수 있다.
* 보통 프로퍼티 에디터가 존재하는 경우 사용하기 적합하다.
* WebDataBinder는 커스텀 프로퍼티 에디터가 우선순위를 갖는다( 즉, 바인딩 작업시 커스텀 프로퍼티 에디터로 먼저 적절하게 적용 가능한지 확인한다.)

```java
@InitBinder
public void initBinder(WebDataBinder dataBinder) {
	dataBinder.registerCustomEditor(int.class, "age", new MinMaxPropertyEditor(0,200));	// int타입의 age라는 이름을 가진 프로퍼티만 MinMaxPropertyEditor를 적용한다.
}
```

* @InitBinder 메소드의 파라미터에는 WebDataBinder 외에도 WebRequest도 사용할 수 있다. WebRequest는 사용할 수 있는 모든 HTTP 요청정보를 담고 있다. 따라서 HTTP 요청에 따라 다른 방식으로 WebDataBainder에 프로퍼티 에디터 등을 설정하는데 사용할 수 있다.

### WebBindingInitializer
* 특정 컨트롤러의 메소드에서만 적용하는 것이 아니라, 모든 컨트롤러에 적용해도 될 만큼 많은 곳에서 필요한 프로퍼티 에디터라면 방법을 달리해서 한 번에 모든 컨트롤러에 적용하는 편이 좋다.
* 이 때에는 WebBindingInitializer를 이용하면 된다.

```java
public class MyWebBindingInitializer implements WebBindintInitializer {
	public void initBinder(WebDataBinder binder, WebRequest request) {
		binder.registerCustomEditor(Level.class, new LevelPropertyEditor());
	}
}
```

```xml
<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter" >
	<property name="webBindingInitializer">
		<bean class="springbook..... MyWebBindingInitializer>
	</property>
</bean>
```
* WebBindingInitializer를 구현해서 만든 클래스를 빈으로 등록하고 @Controller를 담당하는 어댑터 핸들러인 AnnotationMethodHandlerAdapter의 webBindingInitializer 프로퍼티에 DI해준다.
* AnnotationMethodHandlerAdapter 레벨에 바인딩을 초기화하는 빈을 등록해주면 이 빈의 initBinder() 메소드는 모든 컨트롤러의 모든 바인딩 작업에 일괄 적용된다.

### 프로토타입 빈 프로퍼티 에디터
* 프로퍼티 에디터는 싱글톤 빈으로 등록될 수 없다.
	* 다시 복습하자면, 프로퍼티 에디터에 setValue()를 이용해 오브젝트를 일단 넣은 후, getAsText()로 변환된 문자열을 가져오는 식이다.
	* 프로퍼티 에디터에 의해 타입이 변경되는 오브젝트는 한 번은 프로퍼티 에디터 오브젝트 내부에 저장된다. 상태 값을 가지고 있는 오브젝트이기에, 싱글톤 방식이라면 멀티스레드 환경에서 공유해서 사용하게 된다.
* 만일 프로퍼티 에디터가 다른 스프링 빈을 참조해야 한다면 어떨까? 이럴 떄는 자신도 스프링의 빈으로 등록해야 하는 것이 원칙이다. 그러기 위해서는, 프로토타입 빈으로 생성되어야 한다.
	* HTTP 요청 파라미터로 도메인 오브젝트의 ID를 제공받으면, 바인딩 과정에서 ID에 해당되는 실제 도메인 오브젝트로 만들어 주는 경우

```java
public class User {
	int id;
	String name;
	Code userType;
}
```

* 위와 같이 도메인 내 또다른 타입이 들어있는 경우, User만 프로퍼티 에디터로 해결할 경우 userType에 대한 타입 검증 및 바인딩은 수행되지 않는다.
* 이렇게 모델의 파라미터에 바인딩될 때 단순 타입이 아닌 경우 어떻게 바인딩 할 수 있을까? 

#### 별도의 codeid 필드로 바인딩하는 방법
* 가장 단순하지만 서비스 계층이나 컨트롤러에서 추가로 해줘야 할 작업이 있다.
* Code userType 프로퍼티로 직접 바인딩하는 대신 참조 ID 값을 저장할 수 있도록 별도의 임시 프로퍼티를 만들고 이 프로퍼티 값을 받는 것이다.

```java
public class User {
	int id;
	String name;
	Code userType;
	int userTypeId;
}
```

```html
<select name="userTypeId">
	...
</select>
```

```java
@RequestParam("/add")
public void add(@ModelAttribute User user) {
	user.setUserType(this.codeService.getCode(user.getUserTypeId()));
}
```

* 이렇게 되면 전달받은 userTypeId를 통해 컨트롤러나 서비스 계층에서 적절한 Code 타입의 오브젝트로 변경해 userType 프로퍼티를 설정해줄 수 있다.
* 이 방식의 가장 큰 단점은 매번 컨트롤러나 서비스 계층 코드에서 id에 해당하는 임시 프로퍼티 값을 이용해 도메인 오브젝트 타입의 프로퍼티를 설정해주는 작업을 해야한다는 것이다.
* 또 한가지 단점은 User 오브젝트에 굳이 필요하지 않은 userTypeId와 같은 임시 저장용 프로퍼티가 추가돼야 한다는 점이다.

### 모조 오브젝트 프로퍼티 에디터
* 모조 프로퍼티 에디터가 변환해주는 Code 오브젝트는 오직 id 값만 가진 불완전한 오브젝트다.
* id값 외 나머지 값은 모두 null이기 때문에, User타입(상위타입)을 update할 때에만 유용하다.
* 실수로 모조 오브젝트를 사용하는 일을 미연에 방지하기 위해서는 , Code를 확장한 FakeCode를 만들어 적용할 수 있다.
	* 모조 오브젝트를 사용할 때 필요한 id 값을 가져오는 메소드 외 모든 메소드는 오버라이드 해 UnsupportedOperationException 예외를 던지도록 만드는 것이다.

### 프로토타입 도메인 오브젝트 프로퍼티 에디터
* 모조 오브젝트와 다르게 DB에서 읽어온 완전한 Code 오브젝트로 변환해준다.
* 이를 위해서는 프로퍼티 에디터가 CodeService나 CodeDao같은 빈을 DI 받아서 사용해야 하며, 해당 에디터도 빈으로 등록되어야 한다.

```java
@Component
@Scope("prototype")
public class CodePropertyEditor extends PropertyEditorSupport { 
	@Autowired CodeService codeService;
	
	public void setAsText(String text) throws IllegalArgumentException {
		setValue(this.codeService.getCode(Integer.parseInt(text)));
	}
	
	public String getAsText() {
		return String.valueOf(((Code)getValue()).getId());
	}
}
```
* 아래와 같이 사용하고자 하는 컨트롤러에서 Provider를 통해 프로토타입 빈을 사용할 수 있도록 @Inject 애노테이션을 이용하자.

```java
public class UserController {
	@Inject Provider<CodePropertEditor> codeEditorProvider;
	
	@InitBinder public void initBinder(WebDataBinder dataBinder) {
		dataBinder.registerCustomEditor(code.class, codeEditorProvider.get());
	}
}
```

* 위와 같이 작성하면 userType에 id 값으로 전달되어도 온전한 Code 오브젝트로 변환될 것이다.
* 물론 폼의 오브젝트 정보만 갖고 있는 UserFormDto를 사용한다거나, Map<String, String>으로 받아 폼 정보를 서비스 계층으로 보낼 수도 있다.
* 즉, 데이터 중심 아키텍처를 사용하면 도메인 오브젝트의 프로퍼티 타입에 따른 프로퍼티 에디터의 사용 방법 따위는 걱정하지 않아도 된다.

***
참고자료 :  
[토비의 스프링 3.1 Vol.2 4.3 - 모델 바인딩과 검증](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19505671)