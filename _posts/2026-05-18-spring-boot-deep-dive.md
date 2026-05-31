---
layout: post
title: "Spring Boot 核心技术栈深度解析"
subtitle: "IoC、AOP、自动配置与生态全景"
date: 2026-05-18 10:00:00
author: "天使皮卡丘丨"
header-img: "img/post-bg-web.jpg"
catalog: true
tags:
  - 技术
  - Java
  - Spring Boot
  - 后端
---

## Spring 生态全景

Spring 早已超越了单个框架的范畴，成为 Java 领域最庞大的生态系统：

```
            ┌──────────────────────────────┐
            │         Spring Boot           │  ← 快速开发基石
            │    (自动配置 / Starter / Actuator)  │
            ├──────────────────────────────┤
            │     Spring Framework           │  ← 核心容器
            │     (IoC / AOP / MVC)          │
            ├──────────────────────────────┤
            │      Spring Cloud              │  ← 微服务治理
            │  (注册中心 / 配置中心 / 网关)    │
            ├──────────────────────────────┤
            │   Spring Security / Data ...   │  ← 专项能力
            └──────────────────────────────┘
```

## IoC：控制反转

### 传统方式 vs IoC

```
【传统方式】强耦合                     【IoC 方式】松耦合
┌─────────┐                           ┌─────────┐
│ UserSvc │──→ new UserDAO()          │ UserSvc │──→ @Autowired
└─────────┘                           └─────────┘
                                             │
                                      ┌──────┴──────┐
                                      │  IoC 容器    │
                                      │  管理对象     │
                                      └──────┬──────┘
                                             │
                                      ┌──────┴──────┐
                                      │  UserDAO   │
                                      └─────────────┘
```

**IoC 的核心**：对象创建和管理的控制权，从代码中转移到外部容器。

### Bean 的生命周期

```
① 实例化（构造器）
    ↓
② 属性注入（@Autowired）
    ↓
③ Aware 回调（BeanNameAware 等）
    ↓
④ @PostConstruct 初始化方法
    ↓
⑤ Bean 就绪，可被使用
    ↓
⑥ @PreDestroy 销毁方法
    ↓
⑦ 容器关闭
```

### Bean 作用域

| 作用域 | 说明 | 典型场景 |
|--------|------|---------|
| **singleton** | 整个容器仅一个实例（默认） | Service、Repository |
| **prototype** | 每次获取都创建新实例 | 有状态的 Bean |
| **request** | 每个 HTTP 请求一个实例 | Web 请求参数 |
| **session** | 每个 HTTP 会话一个实例 | 用户登录信息 |

## DI：依赖注入

### 三种注入方式

| 方式 | 示例 | 推荐度 |
|------|------|--------|
| **构造器注入** | `public UserService(UserDAO dao)` | ⭐⭐⭐ 首选 |
| **Setter 注入** | `setUserDAO(UserDAO dao)` | ⭐⭐ 可选依赖 |
| **字段注入** | `@Autowired UserDAO dao` | ⭐ 简单但不好测试 |

```java
// ✅ 推荐：构造器注入（不可变、易测试）
@Service
public class UserService {
    private final UserDAO userDAO;
    
    public UserService(UserDAO userDAO) {
        this.userDAO = userDAO;
    }
}

// ⚠️ 字段注入（简洁但不利于测试）
@Service
public class UserService {
    @Autowired
    private UserDAO userDAO;
}
```

## AOP：面向切面编程

### 核心概念

```
横切关注点（Cross-cutting Concerns）：
┌──────────────────────────────────────┐
│  日志记录 │ 事务管理 │ 权限校验 │ 缓存处理  │
└──────────────────────────────────────┘
              ↓ AOP 统一处理
┌──────────────────────────────────────┐
│          业务逻辑（核心关注点）          │
└──────────────────────────────────────┘
```

### AOP 术语

