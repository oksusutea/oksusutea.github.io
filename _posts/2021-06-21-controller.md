---
layout: post
title: @Controller
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: 컨트롤러 어노테이션
fullview: false
comments: true
---

* DefaultAnnotationHandlerMapping은 사용자 요청을 @RequestMapping 정보를 활용해서 컨트롤러 빈의 메소드에 매핑해준다.
* AnnotationMethodHandlerAdapter는 매핑된 메소드를 실제로 호출하는 역할을 담당한다.
* 컨트롤러가 실행되는 과정 : 
	* DispatcherServlet으로부터 HttpServletRequest, HttpServletResponse를 받아 이를 컨트롤러가 사용하는 파라미터 타입으로 변환해서 제공한다.
	* 컨트롤러로부터 받은 결과를 ModelAndView 타입의 오브젝트에 담아 DispatcherServlet에게 돌려준다.

	
## 메소드 파라미터의 종류

* AnnotationMethodHandlerAdapter가 허용하는 핸들러 메소드 파라미터의 종류와 사용 방법, 그리고 리턴 값에 대해 알아본다.

### HttpServletRequest, HttpServletResponse

* 서블릿의 기본 파라미터를 그대로 사용할 수 있다.

### HttpSession
* HttpServletRequest를 통해서 가져올 수 있지만, 세션만 필요한 경우 사용하는 파라미터이다.
* 서버에 따라 멀티스레드 환경에서 안전성이 보장되지 않다.

### WebRequest, NativeWebRequest
* HttpServletRequest의 요청정보를 대부분 그대로 갖고있는, 서블릿 API에 종속적이지 않은 오브젝트 타입이다.
* 원래 서블릿과 포틀릿 환경 양쪽에 모두 적용가능한 범용적인 핸들러 인터셉터를 만들 때 활용하기 위해 만들어졌다.
* 스프링 서블릿/MVC의 컨트롤러에 꼭 필요한 것은 아니다.
* NativeWebRequest는 WebRequest에서 감춰진 HttpServletRequest와 같은 환경종속적인 오브젝트를 가져올 수 있는 메소드가 추가되어 있다.

### Locale
* java.util.Locale 타입으로 DispatcherServlet의 지역정보 리졸버가 결정한 Locale 오브젝트를 받을 수 있다.

### InputStream, Reader
* HttpServletRequest의 getInputStream()을 통해 받을 수 있는 콘텐트 스트림 또는 Reader 타입의 오브젝트를 받을 수 있다.

### OutputStream, Writer
* HttpServletRequest의 getOutputStream()을 통해 받을 수 있는 출력용 콘텐트 스트림 또는 Writer 타입의 오브젝트를 받을 수 있다.

### @PathVariable
* @RequestMapping의 URL에 {}로 들어가는 패스 변수를 받는다.
* 요청 파라미터를 URL의 쿼리 스트링으로 보내는 대신 URL path로 풀어서 쓰는 경우 매우 유용하다.
* 패스 변수는 하나의 URI 템플릿 안에 여러 개를 선언할 수 있다.
* path variable의 변수 타입이 URL상 들어오는 파라미터 타입과 불일치할 경우, **400 - Bad Request** 응답 코드가 전달된다.


```java
@RequestMapping("/member/{membercoede}/order/{orderid}")
public String lookup(@PathVariable("membercode") String code, @PathVariable("orderid") int orderid) {
	....
}
```

### @RequestParam
* 단일 HTTP 요청 파라미터를 메소드 파라미터에 넣어주는 어노테이션이다.
* 가져올 요청 파라미터의 이름을 @RequestParam 애노테이션의 기본 값으로 지정해주면 된다.
* 요청 파라미터의 값은 메소드 파라미터의 타입에 따라 적절하게 변환된다. 
* 하나 이상의 파라미터에 적용할 수 있고, 전체를 갖고오고 싶다면 @RequestParam Map<String, String> params로 선언하면 된다. 파라미터 이름은 맵의 키에, 값은 맵의 값에 담겨 전달된다.

```java

public String view(@RequestParam("id") int id, @RequestParam("name") String name, @RequestParam("file") MultipartFile file) { ... }
```

