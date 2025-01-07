+++
title = 'IOS端免插卡使用美区TikTok配合QuantumultX'
date = 2025-01-07T23:02:38+08:00
draft = false
description = "国内无需拔卡使用TikTok操作记录，需要QuantumultX+美区Apple ID+美区梯子+Windows电脑" 
tags = [ "TikTok","跨境电商"] 
categories = [ "跨境电商"]

+++

因为对跨境电商感兴趣，想看看美区的商店都有哪些国内商品，因此配置手机使用美区TikTok，在此记录过程防止遗忘。

## 前置准备

Windows电脑需要安装：

1. 老版本iTunes（带App Store）：用来下载老版本的TikTok安装包。iTunes下载地址：[旧版本iTunes集合](https://www.theiphonewiki.com/wiki/ITunes)，[直达64位12.6.5.2版本](https://secure-appldnld.apple.com/itunes12/091-87819-20180912-69177170-B085-11E8-B6AB-C1D03409AD2A6/iTunes64Setup.exe)，[直达32位12.6.5.3版本](https://secure-appldnld.apple.com/itunes12/091-87820-20180912-69177170-B085-11E8-B6AB-C1D03409AD2A5/iTunesSetup.exe)。
2. 旧版本app搜索工具：[工具下载地址](https://pan.baidu.com/s/1U95g7PzFXpp1OUS_5WdAzA?pwd=code)。
3. 爱思助手：用来安装iTunes下载下来的旧版本TikTok安装包。理论上iTunes也可以直接将app安装到iPhone不需要爱思助手，但由于iTunes是很老的版本，实际操作有未知bug，用爱思装会很丝滑少了一些麻烦。[爱思助手官网](https://www.i4.cn/)。

iPhone需要准备：

1. 会安装使用QuantumultX科学上网，**需要已有能美区科学上网的配置文件，下面步骤会编辑配置文件！！！**
2. Apple Store里登陆美区Apple ID。

**如果不能满足前置条件就别往下看了！！！**

## 操作步骤

建议每一步文字看完了再进行操作。

### 一、配置QuantumultX

1. 开启**MitM**：生成和配置证书。如果你已经是可以科学上网了，这步骤肯定是配置好了的。
2. 开启**重写**。

3. 配置文件，点击编辑：
   1. 编辑`[rewrite_remote]`，添加内容：`https://raw.githubusercontent.com/Semporia/TikTok-Unlock/master/Quantumult-X/TikTok-US.conf, tag=TikTok, update-interval=86400, opt-parser=false, enabled=true`。
   2. 编辑`[filter_remote]`，添加内容：`https://raw.githubusercontent.com/Semporia/TikTok-Unlock/master/Quantumult-X/TikTok.list, tag=TikTok, force-policy=TikTok, update-interval=86400, opt-parser=false, enabled=true`。
   3. 编辑`[rewrite_local]`，添加内容：`(?<=_region=)US(?=&) url 307 US`。
4. 打开QuantumultX首页`自定义策略`，点TikTok，选择PROXY，选择美区节点。

## 二、注册TikTok(已有账号可忽略)

我是使用Gmail在https://www.tiktok.com/新注册的账号。

## 三、下载和安装旧版本TikTok

这部分比较关键也比较麻烦。全程截图。

1. 打开iTunes，点击账号，登陆美区Apple ID。
2. 搜索TikTok：

![iTunes搜索TikTok.png](https://minio.xudashuai.blog/blog/iTunes搜索TikTok.png)

3. 打开旧版本app搜索工具，搜索TikTok，双击打开，选择21.1.0版本，双击打开：

![搜索美区TikTok.png](https://minio.xudashuai.blog/blog/搜索美区TikTok.png)

![选择21.1.0版本.png](https://minio.xudashuai.blog/blog/选择21.1.0版本.png)

![21.1.0版本拦截.png](https://minio.xudashuai.blog/blog/21.1.0版本拦截.png)

4. 返回iTunes页面，点击TikTok下载按钮：

![下载TikTok.jpeg](https://minio.xudashuai.blog/blog/下载TikTok.jpeg)

5. 此时iTunes下载进度卡死不动，解决步骤：
   1. iTunes点击暂停下载
   2. 打开旧版本app搜索工具，点击停止拦截
   3. iTunes点击恢复下载
6. 检查下载结果版本：右键点击TikTok，查看简介：

![检查TikTok下载版本.png](https://minio.xudashuai.blog/blog/检查TikTok下载版本.png)

7. 使用爱思助手安装TikTok步骤：
   1. 右键点击iTunes资料库中的TikTok安装包：在Windows资源管理器中打开，会看见一个.ipa文件
   2. 电脑通过数据线连接iPhone，打开爱思助手，点击`应用游戏`，将ipa文件拖入即可安装

![爱思安装TikTok ipa.png](https://minio.xudashuai.blog/blog/爱思安装TikTok ipa.png)

## 四、更新最新版TikTok

完成以上步骤，打开TikTok，登陆账号。

会发现有些功能不能用，甚至视频无法刷新等问题。

此时再打开手机上的App Store，搜索TikTok，点击更新，即可正常使用。



文章参考：

> [TikTok-Unlock](https://github.com/Semporia/TikTok-Unlock)
>
> [TikTok 免拔卡安装教程：支持安卓与 iOS](https://icloudnative.io/posts/how-to-use-tiktok-in-china/#ios-%E5%85%8D%E6%8B%94%E5%8D%A1%E4%BD%BF%E7%94%A8-tiktok)



