[Github](https://github.com/jwtk/jjwt) | [在线工具](https://www.jwt.io/)

## 为什么需要 JWT？

在现代 Web 开发中，JWT（Json-Web-Token）成为主流认证方案，核心原因在于其适配前后端分离、分布式及微服务架构的特性：

- **无状态（Stateless）**：服务器无需存储 Session，令牌自包含用户信息，降低分布式系统中 Session 共享的复杂度
- **跨服务传递**：适合微服务架构，用户请求经过多个服务时可直接传递令牌
- **减少数据库访问**：令牌包含核心信息，解析即可获取用户数据，无需频繁查询数据库
- **安全性**：支持过期时间设置，通过签名算法（HS256、RS256 等）防止篡改

## JWT 基础概念

JWT全称为Json-Web-Token，是一种**紧凑且自包含**的轻量级身份验证令牌，用于在网络各方之间安全传递信息，以JSON对象为载体。

JWT的本质是一个字符串，由三部分组成，用点（`.`）符号分隔，结构为：`Header.Payload.Signature`。

示例：eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30

密钥：a-string-secret-at-least-256-bits-long

### Header（头部）

- 作用：描述令牌类型和使用的签名算法
- 格式：JSON 对象，经 Base64 编码
- 核心字段：
    - `typ`：固定为`JWT`（令牌类型）
    - `alg`：签名算法（如`HS256`、`RS256`）

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload（载荷）

- 作用：存储核心信息（用户 ID、权限、过期时间等），JSON 格式，经 Base64 编码
- 声明类型（Claims）：
    - **注册声明（Registered Claims）**：规范预定义的推荐字段（非强制）
        - `iss`（Issuer）：令牌签发者
	    - `exp`（Expiration Time）：令牌过期时间（Unix 时间戳）
	    - `sub`（Subject）：令牌主题（通常是用户 ID）
	    - `aud`（Audience）：令牌接收者
	    - `iat`（Issued At）：令牌签发时间（Unix 时间戳）
	    - `nbf`（Not Before）：令牌生效时间（在此之前无效）
    - **公共声明（Public Claims）**：自定义通用字段，需避免命名冲突（建议用域名前缀，如`com.example.userType`）  
      Public Claims 是**可以被不同系统通用的自定义字段**，但需要遵循一个原则：**避免与其他系统的自定义字段重名**  
      为了做到这一点，JWT 规范建议：
	    - 可以使用 IANA（互联网号码分配机构）维护的 “JWT 注册表”（[点击查看](https://www.iana.org/assignments/jwt/jwt.xhtml)）中的字段，这些字段是经过官方注册的，具有唯一性；
		- 如果自己定义新的 Public Claims，建议使用具有唯一性的命名（比如包含域名前缀，如 `com.example.userType`），避免和其他系统冲突。
    - **私有声明（Private Claims）**：仅特定系统间使用的自定义字段（如`deptCode: "IT-001"`）

> [!TIP] 注意
> 公共声明与私有声明在代码编写上无本质区别，都是**用户自定义的字段**，主要是**使用场景和命名规范**上的区别。

代码示例：

```java
import io.jsonwebtoken.*;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

public class JwtExample {
    // 签名密钥（实际应用中需妥善保管，避免硬编码）
    private static final String SECRET_KEY = "your-256-bit-secret-key-which-should-be-very-long-for-security";
    // 过期时间：2小时
    private static final long EXPIRATION_TIME = 2 * 60 * 60 * 1000;

    public static void main(String[] args) {
        // 1. 生成JWT（包含Public Claims和Private Claims）
        String jwt = generateToken();
        System.out.println("生成的JWT：" + jwt);

        // 2. 解析JWT并获取声明
        parseToken(jwt);
    }

    // 生成JWT
    private static String generateToken() {
        // 设置当前时间和过期时间
        Date now = new Date();
        Date expirationDate = new Date(now.getTime() + EXPIRATION_TIME);

        // 构建Payload（包含注册声明、Public Claims、Private Claims）
        return Jwts.builder()
                // 注册声明（规范预定义）
                .setIssuer("my-system") // iss：签发者
                .setSubject("user123")  // sub：用户ID
                .setIssuedAt(now)       // iat：签发时间
                .setExpiration(expirationDate) // exp：过期时间

                // Public Claims（通用自定义字段，命名规范以避免冲突）
                .claim("email", "user@example.com") // 通用邮箱字段
                .claim("com.example.userType", "premium") // 带域名前缀的用户类型（跨系统通用）
                .claim("phone_number", "13800138000") // 通用手机号字段

                // Private Claims（仅本系统内部使用的自定义字段）
                .claim("deptCode", "IT-001") // 部门编码（仅本系统业务用）
                .claim("internalRole", "editor") // 内部角色（非对外暴露）

                // 签名（使用HS256算法）
                .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
                .compact();
    }

    // 解析JWT并获取声明
    private static void parseToken(String jwt) {
        try {
            Claims claims = Jwts.parserBuilder()
                    .setSigningKey(SECRET_KEY)
                    .build()
                    .parseClaimsJws(jwt)
                    .getBody();

            // 读取注册声明
            System.out.println("\n注册声明：");
            System.out.println("签发者(iss)：" + claims.getIssuer());
            System.out.println("用户ID(sub)：" + claims.getSubject());
            System.out.println("过期时间(exp)：" + new Date(claims.getExpiration().getTime()));

            // 读取Public Claims
            System.out.println("\nPublic Claims：");
            System.out.println("邮箱：" + claims.get("email"));
            System.out.println("用户类型(com.example.userType)：" + claims.get("com.example.userType"));
            System.out.println("手机号：" + claims.get("phone_number"));

            // 读取Private Claims
            System.out.println("\nPrivate Claims：");
            System.out.println("部门编码：" + claims.get("deptCode"));
            System.out.println("内部角色：" + claims.get("internalRole"));

        } catch (JwtException e) {
            System.out.println("JWT验证失败：" + e.getMessage());
        }
    }
}
```

### Signature（签名）

- 作用：验证令牌是否被篡改，确保安全性
- 生成逻辑：对 Base64 编码后的 Header 和 Payload 进行签名，需要用到密钥
    - 对称算法（如 HS256）：签发者和验证者共享同一密钥
    - 非对称算法（如 RS256）：签发者用私钥签名，验证者用公钥验签（更适合分布式系统）

```java
// 签名生成示例（HS256）
Signature = HMACSHA256( base64UrlEncode(Header) + "." + base64UrlEncode(Payload), secret )
```

签名的结果会再次进行Base64编码，成为JWT的第三部分

### JWT 认证流程

1. **签发**：用户登录成功后，服务器生成包含用户信息（Payload）的 JWT，用密钥签名后返回给客户端；
2. **存储**：客户端（如浏览器、App）将 JWT 存储在 localStorage、sessionStorage 或 Cookie 中；
3. **使用**：客户端后续请求时，通过 HTTP 头（如`Authorization: Bearer <JWT>`）携带令牌；
4. **验证**：服务器接收请求后，提取 JWT，用相同的算法和密钥验证签名：
    - 若签名无效（被篡改），拒绝请求；
    - 若签名有效，解码 Payload，检查过期时间（`exp`）等，确认用户身份和权限，允许请求。

![Snipaste_2025-08-30_13-22-31](assets/Snipaste_2025-08-30_13-22-31.jpg)


---

## Spring Security 整合 JWT 实现

### 1. 引入依赖

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.13.0</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.13.0</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.13.0</version>
    <scope>runtime</scope>
</dependency>
```

### 2. 配置文件（application.yaml）

```yaml
spring:
  app:
    jwtSecret: YS1zdHJpbmctc2VjcmV0LWF0LWxlYXN0LTI1Ni1iaXRzLWxvbmc=  # Base64编码的密钥（至少256位）
    jwtExpirationMs: 300000  # 过期时间（5分钟，单位：毫秒）
```

### 3. JWT 工具类（核心组件）

负责生成、解析、验证 JWT 令牌：

```java
import io.jsonwebtoken.*;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import jakarta.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.security.Key;
import java.util.Date;

/**
 * JWT工具类，负责生成、解析和验证JWT令牌
 */
@Component
public class JwtUtils {
    private static final Logger logger = LoggerFactory.getLogger(JwtUtils.class);

    // JWT签名密钥
    @Value("${spring.app.jwtSecret}")
    private String jwtSecret;

    // JWT过期时间(毫秒)
    @Value("${spring.app.jwtExpirationMs}")
    private int jwtExpirationMs;

    /**
     * 从请求头中提取JWT令牌
     *
     * @param request HTTP请求对象
     * @return 提取到的JWT令牌(无Bearer前缀)，若不存在则返回null
     */
    public String getJwtFromHeader(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        logger.debug("Authorization Header: {}", bearerToken);
        // 提取Bearer前缀后的令牌内容
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }

    /**
     * 根据用户信息生成JWT令牌
     *
     * @param userDetails 包含用户信息的对象
     * @return 生成的JWT令牌字符串
     */
    public String generateTokenFromUsername(UserDetails userDetails) {
        String username = userDetails.getUsername();
        return Jwts.builder()
                .subject(username) // 设置主题(用户名)
                .issuedAt(new Date()) // 设置签发时间
                .expiration(new Date(System.currentTimeMillis() + jwtExpirationMs)) // 设置过期时间
                .signWith(key()) // 使用密钥签名
                .compact();
    }

    /**
     * 从JWT令牌中解析出用户名
     *
     * @param token JWT令牌
     * @return 解析出的用户名
     */
    public String getUserNameFromJwtToken(String token) {
        return Jwts.parser()
                .verifyWith((SecretKey) key()) // 验证签名密钥
                .build()
                .parseSignedClaims(token) // 解析令牌
                .getPayload()
                .getSubject(); // 获取主题(用户名)
    }

    /**
     * 生成JWT签名密钥
     *
     * @return 基于配置密钥的HMAC SHA算法密钥
     */
    private Key key() {
        return Keys.hmacShaKeyFor(Decoders.BASE64.decode(jwtSecret));
    }

    /**
     * 验证JWT令牌的有效性
     *
     * @param authToken 需要验证的JWT令牌
     * @return 验证成功返回true，失败返回false
     */
    public boolean validateJwtToken(String authToken) {
        try {
            Jwts.parser().verifyWith((SecretKey) key()).build().parseSignedClaims(authToken);
            return true;
        } catch (MalformedJwtException e) {
            // JWT令牌格式无效
            logger.error("Invalid JWT token: {}", e.getMessage());
        } catch (ExpiredJwtException e) {
            // JWT令牌已过期
            logger.error("JWT token is expired: {}", e.getMessage());
        } catch (UnsupportedJwtException e) {
            // 不支持的JWT令牌类型
            logger.error("JWT token is unsupported: {}", e.getMessage());
        } catch (IllegalArgumentException e) {
            // JWT令牌声明为空
            logger.error("JWT claims string is empty: {}", e.getMessage());
        }
        return false;
    }
}
```

### 4. JWT 认证过滤器

拦截请求并验证 JWT，将认证信息存入 SecurityContext：

```java
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

/**
 * JWT认证过滤器
 * 作用：拦截客户端请求，验证JWT令牌有效性，若有效则将用户认证信息存入SecurityContext，实现无状态认证
 * 继承OncePerRequestFilter确保每次请求仅被过滤一次
 */
@Component
public class AuthTokenFilter extends OncePerRequestFilter {

    private final JwtUtils jwtUtils;
    private final UserDetailsService userDetailsService;

    @Autowired // Spring 4.3+可省略该注解（构造方法唯一时）
    public AuthTokenFilter(JwtUtils jwtUtils, UserDetailsService userDetailsService) {
        this.jwtUtils = jwtUtils;
        this.userDetailsService = userDetailsService;
    }

    private static final Logger logger = LoggerFactory.getLogger(AuthTokenFilter.class);

    /**
     * 核心过滤逻辑：处理每个请求的JWT认证
     */
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        logger.debug("AuthTokenFilter called for URI: {}", request.getRequestURI());
        try {
            String jwt = parseJwt(request);
            if (StringUtils.isNotBlank(jwt) && jwtUtils.validateJwtToken(jwt)) {
                String username = jwtUtils.getUserNameFromJwtToken(jwt);

                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                UsernamePasswordAuthenticationToken authentication =
                        new UsernamePasswordAuthenticationToken(userDetails,
                                null,
                                userDetails.getAuthorities());
                logger.debug("Roles from JWT: {}", userDetails.getAuthorities());

                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            // 设置用户认证信息失败
            logger.error("Cannot set user authentication: {}", e);
        }

        filterChain.doFilter(request, response);
    }

    /**
     * 从请求中提取JWT令牌
     *
     * @param request HTTP请求对象
     * @return 提取到的JWT令牌，若不存在则返回null
     */
    private String parseJwt(HttpServletRequest request) {
        String jwt = jwtUtils.getJwtFromHeader(request);
        logger.debug("AuthTokenFilter.java: {}", jwt);
        return jwt;
    }
}
```

### 5. 异常处理

因为全局异常处理器并不能处理此处的异常。因为这里还没到Controller层

#### 401 异常处理器

```java
import com.alibaba.fastjson2.JSON;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.MediaType;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

/**
 * JWT认证未授权处理器（处理401场景）
 * 作用：当用户未登录/登录凭证无效（如JWT过期、签名错误）时，统一返回JSON格式的401响应，替代默认的HTML错误页
 */
@Component
public class AuthEntryPointJwt implements AuthenticationEntryPoint {
    private static final Logger logger = LoggerFactory.getLogger(AuthEntryPointJwt.class);

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        logger.error("Unauthorized error: {}", authException.getMessage());

        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding("UTF-8");

        final Map<String, Object> body = new HashMap<>();
        body.put("error", "Unauthorized");
        body.put("message", authException.getMessage());
        body.put("path", request.getRequestURI());

        response.getWriter().write(JSON.toJSONString(body));
    }
}
```

#### 403 异常处理器

```java
import com.alibaba.fastjson2.JSON;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.MediaType;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

/**
 * 权限不足处理器（处理403场景）
 * 作用：当用户已登录，但缺少访问目标接口的权限时，统一返回JSON格式的403响应
 */
@Component
public class AuthAccessDeniedHandler implements AccessDeniedHandler {
    private static final Logger logger = LoggerFactory.getLogger(AuthAccessDeniedHandler.class);

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        logger.error("Forbidden error: {}", accessDeniedException.getMessage());
        
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding("UTF-8");

        final Map<String, Object> body = new HashMap<>();
        body.put("error", "Unauthorized");
        body.put("message", accessDeniedException.getMessage());
        body.put("path", request.getRequestURI());

        response.getWriter().write(JSON.toJSONString(body));
    }
}
```


### 6. Security 配置类

```java
import com.example.securitydemo.exception.AuthAccessDeniedHandler;
import com.example.securitydemo.exception.AuthEntryPointJwt;
import com.example.securitydemo.filter.AuthTokenFilter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

