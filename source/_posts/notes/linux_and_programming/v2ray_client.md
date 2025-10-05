---
title: "Setup v2ray client"
excerpt: V2ray客户端配置指南，包括Linux下V2ray安装配置、测试连接方法，以及macOS下V2rayU证书过期问题的解决方案和代码签名修复步骤。
---

## [linux下配置V2ray作为客户端来访问GitHub、G*le等服务](https://www.witersen.com/?p=1408)
```shell
wget https://github.com/v2fly/v2ray-core/releases/download/v4.31.0/v2ray-linux-64.zip
v2ray -test -config config.json
v2ray –config=config.json
curl –socks5 127.0.0.1:1080 https://www.google.com
```

## v2ray 证书过期问题
证书过期问题解决办法：

第一步，执行sudo codesign --force --deep --sign - /Applications/V2rayU.app
第二步，在应用程序中找到V2rayU，右键，显示简介，勾选覆盖恶意软件保护
第三步，打开软件
第四部，执行sudo codesign --force --deep --sign - ~/.V2rayU/V2rayUTool和sudo codesign --force --deep --sign - ~/.V2rayU/v2ray-core/v2ray

然后就能正常运行了。