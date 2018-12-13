*《Spring Boot 实战》汪云飞*

---

### 第九章：Spring Boot 企业级开发

Spring Security是专门针对基于Spring的项目安全框架，充分利用了依赖注入和AOP来实现安全的功能。安全框架有两个重要的概念：认证(Authentication) 和 授权(Authorization)。认证即确认用户可以访问当前系统；授权即确定用户在当前系统下所拥有的功能权限。

它通过提供多一个过滤器来实现所有的安全的功能。只需要实现WebApplicationInitializer接口即可。

Spring Security的配置和Spring MVC的配置类似，只需要在配置类上注解@EnableWebSecurity并让这个类继承WebSecurityConfigurer即可。通过重写configure方法配置相关的安全配置。

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig implements WebSecurityConfigurer{
    @Autowired
    DataSource dataSource;  //获得JDBC 数据库的Bean
    
    @Override
    protected void configure(HttpSecurity http)throws Exception{
        super.configure(http); //请求授权，实现请求拦截
        //用 antMatchers Ant风格的路径匹配
        //   regexMathcers 正则表达式的匹配路径
    }
    @Override
    protected void configure(AuthticationManagerBuilder auth)throws Exception{
        super.configure(auth);
        //用户认证：认证需要有一套数据的来源，而授权则是对某个用户有相应的角色权限。
        auth.inMemeryAuthentication()  //添加内存中的用户
            .withUser("林家宝").password("123").roles("ROLE_ADMIN"); 
        //更高的版本需要用的可能不一样，
        
        auth.jdbcAuthentication().dataSource(dataSource) //指定JDBC数据库
            .usersByUserNameQuery("select username ,password from myusers where username=?")
            .authoriesByUserNameQuery("selsect username,role from roles where username=?"); //还可以定义查询用户和SQL权限。
    }
    @Override
    protected void configure(WebSecurity web)throws Exception{
        super.configure(web);
    }
}
```

关于实现其他数据库，甚至非关系数据库。需要我们自己定义UserDetailService接口。

```java
public class CustomUserService implements UserDetailsService{
    @Autowired
    SysUserRepository userRepository;
    
    @Override
    public UserDetails loadUserByUserName(String username)throws UsernameNotFoundException{
        //User来自 org.springframework.security.core.userdetails.User
        SysUser user=userRepository.findByUsername(username); //系统用户领域对象类
        List<GrantedAuthority> authorities =new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
        return new User(user.getUsername(),user.getPassword(),authorities);
    }
}
```

将我们上面写的类注册。

```java
@Bean 
UserDetailService customUserService(){
    return new CUstomUserService();
}
@Override
protected void configure(AuthenticationManagerBuilder auth)throws Exception{
    auth.userDetailsService(customeUserService());
}
```



##### Spring Boot 的支持

1. 自动配置了一个内存用户，账号为user，密码在程序启动时出现
2. 忽略 /css/** , /js/** , /**/favicon.ico 等静态文件的拦截
3. 自动配置securityFilterChainRegistration的Bean 

在application.properties中以 security 为前缀来配置。

```properties
security.user.name=user
			 .password=
			 .role
		.require-ssl=false #是否需要ssl支持
		.enable-csrl=false #是否开启 跨站请求伪造 支持，默认关闭
```

无须我们上面的各种配置。只需要配置类 继承WebSecurityConfigurer 即可。