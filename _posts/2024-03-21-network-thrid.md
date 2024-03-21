---
title: Network Third week
published: true
---

## <span style="color:#802548">_쿠키, 세션_</span> 
- 쿠키나 세션이나 모두 stateless를 극복하려는 시도이다.
  - 쿠키는 상태를 client에 저장한다.
  - 세션은 상태를 WAS에 저장한다.
  - 쿠키와 session의 차이는 아래와 같다.
    - 쿠키는 클라에 저장돼 접근이 수월해 보안이 취약하다.
    - 세션은 서버에 저장돼 접근이 어려워 보안이 상대적으로 좋다.
    - 대신 쿠키는 서버 연동이 없으니 세션보다 속도가 좋다.
    - 세션은 상태 처리를 하는 프로세스(생성-갱신-만료)를 처리해서 속도가 느리다.
  - 쿠키와 session의 공통점은 아래와 같다.
    - cookie를 기반으로 한다.
    - cookie 기반이기에 web에서만 사용가능하다. 
      - 핸드폰 native에서 사용불가능하다.
      - 물론 핸드폰으로 여는 web에선 사용가능하다.(이른바 웹뷰)
- 쿠키를 client에서 만들수도 있지만, 인증에서 쓰이는 cookie는 서버에서 만드는 편이다. 작동방식은 아래와 같다.
  - 클라이언트가 request를 보낸다
  - 서버는 Set-Cookie header에 sessionid를 넣어 response
  - 차후 request부터 계속 cookie가 header에 담겨감.(browser에 의해 자동으로)

- client-side cookie creation. js다.



```js
function setCookie(name, value, days) {
    var expires = "";
    if (days) {
        var date = new Date();
        date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
        expires = "; expires=" + date.toUTCString();
    }
    document.cookie = name + "=" + (value || "") + expires + "; path=/";
}

// Example: Set a cookie named "username" with value "john_doe" that expires in 1 day
setCookie("username", "john_doe", 1);
```

- server-side cookie creation. java다.



```java
@GetMapping("/set-cookie")
    public String setCookie(HttpServletResponse response) {
        // Set a cookie named "username" with value "john_doe" that expires in 1 day
        Cookie cookie = new Cookie("username", "john_doe");
        cookie.setMaxAge(24 * 60 * 60); // 1 day in seconds
        response.addCookie(cookie);

        return "Cookie set!";
    }
```


- session의 작동방식은 아래와 같다.
  - 클라이언트가 request
  - 서버는 cookie header 확인하여 session-id가 있나 확인
  - 없으면 서버는 세션키를 생성, 세션키를 이용한 저장소 생성, 세션키를 담은 쿠키 생성
  - 서버가 set-cookie header에 session-id 넣어 response
  - 클라이언트는 향후 cookie header에 session-id값을 넣어 request(browser가 담당)
  - 서버는 session-id를 보고 세션저장소 데이터를 활용

- session은 browser별로 부여되는 것이다.
- 따라서 같은 사용자라도 browser를 바꿔 로그인을 하면 다른 session-id가 생성되어 관리된다.
- 당연히 다른 session-id끼리는 정보가 공유되지 않고 다시 쌓여야 한다.
- username이 늘 같은 key여도 value가 다르게 뽑히는 이유가 바로 session key가 다르게 관리되기 때문이다.
- 해당 session key(Java에선 JSessionId)가 가진 key에 맞는 value가 뽑힌다.
```java
@GetMapping("/create-session")
    public String createSession(HttpServletRequest request) {
        // Get the HttpSession object from the HttpServletRequest
        HttpSession session = request.getSession();

        // Set an attribute in the session
        session.setAttribute("username", "john_doe");

        return "Session created!";
    }
```

- 같은 browser여도 instance별로 모두 다르게 session id가 부여된다.
- 같은 chrome이어도 아래와 같이 다양한 instance가 있다.
  - web-app chrome
  - native-app chrome
  - webview chrome
- 이것들마다 서로 모두 다른 session id가 부여된다고 보면 된다.

## <span style="color:#802548">_URI, URL, URN_</span> 
- uri는 인터넷의 자원을 식별할 수 있는 문자열을 의미한다.
  - 그 중 url은 리소스가 위치한 정보를 사용하는 방식이다.
  - urn은 리소스에 이름을 매핑하여 사용하는 방식이다. 
  - 현실에선 uri나 url이나 잘 구분하지 않고 사용되는 경향이 매우 강하다. urn은 쓰이지 않는다.
- uri는 scheme://host/path?query#fragment와 같은 형식으로 구성된다.
  - scheme은 protocol이고, host는 domain이다. 
  - fragment는 자주 쓰이진 않지만 웹페이지의 특정 구역으로 이동하게 할 때 사용한다.


