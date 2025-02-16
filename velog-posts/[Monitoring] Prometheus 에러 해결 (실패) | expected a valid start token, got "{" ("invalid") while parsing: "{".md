<h1 id="문제-상황">문제 상황</h1>
<p>EC2와 프로메테우스를 연동하는 과정에서 <code>expected a valid start token, got &quot;{&quot; (&quot;invalid&quot;) while parsing: &quot;{&quot;</code> 이런 에러가 발생했습니다. </p>
<p>에러 로그로는 관련된 클래스가 없다는 내용이 주였고, 프로메테우스에서 상태를 확인해 보니 사진과 같이 UP 되어 있어야 할 <code>spring-acturator</code> 가 DOWN 되어 있었습니다. 
<img alt="" src="https://velog.velcdn.com/images/leegarden/post/7416a676-8b38-447e-8e55-e7304692015a/image.png" /></p>
<h1 id="시도했던-방법">시도했던 방법</h1>
<h2 id="1-의존성-추가">1. 의존성 추가</h2>
<p><code>build.gradel</code> 에 아래와 같이 추가해 줬습니다. </p>
<pre><code>    // Monitoring
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'io.micrometer:micrometer-registry-prometheus'
    implementation 'org.apache.tomcat.embed:tomcat-embed-core:10.1.18'
    implementation 'org.apache.tomcat.embed:tomcat-embed-el:10.1.18'
    implementation 'org.apache.tomcat.embed:tomcat-embed-websocket:10.1.18'
    implementation 'io.projectreactor:reactor-core:3.6.2'
    implementation 'io.projectreactor.netty:reactor-netty-core:1.1.15'
    implementation 'io.projectreactor.netty:reactor-netty-http:1.1.15'</code></pre><h2 id="2-applicationyml-파일-수정">2. application.yml 파일 수정</h2>
<p><code>application.yml</code> 파일도 다음과 같이 작성해 모든 엔드포인트로 접근할 수 있도록 해줬습니다. </p>
<pre><code># Actuator &amp; Prometheus 설정 추가
management:
  endpoints:
    web:
      exposure:
        include: &quot;*&quot;
  endpoint:
    prometheus:
      enabled: true</code></pre><h2 id="그러나">그러나</h2>
<p>명시적으로 버전 다 추가해서 의존성 넣어주고, <code>application.yml</code> 파일도 고쳐주고 많은 시도를 했지만, 클래스가 없다는 에러 로그는 사라져도 여전히 <code>actuator/prmetheus</code> 엔드포인트가 일반 API 엔드포인트로 인식되어 접근할 수 없다는 내용의 에러는 사라지지 않았습니다.</p>
<p><del>당연하지.. 저렇게 (못)생긴 API는 내 API에 있지도 않으니까</del></p>
<h2 id="3-webconfigjava-작성">3. WebConfig.java 작성</h2>
<p>문제를 좀 더 확인해 보니 <code>actuator/prometheus</code> 가 일반 api로 인식 되어 토큰 관련 문제가 발생하는 거라고 이해할 수 있었습니다. 
<code>SecurityConfig</code> 에서 해당 엔드포인트에 대한 접근은 허용해 줬지만, 어떻게 처리해야 하는지는 처리하지 않았습니다. </p>
<p>왜냐 해당 프로젝트는 웹이 아닌 앱이었기에 <code>WebConfig</code> 설정을 따로 하지 않았기 때문입니다. 이번 기회에 새로 생성하여 해당 엔드포인트가 <code>Spring Boot Actuator</code> 에 의해 처리되도록 하였습니다. </p>
<pre><code>@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer.setUseTrailingSlashMatch(true);
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        // actuator를 제외한 나머지 경로에 대해서만 정적 리소스 처리
        registry.addResourceHandler(&quot;/static/**&quot;)
                .addResourceLocations(&quot;classpath:/static/&quot;);
    }
}</code></pre><h2 id="두-번째-그러나">두 번째 그러나</h2>
<p>이렇게 설정해 줬음에도 불구하고 여전히 일반 api로 인식되어 프로메테우스에 status에서 여전히 토큰 관련 문제를 뱉고 있었습니다. </p>
<p>문제는 이 api의 요청을 GeneralExceptionHandler가 가로채서 하고 있는 것입니다. 
욕심도 많지 자기 것도 아닌데 가로채고;</p>
<p>그래서 해당 클래스의 어노테이션에 패키기를 명확히 해주서 actuator 패키지는 제외되도록 하였습니다. 
<code>@RestControllerAdvice(basePackages = &quot;com.umc.yourun&quot;)  // actuator 패키지 제외</code></p>
<h2 id="새로운-에러-발견">새로운 에러 발견</h2>
<p>이제 위와 같은 설정을 통해 일반 api로 인식되는 문제는 해결했습니다. 
그러나 반가워 해야 될까요 ?
새로운 에러를 만났습니다.
<code>server returned HTTP status 403</code></p>
<h2 id="실패">실패</h2>
<p>저 에러 이후로 온갖 방법들을 다 헤봤지만, 애초에 <code>actuator/prometheus</code> 엔드포인트가 노출되지 않아서 접근 자체가 불가능합니다. </p>
<h3 id="그래서-그냥-ec2에서-프로메테우스-삭제하고-처음부터-설치하려고-합니다">그래서 그냥 ec2에서 프로메테우스 삭제하고 처음부터 설치하려고 합니다.</h3>