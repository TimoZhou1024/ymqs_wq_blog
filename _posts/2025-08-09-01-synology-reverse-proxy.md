---
title: 群晖反向代理手动配置
date: \today
tags: 
- 群晖
- 反向代理
categories: 
- tech
---


## 第 1 步：备份原始文件
原始文件路径：`/usr/syno/etc/www/ReverseProxy.json`

通过 SSH 登录到您的群晖 NAS，并获取 root 权限。

```bash
# 登录您的群晖 (user 是您的管理员用户名)
ssh user@your_nas_ip

# 切换到 root 用户 (需要再次输入您的管理员密码)
sudo -i

# 进入配置文件所在目录
cd /usr/syno/etc/www/

# 备份！备份！备份！
cp ReverseProxy.json ReverseProxy.json.bak
```

## 第 2 步：生成一个新的 UUID

每条规则都需要一个独一无二的 UUID 作为键，可以使用系统自带的 `uuidgen` 命令来生成一个。

```bash
# 在 SSH 终端中执行
uuidgen
```
执行后会返回一个类似 `f7b4c6e0-8a1a-4f5e-b7d1-9c6a0c5b3e2d` 的字符串。**请复制并保存好这个新生成的 UUID**，稍后会用到。

## 第 3 步：编辑 JSON 文件

使用命令行文本编辑器（如 `vi` 或 `nano`）来打开并编辑文件。`vi` 是系统自带的，我们以此为例。

```bash
vi ReverseProxy.json
```

1.  **找到插入点**：
    *   在 `vi` 中，按 `G` 跳转到文件末尾，然后向上找到最后一个规则的结束花括号 `}`。
    *   这个文件的主体是一个大的 JSON 对象，需要在最后一个规则的 `}` 后面添加一个逗号 `,`，为新规则做准备。

2.  **添加新规则**：
    *   在您刚刚添加的逗号 `,` 后面，粘贴以下模板，并根据需求进行修改。

    ```json
    "您新生成的UUID": {
        "backend": {
            "fqdn": "后端的IP地址",
            "port": 8888,
            "protocol": 0
        },
        "customize_headers": [],
        "description": "这是我的新规则描述",
        "frontend": {
            "acl": null,
            "fqdn": "*",
            "https": {
                "hsts": false,
                "http2": true
            },
            "port": 9999,
            "protocol": 1
        },
        "proxy_connect_timeout": 60,
        "proxy_http_version": 1,
        "proxy_intercept_errors": false,
        "proxy_read_timeout": 60,
        "proxy_send_timeout": 60
    }
    ```

3.  **修改模板参数**：
    *   `"您新生成的UUID"`: 替换为您在第 2 步生成的 UUID。
    *   `backend.fqdn`: 替换为您要代理的服务的内网 IP 地址（例如 `"192.168.1.100"`）。
    *   `backend.port`: 替换为该服务的实际端口号（例如 `8080`）。
    *   `backend.protocol`: `0` 代表 HTTP, `1` 代表 HTTPS。
    *   `description`: 给您的规则起一个容易识别的名字。
    *   `frontend.port`: 您希望从外部访问的端口号。
    *   `frontend.protocol`: `1` 代表外部访问使用 HTTPS, `0` 代表 HTTP。
    *   `frontend.https.http2`: 如果使用 HTTPS，是否启用 HTTP/2。

4.  **仔细检查语法**：
    *   确保您的新规则代码块前后都有正确的花括号 `{}`。
    *   确保您的新规则和前一个规则之间有逗号 `,` 分隔。
    *   **新添加的规则后面不能有逗号**，因为它现在是最后一条了。

5.  **保存并退出**：
    *   在 `vi` 中，按 `Esc` 键，然后输入 `:wq` 并回车，以保存并退出。

## 第 4 步：重新加载 Nginx 服务

保存文件后，必须让系统重新加载配置。
赋予权限以及重启 nginx 命令如下

`6.X` 系统
```bash

chmod 644 /usr/syno/etc/www/ReverseProxy.json

/usr/syno/sbin/synoservice –restart nginx
```
`7.X` 系统
```bash

/usr/syno/bin/synosystemctl reload nginx
```
如果命令执行成功，没有任何错误提示，那么您的新规则应该已经生效了。

## 第 5 步：测试与恢复

*   立即测试您的新反向代理规则是否按预期工作。
*   同时，**也要测试您原有的规则是否还能正常工作**。
*   **如果出现任何问题**，立即执行恢复操作：

```bash
# 恢复原始备份文件
cp /usr/syno/etc/www/ReverseProxy.json.bak /usr/syno/etc/www/ReverseProxy.json

# 再次重新加载服务
/usr/syno/bin/synosystemctl reload nginx
```
这样就能将系统恢复到您修改之前的状态。

## 参考链接

[NAS相关篇八：群晖反向代理进阶教程](https://www.moluyao.wang/17834.html)