## <span style="color:#802548">_Restful api_</span> 
- Restful api는 자원의 표현을 통한 상태를 전달하는 데 초점을 둔 HTTP 활용 규약이다.
  - 1990년대의 HTTP api는 GET-POST로 모든 것을 처리했기 때문에 수정,삭제,생성 등이 별도의 method로 구분되지 않았다.
  - uri만으로 api를 통해 자원이 어떻게 변화하는지 파악하기도 어려웠다.
  - 그에따라 uri, http method를 특정 원칙을 준수하며 사용하자는 주장이 나왔는데 그게 바로 restful api다.
  - 복잡한 원칙은 제쳐두고 간단한 몇가지 원칙을 나열하면 아래와 같다. 거의 uri 중 path와 query에 적용된다.
    - 하이픈 허용. 언더바 X
    - 파일확장자 X
    - 소문자 uri
    - uri의 path는 동사가 아닌 명사로
    - uri의 paht는 위계질서를 갖게 구성
  - 이제 복잡한 제약조건을 살펴보자.
    - client - server 분리
      - HTTP api를 쓰는 방법과 동일하다.
    - stateless
      - session을 쓰지 말라는 의미다. 
      - restful api를 준수하려면 jwt같은 인증을 사용해야 한다.
    - cachable
      - 캐시를 사용해야 한다.
      - HTTP api를 쓰는 방법과 동일하다.
      - cache-control header를 사용하면 된다.
    - layered system
      - client는 오로지 rest api서버만 바라보면 되게 서버 아키텍쳐를 구성한다.
      - HTTP api를 쓰는 방법과 동일하다.
    -  Code on Demend
       - 선택조건으로 서버가 보내준 js 코드를 즉시 실행한다.
       - 솔직히 어떻게 작동하는건지 이해 못했다. 써본 적이 없다..
    - uniform interface
      - 리소스 식별
        - 특정 리소스는 오직 하나의 URL만 가져야 한다.
      - 표현을 통한 자원 조작 
        - accept 등 콘텐츠 협상헤더를 사용하여 동일한 uri로 새로운 리소스 표현을 제공할 수 있어야 한다.
        - 보통 text/plain, application/json 등이 자주 쓰이는 리소스 표현 방식이다.
      - 자기서술적 메시지
        - header, body만 보고 HTTP message가 파악될 수 있어야 한다.
      - 애플리케이션의 상태가 Hyperlink를 이용해 전이
        - 관련 있는 resource로 이동할 수 있는 링크가 제공되어야 한다.
        - 기존 HTTP api와는 많이 동떨어져있다.

<img src='/assets/hateoas.png' />



## <span style="color:#802548">_sop와 cors_</span> 
- 맨처음 SSR의 시대에는 client를 별도 origin으로 분리하지 않았다.
- 브라우저 resource 정책으로 sop(same origin policy)를 적용해 다른 origin은 다 막아버리면 됐다.
  - 하지만 front framework가 등장하고 CSR의 시대가 오면서 프론트에서 백엔드로부터 데이터만 받아 직접 render하는 형식이 됐다.
  - CSR에서는 front 서버가 생겨나면서 backend 서버와 완전히 분리되면서 port도 분리되었다. 서로 다른 origin(출처)로 여겨지게 된 것이다.
  - 이제는 sop를 적용할 수가 없었다. 그런 필요에 따라 cors(cross-origin-resources-sharing)가 등장했다. 
  - client에서 보낸 origin header의 값이 server에서 보낸 acess-control-allow-origin header의 값에 포함되면 브라우저는 server에서 온 response를 차단하지 않는 것이다.
- 보통 4가지 header를 설정해서 server에서 보내게 된다.
  - access-control-allow-origin     
  - access-control-allow-credentials
  - access-control-allow-header
  - access-control-allow-method
- 서버에서는 filter가 따로 없다면 request를 받으면 response를 하기 때문에 resource가 바뀌게 될 위험이 있다.
  - 따라서 browser는 server의 resource가 바뀌는 걸 방지하기 위해 cors 위반 여부를 먼저 검사하는 request를 날리게 된다.
  - 그 때 method는 options이며, 따로 body가 없다. 이를 preflight라고한다. 
  - preflight는 status Code가 중요하지 않다. browser가 origin header와 acess-control-allow-origin header가 같은지만 따진다.
  - header에 *를 넣으면 모든 origin이 허용되게 되기 때문에 개발은 편하지만, 보안상 좋지 않다.
    - Spring에서는 별도의 CorsConfig를 만들지 않으면 *처리되어 모든 origin을 허용한다.