* @RequestParam을 사용했다면 해당 파라미터가 반드시 있어야 한다. 없다면 **HTTP 400 - Bad Request**를 받게 될 것이다. 파라미터를 필수가 아니라 선택적으로 제공하게 하려면, required 엘리먼트를 false로 설정해주면 된다.
* String, int와 같은 단순 타입인 경우 @RequestParam을 아예 생략할 수 있다. 하지만 명시적으로 표기하기 위해 @RequestParam을 부여해주는 것을 권장한다.

### @CookieValue
* HTTP 요청과 함께 전달된 쿠키 값을 메소드 파라미터에 넣어주도록 @CookieValue를 사용할 수 있다. 애노테이션의 기본 값에 쿠키 이름을 지정해주면 된다.
* @RequestParam과 마찬가지로 쿠키 값이 반드시 존재해야 한다.

```java
public String check(@CookidValue(value="auth", required=false, defaultValue="NONE" String auth) { ... }
```

### @RequestHeader
* 요청 헤더정보를 메소드 파라미터에 넣어주는 어노테이션이다. 애노테이션의 기본 값으로 가져올 HTTP 헤더의 이름을 지정한다.
* 헤더 값이 반드시 존재해야 한다. 필수 값이 아닌 값으로 설정해주려면 required, defaultValue를 설정해주자.


```java
public void header(@RequestHeader("Host") String host,
				@RequestHeader("Keep-Alive") long keepAlive)
```

### Map, Model, ModelMap
* 모델 정보를 담는데 사용할 수 있는 오브젝트이다.
* addAttribute()를 제공하기 때문에, 이름을 지정해 오브젝트 값을 넣을 수 있다.
* 자동이름 생성기능을 이용해 오브젝트만 넣을 수 있다. 이 때는 오브젝트의 타입정보를 참고하여 모델 이름을 자동으로 부여한다.

```java
@RequestMapping(...)
public void hello(ModelMap model) {
	User user = new User(1, "Spring");
	model.addAttribute(user);
}
```

### @ModelAttribute
* 메소드 파라미터나 메소드 레벨에 적용할 수 있다.
* 클라이언트로부터 컨트롤러가 받는 요청정보 중, 하나 이상 값을 가진 오브젝트 형태로 만들 수 있는 구조적인 정보를 @ModelAttribute 모델이라고 부른다.
* 컨트롤러가 전달받는 오브젝트 형태 정보를 가르킨다.
* 요청 파라미터를 메소드 파라미터에서 1:1로 받으면 @RequestParam이고, 도메인 오브젝트나 DTO의 프로퍼티에 요청 파라미터를 바인딩해서 한 번에 받으면 @ModelAttribute라고 볼 수 있다.
* 하나의 오브젝트에 클라이언트의 요청정보를 담아 한 번에 전달되는 것이기 때문에 이를 커맨드 패턴에서 말하는 커맨드 오브젝트라고 부르기도 한다.
* 주로 URL의 쿼리 스트링으로 들어오는, 검색 조건과 같은 정보를 @ModelAttribute가 붙은 파라미터 타입의 오브젝트에 모두 담아 전달해준다.

@RequestParam 버전 : 

```java
@RequestMapping("/user/search")
public String search(@RequstParam int id, @Request String name, @RequestParma int level, @RequestParam String email, Model model) {
	List<User> list = userService.search(id, name, level, email);
}
```

@ModelAttribute 버전 : 

```java
@RequestMapping("/user/search")
public String search(@ModelAttribute Usersearch userSearch) {
	List<User> list = userService.search(userSearch);
}
```

* @ModelAttribute는 @RequestParam과 마찬가지로 생략 가능하다.
* String, int와 같은 단순 타입은 @RequestParam으로 간주하고, 그 외 복잡한 오브젝트는 @ModelAttribute가 생략됐다고 간주한다.
* @ModelAttribute는 컨트롤러가 리턴하는 모델에 파라미터로 전달한 오브젝트를 자동으로 추가해준다.

