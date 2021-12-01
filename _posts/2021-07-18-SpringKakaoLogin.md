---
title: "[Spring Boot] REST API를 이용한 카카오 로그인 구현"
decription:
categories:
 - SpringBoot
tags:
 - SpringBoot
 - Social
 - Kakao
 - Jwt
---

> Spring Boot + JPA + JWT를 이용하여 백엔드에서 카카오 로그인을 시키기 위해서 어떠한 동작이 이루어지는지 알아보자

# 개요

<hr>

현재 진행중인 프로젝트에서 프론트(리액트), 백엔드(스프링)으로 나누어 소셜 로그인을 개발을 진행하였다. API 서버 역할을 하는 백엔드 입장에서 뷰를 보여줄 수 없었기 때문에 프론트에서 필요한 최소 데이터만 받아 로그인을 시키는 아래와 같은 방식으로 로직을 구성하였다.

![kakaoLogin](/assets/postImages/SpringKakaoLogin/kakaoLogin.JPG)

카카오 개발자 사이트를 보며 그러면 한 단계씩 알아가보자

사이트 → [링크](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api)

# 인가코드 (프론트)

<hr>

프론트가 받아서 백엔드에 전달해야 할 인가코드이다. 아래 사진을 보자.

![kakaoLogin_code](/assets/postImages/SpringKakaoLogin/kakaoLogin_code.JPG)

뷰를 담당하는 프론트는 사용자를 요청 파라미터에 맞춰 카카오톡 로그인 창으로 보내며 로그인을 유도한다. 로그인이 성공하게 된다면 응답으로 빨간색으로 박스 쳐진 Code를 받게 되는데 이것이 인가코드이다.
받게 된 인가코드를 백엔드에게 전달한다.

![kakaoLogin_data](/assets/postImages/SpringKakaoLogin/kakaoLogin_data.JPG)

요청 파라미터 필요한 데이터는 카카오 개발자 사이트로 들어가 애플리케이션을 하나 만들고 메인에 있는 REST API키와 네비게이션 바를 눌러 카카오 로그인을 클릭하게 되면 오른쪽 사진과 같이 리다이렉션 URL를 작성하여 얻을 수 있다. 리다이렉션 URL을 작성하기 전에 꼭 활성화 설정을 ON 해주셔야 한다.

# Access Token (백엔드)

<hr>

프론트에서 받은 인가코드를 이용하여 우리는 Access Token을 얻어야 한다.

![kakaoLogin_access](/assets/postImages/SpringKakaoLogin/kakaoLogin_access.JPG)

위 사진을 보게 되면 grant_type, client_id, redirect_uri, 프론트에게 받은 code를 가지고 요청을 보내게 되면 access_token을 얻을 수 있는 것을 알 수 있다. client_id, redirect_uri는 프론트와 맞춰서 동일하게 작성하면 된다.

![kakaoLogin_data2](/assets/postImages/SpringKakaoLogin/kakaoLogin_data2.JPG)

client_secret을 추가하여 보안을 강화하고 싶으면 카카오 로그인 - 보안에 들어가면 위 사진과 페이지가 나타나게 되고 코드를 발급받으면 된다.

## 구현 소스

``` java
public String getAccessTokenByCode(String code) {

    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);

    LinkedMultiValueMap<String, String> params = new LinkedMultiValueMap<>();
    params.add("grant_type", "authorization_code");
    params.add("client_id", "<REST API>");
    params.add("redirect_uri", "<Redirect_URI>" + "/oauth2/code/kakao");
    params.add("code", code);
    params.add("client_secret", "");

    HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(params, headers);

    String url = "https://kauth.kakao.com/oauth/token";

    ResponseEntity<String> response = restTemplate.postForEntity(url, request, String.class);

    try {
        return objectMapper.readValue(response.getBody(), KakaoAccessTokenDto.class).getAccess_token();
    } catch (JsonProcessingException e) {
        e.printStackTrace();
    }
    return null;
}
```

# 사용자 정보 (백엔드)

<hr>

인가코드로 요청해서 얻게된 access_token으로 사용자 정보를 얻을 수 있다.

![kakaoLogin_user](/assets/postImages/SpringKakaoLogin/kakaoLogin_user.JPG)

access_token을 담아 요청하게 되면 허가된 사용자 정보를 받게된다.

![kakaoLogin_data3](/assets/postImages/SpringKakaoLogin/kakaoLogin_data3.JPG)

받을 데이터 권한을 결정하기 위해서는 카카오 로그인 - 동의 항목에 들어가면 위 사진이 뜨는데 필요한 데이터의 상태를 동의로 필수 동의, 선택 동의로 바꿔주면 된다.

