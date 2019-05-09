# Shiro在Spring Boot中的应用

[Apache Shiro](https://shiro.apache.org/)是一个强大且易用的Java安全框架,执行身份验证、授权、密码和会话管理。本文记录一下shiro在Spring Boot中的使用。官网也提供了简洁的[教程](https://shiro.apache.org/spring-boot.html)。

- 将shiro依赖添加到spring boot项目中，官方提供了spring boot starter。

  ``` xml
  <dependency>
      <groupId>org.apache.shiro</groupId>
      <artifactId>shiro-spring-boot-starter</artifactId>
      <version>1.4.1-SNAPSHOT</version>
  </dependency>
  ```

- 提供Realm的实现，使用java代码配置方式指定Realm。

  指定Realm

  ``` java
  @Bean
  public Realm realm() {
    return new MyShiroRealm();
  }
  ```

  自定义实现Realm。

  ``` java
  /**
   * 自定义Realm
   * Realm：域，Shiro 从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager
   * 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法；
   * 也需要从 Realm 得到用户相应的角色/权限进行验证用户是否能进行操作；可以把 Realm 看
   * 成 DataSource，即安全数据源
   */
  public class MyShiroRealm extends AuthorizingRealm {
  
      /**
       * 注入用户相关的服务。根据项目实际情况处理
       */
      @Autowired
      private ISSvnAccountService accountService;
  	/**
  	 * 用户授权信息，保存用户的role和permission
  	 **/
      @Override
      protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
          SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
          // 本项目中的用户对象为 SSvnAccount，根据项目实际情况处理
          SSvnAccount userInfo = (SSvnAccount) principals.getPrimaryPrincipal();
          // 获取该用户的角色保存到authorizationInfo，也可以保存permission，根据项目权限细粒度处理
          authorizationInfo.addRole(String.valueOf(userInfo.getRole().getValue()));
          return authorizationInfo;
      }
  
      /**
       * 用户身份认证信息，用于登录校验。
       **/
      @Override
      protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token)
              throws AuthenticationException {
          //获取用户的输入的账号.
          String username = (String) token.getPrincipal();
          //通过username从存储介质中查找 User对象.
          //实际项目中，这里可以根据实际情况做缓存
          EntityWrapper<SSvnAccount> ew = new EntityWrapper<>();
          ew.eq(SSvnAccount.COLUMN_USER_ID,username);
          SSvnAccount userInfo = accountService.selectOne(ew);
          if (userInfo == null) {
              throw new UnknownAccountException("用户不存在！");
          }
          return new SimpleAuthenticationInfo(
                  userInfo, // 用户名
                  userInfo.getSvnPwd(), // 密码
                  ByteSource.Util.bytes(userInfo.getSvnUser()), // salt=username+salt
                  getName()  // realm name
          );
      }
  }
  ```

- 配置过滤器链，用于设置路径与过滤器的映射，实现不同的路径对应不同的权限。

  ``` java
  @Bean
  public ShiroFilterChainDefinition shiroFilterChainDefinition() {
      DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
     
      // 设置admin前缀的路径访问权限是登陆用户并且角色为admin
      chainDefinition.addPathDefinition("/admin/**", "authc, roles[admin]");
      
      // 设置docs前缀的路径访问权限是登陆用户并且权限为有文档读写功能
      chainDefinition.addPathDefinition("/docs/**", "authc, perms[document:read]");
      
      // 其他的所有访问路径必须是登陆用户
      chainDefinition.addPathDefinition("/**", "authc");
      return chainDefinition;
  }
  ```

- 使用Shiro注解

  使用Shiro的注解进行安全检查（例如，@RequiresRoles、@RequiresPermissions等）。这些注释在starter中以及自动配置启用了。

  ``` java
  @Controller
  public class AccountInfoController {
  
      @RequiresRoles("admin")
      @RequestMapping("/admin/config")
      public String adminConfig(Model model) {
          return "view";
      }
  }
  ```

## 配置参数

| Key                                               | 默认值       | 描述                                                   |
| ------------------------------------------------- | ------------ | ------------------------------------------------------ |
| shiro.enabled                                     | `true`       | 启用Shiro spring模块                                   |
| shiro.web.enabled                                 | `true`       | 启用Shiro的Spring Web模块                              |
| shiro.annotations.enabled                         | `true`       | 为Shiro的注解启用Spring支持                            |
| shiro.sessionManager.deleteInvalidSessions        | `true`       | 从会话存储中删除无效会话                               |
| shiro.sessionManager.sessionIdCookieEnabled       | `true`       | 启用会话ID到cookie，以进行会话跟踪                     |
| shiro.sessionManager.sessionIdUrlRewritingEnabled | `true`       | 启用会话URL重写支持                                    |
| shiro.userNativeSessionManager                    | `false`      | 如果启用，Shiro将管理HTTP会话而不是容器                |
| shiro.sessionManager.cookie.name                  | `JSESSIONID` | 会话cookie名称                                         |
| shiro.sessionManager.cookie.maxAge                | `-1`         | 会话cookie最长有效期                                   |
| shiro.sessionManager.cookie.domain                | null         | 会话cookie域                                           |
| shiro.sessionManager.cookie.path                  | null         | 会话cookie路径                                         |
| shiro.sessionManager.cookie.secure                | `false`      | 会话cookie安全标志                                     |
| shiro.rememberMeManager.cookie.name               | `rememberMe` | RememberMe cookie名称                                  |
| shiro.rememberMeManager.cookie.maxAge             | one year     | RememberMe cookie最长有效期                            |
| shiro.rememberMeManager.cookie.domain             | null         | RememberMe cookie域名                                  |
| shiro.rememberMeManager.cookie.path               | null         | RememberMe cookie路径                                  |
| shiro.rememberMeManager.cookie.secure             | `false`      | RememberMe cookie安全标志                              |
| shiro.loginUrl                                    | `/login.jsp` | 未经身份验证的用户重定向到登录页面时使用的登录URL      |
| shiro.successUrl                                  | `/`          | 用户登录后的默认登录页面（如果在当前会话中找不到替代） |
| shiro.unauthorizedUrl                             | null         | 页面将用户重定向到未经授权的页面（403页）              |