```java
@Configuration
@Slf4j
public class CorsConfig {
	
	@Bean 
	CorsFilter corsFilter()	{
        .
        .
        .
		config.addAllowedOrigin("http://localhost:3000");
		config.setAllowCredentials(true); 
		config.addAllowedMethod("GET");
		config.addAllowedMethod("POST");
		config.addAllowedMethod("DELETE");
		config.addAllowedMethod("PUT");
		config.addAllowedMethod("PATCH");
		config.addAllowedMethod("*");
        .
        .
        .
	}
	
}
```


## <span style="color:#802548">_csrf_</span> 

- CSRF는 인증된 사용자가 web-app에 특정 request를 보내게 유도하는 보안 공격이다.
- 사용자가 인증한 경우, 요청이 사용자의 동의를 받았는지 확인이 어려운 web의 특성을 악용한 것이다.
- 아래는 간단한 예시다.
```
특정 은행계좌에서 해커의 계좌로 송금하라는 api를 만듦
해당 api를 호출하는 하이퍼링크를 웹사이트에 뿌려둠
해당 링크를 누르면 송금하는 api가 호출되어 송금이 이뤄짐
```

- CSRF를 방어하는 코드를 만드는 방법으로는 대표적으로 세 개가 있다.
  - referrer 검증
    - host와 referrer가 같지 않은 경우는 대부분 악의적인 경우에 의해 위조된 요청이기에 그냥 걸러버린다.
    - 피싱 사이트가 아니라 해당 웹사이트 내에서 일어난 일이면 취약하다. 
  - captcha: 사용자가 의도한 요청인지, 모르게 작동한 요청인지 거를 수 있다. 원인을 모르는 capcha에 대해서도 그냥 다 해주는 소비자의 경우는 무의미하다.
  - CSRF 토큰
    - 서버에서 만든 중요한 request의 경우에는 csrf token을 만들어서 hidden 값으로 넣어두고 서버에서 검증한다. 
    - 가장 안전하지만 가장 귀찮은 방식이다. 매 요청마다 해줘야 하기 때문이다.


- referrer 검증을 Java소스코드로 쓰면 아래와 같다.



```java
public class ReferrerCheck implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String referer = request.getHeader("Referer");
        String host = request.getHeader("host");
        if (referer == null || !referer.contains(host)) {
            response.sendRedirect("/");
            return false;
        }
        return true;
    }
}
```

- csrf토큰을 Java소스코드로 쓰면 아래와 같다.



```java
session.setAttribute("CSRF_TOKEN", UUID.randomUUID().toString());

public class CsrfTokenInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession httpSession = request.getSession();
        String csrfTokenParam = request.getParameter("CSRF_TOKEN");
        String csrfTokenSession = (String) httpSession.getAttribute("CSRF_TOKEN");
        if (csrfTokenParam == null || !csrfTokenParam.equals(csrfTokenSession)) {
            response.sendRedirect("/");
            return false;
        }
        return true;
    }
}
```


## <span style="color:#802548">_xss_</span> 
- 악의적인 사용자가 공격하려는 사이트에 스크립트를 넣어 정보를 탈취하는 공격이다.
- 간단한 예시는 아래와 같다.
```
악의적인 사용자가 보안이 취약한 사이트를 발견했습니다.
보안이 취약한 사이트에서 사용자 정보를 빼돌릴 수 있는 스크립트가 담긴 URL을 만들어 일반 사용자에게 스팸 메일로 전달합니다.
일반 사용자는 메일을 통해 전달받은 URL 링크를 클릭합니다. 일반 사용자 브라우저에서 보안이 취약한 사이트로 요청을 전달합니다.
일반 사용자의 브라우저에서 응답 메시지를 실행하면서 악성 스크립트가 실행됩니다.
악성 스크립트를 통해 사용자 정보가 악의적인 사용자에게 전달됩니다.
```

- 종류는 3가지이다.
  - reflected xss: 
    - 악의적인 사용자가 악성 스크립트가 담긴 URL을 만들어 일반 사용자에게 전달하는 경우
    - 검색키워드를 통해 url script 심기가 통하는지 본 뒤에, 통하면 url 단축기술을 악용해 악성 url 생성
  - stored xss:    
    - 보안이 취약한 서버에 악의적인 사용자가 악성 스크립트를 저장하는 경우
    - 대표적으로 게시글 작성을 악용
  - dom-based xss: 
    - 보안에 취약한 JavaScript 코드로 DOM 객체를 제어하는 경우
    - 대표적으로 uri fragment를 악용

- 방어법은 아래와 같다.
  - '<', '>' 와 같이 태그에 사용되는 기호를 엔티티코드로 변환
  - 서버에서 유효성 검증 수행
  - SCP 헤더 설정
  - 취약한 js 코드 사용 금지

