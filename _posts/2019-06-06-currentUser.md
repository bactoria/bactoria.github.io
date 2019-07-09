---
layout: post
title:  "currentUser"
categories: Spring
author: bactoria
---

* content
{:toc}

## 현재 사용자

스프링 시큐리티로 인증하였을 경우,

현재 사용자 보기

### 1. Authentication 객체로 받기

```java
@RestController
public class TempRestController {

    @GetMapping("/principal")
    public ResponseEntity getAuthentication() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        User principal = (User) authentication.getPrincipal();
        return ResponseEntity.ok(principal);
    }

}
```

**localhost:8080/principal (without accessToken)**
```json
// Response
{
    "principal": "anonymousUser"
}
```

&nbsp;

**localhost:8080/principal  (with accessToken)**
```json
// Response
{
    "password": null,
    "username": "user",
    "authorities": [
        {
            "authority": "ROLE_USER"
        }
    ],
    "accountNonExpired": true,
    "accountNonLocked": true,
    "credentialsNonExpired": true,
    "enabled": true
}
```

&nbsp;


### 2. @AuthenticationPrincipal

```java
@RestController
public class TempRestController {

    @GetMapping("/principal")
    public ResponseEntity getAuthentication(@AuthenticationPrincipal User principal) {
        return ResponseEntity.ok(principal);
    }

}
```

1번과 동일

&nbsp;
&nbsp;

그런데 받아오는게... 영 부실하다. 

로그인을 할 때 UserDetailsService의 `loadUserByUsername` 은 로그인 할 경우에 실행된다.

```java
@Slf4j
@Service
@Transactional
@RequiredArgsConstructor
public class UserService implements UserDetailsService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    @Override
    public org.springframework.security.core.userdetails.UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        User user = userRepository.findByUserIdAndIsDeletedFalse(s)
                .orElseThrow(() -> new UserNotFoundException(s));

        return new org.springframework.security.core.userdetails.User(user.getUserId(), user.getPassword(), authority(user.getRole()));
    }

    private Collection<? extends GrantedAuthority> authority(UserRole roles) {
        return Collections.singleton(new SimpleGrantedAuthority("ROLE_" + roles.name()));
    }

    //...
```

로그인할 때 입력한 Id의 유저를 DB에 호출해서 해당유저의 **Id, Password, 권한 정보** 를 User(시큐리티 객체)에 담아서 리턴한다.

```java
User user = userRepository.findByUserIdAndIsDeletedFalse(userId)
```

만약 비즈니스 로직 상에 **Id, Password, 권한정보** 이외의 추가적인 유저 정보가 필요하다면 매번 유저 정보를 가져와서 쓸 수 있다.

하지만 매번 호출하는 것은 뭔가 구리다.

처음 로그인 시, DB에서 호출하는 시점에서 필요한 정보들을 세션에 저장할 수는 없을까





