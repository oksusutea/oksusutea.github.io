---
layout: post
title: DispatcherServlet
categories: [Spring]
tags: [Spring Framework, Spring MVC]
description: 모든 요청을 가로채서 받는 DispatcherServlet
fullview: false
comments: true
---

`DispatcherServlet`을 사용하면 어떻게 애노테이션을 핸들러로 처리할 수 있는지에 대해 알아보도록 한다. 본 강좌에서는 디버거를 통해 코드를 타고타고 분석하는 방식으로 진행하였다.

### DispatcherServlet과 관련된 항목

* HandlerMapping : 핸들러를 찾아주는 인터페이스
* HandlerAdapter : 핸들러를 실행해주는 인터페이스
* HandlerExceptionResolver : 핸들러의 예외처리를 진행하는 인터페이스
* ViewResolver : 뷰이름을 실제 뷰로 변환하는 과정을 진행해주는 인터페이스


### DispatcherServlet의 동작 순서

#### 1. 요청을 분석한다.(로케일, 테마, 멀티파트등)
url 입력 후, 엔터를 누르면, `DispatcherServlet`의 `doService`메소드를 타면서 웹 서비스 요청을 처리하기 시작한다. `DispatcherServlet`에서 사용되는 몇몇의 정보를 `request`객체에 담은 후`doDispatch()`메소드를 실행한다.

```java
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
        this.logRequest(request);
        Map<String, Object> attributesSnapshot = null;
        if (WebUtils.isIncludeRequest(request)) {
            attributesSnapshot = new HashMap();
            Enumeration attrNames = request.getAttributeNames();

            label95:
            while(true) {
                String attrName;
                do {
                    if (!attrNames.hasMoreElements()) {
                        break label95;
                    }

                    attrName = (String)attrNames.nextElement();
                } while(!this.cleanupAfterInclude && !attrName.startsWith("org.springframework.web.servlet"));

                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }

        request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.getWebApplicationContext());
        request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
        request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
        request.setAttribute(THEME_SOURCE_ATTRIBUTE, this.getThemeSource());
        if (this.flashMapManager != null) {
            FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
            if (inputFlashMap != null) {
                request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
            }

            request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
            request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
        }

        try {
            this.doDispatch(request, response); //실질적인 웹 요청 처리를 진행하는 메소드
        } finally {
            if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted() && attributesSnapshot != null) {
                this.restoreAttributesAfterInclude(request, attributesSnapshot);
            }

        }

    }
```
그럼 `doDispatch`에 대해서 살펴보자. 

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            try {
                ModelAndView mv = null;
                Object dispatchException = null;

                try {
                    processedRequest = this.checkMultipart(request);
                    multipartRequestParsed = processedRequest != request;
                    mappedHandler = this.getHandler(processedRequest); // 그 다음 중요한 부분, 핸들러를 찾아온다.
                    if (mappedHandler == null) {
                        this.noHandlerFound(processedRequest, response);
                        return;
                    }

                    HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
                    String method = request.getMethod();
                    boolean isGet = "GET".equals(method);
                    if (isGet || "HEAD".equals(method)) {
                        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                        if ((new ServletWebRequest(request, response)).checkNotModified(lastModified) && isGet) {
                            return;
                        }
                    }

                    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                        return;
                    }

                    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }

                    this.applyDefaultViewName(processedRequest, mv);
                    mappedHandler.applyPostHandle(processedRequest, response, mv);
                } catch (Exception var20) {
                    dispatchException = var20;
                } catch (Throwable var21) {
                    dispatchException = new NestedServletException("Handler dispatch failed", var21);
                }

                this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
            } catch (Exception var22) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, var22);
            } catch (Throwable var23) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, new NestedServletException("Handler processing failed", var23));
            }

        } finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            } else if (multipartRequestParsed) {
                this.cleanupMultipart(processedRequest);
            }

        }
    }
```
이 메소드가 가장 핵심적인 부분인데, 우선 요청을 분석해서 이 요청이 멀티파트(파일업로드시 사용되는 타입) 요청인지, 로케일은 어떤건지, 테마는 어떤 테마인지에 대한 정보를 판단한다. 그 후, `mappedHandler = this.getHandler(processedRequest);` 이 라인을 통해 핸들러를 찾아온다. 요청을 처리 할 수 있는 핸들러를 찾아오는 것이다. 이 부분을 타고타고 들어가보자. 

#### 2.(핸들러 맵핑에게 위임하여) 요청을 처리할 핸들러를 찾는다.

```java
@Nullable
    protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        if (this.handlerMappings != null) {
            Iterator var2 = this.handlerMappings.iterator();

            while(var2.hasNext()) {
                HandlerMapping mapping = (HandlerMapping)var2.next();
                HandlerExecutionChain handler = mapping.getHandler(request);
                if (handler != null) {
                    return handler;
                }
            }
        }
