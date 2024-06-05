# Spring MVC basic


Spring MVC를 이용해 웹 어플리케이션을 개발하는 방법을 살펴본다.

Spring boot를 공부하기전 비교?를 하기 위해서 살짝 보자.

<!--more-->

Spring을 결국엔 하게 되었다. Spring boot를 많이 사용하지만 MVC의 내용은 알아야 하므로 살펴봅시다.

## Spring MVC의 구성 및 흐름

다른 MVC기반의 프레임 워크와 마찬가지로 Spring MVC도 Controller를 사용하여 Client의 요청을 처리한다. 이 Controller의 역할을 하는 것이 **DispatcherServlet**이다.

|  구성요소   |            설명          |
|-----------|-------------------------|
| DespatcherServlet | Client의 요청을 받음 Controller에게 Client의 요청을 전달하고 Contoller의 return값을 받아 View에 전달 |
| HandlerMapping | Client의 요청 URL을 어떤 Controller가 처리할지 결정 |
| Controller | Client의 요청 처리, 결과를 DispatcherServlet에 알려줌 | 
| ViewResolver | Commander의 처리 결과를 보여줄 View를 결정 |
| View | Commander의 처리 결과를 보여줄 응답을 생성 |

흐름은 다음과 같다.

1. Client의 요청이 DispatcherServlet에 전달 
2. DispatcherServlet은 HandlerMapping을 사용해 Client의 요청이 전달될 Controller 객체 검색
3. DispatcherServlet은 Contoller 객체의 handleRequest() 메소드를 호출해 Client의 요청 처리
4. Contoller.handleRequest() 메소드는 ModelAndView 객체 return
5. DispatcherServlet은 ViewResolver로 부터 처리 결과를 보여줄 View 검색
6. View는 Client에 전송할 응답 생성.

## Spring MVC 웹 개발

1. Client의 요청을 받을 DispatcherServlet을 web.xml 파일에 설정.
2. 요청 URL과 Controller의 매핑 방식 설정.
3. Controller의 처리 결과를 어떤 View로 보여줄지 결정하는 ViewResolver설정
4. Controller를 작성
5. View 코드 작성

### DispatcherServlet 설정 및 Spring context 설정

Spring MVC를 사용하기 위해서 web.xml 파일에 DispatcherServlet을 설정 해줘야한다.  
DispatcherServlet은 Servlet 클래스로서 다음과 같이 설정해주면 된당.

~~~xml
<?xml version="1.0" ?>

<web-app version="2.4"
    xmlns="http://java.sun.com/xml/ns/j2ee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
                        http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
    <servlet>
        <servlet-name>example</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>example</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>
~~~

위의 설정은 *.do로 들어오는 요청을 DispatcherServlet이 처리하도록 하고 있다.

위의 경우 `<servlet-name>example</servlet-name>` 로서 이름이 example 이므로 사용되는 Spring설정 파일은 `WEB-INF/example-servlet.xml` 이다.

만약 다른 이름의 파일을 사용하고 싶다면 `<init-param>`을 추가 해주면 된다.

~~~xml
<servlet>
    <servlet-name>example</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            /WEB-INF/example1.xml,
            /WEB-INF/example2.xml
        </param-value>
    </init-param>
    <load-on-startup>3</load-on-startup>
</servlet>
~~~

여기서 만약 동일한 이름의 Bean을 지정했다면 뒤에 위치한 설정 파일에 명시된 Bean이 우선순위를 가진다.

### HandlerMapping 설정

클라이언트 요청을 Sping의 DispatcherServlet이 처리 하도록 설정했다면 다음은 HandlerMapping을 사용할지의 여부이다.  
HandlerMapping은 클라이언트의 요청을 어떤 Commender가 수행할 지의 여부를 결정한다.

|  구현체   |            설명          |
|-----------|-------------------------|
| BeanNameUrlHandlerMapping | 요청 URI와 동일한 이름을 가진 Controller Bean을 Mapping |
| SimpleUrlHandlerMapping | 경로 Mapping 방식을 사용하여 URI와 Controller Bean을 Mapping |

#### BeanNameUrlHandlerMapping

BeanNameUrlHandlerMapping은 요청 URI와 동일한 이름을 갖는 Controller Bean으로 하여금 클라이언트 요청을 처리한다.

HandelrMapipng Bean을 따로 생성하지 않은 경우 DispatcherServlet은 기본적으로 HandlerMapping의 구현체로 BeanNameUrlHandlerMapping을 사용한다.

#### SimpleUrlHandlerMapping

SimpleUrlHandlerMapping은 Ant 스타일의 Mapping 방식을 사용하여 Controller에 매칭한다.

* ? - 한글자에 매칭
* \* \- 0개 이상의 글자에 매칭
* \*\* \- 0개 이상의 디렉토리에 매칭

### ViewResolver 설정

HandlerMapping을 했다면 이제 ViewResolver를 설정한다.

* internalResourceViewResolver - JSP를 사용하여 View 생성
* VelocityViewResolver - Velocity 템플릿 엔진을 사용하여 View 생성

#### internalResourceViewResolver

다음과 같이 두 개의 프로퍼티를 입력받는다.
* prefix - Controller가 리턴한 뷰 이름 앞에 붙을 접두어
* suffix - Controller가 리턴한 뷰 이름 뒤에 붙을 확장자

~~~xml
<property name="prefix" value="/view/" />
<property name="suffix" value=".jsp" />
~~~

ExampleController가 `example`을 return했다고 하면 View는 `/view/example.jsp`가 된다.

#### VelocityViewResolver

Velocity 템플릿을 사용한다.

### Controller 구현

Controller를 구현하는 방법응ㄴ Controller인터페이스를 implements하는 것이지만 몇가지 기능을 구현하고 있는 클래스만 상속 받아서 구현하는게 일반적이다.

#### Controller interface

org.springframework.web.servlet.mvc.Controller

~~~java
public interface Controller {
    
    ModelAndView handleRequest(HttpServletRequest request,
                               HttpServletResponse response)
                               throws Exception
}
~~~

Controller를 구현하는 방법은 handleRequest() 메소드에 맞게 구현하는 것이다.



### View의 구현

View에서는 ModelAndView 객체에 저장한 객체를 사용할 수 있다.

다음과 같이 ModelAndView에 데이터를 저장했다고 보자.

~~~java
 protected ModelAndView handle(HttpServletRequest request,
            HttpServletResponse response, Object command, BindException errors)
            throws Exception {
        Example example = (Example) examples;
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("hello", example);
        return mv;
    }
~~~

이 경우 JSP에서 다음과 같이 사용할 수 있다.

~~~jsp
example: ${example.hello}
<%
    Example cmd = (Example) request.getAttribute("hello");
%>
~~~

뭐 워낙 예전 방식이라 전자정부프레임워크 쓸때 이런식으로 했던 것으로 기억하는데...  
Spring도 업데이트 되면서 많은 것이 바뀌었으므로 천천히 업데이트 해야겠다.
