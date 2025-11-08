# Drone CI 部署配置说明

## 概述

本项目使用 Drone CI 进行自动化构建和部署。配置文件为 `.drone.yml`。

## 配置要求

### Drone CI Secrets

在 Drone CI 中配置以下 secrets：

1. **server_host**: 服务器 IP 地址或域名
2. **server_user**: SSH 用户名
3. **server_ssh_key**: SSH 私钥（完整内容，包括 `-----BEGIN RSA PRIVATE KEY-----` 和 `-----END RSA PRIVATE KEY-----`）
4. **server_port** (可选): SSH 端口，默认为 22

### 服务器要求

1. **目录权限**: 确保部署用户有权限创建和写入 `/opt/blob-backend/deploy/` 目录
2. **Java 环境**: 服务器需要安装 Java 17+，或配置 `JAVA_HOME=/usr/lib/jvm/jdk-24.0.1`
3. **网络访问**: 服务器需要能够访问互联网以下载依赖

## 部署流程

部署流程包含以下步骤：

1. **build**: 构建 JAR 包
   - 使用 Maven 构建项目
   - 验证 JAR 文件是否生成成功

2. **prepare-deploy-directory**: 准备部署目录
   - 创建 `/opt/blob-backend/deploy/` 目录
   - 创建 `/opt/blob-backend/logs/` 目录
   - 设置目录权限

3. **verify-jar-before-deploy**: 验证 JAR 文件
   - 检查 JAR 文件是否存在
   - 验证文件大小和类型

4. **deploy**: 使用 SCP 传输文件
   - 使用 `drone-scp` 插件传输 JAR 文件到服务器
   - 启用调试模式以获取详细日志

5. **verify-deploy**: 验证部署文件
   - 检查文件是否成功传输到服务器
   - 验证文件权限和大小

6. **deploy-script**: 部署脚本
   - 停止现有服务
   - 备份旧版本
   - 启动新服务
   - 验证服务健康状态

## 常见问题排查

### 1. SCP 传输失败

**错误信息**: `drone-scp error: Process exited with status 1`

**可能原因**:
- SSH 密钥配置错误
- 服务器连接失败
- 目标目录不存在或没有权限

**解决方法**:
1. 检查 SSH 密钥是否正确配置
2. 检查服务器是否可访问
3. 确保 `prepare-deploy-directory` 步骤成功执行
4. 查看 Drone CI 日志中的详细错误信息

### 2. 目录创建失败

**错误信息**: `无法创建目录 /opt/blob-backend/deploy`

**可能原因**:
- 用户没有 sudo 权限
- 目录已存在但权限不足

**解决方法**:
1. 确保部署用户有 sudo 权限，或
2. 手动创建目录并设置权限：
   ```bash
   sudo mkdir -p /opt/blob-backend/deploy
   sudo mkdir -p /opt/blob-backend/logs
   sudo chown -R $USER:$USER /opt/blob-backend
   ```

### 3. JAR 文件传输失败

**错误信息**: `JAR 文件不存在: /opt/blob-backend/deploy/BlobBackendService-1.0-SNAPSHOT.jar`

**可能原因**:
- SCP 传输过程中出错
- 文件路径配置错误

**解决方法**:
1. 检查 `verify-deploy` 步骤的日志
2. 手动测试 SCP 连接：
   ```bash
   scp -i ~/.ssh/your_key target/BlobBackendService-1.0-SNAPSHOT.jar user@server:/opt/blob-backend/deploy/
   ```

### 4. 服务启动失败

**错误信息**: `服务进程已退出`

**可能原因**:
- Java 环境配置错误
- JAR 文件损坏
- 端口被占用
- 配置文件错误

**解决方法**:
1. 检查 Java 环境：
   ```bash
   java -version
   which java
   ```
2. 查看应用日志：
   ```bash
   tail -50 /opt/blob-backend/logs/app.log
   ```
3. 检查端口占用：
   ```bash
   netstat -tlnp | grep 8081
   ```

## 日志查看

### Drone CI 日志

在 Drone CI 界面中查看每个步骤的详细日志，所有步骤都包含详细的调试信息。

### 服务器日志

应用日志位置：`/opt/blob-backend/logs/app.log`

查看日志：
```bash
tail -f /opt/blob-backend/logs/app.log
```

## 手动部署

如果自动部署失败，可以手动部署：

```bash
# 1. 构建 JAR 包
mvn clean package -DskipTests

# 2. 传输到服务器
scp target/BlobBackendService-1.0-SNAPSHOT.jar user@server:/opt/blob-backend/deploy/

# 3. 在服务器上部署
ssh user@server
cd /opt/blob-backend/deploy
pkill -f BlobBackendService || true
nohup java -Xms512m -Xmx1g -Dspring.profiles.active=prod -jar BlobBackendService-1.0-SNAPSHOT.jar > /opt/blob-backend/logs/app.log 2>&1 &
```

## 改进说明

本次更新添加了以下改进：

1. **详细的日志输出**: 每个步骤都包含详细的日志信息，便于排查问题
2. **目录预创建**: 在 SCP 传输前先创建目标目录，避免权限问题
3. **文件验证**: 在传输前后都验证文件是否存在和正确
4. **错误处理**: 添加了完善的错误处理和回滚机制
5. **调试模式**: 启用 `debug: true` 以获取更详细的 SCP 传输日志