- 아래는 Java에서 악성 요청이 들어온 경우 이를 엔티티코드로 변환하기 위한 filter class다.
- 해당 filter class를 wrapper로 감싸서 만들어 준다.


```java
package blog.in.action.filter;

import org.springframework.stereotype.Component;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.IOException;

@Component
public class XssAttackFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        filterChain.doFilter(new RequestWrapper((HttpServletRequest) servletRequest), servletResponse);
    }

    @Override
    public void destroy() {

    }

    private class RequestWrapper extends HttpServletRequestWrapper {

        public RequestWrapper(HttpServletRequest request) {
            super(request);
        }

        @Override
        public String[] getParameterValues(String parameter) {
            String[] values = super.getParameterValues(parameter);
            if (values == null) {
                return null;
            }
            int count = values.length;
            String[] encodedValues = new String[count];
            for (int i = 0; i < count; i++) {
                encodedValues[i] = cleanXSS(values[i]);
            }
            return encodedValues;
        }

        @Override
        public String getParameter(String parameter) {
            String value = super.getParameter(parameter);
            if (value == null) {
                return null;
            }
            return cleanXSS(value);
        }

        @Override
        public String getHeader(String name) {
            String value = super.getHeader(name);
            if (value == null) {
                return null;
            }
            return cleanXSS(value);
        }

        private String cleanXSS(String value) {
            value = value.replaceAll("&", "&amp;");
            value = value.replaceAll("<", "&lt;").replaceAll(">", "&gt;");
            value = value.replaceAll("\\(", "&#40;").replaceAll("\\)", "&#41;");
            value = value.replaceAll("/", "&#x2F;");
            value = value.replaceAll("'", "&#x27;");
            value = value.replaceAll("\"", "&quot;");
            return value;
        }
    }
}
```

- 아래는 취약하다고 일컬어지는 js코드다.
```js
document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent
```

- 그 다음은 CSP 헤더 설정이다.
- script-src는 엄격하지 않은 정책이라 nonce나 hash를 쓰는 걸 구글은 추천하고 있다.
- 이쪽은 잘 모르겠다. 써본적이 없어서..



```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self'; ">
```


## <span style="color:#802548">_sql injection_</span> 
- SQL Injection은 임의의 SQL문을 삽입하여 실행시켜 비정상 동작을 유도하는 행위다.
  - id, 비번 검색에 ' OR 1=1을 넣는 방식이다.
- 이를 막으려면 bind variable을 사용하여야 한다.
- Spring의 Mybatis와 JPA는 대부분 bind variable을 사용하기 때문에 입력값이 문자열로 취급된다.
  - 즉 id가 'OR 1=1'로 취급되는 형태다. 따라서 입력값이 뭐가 들어오든 상관없게 된다.
- 아니면 DROP 등 몇 가지 DML을 입력값에서 거르기도 한다.

```js
function filterNotAcceptedText(text) {
    const regex = /[%=><]/;
    if (regex.test(text)) {
        alert("특수문자는 입력이 허용되지 않습니다");
        text = text.replace(regex, "");
        return false;
    }

    const notAcceptedTextArray = ["OR", "SELECT", "INSERT", "DELETE", "UPDATE", "CREATE", "EXEC", "UNION", "TRUNCATE"];
    for (const element of notAcceptedTextArray) {
        const regex = new RegExp(element, "gi");
        if (regex.test(text)) {
            alert("해당 문자열은 검색이 불가능합니다.");
            text = text.replace(regex, "");
            return false;
        }
    }
    
    return true;
}
```


- 에러메시지에서 db 구조를 유추할 수 없게끔 에러메시지를 정제하는 작업도 필요하다.
- Spring의 경우, JPA든 Mybatis든 JDBC를 이용해 아래와 같이 bind variable의 조합으로 이뤄지는 편이다.
- 따라서 SQL INJECTION 대비를 개발자가 많이 신경쓰지 않아도 된다.



```java
 String sql = "INSERT INTO mytable (column1, column2) VALUES (?, ?)";

try (
    // Establishing a connection to the database
    Connection connection = DriverManager.getConnection(JDBC_URL, USERNAME, PASSWORD);
    
    // Creating a PreparedStatement object
    PreparedStatement preparedStatement = connection.prepareStatement(sql)
) {
    // Setting values for parameters
    preparedStatement.setString(1, "value1");
    preparedStatement.setString(2, "value2");

    // Executing the SQL statement
    int rowsAffected = preparedStatement.executeUpdate();
    
    // Printing the number of rows affected
    System.out.println(rowsAffected + " row(s) affected.");
} catch (SQLException e) {
    e.printStackTrace();
}
```


- Express.js에서도 대부분 아래와 같은 bind variable을 지원한다.



