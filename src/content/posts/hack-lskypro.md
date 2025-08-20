---
title: 记录破解兰空图床（Lsky-Pro）
published: 2025-08-19T21:09:47
description: '兰空图床是一个简洁易用（？）的图床框架，抓包了一下激活过程，这玩意居然没加密...记录一下'
image: '../assets/images/2025-08-20-21-11-48-image.png'
tags: [兰空图床]
category: '记录'
draft: false 
lang: ''
---

> 仅供学习交流，请在下载后24h内删除。体验地址： https://lsky.2x.nz

# 下载包体

兰空图床Pro付费版包： https://r2.072103.xyz/lp223.zpaq （解压密码：2x.nz二叉树树）

# 配置环境

自行参考： https://docs.lsky.pro/guide/install

**推荐使用宝塔面板部署**，1Panel的容器化PHP好像有点问题

如果坚持要用1Panel，推荐使用PHP8.2，出现500，404等状态码问题请自行解决，似乎需要一个特殊的 `fallback` 设置。感谢 fishcpy提供的解决方案！这里是他的部署教程： [AcoFork的兰空图床开心版1panel部署教程 - 福利羊毛 - LINUX DO](https://linux.do/t/topic/882900)

```nginx
    # 全局 404 交给 @fallback 处理，不强制状态码
    error_page 404 @fallback;



    location / {
        try_files $uri $uri/ @fallback;
    }



    # 命名 location：交给 index.php，但不强制 200
    location @fallback {
        rewrite ^ /index.php last;
    }



    # -----------------------------
    # 特殊路径：/api/v2/ 也走 index.php，但不能强制 200
    # -----------------------------
    location ^~ /api/v2/ {
        # 同样使用 @fallback，不强制状态码
        try_files $uri $uri/ @fallback;
    }
```

# 破解授权

首先为你的Linux配置一个HTTP代理，指向 Burp Suite（软件自己找）

```bash
export http_proxy="http://127.0.0.1:8080"
export https_proxy="http://127.0.0.1:8080"
```

![](../assets/images/a5fd2695975981d785cea1af5c0ee9588dc1b9ee.png)

默认Burp仅拦截请求，不拦截响应，需要手动设置一下

![](../assets/images/2690f8470df19d0c4a0f134835a7cbc95c9798fd.png)

然后启用拦截

![](../assets/images/52650c556acc9406923fb824823fe3a04e153d5d.png)

当你通过官方教程到执行 `./install.sh` 的时候

会要求输入域名和授权密钥，域名填你自己的，否则之后上传的图片的预览地址将会不正确！授权密钥随便填！

![](../assets/images/67b17d4c5f5d7ba8d2e2ee348d19bc01c6d42b1d.png)

回车，会开始转圈圈

![](../assets/images/fb540faa472d476e8d6b05a04d01be5a19adb236.png)

查看Burp，发现多了一个请求，首先点击放行

![](../assets/images/8a6dd20b7ad55a9fdad795be358b8486b75de5b7.png)

现在出现了响应，并且状态码为401

![](../assets/images/ce862cb4eeefc2a7a52bea44e4e6ab137a7cd3da.png)

响应那块是可以编辑的，用 https://r2.072103.xyz/lsky_success_223.txt 中的内容替换原响应。然后点击放行

![](../assets/images/b8545b978629815aec471489890a0be62f0a8f89.png)

恭喜，通过授权了

![](../assets/images/fdda3a54fd4a6da5d0c0c9d5ac0fbd5b79ef2b51.png)

安装完毕后也一样

![](../assets/images/79f0f4645235e7cb3ecbe554cb13295bed326be5.png)38dd52c6e.png)

> 注意。如果需要更新新版本，仍然需要有效的授权密钥，否则无法得到新版包体

# 自建授权服务器

自建结束后将 `config/app.php` 内的 `服务接口地址` 改为你的。就不需要每次安装都手动改响应包了

```php
    /**
     * 服务接口地址
     */
    'service_api' => env('APP_SERVICE_API', 'https://huohuastudio.com'),
```
