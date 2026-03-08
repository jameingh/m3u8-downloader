# m3u8-downloader

golang 多线程下载直播流m3u8格式的视屏，跨平台。 你只需指定必要的 flag (`u`、`o`、`n`、`ht`) 来运行, 工具就会自动帮你解析 M3U8 文件，并将 TS 片段下载下来合并成一个文件。


## 功能介绍

1. 下载和解析 M3U8
2. 下载 TS 失败重试 （加密的同步解密)
3. 合并 TS 片段

> 可以下载岛国小电影  
> 可以下载岛国小电影  
> 可以下载岛国小电影    
> 重要的事情说三遍......

## 效果展示
![demo](./demo.gif)

## 参数说明：

```
- u  m3u8下载地址(http(s)://url/xx/xx/index.m3u8)
- o  movieName:自定义文件名(默认为movie)不带后缀 (default "movie")
- n  num:下载线程数(默认24)
- ht hostType:设置getHost的方式(v1: http(s):// + url.Host + filepath.Dir(url.Path); v2: `http(s)://+ u.Host` (default "v1")
- c  cookie:自定义请求cookie (例如：key1=v1; key2=v2)
- r  autoClear:是否自动清除ts文件 (default true)
- s  InsecureSkipVerify:是否允许不安全的请求(默认0)
- sp savePath:文件保存的绝对路径(默认为当前路径,建议默认值)(例如：unix:/Users/xxxx ; windows:C:\Documents)
```

默认情况只需要传`u`参数,其他参数保持默认即可。 部分链接可能限制请求频率，可根据实际情况调整 `n` 参数的值。

## 下载

已经编译好的平台有： [点击下载](https://github.com/llychao/m3u8-downloader/releases)

- m3u8-darwin-amd64
- m3u8-darwin-arm64
- m3u8-linux-386
- m3u8-linux-amd64
- m3u8-linux-arm64
- m3u8-windows-386.exe
- m3u8-windows-amd64.exe
- m3u8-windows-arm64.exe

## 用法

### 交互式使用（推荐）

直接运行程序，根据提示输入即可：

```bash
./m3u8-downloader
```

程序会提示：
1. 输入 m3u8 视频地址
2. 输入文件名（直接回车则使用日期时间格式，如：`video_20260309_150405`）

### 命令行方式

#### 简洁使用

```bash
./m3u8-downloader -u=http://example.com/index.m3u8
```

文件名将提示输入或使用日期时间格式。

#### 完整使用

```bash
./m3u8-downloader -u=http://example.com/index.m3u8 -o=example -n=64 -ht=v1 -c="key1=v1; key2=v2"
```

方法二：
```bash
自己编译：go build -o m3u8-downloader
简洁使用：./m3u8-downloader  -u=http://example.com/index.m3u8
完整使用：./m3u8-downloader  -u=http://example.com/index.m3u8 -o=example -n=16 -ht=v1 -c="key1=v1; key2=v2"
```

### 命令行参数说明

| 参数 | 说明 | 默认值 | 示例 |
|------|------|--------|------|
| `-u` | m3u8 下载地址 | 无 | `http://example.com/index.m3u8` |
| `-o` | 自定义文件名（不带后缀） | 交互输入 | `myvideo` |
| `-n` | 下载线程数 | 24 | `64` (高速网络推荐) |
| `-ht` | 获取 host 的方式 | `v1` | `v1` 或 `v2` |
| `-c` | 自定义请求 cookie | 无 | `"key1=v1; key2=v2"` |
| `-r` | 是否自动清除 ts 文件 | `true` | `false` 保留 ts 文件 |
| `-s` | 允许不安全请求 | `0` | `1` 跳过证书验证 |
| `-sp` | 文件保存路径 | `~/Downloads` | `/path/to/save` |

### 使用示例

```bash
# 使用默认设置下载
./m3u8-downloader -u="http://example.com/video.m3u8"

# 自定义文件名和并发数
./m3u8-downloader -u="http://example.com/video.m3u8" -o=myvideo -n=64

# 使用备用 host 模式
./m3u8-downloader -u="http://example.com/video.m3u8" -ht=v2

# 带 cookie 下载
./m3u8-downloader -u="http://example.com/video.m3u8" -c="session=abc; token=xyz"

# 保存到指定目录
./m3u8-downloader -u="http://example.com/video.m3u8" -sp="/Volumes/SSD/videos"
```

## 性能优化建议

### 并发数设置

| 网络带宽 | 推荐线程数 | 参数 |
|----------|------------|------|
| 100Mbps | 32 | `-n=32` |
| 500Mbps | 64 | `-n=64` |
| 1Gbps+ | 100 | `-n=100` |

> **注意**：部分服务器会限制请求频率，如遇 429 错误请降低 `-n` 值。

### 超时时间

默认超时已优化为 **30 秒**，适应各种网络环境。

### 保存位置

默认保存到 `~/Downloads` 目录，可通过 `-sp` 参数自定义：

```bash
# MacOS
./m3u8-downloader -u="..." -sp="/Volumes/SSD/videos"

# Windows
.\m3u8-downloader.exe -u="..." -sp="D:\Downloads"

# Linux
./m3u8-downloader -u="..." -sp="/mnt/data/videos"
```
2.下载失败的情况,请设置 -ht="v1" 或者 -ht="v2" （默认为 v1）
```golang
func get_host(Url string, ht string) string {
    u, err := url.Parse(Url)
    var host string
    checkErr(err)
    switch ht {
    case "v1":
        host = u.Scheme + "://" + u.Host + path.Dir(u.Path)
    case "v2":
        host = u.Scheme + "://" + u.Host
    }
    return host
}
```
