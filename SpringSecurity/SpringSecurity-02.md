# 一、通过debug源码，解析，更加清楚的了解核心的原理，出现问题可以更好的解决
# 二、密码验证流程
##  1 默认使用 UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter
-     执行方法：
     ```
     	public Authentication attemptAuthentication(HttpServletRequest request,
     			HttpServletResponse response) throws AuthenticationException {
     		if (postOnly && !request.getMethod().equals("POST")) {
     			throw new AuthenticationServiceException(
     					"Authentication method not supported: " + request.getMethod());
     		}
     		String username = obtainUsername(request);
     		String password = obtainPassword(request);
     		if (username == null) {
     			username = "";
     		}
     		if (password == null) {
     			password = "";
     		}
     		username = username.trim();
     		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
     				username, password);
     		// Allow subclasses to set the "details" property
     		setDetails(request, authRequest); (这里最终是AbstractAuthenticationToken)
     		return this.getAuthenticationManager().authenticate(authRequest);
     	} 
     ```
##   2 return this.getAuthenticationManager().authenticate(authRequest); 执行这个方法后，跳入到 AuthenticationManager 这个接口，以其一个实现类来实现这个方法。
     ```
        Authentication authenticate(Authentication authentication) throws AuthenticationException;
     ```
##   3 ProviderManager implements AuthenticationManager, MessageSourceAware,InitializingBean 
     ```
         public Authentication authenticate(Authentication authentication)
                    throws AuthenticationException {
                Class<? extends Authentication> toTest = authentication.getClass();
                AuthenticationException lastException = null;
                Authentication result = null;
                boolean debug = logger.isDebugEnabled(); 
                --AuthenticationProvider 校验的是这个对象
                for (AuthenticationProvider provider : getProviders()) {
                    if (!provider.supports(toTest)) {
                        continue;
                    }
                    if (debug) {
                        logger.debug("Authentication attempt using "
                                + provider.getClass().getName());
                    }
                    try {
                         -- 重点,这里调用了验证方法 ----  
                        result = provider.authenticate(authentication);         
                        if (result != null) {
                            copyDetails(authentication, result);
                            break;
                        }
                    }
                    catch (AccountStatusException e) {
                        prepareException(e, authentication);
                        // SEC-546: Avoid polling additional providers if auth failure is due to
                        // invalid account status
                        throw e;
                    }
                    catch (InternalAuthenticationServiceException e) {
                        prepareException(e, authentication);
                        throw e;
                    }
                    catch (AuthenticationException e) {
                        lastException = e;
                    }
                }
                if (result == null && parent != null) {
                    // Allow the parent to try.
                    try {
                        result = parent.authenticate(authentication);
                    }
                    catch (ProviderNotFoundException e) {
                        // ignore as we will throw below if no other exception occurred prior to
                        // calling parent and the parent
                        // may throw ProviderNotFound even though a provider in the child already
                        // handled the request
                    }
                    catch (AuthenticationException e) {
                        lastException = e;
                    }
                }
                if (result != null) {
                    if (eraseCredentialsAfterAuthentication
                            && (result instanceof CredentialsContainer)) {
                        // Authentication is complete. Remove credentials and other secret data
                        // from authentication
                        ((CredentialsContainer) result).eraseCredentials();
                    }
                    eventPublisher.publishAuthenticationSuccess(result);
                    return result;
                }
                // Parent was null, or didn't authenticate (or throw an exception)
                if (lastException == null) {
                    lastException = new ProviderNotFoundException(messages.getMessage(
                            "ProviderManager.providerNotFound",
                            new Object[] { toTest.getName() },
                            "No AuthenticationProvider found for {0}"));
                }
                prepareException(lastException, authentication);
                throw lastException;
         }
     ```
##   4  进入 public interface AuthenticationProvider
      ```
      	Authentication authenticate(Authentication authentication) throws AuthenticationException;
      ```
