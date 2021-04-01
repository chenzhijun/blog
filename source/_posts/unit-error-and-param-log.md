---
title: 参数-异常统一打印
date: 2017-11-02 11:39:39
tags:
	- Java
categories: Java
---

> 先说明我们业务开发基础框架使用的是 SpringBoot 

## 简述

在项目开发中总是需要知道一些常用信息打印，比如出现异常了你可能需要打印日志，为了便于分析，你可能也需要打印埋点数据，或者请求参数之类的。

这类操作可能在任何地方都有，如果分别取处理，感觉上不是特别合适。需要写大量代码并且维护量大。我的做法就是使用`@Aspect`,`@ControllerAdvice`利用切面来做统一的参数和异常处理。
<!--more-->
## 处理异常

使用`ControllerAdvice`注解来监听全局异常。这样的好处是，所有的出现异常的点我都是直接往上抛，而不用在每个业务里面去处理。在里面我处理了是否为我自己业务逻辑抛出的异常，如果是的话根据错误码返回响应的错误提示。如果不是业务异常，那么我就转换成系统异常，最后给用户的也是“系统出现异常”等提示语，而不会出现异常堆栈代码。

```java

@ControllerAdvice
public class ExceptionHandler {

    private Logger logger = LoggerFactory.getLogger(ExceptionHandler.class);

    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public ResultData exceptionHandler(Exception e) {
        logger.error("系统异常：{}", e);

        ResultData ret = new ResultData<>();
        ret.setData(null);
        ret.setSucceed(false);
        if (e instanceof BusinessException) {
            BusinessException serverException = (BusinessException) e;
            ret.setErrorCode(serverException.getErrorCode());
            ret.setErrorMsg(SpringUtil.getMessage(serverException.getErrorCode(), serverException.getMessage()));
        } else {
            ret.setErrorCode("100002");
            ret.setErrorMsg(SpringUtil.getMessage("100002"));
        }
        return ret;
    }

```

## 处理参数

利用`@Aspect`，我将请求的参数，和响应的返回值都做了相应的日志打印。这种方式可能并不是特别好，因为可能响应的信息很多，那么可能系统很快就会磁盘爆满，但是在前期我想还是很有必要的，后期我们可能根据日志的级别来做响应的控制。

下面的代码主要是监控在 controller 层的参数和响应：

```java
/**
 * @author chen
 * @version V1.0
 * @date 2017/10/27
 */
@Aspect
@Component
public class ParamLogAspect {

    private Logger logger = LoggerFactory.getLogger(ParamLogAspect.class);

    ThreadLocal<Long> startTime = new ThreadLocal<>();

    @Before(value = "execution(public * com.web..*.*(..))")
    public void doBefore(JoinPoint joinPoint) {
        startTime.set(System.currentTimeMillis());
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        logger.info("URL : {}", request.getRequestURL().toString());
        logger.info("HTTP_METHOD : {} ", request.getMethod());
        logger.info("IP : {} ", IPUtil.getIpAddress(request));
        logger.info("CLASS_METHOD : {}.{}", joinPoint.getSignature().getDeclaringTypeName(), joinPoint.getSignature().getName());
        logger.info("REQUEST_PARAM : {}", Arrays.toString(joinPoint.getArgs()));

    }

    @AfterReturning(returning = "resp", value = "execution(public * com.web..*.*(..))")
    public void doAfterReturning(Object resp) throws Exception {
        logger.info("RESPONSE : {}", JSON.toJSONString(resp));
        logger.info("SPEND TIME : {}", (System.currentTimeMillis() - startTime.get()));
    }
}

```

## 总结

总之，方式有很多，我也见过通过实现继承的接口方式来做异常处理的，不管哪种我们的目的都是一个，尽量不给用户不好的提示信息，毕竟我们的目标是：**代码帅，运行快**