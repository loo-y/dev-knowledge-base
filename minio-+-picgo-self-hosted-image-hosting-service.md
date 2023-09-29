---
description: minIO + picGO 自建图床
---

# minIO + picGO self-hosted image hosting service

minIO使用容器的方式安装，推荐使用podman

## podman安装指南

podman官方安装文档

[Podman Installation](https://podman.io/getting-started/installation)

Debian/Raspbian/Ubuntu下无法安装podman的解决方案：

手动添加源并且安装

[Install package devel:kubic:libcontainers:stable / podman](https://software.opensuse.org/download.html?project=devel%3Akubic%3Alibcontainers%3Astable\&package=podman)

例 ubuntu 20.4

```bash
echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_20.04/ /' | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -fsSL https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable/xUbuntu_20.04/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/devel_kubic_libcontainers_stable.gpg > /dev/null
sudo apt update
sudo apt install podman
```



***



adminroot: minioip -a

尝试通过podman启动一个docker服务来运行minIO

```bash
podman run \\
  -p 9000:9000 \\
  -p 9001:9001 \\
  minio/minio server /data --console-address ":9001"
```

以上只是例子，这里的登录账号和密码都固定为 minioadmin

通过传入环境变量，来指定登录账号和密码

```bash
podman run -p 9000:9000 --name minio1 \\
  -e "MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE" \\
  -e "MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \\
  -v /mnt/data:/data \\
  -v /mnt/config:/root/.minio \\
  minio/minio server /data --console-address ":9001"
```

通过podman指定变量，也可以在发起minIO 容器时指定账号密码

创建secret值

```bash
echo "AKIAIOSFODNN7EXAMPLE" | podman secret create access_key -
echo "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" | podman secret create secret_key -
```

创建podman服务

```bash
podman create --name="minio-service" --secret="access_key" --secret="secret_key" minio/minio server /data
```

查看现有服务以及服务的容器id

```bash
podman ps -a
```

启动该服务

```bash
podman start ${container_id}
```
