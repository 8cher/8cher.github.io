---
title: 游戏逆向-翻译某游戏
author: Tu
date: 2022-03-17
description:
category:
layout: post
---

### 起因

某日拿从网上找到了某游戏资源。打开游戏看到了游戏的主界面，顿时眉头一皱，发现事情并不简单。游戏处在汉化了但是又没完全汉化的程度。遂打算凭借一己之力对游戏来一次逆向+汉化。

### 过程

#### 初步分析游戏

查看游戏根目录，发现目录是这样的

```
│  Readme.txt.txt
│  {GameName}.exe
└─{GameName}_Data
   │  level0
   │  level1
   |  ...
   │  mainData
   │  resources.assets
   |  ...
   │  sharedassets0.assets
   |  ...
   ├─Managed
   │      Assembly-CSharp.dll
   |      ...
   ├─Mono
   │     mono.dll
   │     ...
   └─Resources
           unity default resources
           unity_builtin_extra
```

可以得出以下几个结论：

1. 游戏使用 Unity 开发
2. 游戏未采用 il2cpp

然后右键查看{GameName}.exe 的版本信息，可以得知游戏使用 4.6.4.10090627 版本的 Unity 进行的开发。

#### 逆向分析

虽然比较老版本的 Unity 尝试使用 AutoTranslator 应该也可以，但是这里仍旧采用逆向的方法。

首先使用 dnSpy 查看源代码。重点放在查找对字符串进行读取的方法上。

形如

```
bool flag=GUI.Button(position,"{TextHere}")
```

奖杯系统
按钮文字
结尾统计界面
