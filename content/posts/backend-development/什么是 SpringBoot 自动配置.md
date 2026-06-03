---
title: 什么是 SpringBoot 自动配置？
slug: what-is-spring-boot-autowiring
categories: 后端开发
date: 2025-09-27
---

## SpringBoot 的自动配置原理

在启动 `xxxxxApplication` 类上，查看`@SpringBootApplication`注解：

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420213628.png)

我们依次查看 `@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan` 注解。
### @SpringBootConfiguration

点击 `@SpringBootConfiguration`，如下：

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420214337.png)

可以发现底层其实是`@Configuration`注解，这个注解我们应该都不会陌生，在自定义配置类的时候我们会使用到此注解。

`@Configuration` 注解是 Spring 框架中的一个核心注解，用于标识一个类作为 Spring 的配置类。它的主要功能包括：

功能：

1. **定义 Bean**：通过在 @Configuration 类中使用 @Bean 注解的方法，可以定义由 Spring 管理的 Bean
2. **配置 Spring 上下文**：@Configuration 类相当于 XML 配置文件，集中管理应用的配置

总结：`@SpringBootConfiguration` 并非只是简单地包装了 `@Configuration`，而是利用 `@Configuration` 的核心功能，并在此基础上构建了 Spring Boot 特有的配置类注解（语义明确，主配置类），使其成为 Spring Boot 应用配置的核心组成部分
### @EnableAutoConfiguration

“约定大于配置”这么一句话，SpringBoot 靠的就是该注解。简单来说，这个注解可以帮助我们**自动载入**应用程序所需要的所有**默认配置**。

查看 `@EnableAutoConfiguration` 源码，如下：

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420220120.png)

依靠`@AutoConfigurationPackage` 和 `@Import(...)` 这 2 个注解。

查看 `@AutoConfigurationPackage`，如下：

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420221113.png)

该注解也是依靠 `@Import` 注解，只不过导入的内容不同。所以 `@EnableAutoConfiguration` 的功能很大程度上和`Autoxxxx.Register.class`、`AutoxxxxSelector.class`这 2 个类有关。

先点击`xxxx.Register.class`，代码如下：

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420222545.png)

其功能：在**默认**的情况下将主配置类（`@SpringBootApplication`注解标注的类）所在的包及其子包里边的**组件**扫描到 Spring 容器中。

@ComponentScan 注解的功能也是扫描包，二者异同点：

| 特性         | @AutoConfigurationPackage                                    | @ComponentScan                                   |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------- |
| **核心目的** | 注册自动配置的基础包                                         | 显式指定扫描包，注册 Spring 组件 (Bean)          |
| **关注点**   | 自动配置                                                     | 组件扫描和 Bean 注册                             |
| **显式性**   | 隐式 (通常由 `@SpringBootApplication` 引入)                  | 显式 (开发者在配置类上使用)                      |
| **作用范围** | 相对窄，主要影响自动配置                                     | 更广，影响组件扫描和 Bean 注册                   |
| **灵活性**   | 较低，默认注册注解类所在包                                   | 较高，可指定多个包，使用过滤器控制扫描           |
| **机制**     | `@Import(AutoConfigurationPackages.Registrar.class)` 注册包名 | 直接触发组件扫描器，注册 BeanDefinition          |
| **使用场景** | Spring Boot 内部，Starter 开发，一般应用不直接使用           | 应用结构定义，模块化应用，控制扫描范围，常用注解 |

举个例子，你用了 Spring Data JPA，可能会在实体类上写`@Entity`注解。这个`@Entity`注解由`@AutoConfigurationPackage`扫描并加载，而我们平时开发用的`@Controller/@Service/@Component/@Repository`这些注解是由`ComponentScan`来扫描并加载的。

- 简单理解：这二者**扫描的对象是不一样**

再回到`@EnableAutoConfiguration`下，查看`@Import(AutoConfigurationImportSelector.class)`，具体的实现：

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420224626.png)

继续查看`getAutoConfigurationEntry()`方法，查看配置信息的来源：

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420224940.png)

对于 `configurations`配置信息的来源，继续查看`getCandidateConfigurations()`方法，如下：

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420225222.png)

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420225340.png)

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420225417.png)

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420230521.png)

梳理一下：

- 调用链为：`loadFactoryNames() -> loadSpringFactories() -> 扫描所有 jar 包下的 META-INF/spring.factories`
- 图二中的 `key` 也就是 `factoryTypeName` 来自于图 1 中调用的图 4 方法

对于 austin 项目，启动时默认加载了 241 个配置类：

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250420230912.png)
### @ComponentScan

回到`SpringBootApplication`的最后一个功能注解`@ComponentScan`

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250421150610.png)

`@ComponentScan`是一个扫描注解，主要用于自动扫描和注册符合规则的组件或 bean 定义到 Spring 容器中。该注解默认是扫描当前类下的 package，也可以指定/排除包路径扫描，然后将 `@Controller/@Service/@Component/@Repository` 等注解加载到IOC容器中。

再点击`@Filter`注解过滤的`AutoConfigurationExcludeFilter.class`类

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250421152423.png)

- 避免重复扫描这些配置，导致发生冲突
## 总结

`@SpringBootApplication`等同于下面三个注解：

- `@SpringBootConfiguration`
- `@EnableAutoConfiguration`
- `@ComponentScan`

其中`@EnableAutoConfiguration`是关键（启用自动配置），内部实际上就去加载`META-INF/spring.factories`文件的信息，然后筛选出以`EnableAutoConfiguration`为 key 的数据，加载到 IOC 容器中，实现自动配置功能！

![](https://cdn.img.turingzy.cn/2025/Pasted%20image%2020250421155006.png)
# 参考

- [SpringBoot-The @SpringBootApplication Annotation](https://docs.spring.io/spring-boot/tutorial/first-application/index.html#getting-started.first-application.code.spring-boot-application)
- [SpringBoot自动配置原理！](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484637&idx=1&sn=956c14daacc3e09367d9c27458b09f7f&chksm=ebd745dcdca0ccca6c173d32b6f8299f61d950990ee7c6eb2ec676f5ce0ad9b0ba306306a952###rd)