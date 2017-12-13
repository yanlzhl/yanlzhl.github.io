---
layout: post
date: 2017-10-10 11:00
title: Spring 接口工具类
description: 该工具类目前用在两个地方，一是登录用户信息绑定在ThreadLocal线程，二是Spring容器与其他容器不兼容时，获取系统Bean实体或配置属性。通过这个工具类，让我更加钦佩Spring Framework的设计哲学，需踏踏实实地阅读源码。
categories: [Java,pring]
tags: [Java,Spring]
---

> 我将此工具类命名AwareUtil.因Spring提供诸多*Aware的接口，当自定义类实现这些接口后，便可方便从上下文中获取当前的运行环境。如下：

> * org.springframework.context.ApplicationContextAware接口
Spring框架启动时，ApplicationContext初始化实现了该接口的Spring Bean时，会将ApplicationContext的引用作为参数传递给创建的Bean实例，创建的Bean实例可以通过ApplicationContext的引用操作Spring框架的各种资源。
作用与@Autowired标注类似。
> * LoadTimeWeaverAware，加载Spring Bean时织入第三方模块，如AspectJ
> * BeanClassLoaderAware，加载Spring Bean的类加载器
> * BootstrapContextAware，资源适配器BootstrapContext，如JCA,CCI
> * ResourceLoaderAware，底层访问资源的加载器
> * BeanFactoryAware，声明BeanFactory
> * PortletConfigAware，PortletConfig
> * PortletContextAware，PortletContext
> * ServletConfigAware，ServletConfig
> * ServletContextAware，ServletContext
> * MessageSourceAware，国际化
> * ApplicationEventPublisherAware，应用事件
> * NotificationPublisherAware，JMX通知
> * BeanNameAware，声明Spring Bean的名字
> * EnvironmentAware，环境变量读取 

# AwareUtil代码
```java

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.EnvironmentAware;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

/**
 * 获取spring的ApplicationContext。通过ApplicationContext可以获取在spring配置文件中配置的类。
 * 获取spring的environment。通过environment可以获取在spring配置文件中属性值。
 * @author yanlz
 * @data 2017/9/25.
 */
@Component
public class AwareUtil implements ApplicationContextAware,EnvironmentAware {

    private static ApplicationContext applicationContext;

    private static Environment environment;

    /**
     * 获取spring的ApplicationContext。
     *
     * @return Spring的ApplicationContext.
     */
    public static ApplicationContext getContext() {
        return applicationContext;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        AwareUtil.applicationContext = applicationContext;
    }

    /**
     * 获取类型为requiredType的对象
     *
     * @param requiredType
     * @return
     */
    public static <T> T getBean(Class<T> requiredType) {
        return applicationContext.getBean(requiredType);
    }

    @Override
    public void setEnvironment(Environment environment) {
        AwareUtil.environment =environment;
    }

    public static Environment getEnvironment() {
        return environment;
    }
}

```
# ThreadLocal获取登录用户信息
```java
 /**
     * 获取当前登录用户信息
     *
     * @return
     */
    @GetMapping(value = "/cur_user")
    public ResponseModel getInfo() {
        LoginEntity logined = AwareUtil.get();
        ......    
    }
```

# 获取环境变量&对象实例
此段代码环境为Spring Boot，因此具体读取配置文件因具体情况而定。
```
/**
 * quartz执行类
 * @author yanlz
 * @data 2017/9/25.
 */
//@Configuration
//@PropertySource("classpath:application.properties")
public class MigrationJob implements Job {
    private final Logger logger = LoggerFactory.getLogger(getClass());

    private String migrationDirectory = AwareUtil.getEnvironment().getProperty("migration.jar.directory");
    private String migrationScriptName = AwareUtil.getEnvironment().getProperty("migration.script.name");

    private MigrationDao migrationDao = AwareUtil.getBean(MigrationDao.class);
    private MigrationService migrationService = AwareUtil.getBean(MigrationService.class);
    private SimpMessageSendingOperations simpMessageSendingOperations = AwareUtil.getBean(SimpMessageSendingOperations.class);

}
```


