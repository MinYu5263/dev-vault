
## RestTemplate

### 什么是 RestTemplate？

`RestTemplate` 是 Spring 框架提供的一个用于发送 HTTP 请求的模板类，它简化了与 RESTful 服务的交互过程。通过 `RestTemplate`，开发者可以方便地发送 GET、POST、PUT、DELETE 等 HTTP 请求，并处理响应结果。

### 依赖配置

`RestTemplate`属于Spring框架的`spring-web`模块，一般项目都会引用web依赖，所以一般情况是无需额外引入依赖的。

#### 非 Spring Boot 项目

需要手动引入 `spring-web` 依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.3.20</version>
</dependency>
```

#### Spring Boot 项目

如果是Spring Boot项目，请确保引入了`spring-boot-starter-web`，这个依赖中包含了RestTemplate。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### RestTemplate 配置

`RestTemplate` 需要配置为 Spring 容器中的 Bean 才能使用，以下是几种常见配置方式：

#### 默认配置（使用 JDK 自带的 HttpURLConnection）

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        // 配置请求工厂，设置超时时间
        SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
        requestFactory.setConnectTimeout(5000);  // 连接超时时间（毫秒）
        requestFactory.setReadTimeout(5000);     // 数据读取超时时间（毫秒）
        
        // 创建 RestTemplate 实例并设置请求工厂
        return new RestTemplate(requestFactory);
    }
}
```

#### 使用 HttpClient 作为底层客户端

1. 引入 HttpClient 依赖

```xml
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
    <version>5.3</version>
</dependency>
```

2. 配置 RestTemplate

```java
import org.apache.hc.client5.http.classic.HttpClient;
import org.apache.hc.client5.http.config.RequestConfig;
import org.apache.hc.client5.http.impl.classic.HttpClientBuilder;
import org.apache.hc.core5.util.Timeout;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {
        // 创建 HttpClient 实例
        HttpClient httpClient = HttpClientBuilder.create()
                .setDefaultRequestConfig(getRequestConfig())
                .build();
        
        // 创建 HttpComponentsClientHttpRequestFactory 实例
        HttpComponentsClientHttpRequestFactory requestFactory = 
            new HttpComponentsClientHttpRequestFactory(httpClient);
        
        // 创建并返回 RestTemplate 实例
        return new RestTemplate(requestFactory);
    }
    
    /**
     * 配置请求参数
     */
    private RequestConfig getRequestConfig() {
        return RequestConfig.custom()
            .setConnectTimeout(Timeout.ofMilliseconds(5000))  // 连接超时
            .setConnectionRequestTimeout(Timeout.ofMilliseconds(5000))  // 请求超时
            .setResponseTimeout(Timeout.ofMilliseconds(5000))  // 响应超时
            .build();
    }
}
```

#### 使用 OkHttp 作为底层客户端

1. 引入 OkHttp 依赖

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.10.0</version>
</dependency>
```

2. 配置 RestTemplate

```java
import okhttp3.OkHttpClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.OkHttp3ClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

import java.util.concurrent.TimeUnit;

@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {
        // 创建 OkHttpClient 实例并配置超时
        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .connectTimeout(5, TimeUnit.SECONDS)    // 连接超时
                .writeTimeout(5, TimeUnit.SECONDS)     // 写入超时
                .readTimeout(5, TimeUnit.SECONDS)      // 读取超时
                .build();
        
        // 创建 OkHttp3ClientHttpRequestFactory 实例
        OkHttp3ClientHttpRequestFactory requestFactory = 
            new OkHttp3ClientHttpRequestFactory(okHttpClient);
        
        // 创建并返回 RestTemplate 实例
        return new RestTemplate(requestFactory);
    }
}
```

### RestTemplate 基本使用

首先在需要使用的类中注入 `RestTemplate`：

```java
@Autowired
private RestTemplate restTemplate;
```

#### GET请求

GET 请求主要用于获取资源，常用方法有：

- `getForObject()`: 直接返回响应体内容
- `getForEntity()`: 返回包含响应体、状态码、响应头等信息的 `ResponseEntity` 对象
##### 示例 1：无参数的 GET 请求

```java
/**
 * 无参数的 GET 请求
 */
public void testGetWithoutParams() {
    String url = "http://localhost:8080/api/users";
    
    // 使用 getForObject() 直接获取响应体
    User[] users = restTemplate.getForObject(url, User[].class);
    
    // 使用 getForEntity() 获取完整响应信息
    ResponseEntity<User[]> responseEntity = restTemplate.getForEntity(url, User[].class);
    int statusCode = responseEntity.getStatusCodeValue(); // 获取状态码
    User[] responseBody = responseEntity.getBody(); // 获取响应体
    HttpHeaders headers = responseEntity.getHeaders(); // 获取响应头
}
```

##### 示例 2：带路径参数的 GET 请求

```java
/**
 * 带路径参数的 GET 请求
 */
public void testGetWithPathParams() {
    // 使用 {1}, {2} 作为参数占位符, 数字占位符必须从1开始且连续
    String url = "http://localhost:8080/api/users/{1}/{2}";
    
    // 按顺序传递参数
    User user = restTemplate.getForObject(url, User.class, 1, "admin");
    
    // 或者使用 Map 传递参数
    String url2 = "http://localhost:8080/api/users/{id}/{username}";
    Map<String, Object> params = new HashMap<>();
    params.put("id", 1);
    params.put("username", "admin");
    User user2 = restTemplate.getForObject(url2, User.class, params);
}
```

##### 示例 3：带查询参数的 GET 请求

```java
/**
 * 带查询参数的 GET 请求
 */
