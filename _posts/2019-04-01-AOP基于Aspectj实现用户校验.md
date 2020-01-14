---
layout:     post
title:      AOP小demo
subtitle:   keep study
date:       2019-04-01
author:     aZhuang
header-img: img/post-bg-debug.png
catalog: true
tags:
    - AOP
    - 学习笔记
---

## 前言
本博客不作任何商用，单纯自己学习笔记整理，转载会注明出处 ，有疑问请联系邮箱；

### 产品实体类
```
public class Product {
    private Long id;
    private String name;
    public Product() {
    }
    public Product(Long id, String name) {
        this.id = id;
        this.name = name;
    }
    public Long getId() {
        return id;
    }
}
```
### 定义注解
```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface AdminOnly {
}
```
### 设置或者获取用户名的holder
```
public class CurrentUserHolder {
    private static final ThreadLocal<String> holder = new ThreadLocal<>();

    public static String get() {
        return holder.get() == null ? "unknown" : holder.get();
    }

    public static void set(String user) {
        holder.set(user);
    }
}
```
### 定义切面
```
@Aspect
@Component
public class SecurityAspect {
    @Autowired
    AuthSerivce authSerivce;

    //定义切点，扫描有AdminOnly注解的类
    @Pointcut("@annotation(AdminOnly)")
    public void adminOnly(){
    }
	//切点上织入的内容，进行校验
    @Before("adminOnly()")
    public void check(){
        System.out.println("checkAuthing...");
        authSerivce.checkAuth();
    }
}
```
### 校验服务
```
@Component
public class AuthSerivce {
    public void checkAuth() {
        String user = CurrentUserHoader.get();
        if (!"admin".equals(user)) {
            throw new RuntimeException("checkAuth failed!");
        }
    }
}
```
### 产品服务
```
@Service
@AdminOnly
@SpringBootApplication
public class ProductService {
    public void insert(Product product) {
        System.out.println("insert product!");
    }

    public void delete(Long id) {
        System.out.println("delete product!");
    }
}
```
### 测试类
@RunWith(SpringRunner.class)
@SpringBootTest
public class AopClientTest {
    @Autowired
    ProductService productService;

    @Test(expected = Exception.class)
    public void testDelete(){
        CurrentUserHolder.set("azhuang");
        productService.delete(1L);
    }
    
    @Test
    public void testDelete2(){
        CurrentUserHolder.set("admin");
        productService.delete(1L);
    }
}
```

```