### Errors, BindingResult
* @RequestParam은 파라미터의 타입과 받은 값이 일치하지 않을 경우 400 에러를 발생시키지만, @ModelAttribute는 예외를 발생하지 않는다.
* 요청 파라미터의 타입이 모델 오브젝트의 프로퍼티 타입과 일치하는지를 포함한 다양한 방식의 검증 기능을 수행하기 때문이다.
* @ModelAttribute를 통해 폼의 정보를 전달받을 때에는 Errors, BindingResult 타입의 파라미터를 함께 사용해야 한다. 이 파라미터를 사용하지 않으면 스프링은 요청 파라미터의 값에 문제가 없도록 애플리케이션이 보장해준다고 생각한다.
* 타입이 일치하지 않으면 BindException 예외가 던져진다.

```java
@RequestMapping(value="add", method=RequestMethod.POST)
public String add(@ModelAttribute User user, BindingResult bindingResult) { .. }
```

* BingdingResult, Errors는 파라미터 바인딩시 발생한 변환 오류와 모델 검증기를 통해 검증하는 도중 오류가 저장된다.
* 이 파라미터는 @ModelAttribute 파라미터 뒤에 나와야 한다.


### SessionStatus
* 컨트롤러가 제공하는 기능 중, 모델 오브젝트를 세션에 저장했다가 다음 페이지에서 다시 활용하게 해주는 기능이 있다.
* 더이상 세션 내 모델 오브젝트에서 저장할 필요가 없을 경우 코드에서 직접 완료 메소드를 호출 해 세션 안에 저장된 오브젝트를 제거해주어야 한다. 이 때 필요한것이 SessionStatus 오브젝트이다.

### @RequestBody
* HTTP 요청의 본문부분이 그대로 전달된다.
* 일반적인 GET/POST는 거의 사용할 일이 없지만, XML 혹은 JSON 기반의 메시지를 사용하는 요청의 경우 유용하다.
* 이 어노테이션이 붙은 파라미터가 있으면 HTTP 요청의 미디어 타입과 파라미터 타입을 확인한다. 메시지 변환기 중 해당 미디어 타입과 파라미터 타입을 처리할 수 있는 것이 있다면, 요청의 본문을 통째로 변환해 지정된 메소드 파라미터로 전달한다.
* String : StringHttpMessageConverter
* XML : MarshallingHttpMessageConverter
* JSON : MappingJacksonHttpMessageConverter

### @Value
* 빈의 값 주입에서 사용하던 @Value 애노테이션도 컨트롤러 메소드 파라미터에 부여할 수 있다.
* 주로 시스템 프로퍼티나 다른 빈의 프로퍼티 값, 또는 더 복잡한 SpEL을 이용해 클래스의 상수를 읽어오거나 특정 메소드를 호출한 결과 값, 조건식 등을 넣을 수 있다.
* 필드, 초기화 메소드 파라미터에 사용할 수 있다.

```java
public class HelloController {
	@Value("#{systemProperties['os.name']}") String osName
	
	@RequestMapping(...)
	public String hello() {
		String osName = this.osName;
	}
}
```

### @Valid
* JSR-303의 빈 검증기를 이용해 모델 오브젝트를 검증하도록 지시하는 지시자다.
* @ModelAttribute와 자주 사용한다.


***

## 리턴 타입의 종류
* 컨트롤러가 DispatcherServlet에 전달해야 하는 정보는 모델과 뷰다.
* 그렇기 때문에 메소드의 리턴 타입을 결정할 때는 ModelAndView에 들어갈 정보로 어떤 것이 있는지 고려해보아야 한다.

### 자동 추가 모델 오브젝트와 자동생성 뷰 이름

#### @ModelAttribute 모델 오브젝트 또는 커맨드 오브젝트
* 메소드 파라미터 중 @ModelAttribute를 붙인 모델 오브젝트나 커맨트 오브젝트로 처리되는 오브젝트는 자동으로 컨트롤러가 리턴하는 모델에 추가된다.

```java
ppublic void add(@ModelAttribute("user") User user)
public void add(@ModelAttibute User user)
public void add(User user)
```

* 위 메소드는 모드 user라는 이름으로 user 파라미터 오브젝트가 모델에 추가되게 해준다.

#### Map, Model, ModelMap 파라미터
* 컨트롤러 메소드에 Map, Model, ModelMap 타입의 파라미터를 사용하면 미리 생성된 모델 맵 오브젝트를 전달받아 오브젝트를 추가할 수 있다.

