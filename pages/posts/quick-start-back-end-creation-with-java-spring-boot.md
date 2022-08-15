---
title: 用java spring boot快速开发后端api
date: 2022-08-08
description: 如何使用docker配置postgresql数据库，以及用java spring boot快速开发后端api
tag: java, spring boot
author: nicolas2lee
---

# 用java spring boot快速开发后端api

## Postgresql数据库的搭建以及建模
https://www.postgresqltutorial.com/
### 使用docker compose快速创建一个postgresql db实例
docker-compose.yml
```yaml
version: '3.8'
services:
  db:
    image: postgres:14.1-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    ports:
      - '5432:5432'
    volumes:
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```
```
docker-compose up
```
### 创建post和user的表 
```yaml
CREATE TABLE IF NOT EXISTS users(
  id serial PRIMARY KEY,
  username VARCHAR(50) NOT null,
  password VARCHAR(50) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE IF NOT EXISTS posts(
  id serial PRIMARY KEY,
  creator_id serial NOT NULL,
  content VARCHAR (50) NOT NULL,
  last_updated TIMESTAMP NOT NULL,
  FOREIGN KEY (creator_id)
  REFERENCES users(id)
);
```
### 插入一些测试数据
```sql
INSERT INTO users(id, username, password, email) VALUES (1, 'test1', 'password1', 'test1@test.com');
INSERT INTO posts(id, creator_id, content) VALUES (1, 1, 'my first post');
```

## 使用java spring boot搭建后端app
### 初始化项目
[Spring io ](https://start.spring.io/) 可以帮助我们快速创建以及简化一些配置
### 代码分层以及object命名
代码分层是一个宏大的topic，没有万能的架构，常见有2种:
1. 单一模块(mono module)
对于相对简单的项目，你又不想分很多不同层，那么我们就可以创建一个单一模块的项目
2. 多层分层模块(multiple modules)
基于Domain Driven Development架构，[clean architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
#### 单一模块(mono module)
```
|_ exposition(controller or api)
    主要是将核心功能暴露给外界系统，最常见的use case就是RESTful API
    这里常用的java pojo就是Data Transfer Oject
|_ service
    负责核心商业逻辑，进行一些服务的调度
    这里很少定义纯的java pojo
|_ domain
    负责核心商业模型，使其不需要依赖外界系统，比如UI，数据库
    常见的java pojo命名有Value Object, Business Object，个人还是倾向于直接用商业对象命名比如: User, Post, Product
|_ repository
    对于old school来说，和DAO的实质是一样的。主要负责从外部系统中获取数据。最常用的是从数据库里获取数据
    这里很少定义纯的java pojo
|_ entity
    常指数据库中表和java一一对应的mapping
    这里常用的java pojo就是entity
```
常见的开发流程是:
domain -> service -> entity-> repository -> exposition
1. domain和service代表的是核心商业逻辑，所以往往需要和domain experts一起定义
2. 在确定完商业核心逻辑之后，就需要考虑存储层，就是entity和repository 
3. 之后再完成api层
##### repository & entity - 使用spring data进行postgresql连接配置
1. 添加jpa依赖
   ```groovy
   implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
   runtimeOnly 'org.postgresql:postgresql'
   ```
2. data source的配置
    ```yaml
    spring:
      jpa:
        show-sql: true
        open-in-view: false
      datasource:
        url: jdbc:postgresql://localhost:5432/postgres
        username: postgres
        password: postgres
    ```
3. 受用spring data jpa来实现ORM
spring data jpa提供了3种方法来实习sql query
    1. 使用method name来生成
       ```java
       interface PersonRepository extends Repository<Person, Long> {

        List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);

        // Enables the distinct flag for the query
        List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
        List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);
        
        // Enabling ignoring case for an individual property
        List<Person> findByLastnameIgnoreCase(String lastname);
        // Enabling ignoring case for all suitable properties
        List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);
        
        // Enabling static ORDER BY for a query
        List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
        List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
       }
       ```
    2. 使用JPQL   
        ```java
        @Query("SELECT u FROM UserEntity u WHERE u.status = 1")
        Collection<User> findAllActiveUsers();
        ```
    3. Native sql原生sql
        ```java
        @Query(
            value = "SELECT * FROM USERS u WHERE u.status = 1",
            nativeQuery = true)
        Collection<User> findAllActiveUsersNative();
        ```
##### controller - 添加API
```java
@RestController
@RequestMapping("/posts")
public class PostController {
    @GetMapping("/welcome")
    public String welcome(){
        return "welcome";
    }
}
```

[Zalando RESTful API Guide](https://opensource.zalando.com/restful-api-guidelines/#introduction)
### Swagger
使用spring doc来添加swagger，题外话这个lib是我前同事开发的
```groovy
	implementation 'org.springdoc:springdoc-openapi-ui:1.6.9'
```

```java
@Configuration
public class SwaggerConfig {
    @Bean
    public GroupedOpenApi api() {
        return GroupedOpenApi.builder()
                .group("quickstart-api")
                .pathsToMatch("/**")
                .build();
    }
}
```
### Actuator
通过actuator可以方便实现observability，获取health, metrics, log等info
```groovy
	implementation 'org.springframework.boot:spring-boot-starter-actuator'
```
在application.yml种配置
```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'
```