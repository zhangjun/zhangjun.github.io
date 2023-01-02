---
title: VPN service
date: 2023-01-02 11:26:53
tags:
---

# 使用VPS搭建VPN代理
## 原理
<img width="572" alt="image" src="https://user-images.githubusercontent.com/1312389/210215276-f2394f9e-32b3-47ab-8925-0dcc104adadb.png">

## 准备工作
### 1、免费域名
- freenom免费域名申请
   申请地址：https://my.freenom.com/
   申请时需要保证个人资料地址信息与网络ip地址信息一致；国内ip环境，需使用Gooreplacer chrome插件将www.google.com/recaptcha 重定向recaptcha.net/recaptcha
- eu.org免费域名申请
  申请地址：https://nic.eu.org/arf/en (需使用代理)
- https://pp.ua/ 免费域名申请

### 2、域名解析
- https://topdn.net
   配置简单，更新快速
- cloudflare
  个人推荐cloudflare，功能齐全，同时能实现ip地址隐藏

### 3、CDN（可选）
   https://www.cloudflare.com/ 可以实现vps ip地址隐藏，同时也可以解析到境外已被墙ip（例如阿里云香港主机）
## 搭建步骤
### 1、v2ray或者trojan服务器伪装
### 2、客户端v2rayNG配置
### 3、CDN流量中转（可选）
   流量中转目的：1、隐藏VPS ip；2、解救被海外封ip

## 参考链接
- [v2ray使用cloudflare中转流量，拯救被墙ip](https://v2xtls.org/v2ray%E4%BD%BF%E7%94%A8cloudflare%E4%B8%AD%E8%BD%AC%E6%B5%81%E9%87%8F%EF%BC%8C%E6%8B%AF%E6%95%91%E8%A2%AB%E5%A2%99ip/)
- [V2Ray高级技巧：流量伪装](https://itlanyan.com/v2ray-traffic-mask/)
- [拯救被墙的服务器](https://itlanyan.com/recovery-blocked-ip/)
- [V2ray的VLESS协议介绍和使用教程](https://itlanyan.com/introduce-v2ray-vless-protocol/)
- [trojan教程](https://itlanyan.com/trojan-tutorial/)
- [未来的霸主选项：Xray（XTLS+V2ray)](https://www.vjsun.com/656.html)
- [Trojan-Go一键安装脚本（Debian/Ubuntu） Trojan-Go搭建/支持Cloudflare CDN](https://ssrvps.org/archives/7772)