```
`while`구문을 통해 `handlerMappings`을 순회한다는 것은, 여러개의 `HandlerMapping`이 존재한다는 말이다. 보통은 아무런 설정을 하지 않아도 `DispatcherServlet`이 자체적으로 생성하는 맵핑이 두가지 있는데, `BeanNameUrlHandlerMapping`과 `RequestMappingHandlerMapping`이다. 실질적으로 `RequestMappingHandlerMapping` 이 핸들러 맵핑이 애노테이션으로 등록하여 핸들러를 지정한 부분을 처리할 수 있도록 하는 핸들러맵핑이다. 

`HandlerExecutionChain handler = mapping.getHandler(request);` 이 라인에서 핸들러 맵핑이 이 요청(/app/hello등 URL을 요청)을 처리할 수 있는 핸들러를 찾아오게 된다. 결국 해당 요청을 처리할 수 있는 핸들러를 가져와`HandlerExecutionChain`로 반환한다.(여기서 요청에 해당되는 interceptor가 있다면 함께 반환한다)


#### 3. (등록되어 있는 핸들러 어댑터 중에) 해당 핸들러를 실행할 수 있는 핸들러 어댑터를 찾는다.
위 `doDispatch()`메소드에서 우리는 핸들러를 가져왔고, 이어서 `HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());` 라인을 통해 핸들러 어댑터를 찾게 된다.

```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
        if (this.handlerAdapters != null) {
            Iterator var2 = this.handlerAdapters.iterator();

            while(var2.hasNext()) {
                HandlerAdapter adapter = (HandlerAdapter)var2.next();
                if (adapter.supports(handler)) {
                    return adapter;
                }
            }
        }

        throw new ServletException("No adapter for handler [" + handler + "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
    }
```
`DispatcherServlet`은 핸들러만 여러개 있는 것 뿐만 아니라, 핸들러 어댑터도 여러개 가지고 있다. 여기서 우리가 찾아온 핸들러를 실행할 수 있는 핸들러 어댑터를 찾는 과정을 거친다. 여기서는 `RequestMappingHandlerAdapter`를 반환한다.

#### 4. 찾아낸 **핸들러 어댑터**를 사용하여 핸들러의 응답을 처리한다. 

`doDispatch()`메소드에서 `mv = ha.handle(processedRequest, response, mappedHandler.getHandler());`라인을 통해 핸들러의 응답을 처리하게 된다. 여기서 `handle`을 자세히 들여다보면 아래와 같다.

```java
@Nullable
    public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return this.handleInternal(request, response, (HandlerMethod)handler);
    }
```
위와 같이 `handle`메소드는 `handleInternal`을 실행하게 되고, `RequestMappingHandlerAdapter`의 `handleInternal`은 아래와 같다.

```java
protected ModelAndView handleInternal(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
        this.checkRequest(request);
        ModelAndView mav;
        if (this.synchronizeOnSession) {
            HttpSession session = request.getSession(false);
            if (session != null) {
                Object mutex = WebUtils.getSessionMutex(session);
                synchronized(mutex) {
                    mav = this.invokeHandlerMethod(request, response, handlerMethod);
                }
            } else {
                mav = this.invokeHandlerMethod(request, response, handlerMethod);
            }
        } else {
            mav = this.invokeHandlerMethod(request, response, handlerMethod);
        }

        if (!response.containsHeader("Cache-Control")) {
            if (this.getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
                this.applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
            } else {
                this.prepareResponse(response);
            }
        }

        return mav;
    }
```

여기서 `mav = this.invokeHandlerMethod(request, response, handlerMethod);`라인을 통해서 실제 핸들러 메소드를 invoke하게 된다. 즉, 자바의 리플렉션을 사용하여 컨트롤러의 핸들러를 실행하게 되는 것이다. `handlerMethod`객체 안에는 이미 어떤 메소드인지에 대한 정보를 가지고 있기 때문에, 자바 리플렉션을 통해서 메소드를 바로 실행할 수 있다. 
우리는 앞서 해당 요청에 해당되는 컨트롤러의 리턴 값을 `String`으로 하였기 때문에, `mav`의 리턴 값은 `String`일 것이다.

> `@RestController`애노테이션은 `@Controller`에 각 메소드에 `@ResponseBody`를 추가한 것과 동일하다.


#### 5.(부가적으로) 예외가 발생하였다면, 예외 처리 핸들러에 요청 처리를 위임한다.

#### 6. 핸들러의 리턴 값을 보고 어떻게 처리할지 판단한다.

```java
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType, ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {
        HandlerMethodReturnValueHandler handler = this.selectHandler(returnValue, returnType);
        if (handler == null) {
            throw new IllegalArgumentException("Unknown return value type: " + returnType.getParameterType().getName());
        } else {
            handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
        }
    }
```

위 메소드를 통해 핸들러의 리턴 값을 추가적으로 처리한다.

* 뷰 이름에 해당하는  뷰를 찾아서 모델 데이터를 렌더링한다
* `@ResponseBody`가 있다면 Converter를 사용하여 응답 본문을 만든다. 즉, HTTP Message의 본문에 내용을 추가한다.



#### 7.최종적으로 응답을 보낸다.



### 정리
Dispatcher의 동작원리를 정리해보면 아래와 같다. 

1. `doService`메소드로부터 요청을 분석한다. 로케일, 테마, 멀티파트등을 사용하는지 분석한 후 `doDispatch`메소드를 통해 실질적인 요청을 처리하는 과정을 시작한다.
2. 핸들러 맵핑에게 요청을 위임하여 요청을 처리할 핸들러를 찾는다.
	* 핸들러 처리 전 interceptor가 존재한다면, HandlerExecutionChain에 interceptor를 담아서 핸들러를 전달한다.
3. 해당 핸들러를 실행할 수 있는 핸들러 어댑터를 찾는다.
4. 찾아낸 핸들러 어댑터를 통해 핸들러의 응답을 처리한다.
	* interceptor의 postHandle 메소드가 실행된다.
5. 핸들러의 리턴 값을 보고 어떻게 처리할지 판단한다.
6. 최종적으로 응답을 보낸다. 


***
참고자료

1. [스프링 웹 MVC](https://inf.run/dJFi)