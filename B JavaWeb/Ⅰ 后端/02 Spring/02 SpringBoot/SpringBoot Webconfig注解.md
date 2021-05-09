WebMvcConfigurer

- addInterceptors：拦截器
- addViewControllers：页面跳转
- addResourceHandlers：静态资源
- configureDefaultServletHandling：默认静态资源处理器
- configureViewResolvers：视图解析器
- configureContentNegotiation：配置内容裁决的一些参数
- addCorsMappings：跨域
- configureMessageConverters：信息转换器

```java
package com.yeyangshu.controller;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.*;

import java.util.List;

/**
 * @author yeyangshu
 * @version 1.0
 * @date 2021/4/30 22:37
 */
@Configuration
public class MyWebMvcConfigurer implements WebMvcConfigurer {

    /**
     * 拦截器
     * 用于用户登录状态的拦截，日志的拦截等。
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry
                // 实现HandlerInterceptor接口的拦截器实例
                .addInterceptor(myInterceptor())
                // 用于设置拦截器的过滤路径规则
                .addPathPatterns("/**")
                // 用于设置不需要拦截的过滤规则
                .excludePathPatterns("/js/**", "/css/**", "/image/**", "/index/**");
    }

    @Bean
    public MyInterceptor myInterceptor() {
        return new MyInterceptor();
    }

    /**
     * 视图跳转控制器
     * http://localhost:8080/toLogin跳转到http://localhost:8080/login.html
     */
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("toLogin")
                .setViewName("login");
    }

    /**
     * 自定义静态资源映射目录
     * 静态资源
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry
                // 对外暴露的访问路径
                .addResourceHandler("/my/file/**")
                // 内部文件放置的目录
                .addResourceLocations("classpath:/my/file/");
    }

    /**
     * 默认静态资源处理器
     */
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    /**
     * 视图解析器
     */
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {

    }

    /**
     * 内容裁决
     */
    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {

    }

    /**
     * 跨域
     */
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry
                .addMapping("/cors")
                .allowedHeaders("*")
                .allowedMethods("GET", "POST")
                .allowedOrigins("*")
                .maxAge(3600 * 24);
    }

    /**
     * 消息转换器
     */
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {

    }

}

```

MyInterceptor

```java

package com.yeyangshu.controller;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 拦截器实例
 *
 * @author yeyangshu
 * @version 1.0
 * @date 2021/4/30 23:03
 */
public class MyInterceptor implements HandlerInterceptor {

    /**
     * 在业务处理器处理请求之前被调用。
     * 预处理，可以进行编码、安全控制、权限校验等处理
     * true：继续流程
     * false：流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 权限校验
        return false;
    }

    /**
     * 在业务处理器处理请求执行完成后，生成视图之前执行。后处理（但未进行页面渲染），有机会修改ModelAndView
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    /**
     * 在DispatcherServlet完全处理完请求后被调用，可用于清理资源等。返回处理（已经渲染了页面）；
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }


}

```

