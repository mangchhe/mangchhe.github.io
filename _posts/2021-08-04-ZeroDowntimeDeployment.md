---
title: "[DevOps] 무중단 배포에 대해서 알아보고 Nginx와 함께 간단한 실습 예제 구현"
description: 무중단 배포의 종류에 대해서 배워보고 Spring Boot + Nginx를 이용해 블루 그린 배포를 실습해보자
categories:
 - DevOps
tags:
 - Spring Boot
 - Rolling
 - Blue-Green
 - Canary
 - 무중단 배포
 - Nginx
 - Deployment
---

# 무중단 배포?

<hr>

무중단 배포는 말 그대로 서비스를 중단하지 않고 배포를 할 수 있는 것을 의미한다. 이것이 왜 필요할까?

일반적인 배포는 기존 서비스 연결을 끊고 개발된 새로운 서비스로 교체를 한 뒤에 문제가 생기지 않았다면 연결시켜서 서비스를 제공하는 방식이다. 그러면 연결을 끊고 다시 제공하기까지의 공백의 시간이 생기게된다.

만약 우리가 매일 같이 사용하는 카카오톡이 4시간동안 점검을 한다고하여 해당 서비스를 이용하지 못한다고 생각해보자. 서로 실시간으로 대화를 주고 받기 위해서 시작했는데 점검이라서 이용하지 못한다면 사용자들 입장에서는 답답할 것이다. 새로운 서비스를 추가할 때마다 최소 몇분에서 몇시간을 기다려야 한다면 아마 서비스 이용자들은 다른 서비스를 찾아 떠나 이탈이 생겨 회사에는 경제적인 부분에 대해서 타격을 입을 것이다.

특히나 요즘과 같은 개발 트렌드는 폭포수에서 애자일 방식으로 변화하면서 비즈니스 요구사항을 받아들여 빠르게 개선해나고 충족시키기 위해서 릴리즈 주기가 많이 줄어들었는데 그때마다 매번 서비스를 멈출 수는 없는 노릇이기 때문에 무중단 배포를 할 필요가 있다.

설명할 무중단 배포의 종류는 세가지이다.

1. Rolling Deployment
2. Blue-Green Deployment
3. Canary Deployment

아래에서 하나씩 알아보도록 하자.

# 롤링 배포(Rolling Deployment)

<hr>

롤링 배포는 현재 사용 중인 인스턴스 내에서 새 버전으로 점진적으로 하나씩 교체해나가는 것이다.

![rollingDeployment](/assets/postImages/ZeroDowntimeDeployment/rollingDeployment.JPG)

이 방식을 이용하게 된다면 기존에 유지하고 있는 인스턴스로만 진행하기 때문에 추가적으로 비용이 발생하지않는다. 하지만 새로운 버전으로 교체하는 과정에서 일부 인스턴스가 연결되어 있지 않기 때문에 트래픽을 감당할 수 있는 선에서 진행해야하며 이전 버전과 새로운 버전의 호환성 문제도 생각해야 한다.

# 블루-그린 배포(Blue-Green Deployment)

<hr>

블루그린 배포는 블루는 이전 버전, 그린은 새로운 버전으로 지칭하며 구버전과 동일한 인스턴스 환경을 가진 새로운 버전을 만들어 로드 밸런싱에 연결 시키고 이전 버전을 해제시키는 방식이다.

![blueGreenDeployment](/assets/postImages/ZeroDowntimeDeployment/blueGreenDeployment.JPG)

이 방식은 이전 버전과 새로운 버전의 호환성이나 트래픽 문제는 해결이 됐지만 자원을 두배로 소모한다는 단점을 가지고 있다.

# 카나리 배포(Canary Deployment)

<hr>

카나리 배포는 이전 버전의 인스턴스를 새로운 버전의 인스턴스로 점진적으로 바꿔나가는 방식이다.

![canaryDeployment](/assets/postImages/ZeroDowntimeDeployment/canaryDeployment.JPG)

새로운 버전의 제공 범위를 늘려가는 과정 속에서 문제점을 발견하거나 사용자들의 피드백을 받을 수 있다. 하지만 롤링 배포와 마찬가지로 이전 버전과 새로운 버전이 함께 공존하기 때문에 호환성 문제가 생길 수 있다.

# 실습

<hr>

해당 실습은 EC2 Ubuntu 18.04 LTS 환경에서 Spring Boot + Nginx를 이용해 간단한 쉘 스크립트를 작성해 Blue-Green 배포를 해보려고 한다.

