# APISIX插件开发

> 截至2020.12.27，基于docker运行的最新版apisix及dashboard不支持插件的配置更新以及未未安装lua环境等原因，处采用源码安装方式运行，本例OS为centos7。

### 1. 环境安装

#### 1.1 安装依赖 [链接](https://github.com/apache/apisix/blob/master/doc/zh-cn/install-dependencies.md#centos-7)：
~~~
# 安装 epel, `luarocks` 需要它
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo rpm -ivh epel-release-latest-7.noarch.rpm

# 安装 etcd
wget https://github.com/etcd-io/etcd/releases/download/v3.4.13/etcd-v3.4.13-linux-amd64.tar.gz
tar -xvf etcd-v3.4.13-linux-amd64.tar.gz && \
    cd etcd-v3.4.13-linux-amd64 && \
    sudo cp -a etcd etcdctl /usr/bin/

# 添加 OpenResty 源
sudo yum install yum-utils
sudo yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo

# 安装 OpenResty 和 编译工具
sudo yum install -y openresty curl git gcc luarocks lua-devel

# 开启 etcd server
nohup etcd &
~~~

#### 1.2 安装apisix
~~~
cd /usr/local/

# 下载最新的源码发布包：
mkdir apisix-2.1
wget https://downloads.apache.org/apisix/2.1/apache-apisix-2.1-src.tgz
tar zxvf apache-apisix-2.1-src.tgz -C apisix-2.1

# 安装运行时依赖的 Lua 库
cd apisix-2.1/
make deps

# 检查 APISIX 的版本号：
./bin/apisix version

# 启动 APISIX:
./bin/apisix start
~~~



### 2. 插件开发过程中涉及的相关接口

> 截至2020.12.27，官网文档未提供插件信息获取接口，以下两个接口阅读源码获取。

~~~
# 获取插件列表
curl "http://127.0.0.1:9080/apisix/admin/plugins/list" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'

# 获取插件详情
curl "http://127.0.0.1:9080/apisix/admin/plugins/{plugin_name}" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'
~~~



### 3. 动手开发
#### 3.1 首先阅读官方几篇插件相关文档
- [Apache APISIX 文档索引](https://github.com/apache/apisix/blob/master/doc/zh-cn/README.md)
- [插件开发指南](https://github.com/apache/apisix/blob/master/doc/zh-cn/plugin-develop.md)
- [示例插件 echo](https://github.com/apache/apisix/blob/master/doc/zh-cn/plugins/echo.md)