## 구현 소스

``` java
public String getUserInfoByAccessToken(String accessToken) {

    HttpHeaders headers = new HttpHeaders();
    headers.set("Authorization", "Bearer " + accessToken);
    headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);

    LinkedMultiValueMap<String, String> params = new LinkedMultiValueMap<>();

    HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(params, headers);

    String url = "https://kapi.kakao.com/v2/user/me";

    return restTemplate.postForObject(url, request, String.class);
}
```

# 회원가입 & JWT 토큰 생성 (백엔드)

<hr>

이제 얻게된 사용자 정보들을 토대로 회원가입을 진행해야 한다. 이유는 회원가입을 시키지 않는다면 다음에 접속할 때에 소셜 로그인을 하여 했던 작업들이 전부 날아가기 때문이다. 만약에 데이터를 유지할 필요가 없다면 그냥 JWT token만 생성해서 전달해도 무방하다.

## 구현 소스

``` java
@Service
@RequiredArgsConstructor
public class SocialService {

    private final KakaoService kakaoService;
    private final ObjectMapper objectMapper;

    public SocialDto verificationKakao(String code){

        SocialDto socialDto = new SocialDto();
        // 코드를 이용하여 accessToken 추출
        String accessToken = kakaoService.getAccessTokenByCode(code);
        // accessToken을 이용하여 사용자 정보 추출
        String userInfo = kakaoService.getUserInfoByAccessToken(accessToken);

        try {
            JsonNode jsonNode = objectMapper.readTree(userInfo);
            String email = String.valueOf(jsonNode.get("kakao_account").get("email"));
            socialDto.setEmail("kakao_" + email.substring(1, email.length() - 1));
            String name = String.valueOf(jsonNode.get("kakao_account").get("profile").get("nickname"));
            socialDto.setName(name.substring(1, name.length() - 1));
            String imageUrl = String.valueOf(jsonNode.get("kakao_account").get("profile").get("profile_image_url"));
            socialDto.setImageUrl(imageUrl.substring(1, imageUrl.length() - 1));
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }

        return socialDto;
    }
}

```

소셜 서비스 구현 부분이고 access_token으로 받은 사용자 정보를 JsonNode로 변경하여 필요한 데이터만 추출하여 Dto로 담아 반환한다. json인 key:value 형태로 되어있기 때문에 쉽게 추출할 수 있을 것이다.

``` java
@RestController
@RequiredArgsConstructor
public class SocialController {

    private final SocialService socialService;
    private final UserRepository userRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;
    private final Environment env;

    @PostMapping("/users/login/{provider}")
    public ResponseEntity socialLogin(@PathVariable String provider,
                                      @RequestBody(required = false) UserDto userDto,
                                      @RequestParam String code,
                                      HttpServletResponse response){

        SocialDto socialDto = null;

        if (provider.equals("kakao")) {
            socialDto = socialService.verificationKakao(code);
        }
        else {
            return new ResponseEntity(HttpStatus.NOT_FOUND);
        }

        List<UserEntity> users = userRepository.findByEmail(socialDto.getEmail());

        // 서비스에 등록된 회원이 아니라면
        if (users.isEmpty()) {
            userDto.setEmail(socialDto.getEmail());
            userDto.setName(socialDto.getName());
            userDto.setImageUrl(socialDto.getImageUrl());
            userDto.setPassword(bCryptPasswordEncoder.encode(UUID.randomUUID().toString()));
            UserEntity userEntity = DtoToEntity.userDtoToEntity(userDto);
            // 회원가입
            userRepository.save(userEntity);
        }

        // JWT 토큰 생성
        String token = JWT.create()
                .withSubject("JwtToken")
                .withExpiresAt(new Date(System.currentTimeMillis() + Long.parseLong(env.getProperty("token.expiration_time"))))
                .withClaim("email", socialDto.getEmail())
                .sign(Algorithm.HMAC512(env.getProperty("token.secret")));

        // JWT 토큰 헤더에 담아 전달
        response.addHeader(env.getProperty("token.header"), env.getProperty("token.prefix") + token);

        return new ResponseEntity(HttpStatus.OK);
    }
}
```

요청을 받는 컨트롤러 부분이며 유저 정보를 추출하여 서비스에 등록된 회원이면 JWT 토큰만 생성하여 헤더에 담아 전달하고 회원이 아니라면 회원가입을 진행한다.

# 결론

<hr>

이번 카카오톡 소셜 로그인을 진행하면서 REST API를 활용해 볼 수 있었고 다른 서비스의 소셜 로그인 또는 developer에서 제공하는 여러 API들에 대해서 이전보다 쉽게 접근할 수 있게 되었다.
