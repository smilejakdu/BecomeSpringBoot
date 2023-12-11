# 깃허브 액션 스크립트 CD

CD 는 통합적 배포인다. 앞서 우리는 CI 를 통해서 코드를 통합하는것을 봤고 ,

CD를 통해서 통합한 코드를 배포를 해야한다.

준비해야할것은 AWS 가입이 돼있어야한다.



CI 를 구현하기 위해 빌드를 하게되는데&#x20;

build -> libs 폴더안에 보면

<figure><img src="../.gitbook/assets/스크린샷 2023-12-11 오후 10.43.36.png" alt=""><figcaption></figcaption></figure>

위와 같이 돼있다.

두개의 파일이 존재하는데

하나는 일반 jar 파일이고 , 다른 하나는 plain 이라는 접미사가 붙은 jar 파일이다.

jar파일은 애플리케이션 실행에 필요한 의존성을 포함하지 않고 소스 코드의 클래스 파일과 리소스 파일만 포함하고 있다. 따라서 plain 파일만으로는 서비스를 실행할 수 없다.&#x20;

그러면 굳이 생성할 필요가 없기때문에  build.gradle 파일에서 수정이 가능하다.

```java
jar {
    enables = false
}
```

로 수정해서 jar 파일만 생성하도록 할 수 있다.

```yaml
# 워크플로의 이름 지정
name: CI

# 워크플로가 시작될 조건 지정
on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest # 실행 환경 지정
    # 실행 스텝 지정
    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-java@v3
        with:
          distribution: 'corretto' # Corretto 배포판을 사용하도록 설정
          java-version: '17'   # Java 버전을 17로 설정

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build with Gradle
        run: ./gradlew clean build

      # 현재 시간 정보를 가져오는 액션을 사용
      - name: Get current time
        uses: josStorer/get-current-time@v2.0.2
        id: current-time
        with:
          format: YYYY-MM-DDTHH-mm-ss
          utcOffset: "+09:00"

      # 빌드 결과물의 이름을 artifact라는 이름으로 환경 변수에 저장
      - name: Set artifact
        run: echo "artifact=$(ls ./build/libs)" >> $GITHUB_ENV

      # Beanstalk에 배포하는 액션을 사용
      - name: Beanstalk Deploy
        uses: einaregilsson/beanstalk-deploy@v20
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          application_name: showMeYourAbility
          environment_name: showMeYourAbility-env
          version_label: github-action-${{steps.current-time.outputs.formattedTime}}
          region: ap-northeast-2
          deployment_package: ./build/libs/${{env.artifact}}

```

위처럼 CD 스크립트를 작성한다.

만약 AWS 셋팅이 되어 있지 않다면

push 를 날렸을때 build 에러가 발생하게 된다.

