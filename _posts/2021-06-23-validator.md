---
layout: post
title: Validator와 BindingResult, Errors
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: 모델 오브젝트 검증 실패와 관련된 API
fullview: false
comments: true
---

* @ModelAttribute로 지정된 모델 오브젝트의 바인딩 작업이 실패로 끝나는 경우는 두 가지가 있다 :
	* 타입 변환이 불가능한 경우
	* 타입 변환은 성공했지만, 검증기를 이용한 검사를 통과하지 못한 경우

### Validator
* 스프링에서 범용적으로 사용할 수 있는 오브젝트 검증기를 정의할 수 있는 API
* @Controller로 HTTP 요청을 @ModelAttribute 모델에 바인딩 할 때 주로 사용된다.
* 비즈니스 로직에서 검증 로직을 분리하고 싶을 때도 사용 가능하다.

```java
public interface Validator {

	boolean supports(Class<?> clazz);
	void validate(Object target, Errors errors);
```

* Validator 인터페이스는 위와 같이 두 개의 메소드로 구성되어 있다.
* supports() : 해당 검증기가 검증할 수 있는 오프젝트 타입인지 확인해주는 메소드, 통과시에만 validate()를 호출한다.
* validate() : 검증하는 메소드

사용 예시 : 

```java
public Class UserValidator implements Validator {
	public boolean supports(Class<?> clazz) {
		return (User.class.isAssignableFrom(Clazz));
	}
	
	public void validate(Object target, Errors errors) {
		User user = (User)target;
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "field.required");
		if(user.getAge() < 0) {
			errors.rejectValue("name", "field.min", new Object[]{0}, null);	// 세번째 파라미터는 에러코드, 네번째 파라미터는 에러 메시지이다.
		}
	}
}
```

* 검증 후 아무 문제가 없다면 메소드를 정상종료한다.
* 오류 발견시 Errors 인터페이스를 통해 특정 필드나 모델 오브젝트 전체에 대해 오류정보를 등록할 수 있다.
* supports() 메소드 사용시 검증하려는 클래스의 서브클래스가 사용될 수 있으므로 isAssignableFrom() 메소드를 사용하는 것이 좋다.
* validator는 보통 필수 값의 입력 여부나 값의 범위, 길이, 형식 등을 주로 검증조건으로 쓸 때 싸용한다.

스프링 Validator 적용 방법은 아래 네가지가 있다 : 

#### 컨트롤러 메소드 내의 코드
* Validator는 빈으로 등록이 가능하기 때문에, 이를 컨트롤러에서 DI 받아두고 각 메소드에서 필요에 따라 직접 validate()를 호출해 검증 작업을 진행해도 좋다.

```java
@Controller
public class UserController {
	@Autowired UserValidator validator;
	
	@RequestMapping("/add")
	public void add(@modelAttribute User user, BindingResult result) {
		this.validator.validate(user, result);
		if(result.hasErrors()) {
			// 오류가 발견된 경우
		} else { 
			// 발견되지 않았을 경우
		}
	}
}
```

* validate() 메소드 실행 후 BindingResult의 hasErrors() 메소드로 검증작업에서 등록된 오류가 있는지 확인 할 수 있다.

#### @Valid를 이용한 자동검증
* JSR-303의 @javax.validation.Valid 애노테이션을 사용할 수 있다.
* 스프링이 @Valid 애노테이션을 차용했을 뿐 내부적으로는 스프링의 Validator를 이용한 검증이 수행된다.
* 컨트롤러에서 Validator 오브젝트를 가져다 validate() 메소드를 실행하는 대신 바인딩 과정에서 자동으로 검증이 진행된다.
* WebDataBinder에 Validator 타입의 검증용 오브젝트를 설정해 적용한다.

```java
@Controller
public class UserController {
	@Autowired UserValidator validator;
	
	@InitBinder
	public void initBinder(WebDataBinder dataBinder) {
		dataBinder.setValidator(this.validator);
	}
	
	@RequestMapping("/add")
	public void add(@ModelAttribute @Valid User user, BindingResult result) {
	}
	....
}
```
* 컨트롤러 메소드의 @ModelAttribute 파라미터에 @Valid 애노테이션을 추가한다.
* 이 모델을 바인딩하는 WebDataBinder는 @InitBinder 메소드에서 등록된 Validator를 이용해 모델을 검증하고, 그 결과를 BindingResult에 넣는다.

#### 서비스 계층 오브젝트에서의 검증
* 자주 사용되는 방법은 아니다.
* 여러 개의 서비스 계층 오브젝트에서 반복적으로 같은 검증 기능이 사용된다면, Validator로 검증 코드를 분리하고 이를 DI 받아 사용할 수 있다.
* 이 때는 BindingResult 타입의 오브젝트를 직접 만들어서 Validator의 validate() 메소드로 전달해야 하며, BeanPropertyBindingResult를 사용하는 것이 적당하다.

#### 서비스 계층을 활용하는 Validator
* 코드에 의한것이든, @Valid에 의한 것이든 상관없이 Validator를 적용했을 경우 Validator를 빈으로 등록해 서비스 계층의 기능을 사용해 검증 작업을 수행할 수 있다.
* 서비스 계층에 담긴 검증 로직을 재사용하며 그 결과는 컨트롤러 내에 BindingResult를 통해 전달받을 수 있다.


