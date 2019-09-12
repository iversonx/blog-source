---
title: 在老项目中加入Spring Security OAuth2.0实现APP权限认证
date: 2019-06-28
categories: http
tags: 学习笔记
description: 
---

## 1. 背景

前一阵子公司自研的一款APP，被人恶意刷api调用，导致服务器宕机。经追踪发现主要原因有两个：

1. 服务端与APP之间采用的是HTTP协议进行通信；

2. 服务端与APP之间的身份认证采用的原始简单的用户名密码方式。

起码由于事件紧急，以及其他原因，临时解决方案是更换用户密码加密方法；二增加一台服务器。

事件之后，我一直觉得紧急解决方案无法从根源上解决服务器API被恶意频繁调用问题。于是尝试在现有项目上使用身份认证机制取代原始简单的用户密码方式来对APP进行认证。

这个项目使用的Spring框架的版本是4.2.5。 

## 2. 开始改造

### 2.1. 在项目中Maven依赖

```java
<dependency>

    <groupId>org.springframework.security</groupId>

    <artifactId>spring-security-web</artifactId>

    <version>4.2.10.RELEASE</version>

</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>

    <artifactId>spring-security-config</artifactId>

    <version>4.2.10.RELEASE</version>

</dependency>

<dependency>
    <groupId>org.springframework.security.oauth</groupId>

    <artifactId>spring-security-oauth2</artifactId>

    <version>2.2.0.RELEASE</version>

</dependency>
```

因为服务端和APP都是自家研发的，所以我采用OAuth2 密码模式(resource owner password credentials)

### 2.2. 配置认证服务器和资源服务器

#### 2.2.1 认证服务器

```java
// 认证服务器
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    @Autowired
    private DataSource dataSource;

    @Autowired
    private TokenStore tokenStore;

    // 定义TokenStore存储方式在，这里将token存储到数据库
    @Bean
    public TokenStore tokenStore() {
        return new JdbcTokenStore(dataSource);
    }


    // 数据源配置

    @Bean
    public DataSource dataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setUrl("");

        dataSource.setDriverClassName("");

        dataSource.setUsername("");

        dataSource.setPassword("");

        return dataSource;
    }

    // AuthenticationManager用于处理身份验证请求
    // 此案例注入的authenticationManager在WebSecurityConfig配置类中定义

    @Autowired
    private AuthenticationManager authenticationManager;

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        // 配置身份认证处理器

        endpoints.authenticationManager(authenticationManager);

        // 配置TokenServices参数
        DefaultTokenServices tokenServices = new DefaultTokenServices();
        // token存储方式

        tokenServices.setTokenStore(tokenStore);
        // 支持刷新token
        tokenServices.setSupportRefreshToken(true);


        // 设置token的有效期,单位是秒，这里配置了60s

        tokenServices.setAccessTokenValiditySeconds(60);
        endpoints.tokenServices(tokenServices);
    }


    @Override
    public void configure(ClientDetailsServiceConfigurer clients)
            throws Exception {
        // 配置客户端信息从数据库中获取    
        clients.jdbc(dataSource);

    }
}
```

上面的配置类，因为指定了客户端信息和token信息是从数据库中获取，因此需要创建相关的数据库表，spring-security-oauth官方为我们提供创建相关表的脚本,https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/test/resources/schema.sql

在这个案例中，我们只用到`oauth_client_details`,`oauth_access_token`,`oauth_refresh_token`这三张表

```sql
CREATE TABLE `oauth_access_token` (
  `token_id` varchar(256) DEFAULT NULL COMMENT '令牌id',
  `token` blob COMMENT '令牌',
  `authentication_id` varchar(128) NOT NULL COMMENT '认证id',
  `user_name` varchar(256) DEFAULT NULL COMMENT '用户名',
  `client_id` varchar(128) DEFAULT NULL COMMENT '客户端id',
  `authentication` blob COMMENT '认证信息',
  `refresh_token` varchar(256) DEFAULT NULL COMMENT '刷新令牌',
  PRIMARY KEY (`authentication_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='访问令牌';

