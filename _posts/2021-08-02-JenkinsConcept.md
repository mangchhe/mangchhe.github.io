---
title: "Implement CI&CD with jenkins and github"
categories: devops
tags: devops
---

# Jenkins 사용 이유

<hr>

![bored](/assets/postImages/common/bored.jpg)

여러 프로젝트를 진행하면서 배포를 할 때마다 많이 애를 먹었다. 프로젝트를 수정하고 변경사항들을 배포 서버에 적용하기까지 개발하는 시간 못지 않게 많은 시간을 들였던 것 같다.

보통 EC2에서 배포를 하고 나서 변경사항을 적용하기까지는 다음과 같은 과정들을 거쳤다.

``` bash
ssh -i <>.pem 계정명@public IP bash
cd <workspace 폴더>
git pull
chmod 755 gradlew
./gradlew clean
./gradlew build -x test
cd <libs 폴더>
netstat -nltp
kill -9 <이전 버전 서비스 PID>
nohup java -jar <프로젝트명>.jar &
```

이 과정들은 재빌드하기까지 최소한으로 해야될 작업들이고 설정 등 여러 작업들이 추가가 된다면 단순히 오타로 인해 재배포를 해야될 상황이라도 엄청 많은 시간이 소요된다.

하지만 이 방법을 줄이는 방법도 있긴하다. 바로 쉘스크립트를 만들어서 다음 명령어들을 다 적어놓고 사용하는 방법이다. 사실 이 방법도 좀 뒤늦게 알게된 방법이였고 알고나서 정말 신세계였다...

``` bash
// bepo.sh
cd <workspace 폴더>
git pull
chmod 755 gradlew
./gradlew clean
./gradlew build -x test
cd <libs 폴더>
pid=$(ps -eaf | grep chawchaw.jar | grep -v "grep" | awk '{print $2}')
if [ "$pid" = "" ]; then
    echo "not running"
else
    kill -9 $pid
    echo "killed forcefully. (pid : $pid)"
fi
nohub java -jar <프로젝트명>.jar &

// 스크립트 실행
sh bepo.sh
```

다음과 같이 쉘 프로그래밍을 살짝 섞어서 위에서 진행했던 과정들을 쉘 스크립트로 만들어서 스크립트 실행을 하게 되면 이제는 직접 명령어를 칠 필요없이 적어놓았던 루틴대로 알아서 진행되게 될것이다.

이 방법을 이용해서 이전과는 다르게 상당한 발전을 이루어 배포하는 시간을 단축시킬 수 있었지만, 이것도 귀찮다.... 개발하는데에도 바쁜데 배포까지 이러한 노력을 더해야되나... 다른 방법을 찾아보았다.

그래서 이번에 큰맘 먹고 평소에 줄곧 들어왔던 Jenkins를 한번 써보자하여 시작하게 되었다.

# Jenkins란?

<hr>

소프트웨어 개발 시에 CI/CD를 제공하는 소프트웨어이다.

## CI(Continuous Integration)

지속적인 통합을 의미한다. 어플리케이션의 새로운 코드 변경 사항이 정기적으로 빌드 및 테스트 되어 공유 repository로 통합되는 것이다. 보통 어플리케이션 하나를 개발하기 위해서는 다수의 개발자들이 모여서 브랜치를 나누어서 기능 별로 개발하게되고 commit하게 되는데 이러한 자동 빌드 및 테스트 툴이 없다면 빌드, 테스트, 병합을 매 기능마다 수동으로 해줘야하기 때문에 상당히 번거롭게 될 것이다.

## CD(Continuous Deployment)

지속적인 배포를 의미한다. 앞서 말했던 CI, 개발한 내용들이 빌드와 테스트를 거쳐 공유 repository로 통합이 되게 되면 프로덕션으로 즉, 배포(=운영) 서버로 바로 반영되어 제공되어지는 것을 말한다.

## 결론

Jenkins를 이용하면 자동으로 빌드 -> 테스트를 거쳐 공유 repository로 통합되게 되고 통합된 내용들이 자동으로 배포(=운영) 서버로 반영되어 서비스를 제공하게 도와주는 툴이다.

# Jenkins 설치 방법

