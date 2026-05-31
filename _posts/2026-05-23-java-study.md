---
layout: post
title: "Java 核心知识点总结与学习笔记"
subtitle: "从 Hello World 到框架实战的成长记录"
date: 2026-05-23 12:00:00
author: "天使皮卡丘丨"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
  - 学习
  - Java
  - 笔记
  - 基础
---

## 我的 Java 学习之路

```
Hello World (Day 1)
    │
    ▼
基本语法 + 流程控制 (Week 1-2)
    │
    ▼
面向对象 (Week 3-4)
    │
    ▼
集合 + IO + 异常 (Week 5-6)
    │
    ▼
多线程 + 网络编程 (Week 7-8)
    │
    ▼
JDBC + 数据库 (Week 9-10)
    │
    ▼
Spring Boot 入门 (Week 11+)
```

## 面向对象核心

### 封装

```java
public class BankAccount {
    private double balance;  // 私有，外部不可直接访问
    
    // 提供受控的访问方式
    public double getBalance() {
        return balance;
    }
    
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        } else {
            throw new IllegalArgumentException("金额必须为正数");
        }
    }
    
    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        } else {
            throw new IllegalArgumentException("余额不足");
        }
    }
}
```

### 继承

```java
// 父类
class Employee {
    protected String name;
    protected double salary;
    
    public double getAnnualSalary() {
        return salary * 12;
    }
}

// 子类
class Manager extends Employee {
    private double bonus;
    
    @Override
    public double getAnnualSalary() {
        return super.getAnnualSalary() + bonus;  // 重写 + 复用
    }
}
```

### 多态

```java
// 接口定义行为规范
interface Payable {
    double calculatePay();
}

class FullTimeEmployee implements Payable {
    public double calculatePay() {
        return salary / 12;
    }
}

class Contractor implements Payable {
    public double calculatePay() {
        return hourlyRate * hoursWorked;
    }
}

// 多态应用：统一处理不同类型
public void processPayroll(List<Payable> workers) {
    for (Payable p : workers) {
        System.out.println(p.calculatePay());  // 无需关心具体类型
    }
}
```

## 集合框架深度

### 选型指南

```
需要快速查找？
    YES → HashMap / HashSet (O(1))
    NO  → 需要有序？
            YES → 需要排序？
                    YES → TreeMap / TreeSet (O(log n))
                    NO  → LinkedHashMap (插入顺序)
            NO  → 需要线程安全？
                    YES → ConcurrentHashMap
                    NO  → ArrayList / LinkedList
```

### HashMap 原理

```java
// JDK 8+ 中 HashMap 解决冲突的方式：
// 1. 链表长度 < 8：链表
// 2. 链表长度 >= 8：转为红黑树（提升查询效率）

// 扩容机制：默认负载因子 0.75
// 当 size > capacity * 0.75 时，扩容为原来的 2 倍

Map<String, Integer> map = new HashMap<>(16);  // 初始容量 16
map.put("key", 1);

// 底层结构：
// [0] -> (key, value) -> (key, value)  // 链表
// [1] -> 红黑树节点                       // 树化
// [2] -> null
// ...
```

## 多线程与并发

### 线程创建

```java
// 方式1：继承 Thread
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running");
    }
}

// 方式2：实现 Runnable（推荐）
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Runnable running");
    }
}

// 方式3：线程池（实际项目首选）
ExecutorService executor = Executors.newFixedThreadPool(10);
executor.submit(() -> {
    System.out.println("Task running in thread pool");
});
executor.shutdown();
```

### synchronized vs Lock

```java
// synchronized：JVM 内置，自动加锁/释放
public synchronized void method1() {
    // 临界区
}

// Lock：更灵活，可中断、可超时、可条件等待
private final ReentrantLock lock = new ReentrantLock();

public void method2() {
    lock.lock();
    try {
        // 临界区
    } finally {
        lock.unlock();  // 必须在 finally 中释放
    }
}
```

### 常用设计模式

| 模式 | 用途 | Java 示例 |
|------|------|-----------|
| **单例** | 全局唯一实例 | `Runtime.getRuntime()` |
| **工厂** | 创建对象解耦 | `Calendar.getInstance()` |
| **观察者** | 一对多通知 | `PropertyChangeListener` |
| **代理** | 访问控制 | Spring AOP |
| **策略** | 算法族可切换 | `Comparator` 接口 |

```java
// 单例模式（双重检查锁定）
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

// 工厂模式
interface Animal { void speak(); }
class Dog implements Animal { public void speak() { System.out.println("汪汪"); } }
class Cat implements Animal { public void speak() { System.out.println("喵喵"); } }

class AnimalFactory {
    public static Animal create(String type) {
        return switch (type) {
            case "dog" -> new Dog();
            case "cat" -> new Cat();
            default -> throw new IllegalArgumentException("未知类型");
        };
    }
}
```

## JVM 基础知识

```java
// JVM 内存模型
┌─────────────────────────────┐
│         方法区（元空间）        │  类信息、常量、静态变量
├─────────────────────────────┤
│            堆                 │  对象实例、数组
│  ┌──────────┬──────────┐    │
│  │  年轻代   │  老年代   │    │
│  │ Eden/S0/S1│         │    │
│  └──────────┴──────────┘    │
├─────────────────────────────┤
│            栈                 │  局部变量、方法调用
├─────────────────────────────┤
│        本地方法栈             │  Native 方法
├─────────────────────────────┤
│        程序计数器             │  当前执行位置
└─────────────────────────────┘
```

### GC 基础知识

- **Minor GC**：清理年轻代，频繁但快速
- **Major GC / Full GC**：清理整个堆，慢但彻底
- Java 8 默认使用 Parallel GC，Java 9+ 默认 G1 GC

## 学习资源推荐

| 级别 | 资源 |
|------|------|
| 入门 | 《Head First Java》、廖雪峰 Java 教程 |
| 进阶 | 《Effective Java》、《Java 并发编程实战》 |
| 深入 | 《深入理解 Java 虚拟机》（周志明） |
| 实战 | 黑马程序员课程、尚硅谷课程 |

## 总结

Java 是一门值得深耕的语言——生态庞大、岗位众多、社区活跃。打好基础（语法、集合、多线程、JVM），再去学 Spring 全家桶，会事半功倍。

> 📂 项目仓库：[Gitee - javastudy](https://gitee.com/TheShycute/javastudy)（私有）