#### @ModelAttribute 메소드
* @ModelAttribute는 컨트롤러 클래스의 일반 메소드에도 부여할 수 있다.
* @ModelAttribute가 붙은 메소드는 컨트롤러 글래스 안에 정의하지만 컨트롤러 기능을 담당하지 않는다.
* @ModelAttribute 메소드가 생성하는 오브젝트는 클래스 내 다른 컨트롤러 메소드의 모델에 자동으로 추가된다.
* 보통 폼이나 검색조건 페이지 등에 사용되는 참조정보를 조회할 때 많이 쓰인다.


```java
@ModelAttribue("codes")
public List<Code> codes() {
	return codeService.getAllcodes();
}
```

#### BindingResult
* @ModelAttribute와 함께 사용하는 BindingResult 타입의 오브젝트도 모델에 자동으로 추가된다.
* 모델 맵에 추가될 때의 키는 `org.springframework.validation.BindingResult.모델이름`이다.
* JSP, 프리마커등 뷰에 사용되는 커스텀 태그나 매크로에서 사용하기 위해 쓰인다.


### 메소드 리턴 타입의 종류

#### ModelAndView
* 컨트롤러가 리턴해야하는 정보를 담고 있는 가장 대표적인 타입
* 하지만 직접적으로 ModelAndView를 리턴하는 것보다는 편리한 방법이 많아 자주 쓰이지 않는다.


#### String
* 메소드의 리턴 값이 String이면 이 리턴 값은 뷰 이름으로 사용된다.
* 모델 정보는 모델 맵 파라미터로 가져와 추가하는 방법을 사용해야 한다. (Model or ModelMap을 파라미터로 추가해 모델 정보를 넣어주기)

```java
@RequestMapping("/hello")
public String hello(@RequestParam String name, Model model) {
	model.addAttribute("name", name);
	return "hello";
}
```

#### void
* void 값을 리턴 값으로 쓸 경우, RequestToViewNameResolver가 전략을 통해 자동생성되는 뷰 이름이 사용된다.
* 보통 prefix/suffix를 붙여줘서 뷰 이름을 생성한다.

### 모델 오브젝트
* 뷰 이름은 RequestToViewNameResolver를 통해 자동생성하고, 코드를 이용해 모델 오브젝트에 추가할 오브젝트가 하나 뿐이라면, Model 파라미터를 받아 저장하는 대신 모델 오브젝트를 바로 리턴해도 된다.
* 스프링은 리턴 타입이 지정 타입이나 void가 아닐 경우, 이를 모델 오브젝트로 인식해서 모델에 자동으로 추가해준다.

```java
@RequestMapping("/view")
public User view(@RequestParam int id) {
	return userService.getUser(id);
}
```

### Map/Model/ModelMap
* 메소드의 코드에서 Map이나 Model, ModelMap 타입의 오브젝트를 직접 만들어 리턴해주면 이 오브젝트는 모델로 사용된다.
* 컨트롤러 코드에서 필요한 모델 맵은 파라미터로 받아서 사용하면 편리하다.
* 단일 모델 오브젝트를 직접 리턴하는 방식은 지양하자(Map)

```java
@RequestMapping("/view")
public Map view(@RequestParam int id) {
	return userService.getUserMap(id);
}
```

* 위 코드는 map이라는 이름을 가진 맵 오브젝트가 모델에 추가되지 않는다. Map 타입의 리턴 값은 그 자체로 모델 맵으로 인식해 엔트리 하나를 개별적인 모델로 다시 등록해버리기 때문이다.

### View

* 스프링 리턴 타입은 뷰 이름으로 인식한다.
* 뷰 이름대신 뷰 오브젝트를 사용하고 싶다면 리턴타입을 View로 선언하고 뷰 오브젝트를 넘겨주면 된다.

### @ResponseBody
* 메소드가 리턴하는 오브젝트는 뷰를 통해 결과를 만들어내는 모델로 사용되는 대신, 메시지 컨버터를 통해 바로 HTTP 응답의 메시지 본문으로 전환된다.
* @ResponseBody가 적용된 컨트롤러는 리턴 값이 단일 모델 오브젝트이고, 메시지 컨버터가 뷰와 같은 방식으로 동작한다고 이해할 수 있다.


***

## @SessionAttributes와 SessionStatus

