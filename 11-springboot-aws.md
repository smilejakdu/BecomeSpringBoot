# 11 SpringBoot AWS



<figure><img src=".gitbook/assets/스크린샷 2023-12-11 오전 11.03.05.png" alt=""><figcaption></figcaption></figure>

build 를 해서 jar 파일을 업로드하게된다. ec2 인스턴스에서는 was 로 jar 파일로 서버를 운영하게 된다.

ec2 는 하나로 운영할 수 있지만 여러대 운영도 가능하다.&#x20;



<figure><img src=".gitbook/assets/스크린샷 2023-12-11 오전 11.07.15.png" alt=""><figcaption></figcaption></figure>

만약 사용자가 많아지게 되면 , 서버 한대로는 처리가 어렵기 때문에 EC2 서버가 여러대 필요하게 된다.

하지만 계속 서버 여러대 운영하게 되면 비용이 발생하게 된다.&#x20;

그래서 사용자의 요청 횟수에 따라EC2를 늘이거나 줄이게 된다.



사용자의 요청은 분산시킬수 있습니다. 이러한 분산시켜주는 역할을 하는녀석이 로드밸런서 입니다.

로드밸러서는 여러 기능들이 있습니다.&#x20;

요청을 어디로 분산시킬지 그룹을 정해야하는데, 이러한 그룹을 대상그룹이라고 합니다.

## 로드밸런서

AWS (Amazon Web Services)에서 제공하는 로드 밸런서는 인터넷 트래픽을 여러 서버 또는 인스턴스에 분산시키는 역할을 합니다. 이를 통해 어플리케이션의 가용성과 내결함성을 높일 수 있습니다. AWS에서는 주로 세 가지 유형의 로드 밸런서를 제공합니다:

1. **Application Load Balancer (ALB)**:
   * ALB는 HTTP와 HTTPS 트래픽을 처리하는 데 최적화되어 있으며, 고급 라우팅 기능을 제공합니다.
   * 콘텐츠 기반 라우팅을 통해 트래픽을 특정 서비스나 컨테이너에 전달할 수 있습니다.
   * 웹 소켓과 HTTP/2 통신도 지원합니다.
   * ALB는 주로 마이크로서비스와 컨테이너 기반 아키텍처에서 사용됩니다.
2. **Network Load Balancer (NLB)**:
   * NLB는 TCP, UDP, TLS 트래픽에 최적화되어 있으며, 초저지연과 고성능을 제공합니다.
   * 정적 IP 주소 할당이 가능하며, AWS의 VPC 내에서 AWS 리소스나 온프레미스 리소스 간의 연결에 유용합니다.
   * 대규모의 급격한 트래픽 증가에도 빠르게 대응할 수 있습니다.
3. **Classic Load Balancer (CLB)**:
   * CLB는 이전 버전의 AWS 로드 밸런서로, HTTP, HTTPS, TCP 및 SSL 프로토콜을 지원합니다.
   * 간단한 로드 밸런싱 요구사항에 적합하지만, 최신 기능과 최적화는 ALB와 NLB에 비해 제한적입니다.

각 로드 밸런서 유형은 특정 사용 사례와 요구사항에 맞게 설계되었습니다. 예를 들어, 웹 애플리케이션에는 ALB가, TCP 트래픽이 높은 애플리케이션에는 NLB가 적합할 수 있습니다. 로드 밸런서를 통해 트래픽을 분산시킴으로써, 서버 과부하를 방지하고 어플리케이션의 가용성을 높일 수 있습니다. 또한, AWS 로드 밸런서는 자동 확장, 클라우드워치 모니터링, SSL/TLS 암호화와 같은 AWS 서비스와 통합되어 더 강력한 기능을 제공합니다.



AWS 로 운영하다보면 알아야할 개념들이 많습니다.

다 공부하고 운영하기엔 시간이 부족할 수 있습니다.&#x20;

