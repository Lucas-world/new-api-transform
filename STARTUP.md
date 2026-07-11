# new-api 服务启动文档

## 1. 服务信息

### new-api

```text
Compose目录: /usr/dev/transferStation/new-api-transform
Compose文件: /usr/dev/transferStation/new-api-transform/docker-compose.dev.yml
服务名: new-api
网络模式: host
镜像: calciumion/new-api:latest
```

挂载：

```text
/usr/dev/transferStation/new-api-transform/data -> /data
/usr/dev/transferStation/new-api-transform/logs -> /app/logs
```

### PostgreSQL

```text
Compose目录: /www/dk_project/dk_app/postgresql/postgresql_YmzF
Compose文件: /www/dk_project/dk_app/postgresql/postgresql_YmzF/docker-compose.yml
服务名: postgresql_YmzF
网络模式: baota_net
端口: 0.0.0.0:35432 -> 5432
```

挂载：

```text
/www/dk_project/dk_app/postgresql/postgresql_YmzF/data -> /var/lib/postgresql/data
```

### Redis

```text
Compose目录: /www/dk_project/dk_app/redis/redis_TaC6
Compose文件: /www/dk_project/dk_app/redis/redis_TaC6/docker-compose.yml
服务名: redis_TaC6
网络模式: baota_net
端口: 0.0.0.0:26739 -> 6379
```

挂载：

```text
/www/dk_project/dk_app/redis/redis_TaC6/logs -> /logs
/www/dk_project/dk_app/redis/redis_TaC6/redis.conf -> /etc/redis/redis.conf
/www/dk_project/dk_app/redis/redis_TaC6/data -> /data
```

## 2. 启动前检查

确认目录存在：

```bash
ls -lah /usr/dev/transferStation/new-api-transform
ls -lah /www/dk_project/dk_app/postgresql/postgresql_YmzF
ls -lah /www/dk_project/dk_app/postgresql/postgresql_YmzF/data
ls -lah /www/dk_project/dk_app/redis/redis_TaC6
ls -lah /www/dk_project/dk_app/redis/redis_TaC6/data
```

确认 Docker 网络存在：

```bash
docker network ls | grep baota_net || docker network create baota_net
```

## 3. 启动顺序

启动顺序必须是：

```text
PostgreSQL -> Redis -> new-api
```

原因：`new-api` 使用 `host` 网络模式，会直接连接宿主机端口：

```text
PostgreSQL: 127.0.0.1:35432
Redis:      127.0.0.1:26739
```

## 4. 启动 PostgreSQL

```bash
cd /www/dk_project/dk_app/postgresql/postgresql_YmzF
docker compose up -d
```

检查 PostgreSQL：

```bash
docker ps | grep postgresql
ss -lntp | grep 35432
docker logs --tail=80 postgresql_ymzf-postgresql_YmzF-1
```

## 5. 启动 Redis

```bash
cd /www/dk_project/dk_app/redis/redis_TaC6
docker compose up -d
```

检查 Redis：

```bash
docker ps | grep redis
ss -lntp | grep 26739
docker logs --tail=80 redis_tac6-redis_TaC6-1
```

## 6. 启动 new-api

```bash
cd /usr/dev/transferStation/new-api-transform
docker compose -f docker-compose.dev.yml up -d new-api
```

检查日志：

```bash
docker logs -f --tail=100 new-api
```

## 7. 一键启动命令

```bash
docker network ls | grep baota_net || docker network create baota_net

cd /www/dk_project/dk_app/postgresql/postgresql_YmzF
docker compose up -d

cd /www/dk_project/dk_app/redis/redis_TaC6
docker compose up -d

cd /usr/dev/transferStation/new-api-transform
docker compose -f docker-compose.dev.yml up -d new-api

docker ps
docker logs --tail=100 new-api
```

## 8. 常见问题

### new-api 报 PostgreSQL 连接失败

错误类似：

```text
127.0.0.1:35432 connection refused
```

检查 PostgreSQL 是否启动并监听端口：

```bash
ss -lntp | grep 35432
docker logs --tail=100 postgresql_ymzf-postgresql_YmzF-1
```

### Redis 连接失败

检查 Redis 是否启动并监听端口：

```bash
ss -lntp | grep 26739
docker logs --tail=100 redis_tac6-redis_TaC6-1
```

### 容器名冲突

查看现有容器：

```bash
docker ps -a
```

不要删除数据目录，不要执行：

```bash
docker compose down -v
```

`-v` 会删除 Docker volume，存在误删数据风险。