```js
app.get('/users/:username/:password', (req, res) => {
  const { username, password } = req.params;
  const query = 'SELECT * FROM users WHERE username = ? AND password = ?';
  
  pool.query(query, [username, password], (error, results, fields) => {
    if (error) {
      console.error(error);
      res.status(500).send('Error retrieving user data');
      return;
    }
    res.json(results);
  });
});
```

- 아래같은 패턴으로 쓰면 안 된다. bind variable을 사용하지 않았다.
- 이러면 보안 상으로 문제다. ' or 1=1 같은 무적의 치트키 조건문으로 원하는 정보를 다 뺴올 수 있다.
- 또한 Oracle 같은 경우는 SGA의 lib cache에 SQL 자체가 저장되는데, bind variable을 쓰면 딱 한번의 하드파싱으로 SQL을 계속 쓴다.
- 하지만 아래와 같이 bind variable을 쓰지 않으면 SQL문 자체가 lib cache에 저장되므로 username과 password가 바뀔 떄마다 매번 하드파싱이 필요하여 CPU 부족 문제가 대두될 수 있다.



```js
const express = require('express');
const mysql = require('mysql');

const app = express();

const pool = mysql.createPool({
  connectionLimit: 10,
  host: 'localhost',
  user: 'root',
  password: 'password',
  database: 'mydatabase'
});

app.get('/users/:username/:password', (req, res) => {
  const { username, password } = req.params;
  const query = `SELECT * FROM users WHERE username = '${username}' AND password = '${password}'`;
  
  pool.query(query, (error, results, fields) => {
    if (error) {
      console.error(error);
      res.status(500).send('Error retrieving user data');
      return;
    }
    res.json(results);
  });
});
```


## <span style="color:#802548">_웹 캐시_</span> 
- 웹 캐쉬란 client가 요청하는 html, assets, js, css등에 대해 첫 요청 시에 파일을 내려받아 특정 위치에 저장하는 복사본이다.
- 웹 케시에는 세 종류가 있다.
  - browser cache
    - 내부 디스크에 캐쉬한다. 개인에 한정된 Cache다.
    - 브라우저의 Back버튼 또는 이미 방문한 페이지를 재 방문하는 경우 극대화
  - Proxy Caches
    - Client나 Server가 아닌 네트워크 상에서 동작한다.
    - client가 proxy server에 접근해 caches file이 있는지 본다.
    - caches가 있으면 proxy server에서 데이터를 받아온다. 보통 ISP에서 캐시 서버를 만든다.
    - caches가 없으면 proxy server는 origin server로부터 데이터를 요청하여 받아온다. 
    - 받은 데이터는 proxy server에 저장된다. 그리고 client에게 데이터를 전달해준다.

<br/> 

- 캐시의 작동에는 캐시 관련 헤더도 중요하다.
  - 응답 결과가 캐시에 저장될 때 Last Modified(데이터 최종 수정 시간)도 저장된다.
  - response header인 Last-Modified의 값을 보고 언제 마지막으로 데이터가 수정되었는지 알 수 있다.
  - 캐시의 유효기간이 초과 되었어도, If-Modified-Since HTTP 요청 헤더를 사용하여 조건부로 요청을 할 수 있다.
  - 서버는 데이터를 검증하고, 데이터가 수정되지 않았다면 바디를 제외한 HTTP 헤더만 보내어 데이터 수정이 없었음을 알려준다. (상태 코드 304 Not Modified)
  - 해당 헤더는 하루 미만에 대해선 작동하지 않았다.
    - 따라서 ETag(Entity Tag) header가 등장했다.
    - ETag는 캐시 데이터에 고유한 버전 이름을 명시한다. 데이터가 변경되면 이 이름이 변경된다. 
    - 즉 ETag가 변경되면 다른 데이터로 인식된다는 의미다.
    - response의 ETag 값은 클라이언트 캐시에 저장된다.
    - 캐시 시간이 초과되기 전까지, ETag 값을 검증하는 If-None-Match를 요청 헤더에 넣어 보낸다.
    - 데이터가 변경되지 않았다면 ETag 값은 동일하고, 이때는 body 없이 HTTP 헤더만 전송된다. 
    - 상태 코드는 304 Not Modified이며, 브라우저는 캐시의 데이터를 재사용한다.
