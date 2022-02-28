### Spring Security 结合JWT实现URL动态鉴权

Spring Security是Spring家族的一个安全管理框架，主要的功能是认证和授权，与另一个安全框架shiro相比，在Spring Boot还未兴起之前，由于繁琐的配置项，导致诞生以来的很长一段时间都未获得广大研发人员的芳心，相比于shiro的轻便，Spring Security确实稍显复杂.Spring Boot崛起之后，约定大于配置的原则，之前困扰广大程序员的问题顿时迎刃而解，霎时间Spring Security便流行开来，鉴于此种情况，有必要针对Spring Seccurity相关的知识展开讲解，不过本篇教程主要从实战的角度出发，结合JWT机制来讲解Spring Security的动态鉴权。


### maven核心依赖

```
<dependencies>
        <!-- spring boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>


        <!-- mybatis plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus-boot-starter.version}</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>${guava.version}</version>
        </dependency>

        <!-- commons-lang3 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>${commons-lang3.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

            <!-- jwt相关依赖 -->
            <dependency>
                <groupId>io.jsonwebtoken</groupId>
                <artifactId>jjwt</artifactId>
                <version>${jjwt.version}</version>
            </dependency>
        </dependencies>
```

### 数据库设计

数据库的设计采用经典的rbac模型，权限的管控主要是基于角色来控制的。


```
DROP TABLE IF EXISTS `sys_menu`;
CREATE TABLE `sys_menu`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `pid` bigint(20) NULL DEFAULT 0 COMMENT '父级菜单id,一级菜单默认pid为0',
  `menu_name` varchar(30) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '菜单名称',
  `url` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '菜单或按钮的url',
  `icon` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '菜单的icon',
  `type` int(2) NULL DEFAULT NULL COMMENT '1-菜单;2-按钮',
  `sort` int(10) NULL DEFAULT NULL COMMENT '菜单的排序',
  `created_at` timestamp(0) NULL DEFAULT NULL COMMENT '创建时间',
  `updated_at` timestamp(0) NULL DEFAULT NULL COMMENT '修改时间',
  `is_deleted` tinyint(1) NULL DEFAULT 0 COMMENT '是否删除：0-正常,1-删除',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 3 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sys_role
-- ----------------------------
DROP TABLE IF EXISTS `sys_role`;
CREATE TABLE `sys_role`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `role_name` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '角色名称',
  `operator` bigint(20) NOT NULL COMMENT '操作人',
  `created_at` timestamp(0) NULL DEFAULT NULL COMMENT '创建时间',
  `updated_at` timestamp(0) NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP(0) COMMENT '修改时间',
  `is_deleted` tinyint(1) NOT NULL DEFAULT 0 COMMENT '默认0-正常，1-删除',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 6 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sys_role_menu
-- ----------------------------
DROP TABLE IF EXISTS `sys_role_menu`;
CREATE TABLE `sys_role_menu`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `role_id` bigint(20) NULL DEFAULT NULL COMMENT '角色id',
  `menu_id` bigint(20) NULL DEFAULT NULL,
  `created_at` timestamp(0) NULL DEFAULT NULL COMMENT '创建时间',
  `updated_at` timestamp(0) NULL DEFAULT NULL COMMENT '修改时间',
  `is_deleted` tinyint(1) NULL DEFAULT 0 COMMENT '0-正常；1-删除',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 5 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sys_user
-- ----------------------------
DROP TABLE IF EXISTS `sys_user`;
CREATE TABLE `sys_user`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_name` varchar(30) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '用户名',
  `password` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '密码',
  `operator` bigint(20) NULL DEFAULT NULL COMMENT '账号添加操作人',
  `created_at` timestamp(0) NULL DEFAULT NULL COMMENT '创建时间',
  `updated_at` timestamp(0) NULL DEFAULT NULL COMMENT '修改时间',
  `is_deleted` tinyint(1) NULL DEFAULT 0 COMMENT '账号是否禁用：0-正常；1-禁用',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE INDEX `un_username`(`user_name`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 9 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Table structure for sys_user_role
-- ----------------------------
DROP TABLE IF EXISTS `sys_user_role`;
CREATE TABLE `sys_user_role`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) NULL DEFAULT NULL COMMENT '用户id',
  `role_id` bigint(20) NULL DEFAULT NULL COMMENT '角色id',
  `created_at` timestamp(0) NULL DEFAULT NULL COMMENT '创建时间',
  `updated_at` timestamp(0) NULL DEFAULT NULL COMMENT '修改时间',
  `is_deleted` tinyint(1) NULL DEFAULT 0 COMMENT '是否删除:0-正常;1-删除',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 5 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;