이러한 이유로 AWS 에서는 간단히 배포할 수 있게 엘라스틱 빈스톡을 만들었습니다.



<figure><img src=".gitbook/assets/스크린샷 2023-12-11 오전 11.18.49.png" alt=""><figcaption></figcaption></figure>

이후 책에서 생성 배포 방법에 대해서 알려준다.

책에선 그대로 따라하기엔 정보가 부족하다.

IAM 설정도 해줘야하는데 거기에 대한설명이 없어보였다.

```markdown
 Configuration validation exception: Invalid option specification (Namespace:
'aws:elasticbeanstalk:managedactions', OptionName: 'ManagedActionsEnabled'): You
 can't enable managed platform updates when your environment uses the service-linked
 role 'AWSServiceRoleForElasticBeanstalk'. Select a service role that has the
 'AWSElasticBeanstalkManagedUpdatesCustomerRolePolicy' managed policy.

```

위와 같은 에러를 만날 확률이 높다.

## 엘라스틱 빈스톡 서비스 생성

<figure><img src=".gitbook/assets/스크린샷 2023-12-13 오후 8.05.24.png" alt=""><figcaption></figcaption></figure>



화면으로 가서 애플리케이션을 클릭한다.

애플리케이션 이름을 작성하고,

플랫폼을&#x20;

```markdown
Java
Corretto 17 running on 64bit Amazon Linux 2
3.4.5 (Recommended)
```

를 선택한다.

애플리케이션 코드는 샘플 애플리케이션 코드를 선택한다.



{% embed url="https://stackoverflow.com/questions/51619828/aws-elastic-beanstalk-environment-must-have-instance-profile-associated-with-i" %}

위와 같은 에러가 발생할 수 있다. 위의 에러는

**IAM 인스턴스 프로파일 설정**: 'IAM 인스턴스 프로파일' 또는 'IamInstanceProfile' 옵션을 찾아 해당 필드에 `aws-elasticbeanstalk-ec2-role` (또는 원하는 IAM 역할 이름)을 입력합니다.



위의 에러를 해결하다보면&#x20;

{% embed url="https://dangdangee.tistory.com/entry/AWS-Elastic-Beanstalk-%EC%83%81%ED%83%9C-unknown-%EC%98%A4%EB%A5%98-%ED%95%B4%EA%B2%B0%EB%B0%A9%EB%B2%95" %}

이런 에러도 만날 수 있다.



환경설정이 제대로 되면&#x20;



<figure><img src=".gitbook/assets/스크린샷 2023-12-13 오후 10.25.49.png" alt=""><figcaption></figcaption></figure>



이렇게 나오게 된다.

이제 정리를 하면 ,

이후 업로드를 하면 된다.

<figure><img src=".gitbook/assets/스크린샷 2023-12-13 오후 10.27.40.png" alt=""><figcaption></figcaption></figure>

`./gradlew build` 를 통해서 build 를 진행한다.

<figure><img src=".gitbook/assets/스크린샷 2023-12-13 오후 10.31.04.png" alt=""><figcaption></figcaption></figure>

위와 같은 이미지가 보이게 된다.

plain 파일은 그대로 두고 `demo-0.0.1-SNAPSHOT.jar` 파일만 가져와서 진행한다.



<figure><img src=".gitbook/assets/스크린샷 2023-12-13 오후 10.32.43.png" alt=""><figcaption></figcaption></figure>

배포를 시작한다.

정보에 Deploying new version to instacne(s) 라고 나온다.

<figure><img src=".gitbook/assets/스크린샷 2023-12-13 오후 10.34.22.png" alt=""><figcaption></figcaption></figure>

만약 접근 권한을 셋팅하지않았다면 ,&#x20;



<figure><img src=".gitbook/assets/스크린샷 2023-12-13 오후 11.15.51.png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/스크린샷 2023-12-13 오후 11.16.15.png" alt=""><figcaption></figcaption></figure>

위와 같은 에러가 발생한다.

