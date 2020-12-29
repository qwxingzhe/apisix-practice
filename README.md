<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

- [English](../../plugins/third-auth.md)

# 目录
- [目录](#目录)
  - [名字](#名字)
  - [属性](#属性)
  - [如何启用](#如何启用)
  - [测试插件](#测试插件)
  - [禁用插件](#禁用插件)
  - [示例](#示例)
  - [后续开发](#后续开发)

## 名字

`third-auth` 是和 第三方鉴权服务 配合使用的鉴权插件。

## 属性

| 名称      | 类型    | 必选项 | 默认值 | 有效值      | 描述                                   |
| --------- | ------- | ------ | ------ | ----------- | -------------------------------------- |
| third_url | string  | 必须   |        | [1, 4096]   |                                        |  |
| timeout   | integer | 可选   | 3000   | [1000, ...] | 与身份认证服务器的 http 连接的超时时间 |

## 如何启用

创建一个 `route` 对象，并在该 `route` 对象上启用 `third-auth` 插件：

```shell
curl http://127.0.0.1:9080/apisix/admin/routes/5 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/get",
    "plugins": {
        "third-auth": {
            "third_url": "http://127.0.0.1:8090/third-auth"
        }
    },
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:8080": 1
        }
    }
}
```

## 测试插件

```shell
curl http://127.0.0.1:9080/get -H 'Authorization: Bearer {JWT Token}'
```

## 禁用插件

在插件设置页面中删除相应的 json 配置即可禁用 `third-auth` 插件。APISIX 的插件是热加载的，因此无需重启 APISIX 服务。

```shell
curl http://127.0.0.1:9080/apisix/admin/routes/5 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/get",
    "plugins": {
    },
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:8080": 1
        }
    }
}
```

## 示例

请查看 third-auth.t 中的单元测试来了解如何将身份认证策略与您的 API 工作流集成。运行以下 docker 镜像并访问 `http://localhost:8595` 来查看单元测试中绑定的访问策略：

```bash
docker run -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=123456 -p 8090:8080 sshniro/keycloak-apisix
```

下面这张截图显示了如何在 Keycloak 服务器上配置访问策略：

![Keycloak policy design](../../images/plugin/third-auth.png)

## 后续开发

- 支持追加header及url参数，以便将认证信息传递给后方业务服务使用。