시나리오는 기존에 서비스가 8080 또는 8082 포트로 배포되고 있는데 새로운 버전 서비스를 다른 포트로 실행시켜 nginx를 이용해 경로만 새로운 버전으로 이어주고나서 기존 서비스를 해제하는 방식으로 실습을 해나가려고 한다.

``` java
@RestController
@RequiredArgsConstructor
public class TestController {

    private final Environment env;

    @GetMapping("/")
    public String hello() {
        return "현재 실행 중인 서비스의 포트는 : " + env.getProperty("server.port");
    }

}
```

Spring Boot를 이용해 정상적으로 서비스가 변경된 것을 확인하기 위해서 포트를 반환하는 간단한 API 예제를 구현한다.

``` bash
# /etc/nginx/sites-enabled/<>.conf
server {
        listen 80;
        listen [::]80;

        server_name <호스트 네임>;

        include /etc/nginx/conf.d/service-url.inc; # 추가된 부분
        location / {
                # proxy_pass http://node-app;
                proxy_pass $service_url; # 변경된 부분
        }
}

# /etc/nginx/conf.d/service-url.inc
set $service_url http://127.0.0.1:8080;
```

conf.d 폴더에 파일을 만들어 서비스 경로를 환경 변수로 셋팅하고 nginx 설정 파일에 include로 추가해준다. 이런 식으로 해주는 이유는 설정 파일과 같은 경우는 하드 코딩하는 방식으로 관리하기에는 유지보수 측면에서 좋지 않기 때문에 다음과 같은 방식으로 진행한다. 참고로 OS 전역변수로 설정해서 참조하는 방식으로 하면되지 않냐라고 할 수 있는데 nginx에서 그것을 지원하지 않기 때문에 다음과 같이 include해주어야 한다.

``` bash
# git pull
# chmod 755 gradlew
# ./gradlew build
# cp ./build/libs/<프로젝트명>.jar .

serviceA=$(sudo netstat -nltp | grep 8080 | awk '{print $7}' | awk -F '/' '{print $1}')
serviceB=$(sudo netstat -nltp | grep 8082 | awk '{print $7}' | awk -F '/' '{print $1}')

if [ "$serviceA" = "" ]; then
        echo "8082 포트가 존재하여 8080 포트 실행"
        sudo nohup java -jar -Dserver.port=8080 <프로젝트명>.jar &
else
        echo "8080 포트가 존재하여 8082 포트 실행"
        sudo nohup java -jar -Dserver.port=8082 <프로젝트명>.jar &
fi

echo "서비스 구동 대기"

var=1
while [ 1 ]
do
        if [ -n "$(sudo netstat -nltp | grep 8080)" ] && [ -n "$(sudo netstat -nltp | grep 8082)" ]; then
                echo "서비스 구동 완료"
                break
        fi
        echo wait $var ...
        var=`expr $var + 1`
        sleep 1s
done

## 이 부분에 nginx 설정 변경 소스 필요

if [ "$serviceA" = "" ]; then
        echo "8082 포트 프로세스 죽이기"
        sudo kill -9 $serviceB
else
        echo "8080 포트 프로세스 죽이기"
        sudo kill -9 $serviceA
fi
```

위 쉘 스크립트는 현재 실행중인 서비스의 포트 번호를 확인하여 둘중 하나를 실행시키고 삭제하는 스크립트이다.

``` bash
echo "경로 설정"
if [ "$serviceA" = "" ]; then
        echo "8082 -> 8080 경로 변경"
        echo "set \$service_url http://127.0.0.1:8080;" | sudo tee /etc/nginx/conf.d/service-url.inc
else
        echo "8080 -> 8082 경로 변경"
        echo "set \$service_url http://127.0.0.1:8082;" | sudo tee /etc/nginx/conf.d/service-url.inc
fi

sudo service nginx reload
```

위 내용은 아까 nginx 설정을 동적으로 변경하기 위해서 필요한 소스이며 이전 소스에 nginx 설정 변경 소스 필요 부분에 들어가면 된다.

![blueGreenSource](/assets/postImages/ZeroDowntimeDeployment/blueGreenSource.JPG)

합치면 위와 같은 소스가 나오게 된다.

![blueGreenExample](/assets/postImages/ZeroDowntimeDeployment/blueGreenExample.JPG)

쉘 스크립트가 완성되었으니 실행시키게 되면 새로운 서비스를 생성하고 이전 서비스를 제거하고 라우팅 설정까지 진행이 되며 요청을 보내게 되면 올바르게 버전이 변경된 것을 확인할 수 있다.

# Reference

<hr>

- [https://www.samsungsds.com/kr/insights/1256264_4627.html](https://www.samsungsds.com/kr/insights/1256264_4627.html)
