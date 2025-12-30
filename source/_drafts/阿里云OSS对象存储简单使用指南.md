---
title: 阿里云OSS对象存储简单使用指南
tags: [代码]
categories: 代码
---

## 为什么要写这一篇博客

我在做`memedroid_translator`的时候，会有使用serverless或者GitHub Actions的需求，这就要求我需要有一个地方，能够根据url把爬取的图片持久化。如果直接上传到服务器，变成一个网络url，那我其实还不如直接用原网站的url呢；同时我之前询问了Kimi，它给出的解决方案也是使用阿里云OSS这种弹性容器，可以按需储存。它的解释是这样的：

> Q: 使用GitHub Action来运行爬虫并存储图片，是直接存到自己的服务器中好还是存储到oss好
> A: 结论先行：
> 
> GitHub Actions 跑爬虫存图片，优先用 OSS（或任意 S3 兼容对象存储），不要直接回写 GitHub 仓库，也不建议自己再维护一台服务器。原因如下：
>
> | 维度            | 直接存自己服务器                                                  | 存 OSS/Cloudflare R2 等                               |
> | ------------- | --------------------------------------------------------- | --------------------------------------------------- |
> |  网络出口       | Actions 先下载到 runner，再 `scp`/`rsync` 到你服务器，走公网， 速度慢、易掉线  | 直接用官方 SDK 走内网上传， 稳定、秒级完成                          |
> |  存储成本       | 服务器硬盘自己扩容， 费用线性增加                                       | Cloudflare R2 每月 10 GB 免费，阿里云 OSS 按量几分钱/GB， 几乎白嫖  |
> |  运维成本       | 要管机器、备份、磁盘告警、SSH 密钥、防火墙…… 一个人全包                         | 全托管， 零运维                                          |
> |  Action 时长  | 上传大文件占 runner 时间， 2000 min/月免费额度快速消耗                    | 上传耗时短， 节省免费时长                                     |
> |  仓库体积       | 如果误提交到 Git，仓库迅速膨胀， GitHub 1 GB 软限制，5 GB 硬限制             | 文件不进仓库， 仓库保持轻量                                    |
> |  权限/安全      | 服务器密钥硬编码在 Secret， 一旦泄露整机被日                              | OSS 用临时 STS 或最小权限子账号， 泄露只丢一桶图                     |

因此我准备使用OSS来进行存储。同时由于阿里云的网站使用起来非常复杂，所有的服务、控制台、接口全在一块，非常的不方便，同时接口也有很多没有解释到位的地方，因此我就准备写这一篇博客，来讲一下自己在使用阿里云OSS的时候碰到了什么坑。

## 一些基本定义介绍

+ Bucket容器：就是一块存储空间，抽象成一个容器罢了。
+ 对象Object：对象（Object）是OSS存储数据的基本单元，也被称为OSS的文件。和传统的文件系统不同，Object没有文件目录层级结构的关系（摘自阿里云官网）。
+ RAM用户：操作OSS的主体，系统判断能否进行某种操作看的就是该用户是否有对应权限。

顺便说一句，阿里云OSS的库（Python SDK V2）名叫`alibabacloud_oss_v2`，在本文中的`import`作如下规定：

```py
import alibabacloud_oss_v2 as oss # 同步
import alibabacloud_oss_v2.aio as oss_aio # 异步
```


## 连接OSS弹性容器

在连接一个容器之前，要先创建一个Client，传入合法的`access_key_id`和对应的`access_key_secret`，设置所在地区并读取配置。我知道的方法有以下几种。

### 直接读取环境变量

这个方法是官网给岀的方法，要求系统中有`OSS_ACCESS_KEY_ID`和`OSS_ACCESS_KEY_SECRET`这两个变量。

```py
credentials_provider = oss.credentials.EnvironmentVariableCredentialsProvider()
cfg = oss.config.load_default()
cfg.credentials_provider = credentials_provider
cfg.region = 'cn-hangzhou' 
client = oss.Client(cfg) # 同步
client = oss_aio.AsyncClient(cfg) # 异步
```

其中，`credentials_provider`是凭证提供者变量，`cfg`是配置项字典。

### 传入静态变量

这是我在写memedroid_translator时所使用的方法，`access_key_id`和`access_key_secret`通过传字符串的方式给`cfg`。

```py
credentials_provider = oss.credentials.StaticCredentialsProvider(
    access_key_id='access_key_id',
    access_key_secret='access_key_secret',
)
cfg = oss.config.load_default()
cfg.credentials_provider = credentials_provider
cfg.region = 'cn-hangzhou' 
client = oss.Client(cfg) # 同步
client = oss_aio.AsyncClient(cfg) # 异步
```

### 关于`cfg.region`

这是你的bucket所在的地区，必须要对应上，通常来讲是`cn-`+城市名，例如华北2在北京，那么该地区的region就是`cn-beijing`。

## 上传文件

TODO: 把这段和下一段写完

```py
put_object_request = oss.PutObjectRequest(
    bucket='bucket_name',
    key=name,
    body=data
)
client.put_object(put_object_request) # 同步，异步要加上await
```

## 下载文件



## 匹配

```py
 prefix = (
        CONFIG["crawler"]["save_path"]
        if mode == "en"
        else CONFIG["translate"]["output_path"]
    )
    try:
        object_keys = []
        continuation_token = None
        get_objects_request = oss.ListObjectsV2Request(
            bucket=CONFIG["oss"]["bucket_name"],
            # 这里其实写死也没事，毕竟阿里云oss用的就是'/'，如果你用Path的话，在win上面反而会拼错
            prefix=prefix + "/",
            max_keys=CONFIG["translate"]["max_key_length"],
            continuation_token=continuation_token,
        )
        # 获取对象列表
        result = await OSS_CLIENT.list_objects_v2(get_objects_request)
        if result.contents is not None:
            for obj in result.contents:
                object_keys.append(obj.key)
        else:
            raise Exception("No objects found in OSS bucket.")
    except Exception as e:
        print(f"OSS get object error: {e}")
    finally:
        await OSS_CLIENT.close()
        return object_keys

```