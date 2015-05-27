### 2. 서버가 시작하는 시점에 부모 ApplicationContext와 자식 ApplicationContext가 초기화되는 과정에 대해 구체적으로 설명해라.
* 
ContextLoaderListener가 생성하는 webApplication context는 웹 애플리케이션에서 부모 컨텍스트가 된다. 그리고 Dispatcher Servlet이 생성하는 webApplication context는 자식 컨텐스트가 된다. 그래서 Dispatcher servlet은 여러개를 설정하는 것이 가능하고 Dispatcher servlet들이 공통으로 필요로 하는 빈을 ContextLoaderListener를 이용하여 설정한다. 

먼저 ContextLoaderListener에 설정한 context가 초기화되는데, 설정 파일을 따로 명시하지 않으면 WEB-INF/applicationContext.xml을 사용하여 context를 초기화한다. 그 후에 각각의 자식 servlet context들이 초기화된다. 이때, 자식 컨텍스트에서는 공통으로 사용할 수 있는 부모 컨텍스트를 참조할 수 있지만, 부모 컨텍스트에서는 아직 초기화되지 않은 자식 컨텍스트를 참조할 수 없다.  


### 3. 서버 시작 후 http://localhost:8080으로 접근해서 질문 목록이 보이기까지 흐름에 대해 최대한 구체적으로 설명하라. 
* 
localhost:8080으로 접근하면, dispatcher servlet이 시작된다. dispatcher servlet은 next-servlet.xml을 읽어 빈을 초기화시킨다. 그리고 url-pattern으로(/) 설정된 패턴에 따라서 Dispatcher servlet이 모든 요청을 가로챈다. 
따라서 localhost:8080에 대한 요청이 Controller에 설정해 놓은 request mapping에 따라서 QuestionController의 list 메소드를 실행시킨다. list 메소드에서는 qnaService의 findAll 메소드로 질문목록을 가져와서 model에 담고 qna/list를 return 한다. next-servlet.xml에 설정해 놓은 InternalResourceViewResolver에서는 return한 string에 suffix, prefix를 추가하여 jsp의 위치를 파악하여 forward시킨다. 여기서 필요한 css, js파일은 next-servlet.xml에 mvc:resources 로 mapping 해놓은 위치로 다시 요청을 보내가져 온다.

### 9. UserService와 QnaService 중 multi thread에서 문제가 발생할 가능성이 있는 소스는 무엇이며, 그 이유는 무엇인가?
* 
Spring의 기본 Scope으로 설정되어 있으면, 싱글톤으로 빈을 하나만 생성해서 사용하고, 싱글톤이 아닌 프로토타입 범위의 빈 배포는 특정한 빈에 대한 요청마다 새로운 빈(bean) 인스턴스를 생성한다. 이는 빈이 다른 빈에 주입되거나 컨테이너에 getBean() 메서드를 호출해서 빈을 요청하는 것을 의미한다. 따라서 다수의 사용자 요청을 받아주어야 하는 multithread 상황에서는 scope를 프로토타입으로 설정해야 한다. 