<hr>

해당 내용은 AWS EC2, Ubuntu 18.04 LTS에서 진행된다.

``` bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | sudo apt-key add -
echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt update
```

다음 명령어를 치고 패키지 정보를 업데이트 한다.

``` bash
GPG error: https://pkg.jenkins.io/debian-stable binary/ Release: The following signatures couldnt be verified because the public key is not available: NO_PUBKEY <키 값>
```

만약 위와 같은 GPU error를 만나게 된다면 에러 메시지 맨 뒤에 키값을 가지고 해결한다.

``` bash
$ sudo apt-key adv --keyserver  keyserver.ubuntu.com --recv-keys <키 값>

Executing: /tmp/apt-key-gpghome.rJM1fpqRPo/gpg.1.sh --keyserver keyserver.ubuntu.com --recv-keys FCEF32E745F2C3D5
gpg: key <키 값>: public key "Jenkins Project <jenkinsci-board@googlegroups.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

다음 명령어를 치게 되면 오류가 해결 될 것이다.

``` bash
sudo apt update
sudo apt install jenkins
sudo service jenkins status
```

![jenkinsError](/assets/postImages/JenkinsConcept/jenkinsError.JPG)

jenkins 실행에 실패했다고 뜨게되는데 이유를 확인하니 java path를 찾지 못했다고 나온다. 당연하다 아직 자바를 설치하지 않았으니 설치한다.

``` bash
apt-cache search jdk # 설치 가능 jdk 검색
sudo apt install openjdk-8-jdk # java 설치
java -version # 자바가 제대로 설치되었는지 확인
```

자바도 제대로 설치하였으니 이제 jenkins를 다시 재구동시켜주면 된다.

``` bash
sudo service jenkins restart
```

![jenkinsSuccess](/assets/postImages/JenkinsConcept/jenkinsSuccess.JPG)

jenkins port는 default로 8080으로 설정되어있고 웹 서버도 보통 8080으로 설정되어있으니 jenkins port를 설정 파일로 가서 9090으로 변경해주고 하는 김에 시간도 한국 시간으로 맞춰준다.

``` bash
sudo vim /etc/default/jenkins
# 마지막에 추가(한국 시간 적용)
JAVA_ARGS="-Djava.awt.headless=true -Dorg.apache.commons.jelly.tags.fmt.timeZone=Asia/Seoul"
```

![jenkinsPort](/assets/postImages/JenkinsConcept/jenkinsPort.JPG)

``` bash
sudo service jenkins restart # jenkins service 재시작
netstat -nltp # 8080 -> 9090 port 변경 확인
```

![jenkinsInstall1](/assets/postImages/JenkinsConcept/jenkinsInstall1.JPG)

주소창에 <public_id>:9090 으로 접속하게되면 다음과 같은 창이 뜨게되고 /var/lib/jenkins/secrets/initialAdminPassword 파일 안에서 최초 비밀번호를 얻어와 작성하고 Continue를 누른다. 접속이 안된다면 aws로 가서 설정한 보안 그룹으로 가서 9090 포트를 열어주어야 한다.

![jenkinsInstall2](/assets/postImages/JenkinsConcept/jenkinsInstall2.JPG)

이 글을 읽는다면 jenkins를 처음 접해보는 상황이라 무슨 플러그인이 필요한지도 모를테니 그냥 jenkins에서 추천하는 왼쪽으로 클릭하고 넘어간다.

![jenkinsInstall3](/assets/postImages/JenkinsConcept/jenkinsInstall3.JPG)

설치가 끝나면 관리자 아이디 생성 창이 뜨면 내용을 작성한 후 Save and Continue를 누른다.

![jenkinsInstall4](/assets/postImages/JenkinsConcept/jenkinsInstall4.JPG)

jenkins url을 설정한 후 Start using Jenkins를 누르고 설치를 완료한다.

# Jenkins 설정 전 준비사항

<hr>

jenkins와 연동할 github 저장소와 Spring Boot를 이용해 간단한 API 예제를 준비한다.

![createGitRepository](/assets/postImages/JenkinsConcept/createGitRepository.JPG)

우선 사용할 깃허브 원격저장소를 private로 생성한다. 나는 예제로 test 이름으로 생성하였다.

``` java
@RestController
public class TestController {

