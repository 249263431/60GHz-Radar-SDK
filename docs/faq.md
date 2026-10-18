---
layout: default
title: 常见问题 FAQ
---

# 常见问题 FAQ

## 雷达能穿墙吗？
毫米波对布帘、薄木门等非金属材料有一定穿透，但**混凝土墙、金属会严重衰减**。雷达需正对监测区域安装，不要隔着承重墙使用。

## 需要上云吗？数据去哪？
不需要。雷达通过 **本地 MQTT broker**（如 Mosquitto / Artemis）上报数据，默认**不出家庭网络**。上云是可选项，由你自己决定。

## 隐私怎么样？和摄像头比呢？
雷达只输出距离、速度、存在 / 跌倒状态等点云级信息，**不拍视频、不录音**，比摄像头隐私友好得多，适合浴室、卧室等私密空间。

## 必须用 2.4GHz Wi-Fi 吗？
是的。雷达配网走 **2.4GHz Wi-Fi**，不支持 5GHz。请确保路由器开启了 2.4G 频段（很多双频路由默认合并，需在后台分开）。

## 语音 / 报警语言怎么选？
固件支持中 / 英文语音播报，可在配套 App 或配置项里切换。详见各产品指南。

## 怎么接 Home Assistant？
见 [Home Assistant 接入（已验证）]({{ "/2026/10/18/home-assistant-radar-integration.html" | relative_url }})，以及跌倒 / 睡眠 / 存在雷达各自的 HA 接入分篇。

## 供电与安装位置有什么要求？
见 [睡眠雷达安装位置说明]({{ "/installation/2026/10/16/sleep-radar-installation-position.html" | relative_url }})。

## 延迟多大？实时性如何？
见 [雷达延迟解析]({{ "/fundamentals/2026/10/17/radar-latency-explained.html" | relative_url }})。

## 宣传的探测距离 / 角度是不是都靠谱？
见 [雷达规格与能力边界]({{ "/fundamentals/2026/10/17/radar-spec-limits.html" | relative_url }})。

## 原理是什么？毫米波怎么测呼吸 / 跌倒？
见 [毫米波雷达工作原理]({{ "/fundamentals/2026/10/17/how-mmwave-radar-works.html" | relative_url }})。

## 买哪款？
见 [该买哪款对比页]({{ "/compare.html" | relative_url }})。

## 还有问题？
欢迎在 [Tindie 商店](https://www.tindie.com/stores/fuxinsuo/){:target="_blank"} 留言，或到 GitHub 提 Issue。
