<h2 id="intro">Intro</h2>
<p>프로메테우스와 그라파나 개념 정리를 통해 어떻게 모니터링을 하는지 이해할 수 있었습니다.
이제 실제로 이 두 툴을 사용해서 직접 모니터링을 해보려고 합니다. 
현재 참여 중인 <strong>프로젝트의 시스템 및 애플리케이션 메트릭을 모니터링</strong> 해보겠습니다.</p>
<blockquote>
</blockquote>
<ol>
<li>간단한 프로젝트라 하나의 EC2에 스프링, 프로메테우스와 그라파나를 함께 띄웠습니다.</li>
<li>프로젝트의 기획 특성상 메모리 및 CPU 사용량을 확인하고 폭주에 대응하는 것이 특히 중요하다 판단하여 알림 기능을 함꼐 하였습니다. </li>
<li>매트릭 클래스를 작성하여 각각의 API 호출 횟수를 확인하였습니다.</li>
</ol>
<h1 id="1-스프링-코드-작성">1. 스프링 코드 작성</h1>
<p><code>build.gradle</code> </p>
<pre><code>    // Monitoring
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'io.micrometer:micrometer-registry-prometheus'
    implementation 'org.apache.tomcat.embed:tomcat-embed-core:10.1.18'
    implementation 'org.apache.tomcat.embed:tomcat-embed-el:10.1.18'
    implementation 'org.apache.tomcat.embed:tomcat-embed-websocket:10.1.18'
    implementation 'io.projectreactor:reactor-core:3.6.2'
    implementation 'io.projectreactor.netty:reactor-netty-core:1.1.15'
    implementation 'io.projectreactor.netty:reactor-netty-http:1.1.15'</code></pre><p><code>application.yml</code> </p>
<pre><code># Actuator &amp; Prometheus 설정 추가
management:
  endpoints:
    web:
      exposure:
        include: &quot;*&quot;
  endpoint:
    prometheus:
      enabled: true</code></pre><p><code>ApiMetricsAspect.java</code> </p>
<pre><code>import io.micrometer.core.instrument.MeterRegistry;
import jakarta.servlet.http.HttpServletRequest;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;
import io.micrometer.core.instrument.Counter;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

@Aspect
@Component
@Slf4j
public class ApiMetricsAspect {
    private final MeterRegistry meterRegistry;

    public ApiMetricsAspect(MeterRegistry registry) {
        this.meterRegistry = registry;
    }

    @Before(&quot;execution(* com.umc.yourun.controller..*.*(..))&quot;)
    public void countApiCall(JoinPoint joinPoint) {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder
                .currentRequestAttributes())
                .getRequest();

        String path = request.getRequestURI();

        // Challenges API 모니터링
        if (path.startsWith(&quot;/api/v1/challenges/&quot;)) {
            String type = path.contains(&quot;/crew&quot;) ? &quot;crew&quot; :
                    path.contains(&quot;/solo&quot;) ? &quot;solo&quot; : &quot;other&quot;;

            String endpoint = path.substring(&quot;/api/v1/challenges/&quot;.length());

            Counter.builder(&quot;api.calls&quot;)
                    .tag(&quot;category&quot;, &quot;challenges&quot;)
                    .tag(&quot;type&quot;, type)
                    .tag(&quot;endpoint&quot;, endpoint)
                    .description(&quot;Number of Challenge API calls&quot;)
                    .register(meterRegistry)
                    .increment();
        }

