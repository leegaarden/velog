<p>인스턴스 설정을 모두 마쳤으면 먼저 수동 배포를 통해 정상 배포되는 것을 확인하고, jenkins 배포를 할 거다. </p>
<h1 id="ec2-수동-배포">EC2 수동 배포</h1>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/e653f83e-c88b-40b1-b143-7a877335de87/image.png" /></p>
<h1 id="1-java-및-git-설치">1. Java 및 Git 설치</h1>
<h2 id="11-java-설치">1.1 Java 설치</h2>
<p>프로젝트의 jdk 버전에 맞게 설치를 해주는데 나는 17 버전으로 설치를 했다. </p>
<p><code>sudo apt update</code> 를 먼저 해 최신화를 시켜준다. 
<code>sudo apt install openjdk-17-jdk -y</code> 로 설치하면 끝이다. (시간은 소요된다.)</p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/a3d255b6-4854-4acd-8268-4dde9bc478bc/image.png" /></p>
<p>다 됐으면 <code>java -version</code>으로 올바르게 설치가 됐는지까지 확인해 주면 된다.</p>
<h2 id="12-git-설치">1.2 Git 설치</h2>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/3b1a0fd4-58f3-426d-bfce-368ff9061eb0/image.png" /></p>
<p><code>sudo apt-get install git</code></p>
<p><code>git --version</code>
을 통해 설치와 버전 확인을 할 수 있다. </p>
<h2 id="13-ssh-키-발급">1.3 ssh 키 발급</h2>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/e453d7fc-211d-4f6e-bb00-52afd55107e8/image.png" /></p>
<pre><code>cd ~/.ssh
ssh-keygen -t rsa -C 깃허브메일</code></pre><p>깃 클론을 받기 위한 ssh 키를 발급 받는다. 
이 키는 깃 클론에만 사용되는 게 아니라, 이후에 있을 젠킨스 인스턴스 연결에서도 사용된다. </p>
<p><code>cat id_rsa.pub</code> 를 통해 공공키를 출력하고 이를 복사해서 깃허브 설정을 통해 넣어주면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/cf8803d6-dc95-4d15-9e02-e0da6a2e0e3b/image.png" /></p>
<p>GitHub -&gt; Settings -&gt; SSH and GPG keys 에 새롭게 등록해 준다. </p>
<h2 id="14-git-clone-받기">1.4 git clone 받기</h2>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/d7c71dbf-96d6-4487-a6d1-73109bf56c8b/image.png" /></p>
<p>이제 깃 클론을 받을 수 있다. Code에서 HTTPS가 아닌 SSH를 클릭하여 주소를 복사한 뒤 인스턴스에 <code>git clone 복사한 값</code>    을 넣어주면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/9149bd52-4b3f-4988-8108-2fa76dc7c5a6/image.png" /></p>
<p>이렇게 정상적으로 클론을 받은 것을 확인할 수 있다. </p>
<p>이대로 빌드와 배포를 하면 될 것 같지만, 우리는 암복호화도 해야 하고 가상 메모리도 늘려줘야 한다. </p>
<h2 id="15-인스턴스-암복호화">1.5 인스턴스 암복호화</h2>
<p><code>sudo nano .bashrc</code>를 하면 설정 파일이 열린다. 
가장 아래에 <code>export DB_USER = 값</code> 이런 식으로 스프링부트에서 암호화 했던 값을 넣어주면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/a26ec0c7-30a4-4d6a-b96b-57f656e1ccc0/image.png" /></p>
<p><code>source .bashrc</code>를 통해 설정한 값들을 적용 시켜주고 <code>echo $변수명</code> 으로 잘 적용이 됐는지 한 번 더 확인한다. </p>
<h2 id="16-가상-메모리-설정">1.6 가상 메모리 설정</h2>
<p>가상 메모리 설정을 하는 이유는, 우리는 프리티어용인 t3.micro 버전의 인스턴스 이기 때문에 빌드 중간에 멈춤 현상이 발생한다. </p>
<p>이를 방지하기 위해 가상메모리로 메모리를 늘려줘야 한다. </p>
<blockquote>
<p>출처: <a href="https://diary-developer.tistory.com/32">https://diary-developer.tistory.com/32</a> [일반인의 웹 개발일기:티스토리]</p>
</blockquote>
<h3 id="161-swapfile-메모리-할당">1.6.1 swapfile 메모리 할당</h3>
<p>스왑 메모리는 램 메모리의 2배 또는 그 이상을 추천하기 때문에 스왑 메모리를 2GB로 설정했다. 
<code>sudo dd if=/dev/zero of=/swapfile bs=128M count=16</code></p>
<h3 id="162-swapfile-권한-설정">1.6.2 swapfile 권한 설정</h3>
<p>읽기와 쓰기가 가능하도록 파일의 권한을 수정해 줘야 한다. 
<code>sudo chmod 600 /swapfile</code> </p>
<h3 id="163-swap-공간-생성">1.6.3 swap 공간 생성</h3>
<p>이제 스왑 공간을 생성해 준다. 
<code>sudo mkswap /swapfile</code> </p>
<h3 id="164-swapfile-스왑-메모리-추가">1.6.4 swapfile 스왑 메모리 추가</h3>
<p>스왑 공간에 swapfile을 추가한다. 
<code>sudo swapon /swapfile</code> </p>
<p><code>sudo swapon -s</code> 명령어를 통해 스왑으로 사용하는 파일의 경로 및 이름, 타입, 크기, 사용 중인 부분, 우선 순위등이 보이는지 확인한다. </p>
<h3 id="165-swap-파일-시스템-설정">1.6.5 swap 파일 시스템 설정</h3>
<p>시스템 부팅 시마다 자동으로 활성화 되도록 파일 시스템을 수정한다. </p>
<p><code>sudo nano /etc/fstab</code> 을 통해 연 파일에 <code>/swapfile swap swap defaults 0 0</code>을 추가하고 저장한다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/ff765f2c-af14-4e8e-92bd-07774107fe77/image.png" /></p>
<h3 id="166-메모리-상태-확인">1.6.6 메모리 상태 확인</h3>
<p><code>free</code> 명령어로 ec2 메모리 상태를 확인한다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/52c7c132-0398-471b-8c43-483b2b35d6b0/image.png" /></p>
<p>마지막 라인인 Swap: 부분을 보면 잘 되어 있는 걸 확인할 수 있다. </p>
<h3 id="167-명령어-정리">1.6.7 명령어 정리</h3>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/ecbaaf29-8444-406e-9ab1-6f35c9e92f7f/image.png" /></p>
<pre><code> sudo dd if=/dev/zero of=/swapfile bs=128M count=16
 sudo chmod 600 /swapfile
 sudo mkswap /swapfile
 sudo swapon /swapfile
 sudo swapon -s
 sudo nano /etc/fstab 
 파일에 /swapfile swap swap defaults 0 0  작성 
 free</code></pre><h1 id="2-수동-배포">2. 수동 배포</h1>
