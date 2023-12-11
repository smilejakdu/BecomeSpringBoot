# 깃허브 액션 스크립트 CI

지속적 통합을 하기위해서 .github 폴더안에 workflows 폴더를 생성한다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-11 오후 8.20.21.png" alt=""><figcaption></figcaption></figure>

생성하고 난이후 cicd.yml 파일을 생성한다.

```markdown
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

```

위와 같이 작성하고 난뒤에 git push 를 한다.&#x20;

지속적 통합을 하기위해선 테스트 코드들이 정상적으로 works 하도록 작성해야 한다.

만약 테스트 코드가 통과 되지 않는다면 에러가 발생한다.



<figure><img src="../.gitbook/assets/스크린샷 2023-12-11 오후 8.26.29.png" alt=""><figcaption></figcaption></figure>

테스트 코드가 통과되게 수정이 됐으면 다시 main branch 에서 push 를 날린다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-11 오후 8.27.35.png" alt=""><figcaption></figcaption></figure>

정상적으로 push 가 됐다.