사용자가 게시글 수정을 요청했을 때, 읽기 전용 값을 반영하기 위한 방법

### 히든 필드
* 가장 지양해야 할 방법이다.
* HTML 소스를 열어보면 그 내용과 필드 이름까지 쉽게 알아낼 수 있고, 조작이 쉽기 때문이다.

### DB 재조회
* 폼으로부터 수정된 정보를 받아 User 오브젝트에 넣어줄 쌔, 빈 User 오브젝트 대신 DB에서 다시 읽어온 User 오브젝트를 이용하는 것이다.
* 이 방법은 폼을 submit 할 때마다 DB에서 사용자 정보를 다시 읽는 부담이 있다.
* 복사할 필드를 잘못 선택하거나 빼먹으면 문제가 발생할 수 있다.


### 계층 사이 강한 결합
* 강한 결합이라는 의미는 각 계층의 코드가 다른 계층에서 어떤 일이 일어나고 어떤 작업을 하는지를 자세히 알고, 그에 대응해 동작하도록 하는 방식이다.
* 계층간 강한 의존성을 주고 결합도를 높이면 처음에는 만들기 쉽지만, 화면을 중심으로 작성해 중복되는 코드가 많아진다.
* 이럴 바에는 DTO로 불러오는 것 보다 Map으로 가져오는 것이 더 낫다.

### @SessionAttributes
* 스프링의 해결책은 세션을 이용하는 것이다.
* 컨트롤러 메소드가 생성하는 모델 정보중, @SessionAttributes에 지정한 이름과 동일한 것이 있다면 이를 세션에 저장해준다.
* @ModelAttributes가 지정된 파라미터가 있을 때 이 파라미터에 전달해줄 오브젝트를 세션에서 가져올 수 있다.


```java
@Controller
@SessionAttributes("user")
public class UserController {
	....
	@RequestMapping(value="/user/edit", method=RequestMethod.GET)
	public String form(@RequestParam int id, Model model) {
		model.addAttribute("user", userService.getUser(id));
		return "user/edit";
	}
	
	@RequestMapping(value="/user/edit", method=RequestMethod.POST)
	public String submit(@ModelAttribute User user) {
		...
	}
}
```
* form() 메소드에서는 user라는 이름으로 DB에서 가져온 User 오브젝트를 모델에 추가한다.
* submit()에서는 세션에 저장된 User 오브젝트를 가져와 폼에서 전송해준 파라미터만 바인딩한 뒤, 컨트롤러의 user 파라미터로 넘겨준다.
* @SessionAttributes는 하나 이상의 모델을 세션에 저장하도록 지정할 수 있다. @SessionAttributes의 설정은 클래스의 모든 메소드에 적용된다.
* 컨트롤러 메소드에 의해 생성되는 모든 종류의 모델 오브젝트는 @SessionAttributes에 저잘될 이름을 갖고 있는지 확인한다.
* 마찬가지로 @ModelAttributes로 지정된 파라미터와 @ModelAttributes가 생략됐지만 스프링이 모델로 인식하는 빈 오브젝트 등은 모두 @SessionAttributes에 의해 저장된 세션 값을 가져와 사용할지 확인하는 대상이다.
* @SessionAttributes의 기본 구현인 HTTP 세션을 이용한 세션 저장소는 모델 이름을 세션에 저장할 애트리뷰트 이름으로 사용한다.

### SessionStatus
* @SessionAttributes를 더이상 사용하지 않을 경우 코드로 제거해주어야 한다는 점을 잊지 말자.
* 이를 제거하는 책임은 컨트롤러에게 있다.
* 보통은 폼의 작업을 마무리 하는 코드에서 SessionStatus의 setComplete() 메소드를 호출해 세션에 저장해뒀던 오브젝트를 제거해줘야 한다.


```java
@RequestMapping(value="/user/edit", method=RequestMethod.POST)
	public String submit(@ModelAttribute User user, SessionStatus sessionStatus) {
		this.userService.updateUser(user);
		sessionStatus.setComplete();	// 현재 컨트롤러에 의해 세션에 저장된 정보를 모두 제거해준다.
		return "user/editsuccess";
	}
```

***
참고자료 :  
[토비의 스프링 3.1 Vol.2 4.2 - @Controller](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=19505671)