```

### 整合案例

- UserDetails实现类

```
@Data
public class SecurityUserDetails implements UserDetails {

    // 用户
    private transient SysUser sysUser;

    //角色
    private transient List<SysRole> roleList;


    public SecurityUserDetails() {}

    public SecurityUserDetails(SysUser user) {
        if (ObjectUtils.isNotEmpty(user)) {
            this.sysUser = user;
        }
    }

    public SecurityUserDetails(SysUser user, List<SysRole> roleList) {
       if (ObjectUtils.isNotEmpty(user)) {
           this.sysUser = user;
           this.roleList = roleList;
       }
    }

    /**
     * 筛选出当前用户拥有的角色名称：例如admin、super...
     */
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<GrantedAuthority> authorities = Lists.newArrayList();
        Optional.ofNullable(this.roleList).ifPresent(roles -> {
            roles.forEach(role -> {
                authorities.add(new SimpleGrantedAuthority(role.getRoleName()));
            });
        });
        return authorities;
    }

    // 下面的获取用户、密码、账号是否过期、是否锁定、凭证是否过期、是否有用的这些凭证需要根据实际的需求来配置

    @Override
    public String getPassword() {
        return sysUser.getPassword();
    }

    @Override
    public String getUsername() {
        return sysUser.getUsername();
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


    /**
     * 账号是否启用
     */
    @Override
    public boolean isEnabled() {
        return !sysUser.getIsDeleted();
    }
}
```

-  UserDetailsService实现类

```
@Service("userDetailsService")
public class SecurityUserDetailsService implements UserDetailsService {

    @Resource
    private SysUserMapper sysUserMapper;

    @Resource
    private SysUserRoleMapper sysUserRoleMapper;

    @Resource
    private SysRoleMapper sysRoleMapper;

    /**
     * 根据用户名获取账号的信息
     *
     * @param username
     * @return
     */
    @Override
    public SecurityUserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        SysUser sysUser = sysUserMapper.selectOne(new QueryWrapper<SysUser>().eq("user_name", username).eq("is_deleted", false));
        Optional.ofNullable(sysUser).orElseThrow(() -> new BizException("用户名不存在"));
        return new SecurityUserDetails(sysUser, getRolesByUserId(sysUser.getId()));
    }


    /**
     * 根据userId获取用户角色权限信息
     *
     * @param userId
     * @return
     */
    private List<SysRole> getRolesByUserId(Long userId) {
        List<SysRole> roles = Lists.newArrayList();
        Optional.ofNullable(sysUserRoleMapper.selectList(new QueryWrapper<SysUserRole>().eq("user_id", userId).eq("is_deleted", false))).ifPresent(s -> {
            s.stream().filter(sysUserRole -> !sysUserRole.getIsDeleted()).map(SysUserRole::getRoleId).collect(Collectors.toList()).forEach(r -> {
                    roles.add(sysRoleMapper.selectById(r));
                });
            });
        return roles;
    }
}
```

- 忽略的白名单路径

```
secure:
  ignored:
    urls:
      - /sysUser/v1/login
      - /sysUser/v1/logout
      - /sysUser/v1/addUser


@Data
@Component
@ConfigurationProperties(prefix = "secure.ignored")
public class IgnoredUrlsConfig {

