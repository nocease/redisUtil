# redisUtil
redis工具类，在spring中使用。

如果要在拦截器中使用：
示例：
1.拦截器配置：

package cn.nocease.filter;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class InterceptorConfig extends WebMvcConfigurerAdapter {

    @Bean
    public SessionInterceptor sessionInterceptor() {
        return new SessionInterceptor();
    }

    @Bean
    public AdminInterceptor adminInterceptor() {
        return new AdminInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //拦截器链
        /**
         * addPathPatterns 用于添加拦截规则
         * excludePathPatterns 用户排除拦截
         */

        InterceptorRegistration SessionInterceptor = registry.addInterceptor(sessionInterceptor());
        InterceptorRegistration adminInterceptor = registry.addInterceptor(adminInterceptor());

        adminInterceptor.addPathPatterns("/**");
        //不拦截登录
        adminInterceptor.excludePathPatterns("/logining/login");
        adminInterceptor.excludePathPatterns("/logining/toLogin");
        //不拦截前台
        adminInterceptor.excludePathPatterns("/QnewsController/*");
        adminInterceptor.excludePathPatterns("/index.html");

        SessionInterceptor.addPathPatterns("/**");

        super.addInterceptors(registry);
    }

}

2.一个拦截器
package cn.nocease.filter;

import cn.nocease.util.CookieUtil;
import cn.nocease.util.RedisUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class AdminInterceptor implements HandlerInterceptor {
    @Autowired
    private RedisUtil redisUtil;

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object object,
                                Exception exception) throws Exception {
        // System.out.println("3.整个请求结束之后被调用......CustomInterceptor1......");
    }


    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object object, ModelAndView view)
            throws Exception {
        // System.out.println("2. 请求处理之后进行调用，但是在视图被渲染之前......CustomInterceptor1......");
    }


    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object object) throws Exception {
        CookieUtil cu = new CookieUtil(request, response);
        if (redisUtil.get(cu.getCookie("RedisSessionId")) == null || redisUtil.get(cu.getCookie("RedisSessionId")).equals("\"\"") || redisUtil.get(cu.getCookie("RedisSessionId")).equals("")) {
            response.sendRedirect(request.getContextPath() + "/logining/login");
            return false;
        } else {
            return true;
        }
    }