##   5 进入  abstract class AbstractUserDetailsAuthenticationProvider implements AuthenticationProvider, InitializingBean, MessageSourceAware,实现了AuthenticationProvider 接口的authenticate(authentication)方法
     ```
     public Authentication authenticate(Authentication authentication)
     			throws AuthenticationException {
     		Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,
     				messages.getMessage(
     						"AbstractUserDetailsAuthenticationProvider.onlySupports",
     						"Only UsernamePasswordAuthenticationToken is supported"));
     		// Determine username
     		String username = (authentication.getPrincipal() == null) ? "NONE_PROVIDED": authentication.getName();
     		boolean cacheWasUsed = true;
     		UserDetails user = this.userCache.getUserFromCache(username);
     		if (user == null) {
     			cacheWasUsed = false;
     			try {
     				user = retrieveUser(username,
     						(UsernamePasswordAuthenticationToken) authentication);
     			}
     			catch (UsernameNotFoundException notFound) {
     				logger.debug("User '" + username + "' not found");
     				if (hideUserNotFoundExceptions) {
     					throw new BadCredentialsException(messages.getMessage(
     							"AbstractUserDetailsAuthenticationProvider.badCredentials",
     							"Bad credentials"));
     				}
     				else {
     					throw notFound;
     				}
     			}
     			Assert.notNull(user,
     					"retrieveUser returned null - a violation of the interface contract");
     		}
     		try {
     			preAuthenticationChecks.check(user);
     			additionalAuthenticationChecks(user,
     					(UsernamePasswordAuthenticationToken) authentication);
     		}
     		catch (AuthenticationException exception) {
     			if (cacheWasUsed) {
     				// There was a problem, so try again after checking
     				// we're using latest data (i.e. not from the cache)
     				cacheWasUsed = false;
     				user = retrieveUser(username,
     						(UsernamePasswordAuthenticationToken) authentication);
     				preAuthenticationChecks.check(user);
     				additionalAuthenticationChecks(user,
     						(UsernamePasswordAuthenticationToken) authentication);
     			}
     			else {
     				throw exception;
     			}
     		}
     		postAuthenticationChecks.check(user);
     		if (!cacheWasUsed) {
     			this.userCache.putUserInCache(user);
     		}
     		Object principalToReturn = user;
     		if (forcePrincipalAsString) {
     			principalToReturn = user.getUsername();
     		}
     		return createSuccessAuthentication(principalToReturn, authentication, user);
     	}
     ```
-    在方法中调用自身的一个抽象方法；
      ```
         protected abstract void additionalAuthenticationChecks(UserDetails userDetails,UsernamePasswordAuthenticationToken authentication)
                    throws AuthenticationException;
      ```
##   6 通过实现该抽象类的子类 class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider,重写 additionalAuthenticationChecks方法
     ```
     protected void additionalAuthenticationChecks(UserDetails userDetails,
     			UsernamePasswordAuthenticationToken authentication)
     			throws AuthenticationException {
     		Object salt = null;
     		if (this.saltSource != null) {
     			salt = this.saltSource.getSalt(userDetails);
     		}
     		if (authentication.getCredentials() == null) {
     			logger.debug("Authentication failed: no credentials provided");
     			throw new BadCredentialsException(messages.getMessage(
     					"AbstractUserDetailsAuthenticationProvider.badCredentials",
     					"Bad credentials"));
     		}
     		String presentedPassword = authentication.getCredentials().toString();
     		//验证密码， passwordEncoder 是客户端穿回来的密码，userDetails.getPassword()是数据库中的密码
     		if (!passwordEncoder.isPasswordValid(userDetails.getPassword(),presentedPassword, salt)) {
     			logger.debug("Authentication failed: password does not match stored value");
     			throw new BadCredentialsException(messages.getMessage(
     					"AbstractUserDetailsAuthenticationProvider.badCredentials",
     					"Bad credentials"));
     		}
     	}
     ```
# 三、其他有关SpringSecurity的笔记，包括核心认识，demo地址
 [SpringSecurity初步学习](https://yq.aliyun.com/articles/721855?spm=a2c4e.11155435.0.0.68d63312mfmg2w)