| 术语 | 说明 |
|------|------|
| **Aspect（切面）** | 横切逻辑的模块，如日志切面 |
| **Join Point（连接点）** | 程序执行中可插入切面的点 |
| **Advice（通知）** | 切面在特定连接点的行为 |
| **Pointcut（切入点）** | 匹配连接点的表达式 |
| **Weaving（织入）** | 将切面应用到目标对象的过程 |

### 五种通知类型

| 通知 | 执行时机 |
|------|---------|
| **@Before** | 目标方法执行前 |
| **@After** | 目标方法执行后（无论成败） |
| **@AfterReturning** | 目标方法成功返回后 |
| **@AfterThrowing** | 目标方法抛出异常后 |
| **@Around** | 包裹目标方法（最灵活） |

## Spring Boot 自动配置

### 核心理念：约定大于配置

```
传统 Spring：需要大量 XML / Java Config
Spring Boot：开箱即用，零配置启动

@SpringBootApplication = 
    @Configuration       // 配置类
    + @EnableAutoConfiguration  // 自动配置
    + @ComponentScan     // 组件扫描
```

### 自动配置原理

```
① @SpringBootApplication 启动
    ↓
② @EnableAutoConfiguration 激活
    ↓
③ 扫描 classpath 下 META-INF/spring.factories
    ↓
④ @Conditional 条件注解判断
    ├─ @ConditionalOnClass  → 类存在时生效
    ├─ @ConditionalOnMissingBean → Bean 不存在时生效
    ├─ @ConditionalOnProperty → 配置存在时生效
    └─ @ConditionalOnWebApplication → Web 环境时生效
    ↓
⑤ 满足条件的配置类自动加载
```

### 常用 Starter

| Starter | 包含功能 |
|---------|---------|
| `spring-boot-starter-web` | Spring MVC + Tomcat |
| `spring-boot-starter-data-jpa` | JPA + Hibernate |
| `spring-boot-starter-security` | Spring Security |
| `spring-boot-starter-test` | JUnit + Mockito |
| `mybatis-spring-boot-starter` | MyBatis 集成 |

## Spring MVC 请求流程

```
用户请求
    │
    ▼
DispatcherServlet（前端控制器）
    │
    ▼
HandlerMapping（查找处理器）
    │
    ▼
HandlerInterceptor（拦截器链）
    │
    ▼
HandlerAdapter（执行处理器）
    │
    ▼
Controller（业务处理）
    │
    ▼
ViewResolver（视图解析）
    │
    ▼
返回响应
```

## Spring 事务管理

### 声明式事务

```
@Transactional 的核心属性：

propagation（传播行为）
├─ REQUIRED（默认）：有事务则加入，无则新建
├─ REQUIRES_NEW：总是新建独立事务
├─ SUPPORTS：有事务则加入，无则以非事务运行
└─ MANDATORY：必须在事务中运行，否则报错

isolation（隔离级别）
├─ DEFAULT：使用数据库默认级别
├─ READ_COMMITTED：避免脏读
├─ REPEATABLE_READ：避免不可重复读
└─ SERIALIZABLE：避免幻读（性能最低）
```

### 事务失效的常见场景

1. **方法非 public**：Spring AOP 基于代理，private 方法不走代理
2. **同类方法调用**：`this.method()` 不触发 AOP 代理
3. **异常被捕获**：`try-catch` 吃掉异常，事务不回滚
4. **非运行时异常**：默认只对 `RuntimeException` 回滚

## 总结

Spring 生态的核心思想可以归纳为三个关键词：

- **解耦**：IoC 和 DI 让组件松散协作
- **重用**：AOP 统一处理横切逻辑
- **简化**：自动配置让开发聚焦业务

掌握这些核心概念，再去看 Spring Boot、Spring Cloud，会发现它们的内核始终如一。

> 相关文章：[苍穹外卖：Spring Boot + Vue.js 全栈项目实战](/2026/05/28/sky-takeout/)
> 相关文章：[Java 全栈开发学习路线与实战总结](/2026/05/27/darkhorse-learning/)
