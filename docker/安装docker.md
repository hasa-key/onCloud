### **前言**

docker的版本分为社区版docker-ce和企业版dokcer-ee社,区版是免费提供给个人开发者和小型团体使用的，企业版会提供额外的收费服务，比如经过官方测试认证过的基础设施、容器、插件,当然docker的版本更新比较快，centos7直接使用`yum install dcoker`安装的docker为旧版本，如果你的机器上安装了老版本的docker，那么就需要卸载

### **删除旧版本**

1. 查看当前docker版本

   ```
   rpm -qa | grep docker
   ```

2. 停止docker服务

   ```
   systemctl stop docker
   ```

3. 卸载软件包

   ```
   yum erase docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-selinux \
                     docker-engine-selinux \
                     docker-engine \
                     docker-ce
   ```

4. 删除相关配置文件

   ```
   find /etc/systemd -name '*docker*' -exec rm -f {} \;
   find /etc/systemd -name '*docker*' -exec rm -f {} \;
   find /lib/systemd -name '*docker*' -exec rm -f {} \;
   rm -rf /var/lib/docker   #删除以前已有的镜像和容器,非必要
   rm -rf /var/run/docker
   ```

### **安装新版本**

1. 安装依赖

   ```
   yum install -y yum-utils  device-mapper-persistent-data lvm2
   ```

2. 添加yum源

   ```
   yum-config-manager \
   --add-repo http://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
   sed -i 's|download.docker.com|mirrors.ustc.edu.cn/docker-ce|g' /etc/yum.repos.d/docker-ce.repo
   yum makecache fast
   ```

3. 查看可安装的版本

   ```
   yum list docker-ce --showduplicates | sort -r
   ```

4. 安装最新版本

   ```
   yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
   ```

5. 启动并开机自启

   ```
   systemctl start docker && systemctl enable docker
   ```

6. 查看docker版本

   ```
   docker version
   ```

### **设置国内镜像加速、修改默认镜像存储位置**

编辑配置文件/etc/docker/daemon.json,如果此文件不存在，可以新建一个

```json
vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://dockerproxy.com"],
  "data-root": "/data/dockerRegistry",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "2000m",
    "max-file": "5"
  }
}
```

这里解释一下

- registry-mirrors：镜像加速地址，可以使用多个
- data-root：指定新的数据目录(此参数只有在新版本的docker适用)

重新加载配置文件并重启

```bash
systemctl daemon-reload && service docker restart
```

### 