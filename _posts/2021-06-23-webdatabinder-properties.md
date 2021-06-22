---
layout: post
title: WebDataBinder 설정 항목
categories: [spring]
tags: [java, spring, 토비의 스프링]
description: WebDataBinder 바인딩 옵션
fullview: false
comments: true
---

* WebDataBinder는 HTTP 요청정보를 컨트롤러 메소드의 파라미터나 모델에 바인딩할 때 사용되는 바인딩 오브젝트다.
* 커스텀 프로퍼티 에디터나 컨버전 서비스 외에 여러가지 유용한 바인딩 옵션을 지정할 수 있다.


### allowedFields, disallowedFields
* 스프링은 WebDataBinder 안에 바인딩이 허용된 필드 목록을 넣을 수 있는 allowedFields와 금지 필드 목록을 넣을 수 있는 disallowedFields 프로퍼티를 설정할 수 있게 해준다.
* @InitBinder나 WebBindingInitializer에서 WebDataBinder 오브젝트의 프로퍼티를 설정해줌으로써 적용할 수 있다.

```java
@InitBinder
public void initBinder(WebDataBinder dataBinder) {
	dataBinder.setAllowedFields("name", "email", "tel");
}
```

* 해당 컨트롤러의 모든 @ModelAttribute 바인딩에서 name, email, tel 세가지 HTTP 요청 파라미터 외에는 사용하지 못하도록 허용된 필드를 지정하는 코드이다.
* 이 세개 필드 외 다른 이름을 가진 HTTP 파라미터가 오면 모델 오브젝트의 바인딩에서 제외된다.

### requiredFields
* 컨트롤러가 자신이 필요로 하는 필드 정보가 폼에서 모두 전달됐는지를 확인하고 싶을 때 사용한다.
* 필요하다면 필수 HTTP 파라미터를 WebDataBinder에게 알려주고, 바인딩 시 필우 파라미터에서 빠진게 있다면 바인딩 에러로 처리하도록 만들어 줄 수 있다.


### fieldMarkerPrefix
* 이 프로퍼티 값을 다르게 설정해서 사용하기 위해서라기 보다는, 어떤 방식으로 사용되는지 이해하는 것이 더 중요하다.
* 만일 어떠한 프로퍼티의 체크박스를 아무것도 선택하지 않은 상태로 submit하게 되면, 해당 프로퍼티는 HTTP요청으로 들어오지 않을 것이다.
* 유일한 해결책은 바인더에게 특정 필드를 체크박스라고 알려줘서, 만약 해당 필드의 이름을 가진 HTTP 요청 파라미터가 전달되지 않으면, 이는 체크박스를 해제했기 때문이라고 판단할 수 있게 해줘야 한다.
* 이를 위해 스프링은 특별한 접두어가 붙은 필드 이름을 가진 마커 히든 필드를 추가하는 방식을 사용한다.

```html
<input type="checkbox" name="autoLogin"	/>
<input type="hidden" name="_autoLogin"	value="on"	/>
```

* 체크박스를 넣을 때, 이렇게 히든 필드를 하나 추가하고, 체크박스의 필드 이름 앞에 밑줄(_)을 붙인 것을 히든 필드 이름으로 넣는다.  체크박스 필드의 이름 앞에 붙는 이런 밑줄과 같은 것을 필드마커라고 부른다.
* 스프링은 필드마커가 있는 HTTP 파라미터를 발견하면, 필드마커를 제외한 이름의 필드가 폼에 존재한다고 생각한다. 그런데 그 필드 이름의 파라미터가 요청정보에 존재하지 않는다면, 이는 체크박스를 해제했기 때문이라고 판단하고 그 이름에 해당하는 프로퍼티 값을 리셋해준다.
* 필드마커로 사용되는, 히든 필드의 이름 앞에 붙이는 접두어를 지정한다.
* 기본 값은 밑줄 하나이나, 다른 필드마커를 사용하고 싶다면 WebDataBinder의 setFieldMarker() 메소드를 이용해 변경해주면 된다.

### fieldDefaultPrefix
* WebDataBinder의 프로퍼티인 fieldDefaultPrefix의 디폴트 값은 느낌표(!)이다.
* 보통 필드 디폴트는 히든 필드를 이용해 체크박스에 대한 디폴트 값을 지정하는데 사용한다.
* WebDataBinder의 fieldDefaultPrefix를 이용하면 기본으로 설정된 느낌표 대신 다른 접두어를 지정할 수 있다.
	* 느낌표로 시작하는 필드 이름은 사용될 일이 없으므로 특별한 이유가 없다면 변경하지 않는 것이 좋다. 대신, 필드 디폴트를 이용해 체크박스의 디폴트 값을 설정해주는 방법은 잘 기억해두자.

```html
<input type="hidden" name"!type" value="member"	/>
```
* 위와 같이 작성하면, 체크박스를 하나도 체크하지 않았을 경우 !type라는 필드 디폴트를 확인하고, type라는 이름의 파라미터가 존재하지 않는다면 필드 디폴트의 value 값에 담긴 내용을 해당 프로퍼티에 바인딩해준다.