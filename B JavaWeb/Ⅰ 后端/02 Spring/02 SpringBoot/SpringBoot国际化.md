# 国际化

## 1 LocaleResolver



![image-20210506204131618](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210506204131618.png)

默认四个实现类：

- AcceptHeaderLocaleResolver
- FixedLocaleResolver
- CookieLocaleResolver
- SessionLocaleResolver

AcceptHeaderLocaleResolver

```java
/**
 * LocaleResolver实现的抽象基类。 提供对默认语言环境的支持
 */
public abstract class AbstractLocaleResolver implements LocaleResolver {

	@Nullable
	private Locale defaultLocale;


	/**
	 * 设置默认的语言环境，如果未找到其他语言环境，此解析器将返回该语言环境。
	 */
	public void setDefaultLocale(@Nullable Locale defaultLocale) {
		this.defaultLocale = defaultLocale;
	}

	/**
	 * 返回此解析器应使用的默认语言环境（如果有）。
	 */
	@Nullable
	protected Locale getDefaultLocale() {
		return this.defaultLocale;
	}

}
```







## 2 样例

WebConfig

```java
package com.yeyangshu.springboot;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.i18n.LocaleChangeInterceptor;
import org.springframework.web.servlet.i18n.SessionLocaleResolver;

/**
 * @author yeyangshu
 * @version 1.0
 * @date 2021/5/7 0:35
 */
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(localeChangeInterceptor());
    }

    @Bean
    public LocaleResolver localeResolver() {
//        SessionLocaleResolver sessionLocaleResolver = new SessionLocaleResolver();
//        sessionLocaleResolver.setDefaultLocale(Locale.US);
//        return sessionLocaleResolver;
        return new MessageLocaleResolver();
    }

    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor localeChangeInterceptor = new LocaleChangeInterceptor();
        localeChangeInterceptor.setParamName("lang");
        return localeChangeInterceptor;
    }

}
```

MessageLocaleResolver

```java
package com.yeyangshu.springboot;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.LocaleResolver;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

/**
 * @author yeyangshu
 * @version 1.0
 * @date 2021/5/7 0:34
 */
@Slf4j
public class MessageLocaleResolver implements LocaleResolver {
    @Override
    public Locale resolveLocale(HttpServletRequest httpServletRequest) {
        log.info(httpServletRequest.getParameter("lang"));
        return new Locale("zh", "CN");
    }

    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {

    }
}
```

TestController

```java
package com.yeyangshu.springboot;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.context.i18n.LocaleContextHolder;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;

/**
 * @author yeyangshu
 * @version 1.0
 * @date 2021/5/7 0:28
 */
@Slf4j
@RestController
public class TestController {

    @Autowired
    MessageSource messageSource;

    @PostMapping("/test")
    public String test(HttpServletRequest request) {
        log.info(request.getCookies().toString());
        return messageSource.getMessage("user", null, LocaleContextHolder.getLocale());
    }

}
```

配置文件

![image-20210507005222324](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20210507005222324.png)