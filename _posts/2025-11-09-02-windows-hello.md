---
title: Windows Hello 人脸识别失败
date: \today
tags: 
- Windows Hello
- 人脸识别
- PIN
categories: 
- debug
---

## 问题描述
Windows Hello的人脸识别莫名其妙寄了，每次开机都提示“无法识别您的脸，请使用PIN码登录”。
转到设置页发现人脸识别需要重新设置，按了`Set up`之后要求输入PIN码，输入正确的PIN码后报错如下：
![报错](https://lsky.ymqs.top/i/2025/11/10/6911137d964a3.png)

网上查了一圈解决方案都不太管用，最后自己捣鼓一通算是解决了。

## 解决方案
1. 按下 `Win + R` 键，输入 `services.msc` 并回车。
2. 在服务列表中，找到并右键单击 `Windows Biometric Service`。选择“停止”。
3. 打开文件资源管理器，在地址栏输入路径 `C:\Windows\System32\WinBioDatabase` 并回车。删除 WinBioDatabase 文件夹内的所有 .DAT 文件。
4. 回到刚才打开的服务窗口，右键单击 `Windows Biometric Service`。选择“启动”。
5. 回到设置页面，在PIN码处点击删除（Remove）PIN码，然后再设置人脸就能正确调用摄像头了。
![Remove Pin](https://lsky.ymqs.top/i/2025/11/10/691114c99a621.png)

## 设备信息
- 设备型号: Thinkbook 16+
- 操作系统: Windows 11 22H2
