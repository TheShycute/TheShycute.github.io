---
layout: post
title: "黑马程序员学习之旅——从 Java 基础到微服务"
subtitle: "一套课程，走完 Java 全栈开发的学习路径"
date: 2025-08-10 12:00:00
author: "TheShycute"
header-img: "img/post-bg-web.jpg"
catalog: true
tags:
  - 学习
  - Java
  - Spring Boot
  - 前端
  - 课程笔记
---

## 缘起

黑马程序员是我系统学习 Java 全栈开发的重要起点。很多人问我为什么选择黑马——答案很简单：**项目驱动 + 体系完整**。

## 完整学习路线

```
Java 基础 (1-2 月)
    │
    ▼
数据库 + JDBC (3-4 周)
    │
    ▼
前端基础 HTML/CSS/JS (3-4 周)
    │
    ▼
Java Web + Servlet (4 周)
    │
    ▼
Spring 全家桶 (6-8 周)
    │
    ▼
项目实战 (4-6 周)
    │
    ▼
微服务 + 分布式 (4 周)
```

## 第一阶段：Java 基础

### 核心知识点

```java
// 面向对象三大特性
public class Animal {
    private String name;  // 封装：私有属性
    
    public Animal(String name) {
        this.name = name;
    }
    
    public void makeSound() {
        System.out.println("...");
    }
}

// 继承
public class Dog extends Animal {
    public Dog(String name) {
        super(name);  // 调用父类构造
    }
    
    // 多态：重写父类方法
    @Override
    public void makeSound() {
        System.out.println("汪汪！");
    }
}
```

### 重点与易错点

| 知识点 | 常见误区 | 正确理解 |
|--------|---------|---------|
| `==` vs `equals()` | 以为 `==` 比较内容 | `==` 比较引用地址，`equals()` 比较内容 |
| String 不可变性 | 以为修改了原字符串 | 每次操作都返回新对象 |
| 静态方法 | 在静态方法中用 `this` | 静态方法属于类，不能访问实例成员 |
| 集合线程安全 | 盲目用 Vector | 优先用 `Collections.synchronizedList()` 或 CopyOnWriteArrayList |

### 集合框架核心

```java
// List：有序可重复
List<String> list = new ArrayList<>();
list.add("Java");
list.add("Python");

// Set：不可重复
Set<Integer> set = new HashSet<>();
set.add(1);
set.add(1);  // 不会重复添加

// Map：键值对
Map<String, Integer> map = new HashMap<>();
map.put("张三", 90);
map.get("张三");  // 90

// 遍历方式：Stream API
list.stream()
    .filter(s -> s.startsWith("J"))
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

## 第二阶段：数据库

### MySQL 重点

```sql
-- 多表查询
SELECT 
    o.id AS order_id,
    u.name AS user_name,
    d.name AS dish_name,
    od.number,
    od.amount
FROM orders o
JOIN user u ON o.user_id = u.id
JOIN order_detail od ON o.id = od.order_id
JOIN dish d ON od.dish_id = d.id
WHERE o.status = 1
ORDER BY o.create_time DESC;

-- 索引优化
CREATE INDEX idx_user_status ON orders(user_id, status);

-- EXPLAIN 分析查询
EXPLAIN SELECT * FROM orders WHERE user_id = 123;
```

### MyBatis 核心

```xml
<!-- mapper XML -->
<select id="getOrderWithDetail" resultMap="orderDetailMap">
    SELECT o.*, od.*, d.name as dish_name
    FROM orders o
    LEFT JOIN order_detail od ON o.id = od.order_id
    LEFT JOIN dish d ON od.dish_id = d.id
    WHERE o.id = #{orderId}
</select>

<resultMap id="orderDetailMap" type="OrderVO">
    <id property="id" column="id"/>
    <result property="number" column="number"/>
    <collection property="details" ofType="OrderDetailVO">
        <result property="dishName" column="dish_name"/>
        <result property="number" column="number"/>
        <result property="amount" column="amount"/>
    </collection>
</resultMap>
```

## 第三阶段：Spring 全家桶

### Spring Framework 核心

| 概念 | 说明 | 代码示例 |
|------|------|---------|
| **IoC** | 控制反转，对象创建权交给容器 | `@Autowired` |
| **AOP** | 面向切面编程，统一处理横切逻辑 | `@Aspect` + `@Around` |
| **DI** | 依赖注入，解耦组件间依赖 | `@Service` + `@Repository` |

```java
// AOP 实现日志记录
@Aspect
@Component
public class LogAspect {
    
    @Around("@annotation(org.springframework.web.bind.annotation.PostMapping)")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        String method = joinPoint.getSignature().getName();
        Object[] args = joinPoint.getArgs();
        
        System.out.println("调用方法: " + method);
        long start = System.currentTimeMillis();
        
        Object result = joinPoint.proceed();
        
        long time = System.currentTimeMillis() - start;
        System.out.println("耗时: " + time + "ms");
        
        return result;
    }
}
```

### Spring Boot 自动配置

```yaml
# application.yml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/takeout
    username: root
    password: ${DB_PASSWORD}  # 环境变量注入
  
  redis:
    host: localhost
    port: 6379
    
mybatis:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
```

## 第四阶段：项目实战

两个完整的实战项目覆盖了日常开发的绝大多数场景：

### 学成在线（教育平台）
- SSM 框架整合（Spring + Spring MVC + MyBatis）
- 前后端分离架构
- RESTful API 设计
- 用户认证与授权

### 苍穹外卖（餐饮平台）
- Spring Boot 自动配置
- Vue.js + Element UI 前端
- Redis 缓存优化
- 订单状态机设计
- 微信支付集成（模拟）

## 学习心得

### ❌ 常见陷阱

1. **只看不写**：看视频觉得自己懂了，一敲代码全是 bug
2. **求多不求精**：同时学 Java、Python、Go……最后一门都不精
3. **跳过基础**：直接学框架，遇到问题连异常信息都看不懂
4. **完美主义**：想把每个知识点都学透再往下走，结果永远停在开头

### ✅ 高效方法

1. **番茄工作法**：25 分钟专注学习 + 5 分钟休息
2. **隔天复习**：当天学的第二天回顾，遗忘曲线管理
3. **输出驱动**：学完一个模块就写博客/做笔记
4. **项目思维**：每学一个技术点，想想它在项目中怎么用的

> "编程不是学会的，是做会的。" ——这句话值一万节课。

> 📂 学习代码：[GitHub - xuechengzaixian](https://github.com/TheShycute/xuechengzaixian)
> 📂 练习代码：[Gitee - javastudy](https://gitee.com/TheShycute/javastudy)（私有）