/**
 * SecurityConfig
 */
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    private final AuthTokenFilter authenticationJwtTokenFilter;
    private final AuthEntryPointJwt unauthorizedHandler;
    private final AuthAccessDeniedHandler forbiddenHandler;

    @Autowired // Spring 4.3+可省略该注解（构造方法唯一时）
    public SecurityConfig(AuthTokenFilter authenticationJwtTokenFilter, AuthEntryPointJwt unauthorizedHandler, AuthAccessDeniedHandler forbiddenHandler) {
        this.authenticationJwtTokenFilter = authenticationJwtTokenFilter;
        this.unauthorizedHandler = unauthorizedHandler;
        this.forbiddenHandler = forbiddenHandler;
    }


    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf(AbstractHttpConfigurer::disable);// 禁用CSRF
        http.authorizeHttpRequests(authorizeRequests ->
                authorizeRequests.requestMatchers("/auth/**").permitAll().anyRequest().authenticated());

        http.exceptionHandling(exception -> exception
                .authenticationEntryPoint(unauthorizedHandler)
                .accessDeniedHandler(forbiddenHandler));

        http.sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
        http.addFilterBefore(authenticationJwtTokenFilter, UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 7. 解决循环依赖问题

完成上面的步骤后启动项目，发现报错，报错信息如下：

```plaintext
Description:

The dependencies of some of the beans in the application context form a cycle:

┌─────┐
|  authTokenFilter defined in file [E:\Projects\IdeaProjects\securitydemo\target\classes\com\example\securitydemo\filter\AuthTokenFilter.class]
↑     ↓
|  inMemoryUserDetailsManager defined in class path resource [org/springframework/boot/autoconfigure/security/servlet/UserDetailsServiceAutoConfiguration.class]
↑     ↓
|  securityConfig defined in file [E:\Projects\IdeaProjects\securitydemo\target\classes\com\example\securitydemo\config\SecurityConfig.class]
└─────┘
```

这是一个循环依赖的问题，当你没有自定义 UserDetailsService 时，Spring Boot 会自动配置 `InMemoryUserDetailsManager`（这是 UserDetailsService 的默认实现）

可以看到在UserDetailsServiceAutoConfiguration中有如下的配置

![file-20250903201433690](assets/file-20250903201433690.png)

这说明inMemoryUserDetailsManager需要用到passwordEncoder这个bean对象，而这个bean对象，我们定义在了SecurityConfig，所以就导致了InMemoryUserDetailsManager依赖securityConfig。

解决方法也很简单：

方案一：在过滤器的构造注入中，在UserDetailService前加上@lazy注解

![file-20250903201651106](assets/file-20250903201651106.png)

方案二：

单独创建配置类声明`PasswordEncoder`（推荐）

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

/**
 * CommonConfig
 */
@Configuration
public class CommonConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

```

两种方案都能解决此问题，任选其一。

不过在实际的开发中，我们一般会自定义UserDetailService，会从数据库中读取用户，所以不会出现上面的情况。
### 8. 自定义用户与登录接口

#### 自定义用户

##### 内存用户

现在我们就先演示此种方式，使用内存用户

若要自定义用户，代码如下：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

/**
 * CommonConfig
 */
@Configuration
public class CommonConfig {
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.withUsername("user")
                .password(passwordEncoder().encode("123456"))
                .roles("USER")
                .build();

        UserDetails admin = User.withUsername("admin")
                .password(passwordEncoder().encode("admin123"))
                .roles("ADMIN")
                .build();
        // 使用内存存储用户信息
        return new InMemoryUserDetailsManager(user, admin);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

##### 从数据库获取

1. 基础实体类（不实现任何接口）

```java
@Data
public class UserInfo {
    private Long id;
    private String username;  // 登录用户名
    private String password;  // 加密后的密码
    private boolean enabled;  // 账号是否可用
    private List<String> roles;  // 角色列表，如["ADMIN", "USER"]
}
```

2. UserDetailsService 实现（核心转换逻辑）

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserInfoService userInfoService;  // 假设已实现从数据库查询用户

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 1. 从数据库查询用户
        UserInfo userInfo = userInfoService.getByUsername(username);
        if (userInfo == null) {
            throw new UsernameNotFoundException("用户不存在: " + username);
        }

        // 2. 转换角色为Spring Security需要的权限格式
        Collection<GrantedAuthority> authorities = userInfo.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))  // 必须加ROLE_前缀
                .collect(Collectors.toList());

        // 3. 使用Spring Security提供的User类作为UserDetails实现
        return User.withUsername(userInfo.getUsername())
                .password(userInfo.getPassword())  // 密码需已加密（如BCrypt）
                .authorities(authorities)
                .accountExpired(false)  // 简化：默认账号未过期
                .accountLocked(false)   // 简化：默认账号未锁定
                .credentialsExpired(false)  // 简化：默认密码未过期
                .enabled(userInfo.isEnabled())  // 使用数据库中的启用状态
                .build();
    }
}
```

3. 替换JWT过滤器中的UserDetailsService注入的依赖

##### 从数据库获取（实现 UserDetails）

1. 实体类（实现 UserDetails）

```java
import lombok.Data;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import java.util.Collection;
import java.util.List;
import java.util.stream.Collectors;

@Data
public class UserInfo implements UserDetails {
    private Long id;
    private String username;
    private String password;
    private boolean enabled = true;
    private boolean accountNonLocked = true;
    private boolean accountNonExpired = true;
    private boolean credentialsNonExpired = true;
    private List<String> roles;  // 角色列表，如["ADMIN", "USER"]

    // 角色转换为权限
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roles.stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                .collect(Collectors.toList());
    }

    // 实现UserDetails的状态方法
    @Override
    public boolean isAccountNonExpired() {
        return accountNonExpired;
    }

    @Override
    public boolean isAccountNonLocked() {
        return accountNonLocked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return credentialsNonExpired;
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }
}
```

2. UserDetailsService 实现

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserInfoMapper userInfoMapper;  // 注入mapper

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 通过mapper查询用户
        UserInfo userInfo = userInfoMapper.selectByUsername(username);
        
        // 用户不存在则抛出异常
        if (userInfo == null) {
            throw new UsernameNotFoundException("用户不存在: " + username);
        }
        
        // 查询用户角色（实际项目中可能需要单独查询）
        List<String> roles = userInfoMapper.selectRolesByUserId(userInfo.getId());
        userInfo.setRoles(roles);
        
        // 直接返回实现了UserDetails的实体类
        return userInfo;
    }
}
```

3. 替换JWT过滤器中的UserDetailsService注入的依赖

#### 登录接口

这个时候我们要定义一个登录接口，在添加接口前，我们还需要先添加一个配置AuthenticationManager

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

/**
 * CommonConfig
 */
@Configuration
public class CommonConfig {
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.withUsername("user")
                .password(passwordEncoder().encode("123456"))
                .roles("USER")
                .build();

        UserDetails admin = User.withUsername("admin")
                .password(passwordEncoder().encode("admin123"))
                .roles("ADMIN")
                .build();
        // 使用内存存储用户信息
        return new InMemoryUserDetailsManager(user, admin);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration builder) throws Exception {
        return builder.getAuthenticationManager();
    }
}
```