- 캐시가 언제 만료되는 지는 cache-control header에서 정의한다.
  - max-age 
    - 캐시 유효 시간 (초단위)를 의미한다.
  - no-cache
    - 데이터는 캐시를 해도 되지만, 항상 origin 서버에서 검증하고 사용되야 함을 의미한다.
    - 원서버에서 바뀐게 없다면 304를 return하니 원서버에서부터 client로 이미지를 끌고오지 않아도 된다.
    - 그럼 원서버에서 proxy server로 더 빠르게 오고, proxy server에서 캐싱된 resource를 return한다.
  - no-store 
    - 데이터에 민감한 정보가 있으니, 저장하지 않고 메모리에서만 빨리 사용하고 지우는 것을 의미한다.
    - 캐싱을 쓰면 안된다는 의미다. 매번 원서버에서 원본 resource를 받아와야 한다.
  - Expires
    - 캐시 만료일을 정확한 날짜로 지정할 수도 있다.



## <span style="color:#802548">_프록시 서버_</span> 
- 프록시 서버는 클라이언트와 원서버 사이에서 중개를 담당하는 서버를 칭한다.
  - 프록시 서버라고 말하는 것들은 대개 포워드 프록시 형태인데, 내부망에서 사용된다.
  - 포워드 프록시에 요청을 하게 되면, 캐싱된 원본이 있다면 멀리 있는 원서버까지 도달하지 않고 응답을 받을 수 있다. 
  - 포워드 프록시에 요청을 하게 되면, 익명성을 유지할 수 있다. 원서버는 어떤 client가 보냈는지 모르고 그저 proxy server가 보냈다는 것만 알기 때문이다.
  - 포워드 프록시에 요청을 하게 되면, 응답을 읽어 필터링을 할 수 있다. 사실상 해당 url을 차단하는 효과를 발휘한다.
  - 포워드 프록시는 클라이언트를 대신하여 요청하는 개념이다.
- 리버스 프록시는 웹 서버 앞에서 클라이언트의 요청을 웹서버에 전달한다.
  - 리버스 프록시가 요청을 받게 되면, 캐싱된 원본이 있다면 멀리 있는 원서버까지 도달하지 않고
  - 리버스 프록시가 요청을 받게 되면, WAS가 은폐되기 때문에 보안이 더 우수하다.
  - 리버스 프록시가 요청을 받게 되면, SSL을 중앙화해서 관리하기 매우 편하다. 각 WAS가 모두 하나의 reverse proxy에서 관리되기 때문이다.
  - 리버스 프록시는 서버를 대신하여 클라이언트의 요청을 받는 개념이다.
  - nginx는 웹서버로서 static content도 처리하지만 reverse proxy로서 WAS로 포워딩도 담당한다.
  - web server와 reverse proxy를 분리하는 경우는 많지는 않다고 한다.


## <span style="color:#802548">_타임아웃_</span> 
- connection timeout과 read timeout이 있다.
- connection timeout
  - 종단 간 연결하는데 소요되는 최대 시간을 넘어섰을 때 발생. 
  - 이 때의 연결이란 TCP 3 way handshake를 통한 TCP 연결이다.
  - 보통 5초에서 30초 정도로 잡힌다고 한다.
  - 원인은 보통 아래와 같다. 방화벽이 대부분의 원인이다.
    - 방화벽에서 커넥션 요청을 먹어 버렸다
    - 서버가 너무 바쁘다
    - 커넥션풀이 모두 사용중이다.
- read timeout
  - 연결된 종단 간에 데이터를 주고 받을 때 소요되는 최대 시간을 넘어섰을 때 발생.
  - 보통 10초에서 60초 정도로 잡힌다고 한다.
  - 원인은 보통 아래와 같다. 처리시간이 대부분의 원인이다.
    - 서버에서 request를 처리하는 데 오래 걸린다
    - 네트워크 대역폭



<img src="/assets/connection-timeout.png" />


## <span style="color:#802548">_로드밸런서_</span> 
- L4 로드밸런서는 TCP 및 UDP 프로토콜을 기반으로 클라이언트와 서버 간의 트래픽을 분산시킨다.
  - L4 로드 밸런서는 클라이언트의 IP 주소와 포트, 서버의 IP 주소와 포트를 기반으로 로드 밸런싱을 수행합니다.
  - 기능이 적은 만큼 빠르다.
- L7 로드밸런서는 HTTP 및 HTTPS 프로토콜을 기반으로 클라이언트와 서버 간의 트래픽을 분산시킨다.
  - request 내용(URL, 헤더, 쿠키 등)을 기반으로 로드 밸런싱을 수행한다.
  - URL에 따라 부하를 분산시키거나, HTTP 헤더에 따라, 쿠키에 따라 부하를 분산하는 등 클라이언트의 요청을 보다 세분화해 서버에 전달할 수 있다.
  - 기능이 많은 만큼 좀 느리다.
- nginx는 L4와 L7 모두 기능할 수 있다. conf를 어떻게 쓰느냐에 따라 달라진다.

- 아래는 L4 loadbalancer로 쓴 것이다.