        // Users API 모니터링
        else if (path.startsWith(&quot;/api/v1/users/&quot;)) {
            String type;
            if (path.contains(&quot;/mate&quot;)) {
                type = &quot;mate&quot;;
            } else if (path.contains(&quot;/runnings&quot;)) {
                type = &quot;runnings&quot;;
            } else {
                type = &quot;users&quot;;
            }

            String endpoint = path.substring(&quot;/api/v1/users/&quot;.length());

            Counter.builder(&quot;api.calls&quot;)
                    .tag(&quot;category&quot;, &quot;users&quot;)
                    .tag(&quot;type&quot;, type)
                    .tag(&quot;endpoint&quot;, endpoint)
                    .description(&quot;Number of Users API calls&quot;)
                    .register(meterRegistry)
                    .increment();
        }
    }
}</code></pre><p>주 API인 users와 challenges를 모니터링하되, challenges에서는 crew, solo로 타입을 나누었고 users에서는 mate, runnings, general(users)로 나눠 타입별로도, 각각의 API로도 횟수를 알 수 있도록 클래스를 작성했습니다. </p>
<p>Security 설정이 되어 있을 경우, <code>&quot;/actuator/**&quot;</code> 를 허용 엔드포인트에 추가해 스프링에서 메트릭을 수집할 수 있도록 하였습니다. </p>
<p>또한 프로메테우스 관련 api가 일반 api로 인식되어 GeneralExceptionHandler이 가로채지 않도록 어노테이션을 이와 같이 패키지를 명시해줬습니다. 
<code>@RestControllerAdvice(basePackages = &quot;com.패키지.패키지&quot;)  // actuator 패키지 제외</code></p>
<h1 id="2-prometheus-설치">2. Prometheus 설치</h1>
<p>보안 그룹 - 인바운드 규칙 9090(프로메테우스), 3000(그라파나)를 열어줍니다. 
<img alt="" src="https://velog.velcdn.com/images/leegarden/post/f287bd36-af55-42f0-8730-dbad50288894/image.png" /></p>
<p>이제 EC2에 접속하여 프로메테우스 설치를 진행합니다. </p>
<pre><code># Prometheus 다운로드 및 설치
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvfz prometheus-2.45.0.linux-amd64.tar.gz
cd prometheus-2.45.0.linux-amd64</code></pre><p>설치를 하면 <code>prometheus.yml</code> 파일이 생성되며, 아래와 같이 수정해 주면 됩니다. </p>
<pre><code># my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - &quot;first_rules.yml&quot;
  # - &quot;second_rules.yml&quot;

# A scrape configuration containing exactly one endpoint to scrape:
scrape_configs:
  # Prometheus 자체 모니터링
  - job_name: &quot;prometheus&quot;
    static_configs:
      - targets: [&quot;EC2 IP 주소:9090&quot;]

  # Spring Boot 애플리케이션 모니터링 추가
  - job_name: 'spring-actuator'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['EC2 IP 주소:8080']</code></pre><p>이제 메트릭 수집을 위해 프로메테우스를 백그라운드로 실행해 줍니다.
<code>nohup ./prometheus &gt; prometheus.log 2&gt;&amp;1 &amp;</code>
올바르게 실행되고 있는지 확인합니다. 
<code>ps aux | grep prometheus</code></p>
<p>이제 9090포트로 접속을 하면 아래 사진과 같이 프로메테우스 UI를 볼 수 있습니다. 
<img alt="" src="https://velog.velcdn.com/images/leegarden/post/d0c0a51b-842b-433b-9acd-d7fa11d9687c/image.png" /></p>
<p>그리고 상태를 확인해야 하기에, <code>Status - Targets</code> 에 들어가 모두 UP이 되어있는지 확인해 줍니다. </p>
<h1 id="3-grafana-설치">3. Grafana 설치</h1>
<p>이제 그라파나를 설치 및 실행해 줍니다. </p>
<pre><code>cd ~

# Grafana 저장소 추가를 위한 패키지들 설치
sudo apt-get install -y apt-transport-https software-properties-common wget

# Grafana GPG 키 추가
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg &gt; /dev/null

# Grafana 저장소 추가
echo &quot;deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main&quot; | sudo tee /etc/apt/sources.list.d/grafana.list

# 패키지 목록 업데이트
sudo apt-get update

# Grafana 설치
sudo apt-get install grafana

# Grafana 서비스 시작 및 부팅 시 자동 시작 설정
sudo systemctl start grafana-server
sudo systemctl enable grafana-server</code></pre><p>올바르게 설치가 되면 3000 포트로 접속시 아래와 같이 그라파나 UI 로그인 페이지에 접속하게 됩니다. 
username, password 동일하게 admin으로 로그인 가능하며 이후에 수정할 수 있습니다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/8b16c336-9e08-475d-aa64-cd9b382b3b89/image.png" /></p>
<p>만약 접속이 안 된다면 다음 명령어를 통해 그라파나 접속 상태를 확인할 수 있습니다. 
<code>sudo systemctl status grafana-server</code> </p>
<h1 id="4-prometheus--grafana-연결">4. Prometheus &amp; Grafana 연결</h1>
<p>이제 설치 및 접속은 모두 끝났으니 프로메테우스에서 수집한 메트릭을 그라파나에서 모니터링 할 수 있도록 연결을 해보겠습니다. </p>
<p>스프링과 프로메테우스 연결은 <code>prometheus.yml</code> 에서 했듯이, 프로메테우스와 그라파나 연결은 그라파나에 접속해서 해줍니다. </p>
<h2 id="4-1-data-source-추가">4-1. Data Source 추가</h2>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/b4556a07-17a2-4aa8-9cb1-0d0bd83b0c16/image.png" />
로그인 후 데이터 소스 추가를 클릭하여 프로메테우스를 선택해 줍니다. 
해당 화면을 통해 다양한 데이터 베이스와 사용이 가능하다는 것을 알 수 있었습니다. </p>
<h2 id="4-2-url-설정">4-2. URL 설정</h2>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/a7ec98b9-e621-41ac-b10e-3f94112edd73/image.png" /></p>
<p>이제 Connection에 <code>EC2 IP 주소:9090</code>을 작성하여 프로메테우스와 연결해주면 됩니다. </p>
<blockquote>
<p>해당 작업을 하면서 굉장히 많은 오류와 어려움이 있었는데 아래 게시글에서 원인과 방법들을 정리했습니다. 
<a href="https://velog.io/@leegarden/Monitoring-Prometheus-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0-expected-a-valid-start-token-got-invalid-while-parsing">https://velog.io/@leegarden/Monitoring-Prometheus-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0-expected-a-valid-start-token-got-invalid-while-parsing</a></p>
</blockquote>