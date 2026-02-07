# Skill 16: `docker-compose-fullstack` — Docker Compose 全栈部署

**适用场景**: 需要容器化部署前后端 + 中间件的项目。

**Dify 源码引用**:

- `docker/docker-compose-template.yaml` — 主编排模板
- `docker/docker-compose.middleware.yaml` — 中间件独立编排
- `docker/nginx/` — Nginx 反向代理配置
- `docker/certbot/` — SSL 证书自动续期
- `docker/.env.example` — 54KB 完整配置项清单

**服务编排结构**:

```yaml
# docker-compose.yaml 核心服务
services:
  api:                    # Flask 后端
    image: dify-api
    depends_on: [db, redis, weaviate]
    environment:
      - DB_HOST=db
      - REDIS_HOST=redis
    deploy:
      replicas: 2         # 水平扩展

  worker:                 # Celery Worker
    image: dify-api
    command: celery worker
    depends_on: [db, redis]

  web:                    # Next.js 前端
    image: dify-web
    depends_on: [api]

  nginx:                  # 反向代理
    image: nginx:alpine
    ports: ["80:80", "443:443"]
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./certbot/www:/var/www/certbot

# docker-compose.middleware.yaml 中间件
  db:
    image: postgres:15-alpine
    volumes: [db_data:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}

  weaviate:               # 向量数据库
    image: semitechnologies/weaviate
```

**关键规则**:

- 应用服务与中间件分离编排，中间件可独立升级
- 使用 `.env` 文件统一管理所有配置，`docker-compose.yaml` 中只引用变量
- Nginx 作为唯一入口，后端服务不直接暴露端口
- SSRF Proxy 拦截后端对内网的非法请求

**反模式**:

- ❌ 在 `docker-compose.yaml` 中硬编码密码和密钥
- ❌ 所有服务写在一个 compose 文件中（应分离中间件）
- ❌ 后端服务直接暴露 `5001:5001` 端口到宿主机