### JSR-303 빈 검증 기능
* 표준 스펙으로 인정받은 JSR-303은 모델 오브젝트의 필드에 달린 제약조건 어노테이션을 이용해 검증을 진행한다.
* @NotNull, @Min, @Max등이 있다.
* 이 어노테이션을 사용하려면 LocalValidatorFactoryBean을 빈으로 등록해야 한다. 이 팩토리 빈이 생성하는 빈의 타입은 스프링의 Validator다.
* 스프링의 Validator는 PropertyEditor, ConversionService와 마찬가지로 WebDataBinder를 이용해 초기화 할 수 있으므로, WebBindingInitializer를 이용해 모든 컨트롤러에 일괄적으로 적용할 수 있다.


### BindingResult와 MessageCodeResolver
* 모델의 바인당 작업 중 나타난 타입 변환 오류정보와 검증 작업에서 발견된 검증 오류정보가 모두 저장된다.
* 보통 컨트롤러에 의해 폼을 다시 띄울 때 활용된다.

```java
ValidationUtils.rejectIfEmpty(errors, "name", "field.required");
```

* 위 코드에서 세번째 파라미터인 `field.required`는 messages.properties의 프로퍼티 키 값이다. 프로퍼티 파일에는 아래와 같이 선언해야 한다.

```
fields.required = 필수 입력 항목입니다.
```

* 에러코드는 MessageCodeResolver를 통해 좀 더 상세한 메시지 키 값으로 확장된다. 스프링이 디폴트로 사용하는 MessageCodeResolver인 DefaultMessageCodeResolver는 이를 좀 더 확장해 아래와 같이 키 후보를 생성한다 : 
	* 에러코드.모델이름.필드이름 : field.required.user.name
	* 에러코드.필드이름 : field.required.name
	* 에러코드.타입이름 : field.required.nUser
	* 에러코드 : field.required
* 위에서부터 우선적으로 메세지를 찾는데 사용된다. 우선순위가 높은 메시지 키가 먼저 발견되면 그 이후에 일치하는 것이 발견되더라도 무시된다.


* 바인딩 작업 중 타입 변환을 할 수 없는 경우, Validator처럼 직접 에러코드를 만들 수 없기 때문에 스프링이 typeMismatch라는 에러 코드를 지정해준다.
* 그에 맞는 메시지를 messages.properties에 넣어줄 수 있다.
* 검증 작업 중 발견한 오류가 특정 필드에 속한 것이 아닐 경우, reject()를 사용해 모델 레벨의 글로벌 에러로 등록할 수 있다.이 때는 필드 이름이 없어 에러코드와 모델 이름을 이용해 아래 두가지 메시지 키 후보를 생성해준다 : 
	* 에러코드.모델이름 : invalid.data.user
	* 에러코드 : invalid.data

### MessageSource
* MessageCodeResolver를 통해 여러 개의 후보로 만들어진 메시지 코드는 어떻게 MessageSourceResolver를 한 번 더 거쳐 최종적인 메시지로 만들어진다.
* MessageSource의 구현 방식은 두가지가 있다 : 
	* 코드로 메시지를 등록할 수 있는 StaticMessageSource
	* messages.properties 리소스 번들 방식을 사용하는 ResourceBundleMessageSource => 일정시간마다 파일 변경 여부를 확인해 메시지를 갱신해준다.

MessageSource는 아래 네가지 정보를 활용해 최종적인 메시지를 만들어낸다 : 

* **코드** : BindingResult 또는 Errors에 등록된 에러 코드를 DefaultMessageCodeResolver를 이용해 필드와 오브젝트 이름의 조합으로 만들어낸 메시지 키 후보 값이 있다. 메시지 키 이름 후보가 MessageSource의 첫번째 정보인 코드로 사용된다.
* **메시지 파라미터 배열** : BindingResult나 Errors의 rejectValue(), reject()에는 Object[] 타입의 세 번째 파라미터를 지정할 수 있다.
	* `field.min={0}보다 적은 값은 사용할 수 없습니다.` 라고 입력하고, rejectValue()의 세번째 파라미터를 new Object[] {100} 이라고 했다면 최종 메시지는 "100보다 작은 값은 사용할 수 없습니다"라고 만들어질 것이다.
* **디폴트 메시지** : 메시지 키 후보 값을 모두 이용해 messages.properties를 찾아봤지만 메시지가 발견되지 않을 경우 사용할 수 있는 방법이다.
	* `rejectValue("name","field.required", null, "입력해주세요");` 라고 적고, 프로퍼티 파일에서 field.required를 비롯한 네가지 키에 해당하는 메시지를 찾을 수 없다면, 그 때 디폴트 메시지가 사용된다.
	* 에러코드를 충실히 적용했다면 디폴트 메시지는 생략하거나 null로 넣어주면 된다. 
* **지역정보** : LocalResolver에 의해 결정된 현재의 지역정보이다.

* 이 네가지 정보에 의해 MessageSource가 최종 결정한 메시지가 폼에 나타날 것이다.
* 메시지 소스는 디폴트로 등록되지 않기 때문에, 아래와 같이 빈으로 등록해주어야 한다.

```xml
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
	<property name="basename" value="messages"		/>
</bean>
```

* 보통 메시지를 담은 프로퍼티 파일의 이름은 messages.properties를 기본으로 사용한다.
* 다른 이름을 사용하고 싶다면 property value를 수정하면 된다.