    private List<String> urls = Lists.newArrayList();

}
```

- JwtAuthenticationTokenFilter

```
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Value("${jwt.tokenHeader}")
    private String tokenHeader;

    @Value("${jwt.tokenHead}")
    private String tokenHead;

    @Resource
    private JwtTokenUtil jwtTokenUtil;

    @Resource
    private SecurityUserDetailsService securityUserDetailsService;

    @Resource
    private IgnoredUrlsConfig ignoredUrlsConfig;

    @Override
    protected void doFilterInternal(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
                                    FilterChain filterChain) throws ServletException, IOException {
        // 白名单中的URL直接放行
        PathMatcher pathMatcher = new AntPathMatcher();
        for (String url : ignoredUrlsConfig.getUrls()) {
            if (pathMatcher.match(url, httpServletRequest.getRequestURI())) {
                System.out.println("JwtAuthenticationTokenFilter 白名单通过..." + httpServletRequest.getRequestURI());
                filterChain.doFilter(httpServletRequest, httpServletResponse);
                return;
            }
        }

        String header = httpServletRequest.getHeader(tokenHeader);
        if (header != null && header.startsWith(tokenHead)) {
            String authToken = header.substring(tokenHead.length());
            String username = jwtTokenUtil.getUserNameFromToken(authToken);
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                SecurityUserDetails userDetails = securityUserDetailsService.loadUserByUsername(username);
                if (jwtTokenUtil.validateToken(authToken, userDetails)) {
                    UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                    authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(httpServletRequest));
                    SecurityContextHolder.getContext().setAuthentication(authenticationToken);
                    filterChain.doFilter(httpServletRequest, httpServletResponse);
                    return;
                }
            }
        }

        // token验证未通过, 直接返回错误
        httpServletResponse.setContentType("text/json;charset=utf-8");
        httpServletResponse.getWriter().write(JSON.toJSONString(ResultUtils.wrapException(ResponseCode.UNAUTHORIZED, "token验证失败")));
    }
```

-  SecurityAbstractSecurityInterceptor

```
@Component
public class SecurityAbstractSecurityInterceptor extends AbstractSecurityInterceptor implements Filter {


    @Resource
    private UrlFilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource;

    @Resource
    private IgnoredUrlsConfig ignoredUrlsConfig;

    @Resource
    public void setAccessDecisionManager(UrlAccessDecisionManager urlAccessDecisionManager) {
        super.setAccessDecisionManager(urlAccessDecisionManager);
    }


    @Override
    public Class<?> getSecureObjectClass() {
        return FilterInvocation.class;
    }

    @Override
    public SecurityMetadataSource obtainSecurityMetadataSource() {
        return this.urlFilterInvocationSecurityMetadataSource;
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        FilterInvocation fi = new FilterInvocation(servletRequest, servletResponse, filterChain);

        // OPTIONS请求直接放行
        if (request.getMethod().equals(HttpMethod.OPTIONS.toString())) {
            System.out.println("SecurityAbstractSecurityInterceptor OPTIONS请求放行");
            fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
        }

        // 白名单直接放行
        PathMatcher pathMatcher = new AntPathMatcher();
        for (String path : ignoredUrlsConfig.getUrls()) {
            if (pathMatcher.match(path, request.getRequestURI())) {
                System.out.println("SecurityAbstractSecurityInterceptor 白名单通过：" + request.getRequestURI());
                fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
                return;
            }
        }

        //此处会调用AccessDecisionManager中的decide方法进行鉴权操作
        InterceptorStatusToken token = super.beforeInvocation(fi);
        try {
            fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
        } finally {
            super.afterInvocation(token, null);
        }
    }
}

```

- UrlFilterInvocationSecurityMetadataSource

```
@Component
public class UrlFilterInvocationSecurityMetadataSource implements FilterInvocationSecurityMetadataSource {

    @Resource
    private SysMenuMapper sysMenuMapper;

    @Resource
    private SysRoleMapper sysRoleMapper;

    @Resource
    private SysRoleMenuMapper sysRoleMenuMapper;

    @Resource
    private IgnoredUrlsConfig ignoredUrlsConfig;

