# JWT 로 로그인/로그아웃 구현하기



<figure><img src="../.gitbook/assets/스크린샷 2023-12-10 오후 1.38.34.png" alt=""><figcaption></figcaption></figure>

1. 클라이언트가 아이디와 비밀번호를 서버에게 전달하면서 인증을 요청하면
2. 서버는 아이디와 비밀번호를 확인해 유효한 사용자인지 검증합니다. 유효한 사용자면 토큰을 생성해서 응답합니다.
3. 클라이언트는 서버에서 준 토큰을 저장합니다.
4. 이후 인증이 필요한 API를 사용할 때 토큰을 함께 보냅니다.
5. 그러면 서버는 토큰이 유효한지 검증합니다.
6. 토큰이 유효하다면 클라이언트가 요청한 내용을 처리합니다.

## 토큰 기반 인증의 특징

과정만 보면 간단합니다.

토큰 기반 인증은 어떤 특징이 있는 걸까요? 대표적으로 세가지 특징을 듭니다.

토큰 기반 인증은 무상태성, 확장성, 무결성이라는 특징이 있습니다.

### 무상태성

무상태성은 사용자의 인증 정보가 담겨 있는 토큰이 서버가 아닌 클라이언트에 있으므로 서버에 저장할 필요가 없습니다.

서버가 뭔가 데이터를 유지하고 있으려면 그만큼 자원을 소비해야하죠.

그런데 토큰 기반 인증에서는 클라이언트에서 인증 정보가 담긴 토큰을 생성하고 인증합니다.

따라서 클라이언트에서는 사용자의 인증 상태를 유지하면서 이후 요청을 처리해야 하는데 이것을 상태를 관리한다고 합니다.

이렇게 하면 서버 입장에서는 클라이언트의 인증 정보를 저장하거나 유지하지 않아도 되기 때문에 완전한 무상태로 효율적인 검증을 할 수 있습니다.



### 확장성

무상태성은 확장성에 영향을 줍니다.

서버를 확장할 때 상태 관리를 신경 쓸 필요가 없으니 서버확장에도 용이한 것이죠.

물건을 파는 서비스가 있고, 결제를 위한 서버와 주문을 위한 서버가 분리되어 있다고 하면 세션 인증 기반은 각각 API에서 인증을 해야되는 것과는 달리 토큰 기반 인증에서는 토큰을 가지는 주체는 클라이언트가 된다.

주체가 클라이언트이기 때문에 하나의 토큰으로 결제서버 , 주문 서버 등 요청을 보낼 수 있습니다.



### 무결성

토큰을 발급한 이후에는 토큰 정보를 변경하는 해우이를 할 수 없습니다.

즉, 토큰의 무결성이 보장됩니다.

만약 누군가 토큰을 한 글자라도 변경하면 서버에서는 유효하지 않은 토큰이라고 판단하는것이죠



### JWT



<figure><img src="../.gitbook/assets/스크린샷 2023-12-10 오후 4.06.15.png" alt=""><figcaption></figcaption></figure>

토큰을 간단히 만든다고 하면 ,&#x20;

```markdown
{
    "typ": "JWT",
    "alg": "HS256"
}
```

으로 만들수 있지만 더 많은 정보를 담을 수 있다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-10 오후 4.15.14.png" alt=""><figcaption></figcaption></figure>

```markdown
{
    "iss": "ash@gmail.com",
    "iat" : 2125125612,
    "exp" : 2125125612,
    "email": "email@gmail.com",   
}
```



### 리프레시 토큰



<figure><img src="../.gitbook/assets/스크린샷 2023-12-10 오후 4.21.51.png" alt=""><figcaption></figcaption></figure>

1. 클라이언트가 서버에게 인증을 요청합니다.
2. 서버는 클라이언트에서 전달한 정보를 바탕으로 인증 정보가 유효한지 확인한 뒤, 액세스 토큰과 리프레시 토큰을 만들어 클라이언트에게 전달합니다. 클라이언트는 전달받은 토큰을 저장합니다.
3. 서버에서 생성한 리프레시 토큰은 DB에도 저장해둡니다.
4. 인증을 필요로 하는 API를 호출할 때 클라이언트에 저장된 액세스 토큰과 함께 API를 요청합니다.
5. 서버는 전달받은 액세스 토큰이 유효한지 검사 -> 클라이언트에서 요청한 내용을 처리
6. 액세스 토큰이 만료 -> 클라이언트에서 서버에게 API 요청
7. 만료된 토큰이면 유효하지 않기 때문에 토큰 만료 에러 전달
8. 클라이언트는 저장해둔 리프레시 토큰과 함께 새로운 액세스 토큰을 발급 요청
9. 서버는 DB에서 리프레시 토큰을 조회한 후 전달 받은 리프레시 토큰과 저장해둔 리프레시 토큰과 같은지 확인
10. 유효한 리프레시 토큰이라면 새로운 액세스 토큰을 생성한 뒤 응답
11. 이후 4번과 같이 다시 API 요청



## jwt 서비스 구현

```markdown
dependencies {
    testAnnotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.projectlombok:lombok'
    implementation 'io.jsonwebtoken:jjwt:0.9.1' // 자바 JWT 라이브러리
    implementation 'io.jsonwebtoken:jjwt:0.9.1'
    implementation 'javax.xml.bind:jaxb-api:2.3.1'
}

```

