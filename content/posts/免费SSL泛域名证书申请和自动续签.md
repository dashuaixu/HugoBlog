+++
title = '免费SSL泛域名证书申请和自动续签'
date = 2025-08-09T14:07:55+08:00
draft = true
description = "申请Let’s Encrypt免费泛域名和自动续签" 
tags = [ "Let’s Encrypt", "阿里云"] 
categories = [ "建站"]

+++

[Let’s Encrypt](https://letsencrypt.org/getting-started/)官方推荐使用[https://certbot.eff.org/](https://certbot.eff.org/)客户端来自动化申请证书和执行续签操作，自动化续签可以使用`crontab -e`来实现，每次续签时还需要修改DNS服务商的TXT记录值，使用certbot的hook功能