public void testGetWithQueryParams() {
    String url = "http://localhost:8080/api/users?name={name}&age={age}";
    
    Map<String, Object> params = new HashMap<>();
    params.put("name", "张三");
    params.put("age", 25);
    
    User[] users = restTemplate.getForObject(url, User[].class, params);
}
```

#### POST请求

POST 请求主要用于创建资源，常用方法有：

- `postForObject()`: 发送 POST 请求并返回响应体
- `postForEntity()`: 发送 POST 请求并返回完整的 `ResponseEntity`
- `postForLocation()`: 发送 POST 请求并返回新资源的 URI

##### 示例 1：提交 JSON 数据

```java
/**
 * 发送 POST 请求提交 JSON 数据
 */
public void testPostJson() {
    String url = "http://localhost:8080/api/users";
    
    // 创建请求体对象
    User user = new User();
    user.setName("张三");
    user.setAge(25);
    
    // 设置请求头为 JSON
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
    HttpEntity<User> request = new HttpEntity<>(user, headers);
    
    // 发送 POST 请求
    User createdUser = restTemplate.postForObject(url, request, User.class);
    
    // 或者使用 postForEntity 获取完整响应
    ResponseEntity<User> responseEntity = restTemplate.postForEntity(url, request, User.class);
}
```

##### 示例 2：提交表单数据

```java
/**
 * 发送 POST 请求提交表单数据
 */
public void testPostForm() {
    String url = "http://localhost:8080/api/login";
    
    // 创建表单参数
    MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
    params.add("username", "admin");
    params.add("password", "123456");
    
    // 设置请求头为表单类型
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
    HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(params, headers);
    
    // 发送 POST 请求
    ResponseEntity<LoginResponse> response = restTemplate.postForEntity(url, request, LoginResponse.class);
}
```

#### PUT请求

PUT 请求主要用于更新资源，常用方法为 `put()`：

```java
/**
 * 发送 PUT 请求更新资源
 */
public void testPut() {
    String url = "http://localhost:8080/api/users/{id}";
    
    // 创建更新对象
    User user = new User();
    user.setName("张三");
    user.setAge(26);
    
    // 设置请求头
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
    HttpEntity<User> request = new HttpEntity<>(user, headers);
    
    // 发送 PUT 请求
    restTemplate.put(url, request, 1); // 1 是路径参数 id 的值
}
```

#### DELETE请求

DELETE 请求主要用于删除资源，常用方法为 `delete()`：

```java
/**
 * 发送 DELETE 请求删除资源
 */
public void testDelete() {
    String url = "http://localhost:8080/api/users/{id}";
    
    // 发送 DELETE 请求
    restTemplate.delete(url, 1); // 1 是要删除的资源 ID
}
```

#### 通用请求方法 exchange ()

`exchange()` 方法是一种通用的请求方法，可以处理各种 HTTP 方法，灵活性更高：

```java
/**
 * 使用 exchange() 方法发送请求
 */
public void testExchange() {
    String url = "http://localhost:8080/api/users/{id}";
    
    // 设置请求头
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
    headers.set("Authorization", "Bearer token123");
    
    // 创建请求实体
    HttpEntity<Void> request = new HttpEntity<>(headers);
    
    // 发送 GET 请求
    ResponseEntity<User> response = restTemplate.exchange(
        url, 
        HttpMethod.GET,  // 指定 HTTP 方法
        request, 
        User.class, 
        1  // 路径参数
    );
}
```

#### 文件上传与下载

##### 文件上传

```java
/**
 * 文件上传
 */
public void testFileUpload() {
    String url = "http://localhost:8080/api/upload";
    
    // 创建文件资源
    Resource fileResource = new FileSystemResource("D:/test.jpg");
    
    // 设置表单参数
    MultiValueMap<String, Object> params = new LinkedMultiValueMap<>();
    params.add("file", fileResource);
    params.add("description", "测试文件");
    
    // 设置请求头
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.MULTIPART_FORM_DATA);
    HttpEntity<MultiValueMap<String, Object>> request = new HttpEntity<>(params, headers);
    
    // 发送文件上传请求
    ResponseEntity<String> response = restTemplate.postForEntity(url, request, String.class);
}
```

##### 文件下载

```java
/**
 * 文件下载
 */
public void testFileDownload() {
    String url = "http://localhost:8080/api/download/{fileId}";
    
    // 发送下载请求
    ResponseEntity<Resource> response = restTemplate.getForEntity(url, Resource.class, 1);
    
    // 获取响应体（文件资源）
    Resource resource = response.getBody();
    if (resource != null) {
        try {
            // 将文件保存到本地
            Files.copy(resource.getInputStream(), Paths.get("D:/download.jpg"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### 异常处理

```java
/**
 * 异常处理示例
 */
public void testExceptionHandling() {
    String url = "http://localhost:8080/api/users/{id}";
    
    try {
        User user = restTemplate.getForObject(url, User.class, 999);
    } catch (HttpClientErrorException e) {
        // 处理 4xx 错误
        System.out.println("HTTP 状态码: " + e.getRawStatusCode());
        System.out.println("错误信息: " + e.getStatusText());
    } catch (HttpServerErrorException e) {
        // 处理 5xx 错误
        System.out.println("服务器错误: " + e.getStatusText());
    } catch (RestClientException e) {
        // 处理其他 REST 客户端异常
        e.printStackTrace();
    }
}
```

## WebClient

## RestClient

### 什么是 RestClient？

`RestClient` 是 Spring Framework 6.1 及 Spring Boot 3.2 引入的新一代 HTTP 客户端工具，用于替代传统的 `RestTemplate`。它采用流畅的链式 API 设计，支持同步请求处理，同时提供更简洁的语法和更灵活的响应处理方式，是 Spring 官方推荐的现代 HTTP 客户端方案。

## Apache HttpClient

## OkHttp