```sh
# Define upstream servers
upstream backend_servers {
    server 192.168.1.10:80;
    server 192.168.1.11:80;
    server 192.168.1.12:80;
}

# Configure load balancing
server {
    listen 80;
    
    location / {
        proxy_pass http://backend_servers;
        # Other proxy settings such as timeouts, buffers, etc. can be configured here
    }
}
```

- 아래는 L7 loadbalancer로 쓴 것이다.
- 헤더값 혹은 uri값을 보고 ngnix가 reverse proxy로서 request를 해당 WAS로 forwarding시킨다.



```sh
# Define upstream servers
upstream backend_servers_a {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

upstream backend_servers_b {
    server 192.168.1.20:8080;
    server 192.168.1.21:8080;
}

# Map request headers to variables
map $http_my_custom_header $backend_group {
    default backend_servers_a;
    "group_b" backend_servers_b;
}

# Configure load balancing based on mapped variable
server {
    listen 80;

    location / {
        proxy_pass http://$backend_group;
        # Other proxy settings such as timeouts, buffers, etc. can be configured here
    }
}
```

- 로드 밸런싱 알고리즘은 아래와 같다.
  - Round Robin
    -기본 설정이다. 그냥 순서대로 진행되는 식이다.
  - Least Connections
    - 연결된 횟수가 가장 적은 서버로 연결되는 알고리즘이다.
  - IP Hash
    - 클라이언트 IP를 해싱하여 특정 클라이언트는 특정 서버로 연결되게 할 수 있다. Stick Session 세션 방식 처럼 동작하게 할 수 있다.
  - Generic Hash
    - 사용자가 정의한 다양한 변수를 조합해 트래픽을 분산할 수 있다.
  - Random
    - 트래픽을 무작위로 분배한다.



```sh
upstream samplecluster {
  least_conn;
  server 111.111.111.111:8080;
  server 222.222.222.222:8080;
  server 333.333.333.333:8080;
}
```

- upstream directive 없이 바로 server directive로 시작하는 경우 알고리즘을 지정할 수 없다. 
- 그냥 순서대로 뿌려주는 RR이 선택된다.



```sh
server {
    listen 80;

    location / {
        # Specify multiple backend servers directly in the proxy_pass directive
        proxy_pass http://backend1.example.com:8080 http://backend2.example.com:8080 http://backend3.example.com:8080;
        # Other proxy settings such as timeouts, buffers, etc. can be configured here
    }
}
```

## <span style="color:#802548">_JWT_</span> 
- JWT는 유저를 인증하고 식별하기 위한 토큰(Token) 기반 인증이다.
- Oauth server에서 인증 전략으로 자주 채택된다.
  - 하나의 WAS에 묶인 session과 다르게 이기송 서버간에도 인증이 가능해 SSO나 MSA에서 많이 쓰인다.
  - 세션과 쿠키가 존재하지 않는 Native app 환경에서도 많이 쓰인다.
- JWT의 구성은 아래와 같다.
  - header
    - algorithm, token type
  - payload
    - subject, exp, iss 등 claim(민감하지 않은 사용자정보)
  - signature
    - header와 payload를 암호화
- JWT의 원칙은 아래와 같다.
  - hedaer와 payload는 base64 url safe한 문자열로 구성되어야 한다. (url에서 쓸 수있게 변경. "+" 대신 "-"를 사용, "/" 대신 "_"를 사용, 패딩 문자 "="는 제거)
  - 전자서명을 복호화한 값이 실제 온 header/payload와 일치해야 한다.
- 전자서명 방식은 대칭키 교환(HMAC)도 있고, 비대칭키 교환(RSA)도 있다.
  - 속도가 중요하면 대칭키 교환을, 보안이 중요하면 비대칭키 교환을 사용한다.
- 대칭키 전자서명의 작동 방식은 아래와 같다.
  - request가 오면 클라이언트에게 첫 token을 발급한다.
  - 이 때 전자서명은 서버의 대칭키로 암호화된다.
  - 받은 response에서 토큰을 쿠키나 localStorage에 저장한다.
  - 다음 request를 보낼 때 토큰 값을 넣어 보낸다.
  - 서버는 요청이 오기 전 filter에서 대칭키로 복호화하여 전자서명 위조 여부를 확인한다.
  - 전자서명이 위조되지 않았다면 실제 요청을 처리하여 결과값을 client에 전달한다.