    @GetMapping("/")
    String index() {
        return "First Commit";
    }
}
```

``` bash
git clone https://github.com/<사용자 ID>/test.git
cd test
git add .
git commit -m "First Commit"
git push
```

간단 예제로 API를 하나 만들고 파일들을 clone한 로컬 저장소에 넣고 push한다.

``` bash
sudo dd if=/dev/zero of=/swapfile bs=128M count=32
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon -s
sudo vi /etc/fstab # 열어서 아래 내용 추가
/swapfile swap swap defaults 0 0 # 파일 끝에 추가
free # Mem 말고 Swap 부분이 추가된 것을 볼 수 있다.
```

만약에 자신이 EC2 프리 티어를 쓰고 있다면 위 방식으로 HDD 일정 부분을 RAM처럼 사용하게 해서 메모리를 늘려주어야 한다. 이렇게 하지 않는다면 jenkins에서 빌드를 누르면 cpu가 100%까지 올라가 먹통이 되는 것을 볼 수 있다. 나는 안 그럴 거라고 생각한다면 이 설정을 하지 않고 한번 경험해보는 것도 추천한다.

# Jenkins 프로젝트 생성&빌드

<hr>

![jenkinsConfig1](/assets/postImages/JenkinsConcept/jenkinsConfig1.JPG)

새로운 item을 클릭하고 프리스타일 프로젝트로 아이템을 생성한다.

![jenkinsConfig2](/assets/postImages/JenkinsConcept/jenkinsConfig2.JPG)

빨간색 박스 화살표를 따라 차례대로 체크하고 적용하고자하는 깃 저장소 주소를 입력하고 저장소 소유주의 아이디와 비밀번호를 적어 Credential을 만들어 적용한다.

![jenkinsConfig3](/assets/postImages/JenkinsConcept/jenkinsConfig3.JPG)

원격 저장소에 받아올 브랜치명과 빌드 쉘 명령어를 적어주고 저장을 누르고 jenkins 프로젝트를 생성한다. 생성된 프로젝트에 들어가 빌드가 정상적으로 작동하는지 Build now를 누르고 실행되고 있는 작업번호에 보면 처음이라면 #1 이라고 적혀있을 것이다. 들어가서 Console Output을 눌러 적었던 쉘 명령어가 정상적으로 작동하는지 확인한다.

![jenkinsConfig4](/assets/postImages/JenkinsConcept/jenkinsConfig4.JPG)

만약에 다음과 같이 실패가 뜨게된다면 현재 설치되어있는 jdk는 8버전이고 프로젝트를 빌드하기 위해 필요한 jdk 버전은 11버전이라 그렇다.

``` bash
sudo apt-cache search jdk
sudo apt install openjdk-11-jdk
java -version
```

프로젝트 빌드시 사용되는 jdk 버전 정하는 방법은 두 가지가 있다.

``` bash
$ sudo update-alternatives --config java

There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
* 2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

Press <enter> to keep the current choice[*], or type selection number:
```

첫 번째는 쉘 명령어로 전역으로 jdk 버전을 선택하여 설정하는 방식이다.

![jenkinsConfig5](/assets/postImages/JenkinsConcept/jenkinsConfig5.JPG)

두 번째는 jenkins 설정 내에서 글로벌 설정에서 jdk를 버전을 설정하는 방식이다. 만약 프로젝트 자바 버전과 jenkins 자바 버전이 서로 다르다면 이 방식을 이용하는 것이 좋을 것으로 판단된다.

![jenkinsConfig6](/assets/postImages/JenkinsConcept/jenkinsConfig6.JPG)

설정 후에 다시 Build Now를 클릭해 재빌드하게 되면 성공적으로 빌드가 되는 것을 확인할 수 있다.

# Github 변경 감지 설정

<hr>

Webhook을 설정해서 원격 git 저장소에 변경사항이 생겼을 경우 자동으로 빌드되게 만들어보자

![jenkinsConfig7](/assets/postImages/JenkinsConcept/jenkinsConfig7.JPG)

해당 프로젝트에 들어가서 빨간색 박스 순서대로 진행해주고 추가했을 때 왼쪽에 초록색 체크버튼이 나타나면 된다. 주의해야될 점은 payload URL 맨 마지막에 /로 끝나야 하며 /로 끝나지 않을 경우 빨간색 경고 표시나 나타나게 된다.

옵션을 just the push event로 체크하였기 때문에 push 발생할 시에만 이벤트 감지해 재빌드&배포가 발생되게 되고 추가적으로 다른 것도 감지하고 하려면 Let me select individual events를 클릭하면 선택 항목이 엄청 많으니 필요한 부분을 설정하면 된다.

``` java
@RestController
public class TestController {