CREATE TABLE `oauth_client_details` (
  `client_id` varchar(128) NOT NULL COMMENT '客户端id',
  `resource_ids` varchar(256) DEFAULT NULL COMMENT '资源id',
  `client_secret` varchar(256) DEFAULT NULL COMMENT '授权密钥',
  `scope` varchar(256) DEFAULT NULL COMMENT '范围',
  `authorized_grant_types` varchar(256) DEFAULT NULL COMMENT '授权类型',
  `web_server_redirect_uri` varchar(256) DEFAULT NULL COMMENT 'url',
  `authorities` varchar(256) DEFAULT NULL COMMENT '授权',
  `access_token_validity` int(11) DEFAULT NULL COMMENT '令牌有效时间',
  `refresh_token_validity` int(11) DEFAULT NULL COMMENT '刷新令牌有效时间',
  `additional_information` varchar(4096) DEFAULT NULL COMMENT '扩展信息',
  `autoapprove` varchar(256) DEFAULT NULL COMMENT '自动审核',
  PRIMARY KEY (`client_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='授权客户端明细';

CREATE TABLE `oauth_refresh_token` (
  `id` bigint(64) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id',
  `token_id` varchar(256) DEFAULT NULL COMMENT '令牌id',
  `token` blob COMMENT '令牌',
  `authentication` blob COMMENT '授权信息',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='刷新令牌';
```

以上3个表的详细描述可以参考http://andaily.com/spring-oauth-server/db_table_description.html

因为我配置了客户端信息从数据库都，因此需要在`oauth_client_details`表插入一条数据。

```sql
INSERT INTO `oauth_client_details` ( `client_id`, `client_secret`, `scope`, `authorized_grant_types`, `authorities`, `autoapprove` )
VALUES( 'app', '123456', 'all', 'password,refresh_token', '{}', 'true' );
```

#### 2.2.2 资源服务器

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        // 这里配置表示所有/v1/开头的url都需要进行认证后才可以访问

        http.authorizeRequests()
                .anyRequest()
                .authenticated()
                .and()
                .requestMatchers()
                .antMatchers("/v1/**");
    }
}
```

#### 2.2.3 WebSecurityConfig

因为我们Web端和APP 接口在同一个工程中，因此为了不影响Web端，需要增加以下配置

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsService userService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 禁用CSRF支持，默认开启
        // 开启时，可以遇到
        // Could not verify the provided CSRF token because your session was not found 错误

        http.csrf().disable();
        // 因为Web页面使用iframe，而 Spring Security默认禁用iframe的内容，所以页面可能无法正常显示。
        // 该配置阻止将X-Frame-Options头，写入响应头中

        http.headers().frameOptions().disable();


    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() 
        throws Exception {
        AuthenticationManager authenticationManager 
            = super.authenticationManagerBean();
        return authenticationManager;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) 
        throws Exception {
        // 指定加载用户数据的接口

        auth.userDetailsService(userService);
    }

}
```

#### 2.2.4 实现加载用户数据的接口

```java
public class UserPrincipal implements UserDetails {
    private static final long 
        serialVersionUID = 5564363430024390596L;
    // 系统原先的用户信息类
    // 自定义    

    private User user;

    public UserPrincipal(User user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> 
        getAuthorities() {
        // 该用户拥有的权限
        // 权限通常从数据库获取
        // 如果没有返回null或空list

        List<GrantedAuthority> authorities = 
            new ArrayList<GrantedAuthority>();
        return authorities;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
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

    public User getUser() {
        return user;
    }
}


@Service
public class UserDetailsServiceImpl implements UserDetailsService {
    @Autowired
    private UserService userService;
    @Override
    public UserDetails loadUserByUsername(String username) 
        throws UsernameNotFoundException {
        // 这里需要自己定义根据用户名从数据库获取用户信息包括存储的密码
        // Security会对用户密码进行校验

        User user = userService.getUserByUserName(username);
        if (user == null) {
            throw new UsernameNotFoundException(username);
        }
        return new UserPrincipal(user);
    }
}
```

很多博文，通常介绍到这里就完事了。那是因为现在通常都是使用的是`spring-boot`。而我的项目使用的SpringMvc，需要配置过滤器，所有还需要修改`web.xml`

#### 2.2.5 在web.xml中添加过滤器

```xml
<filter>
    <filter-name>springSecurityFilterChain</filter-name>

    <filter-class>
          org.springframework.web.filter.DelegatingFilterProxy    
    </filter-class>

</filter>


<filter-mapping>

    <filter-name>springSecurityFilterChain</filter-name>

    <url-pattern>/*</url-pattern>

</filter-mapping>
```

### 2.3. 验证

#### 2.3.1 获取token

```http
curl -X POST --user app:123456 http://localhost:8080/oauth/token
-H "accept: application/json" -H "content-type: application/x-www-form-urlencoded" -d "grant_type=password&username=admin&password=123456&scope=all"

# 响应
{
    "access_token": "a2dd17e2-ab7a-4759-923e-f13c0691ee53",
    "token_type": "bearer",
    "refresh_token": "a7711f78-f798-4fc0-a126-bedaa18d6e4d",
    "expires_in": 59,
    "scope": "all"
}
```

这里`app:123456`是表`oauth_client_details`中`client_id`和`client_secret`字段的值。

此时`oauth_access_token`和`oauth_refresh_token`表中会插入一条数据。

然后在访问`/v1/**`请求时，将access_token带上,例如

`http://localhost:8080/v1/userinfo?access_token=a2dd17e2-ab7a-4759-923e-f13c0691ee53`

#### 2.3.2 刷新token

当token过期时，我们需要重新获取token，这时有两种方式获取token：

1. 采用2.3.1方式获取新的token；

2. 如果开启了refresh token，则refresh_token获取新的token。

```http
curl -X POST --user app:123456 http://localhost:8080/oauth/token 
-H "accept: application/json" -H "content-type: application/x-www-form-urlencoded" 
-d "grant_type=refresh_token&refresh_token=a7711f78-f798-4fc0-a126-bedaa18d6e4d"

# 响应
{
    "access_token": "2b8b466c-213d-499f-acb9-eeb9cbc1a895",
    "token_type": "bearer",
    "refresh_token": "a7711f78-f798-4fc0-a126-bedaa18d6e4d",
    "expires_in": 59,
    "scope": "all"
}
```
