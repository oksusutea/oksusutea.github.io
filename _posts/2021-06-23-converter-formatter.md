---
layout: post
title: Converter와 Formatter
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: 멀티스레드 환경에서 안전하게 사용할 수 있는 타입 변환 API
fullview: false
comments: true
---

* PropertyEditor는 매번 바인딩을 할 때마다 새로운 오브젝트를 만들어야 한다.
* Converter는 변환 과정에서 메소드가 한 번만 호출된다. 즉, 변환 작업 중에 상태를 인스턴스 변수로 저장하지 않는다.
* 그로 인해 멀티스레드 환경에서 안전하게 사용할 수 있고, 모든 변환 작업이 필요한 오브젝트에서 DI 받아 사용할 수 있다.

### Converter
* Converter 메소드는 소스 타입에서 타깃 타입으로의 단방향 변환만 지원한다.
* 물론 PropertyEditor처럼 소스와 타깃을 바꿔 컨버터를 하나 더 만들면 양방향 변환이 가능하다.
* Converter는 한쪽이 스트링 타입의 문자열로 고정되어 있지 않다.

```java
public interface Converter<S, T> {
	T convert(S source);
}
```

* convert() 메소드는 단순하다. 소스 타입의 오브젝트를 받아 타깃 타입으로 변환해주면 된다.

```java
public class LevelToStringConverter implements Converter<Level, String> {
	public String convert(Level level) {
		return String.valueOf(level.intValue());
	}
}
```

### ConversionService

* 컨트롤러의 바인딩 작업에도 컨버터를 적용하기 위해서는 ConversionService 타입의 오브젝트를 통해 WebDataBinder에 설정해주어야 한다.
* ConversionService는 여러 종류의 컨버터를 이용해서 하나 이상의 타입 변환 서비스를 제공해주는 오브젝트를 만들 때 사용하는 인터페이스다.
* 보통 ConversionService를 구현한 GenericConversionService 클래스를 빈으로 등록해서 사용하면 된다.
* GenericConversionService는 일반적으로 빈으로 등록하고 필요한 컨트롤러에서 DI받아 @InitBinder 메소드를 통해 WebDataBinder에 설정하는 방식으로 사용한다.

메소드 파라미터를 위해 사용하는 WebDataBinder에 GenericConversionService를 설정하는 방법은 두가지가 있다 : 

#### @InitBinder를 통한 수동 등록
* 일부 컨트롤러에만 직접 구성한 ConversionService를 적용한다거나, 하나 이상의 ConversionService를 만들고 컨트롤러에 따라 다른 변환 서비스를 선택하고 싶다면 @InitBinder 메소드를 통해 직접 원하는 ConversionService를 설정해줄 수 있다.
* GenericConversionService에 컨버터를 등록하는 방법 : 
	* GenericConversionService를 상속해서 새로운 클래스를 만들고, 생성자에 addConverter() 메소드를 이용해 추가할 컨버터 오브젝트를 등록하는 방법
	* 추가할 컨버터 클래스를 빈으로 등록해두고, ConversionServiceFactoryBean을 이용해 프로퍼티로 DI 받은 컨버터들로 초기화된 GenericConversionService를 가져오는 방법 (클래스를 만들지 않고 설정만으로 사용가능)

```xml
<bean class="org.springframework.context.support.ConversionServiceFactoryBean">
	<property name="converters">
		<set>
			<bean class="springbook...LevelToStringConverter"    />   
			<bean class="springbook...StringToLevelConverter"    />
		</set>
	</property>
</bean>
```	

```java
@Controller
public static class SearchController {
	@Autowired
	ConversionService conversionService;
	
	@InitBinder
	public void initBinder(WebDataBinder dataBinder) {
		data.setConversionService(this.conversionService);
	}
}
```

* ConversionServiceFactoryBean 빈을 통해 등록된 컨버터들이 모든 컨트롤러의 메소드 파라미터 바인딩에 사용된다.
* 매번 개별적으로 오브젝트를 만들어주어야 하는 프로퍼티 에디터와 달리, 컨버터를 비롯한 새로운 타입 변환 오브젝트는 싱글톤으로 가능하기 때문에 이렇게 하나의 컨버전 서비스에 모두 등록하고 한 번에 지정할 수 있다.
* WebDataBinder는 하나의 ConversionService 타입 오브젝트만 허용한다.

#### ConfigurableWebBindingInitializer를 이용한 일괄 등록
* 컨버터는 싱글톤이기 때문에 모든 컨트롤러의 WebDataBinder에 적용해도 문제 없다.
* 그렇기 때문에 번거롭게 DI와 @InitBinder 메소드를 사용해 일일이 컨버전 서비스를 지정해주는 대신, 모든 컨트롤러에 한 번에 적용하는 방법도 있다.
* 이 때 WebBindingInitializer를 사용하며, 이건 모든 컨트롤러에 일괄 적용되는 @InitBinder 메소드를 정의한 것이라고 볼 수 있다.

```xml
<bean class="org.springframework.web.servlet.mvc.annotation.AnntationMethodHandlerAdapter">
	<property name="webBindingInitializer" ref="webBindingInitializer"   />
</bean>

<bean class="org.springframework.web.bind.support.ConfigurableWebBindingInitializer">
	<property name="conversionService" ref="conversionService"      /> 
</bean>

<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
	<property name="converters">
		... <!-- 적용할 컨버터 빈 목록 -->
	</property>
</bean>
```

* 이렇게 작성하면 일일이 빈을 DI받고 @InitBinder를 이용해 바인더를 초기화하는 코드를 제거할 수 있어 단순해진다.


***

