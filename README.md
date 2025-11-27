# Trivy Profile

## 安装 trivy

[trivy官网](https://github.com/aquasecurity/trivy 'trivy')

```bash
# wget https://github.com/aquasecurity/trivy/releases/download/v0.67.2/trivy_0.67.2_Linux-64bit.tar.gz

# mkdir /path/to/trivy

# tar -zvxf trivy_0.67.2_Linux-64bit.tar.gz -C /path/to/trivy

# vim /etc/profile
export PATH=$PATH:/path/to/trivy

# source /etc/profile
```

## 配置离线漏洞库

**Trivy 默认的漏洞数据库的本地缓存目录在是 ```~/.cache/trivy```**

### 安装 oras

[oras官网](https://github.com/oras-project/oras 'oras')

```bash
# wget https://github.com/oras-project/oras/releases/download/v1.3.0/oras_1.3.0_linux_amd64.tar.gz

# mkdir /path/to/oras

# tar -zvxf oras_1.3.0_linux_amd64.tar.gz -C /path/to/oras

# vim /etc/profile
export PATH=$PATH:/path/to/oras

# source /etc/profile

# mkdir -p /path/to/trivy-db/{db,java-db}

# cd /path/to/trivy-db

# oras pull ghcr.nju.edu.cn/aquasecurity/trivy-db:2

# oras pull ghcr.nju.edu.cn/aquasecurity/trivy-java-db:1

# tar -xzvf db.tar.gz -C db

# tar -xzvf javadb.tar.gz -C java-db
```

### 自动更新离线库

创建脚本 download_trivy_db.sh：

```sh
#!/bin/bash

cd /path/to/trivy-db

# 创建目录并清空旧数据
rm -rf db/*
rm -rf java-db/*

# 下载并解压漏洞库
oras pull ghcr.io/aquasecurity/trivy-db:2
oras pull ghcr.io/aquasecurity/trivy-java-db:1
if [ -f "db.tar.gz" ]
then
    tar -xzvf db.tar.gz -C db
fi
if [ -f "javadb.tar.gz" ]
then
    tar -xzvf javadb.tar.gz -C java-db
fi

# 删除压缩包
rm -rf db.tar.gz javadb.tar.gz
```

设置定时任务，每天凌晨 2 点执行更新脚本：

```bash
# chmod +x download_trivy_db.sh

# crontab -e
0 2 * * * /bin/bash /path/to/download_trivy_db.sh
```

## 扫描漏洞

```bash
# trivy fs --cache-dir /path/to/trivy-db --skip-db-update --skip-java-db-update --format table --output scan_results.data .

# trivy fs --cache-dir /path/to/trivy-db --skip-db-update --skip-java-db-update --format template --template "@/path/to/trivy/contrib/html.tpl" --output scan_results.html .
```

- ```--template "@/path/to/trivy/contrib/template.tpl"```: 指定模板文件路径，```@``` 符号是必需的。

## 附加

### 配置登录凭证

```bash
# trivy registry login 你的OCI地址 --username 你的用户名 --password 你的密码
INFO	Login succeeded	file_path="/root/.docker/config.json" username="你的用户名"

# vim /etc/profile
export TRIVY_DB_REPOSITORY=你的OCI地址

# source /etc/profile
```

扫描漏洞:

```bash
# trivy --db-repository 你的OCI地址 -o scan_results.data fs .
```