    /**
     * 用户访问的路径的权限配置信息role_login
     *
     * @param object
     * @return
     */
    @Override
    public Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException {
        String requestURL = ((FilterInvocation)(object)).getRequestUrl();

        // 白名单中的URL直接放行
        PathMatcher pathMatcher = new AntPathMatcher();
        for (String url : ignoredUrlsConfig.getUrls()) {
            if (pathMatcher.match(url, requestURL)) {
                System.out.println("UrlFilterInvocationSecurityMetadataSource 白名单通过..." + requestURL);
                return null;
            }
        }

        // 找出匹配的URL对应的角色Role_name，放到指定变量中
        List<SysMenu> menuList = sysMenuMapper.selectList(new QueryWrapper<SysMenu>().eq("is_deleted", false));
        for (SysMenu m : menuList) {
            if (requestURL.equals(m.getUrl())) {
                List<String> roleNames = Lists.newArrayList();
                Optional.ofNullable(sysRoleMenuMapper.selectList(new QueryWrapper<SysRoleMenu>().eq("is_deleted", false))).ifPresent(srm -> {
                    srm.forEach(s -> {
                        Optional.ofNullable(sysRoleMapper.selectOne(new QueryWrapper<SysRole>().eq("is_deleted", false).eq("id", s.getRoleId()))).ifPresent(sysRole -> {
                            roleNames.add(sysRole.getRoleName());
                        });
                    });
                });
                return SecurityConfig.createList(roleNames.toArray(new String[roleNames.size()]));
            }
        }

        // 路径没有对应哪一个角色才能访问ROLE_LOGIN
        return SecurityConfig.createList("NEED_LOGIN");
    }


    /**
     * 全部已配置的权限的路径
     *
     * @return
     */
    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return null;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return FilterInvocation.class.isAssignableFrom(aClass);

    }
}
```

- UrlAccessDecisionManager

```
@Component
public class UrlAccessDecisionManager implements AccessDecisionManager {

    @Override
    public void decide(Authentication authentication, Object o, Collection<ConfigAttribute> collection) throws AccessDeniedException, InsufficientAuthenticationException {
        if (CollectionUtils.isEmpty(collection)) {
            return;
        }

        if (CollectionUtils.isEmpty(authentication.getAuthorities())) {
            throw new AccessDeniedException("该账号没有任何权限");
        }

        System.out.println("UrlAccessDecisionManager 用户拥有的路径访问权限:" + authentication.getAuthorities().toString());

        for (ConfigAttribute configAttribute : collection) {
            String needRole = configAttribute.getAttribute();
            System.out.println("UrlAccessDecisionManager 用户访问需要的角色权限：" + needRole);

            if ("ROLE_LOGIN".equals(needRole)) {
                if (authentication instanceof AnonymousAuthenticationToken) {
                    throw new BadCredentialsException("用户未登录！");
                } else {
                    return;
                    // throw new AccessDeniedException("用户没有权限访问该URL");
                }
            }

            // 当前用户拥有的角色
            Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
            for (GrantedAuthority authority : authorities) {
                if (authority.getAuthority().equals(needRole)) {
                    return;
                }
            }
        }
        System.out.println("UrlAccessDecisionManager 权限验证失败");
        throw new AccessDeniedException("权限不足，请联系管理员！");
    }