- 비대칭키 교환 전자서명은 보통 요청 처리 서버(api 서버)와 인증 서버(발급 및 인증)가 다르다.
- 물론 클라이언트가 verify를 해도 되지만.. 보통은 그렇게 하지 않는다고 한다. 서버보다 클라가 느려서 그럴 것이다.
  - client가 인증 서버에 로그인 request를 보내 access token을 받는다.
    - 이 때 인증 서버에서는 토큰의 signature는 인증 서버의 개인키로 암호화된다.
  - client는 token을 받고 api 서버에서 요청을 보낸다.
  - api 서버는 인증 서버의 공개키로 토큰 내 전자서명의 유효성을 검증한다.
    - 유효하다면, api 서버는 request를 처리하여 response를 보낸다.
    - 토큰이 만료됐다면, api 서버에서 401 에러를 return한다.
  - client는 401 오류의 경우에 인증 서버로 다시 request를 보내 token을 받는다.
  - 새로운 token 값으로 바꿔서 api 서버에 또 request를 보낸다.
  - api 서버는 토큰이 유효함을 보고 response를 내준다.


- 프론트에서 axios를 활용한다면 아래와 같이 interceptor를 활용해 비교적 쉽게 짤 수 있다.
- 아래는 request interceptor다. response interceptor도 찾아보면 예시가 많이 있다.



```js
export const instance = axios.create({
  timeout: 10 * 1000,
  baseURL: apiRootPath,
});

instance.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    if (config.url !== '/login') {
      const accessToken = localStorage.getItem('accessToken');

      config.headers['authorization'] = `Bearer ${accessToken}`;
    }

    return config;
  },
  error => {
    return Promise.reject(error);
  }
);
```

- 참고로 비대칭키 교환에서 인증과 api(resource) 서버가 분리된 경우, api 서버가 아닌 인증 서버로 request를 날리는 것이기 때문에 baseURL이 적힌 instnace를 쓸 수 없다.
- 대부분의 경우 공유되는 axios instance는 api 서버(resource 서버)를 대상으로 baseURL을 지정하기 때문이다.
- 따라서 refreshToken으로 accessToken을 새로 받는 로그인 연장의 경우에만, 인증서버로 request를 날리게끔 해야 한다.
- 따라서 response interceptor에서 baseURL이 다른 axiosRequestConfig 객체를 따로 만들어야 한다.
- 물론 많은 경우 localStorage 자체를 이용하기보단 store를 이용하게 된다.



```js
instance.interceptors.response.use(
  response => {
    return response;
  },
  async (error: AxiosError) => {
    if (error.response?.status === 401) {
      const tokenResponse = await axios.post('인증서버url', loginId);
      const accessToken = tokenResponse.data;
      localStorage.setItem('accessToken', accessToken);
      
      const newRequestConfig: AxiosRequestConfig = {
        ...error.config,
        headers: {
          authorization: `Bearer ${accessToken}`,
          'content-type': 'application/json',
        },
      };

      const result = await axios(newRequestConfig);
      return result;
    }

    return Promise.reject(error);
  }
)
```

## <span style="color:#802548">_출처_</span> 
- https://jaeseongdev.github.io/development/2021/06/15/REST%EC%9D%98-%EA%B8%B0%EB%B3%B8-%EC%9B%90%EC%B9%99-6%EA%B0%80%EC%A7%80/ - RESTFUL api
- https://www.bugbountyclub.com/pentestgym/view/47 - csrf 
- https://devscb.tistory.com/123 - csrf
- https://junhyunny.github.io/information/security/spring-mvc/reflected-cross-site-scripting/ - xss
- https://portswigger.net/web-security/cross-site-scripting/dom-based - xss
- https://developer.chrome.com/docs/lighthouse/best-practices/csp-xss?hl=ko - csp
- https://www.mois.go.kr/synap/skin/doc.html?fn=BBS_201405271120475982&rs=/synapFile/202403/&synapUrl=%2Fsynap%2Fskin%2Fdoc.html%3Ffn%3DBBS_201405271120475982%26rs%3D%2FsynapFile%2F202403%2F&synapMessage=%EC%A0%95%EC%83%81 - 정부의 secure java coding
- https://velog.io/@citron03/%EC%9B%B9-%EC%BA%90%EC%8B%9C-WEB-Cache-%EC%A0%95%EB%A6%AC - 웹 캐시
- https://www.youtube.com/watch?v=JqCgJI-Nk88 - 프록시, 리버스 프록시, 포워드 프록시
- https://aws-hyoh.tistory.com/m/149 - 로드밸런서 스위치
- https://alden-kang.tistory.com/m/20#:~:text=%EB%A8%BC%EC%A0%80%20Connection%20Timeout%EC%9D%80%20%EC%A2%85%EB%8B%A8,%EC%82%AC%EC%9A%A9%EB%90%98%EB%8A%94%20%ED%83%80%EC%9E%84%EC%95%84%EC%9B%83%20%EC%9E%85%EB%8B%88%EB%8B%A4 - connection/read timeout