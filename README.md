# 团团校园后端服务（school-circle-server）

团团校园项目的后端服务部署包，包含后端服务 Jar、管理页面、数据库初始化脚本与一键启动脚本。

## 一、目录结构

| 文件/目录 | 说明 |
| --- | --- |
| `yudao-server.jar` | 后端服务 Jar 包（Spring Boot 2.7.18，JDK 8 构建） |
| `school_circle.sql` | 数据库初始化脚本（含全部表结构与初始化数据） |
| `deploy.sh` | 一键启动后端服务脚本（备份 → 停止 → 部署 → 启动 → 健康检查） |
| `schoolAdmin/` | 后端服务管理页面（团团校园管理系统前端静态资源） |

## 二、环境要求

| 依赖 | 版本要求 | 说明 |
| --- | --- | --- |
| JDK | 1.8+ | Jar 包由 JDK 8 构建，需 JRE 8 及以上 |
| MySQL | 5.7 / 8.0 | 本地数据库，默认端口 `3306` |
| Redis | 5.0+ | 本地缓存，默认端口 `6379` |
| Nginx | 可选 | 用于托管 `schoolAdmin/` 管理页面 |

> 后端服务默认使用 `prod` 环境启动（端口 `48080`），prod 环境的数据库与 Redis 均指向本机
> `127.0.0.1`，与下方部署步骤保持一致。

## 三、部署步骤

### 步骤 1：创建并初始化本地数据库

1. 创建数据库 `school_circle`，账号 `root`，密码 `xtt@2026`：

```bash
mysql -uroot -p -e "CREATE DATABASE IF NOT EXISTS school_circle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

> 也可以使用 Navicat / DataGrip 等工具创建，字符集选择 `utf8mb4`。

2. 导入数据库初始化脚本：

```bash
mysql -uroot -p school_circle < school_circle.sql
```

> 使用 Navicat 导入时，选择 `school_circle` 数据库后右键 →「运行 SQL 文件」选择 `school_circle.sql` 即可。

### 步骤 2：创建并配置本地 Redis

1. 启动本地 Redis，监听默认端口 `6379`。
2. 设置 Redis 密码为 `xtt@2026`。

**macOS（Homebrew）方式：**

```bash
brew install redis
redis-cli config set requirepass xtt@2026
```

或修改 `redis.conf` 中的 `requirepass xtt@2026` 后重启 Redis，使其永久生效。

**Docker 方式：**

```bash
docker run -d --name school-redis -p 6379:6379 redis redis-server --requirepass xtt@2026
```

3. 验证 Redis 连接：

```bash
redis-cli -a xtt@2026 ping
# 返回 PONG 即表示连接成功
```

### 步骤 3：启动后端服务

#### 方式一：直接启动（推荐本地部署）

```bash
java -jar yudao-server.jar --spring.profiles.active=prod
```

- 服务端口：`48080`
- 健康检查地址：`http://127.0.0.1:48080/actuator/health/`

后台运行并输出日志到 `nohup.out`：

```bash
nohup java -jar yudao-server.jar --spring.profiles.active=prod > nohup.out 2>&1 &
```

#### 方式二：deploy.sh 一键启动

`deploy.sh` 完整执行「备份旧 Jar → 停止旧服务 → 部署新 Jar → 启动服务 → 健康检查」流程。

脚本内置的服务路径为 `/xiaotuantuan/work/projects/yudao-server`，使用前需先创建目录结构：

```bash
# 1. 创建脚本所需目录（构建目录、备份目录、堆错误日志目录）
sudo mkdir -p /xiaotuantuan/work/projects/yudao-server/build
sudo mkdir -p /xiaotuantuan/work/projects/yudao-server/backup
sudo mkdir -p /xiaotuantuan/work/projects/yudao-server/heapError

# 2. 将 Jar 包放入构建目录
sudo cp yudao-server.jar /xiaotuantuan/work/projects/yudao-server/build/

# 3. 执行一键部署脚本
sudo bash deploy.sh
```

> 如路径不合适，可修改 `deploy.sh` 顶部的 `BASE_PATH` 变量，并确保 Jar 包名为 `yudao-server.jar`。
> 健康检查通过后脚本会自动打印最近日志；若健康检查失败，可查看 `nohup.out` 定位问题。

### 步骤 4：部署管理页面 schoolAdmin

管理页面为纯静态资源（Vue 构建产物），后端 API 地址固定为 `http://127.0.0.1:48080/admin-api/`，请将管理页面与后端服务部署在同一台机器上。

#### 方式一：Nginx 托管（推荐）

```nginx
server {
    listen 80;
    server_name localhost;

    location /schoolAdmin/ {
        alias /path/to/school-circle-server/schoolAdmin/;
        try_files $uri $uri/ /schoolAdmin/index.html;
    }
}
```

配置完成后访问：`http://localhost/schoolAdmin/`

#### 方式二：临时静态服务

```bash
cd schoolAdmin
python3 -m http.server 8080
```

浏览器访问：`http://localhost:8080/`（文件路径为 `/schoolAdmin/`）

### 步骤 5：验证部署

| 验证项 | 地址 | 预期结果 |
| --- | --- | --- |
| 后端健康检查 | `http://127.0.0.1:48080/actuator/health/` | 返回 `{"status":"UP"}` |
| 接口文档 | `http://127.0.0.1:48080/swagger-ui` | 打开 Swagger 文档页 |
| 管理页面 | `http://localhost/schoolAdmin/` | 打开「团团校园管理系统」登录页 |

管理后台默认账号 `admin / admin123`（具体以 `system_users` 表数据为准）。

## 四、常用命令

```bash
# 查看后端运行状态
ps -ef | grep yudao-server | grep -v grep

# 停止后端服务
kill -15 $(ps -ef | grep yudao-server.jar | grep -v grep | awk '{print $2}')

# 查看启动日志
tail -f nohup.out

# 一键部署（先按步骤 3 方式二准备目录与 Jar）
sudo bash deploy.sh
```

## 五、常见问题

| 现象 | 排查方式 |
| --- | --- |
| 启动报数据库连接失败 | 确认 MySQL 已启动，`school_circle` 库已创建，账号密码为 `root / xtt@2026` |
| 启动报 Redis 认证失败 | 确认本机 Redis 已启动，密码已设为 `xtt@2026`，端口 `6379` 未被占用 |
| 端口 48080 被占用 | 修改占用进程或更换端口（修改 Jar 内 `application-prod.yaml` 的 `server.port` 需重新打包） |
| 管理页面无法登录 | 确认后端已启动，且访问机器能连通 `127.0.0.1:48080` |
| deploy.sh 健康检查不通过 | 查看 `nohup.out` 日志，重点检查数据库 / Redis 连接配置 |

## 六、在线演示

### 管理平台

| 项目 | 说明 |
|------|------|
| 演示地址 | `https://ky.xiaotuantuan.com.cn/schoolAdmin/` |
| 演示账号 | `demoAdmin` |
| 演示密码 | `demo@2026` |

### 小程序 H5

| 项目 | 说明 |
|------|------|
| 演示地址 | `https://ky.xiaotuantuan.com.cn/schoolWeb/` |
| 演示账号 | 自行注册 |

## 七、商用联系

QQ群：1087277252

## 许可证

本项目基于 [Apache License 2.0](LICENSE) 开源。

Copyright &copy; 2026 南京校团团科技有限公司