    @Override
    public boolean supports(ConfigAttribute configAttribute) {
        return true;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return true;
    }
}
```

- UrlAccessDeniedHandler

```
@Component
public class UrlAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException e) throws IOException, ServletException {
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        response.setContentType("text/json;charset=utf-8");
        response.getWriter().write(JSON.toJSONString(ResultUtils.wrapException(ResponseCode.FORBIDDEN)));
    }
}
```

- JwtAuthorizationEntryPoint

```
@Component
public class JwtAuthorizationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException e) throws IOException, ServletException {
        response.setContentType("text/json;charset=utf-8");
        response.getWriter().write(JSON.toJSONString(ResultUtils.wrapException(ResponseCode.UNAUTHORIZED)));
    }
}
```

- WebSecurityConfig

```
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {


    /**
     * 可绕过的白名单请求url
     */
    @Resource
    private IgnoredUrlsConfig ignoredUrlsConfig;

    /**
     * 自定义未登录或登录过期返回结果
     */
    @Resource
    private JwtAuthorizationEntryPoint jwtAuthorizationEntryPoint;

    /**
     * 自定义的JWT安全过滤器
     */
    @Resource
    private JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;


    /**
     * 权限拒绝处理逻辑
     */
    @Resource
    private UrlAccessDeniedHandler urlAccessDeniedHandler;

    /**
     * 安全拦截器
     */
    @Resource
    private SecurityAbstractSecurityInterceptor securityAbstractSecurityInterceptor;

    @Resource
    private UrlFilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource;

    @Resource
    private UrlAccessDecisionManager urlAccessDecisionManager;



    /**
     * 配置自定义实现的UserDetailsService对象
     */
    @Resource
    private SecurityUserDetailsService securityUserDetailsService;



    /**
     * 配置PasswordEncoder对象
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }



    public WebSecurityConfig(JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter,
              JwtAuthorizationEntryPoint jwtAuthorizationEntryPoint,
              SecurityAbstractSecurityInterceptor securityAbstractSecurityInterceptor,
              UrlFilterInvocationSecurityMetadataSource urlFilterInvocationSecurityMetadataSource,
              UrlAccessDecisionManager urlAccessDecisionManager,
              UrlAccessDeniedHandler urlAccessDeniedHandler
              ) {
            this.jwtAuthenticationTokenFilter = jwtAuthenticationTokenFilter;
            this.jwtAuthorizationEntryPoint = jwtAuthorizationEntryPoint;
            this.securityAbstractSecurityInterceptor = securityAbstractSecurityInterceptor;
            this.urlFilterInvocationSecurityMetadataSource = urlFilterInvocationSecurityMetadataSource;
            this.urlAccessDeniedHandler = urlAccessDeniedHandler;
            this.urlAccessDecisionManager = urlAccessDecisionManager;
    }

    /**
     *配置自定义的UserDetailsService和passwordEncoder实现
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService()).passwordEncoder(passwordEncoder());
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers(HttpMethod.GET,
            "/favicon.ico",
            "/*.html",
            "/**/*.css",
            "/**/*.js");
    }

    /**
     * 各种过滤器和拦截器的核心配置
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors().and().csrf().disable();// 禁用CSRF开启跨域
        ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry registry = http.authorizeRequests();

        // URL权限认证
        registry.withObjectPostProcessor(new ObjectPostProcessor<FilterSecurityInterceptor>() {

            @Override
            public <O extends FilterSecurityInterceptor> O postProcess(O o) {
                o.setSecurityMetadataSource(urlFilterInvocationSecurityMetadataSource);
                o.setAccessDecisionManager(urlAccessDecisionManager);
                return o;
            }
        });

        // 放行白名单URL
        ignoredUrlsConfig.getUrls().forEach(url -> {
            System.out.println("白名单放行URL:" + url);
            registry.antMatchers(url).permitAll();
        });

        // 允许跨域请求的OPTIONS请求
        registry.antMatchers(HttpMethod.OPTIONS).permitAll();

        registry.and()
                .authorizeRequests()
                .anyRequest()
                .authenticated()

                //关闭跨站请求防护及不适用session
                .and()
                .csrf()
                .disable()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)

                // 自定义权限拒绝处理类
                .and()
                .exceptionHandling()
                .accessDeniedHandler(urlAccessDeniedHandler)    //授权
                .authenticationEntryPoint(jwtAuthorizationEntryPoint) //认证

                // 自定义权限拦截JWT过滤器
                .and()
                .addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);

        // 动态权限校验过滤器
         registry.and().addFilterBefore(securityAbstractSecurityInterceptor, FilterSecurityInterceptor.class);

    }
}
```


### 小结

整体而言，Spring Security的配置是比较繁琐的，但是熟练掌握之后发现其实也很简单的，最终的思想无法就是整合判断是否具有访问URL的权限及资源的权限


### 参考资料

1、https://www.1024sou.com/article/513666.html

2、https://www.cnblogs.com/zhengqing/p/11704229.html

3、https://blog.51cto.com/u_15082397/2590650

4、https://juejin.cn/post/7023183872422576136

5、https://www.bbsmax.com/A/WpdK3ywXdV/