登录接口

```java
import com.example.securitydemo.dto.LoginParams;
import com.example.securitydemo.utils.JwtUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

/**
 * AuthController
 */
@RestController
@RequestMapping("/auth")
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final JwtUtils jwtUtils;

    @Autowired
    public AuthController(AuthenticationManager authenticationManager, JwtUtils jwtUtils) {
        this.authenticationManager = authenticationManager;
        this.jwtUtils = jwtUtils;
    }

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginParams loginParams) {
        Authentication authentication;
        try {
            authentication = authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(loginParams.getUsername(), loginParams.getPassword()));
        } catch (AuthenticationException e) {
            Map<String, Object> map = new HashMap<>();
            map.put("message", "Bad credentials");
            map.put("status", false);
            return new ResponseEntity<Object>(map, HttpStatus.NOT_FOUND);
        }

        SecurityContextHolder.getContext().setAuthentication(authentication);
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        String jwtToken = jwtUtils.generateTokenFromUsername(userDetails);

        Map<String, Object> response = new HashMap<>();
        response.put("accessToken", jwtToken);
        return ResponseEntity.ok(response);
    }
}
```


## 使用示例

1. 调用登录接口获取令牌：![file-20250903212006922](assets/file-20250903212006922.png)  
   返回：`{"accessToken": "eyJhbGciOiJIUzI1NiJ9..."}`
2. 携带令牌访问受保护接口，可以看到请求成功：![file-20250903211718163](assets/file-20250903211718163.png)

通过以上步骤，基本实现基于 JWT 的无状态认证，适配前后端分离