<pre><code>cd Server (프로젝트 폴더 명)
chmod +x gradlew
./gradlew clean build -x test

cd build/libs
java -jar 프로젝트명-0.0.1-SNAPSHOT.jar </code></pre><p>위 명령어로 빌드와 배포를 간단하게 할 수 있다. 백그라운드 배포가 좋지만 지금은 배포가 인스턴스에서 되는지만 확인하는 것이기에 포그라운드 배포를 진행하였다.</p>
<p>백그라운드 명령어: <code>nohup java -jar 프로젝트명-0.0.1-SNAPSHOT.jar &amp;</code></p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/86280b41-d41d-4aeb-8f9a-2c37c928a382/image.png" /><img alt="" src="https://velog.velcdn.com/images/leegarden/post/4b92d7ec-d262-4134-a872-9ba8c8938fa4/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/86537da7-ff8e-4842-a536-0ec26455fcc5/image.png" /></p>
<p>이제 배포도 정상적으로 잘 되었기에 스웨거에 접속을 해서 다시 확인해주면 수동 배포는 이렇게 마무리 된다. </p>
<hr />
<p>이제 이렇게 익힌 감으로 jenkins ci/cd를 진행해 보려고 한다. </p>
<h1 id="jenkins-cicd">Jenkins CI/CD</h1>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/306d6da2-254b-4ee6-a24d-42847bc22f1d/image.png" /></p>
<p>이런 구조의 CI/CD를 진행할 건데, 인스턴스는 하나 이기에 단일 배포이다. </p>
<h1 id="1-jenkins-설치">1. Jenkins 설치</h1>
<p>jenkins 설치를 위해서는 java 설치가 요구되는데, 우리는 이전에 미리 jdk를 설치했으니 jenkins만 잘 설치하면 된다. </p>
<h2 id="11-key-다운로드-및-추가">1.1 key 다운로드 및 추가</h2>
<p> <code>curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc &gt; /dev/null</code></p>
<p>  <img alt="" src="https://velog.velcdn.com/images/leegarden/post/8763fcc8-e5a7-45a1-868e-e1325e69576d/image.png" /></p>
<h2 id="12-jenkins-저장소-추가">1.2 jenkins 저장소 추가</h2>
<p>  <code>echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list &gt; /dev/null</code></p>
<p>  <img alt="" src="https://velog.velcdn.com/images/leegarden/post/5bcfa1c7-1c9b-4575-9fff-09819d156304/image.png" /></p>
<h2 id="13-apt-업데이트-및-jenkins-설치">1.3 apt 업데이트 및 jenkins 설치</h2>
<p>항상 ec2 ubuntu 환경에서 무언 가를 설치할 때는 update를 하는 게 좋다. 
그리고 젠킨스는 용량이 커서 시간이 꽤나 오래 걸린다. </p>
<pre><code>sudo apt update
sudo apt install jenkins -y</code></pre><h2 id="14-jenkins-port-번호-변경">1.4 jenkins port 번호 변경</h2>
<p>스프링 서버 포트도 8080인데 jenkins도 8080이어서 포트 번호를 변경해 줘야 한다. 
앞 포스트에서 7070으로 보안그룹을 열어줬기에 맞춰서 수정해주면 된다. </p>
<p><code>sudo nano /etc/default/jenkins</code></p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/e4d3e18c-62d1-493c-8134-962fb0e7920c/image.png" /></p>
<p>위 파일을 열어 포트 번호를 수정해 주면 된다. </p>
<h2 id="15-jenkins-시작">1.5 jenkins 시작</h2>
<pre><code>sudo systemctl restart jenkins
sudo systemctl enable jenkins</code></pre><p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/9807c4a9-c5e6-49e1-8fde-f283e21828d4/image.png" /></p>
<h2 id="16-접속을-위한-초기-비밀번호-확인">1.6 접속을 위한 초기 비밀번호 확인</h2>
<p><code>sudo cat /var/lib/jenkins/secrets/initialAdminPassword</code> 로 나오는 게 이후 접속해서 입력할 비밀번호이다. </p>
<h2 id="17-jenkins-상태-확인">1.7 jenkins 상태 확인</h2>
<p>이제 위의 과정을 모두 거치고 <code>sudo systemctl status jenkins</code>로 상태가 running인 걸 확인해 주면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/d53f6a19-79e3-473c-9939-90e366d51d31/image.png" /></p>
<p>로그가 찍힐 텐데 ctrl + c 로 나와주면 된다. </p>
<p>여기까지 하면 젠킨스 설치도 끝났다. </p>
<h2 id="🔥트러블-슈팅">🔥트러블 슈팅</h2>
<p>문제: 포트 번호를 7070으로 했는데 접속 시에 8080으로 해야 함. 
원인: 설정 파일이 제대로 읽히지 않았던 것. </p>
<h3 id="1-jenkins-중지">1. jenkins 중지</h3>
<p><code>sudo systemctl stop jenkins</code>
젠킨스 설정을 새로 해줄 거니까 우선 중지를 시킨다. </p>
<h3 id="2-jenkins-설정-파일-찾기">2. jenkins 설정 파일 찾기</h3>
<p><code>find / -name jenkins.war 2&gt;/dev/null</code>
젠킨스 설정 파일을 직접 수정해 줘야 하기 때문에 어떤 경로를 찾아야 한다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/19db11bf-4819-4e25-8e55-43226b44aebe/image.png" /></p>
<h3 id="3-포그라운드로-실행">3. 포그라운드로 실행</h3>
<pre><code>cd /usr/share/java
sudo java -jar jenkins.war --httpPort=7070</code></pre><p>프로그라운드 실행은 시간이 조금 소요되는데, 정상적으로 잘 됐으면 7070 포트로 접속했을 때 젠킨스로 접속이 된다.
그리고 출력은 이와 같다. </p>
<pre><code>**
**
**
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:
초기비밀번호
This may also be found at: /root/.jenkins/secrets/initialAdminPassword
**
**
**
2024-11-28 10:56:45.377+0000 [id=30]    INFO    jenkins.InitReactorRunner$1#onAttained: Completed initialization
2024-11-28 10:56:45.423+0000 [id=23]    INFO    hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up and running
2024-11-28 10:56:47.036+0000 [id=44]    INFO    h.m.DownloadService$Downloadable#load: Obtained the updated data file   for hudson.tasks.Maven.MavenInstaller
2024-11-28 10:56:47.038+0000 [id=44]    INFO    hudson.util.Retrier#start: Performed the action check updates server   successfully at the attempt #1</code></pre><h3 id="4-항상-7070-포트를-사용하도록-설정">4. 항상 7070 포트를 사용하도록 설정</h3>
<p><code>sudo nano /etc/systemd/system/jenkins.service</code> 파일을 열면 비어있을 텐데 아래 내용으로 새로 만들어 주면 된다. </p>
<pre><code>[Unit]
Description=Jenkins Continuous Integration Server
Requires=network.target
After=network.target

