# spring-拦截器做并发控制

### 一、spring-并发控制相关代码

#### 1、控制器代码

```
package com.test.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("test")
public class TestController {

    @RequestMapping("a")
    public String test(
            @RequestParam("user") String user
    ) throws InterruptedException {
        System.out.println(user);
        return user;
    }
}
```

#### 2、拦截器配置

```
package com.test.config;

import com.test.interceptor.ConcurrentCheckInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class InterceptorConfig extends WebMvcConfigurerAdapter {

    @Autowired
    ConcurrentCheckInterceptor concurrentCheckInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //注册并发控制拦截器
        registry.addInterceptor(concurrentCheckInterceptor)
                .addPathPatterns("/**");

        super.addInterceptors(registry);
    }
}
```

#### 3、拦截器代码

```
package com.test.interceptor;

import com.google.common.util.concurrent.AtomicLongMap;
import lombok.extern.slf4j.Slf4j;
import org.springframework.lang.Nullable;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.concurrent.atomic.AtomicInteger;

@Slf4j
@Component
public class ConcurrentCheckInterceptor implements HandlerInterceptor {
    AtomicLongMap<Integer> atomicLongMap = AtomicLongMap.create();
    public static AtomicInteger total = new AtomicInteger(0);
    public static AtomicInteger finish = new AtomicInteger(0);
    public static AtomicInteger refuse = new AtomicInteger(0);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        //这个地方验证并发，记录并发数
        String user = request.getParameter("user");
        if (user == null) {
            throw new RuntimeException();
        }
        total.incrementAndGet();
        int ecId = Integer.parseInt(user.replaceAll("^(0+)", ""));
        long count = atomicLongMap.incrementAndGet(ecId);
        //超出并发数限制，返回响应，同时计数减1
        log.info("当前调用次数：" + count);
        if (count > 100) {
            refuse.incrementAndGet();
            log.info("the user[" + 1 + "] is over concurrent limit:" + 100 + " counter:" + count);
            long a = atomicLongMap.decrementAndGet(ecId);
            handleFalseResponse(response);
            return false;
        }

        return true;
    }

    //异常也能够走到这个地方
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
        finish.incrementAndGet();
        String user = request.getParameter("user");
        if (user != null) {
            int ecId = Integer.parseInt(user.replaceAll("^(0+)", ""));
            long a = atomicLongMap.decrementAndGet(ecId);
            log.info("结束调用次数：" + a);
            log.info("当前total:" + total.get() + ";当前finish:" + finish.get() + ";当前refuse:" + refuse.get());
        }
    }

    private void handleFalseResponse(HttpServletResponse response)
            throws Exception {
        response.setStatus(200);
        response.setContentType("application/json");
        response.setCharacterEncoding("UTF-8");

        response.getWriter().write("{\"code\":\"1\",\"data\":\"null\",\"msg\":\"并发超限\"}");
        response.getWriter().flush();
    }
}
```

### 二、使用jmeter进行压测

**配置参数如下：** 线程数300，循环次数2次，设置http请求和参数，执行

![jmeter压测参数配置](<../.gitbook/assets/image (43).png>)

**查看控制台打印数据：**&#x20;

并发限流日志&#x20;

![被限流打印日志](<../.gitbook/assets/image (16).png>)

最终结果日志&#x20;

![执行完毕最终记录的数据](<../.gitbook/assets/image (26).png>)

我们发现，数据是对的上的，同时有请求确实被并发限流了
