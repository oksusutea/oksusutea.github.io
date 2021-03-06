---
layout: post
title: 사이드 프로젝트 - 1. DB설계
categories: [Java, Spring Boot, Database]
tags: [Java, Side Project]
description: 사이드프로젝트 DB설계 관련 포스트
fullview: false
comments: true
---

업무적으로 프로젝트를 수행할 때에는  
* 고객 요구사항 분석 -> 공수 산정 -> 화면 설계 -> DB 설계 -> 개발 <-> 단위 테스트 -> 통합 테스트 -> 론칭  
의 순서로 프로젝트를 수행하였었다.  지금 개인적으로 하고 있는 프로젝트는, 우선 요구사항 (필요한 기능)을 대략적으로 잡아놓은 상태이고, 화면 설계는 일반 강의 플랫폼을 참고하여 구성할 계획이기에 바로 DB 설계 단으로 넘어왔다.(물론 설계 후 틈틈히 화면 설계를 하겠지만!)  
가장 중요한 부분이기에, 어느 DB를 사용할지, 어떤 erd 툴을 사용할지 부터 고민에 빠지기 시작했다ㅎ_ㅎ  
하지만 언제까지나 고민만 하고 있을 수는 없는 법, 찬찬히 찾아보고 하나하나씩 해결해나가기로 했다. 


## 어떤 DB를 사용할 것인가?
RDBMS vs no SQL

Oracle DB vs MySQL vs Postgre DB

## erd 툴은?
부끄러운 이야기이지만, 업무적으로는 erd 툴을 다루어 본 적이 없었다.(소규모 플젝은 그냥 DB만 설계하고, 대규모 플젝에 잠깐 발담가 봤을 때는 exerd를 살짝 접해봤었다.)   
하지만 개인적으로 하고 있는 프로젝트이기에.. 가장 편리하고, 손쉽게 쓸 수 있는 툴이 어떤 것이 있을까 찾아보던 와중에 좋은 사이트를 발견했다.  
* 사이트 주소 : [http://erdclud.com]()  
웹 기반으로 설치가 필요 없고, 팀원들과 실시간으로 동시 작업할 수 있는 클라우드 서비스까지 제공해준다고 한다.(물론 나는 개인플젝으로 하고 있지만)  


## DB 설계
 강의글을 크롤링해서 주로 조회용으로 쓰이기 때문에, 도메인은 크게  
 1) 강의(게시글)  
 2) 플랫폼(플랫폼 정보 뿌려주기)  
 3) 카테고리  
 로 생각해 보았다.  
 여기서 스프링 시큐리티도 사용하고 싶어 사용자 로그인/회원가입쪽도 추가 할 예정이다.





***

앞서 작성한 내용은 추후 개발되는 내용에 따라 점차 변화가 생길 수 있으며,   
바뀌는 내용에 따라 함께 업데이트 해 줄 예정이다.  
반영해야 할 내용 :   
1.  DB 선정 사유  
2. Cloud Erd 사용법