자바에서 JWT를 사용하기 위한 라이브러리를 추가하고 XML 문서와 자바 객체 간 매핑을 자동화 하는 jax-api 를 추가합니다.



### 토큰 제공자 추가하기.



![](<../.gitbook/assets/스크린샷 2023-12-10 오후 4.43.15.png>)

파일 위치는 상관이 없지만 우선 config -> jwt -> JwtProperties 파일을 생성한다.

```java
@Setter
@Getter
@Component
@ConfigurationProperties("jwt")
public class JwtProperties {

    private String issuer;
    private String secretKey;
}


```

@ConfigurationProperties("jwt") 애너테이션은 자바 클래스에 프로피티값을 가져와서 사용하는 애너테이션이다.&#x20;

이후 TokenProvider 파일을 생성한다.

```java
@RequiredArgsConstructor
@Service
public class TokenProvider {

    private final JwtProperties jwtProperties;

    public String generateToken(User user, Duration expiredAt) {
        Date now = new Date();
        return makeToken(new Date(now.getTime() + expiredAt.toMillis()), user);
    }

    private String makeToken(Date expiry, User user) {
        Date now = new Date();

        return Jwts.builder()
                .setHeaderParam(Header.TYPE, Header.JWT_TYPE)
                .setIssuer(jwtProperties.getIssuer())
                .setIssuedAt(now)
                .setExpiration(expiry)
                .setSubject(user.getEmail())
                .claim("id", user.getId())
                .signWith(SignatureAlgorithm.HS256, jwtProperties.getSecretKey())
                .compact();
    }

    public boolean validToken(String token) {
        try {
            Jwts.parser()
                    .setSigningKey(jwtProperties.getSecretKey())
                    .parseClaimsJws(token);

            return true;
        } catch (Exception e) {
            return false;
        }
    }

    // 토큰 기반으로 인증 정보를 가져오는 메서드
    public Authentication getAuthentication(String token) {
        Claims claims = getClaims(token);
        Set<SimpleGrantedAuthority> authorities = Collections.singleton(new SimpleGrantedAuthority("ROLE_USER"));

        return new UsernamePasswordAuthenticationToken(new org.springframework.security.core.userdetails.User(claims.getSubject
                (), "", authorities), token, authorities);
    }
    
    // 토큰 기반으로 유저 ID를 가져오는 메서드
    public Long getUserId(String token) {
        Claims claims = getClaims(token);
        return claims.get("id", Long.class);
    }

    private Claims getClaims(String token) {
        return Jwts.parser()
                .setSigningKey(jwtProperties.getSecretKey())
                .parseClaimsJws(token)
                .getBody();
    }
}

```

테스트 코드를 작성합니다.



<figure><img src="../.gitbook/assets/스크린샷 2023-12-10 오후 4.48.27.png" alt=""><figcaption></figcaption></figure>

```java
@SpringBootTest
class TokenProviderTest {

    @Autowired
    private TokenProvider tokenProvider;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private JwtProperties jwtProperties;

    @DisplayName("generateToken(): 유저 정보와 만료 기간을 전달해 토큰을 만들 수 있다.")
    @Test
    void generateToken() {
        // given
        User testUser = userRepository.save(User.builder()
                .email("user@gmail.com")
                .password("test")
                .build());

        // when
        String token = tokenProvider.generateToken(testUser, Duration.ofDays(14));

        // then
        Long userId = Jwts.parser()
                .setSigningKey(jwtProperties.getSecretKey())
                .parseClaimsJws(token)
                .getBody()
                .get("id", Long.class);

        assertThat(userId).isEqualTo(testUser.getId());
    }

    @DisplayName("validToken(): 만료된 토큰인 경우에 유효성 검증에 실패한다.")
    @Test
    void validToken_invalidToken() {
        // given
        String token = JwtFactory.builder()
                .expiration(new Date(new Date().getTime() - Duration.ofDays(7).toMillis()))
                .build()
                .createToken(jwtProperties);

        // when
        boolean result = tokenProvider.validToken(token);

        // then
        assertThat(result).isFalse();
    }


    @DisplayName("validToken(): 유효한 토큰인 경우에 유효성 검증에 성공한다.")
    @Test
    void validToken_validToken() {
        // given
        String token = JwtFactory.withDefaultValues()
                .createToken(jwtProperties);

        // when
        boolean result = tokenProvider.validToken(token);

        // then
        assertThat(result).isTrue();
    }


    @DisplayName("getAuthentication(): 토큰 기반으로 인증정보를 가져올 수 있다.")
    @Test
    void getAuthentication() {
        // given
        String userEmail = "user@email.com";
        String token = JwtFactory.builder()
                .subject(userEmail)
                .build()
                .createToken(jwtProperties);

        // when
        Authentication authentication = tokenProvider.getAuthentication(token);

        // then
        assertThat(((UserDetails) authentication.getPrincipal()).getUsername()).isEqualTo(userEmail);
    }

    @DisplayName("getUserId(): 토큰으로 유저 ID를 가져올 수 있다.")
    @Test
    void getUserId() {
        // given
        Long userId = 1L;
        String token = JwtFactory.builder()
                .claims(Map.of("id", userId))
                .build()
                .createToken(jwtProperties);

        // when
        Long userIdByToken = tokenProvider.getUserId(token);

        // then
        assertThat(userIdByToken).isEqualTo(userId);
    }
}

```





