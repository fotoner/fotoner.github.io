---
title: "Vue를 Tomcat에 배포하기"
last_modified_at: 2022-11-27T23:00:00
categories: 
 - Vue
tags:
 - Vue
 - SPA
 - Tomcat
---

오늘은 SPA 프레임워크, 특히 Vue를 Tomcat에 배포하는 과정을 정리해보았다.

(vue를 기준으로 작성하기는 하지만 다른 SPA앱에서도 동일하게 적용된다)

![image](/assets/images/posts/vue/03.png)

우선 프로젝트를 빌드하면 다음과 같은 파일들이 생길텐데 이걸 톰캣 webapp에 넣어 보았다.

![image](/assets/images/posts/vue/04.png)

그러면 다음과 같이 호스팅이 이루어지고 위의 네비게이션 링크도 클릭했을때 정상적으로 링킹 되는 것 처럼 보인다. 하지만...

![image](/assets/images/posts/vue/05.png)

외부에서 링크를 다이렉트로 접속하거나 주소를 직접 입력했을때 다음과 같이 404를 뿜어낸다. 이는 Vue를 비롯한 SPA의 근본적인 특징에서 발생되는 오류이다.

SPA는 하나의 페이지 만을 불러오고 다른 페이지로 이동할 시에 SPA 라우터 상에서 URL-Path에 맞게 적절한 컴포넌트를 다시 렌더링하여 출력하게된다. 즉 서버로부터 별도로 페이지를 받지 않는다. 

따라서 위의 이미지와 같이 실제 SPA앱에서는 단 하나의 index.html 파일만 존재하는데 당연히 사이트의 루트로 접속시에는 해당 html파일이 로드 되지만 다른 path로 직접 접속하면 이 html 파일을 받지 못하고 404를 뿜어낸다. 그렇다면 어떻게 해결하면 될까? 


![image](/assets/images/posts/vue/07.png)

우선 프로젝트의 public에 다음과 같이 WEB-INF 폴더를 만들고 web.xml에 다음과 같은 설정을 해주면 된다. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" version="3.0" metadata-complete="true">
  <display-name>[여기에 프로젝트 이름 입력]</display-name>
  <description>
      [여기에 설명 입력]
  </description>
  <!-- 아래의 옵션이 필요하다 -->
  <error-page>  
   <error-code>404</error-code>  
   <location>/</location>  
  </error-page>  
</web-app>
```

다음과 같이 404 에러시의 페이지위치를 그냥 루트로 설정하면 된다. 그러면 지금 현재의 URL-path에 맞는 페이지가 출력이 되게 되고 정상적인 서비스가 가능해진다. 
(만약 별도의 404 페이지가 필요하면 Vue 라우터 상에서 별도의 페이지를 만들어 등록시키면 된다)