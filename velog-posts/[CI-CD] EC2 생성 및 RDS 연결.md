<p>AWS에서 EC2와 RDS 설정을 하는 방법을 정리했다. 토이프로젝트 배포용이었기 때문에 프리티어로만 진행을 했다.</p>
<h1 id="1-ec2">1. EC2</h1>
<p>배포를 위한 EC2 먼저 생성을 해주는데, 나는 순서를 인스턴스 먼저 생성해주며 이 과정에서 보안그룹까지 새로 생성해서 진행을 해줄 예정이다. </p>
<h2 id="11-인스턴스-생성">1.1 인스턴스 생성</h2>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/24746252-50aa-4997-9d69-058951c8255e/image.png" /></p>
<p>인스턴스의 적절한 이름과 OS만 ubuntu로 설정해준 후 추가로 설정할 건 없다. </p>
<p>그래도 프리티어로 설정이 되어 있는지 확인해 주면 좋다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/fff6fa31-b4f7-4f07-aad4-c7bc20ed2b96/image.png" /></p>
<p>원래는 프로젝트 별로 키 페어를 생성해 줬는데 이번 계정은 연습을 위한 계정이기 때문에 cloud-key 하나로 관리를 해보려고 한다. (비추)</p>
<p>나는 주로 putty로 인스턴스에 접속하기 때문에 .ppk 형식으로 생성을 했다. </p>
<p>팀원들에게 공유할 때는 (특히 맥북 팀원에게 공유할 때는 .pem 키로 주는 게 좋기에 원래는 .pem형식으로 다운 받은 후 .ppk로 변환을 했다. 변환은 puttygen 이란 프로그램으로 쉽게 가능하다)</p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/ae45f36a-2a26-43c6-b7e4-6da1bb6597e7/image.png" /></p>
<p>기본이 8로 되어 있는데 30이 최대니까 맞게 수정해준다. </p>
<h2 id="12-보안그룹-설정">1.2 보안그룹 설정</h2>
<p>네트워크 설정에서 [편집] 버튼을 눌러 프로젝트에 맞게 보안그룹을 편집할 수 있다. </p>
<p>나는 기본적으로 열어야 하는 포트들은 미리 열어두는 것을 선호한다. </p>
<p>(나중에 또 편집하려면 은근 귀찮다..)</p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/fe1bb956-e78d-4c49-a7a5-747e5337b8a3/image.png" /><img alt="" src="https://velog.velcdn.com/images/leegarden/post/9d31ed7d-1679-44c8-a717-b648df971787/image.png" /><img alt="" src="https://velog.velcdn.com/images/leegarden/post/42e6d2cf-8024-4e46-aa8f-653572ba9632/image.png" /></p>
<p>인스턴스 접속은 ssh로 접속하기 때문에 ssh나 http, https는 미리 열어두었다. </p>
<p>그리고 spring boot 기본 포트인 8080과 jenkins 포트를 7070으로 설정할 것이기에 7070과 mysql 포트인 3306도 함께 열어두었다. </p>
<p>jenkins도 원래 8080 포트여서 jenkins 설치 이후에 수정을 해줘야 한다. </p>
<p>원래 소스 유형은 지정된 ip에서만 허용하는 게 좋은데 빠르게 배포하고자 그냥 위치 무관으로 했다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/7f44862f-fa9c-49d0-bb67-0a6eeed36316/image.png" /></p>
<p>시간이 좀 지나면 이렇게 정상적으로 실행 중인 걸 볼 수 있다. </p>
<h2 id="13-탄력적-ip-연결">1.3 탄력적 IP 연결</h2>
<p>이렇게 생성한 인스턴스는 고정 IP가 없기 때문에 종료 하고 새로 킬 때마다 매번 주소가 바뀐다. </p>
<p>이를 방지하고 고정 IP를 부여하는 서비스를 탄력적 IP에서 할 수 있다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/a5ccac03-9a0b-433e-af90-81b7d8443ab9/image.png" />단력적 IP는 아주 간단하게 생성할 수 있다. [탄력적 IP주소 할당] 버튼을 누르면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/606c9886-e239-4130-a87e-115d3c130d06/image.png" /></p>
<p>이 화면이 보이는데 아무것도 건들지 않고 [할당] 버튼을 누르면 IP 주소를 새로 받을 수 있다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/f158ba8e-95fb-4308-bdaf-37518a169ccf/image.png" /></p>
<p>새롭게 생성된 주소를 선택하고 위에서 생성한 인스턴스에 연결해주면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/d4ca8006-cde7-49c1-a693-b19ccf140090/image.png" /></p>
<p>running 중인 인스턴스만 선택 가능하다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/7100b8cb-46d8-48b0-8e22-21a7d992bba3/image.png" /></p>
<p>인스턴스 정보에서 확인해 보면 퍼블릭 IP 주소에 잘 연결되어 있다.</p>
<p>이제 이 주소로 인스턴스에 접속해 주면 된다. </p>
<h2 id="14-인스턴스-접속">1.4 인스턴스 접속</h2>
<p>나는 putty를 통해서 접속하는 걸 선호한다. 미리 만들어두면 이후에 접속할 때도 매우 빠르게 접속할 수 있다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/de6139c0-f8be-441e-8b74-8e08201e8fbf/image.png" /></p>
<p>먼저 인스턴스를 생성하면서 함께 생성한 키를 Connection-Auth-Credentials에 넣어준다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/185c2897-3fc5-4c65-b9b7-52aa7efab3ac/image.png" /></p>
<p>그리고 주의할 게 Host Name에 꼭 <code>ubuntu@퍼블릭IP주소</code>를 넣어줘야 한다. </p>
<p>포트는 ssh이기에 자동으로 22번 포트가 적혀있다. 이름만 적절하게 입력해 주고 [save] 버튼을 눌러주면 된다. 이후에 이 aws-practice만 더블클릭하면 쉽게 접속할 수 있는 것이다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/6e5ccb6d-8d31-4434-bba0-f4e300fda41a/image.png" /></p>
<p>[open] 버튼을 누르면 이런 화면이 뜰 텐데 [Accept] 버튼을 누르면 된다. </p>
<p>정상적으로 접근이 됐다는 뜻이다. 이제 여기까지 왔으면 인스턴스 관련된 것은 모두 다 했다. </p>
<h1 id="2-rds">2. RDS</h1>
<h2 id="21-데이터베이스-생성">2.1 데이터베이스 생성</h2>
<p>이제 rds 설정을 위해 데이터베이스를 생성해줘야 한다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/adafb28d-b4dd-4e02-a283-c06b0f6dd46d/image.png" /><img alt="" src="https://velog.velcdn.com/images/leegarden/post/44b0d910-cc80-4b1d-ae09-f03e8c6598fb/image.png" /></p>
<p>rds는 과금에 있어 주의를 해야 하는 자원이기 때문에 꼭!! 프리티어 설정을 해줘야 한다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/4ea7b165-28ea-4084-87e2-d666cdd2c1c9/image.png" /></p>
<p>마스터 사용자 이름도 선호하는 이름으로 수정하는 걸 추천한다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/41bfb1ce-bb95-4a6d-94d0-1a3091c99662/image.png" /></p>
<p>인스턴스 구성 또한 과금을 최소화할 수 있는 t3.micro로 해준다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/e0633491-eda3-44ec-a4b5-b5868f9e9fc8/image.png" /><img alt="" src="https://velog.velcdn.com/images/leegarden/post/ada06c76-f08c-4156-b87e-b80c2fbdd7b6/image.png" /></p>
<p>스토리지도 자동조절을 체크 해제해줘야 과금을 줄일 수 있다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/3a0d6299-6f38-4cd5-b373-f167905ea979/image.png" /></p>
<p>EC2 컴퓨팅 리소스 연결을 안 해야 퍼플릭 엑세스를 허용할 수 있기에 우선은 이렇게만 설정을 해준다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/1533fd2d-d2c3-4f56-a68e-1adf82b6e6f9/image.png" /></p>
<p>앞에서 생성해둔 보안그룹을 선택해줬다. (웬만한 건 열려있으니까)</p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/048b1836-96af-4402-84f1-265b05ccb61a/image.png" /></p>
<p>초기 데이터베이스 이름을 설정해줘야 나중에 연결할 때 편하다. </p>
<p>지금 설정해두지 않으면 연결하려고 할 때 CREATE 해주는 과정이 생기기 때문이다. </p>
<p>그리고 자동 백업을 해제해 줘야 과금을 피할 수 있다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/f20d77b9-33ae-43b8-b5aa-f7e792ca7e50/image.png" /></p>
<p>자동 업그레이드나 삭제 방지 활성화도 모두 해제해 줘야 과금을 피한다. </p>
<p>이렇게 하면 RDS 생성은 끝이다. RDS는 과금을 피하기 위해 해야 하는 설정들이 조금씩 있어서 꼼꼼하게 살필 필요가 있다. </p>
<h2 id="22-파라미터-연결">2.2 파라미터 연결</h2>
<p>여기까지만 하면 한글 설정 및 time zone이 맞지 않아 정상적으로 프로젝트를 진행할 수 없다. </p>
<p>이러한 점은 paramter group을 통해 수정해줄 수 있다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/45c1db7f-8fde-45ae-9394-027c2b7cb3dd/image.png" /></p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/098f076c-86b5-466b-b82c-bc414e93285d/image.png" /></p>
<p>생성했던 데이터베이스 엔진에 맞춰 생성해 주면된다. </p>
<p>이렇게 생성하고 유형에서 DB Paramter Group으로 변경해서 총 두 개의 파라미터 그룹을 생성해 주면 된다. </p>
<p>아래 설정들을 위해 클러스터 파라미터를 수정해 줄 거다. </p>
<h3 id="한글-설정">한글 설정</h3>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/aa0e2f6f-9c7a-44c4-9333-d3ba06c5bb37/image.png" /></p>
<p><code>character_set</code>을 검색해서 나온 6개의 항목에 모두 <code>utf8mb4</code>를 작성해 주면 된다. </p>
<h3 id="한국-시간-설정">한국 시간 설정</h3>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/b50fefce-e2f8-4da6-b36b-c6a1e01c376a/image.png" /></p>
<p>time_zone을 검색해서 Asia/Seoul도 입력해주면 된다. </p>
<p>이렇게 설정한 파라미터를 이제 데이터베이스에 연결해 줘야 한다. </p>
<p>생성된 데이터베이스를 수정하여 추가 구성에서 db-parameter로 수정해주면 된다. </p>
<p><img alt="" src="https://velog.velcdn.com/images/leegarden/post/a712730f-ab32-4a99-b79a-eb7ec09e6e48/image.png" /></p>
<p>이제 엔트포인트를 통해 접속할 수 있다.
<img alt="" src="https://velog.velcdn.com/images/leegarden/post/606fadbf-77b6-4f24-b2da-3a9988d9e9f6/image.png" /></p>