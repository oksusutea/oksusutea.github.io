---
layout: post
title: 스프링에서 캐싱을 가능하게 해주는 EnableCaching
wcategories: [Spring Framework]
tags: [Java, Spring Framework]
description: 
fullview: false
comments: true
---

### 캐시를 사용하는 이유
* 자주 사용하는 데이터를 매번 DB에서 조회하기에는 비용이 많이 발생
* 캐시도 일종의 데이터이기 때문에 개인화되지 않는 데이터, 특정 유저에 종속되지 않는 데이터를 미리 저장해두는 것이 효율적이다


### 캐시 등록을 위한 설정
XML태그와 어노테이션을 통해 스프링에서 캐싱을 가능하게 할 수 있는 방법이 있다. 
* `@EnableCaching` 을 Configuration 파일에 등록하거나
*  `<cache:annotation-driven/>` 을 태그로 등록
하게 되면 캐싱할 때 필요한 필수 컴포넌트들(캐시매니저 etc)이 자동으로 등록된다.
* 등록시 CacheManager는 필수로 등록해주어야 한다.
	* SimpleCacheManager, EhCacheCacheManager, RedisCacheManager, ConcurrentMapCacheManager, CompositeCacheManager(여러 공간에 캐시를 둘 수 있다) 등이 있다.

### 캐시 등록

```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Import(CachingConfigurationSelector.class)  
public @interface EnableCaching {
```

이렇게 Configuration 파일에 @EnableCaching을 붙여주면, 이제 어느 메소드에서든지 캐싱을 진행할 수 있다. 내가 캐싱하고자 하는 메소드에 @Cacheable이라는 어노테이션을 붙여주면 된다.

```java
@Cacheable("books")
public Book findBook(ISBN isbn) {...}
```
* 위와 같이 붙여주면, 이 데이터는 캐시할 수 있는 데이터다 라고 명시하게 되는 것이다.
* 해당 메서드 호출 시점에, 캐시가 있으면 가져가고, 없으면 메소드를 실행시켜 결과를 캐시에 담는다.

#### 캐시를 찾는 과정
* 요청이 온다
* 캐시에 해당 데이터가 있는지 찾는다
	* 있을 경우 해당 캐시 값을 바로 가져온다
* 없을 경우 요청을 처리한 후 리턴한다
* 리턴한 값을 캐시에 저장한다

### 캐시 업데이트
```java
@CachePut("books")
public Book findBook(ISBN isbn) {...}
```
* 메서드 실행에 영향을 끼치지 않고 캐시를 계속 갱신해야 하는 경우 @CachePut을 사용한다.
* 즉 매번 캐시 값을 업데이트 해야 할 경우 사용한다.

#### @Cacheable vs @CachePut
* @Cacheable은 캐시를 사용해 메서드 실행을 생략한다.
* @CachePut은 캐시 존재 여부와 상관없이 매번 메소드를 실행시킨다.

### 캐시 제거
```java
@CacheEvict("books")
public Book findBook(ISBN isbn) {...}
```
* @CacheEvict는 특정 캐시를 제거할 때 사용된다.
* @CacheEvict는 void 메소드에 붙여서 사용할 수 없다(반환한 값을 캐시에서 제거해야 하기 때문에)


#### 어노테이션의 동작 원리
* AOP 기반으로 @EnableCaching이라는 것을 통해 캐싱을 활성화 시키면 캐시와 관련된 Configuration을 등록해 동작할 수 있도록 해준다.

```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Import(CachingConfigurationSelector.class)  
public @interface EnableCaching {
  
   boolean proxyTargetClass() default false;  
   AdviceMode mode() default AdviceMode.PROXY;
```

* @EnableCaching에서 default로 프록시로 어드바이스를 생성하게 되어 있다.

```java
public class CachingConfigurationSelector extends AdviceModeImportSelector<EnableCaching> {   
  ...
  
 static {  
  ClassLoader classLoader = CachingConfigurationSelector.class.getClassLoader();  
  jsr107Present = ClassUtils.isPresent("javax.cache.Cache", classLoader);  
  jcacheImplPresent = ClassUtils.isPresent(PROXY_JCACHE_CONFIGURATION_CLASS, classLoader);  
  }  
  
  @Override  
  public String[] selectImports(AdviceMode adviceMode) {  
  switch (adviceMode) {  
	  case PROXY:  
            return getProxyImports();  
	 case ASPECTJ:  
            return getAspectJImports();  
	 default:  
            return null;  
  }  
 }  
 
  private String[] getProxyImports() {  
	  List<String> result = new ArrayList<>(3);  
	  result.add(AutoProxyRegistrar.class.getName());  
	  result.add(ProxyCachingConfiguration.class.getName());  
	 if (jsr107Present && jcacheImplPresent) {  
	  result.add(PROXY_JCACHE_CONFIGURATION_CLASS);  
	  }  
	  return StringUtils.toStringArray(result);  
  }
  ...
}
```

* `CachingConfigurationSelector` 에서는  모드에 따라 임포트살 내트를 할 지 결정한다.
* `@EnableCaching`에서는 Proxy가 default 값으로 되어 있어, getProxyImports()를 실행하게 된다.
* 이 과정중에 자동으로 프록시로 등록해줄 수 있는 `AutoProxyRegistrar` 와, 프록시 기반 캐싱을 가능케 하기 위해 필수로 등록해야하는 인프라 빈을 설정해주는 `ProxyCachingConfiguration` 을 임포트 대상으로 등록해준다.
* 그 외에 Jcache와 관련된 설정파일이 있다면 그것도 같이 import 할 수 있도록 해준다.

#### ~~Selector?
* 어떤 규약 조건에 따라 다른 Configuration을 적용할 수 있도록 하는 설정 파일
