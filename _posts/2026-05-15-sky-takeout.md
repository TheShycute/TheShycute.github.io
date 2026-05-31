---
layout: post
title: "苍穹外卖——前后端分离餐饮系统的全栈实践"
subtitle: "Spring Boot + Vue.js 企业级项目从零到一"
date: 2026-05-15 12:00:00
author: "TheShycute"
header-img: "img/post-bg-alibaba.jpg"
catalog: true
tags:
  - 项目
  - Java
  - Spring Boot
  - Vue
  - 全栈
---

## 项目介绍

苍穹外卖是一个完整的外卖点餐平台，涵盖三大端：用户端（C 端）、商家端（B 端）、管理后台。通过这个项目，我完成了从前端到后端的完整全栈开发实践。

## 系统架构

```
┌──────────────────────────────────────────────┐
│              Nginx (反向代理 + 静态资源)         │
├──────────────┬──────────────┬────────────────┤
│   用户端      │   商家端      │   管理后台       │
│  Vue.js      │  Vue.js      │  Vue.js        │
├──────────────┴──────────────┴────────────────┤
│              统一 API 网关 (Spring Boot)        │
├──────────────┬──────────────┬────────────────┤
│  用户服务     │  订单服务     │  菜品服务        │
├──────────────┴──────────────┴────────────────┤
│              MySQL + Redis                    │
└──────────────────────────────────────────────┘
```

## 数据库设计

### 核心表结构

```sql
-- 用户表
CREATE TABLE user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    openid VARCHAR(45) UNIQUE COMMENT '微信openid',
    name VARCHAR(32),
    phone VARCHAR(11),
    sex VARCHAR(2),
    avatar VARCHAR(500),
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 分类表
CREATE TABLE category (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    type INT COMMENT '1:菜品分类 2:套餐分类',
    name VARCHAR(32),
    sort INT DEFAULT 0,
    status INT DEFAULT 1,
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 菜品表
CREATE TABLE dish (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(64) NOT NULL,
    category_id BIGINT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    image VARCHAR(500),
    description VARCHAR(255),
    status INT DEFAULT 1 COMMENT '0:停售 1:起售',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_category (category_id)
);

-- 订单表
CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    number VARCHAR(50) UNIQUE COMMENT '订单号',
    status INT COMMENT '1:待付款 2:待接单 3:已接单 4:派送中 5:已完成 6:已取消',
    user_id BIGINT NOT NULL,
    address_book_id BIGINT,
    order_time DATETIME,
    checkout_time DATETIME,
    pay_method INT COMMENT '1:微信 2:支付宝',
    amount DECIMAL(10,2),
    remark VARCHAR(100),
    INDEX idx_user (user_id),
    INDEX idx_status (status)
);

-- 订单明细表
CREATE TABLE order_detail (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(64),
    order_id BIGINT NOT NULL,
    dish_id BIGINT,
    number INT DEFAULT 1,
    amount DECIMAL(10,2),
    INDEX idx_order (order_id)
);
```

### 缓存策略（Redis）

```java
@Service
public class DishService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 热门菜品走缓存
    @Cacheable(value = "dish", key = "#categoryId")
    public List<DishVO> getDishByCategory(Long categoryId) {
        return dishMapper.selectByCategory(categoryId);
    }
    
    // 更新时清除缓存
    @CacheEvict(value = "dish", key = "#dish.categoryId")
    public void updateDish(Dish dish) {
        dishMapper.update(dish);
    }
    
    // 用户购物车存 Redis
    public void addToCart(Long userId, CartItem item) {
        String key = "cart:" + userId;
        redisTemplate.opsForHash().put(key, item.getDishId().toString(), item);
    }
}
```

## API 接口设计

### RESTful API 规范

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/user/dish/list?categoryId=1` | 获取分类菜品 |
| POST | `/api/user/order/submit` | 提交订单 |
| PUT | `/api/user/order/cancel/{id}` | 取消订单 |
| GET | `/api/admin/dish/page?page=1&pageSize=10` | 分页查询菜品 |
| POST | `/api/admin/dish` | 新增菜品 |
| PUT | `/api/admin/dish` | 修改菜品 |
| DELETE | `/api/admin/dish/{id}` | 删除菜品 |

### 统一响应格式

```java
@Data
public class Result<T> {
    private Integer code;   // 1:成功 0:失败
    private String msg;     // 提示信息
    private T data;         // 数据
    
    public static <T> Result<T> success(T data) {
        Result<T> r = new Result<>();
        r.code = 1;
        r.data = data;
        return r;
    }
    
    public static <T> Result<T> error(String msg) {
        Result<T> r = new Result<>();
        r.code = 0;
        r.msg = msg;
        return r;
    }
}
```

### JWT 认证流程

```
用户登录 → 验证账号密码
    │
    ▼
生成 JWT Token（包含用户ID、角色）
    │
    ▼
前端存储 Token → 后续请求在 Header 中携带
    │
    ▼
后端拦截器验证 → @JwtToken 注解标记
```

```java
@Component
public class JwtInterceptor implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                            HttpServletResponse response, 
                            Object handler) {
        String token = request.getHeader("Authorization");
        if (token == null || !token.startsWith("Bearer ")) {
            throw new UnauthorizedException("未登录");
        }
        
        Claims claims = JwtUtil.parseToken(token.substring(7));
        Long userId = claims.get("userId", Long.class);
        
        // 存入 ThreadLocal
        UserContext.setCurrentId(userId);
        return true;
    }
}
```

## 功能模块详解

### 用户端功能流

```
浏览菜品 → 加入购物车 → 确认订单 → 选择地址 → 提交支付
    │                                              │
    └──────────────── 查看订单状态 ←────────────────┘
```

- 🛒 **智能购物车**：基于 Redis 实现，支持多店铺隔离
- 📍 **地址管理**：CRUD + 默认地址设置
- 💳 **支付集成**：对接微信支付 API（模拟）
- ⭐ **评价系统**：订单完成后可对菜品评分

### 商家端核心功能

- 📊 **数据概览**：今日营业额、订单数、热门菜品 TOP10
- 🍳 **菜品管理**：支持批量上下架、图片上传
- 📋 **订单处理**：接单、拒单、出餐、完成
- 📈 **销售统计**：按日/周/月维度展示趋势图

### 管理后台

- 👥 **员工管理**：账号启停、角色分配
- 🏪 **分类管理**：菜品分类和套餐分类的维护
- 📊 **数据报表**：平台整体运营数据

## 项目亮点总结

| 亮点 | 技术实现 |
|------|---------|
| 前后端分离 | Vue.js + Spring Boot，通过 API 网关通信 |
| 无状态认证 | JWT Token + 拦截器 + ThreadLocal |
| 缓存优化 | Spring Cache + Redis 缓存热门数据 |
| 文件上传 | 阿里云 OSS 对象存储 |
| 接口文档 | Swagger/Knife4j 自动生成 |
| 全局异常处理 | @RestControllerAdvice 统一处理 |

## 收获与反思

苍穹外卖是我第一个完整的全栈项目。最大的收获不是学会了 Spring Boot 或 Vue，而是理解了**前后端协作的完整链路**——从需求分析到数据库设计，从 API 设计到前端开发，从测试到部署。

> 项目仓库：[Gitee - sky-take-out](https://gitee.com/TheShycute/sky-take-out)（私有）
