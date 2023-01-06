---
title: SpringBoot2.X Security Oauth2 JWT
date: 2020-10-20 15:47:00
tags:
    - java
categories: java
---

# 简介
[官网地址](hhttps://spring.io/projects/spring-security)

[官方文档](https://docs.spring.io/spring-security/site/docs/4.0.x/reference/html/introduction.html#what-is-acegi-security)

Spring Security，这是一种基于 Spring AOP 和 Servlet 过滤器的安全框架。它提供全面的安全性解决方案，同时在 Web 请求级和方法调用级处理身份确认和授权。


Spring Security当前支持与所有以下技术的身份验证集
- HTTP BASIC authentication headers (an IETF RFC-based standard)
- HTTP Digest authentication headers (an IETF RFC-based standard)
- HTTP X.509 client certificate exchange (an IETF RFC-based standard)
- LDAP (a very common approach to cross-platform authentication needs, especially in large environments)
- Form-based authentication (for simple user interface needs)
- OpenID authentication
- Authentication based on pre-established request headers (such as Computer Associates Siteminder)
- JA-SIG Central Authentication Service (otherwise known as CAS, which is a popular open source single sign-on system)
- Transparent authentication context propagation for Remote Method Invocation (RMI) and HttpInvoker (a Spring remoting protocol)
- Automatic "remember-me" authentication (so you can tick a box to avoid re-authentication for a predetermined period of time)
- Anonymous authentication (allowing every unauthenticated call to automatically assume a particular security identity)
- Run-as authentication (which is useful if one call should proceed with a different security identity)
- Java Authentication and Authorization Service (JAAS)
- JEE container autentication (so you can still use Container - Managed Authentication if desired)
- Kerberos
- Java Open Source Single Sign On (JOSSO) *
- OpenNMS Network Management Platform *
- AppFuse *
- AndroMDA *
- Mule ESB *
- Direct Web Request (DWR) *
- Grails *
- Tapestry *
- JTrac *
- Jasypt *
- Roller *
- Elastic Path *
- Atlassian Crowd *
- Your own authentication systems (see below) 

涉及以下系统组件：
客户端：它可以是任何平台上的任何Web服务使用者。简而言之，它可以是另一个WebService，UI应用程序或Mobile平台，它们希望通过应用程序以安全的方式读写数据。
授权服务器：验证用户凭据，并颁发令牌。它根据不同的授予类型发行令牌。下面列出了最常见的[OAuth 2.0](hhttps://tools.ietf.org/html/draft-ietf-oauth-v2-31)授权类型：
- Authorization Code
- Implicit
- Password
- Client Credentials
- Device Code
- Refresh Token

名词解释可以看[SpringOauth2官方文档](https://projects.spring.io/spring-security-oauth/docs/oauth2.html)

# 架构设计
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/spring-boot-authentication-spring-security-architecture.png)

## 名词解释

– WebSecurityConfigurerAdapter 是我们安全实施的关键。它提供HttpSecurity配置以配置cors，csrf，会话管理以及受保护资源的规则。我们还可以扩展和定制包含以下元素的默认配置。

– UserDetailsService 接口有一种方法可以按用户名加载User并返回UserDetailsSpring Security可以用于认证和验证的对象。

– UserDetails 包含用于构建身份验证对象的必要信息（例如：用户名，密码，授权机构）。

– UsernamePasswordAuthenticationToken 从登录请求中获取{用户名，密码}，AuthenticationManager将使用它来认证登录帐户。

– AuthenticationManager 有一个DaoAuthenticationProvider（与帮助UserDetailsService＆PasswordEncoder）来验证UsernamePasswordAuthenticationToken对象。如果成功，则AuthenticationManager返回完全填充的身份验证对象（包括授予的权限）。

– OncePerRequestFilter 对我们API的每个请求执行一次。它提供了一种doFilterInternal()方法，我们将实现解析和验证JWT，加载用户详细信息（使用UserDetailsService），检查授权（使用UsernamePasswordAuthenticationToken）。

– AuthenticationEntryPoint 当客户端未经身份验证访问受保护的资源时，将捕获未经授权的错误并返回401。

## 认证流程
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/oauth-auth-code_c4oi91.png)

# 实践
## 引入maven包
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>org.springframework.security.oauth2</groupId>
	<artifactId>spring-security-oauth2-parent</artifactId>
	<version>1.0.0.BUILD-SNAPSHOT</version>
	<packaging>pom</packaging>

	<modules>
		<module>auth-server</module>
		<module>client-app</module>
		<module>resource-server</module>
	</modules>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.security.oauth</groupId>
				<artifactId>spring-security-oauth2</artifactId>
				<version>2.4.0.RELEASE</version>
			</dependency>
			<dependency>
				<groupId>org.springframework.security.oauth.boot</groupId>
				<artifactId>spring-security-oauth2-autoconfigure</artifactId>
				<version>2.2.1.RELEASE</version>
			</dependency>
			<dependency>
				<groupId>org.springframework.security</groupId>
				<artifactId>spring-security-jwt</artifactId>
				<version>1.1.0.RELEASE</version>
			</dependency>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-data-redis</artifactId>
				<version>2.2.1.RELEASE</version>
			</dependency>
			<dependency>
				<groupId>io.jsonwebtoken</groupId>
				<artifactId>jjwt</artifactId>
				<version>0.9.1</version>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

核心代码：
```
package org.springframework.security.oauth.config;


import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@Configuration
public class CustomerAuthenticationEntryPointConfiguration implements AuthenticationEntryPoint {


    @Override
    public void commence(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
        //处理异常消息通知
        httpServletResponse.getWriter().write("暂无权限访问");
    }
}

```
```
package org.springframework.security.oauth.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.User;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@Configuration
public class CustomerAuthenticationProviderConfiguration implements AuthenticationProvider {
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        //这里做自定义的用户校验,可以对用户进行继承扩展
        User user = new User((String) authentication.getPrincipal(), (String) authentication.getCredentials(), AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
        //这里可以通过继承AbstractAuthenticationToken进行扩展
        UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken = new UsernamePasswordAuthenticationToken(authentication.getPrincipal(), authentication.getCredentials());
        usernamePasswordAuthenticationToken.setDetails(user);
        SecurityContextHolder.getContext().setAuthentication(usernamePasswordAuthenticationToken);
        return usernamePasswordAuthenticationToken;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return false;
    }
}

```
```
package org.springframework.security.oauth.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@Configuration
public class CustomerAuthenticationManager implements AuthenticationManager {

    @Autowired
    private CustomerAuthenticationProviderConfiguration customerAuthenticationProviderConfiguration;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        return customerAuthenticationProviderConfiguration.authenticate(authentication);
    }
}
```
```
package org.springframework.security.oauth.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.oauth.properties.Oauth2ClientProperties;
import org.springframework.security.oauth.properties.Oauth2Properties;
import org.springframework.security.oauth2.config.annotation.builders.InMemoryClientDetailsServiceBuilder;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@Configuration
@EnableAuthorizationServer
public class CustomerAuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private TokenStore tokenStore;
    @Autowired
    private JwtAccessTokenConverter jwtAccessTokenConverter;
    @Autowired
    private CustomerAuthenticationManager customerAuthenticationManager;
    @Autowired
    private Oauth2Properties oauth2Properties;
    @Autowired
    private CustomerWebResponseExceptionTranslator customerWebResponseExceptionTranslator;

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer authorizationServerEndpointsConfigurer) {
        authorizationServerEndpointsConfigurer.tokenEnhancer(jwtAccessTokenConverter)
                .tokenStore(tokenStore)
                .authenticationManager(customerAuthenticationManager)
                .exceptionTranslator(customerWebResponseExceptionTranslator);
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer authorizationServerSecurityConfigurer) throws Exception {
        authorizationServerSecurityConfigurer.tokenKeyAccess("permitAll()")
                .allowFormAuthenticationForClients()
                .checkTokenAccess("isAuthenticated");
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clientDetailsServiceConfigurer) throws Exception {
        InMemoryClientDetailsServiceBuilder inMemoryClientDetailsServiceBuilder = clientDetailsServiceConfigurer.inMemory();
        for (Oauth2ClientProperties oauth2ClientProperties : oauth2Properties.getClients()) {
            inMemoryClientDetailsServiceBuilder.withClient(oauth2ClientProperties.getClientId())
                    .secret(bCryptPasswordEncoder().encode(oauth2ClientProperties.getClientSecret()))
                    .accessTokenValiditySeconds(oauth2ClientProperties.getAccessTokenValiditySeconds())
                    .refreshTokenValiditySeconds(oauth2ClientProperties.getRefreshTokenValiditySeconds())
                    .authorizedGrantTypes("refresh_token", "password", "authorization_code")
                    .scopes(oauth2ClientProperties.getScopes());
        }
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```
```
package org.springframework.security.oauth.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.oauth.filter.CustomerOauth2AuthenticationProcessingFilter;
import org.springframework.security.web.authentication.AnonymousAuthenticationFilter;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@Configuration
@EnableWebSecurity
public class CustomerSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomerAuthenticationManager customerAuthenticationManager;
    @Autowired
    private CustomerAuthenticationEntryPointConfiguration customerAuthenticationEntryPointConfiguration;
    @Autowired
    private CustomerOauth2AuthenticationProcessingFilter customerOauth2AuthenticationProcessingFilter;

    @Override
    protected void configure(HttpSecurity httpSecurity) throws  Exception{
        httpSecurity.authorizeRequests().anyRequest().authenticated().and().addFilterBefore(customerOauth2AuthenticationProcessingFilter, AnonymousAuthenticationFilter.class)
                .csrf().disable().exceptionHandling().authenticationEntryPoint(customerAuthenticationEntryPointConfiguration);
    }

    @Override
    public AuthenticationManager authenticationManager(){
        return customerAuthenticationManager;
    }

}
```
```
package org.springframework.security.oauth.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.http.ResponseEntity;
import org.springframework.security.oauth2.common.exceptions.OAuth2Exception;
import org.springframework.security.oauth2.provider.error.WebResponseExceptionTranslator;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@Configuration
public class CustomerWebResponseExceptionTranslator implements WebResponseExceptionTranslator<OAuth2Exception> {
    @Override
    public ResponseEntity<OAuth2Exception> translate(Exception e) throws Exception {
        //自定义认证失败信息
        return null;
    }
}
```
```
package org.springframework.security.oauth.config.token;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AbstractAuthenticationToken;
import org.springframework.security.oauth2.provider.OAuth2Authentication;
import org.springframework.security.oauth2.provider.token.AuthenticationKeyGenerator;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@Configuration
public class CustomerAuthenticationKeyGeneratorConfiguration implements AuthenticationKeyGenerator {


    @Override
    public String extractKey(OAuth2Authentication authentication) {
        //自定义token生成，可以用来实现ip控制登录用户只能有一个，顶掉登录
        return null;
    }
}

```
```
package org.springframework.security.oauth.config.token;

import io.jsonwebtoken.JwtHandlerAdapter;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.common.OAuth2AccessToken;
import org.springframework.security.oauth2.provider.OAuth2Authentication;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@Configuration
public class CustomerJwtAccessTokenConverterConfiguration extends JwtAccessTokenConverter {

    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken oAuth2AccessToken, OAuth2Authentication oAuth2Authentication){
       //可以实现jwt的token放一些用户信息
        return super.enhance(oAuth2AccessToken,oAuth2Authentication);
    }
}
```
```
package org.springframework.security.oauth.config.token;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.security.oauth.properties.Oauth2Properties;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.redis.RedisTokenStore;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@Configuration
public class CustomerJwtStoreConfiguration {

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;
    @Autowired
    private Oauth2Properties oauth2Properties;
    @Autowired
    private CustomerAuthenticationKeyGeneratorConfiguration customerAuthenticationKeyGeneratorConfiguration;
    @Autowired
    private CustomerJwtAccessTokenConverterConfiguration customerJwtAccessTokenConverterConfiguration;

    @Bean
    public TokenStore redisTokenStore() {
        RedisTokenStore redisTokenStore = new RedisTokenStore(redisConnectionFactory);
        redisTokenStore.setAuthenticationKeyGenerator(customerAuthenticationKeyGeneratorConfiguration);
        return redisTokenStore;
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        customerJwtAccessTokenConverterConfiguration.setSigningKey(oauth2Properties.getJwtSigningKey());
        return customerJwtAccessTokenConverterConfiguration;
    }
}
```
```
package org.springframework.security.oauth.filter;

import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.oauth.config.CustomerAuthenticationProviderConfiguration;
import org.springframework.security.oauth.properties.Oauth2Properties;
import org.springframework.stereotype.Component;
import org.springframework.util.AntPathMatcher;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;


/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@Component
public class CustomerOauth2AuthenticationProcessingFilter implements Filter, InitializingBean {

    @Autowired
    private Oauth2Properties oauth2Properties;
    @Autowired
    private CustomerAuthenticationProviderConfiguration customerAuthenticationProviderConfiguration;

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        //白名单过滤
        AntPathMatcher antPathMatcher = new AntPathMatcher();
        for (String path : oauth2Properties.getPermitUrl().split(",")) {
            if (antPathMatcher.match(path, httpServletRequest.getPathInfo())) {
                filterChain.doFilter(servletRequest, servletResponse);
            }
        }
        //自定义验证
        Authentication authentication = getFromRequest(httpServletRequest);
        if (authentication == null) {
            SecurityContextHolder.clearContext();
        } else {
            SecurityContextHolder.getContext().setAuthentication(customerAuthenticationProviderConfiguration.authenticate(authentication));
        }
        filterChain.doFilter(servletRequest, servletResponse);
    }

    private Authentication getFromRequest(HttpServletRequest httpServletRequest) {
        //todo 这里可以自己去实现
        return null;
    }

    @Override
    public void afterPropertiesSet() throws Exception {

    }
}
```
```
package org.springframework.security.oauth.properties;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
public class Oauth2ClientProperties {

    private String clientId;
    private String clientSecret;
    private String scopes;
    private Integer accessTokenValiditySeconds;
    private Integer refreshTokenValiditySeconds;


    public String getClientId() {
        return clientId;
    }

    public void setClientId(String clientId) {
        this.clientId = clientId;
    }

    public String getClientSecret() {
        return clientSecret;
    }

    public void setClientSecret(String clientSecret) {
        this.clientSecret = clientSecret;
    }

    public String getScopes() {
        return scopes;
    }

    public void setScopes(String scopes) {
        this.scopes = scopes;
    }

    public Integer getAccessTokenValiditySeconds() {
        return accessTokenValiditySeconds;
    }

    public void setAccessTokenValiditySeconds(Integer accessTokenValiditySeconds) {
        this.accessTokenValiditySeconds = accessTokenValiditySeconds;
    }

    public Integer getRefreshTokenValiditySeconds() {
        return refreshTokenValiditySeconds;
    }

    public void setRefreshTokenValiditySeconds(Integer refreshTokenValiditySeconds) {
        this.refreshTokenValiditySeconds = refreshTokenValiditySeconds;
    }
}

```
```
package org.springframework.security.oauth.properties;

import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * @author dinghuang123@gmail.com
 * @since 2020/10/20
 */
@ConfigurationProperties(prefix = "security.oauth2")
public class Oauth2Properties {

    private String jwtSigningKey;
    private String permitUrl;

    private Oauth2ClientProperties[] clients = {};

    public String getJwtSigningKey() {
        return jwtSigningKey;
    }

    public void setJwtSigningKey(String jwtSigningKey) {
        this.jwtSigningKey = jwtSigningKey;
    }

    public String getPermitUrl() {
        return permitUrl;
    }

    public void setPermitUrl(String permitUrl) {
        this.permitUrl = permitUrl;
    }

    public Oauth2ClientProperties[] getClients() {
        return clients;
    }

    public void setClients(Oauth2ClientProperties[] clients) {
        this.clients = clients;
    }
}
```
```
server:
  port: 8090
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    password:
security:
  oauth2:
    jwtSigningKey: admin
    permitUrl: /oauth/**
    clients[0]:
      clientId: client
      clientSecret: client
      scopes: all
      accessTokenValiditySeconds: 864000
      refreshTokenValiditySeconds: 864000
logging:
  level:
    root: WARN
    org.springframework.web: INFO
    org.springframework.security: INFO
    org.springframework.security.oauth2: INFO
```

## 验证
```
curl -H "Content-Type: application/json" -X POST "http://localhost:8090/oauth/token?client_id=client&client_secret=client&grant_type=pwssword&username=admin&password&admin"
```

[代码地址](https://github.com/dinghuang/spring-security-oauth2-demo.git)


# 结合网关
## 架构设计
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/viaL4.png)
