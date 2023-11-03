---
title: Java开发踩坑记录
date: 2021.11.3
author: Aik
img: ../images/deepLearning.jpg
top: true
hide: false
cover: true
coverImg: /images/1.jpg
password: 
toc: true
mathjax: true
summary: Java开发踩坑记录
categories: Java
tags:
  - Java
  - Mybatis
  - Redis
---

<!-- more -->

### mybatis-plus

Spring Boot整合mybatis-plus时，使用@Mapper注解无效，无法注入Bean，使用@MapperScan指定扫描路径可以。

```java
rg.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'teacherInfoServiceImpl': Unsatisfied dependency expressed through field 'baseMapper'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.xwy.core.mapper.TeacherInfoMapper' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:713) ~[spring-beans-5.3.30.jar:5.3.30]
	......
```

尝试删除springfox-swagger2依赖，如果可以解决那么通过下面的方式也同样可以解决。

```xml
mvc:
    pathmatch:
      matching-strategy: ant_path_matcher
	  # matching-strategy: path_pattern_parser
```

swagger2采用ant_path_matcher来完成路径匹配，Spring Boot高版本默认采用path_pattern_parser进行匹配。

**注意检查@Mapper的引用包是不是``import org.apache.ibatis.annotations.Mapper;``**



