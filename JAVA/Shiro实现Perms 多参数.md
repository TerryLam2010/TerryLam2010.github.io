Shiro实现Perms 多参数

重新实现一个拦截器

```
public class PermsAuthorizationFilter extends AuthorizationFilter {
  
    @Override  
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue)
            throws Exception {  
        Subject subject = getSubject(request, response);
        String[] permsArray = (String[]) mappedValue;
  
        if (permsArray == null || permsArray.length == 0) {
            return true;
        }   
  
        for(int i=0;i<permsArray.length;i++){
            if(subject.isPermitted(permsArray[i])){
                return true;    
            }    
        }    
        return false;    
    }  
  
}  
```

```
@Configuration
@Import(ShiroManager.class)
public class ShiroConfig {

	@Autowired
	ResourceDao resourceDao;

	@Autowired
	AdminProperties adminProperties;

	@Bean(name = "realm")
	@DependsOn("lifecycleBeanPostProcessor")
	@ConditionalOnMissingBean
	public Realm realm() {
		return new MyRealm();
	}

	/**
	 * 用户授权信息Cache
	 */
	@Bean(name = "shiroCacheManager")
	@ConditionalOnMissingBean
	public CacheManager cacheManager() {
		return new MemoryConstrainedCacheManager();
	}

	@Bean(name = "securityManager")
	@ConditionalOnMissingBean
	public DefaultSecurityManager securityManager() {
		DefaultSecurityManager sm = new DefaultWebSecurityManager();
		sm.setCacheManager(cacheManager());
		return sm;
	}

	@Bean(name = "shiroFilter")
	@DependsOn("securityManager")
	@ConditionalOnMissingBean
	public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultSecurityManager securityManager, Realm realm, Environment environment) {

		securityManager.setRealm(realm);
		ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
		shiroFilter.setSecurityManager(securityManager);
		shiroFilter.setLoginUrl(adminProperties.getLoginUrl());
		shiroFilter.setSuccessUrl("/admin/index");
		shiroFilter.setUnauthorizedUrl(adminProperties.getDomain()+environment.getProperty(ConfigKeyConstant.SERVER_SERVLET_CONTEXT_PATH)+"/previlige/no");
		//shiroFilter.setUnauthorizedUrl("/previlige/no");
		Map<String, Filter> filters = new LinkedHashMap<String, Filter>();
		EnvFilter envFilter = new MyEnvFilter();
		filters.put("envFilter", envFilter);
		filters.put("anyPerms", new PermsAuthorizationFilter());
		//filters.put("corsFilter", new MyHttpAuthenticationFilter());

		shiroFilter.setFilters(filters);
		setUrlFilter(shiroFilter);
		return shiroFilter;
	}

	private void setUrlFilter(ShiroFilterFactoryBean shiroFilterFactoryBean) {
		List<Resource> resources = resourceDao.findAll();
		Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
		filterChainDefinitionMap.put("/admin/login", "anon");
		filterChainDefinitionMap.put("/admin/session", "anon");
		filterChainDefinitionMap.put("/static/**", "anon");

		
		//权限范围过滤器从小写到大
		Map<String,String> urls = new LinkedHashMap<>();
		for (Resource resource : resources) {

			if (StringUtils.isNotBlank(resource.getSourceUrl()) && StringUtils.isNotBlank(resource.getSourceKey())) {
				if(urls.containsKey(resource.getSourceUrl())){
					String value = urls.get(resource.getSourceUrl()) + "," + resource.getSourceKey();
					urls.put(resource.getSourceUrl(),value);
				}else{
					urls.put(resource.getSourceUrl(),resource.getSourceKey());
				}
			}
		}
		urls.forEach((key,value)->{
			filterChainDefinitionMap.put(key, "anyPerms[" + value + "],envFilter");
		});

        filterChainDefinitionMap.put("/admin/**", "authc,envFilter");
		shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);

	}

}
```

