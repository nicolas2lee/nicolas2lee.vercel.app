---
title: 用java spring boot快速开发后端api
date: 2022-07-24
description: Quick start of creation a back end by using java & spring boot with sql db
tag: java, spring boot
author: nicolas2lee
---

# 用java spring boot快速开发后端api
 
## 初始化开发框架
https://start.spring.io/
## 添加第一个API
### API guideline
https://opensource.zalando.com/restful-api-guidelines/#introduction
## postgresql数据库的搭建以及建模
https://www.postgresqltutorial.com/
### 使用docker compose快速创建一个postgresql db实例
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
  creator_id VARCHAR(50) NOT NULL,
  content VARCHAR (50) NOT NULL,
  last_updated TIMESTAMP NOT NULL,
  updated_by TIMESTAMP,
  FOREIGN KEY (creator_id)
  REFERENCES users(id)
);
```
### 插入一些测试数据
```sql
INSERT INTO users(id, username, password, email) VALUES (1, 'test1', 'password1', 'test1@test.com');
INSERT INTO posts(id, creator_id, content) VALUES (1, 1, 'my first post');
```
