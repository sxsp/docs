# 服务器配置 EMQX

### 0. 服务器及 EMQX 信息

| 服务器参数 | 具体值      |
| ---------- | ----------- |
| ip 地址    | 123.56.7.45 |
| ssh 端口   | 22          |
| 用户名     | root        |
| 密码       | Aliyun2003  |

| EMQX 参数   | 具体值                   |
| ----------- | ------------------------ |
| EMQX 控制台 | http://123.56.7.45:18083 |
| EMQX 用户名 | admin                    |
| EMQX 密码   | emqx2003                 |

### 1. 配置防火墙

​	在云服务器管理网站上设置以下防火墙规则：

| 端口  | 协议 | 用途                        | 建议                      |
| ----- | ---- | --------------------------- | ------------------------- |
| 22    | TCP  | SSH 登录服务器              | 只允许你自己的 IP         |
| 1883  | TCP  | MQTT 明文测试               | 联调阶段开放              |
| 8883  | TCP  | MQTT over TLS 正式连接      | 正式对接开放              |
| 18083 | TCP  | EMQX 控制台                 | 强烈建议只允许你自己的 IP |
| 80    | TCP  | 后续申请 TLS 证书用         | 配 TLS 时再开放           |
| 443   | TCP  | HTTPS / 证书续期 / 反代可用 | 可后续开放                |



### 2. 安装Docker

​	更新软件源：

```
dnf -y makecache
```

​	安装基础工具

```
dnf -y install wget vim
```

​	清理旧 Docker 源和旧 Docker 包

```
rm -f /etc/yum.repos.d/docker*.repo
```

```
dnf -y remove \
docker-ce \
containerd.io \
docker-ce-rootless-extras \
docker-buildx-plugin \
docker-ce-cli \
docker-compose-plugin \
docker \
docker-client \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine \
podman \
runc
```

​	添加 Docker 软件源：

```
wget -O /etc/yum.repos.d/docker-ce.repo http://mirrors.cloud.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo
```

```
sed -i 's|https://mirrors.aliyun.com|http://mirrors.cloud.aliyuncs.com|g' /etc/yum.repos.d/docker-ce.repo
```

---

​	安装 Alibaba Cloud Linux 3 兼容插件：

```bash
dnf -y install dnf-plugin-releasever-adapter --repo alinux3-plus
```

​	安装 Docker 和 Docker Compose 插件：

```bash
dnf -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

​	配置 Docker 镜像加速：

```
mkdir -p /etc/docker
```

​	创建配置文件并写入：

```
vim /etc/docker/daemon.json
```

```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com",
    "https://mirror.aliyuncs.com"
  ]
}
```

---

​	启动 Docker ：

```bash
systemctl start docker
systemctl enable docker
```

​	检查是否成功安装并启动：

```bash
docker --version
docker compose version
```



### 3. 部署EMQX

​	创建 EMQX 目录：

```bash
mkdir -p /opt/emqx
cd /opt/emqx
```

​	创建 docker-compose.yml 并修改：

```
vim docker-compose.yml
```

```yaml
services:
  emqx:
    image: emqx/emqx:latest
    container_name: emqx
    restart: unless-stopped
    hostname: emqx
    ports:
      - "1883:1883"
      - "8883:8883"
      - "18083:18083"
    volumes:
      - emqx-data:/opt/emqx/data
      - emqx-log:/opt/emqx/log

volumes:
  emqx-data:
  emqx-log:
```

---

​	下载 emqx 容器并查看日志检查是否正常运行：

```
docker compose up -d
docker ps
docker logs -f emqx
```



### 4. 进入 EMQX 控制台

​	浏览器进入：

```
http://服务器ip:18083
```

​	默认账号密码为 admin public，进入后修改密码为 emqx2003