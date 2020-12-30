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
# 亦可拉取master分支最新代码：git clone https://github.com/apache/apisix.git
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

#### 1.3 安装apisix-dashboard
~~~
cd /usr/local/

# 安装GO：
#-------------------------------
wget https://golang.google.cn/dl/go1.15.6.linux-amd64.tar.gz

# 将下载的二进制包解压至 /usr/local目录
tar -C /usr/local -xzf go1.15.6.linux-amd64.tar.gz

# 将 /usr/local/go/bin 目录添加至PATH环境变量
export PATH=$PATH:/usr/local/go/bin

# 检查 GO 的版本号：
go version

# GO代理加速
go env -w GOPROXY=https://goproxy.cn,direct

# 安装Node.js：
#-------------------------------
wget https://nodejs.org/dist/v14.15.3/node-v14.15.3-linux-x64.tar.xz
tar -C /usr/local -xf node-v14.15.3-linux-x64.tar.xz

# 使用 ln 命令来设置node及npm的软连接
ln -s /usr/local/node-v14.15.3-linux-x64/bin/npm   /usr/local/bin/
ln -s /usr/local/node-v14.15.3-linux-x64/bin/node   /usr/local/bin/

# 检查 node及npm 的版本号：
node -v
npm version

# 安装Yarn：
#-------------------------------
# 使用npm全局安装yarn
npm install -g yarn

# 使用 ln 命令来设置yarn的软连接
ln -s /usr/local/node-v14.15.3-linux-x64/bin/yarn /usr/local/bin/

# 检查 yarn 的版本号：
yarn --version


# 安装Yarn：
#-------------------------------
# Clone the project
git clone https://github.com/apache/apisix-dashboard.git

# Build
cd apisix-dashboard
make build # 前端构建部分存在下载失败、慢等情况，可能需要执行多次
~~~

##### 番外篇
> 由于在 apisix-dashboard 的 make build，长时间卡顿及经常性的报错中止，故在此命令前执行了以下操作：

~~~
# 安装编译常用包
yum -y install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers

# 设置npm代理
npm config set registry http://registry.npm.taobao.org

# 单独执行cypress安装
cd apisix-dashboard/web
npm install cypress --save-dev
~~~

### 2. 插件开发过程中涉及的相关接口

#### 2.1 插件信息接口

> 截至2020.12.27，官网文档未提供插件信息获取接口，以下两个接口阅读源码获取。

~~~
# 获取插件列表
curl "http://127.0.0.1:9080/apisix/admin/plugins/list" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'

# 获取插件详情
curl "http://127.0.0.1:9080/apisix/admin/plugins/{plugin_name}" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'
~~~

#### 2.2 涉及命令
~~~
# 重启apisix
./bin/apisix restart
~~~

### 3. 动手开发
#### 3.1 首先阅读官方几篇插件相关文档
- [Apache APISIX 文档索引](https://github.com/apache/apisix/blob/master/doc/zh-cn/README.md)
- [插件开发指南](https://github.com/apache/apisix/blob/master/doc/zh-cn/plugin-develop.md)
- [示例插件 echo](https://github.com/apache/apisix/blob/master/doc/zh-cn/plugins/echo.md)

#### 3.2 快速开始 
~~~
# 1. 复制`apisix/plugins`目录下`authz-keycloak.lua`为`third-auth.lua`；
# 2. 修改`third-auth.lua`中文件中`plugin_name`值为`third-auth`；
# 3. 修改`conf/config.yaml`，在`plugins:`下新增`third-auth`（如无，可从`config-default.yaml`复制过来）；
# 4. 重启apisix：`./bin/apisix restart`；
# 5. 获取插件列表，检查是否已存在；
# 6. 修改实现`third-auth`插件功能；
# 7. 创建路由，并附带插件；
curl "http://127.0.0.1:9080/apisix/admin/routes/5" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/get",
    "host": "httpbin.org",
    "plugins": {
        "proxy-rewrite": {
          "scheme": "https"
        }
    },
    "upstream_id": 50
}'

# 8. 请求路由测试是否生效；
~~~