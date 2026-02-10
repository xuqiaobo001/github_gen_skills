---
name: docker-compose-fullstack
description: Use this skill when setting up Docker Compose for full-stack deployment, organizing application services and middleware separately, configuring Nginx reverse proxy, or managing container orchestration. Triggers on keywords like "Docker Compose", "container deployment", "Nginx reverse proxy", "middleware orchestration".
version: 1.0.0
---

# Docker Compose 全栈部署

## Overview

服务分离编排（API/Worker/Web/Nginx），中间件独立编排，Nginx 统一入口，SSRF 代理过滤。

**Keywords**: Docker Compose, container, Nginx, reverse proxy, middleware, deployment

## Dify 源码引用

- `docker/docker-compose-template.yaml` — 主编排模板
- `docker/docker-compose.middleware.yaml` — 中间件独立编排
- `docker/nginx/` — Nginx 反向代理配置
- `docker/.env.example` — 完整配置项清单

## 服务编排结构

```yaml
services:
  api:
    image: dify-api
    depends_on: [db, redis, weaviate]
    deploy:
      replicas: 2

  worker:
    image: dify-api
    command: celery worker
    depends_on: [db, redis]

  web:
    image: dify-web
    depends_on: [api]

  nginx:
    image: nginx:alpine
    ports: ["80:80", "443:443"]
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
```

## 关键规则

- 应用服务与中间件分离编排
- 使用 `.env` 文件统一管理配置
- Nginx 作为唯一入口，后端不直接暴露端口
- SSRF Proxy 拦截内网非法请求

## 反模式

- 在 compose 文件中硬编码密码和密钥
- 所有服务写在一个 compose 文件中
- 后端服务直接暴露端口到宿主机
