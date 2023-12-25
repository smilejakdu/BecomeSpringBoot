# 깃허브 액션 스크립트 작성 CI



## 깃허브 액션 스크립트 작성 CI

프로젝트 상단에 .github 디렉터리를 만들어준다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-11 오후 6.08.37.png" alt=""><figcaption></figcaption></figure>

```sh
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
          distribution: 'zulu'
          java-version: '17'
          
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew
        
      - name: Build with Gradle
        run: ./gradlew clean build
```

`./gradlew clean build` 하게 되면 그레들로 build 이전 상태로 돌리고 다시 빌드하는 명령어 입니다.

물론입니다. 이 코드는 GitHub Actions을 통해 Continuous Integration (CI) 워크플로를 설정하는 YAML 파일의 일부입니다. 각 줄의 기능을 설명하겠습니다:

1. `name: CI`: 이 워크플로의 이름을 "CI"로 지정합니다.
2. `on: push: branches: [ main ]`: 이 워크플로는 `main` 브랜치에 코드가 푸시될 때마다 실행됩니다.
3. `jobs: build:`: "build"라는 작업을 정의합니다. 이 작업은 워크플로에서 실행할 실제 작업들을 포함합니다.
4. `runs-on: ubuntu-latest`: 이 작업은 최신 버전의 Ubuntu 운영 체제에서 실행됩니다.
5. `- uses: actions/checkout@v3`: GitHub Actions의 `checkout` 액션을 사용합니다. 이는 워크플로가 실행되는 러너에 코드 리포지토리를 체크아웃(복사)하기 위해 필요합니다.
6. `- uses: actions/setup-java@v3`: Java 환경을 설정하는데 사용되는 GitHub Actions의 `setup-java` 액션입니다.
7. `with: distribution: 'zulu'`: 이 액션을 통해 사용될 Java 배포판을 'zulu'로 지정합니다.
8. `java-version: '17'`: 사용할 Java 버전을 '17'로 지정합니다.
9. `- name: Grant execute permission for gradlew`: gradlew 파일에 실행 권한을 부여하기 위한 스텝의 이름입니다.
10. `run: chmod +x gradlew`: 실제로 gradlew 파일에 실행 권한을 부여하는 리눅스 명령어를 실행합니다.
11. `- name: Build with Gradle`: Gradle을 사용하여 프로젝트를 빌드하는 스텝의 이름입니다.
12. `run: ./gradlew clean build`: Gradle 래퍼를 사용하여 프로젝트를 'clean'하고 'build'하는 명령을 실행합니다. 이는 프로젝트를 깨끗하게 정리하고 새로 빌드하는 데 사용됩니다.



위의 CI 는 매우 유용하다 만약에 테스트 코드가 실패하게 되면 코드 통합이 되지 않는다.&#x20;

그래서 테스트 코드가 중요하다.&#x20;

테스트 코드에서 검증이 안되면 사전에 이슈 잡기가 수월해진다.





<figure><img src="../.gitbook/assets/스크린샷 2023-12-11 오후 6.52.13.png" alt=""><figcaption></figcaption></figure>

위와 같이 테스트코드가 통과 되지 않는다면 통합이 되지 않는다.

다시 테스트 코드를 수정해서 git push 를 하게 되면 ,&#x20;





<figure><img src="../.gitbook/assets/스크린샷 2023-12-11 오후 7.13.16.png" alt=""><figcaption></figcaption></figure>

정상적으로 코드 통합이 된다.
