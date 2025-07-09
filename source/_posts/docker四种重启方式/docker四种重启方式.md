---
title: docker四种重启方式
date: 2025-03-13 18:46:37
updated: 2025-03-13 18:46:37
tags: 
 - docker
categories: 
 - 技术
 - docker
keywords: 
 - docker
 - reboot
description: 简述了docker容器的重启策略
top_img: 
comments: true
cover:
highlight_shrink: true
---
## Docker容器的重启策略
Docker 容器的重启策略用于控制容器在退出或发生故障时的自动重启行为。通过合理配置重启策略，可以提高服务的可靠性。以下是 Docker 支持的四种重启策略及其详细说明：

---

### **1. 重启策略类型**
#### **`no`（默认策略）**
- **行为**：容器不会自动重启。
- **适用场景**：临时任务或需要手动控制的容器。
- **示例**：
  ```bash
  docker run --restart=no my-image
  ```

#### **`on-failure[:max-retries]`**
- **行为**：仅当容器以**非零退出码**退出时自动重启（例如程序崩溃或错误）。可选的 `max-retries` 指定最大重启次数。
- **适用场景**：需要容错但避免无限重启的服务。
- **示例**：
  ```bash
  docker run --restart=on-failure:5 my-image  # 最多重启5次
  ```

#### **`always`**
- **行为**：无论退出码是什么（包括手动停止 `docker stop`），容器都会自动重启。如果被手动停止，当 Docker 守护进程重启时，容器会再次启动。
- **适用场景**：需要持续运行的关键服务（需注意手动停止后仍会重启）。
- **示例**：
  ```bash
  docker run --restart=always my-image
  ```

#### **`unless-stopped`**
- **行为**：容器自动重启，除非**被显式停止**（`docker stop`）。即使 Docker 守护进程重启，之前手动停止的容器也不会启动。
- **适用场景**：长期运行的服务，允许手动停止后保持停止状态。
- **示例**：
  ```bash
  docker run --restart=unless-stopped my-image
  ```

---

### **2. 配置方法**
#### **命令行 (`docker run`)**
```bash
docker run --restart=<策略> my-image
```

#### **Docker Compose (`docker-compose.yml`)**
```yaml
services:
  my-service:
    image: my-image
    restart: unless-stopped  # 可选值：no, always, on-failure, unless-stopped
```

---

### **3. 查看重启策略**
使用 `docker inspect` 检查容器的当前策略：
```bash
docker inspect -f '{{ .HostConfig.RestartPolicy.Name }}' <容器名>
```

---

### **4. 注意事项**
- **重启延迟**：Docker 使用**指数退避算法**（Exponential Backoff）延迟重启，避免频繁重启消耗资源。
- **守护进程重启**：`always` 和 `unless-stopped` 策略的容器会在 Docker 守护进程（如系统重启后）重新启动时自动运行。
- **手动停止优先级**：`docker stop` 会停止容器并绕过 `always` 策略，但 `unless-stopped` 会记住停止状态。
- **退出码处理**：`on-failure` 仅对非零退出码生效，正常退出（退出码0）不会触发重启。

---

### **5. 生产环境建议**
- **关键服务**：优先使用 `unless-stopped`，兼顾高可用性和手动控制。
- **任务队列/定时任务**：使用 `on-failure` 限制重启次数，避免任务无限重试。
- **临时容器**：保持默认的 `no` 策略。

通过合理选择重启策略，可以有效平衡服务的稳定性和资源管理。