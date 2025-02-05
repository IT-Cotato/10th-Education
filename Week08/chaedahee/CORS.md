# CORS : why, how

CORS(교차 출처 자원 공유)
웹 애플리케이션에서 서로 다른 출처 간의 자원 접근을 허용하는 보안 메커니즘이다.
<br/>CORS의 개념, 발생 원인, 문제 해결 방법

## CORS 개념

`Cross-Origin Resource Sharing`의 약자 <br/>
교차 출처 간의 자원 공유를 의미<br/>
웹 애플리케이션이 다른 도메인 또는 포트에서 호스팅되는 자원에 접근할 때 발생하는 **보안 정책**.<br/>
웹 브라우저는 기본적으로 동일 출처 정책(SOP)에 따라 서로 다른 출처 간의 자원 요청을 차단한다.

### 에러 발생 원인

주로 클라이언트(예: 리액트 앱)에서 서버(예: 스프링 백엔드)로 요청을 보낼 때 발생한다.<br/>

ex.

- 클라이언트가 `localhost:3000`에서 서버의 `localhost:44300`으로 POST 요청을 보낼 경우
- 만약 서버가 적절한 CORS 설정을 하지 않았다면 브라우저는 요청을 차단한다.

### 에러메시지

브라우저에서 발생하는 CORS 에러 메시지 :
![alt text](image.png)

중요한 부분은 여기다.

> "localhost 44300 링크에서 localhost 3000 출처로 가져올 수 있는 엑세스가 CORS 정책에 의해 차단되었습니다."

이 메시지는 요청한 자원에 'Access-Control-Allow-Origin' 헤더가 없음을 나타낸다. 즉 CORS 정책에 따라 요청이 차단된 경우.

## 출처(Origin) 개념

웹에서 출처란 프로토콜, 호스트, 포트의 조합으로 정의

> ex. URL `https://www.google.com:80/search?q=cotato`의 구성 요소 :

- 프로토콜: `https://`
- 호스트: `www.google.com`
- 포트: `:80`
- 경로: `/search`
- 쿼리: `?q=cotato`
- 프래그먼트: `#first`

여기서 출처는 **_프로토콜, 호스트, 포트_** 를 뜻한다.

출처가 동일할 경우에는 자유롭게 자원을 공유할 수 있지만, <br/>
다른 출처 간의 상호작용은 제한된다.

## 동일 출처 정책(SOP) : same origin policy

동일한 출처에서만 자원을 공유할 수 있다.
보안상의 이유로 다른 출처 간의 자원 접근을 차단하는 정책이다.<br/>

#### why?

Cross-Site Request Forgery(CSRF), Cross-Site Scripting(XSS) 공격으로부터 보호하기 위해

그러나 이러한 제한은 개발자가 필요한 자원에 접근하는 것을 어렵게 하기 때문에

CORS를 통해 예외적으로 특정 조건을 만족하는 경우

다른 출처의 자원에 접근할 수 있도록 허용합니다.

## 브라우저 정책

에러메시지가 발생하는 곳은 서버도 클라이언트도 아니다.

정확히는 브라우저에서 출처를 분석해서

다른 경우에는 받을 수 없도록 차단을 한 것

즉 이 에러메시지는 sop 브라우저 정책을 위반하였으니 cors 정책을 사용할 수 있는 설정을 추가해서 다른 출처의 자원 공유를 허용하도록 하라는 의미를 가진다.

사실 해결할 수 있는 방법을 알려준 것

### 누구의 역할?

- 서버 개발자 : 이 에러의 해결책은 가장 표준적으로 사용되는 것이 CORS 응답 헤더 설정이고, 이는 서버에서 설정해야 한다.
- 클라이언트 개발자 : 프록시 서버로 설정할 수 있다.

> **누구의 역할이다가 아닌, 모두가 개념을 이해하는 것이 중요하다.**

## CORS의 동작 원리

CORS는 클라이언트가 서버에 요청을 보낼 때, 요청 헤더에 `Origin` 필드를 포함한다. 서버는 이 요청에 대해 `Access-Control-Allow-Origin` 헤더를 통해 허용된 출처를 명시한다.

브라우저는 이 두 출처를 비교하여 다를 경우 cors 에러를 반환하고 해당 응답은 없어진다.<br/>
출처가 같을 경우 요청은 유효하고 정상적으로 자바스크립트에 응답이 반환된다.

CORS 관련된 내용은 브라우저와 서버 사이를 집중!!

### 요청 시나리오

CORS 요청의 세 가지 시나리오 :

1. **Simple Request**:
   - 서버에 별도의 확인 없이 본 요청을 보내는 방식
   - 이 경우 `Content-Type` 헤더가 일반적으로 통신에 쓰이는 xml, json 형식이 아니다.
   - 거의 사용되지 않는다.
2. **Preflight Request**:
   - 클라이언트가 서버에 `OPTIONS` 메소드를 사용하여 **예비 요청**을 보내고,
   - 서버가 허용된 출처를 응답하는 과정이다.
   - 이 과정에서 출처가 다를 경우 CORS 에러가 발생
   - 같을 경우 본 요청을 정상적으로 보낸다. `OPTIONS` 아님
3. **Credentialed Request**:
   - Preflight와 유사
   - 인증 정보를 포함한 요청
   - 보안상의 이유로 출처를 명확히 설정
   - `Access-Control-Allow-Origin`에서 와일드카드 `*` 사용이 불가능하다.
   - credentials 필드를 true 옵션으로 지정해 전송

## CORS 문제 해결 방법

### 1. 서버 측에서 `Access-Control-Allow-Origin` 헤더를 적절히 설정한다.

Credential 옵션 설정하는 방법

- 클라이언트 Javascript
  ```javascrip
  //fetch 메서드
  fetch("https://localhost:8080/api", {
  	method: "POST",
  	credentials: "include", // 인증 정보 포함
    body: JSON.stringify({
        userId: 1,
    }),
  })
  //axios 라이브러리
  axios.post('https://localhost:8080/api', {
    profile: { username: username, password: password }
  }, {
  	withCredentials: true // 인증 정보 포함
  })
  ```
- 서버 Java : 전역 세팅

  ```java
  @Configuration
  public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("https://localhost:3000") // 허용 출처
                .allowCredentials(true); // 인증 정보 포함
    }
  }
  ```

### 2. 프론트엔드에서 프록시 서버를 구축한다.

```json
// vite.config.json
export default defineConfig({
  server: {
    proxy: {
      '/api': {
        target: 'http://www.google.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
        secure: false,
        ws: true,
      },
    },
  },
});
```

### 3. 크롬 확장 프로그램을 통해 로컬 환경에서 CORS 문제를 우회한다.

[Allow CORS: Access-Control-Allow-Origin](https://chromewebstore.google.com/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf?pli=1)