    @GetMapping("/")
    String index() {
        return "Second Commit";
    }
}
```

``` bash
git add .
git commit -m "Second Commit"
git push
```

![jenkinsConfig8](/assets/postImages/JenkinsConcept/jenkinsConfig8.JPG)

예제를 수정해서 원격 저장소로 push를 하게 되면 위 사진과 같이 push를 감지해서 설정해놓았던 내용대로 재빌드&배포를 실행하는 것을 확인할 수 있다.

# 빌드 이후 배포 설정

<hr>

빌드는 성공적으로 동작하니 이제 배포를 해보도록 하자

![jenkinsConfig9](/assets/postImages/JenkinsConcept/jenkinsConfig9.JPG)

플러그인 관리에 들어가사 post build task라는 플러그인을 설치한다.

![jenkinsConfig10](/assets/postImages/JenkinsConcept/jenkinsConfig10.JPG)

``` bash
export BUILD_ID=dontKillMe # 없으면 프로세스가 생성되었다가 죽어버림
JENKINS_HOME=/var/lib/jenkins

# ps -a 세션 리더(로그인 쉘)을 제외하고 터미널에 종속되지 않은 모든 프로세스 출력
# ps -e 커널 프로세스를 제외한 모든 프로세스 출력
# ps -f 풀 포맷으로 출력, PID와 같은 내용 추가
# grep "매칭되는 pattern이 존재하는 라인만"
# grep -v "매칭되는 Pattern이 존재하지 않는 라인만"
# awk {print $2} 두번째 필드만 출력
pid=$(ps -eaf | grep <프로젝트명>.jar | grep -v "grep" | awk '{print $2}')

if [ "$pid" = "" ]; then
  echo "<프로젝트명> web application is not running."
else
  echo "<프로젝트명> web application proceess killed forcefully. (pid: $pid)"
fi

cp $JENKINS_HOME/workspace/<jenkins 프로젝트명>/build/libs/<프로젝트명>.jar .
nohup java -jar <프로젝트명>.jar &
```

해당 프로젝트 구성에 들어가 빌드 후 처리에 BUILD SUCCESS 일 때 실행될 쉘 스크립트 내용을 적어준다. 내용은 기존 프로젝트 내용으로 돌고 있는 프로세스를 kill하고 새로 빌드된 서비스를 실행시킨다. 이로써 빌드 후 배포 설정도 끝이 났다.

``` java
@RestController
public class TestController {

    @GetMapping("/")
    String index() {
        return "Third Commit";
    }
}
```

``` bash
git add .
git commit -m "Third Commit"
git push
```

확인해보기 위해서 내용을 변경하고 다시 push 한다.

![jenkinsConfig11](/assets/postImages/JenkinsConcept/jenkinsConfig11.JPG)

push를 감지하고 방금 설정한 배포 스크립트가 시작되어 자동 배포가 되는 것을 확인할 수 있다.

![jenkinsConfig12](/assets/postImages/JenkinsConcept/jenkinsConfig12.JPG)

이렇게 자동 배포가 된 내용들은 변경 사항에 들어가서 쉽게 확인할 수도 있다.

# Reference

<hr>

- [https://hbot.tistory.com/40](https://hbot.tistory.com/40)
- [https://tape22.tistory.com/22](https://tape22.tistory.com/22)
- [https://timevoyage.tistory.com/147](https://timevoyage.tistory.com/147)
- [https://blog.jiniworld.me/94](https://blog.jiniworld.me/94)
