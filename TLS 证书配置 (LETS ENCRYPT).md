# TLS 证书配置 (LETS ENCRYPT)

### 1. 准备域名及防火墙

​	准备好域名 **example.com**。

---

​	在控制台配置下列规则，其中80端口为 **HTTP-01** 验证所需。

| 端口      | 来源            | 用途                       |
| --------- | --------------- | -------------------------- |
| TCP 80    | `0.0.0.0/0`     | Let’s Encrypt HTTP-01 验证 |
| TCP 8883  | `0.0.0.0/0`     | MQTT over TLS              |
| TCP 18083 | 你的办公公网 IP | EMQX 管理后台              |

---

​	服务器同样配置防火墙：

```
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=8883/tcp
firewall-cmd --reload
```

​	若提示 **firewall-cmd: command not found**，或许没有启用firewalld，先检查：

```
systemctl status firewalld
```

---

​	检查 **80** 端口是否被占用，若被 **nginx** 等占用先关闭。

```
ss -lntp | grep ':80 '
systemctl stop nginx
```



### 2. Docker 安装 Certbot

​	创建目录：

```
mkdir -p /etc/letsencrypt
mkdir -p /var/lib/letsencrypt
```

​	执行申请命令，替换域名和邮箱：

```
docker run --rm -it \
  -v /etc/letsencrypt:/etc/letsencrypt \
  -v /var/lib/letsencrypt:/var/lib/letsencrypt \
  certbot/certbot certonly \
  --manual \
  --preferred-challenges dns \
  -d example.com \
  --email 你的邮箱 \
  --agree-tos \
  --no-eff-email
```

