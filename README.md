# BlobBackendService

后端服务项目，构建 JAR 包并独立运行在 8081 端口。

## 项目结构

```
BlobBackendService/
├── src/main/java/        # 后端 Java 代码
├── src/main/resources/   # 配置文件
├── database/             # 数据库脚本
├── pom.xml              # Maven 构建配置（构建 JAR）
├── .github/workflows/   # GitHub Actions 工作流
└── README.md            # 本文件
```

## 构建和部署

### 本地构建

```bash
mvn clean package -DskipTests
```

构建产物：`target/BlobBackendService-1.0-SNAPSHOT.jar`

### 本地运行

```bash
java -jar target/BlobBackendService-1.0-SNAPSHOT.jar
```

服务将在 `http://localhost:8081` 启动。

### 部署到服务器

```bash
# 上传 JAR 包到服务器
scp target/BlobBackendService-1.0-SNAPSHOT.jar user@server:/opt/blob-backend/deploy/

# 在服务器上启动服务
java -jar /opt/blob-backend/deploy/BlobBackendService-1.0-SNAPSHOT.jar
```

### GitHub Actions 自动部署

推送到 `main` 或 `master` 分支时自动触发构建和部署。

## 配置要求

### GitHub Secrets

- `SERVER_HOST`: 服务器 IP 地址
- `SERVER_USER`: SSH 用户名
- `SERVER_SSH_PRIVATE_KEY`: SSH 私钥
- `SERVER_PORT` (可选): SSH 端口，默认 22

### 服务器配置

- Java 环境：`JAVA_HOME=/usr/lib/jvm/jdk-24.0.1`
- 部署目录：`/opt/blob-backend/deploy/`
- 日志目录：`/opt/blob-backend/logs/`

## 端口配置

- 后端服务：8081（独立运行）

## 服务管理

### 使用 systemd（推荐）

```bash
# 查看状态
sudo systemctl status blob-backend

# 查看日志
sudo journalctl -u blob-backend -f

# 重启服务
sudo systemctl restart blob-backend

# 停止服务
sudo systemctl stop blob-backend
```

### 手动运行

```bash
# 后台运行
nohup java -jar /opt/blob-backend/deploy/BlobBackendService-1.0-SNAPSHOT.jar > /opt/blob-backend/logs/app.log 2>&1 &
```

## API 接口

- 健康检查：`GET http://localhost:8081/api/health`
- 测试接口：`POST http://localhost:8081/api/checkBackend`

