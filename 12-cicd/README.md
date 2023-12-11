# 12 CICD

빌드부터 배포까지의 과정을 자동화 할 수 있고, 또 잘되는지 모니터링할 수 있습니다.

CI는 지속적 통합, CD는 지속적 제공이라는 의미가 있습니다.

* 지속적 통합: 깃허브와 같은 코드 저장소에 자동으로 업로드 하는 과정
* 지속적 배포: AWS와 같은 배포 환경으로 보내는것을 의미



## 깃허브 연동

깃허브를 설치한다.

git --version -> git version 이 나온다면 제대로 설치한것이다.

이후 email 과 username 을 셋팅한다.

```markdown
git config --global user.name "username"
git config --global user.email "user email"
```

SSH 로 접속하기 위해서 인증 정보를 등록하기로 하자.&#x20;

PC마다 별도의 SSH 키를 등록해야합니다.

```markdown
ssh-keygen -t rsa -C "git user email"
```



<figure><img src="../.gitbook/assets/스크린샷 2023-12-11 오후 12.07.32.png" alt=""><figcaption></figcaption></figure>

위의 키 값을 github 에 들어가면 settings 라고 보일것이다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-11 오후 12.08.23.png" alt=""><figcaption></figcaption></figure>

에 들어가면 New SSH Key 버튼을 클릭해서 등록해주면 된다.