### Formatter와 FormattingConversionService
* 포매터는 스트링 타입의 폼 필드 정보와 컨트롤러 메소드 파라미터 사이에 양방향으로 적용할 수 있도록 두 개의 변환 메소드를 갖고 있다.
* Formatter는 Converter처럼 스프링이 기본적으로 지원하는 범용적인 타입 API가 아니다.
* 따라서 Formatter를 구현해 만든 타입 변환 오브젝트를 GenericConversionService에 직접 등록할 수 없다. 대신 Formatter 구현 오브젝트를 GenericConverter 타입으로 포장해서 등록해주는 기능을 가진 FormattingConversionService를 통해 적용 가능하다.

```java
String print(T object, Locale locale);
T parse(String text, Locale locale) throws ParseException;
```

* Formatter의 메소드에는 변화 메소드에 오브젝트나 문자열 뿐만 아니라 Locale 타입의 현재 지역정보도 함께 제공된다.
* 컨트롤러 내 바인딩과 타입 변환에 사용되도록 특화된 인터페이스이다.

FormattingConversionServiceFactoryBean에서 자동으로 등록되는 포맷터 두가지 : 

#### @NumberFormat
* 다양한 타입의 숫자 변환을 지원하는 포맷터
* 문자열로 표현된 숫자를 java.lang.Number 타입의 오브젝트로 상호 변환해준다. (Byte, Double, Float, Integer, Long, Short, BigInteger, BigDecimal)
* 엘리먼트로 style과 pattern을 지정할 수 있다 
	* style : NUMBER, CURRENCY, PERCENT (현재 지역정보 기준으로 표시 지원)
	* pattern : '#,##,##0.0#' 과 같은 식으로 숫자를 표현하는 패턴을 지정할 수 있다.


```java
class Product {
	@NumberFormat("$###,##0.00")  //$1000.23과 같이 표기가능
	BigDecimal price; 
}
```
* @NumberFormat 어노테이션을 사용하기 위해서는 FormattingConversionServiceFactoryBean을 빈으로 직접 등록해 컨트롤러에 DI받아 @InitBinder를 통해 바인더에게 지정해주어도 되고, ConfigurableWebBindingInitializer로 전체 컨트롤러에 일괄 적용도 가능하다.

#### @DateTimeFormat
* 날짜와 시간 정보 관리 라이브러리인 Joda Time을 이용하는 어노테이션 기반 포맷터인 @DateTimeFormat을 이용할 수 있다.
* JDK의 Date, Calendar, Long과 Joda 라이브러리의 LocalDate, LocalTime, LocalDateTime, DateTime이 있다.
* 스타일은 style 엘리먼트를 이용해 지정할 수 있으며, S(short), M(Medium), L(Long), F(Full) 네개의 문자를 날짜와 시간에 대해 각각 한 글자씩 사용해 두 개의 문자로 만들어 지정한다.
* 날짜와 시간 중 하나만 사용한다면, 다른 쪽은 '?'로 두면 된다.
* 디폴트는 가장 단순한 포맷인 SS이다.

```java

@DateTimeFormat(style="F-")	// 시간은 생략하고 날짜만 현재 지역정보에 따른 풀 포맷 이용
Calendar Birthday

@DateTimeFormat(pattern="yyyy/MM/dd")
Date orderDate;
```

***
### 바인딩 기술의 적용 우선순위와 활용 전략
* 스프링에서 모델 바인딩을 위한 타입 변환 기능은 매우 다양한 방법으로 작성하고 적용할 수 있다.
* 어떤 경우에 어떤 방식을 적용해야 할까?


#### 사용자 정의 타입의 바인딩을 위한 일괄 적용 : Converter
* 애플리케이션에서 정의한 타입이면서 모델에서 자주 활용되는 타입
* 서비스 계층의 빈을 활용해 변환 작업을 해야 하는 경우에도 사용

#### 필드와 메소드 파라미터, 애노테이션 등의 메타정보를 활용하는 조건부 변환 기능 : ConditionalGenericConverter
* 바인딩이 일어나는 필드와 메소드 파라미터 등 조건에 따라 변환을 할지 말지를 결정한다거나, 이런 조건을 변환 로직에 참고할 필요가 있을 경우
* 가장 유연하고 강력한 기능을 가졌지만, 그만큼 복잡하고 구현이 까다롭다.

#### 애노테이션 정보를 활용한 HTTP 요청과 모델 필드 바인딩 : AnnotationFormatterFactory와 Formatter
* @NumberFormat, @DateTimeFormat처럼 필드에 부여하는 애노테이션 정보를 이용해 변환 기능을 지원해야 하는 경우
* AnnotationFormatterFactory 구현 -> FormattingConversionService에 등록 -> ConditionalGenericConver로 변환

#### 특정 필드에만 적용되는 변환 기능 : PropertyEditor
* 프로퍼티 에디터는 지정된 이름을 가진 필드에 제한적으로 적용할 수 있다.
* 하나 이상의 프로퍼티 에디터 등록 작업을 여러 컨트롤러에 반복적으로 진행해야 한다면, PropertyEditorRegistarar를 사용하자.

#### 동일한 타입에 두 개 이상의 변환 기능이 중복될 경우?
* 커스텀 프로퍼티 에디터 -> 컨버전 서비스의 컨버터 -> 스프링 내장 디폴트 프로퍼티 에디터 순으로 적용된다.
* WebBindingInitializer로 일괄 적용한 컨버전 서비스나 프로퍼티 에디터는 @InitBinder에서 직접 등록한 프로퍼티 에디터나 컨버전 서비스보다 우선순위가 뒤진다.
* 컨버전 서비스는 단일 등록으로 @InitBinder에 직접 등록하면 WebBindingInitializer를 통해 등록된 공통 컨버전 서비스를 대체한다.