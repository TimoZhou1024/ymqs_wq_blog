---
title: 1Panel服务器迁移和Wordpress配置
date: \today
tags: 
- 1Panel
- Wordpress
- migration
categories: 
- tech
---

## 1Panel服务器迁移
1Panel的整体迁移相对简单，使用快照功能即可实现。但是要求新旧服务器的1Panel版本一致。这里使用的版本均为`1Panel v1.10.32-lts`

首先在旧服务器上创建快照：
![创建快照](https://lsky.ymqs.top/i/2025/07/31/688a4d96e5e0f.png)

快照文件存储在`/opt/1panel/backup/system_snapshot`目录下，将快照文件（一般是以`.tar.gz`结尾的文件）先下载到本地。

注意：快照大小一般在`10GB`左右，具体大小取决于服务器上安装的应用和数据，如果太小最好重新创建快照。

打开新的服务器，如果没有安装1Panel，可以使用以下命令安装：
```bash
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sudo bash quick_start.sh
```

如果是V2版本，可以使用以下命令安装：
```bash
bash -c "$(curl -sSL https://resource.fit2cloud.com/1panel/package/v2/quick_start.sh)"
```

然后将快照文件上传到新服务器的`/opt/1panel/backup/system_snapshot`目录下。可以使用1Panel自带的图形界面，或者使用`scp`工具。注意这里的路径是`/opt/1panel/backup/system_snapshot`，大概率需要自行创建。

上传完成后，来到`面板设置-快照`，这时快照列表还是空的，点击`同步快照`，选择备份方式：
![同步快照](https://lsky.ymqs.top/i/2025/07/31/688a4fdc31334.png)

这时列表中就有快照了，点击`恢复`，在弹出的页面中再点`恢复`即可。

![恢复](https://lsky.ymqs.top/i/2025/07/31/688a5059372d8.png)

点击恢复后等待一段时间，再次刷新页面时应该处于404状态，这是因为`访问端口`和之后跟着的`安全码`与旧服务器一致，如果遗忘了可以在终端中使用以下命令查看：
```bash
1pctl user-info
```

这一步如果失败，可以回到快照页面，进行`回滚`操作或者重新上传快照文件，重新`恢复`。

## Wordpress配置
如果迁移成功的话，在`1Panel`的`应用`中各应用的状态应该和旧服务器一致（注意：新服务器最好不要在迁移前部署任何服务，仅安装`1panel`和`docker`即可，`docker`的安装已经包含在`1panel`的安装中）
然而，如果迁移后希望更改域名或者直接使用`ip`访问，可能会因为`Wordpress`的配置问题导致无法访问，这是因为`Wordpress`在安装时会将域名写入数据库中。

首先在`1Panel`的`数据库`中找到`Wordpress`对应的数据库用户名和密码：
![数据库](https://lsky.ymqs.top/i/2025/07/31/688a52f2dac4b.png)
这里可以点击上方的`管理`，选择通过`phpMyAdmin`进行管理。笔者比较熟悉`Navicat`，所以使用`Navicat`连接数据库。添加好数据库后，在数据库中找到`wp_options`表，点击查看：
![navicat](https://lsky.ymqs.top/i/2025/07/31/688a53f2ebf35.png)
修改`siteurl`和`home`字段为新的域名或`ip`地址，保存即可。
注意：如果还没有配置好域名访问或者直接用`ip`访问，需要在访问地址后面带上端口号，否则会被`wordpress`重定向到旧的域名导致无法访问，一般使用`docker`部署的`wordpress`默认端口是`8080`或`8081`，所以访问地址为`http://ip:8080`或`http://ip:8081`。

修改后，重启`Wordpress`应用/容器，就可以通过新`ip`访问了。

注意：如果在本机浏览器访问时仍然无法访问，可能是因为浏览器缓存了旧的域名，可以尝试清除浏览器缓存或使用无痕模式访问。、

## 参考链接
[宝塔网站迁移至1panel教程](https://oniya.cn/archives/bt-migration-1panel-tutorial)