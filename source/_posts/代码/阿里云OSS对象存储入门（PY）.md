---
title: 阿里云OSS对象存储入门（PY）
tags: [代码]
date: 2026-01-06 20:04:08
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

在这之前，我们应该先讲一下如何获取一个OSS容器。

注册阿里云账号之后，来到阿里云OSS容器的[控制台页面](https://oss.console.aliyun.com/overview)，点击“创建Bucket”，就会看到下面这张图。设置好自己Bucket的配置，然后就可以了。

![很多配置我也看不懂，看看官网的介绍就行了](1.png)

拥有一个Bucket之后，我们需要注册一个RAM用户并获取API Key。来到[RAM访问控制的控制台界面](https://ram.console.aliyun.com/overview),点击创建用户，添加一个用户。一定要牢记自己的Key和Secret，并为对应的用户分配能够操作OSS容器的权限，这样就成功注册一个能够用代码操作OSS的用户了。

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
client.close() # 使用后记得释放资源
```

### 关于`cfg.region`

这是你的bucket所在的地区，必须要对应上，通常来讲是`cn-`+城市名，例如华北2在北京，那么该地区的region就是`cn-beijing`。至于海外的服务器，由于我手头没有海外的服务器，也没看到阿里云官方的文档，因此我也不知道是啥。

## 上传文件

一次上传文件操作是通过构建一个`PutObjectRequest`来完成的，有一些是必填字段，剩下的则是选填字段，这里列出必填和一些重要的选填字段。

必填：
+ `bucket`: `str`，你要上传的目的容器名。
+ `key`: `str`，对象的键名，说白了就是文件名。

选填：
+ `body`: 对象的主体，支持`str`、`bytes`、`IO[str]`、`IO[bytes]`、`Iterable[bytes]`。
+ ...（其他的我还没用过，文档也没写，只能等以后探索了）

传入一个字符串文本`sample_string`的代码如下，我们先创建一个`PutObjectRequst`，然后用`client.put_object`方法来传：

```py
put_object_request = oss.PutObjectRequest(
    bucket='bucket_name',
    key='object_key',
    body='sample_string'
)
client.put_object(put_object_request) # 同步，异步要加上await
```

当你知道文件类型的时候，你也可以直接传bytes，配上文件名，就可以达到上传图片的效果。我不知道这是否是最佳实践，因为我在文档里没看见。但我在memedroids_translator中就用的这个，确实可以用。

如果想创建文件夹的话，直接修改`key`参数即可。注意不需要用什么`Path`之类的，OSS里用的都是`/`。

```py
correct_key = 'a' + '/' 'b' # 正确
invalid_key = Path('a') / 'b' # 错误
```

## 下载文件

和上传文件类似，下载文件要构建的是一个`GetObjectRequest`，这里要传的参数就只用容器名就可以了。

```py
get_object_request = oss.GetObjectRequest(
    bucket='bucket_name',
    key='object_key',
)
result = client.get_object() # 同步，异步要加上await
```

得到`result`后，你需要通过它的`body`属性来获取，因为`result`的`GetObjectResult`对象本质上是一个“响应包装器”，而读取又有两种方式，完整读取和分块读取。虽然我没用过分块读取，但由于官方比较推荐，所以还是放过来吧，下次做项目的时候用用。

```py
# 完整读取
content_bytes = result.body.read()          # 异步这里有点特殊， await result.body.read()在vscode中会报错，但我亲测是能跑的
# 分块读取，因为我没用过，所以从阿里云官方文档中直接粘过来了
with result.body as body_stream:
    chunk_path = "./get-object-sample-chunks.txt"
    total_size = 0
    with open(chunk_path, 'wb') as f:
        # 使用256KB块大小（可根据需要调整block_size参数）
        for chunk in body_stream.iter_bytes(block_size=256 * 1024):
            f.write(chunk)
            total_size += len(chunk)
            print(f"已接收数据块：{len(chunk)} bytes | 累计：{total_size} bytes")
```

## 文件名查找

不是在任何情况下我们都能知道文件的名字的，因此查找合适的文件名也是一个刚需。OSS也给了我们这个功能，利用`ListObjectsV2Request`（我也不知道为什么是V2，也许是SDK版本是V2吧）就可以了。

`ListObjectV2Request`的一些比较重要的参数如下：

+ `bucket`：`str`，容器的名字。
+ `prefix`：`str`，你要查找的文件名字的前缀，不能以`/`开头，如果为空则返回所有文件的`key`。
+ `max_keys`：`number`，返回文件的最大数量。
+ `continuation_token`：`str`，查找的起始位置。

其中，`prefix`和`delimiter`参数还可以进行组合使用。因为`prefix`默认是递归查找的，修改`delimiter`就可以控制是否是递归查找了。例如，一个Bucket中有三个Object，分别为fun/test.jpg、fun/movie/001.avi和fun/movie/007.avi。如果设定prefix为fun/，则返回三个Object；如果在prefix设置为fun/的基础上，将delimiter设置为正斜线（/），则返回fun/test.jpg和fun/movie/（摘自阿里云官方文档）。

```py
    get_objects_request = oss.ListObjectsV2Request(
        bucket='bucket_name',
        prefix='prefix',
        max_keys=100,
        continuation_token='continuation_token',
    )
    result = OSS_CLIENT.list_objects_v2(get_objects_request) # 一样的，也是同步
```

## 总结

本文讲述了我在刚使用阿里云OSS对象存储的基本方法，能够实现基本的上传下载和查找功能。如果以后用到了更多的功能，还会继续分享，进行一个知识的沉淀的。在代码层面OSS存储其实并没有比本地存储更加的简便，但是确实更适合在github actions这种场景上跑。同时由于阿里云的文档比较分散，以后更新这篇文章的时候，就可以把文档之类的整合起来，方便查找和使用。

## 参考链接

+ [阿里云对象存储官方文档](https://help.aliyun.com/zh/oss/?spm=a2c4g.11186623.0.0.485a60a1wyLbyf)
+ [阿里云对象存储API参考](https://help.aliyun.com/zh/oss/developer-reference/list-of-operations-by-function)（虽然页面上的导航显示的是他是上面那个链接的一部分，但是其实是找不到的）
+ [alibaba_cloud_oss_v2 Documention](https://gosspublic.alicdn.com/sdk-doc/alibabacloud-oss-python-sdk-v2/latest/index.html)

## 更新记录

1. 2026-1-6 发布第一稿