[Service]
Type=simple
User=jenkins
Environment=&quot;JENKINS_HOME=/var/lib/jenkins&quot;
ExecStart=/usr/bin/java -Dhudson.model.DirectoryBrowserSupport.CSP=&quot;&quot; -jar /usr/share/java/jenkins.war --httpPort=7070
Restart=always

[Install]
WantedBy=multi-user.target</code></pre><p>파일 저장한 뒤에 <code>sudo chmod 644 /etc/systemd/system/jenkins.service</code>로 권한을 변경해 줘야 한다. </p>
<h3 id="5-서비스-재시작-및-상태-확인">5. 서비스 재시작 및 상태 확인</h3>
<pre><code>sudo systemctl daemon-reload
sudo systemctl enable jenkins
sudo systemctl start jenkins

sudo systemctl status jenkins</code></pre><p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/64e39d71-773f-4809-a68d-fd7384adf6a9/image.png" /></p>
<p>다행히 이렇게 7070 포트를 사용해서 접속할 수 있게 됐다. </p>
<h1 id="2-jenkins-접속">2. jenkins 접속</h1>
<p>이제 http://[instance-IP]:[jenkins-port] 로 접속해 주면 된다. </p>
<p>나의 경우 <a href="http://52.78.96.4:7070">http://52.78.96.4:7070</a> 으로 접속하였다. 
(이 벨로그 이후 해당 인스턴스는 파기했다.)</p>
<p>위에서 받은 초기 비밀번호를 넣어주면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/c4cc5526-05b9-4bf9-a39d-dd399b2b31a0/image.png" /></p>
<p>올바르게 넣었으면 아래 화면이 보이는데 설치하면 된다. 
<img alt="" src="https://velog.velcdn.com/images/leegarden/post/85ffa0a4-290d-47b1-b11a-3301e07f7909/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/911140f5-e79b-4c36-9436-0e7537981f00/image.png" /></p>
<p>재접속 할 때 로그인하게 될 수도 있어서 메모해두는 걸 추천합니다. </p>
<h1 id="3-ec2-jenkins-연동">3. EC2 jenkins 연동</h1>
<h2 id="31-플러그인-설치">3.1 플러그인 설치</h2>
<p>우선 jenkins에서 설치해야 하는 플로그인이 있다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/fc421dff-ce5a-4a31-a218-9aa1996ce4b0/image.png" /></p>
<p>[Plugins] 버튼을 눌러 <code>publish over ssh</code> 를 다운 설치해준다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/b52f45c0-6e77-4d62-b766-672a5fa265f8/image.png" /></p>
<h2 id="32-시스템-설정">3.2 시스템 설정</h2>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/4af888bd-e01c-43ed-8cc3-d3cf7e45d813/image.png" /></p>
<p>시스템 설정에서는 ec2에서 했던 환경변수 설정과 인스턴스 연결을 해줄 거다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/bfc60f1e-61f6-4b6e-b676-b51e7b7c9077/image.png" /></p>
<p>환경변수는 자신의 프로젝트에 맞게 추가해 주면 된다. </p>
<p>이제 ssh 설정을 해줘야 하는데, 가장 맨 아래로 내려가면 Publish over ssh 라는 목차를 볼 수 있다. </p>
<p>초반에 발급받은 ssh 키를 사용해서 진행할 것이다. </p>
<pre><code>cd ~/.ssh 
ls</code></pre><p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/2986a286-b114-46f6-afb7-9d2f631dc748/image.png" />
이동해서 조회를 해보면 이런 키가 뜬다. </p>
<ul>
<li>autorized_keys : 인증 관련 키들이 있는  파일</li>
<li>id_rsa : 개인 키</li>
<li>id_rsa.pub : 공개 키</li>
</ul>
<p>우리는 하나의 인스턴스에서, 젠킨스로 배포하는 거니까 현재 인스턴스의 공개키를 복사해서 autorized_keys에 붙여주면 된다. </p>
<p>_이 과정은 젠킨스 배포 인스턴스가 배포할 인스턴스를 신뢰한다는 의미이다. 
보통은 젠킨스 배포 인스턴스, 서비스 배포 인스턴스를 각각 두니까 이 과정이 잘 이해가 가는데, 나는 하나의 인스턴스에서 배포했기 때문에 처음에 이 과정이 잘 이해가지 않았다. _</p>
<pre><code>cat id_rsa.pub
sudo nano autorized_keys</code></pre><p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/32f3664f-d353-45ad-82e3-77c75defa5a6/image.png" /></p>
<p>이렇게 기존에 맨 윗 줄이 작성되어 있는데 줄바꿈하여 복붙해주고 저장하면 된다. </p>
<p>이제 개인키(id_rsa)를 출력해서 젠킨스에 넣어주면 된다.
<img alt="" src="https://velog.velcdn.com/images/leegarden/post/981be9f6-6e51-4dae-a147-10d8db3e233f/image.png" /></p>
<p>개인키는 공개되지 않도록 유의해야 한다. 
(다시 말하지만 나는 업로드 후 모든 자원을 파기했다.)</p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/6b111d7d-bb9b-40af-8695-b4608c061d42/image.png" /></p>
<blockquote>
<p>Name: 본인이 사용할 임의의 SSH Server의 Name을 입력하면 됩니다.
Hostname: 실제로 접속할 원격 서버 ip, 접속 경로를 입력합니다. ex) 퍼블릭 IPv4 주소: 3.37.87.X
Username: 접속할 원격 서버의 user 이름입니다. ex) ubuntu입니다.
Remote Directory: 원격서버에서 접속하여 작업을 하게 되는 디렉토리 입니다.</p>
</blockquote>
<blockquote>
<p>출처: <a href="https://velog.io/@sa1341/Jenkins%EC%97%90%EC%84%9C-EC2%EB%A1%9C-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0">https://velog.io/@sa1341/Jenkins%EC%97%90%EC%84%9C-EC2%EB%A1%9C-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0</a></p>
</blockquote>
<p>위 단계를 모두 하고 SSH server에 ip주소와 정보들을 입력해주면 된다.
올바르게 입력했을 경우 [Test Configuration] 를 하면 Success가 뜬다. </p>
<p>만약 실패일 경우, 대부분 인증 키를 잘못 한 걸 테니 천천히 다시 해보는 걸 추천한다. </p>
<h1 id="4-github-jenkins-연동">4. GitHub jenkins 연동</h1>
<h2 id="41-github토큰-발급">4.1 GitHub토큰 발급</h2>
<p>우선 깃허브에서 토큰을 받아야 한다. 
GitHub -&gt; Settings -&gt; Developer settings(왼쪽아래) -&gt; Personal access tokens 해당 페이지에서 Generate new token을 클릭하면 토큰 발급을 받을 수 있다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/94e2dca7-0793-4ec3-9346-9a5014aa2c0e/image.png" /></p>
<p>Note(토큰이름)를 적고, repo, admin_hook 을 체크하고 토큰을 생성합니다.
<img alt="" src="https://velog.velcdn.com/images/leegarden/post/0d15e8da-a2f4-4dec-9ef0-85ca597618fa/image.png" /></p>
<p>토큰은 유출되지 않도로 유의하며 다시 확인이 안 되니 메모해둔다. </p>
<h2 id="42-jenkins-credentials-등록">4.2 Jenkins Credentials 등록</h2>
<p>위에서 들어갔던 system 페이지에서 GitHub 부분을 볼 수 있다. 
Server를 추가해준 다음에, Credintials을 아래와 같이 추가해 주면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/158db4ed-09c2-46f7-940b-9126382af854/image.png" /></p>
<p>Kind를 Secret text로 해주고 발급받은 토큰을 넣어주면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/0e4595e4-8b21-4698-9508-47a984313c19/image.png" /></p>
<p>그리고 Credentials를 none에서 추가한 인증으로 바꿔주면 테스트를 성공한다. </p>
<h1 id="5-jenkins-배포-프로젝트-생성">5. jenkins 배포 프로젝트 생성</h1>
<p>메인 화면에서 [New Item]을 누르면 방식들을 선택할 수 있다. 
파이프라인이 무중단 배포에 더 적합하지만 나는 단일 배포이기도 하고 무엇보다 파이프라인을 잘 못 적어서 쉘스크립트로 먼저 해보겠다..!</p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/0e9b7f77-cd15-4e71-9968-b6e470bc2921/image.png" /></p>
<p>생성하고 프로젝트에 들어가면 아래와 같은 화면을 볼 수 있고 프로젝트 설정은 [구성]에서 해준다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/f21c3840-55e7-4d86-a88b-2500a2919bcc/image.png" /></p>
<p>배포할 깃허브 url 주소를 넣어준다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/6bc6fec8-4932-487c-a381-fb376661e115/image.png" /></p>
<p>여기서 브랜치 설정이 초기에 master로 되어 있는데, 나는 develop 기준으로 했다. 
(이후에 웹 후크 관련된 부분에서 더 자세하게 알 수 있다.)</p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/c672e5d0-c46c-48a8-97e8-01a6e956ccde/image.png" /></p>
<p>이제 소스 코드 관리를 해줄 건데, Credential을 앞서 깃허브에서 발급한 토큰으로 생성해줬다.
이번에는 위와 다르게 Username with password 형식으로 아래와 같이 생성했다.
username에는 깃허브 username을 입력하면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/a18f84cb-2127-4533-8e2f-65cc88369167/image.png" /></p>
<blockquote>
<p>Kind : Username with password
Username : 깃허브 유저 네임
Password : 깃허브 토큰</p>
</blockquote>
<p>여기까지 하고 [지금 빌드]를 누르면 빌드가 잘 된 것을 확인할 수 있다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/58e4e48e-e698-4820-adaa-0ad14ef7d78f/image.png" /></p>
<p>이제 빌드 후 조치를 해보겠다. </p>
<h1 id="6-jenkins-github-webhook-배포">6. jenkins GitHub webhook 배포</h1>
<p>웹후크 설정을 해주지 않으면 배포할 때마다 젠킨스에 들어가서 빌드 버튼을 눌러줘야 한다.
여간 귀찮은 일이 아니기에 깃허브에서 이벤트가 발생하면 자동으로 배포가 되도록 웹후크 설정을 하고자 한다. </p>
<h2 id="61-github-webhook-token">6.1 GitHub webhook token</h2>
<p>배포하려는 프로젝트의 세팅에 들어가서 웹후크를 추가해 준다. </p>
<p>기존에 있던 거는 디스코드 웹후크이다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/176cef5f-4a1e-4d9c-a410-ad144cdd8c94/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/c56025ef-add1-4b46-8363-2806bb2b7954/image.png" /></p>
<blockquote>
<p>Payload URL : 배포 IP주소와 젠킨스 포트 번호로만 수정하고 url 주소는 건들지 말 것
Content type : application/json</p>
</blockquote>
<p>이렇게만 수정해 주고 생성하면 된다. </p>
<p>이제 빌드 유발을 깃허브 웹후크로 하니까 구성 이와 같이 체크하면 된다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/80334313-150d-4d09-a88c-5da87bd3228f/image.png" /></p>
<h3 id="빌드-조치">빌드 조치</h3>
<p>나는 쉘 스크립트로 작성할 거니까 Execute shell 을 선택해 준다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/d8bf208a-9ee1-4fe5-9962-df74dd063005/image.png" /></p>
<p>이제부터는 위에서 수동 배포했던 것과 같은 개념으로 진행된다. </p>
<pre><code>chmod +x gradlew
./gradlew clean build -x test</code></pre><p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/52814823-58f6-434b-9ab8-cc43b93fc3fb/image.png" /></p>
<p>이제 빌드를 했으니까 빌드 후 조치에 대해 작성을 해줘야 한다.
우리는 인스턴스에 ssh로 접속을 하니까 <code>Send build artifacts over SSH</code>를 선택해 주면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/8d12cb81-f0c5-43ba-8ff5-f591b4dda07a/image.png" /></p>
<p>우리는 접속한 인스턴스가 하나 밖에 없기에 서버는 고정이고, [고급] 탭에서 Verbose output in console을 체크해 주면 된다.</p>
<p>이 기능은 콘솔에 진행 과정을 보이겠다는 의미이다. 이걸 쓰면 빌드 오류 시에 콘솔에서 확인할 수 있기 때문에 체크를 하는 게 좋다 </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/6ad8875d-3aac-45ac-ba14-507de676623b/image.png" /></p>
<p>인스턴스를 골랐다면 이제 어떤 파일을 보낼지 설정해줘야 한다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/db477c24-8dc0-4e42-aa7c-a685f7c94017/image.png" /></p>
<blockquote>
<p>Source files : 빌드 후 생성되는 war 파일의 경로
Remove prefix : 전송 시 제거할 경로 부분
Remote directory : 배포 디렉토리 경로로 보통 프로젝트 이름
Exce command : 배포 후 실행할 명령어들</p>
</blockquote>
<pre><code>cd ~/Server &amp;&amp; \
ps -ef | grep java | grep -v grep | awk '{print $2}' | xargs -r sudo kill -9; \
sleep 5; \
sudo nohup java -jar perfuinder-0.0.1-SNAPSHOT.war &gt; output.log 2&gt;&amp;1 &amp;</code></pre><h3 id="🔥트러블-슈팅-1">🔥트러블 슈팅</h3>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/7cfd786f-3fa5-4e9e-b85e-be7ac1658b1a/image.png" /></p>
<p>이렇게 파일 전송은 되는데 포트를 죽이는 게 권한 문제로 인해 되지 않았었다. 
<img alt="" src="https://velog.velcdn.com/images/leegarden/post/c7c1f8db-02db-48ba-8c85-1eb20789c0fe/image.png" /></p>
<p>그래서 Exce Command를 위와 같이 수정해 주니 잘 됐다. 
아마 <code>sudo</code> 때문인듯 싶다. </p>
<h1 id="마무리">마무리</h1>
<p>깃허브 develop 브랜치에서 수정을 하니, [지금 빌드]를 누르지 않았는데도 빌드가 들어간 것을 확인할 수 있다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/40fe4abd-d838-4b71-9e6b-b60e7c37c4a6/image.png" /></p>
<p>콘솔을 확인해 보니 빌드가 되고 있다.
<img alt="" src="https://velog.velcdn.com/images/leegarden/post/2e2614ad-0a1a-458b-b33a-956bbd918585/image.png" /></p>
<p>그리고 화면을 테스트 해보고자 스웨거에 접속하니 아래와 같이 잘 됐다. 
<img alt="" src="https://velog.velcdn.com/images/leegarden/post/3fb3a948-6e6a-4f2f-befe-bb61f496121a/image.png" /></p>
<hr />
<p>이렇게 webhooks 까지 곁들인 CI/CD 정리를 해봤다. </p>
<p>나중에 하게될 일이 생겼을 때 참고하려고 세세하게 적긴 했는데 내용이 생각보다 많다.
그럼 끝. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/8a721a1d-331c-4da7-89d4-c81c939e4abf/image.png" /></p>