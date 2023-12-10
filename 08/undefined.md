# 인증과 인가

인증은 사용자의 신원을 입증하는 과정입니다.

예를들어 사용자가 사이트에 로그인을 할 때 누구인지 확인하는 과정을 인증이라고 합니다.

인가는 인증과는 다릅니다. 인가는 사이트의 특정 부분에 접근할 수 있는지 권한을 확인하는 작업입니다.

예를 들어 관리자는 관리자 페이지에 들어갈 수 있지만 일반 사용자는 관리자 페이지에 들어갈 수 없습니다.

이런 권한을 확인하는 과정을 인가라고 합니다.

## 스프링 시큐리티

스프링 시큐리티를 사용하면 아주 쉽게 처리할 수 있습니다.

스프링 시큐리티는 스프링 기반 애플리케이션의 보안을 담당하는 스프링 하위 프레임워크입니다.

보안 관련 옵션을 많이 제공하죠. 그리고 애너테이션으로 설정도 매우 쉽습니다.&#x20;

CSRF 공격, 세션 고정공격을 방어해주고, 요청 헤더도 보안 처리를 해주므로 개발자가 보안 관련 개발을 해야하는 부담을 크게 줄여줍니다.



## User Entity

```java
package me.shinsunyoung.springbootdeveloper.domain;

import jakarta.persistence.*;
import lombok.AccessLevel;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.List;

@Table(name = "users")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Entity
public class User implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", updatable = false)
    private Long id;

    @Column(name = "email", nullable = false, unique = true)
    private String email;

    @Column(name = "password", nullable = false)
    private String password;

    @Builder
    public User(String email, String password, String auth) {
        this.email = email;
        this.password = password;
    }


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority("user"));
    }

    @Override
    public String getUsername() {
        return email;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}

```



### 레포지터리 만들기

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```



### 서비스 메서드 코드 작성

```java
@RequiredArgsConstructor
@Service
public class UserDetailService implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    @Override
    public User loadUserByUsername(String email) {
        return userRepository.findByEmail(email)
            .orElseThrow(() -> new IllegalArgumentException((email)));
    }
}
```



스프링 시큐리티에서 사용자의 정보를 가져오는 UserDetailsService 인터페이스를 구현합니다.

필수로 구현해야하는 loadUserByUsername() 메서드를 오버라이딩 해서 사용자 정보를 가져오는 로직을 작성합니다.



### 시큐리티 설정하기

<figure><img src="../.gitbook/assets/스크린샷 2023-12-10 오후 1.02.14.png" alt=""><figcaption></figcaption></figure>

WebSecurityConfig.java

```java
package me.shinsunyoung.springbootdeveloper.config;

import lombok.RequiredArgsConstructor;
import me.shinsunyoung.springbootdeveloper.service.UserDetailService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityCustomizer;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

import static org.springframework.boot.autoconfigure.security.servlet.PathRequest.toH2Console;

@RequiredArgsConstructor
@Configuration
public class WebSecurityConfig {

    private final UserDetailService userService;

    @Bean
    public WebSecurityCustomizer configure() {
        return (web) -> web.ignoring()
                .requestMatchers(toH2Console())
                .requestMatchers("/static/**");
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                .authorizeRequests()
                    .requestMatchers("/login", "/signup", "/user").permitAll()
                    .anyRequest().authenticated()
                .and()
                .formLogin()
                    .loginPage("/login")
                    .defaultSuccessUrl("/articles")
                .and()
                .logout()
                    .logoutSuccessUrl("/login")
                    .invalidateHttpSession(true)
                .and()
                .csrf().disable()
                .build();
    }

    @Bean
    public AuthenticationManager authenticationManager(HttpSecurity http, BCryptPasswordEncoder bCryptPasswordEncoder, UserDetailService userDetailService) throws Exception {
        return http.getSharedObject(AuthenticationManagerBuilder.class)
                .userDetailsService(userService)
                .passwordEncoder(bCryptPasswordEncoder)
                .and()
                .build();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

```



이후 UserService.java 파일을 수정한다.

```java
@RequiredArgsConstructor
@Service
public class UserService {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;
    
    public Long save(AddUserRequest dto) {
        return userRepository.save(User.builder()
            .email(dto.getEmail())
            .password(bCryptPasswordEncoder.encode(dto.getPassword()))
            .build()).getId();
    }
}
```



```java
@RequiredArgsConstructor
@Controller
public class UserApiController {

    private final UserService userService;

    @PostMapping("/user")
    public String signup(AddUserRequest request) {
        userService.save(request);
        return "redirect:/login";
    }

    @GetMapping("/logout")
    public String logout(HttpServletRequest request, HttpServletResponse response) {
        new SecurityContextLogoutHandler().logout(request, response, SecurityContextHolder.getContext().getAuthentication());
        return "redirect:/login";
    }
}

```

로그인 ,
