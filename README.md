<div align="center">
<img style="max-width:50%;" src="pic/logo.png" alt="NASSAV" />
<br>
</div>

<div align="center">
  <img src="https://img.shields.io/github/stars/Satoing/NASSAV?style=for-the-badge&color=FF69B4" alt="Stars">
  <img src="https://img.shields.io/github/forks/Satoing/NASSAV?style=for-the-badge&color=FF69B4" alt="Forks">
  <img src="https://img.shields.io/github/issues/Satoing/NASSAV?style=for-the-badge&color=FF69B4" alt="Issues">
  <img src="https://img.shields.io/github/license/Satoing/NASSAV?style=for-the-badge&color=FF69B4" alt="License">
</div>

## 项目简介

NASSAV 是一个基于 Python 开发的多源影视资源下载管理工具，支持从多个数据源自动下载、整理和刮削影视资源。

项目采用模块化设计，支持自定义下载器，并提供了完整的元数据管理功能和配套服务。

## 核心特性

- 🎥 多源下载支持：支持 MissAV、Jable、HohoJ、Memo （持续添加中）等多个数据源
- 📝 智能元数据管理：从JavBus自动获取影片信息、封面、海报等元数据
- 🔄 队列管理：支持批量下载任务管理。使用sqlite去重，防止重复下载
- 🌐 远程控制：提供 HTTP API 接口，支持远程控制下载任务
- 🔒 文件锁机制：确保同一时间只有一个下载任务运行
- 🎨 媒体服务器兼容：自动生成 NFO 文件，支持主流媒体服务器

## 系统要求

- **稳定的网络连接和代理服务**
- Python 3.11.2 或更高版本
- FFmpeg
- Go 1.22.6 或更高版本（仅用于编译 HTTP 服务器）

## 安装指南

1. 克隆项目并安装依赖：
```bash
git clone https://github.com/Satoing/NASSAV.git
cd NASSAV
pip3 install -r requirements.txt
```

2. 安装 FFmpeg：
```bash
sudo apt install ffmpeg
```

3. 配置项目：
   - 复制 `cfg/configs.json.example` 为 `cfg/configs.json`
   - 修改配置文件中的关键参数：
     - `SavePath`：设置视频保存路径
     - `Proxy`：配置代理服务器地址（如果不需要使用代理，设置成""即可）
     - `Downloader`：配置下载器及其优先级
     - `IsNeedVideoProxy`：下载视频是否优先使用代理（最终都会尝试使用代理和不使用代理）

## 使用方法

### 基本使用

0. 初始化，修改配置文件。主要关注的字段：
    - SavePath：下载保存的位置
    - Proxy：http代理服务器url（如果不需要使用代理，设置成""即可）
    - IsNeedVideoProxy：下载视频是否使用代理
```json
{
    "LogPath": "./logs",
    "SavePath": "/vol2/1000/MissAV",
    "DBPath": "./db/downloaded.db",
    "QueuePath": "./db/download_queue.txt",
    "Proxy": "http://127.0.0.1:7897",
    "IsNeedVideoProxy": false,
}
```

1. 如果本地已有资源，需要先整理目录结构，车牌号大写作为文件夹名字，视频同名放在文件夹里面：

```
...
├── SVGAL-009
│   └── SVGAL-009.mp4
├── STCVS-007
│   └── STCVS-007.mp4
...
```
然后执行`python3 metadata.py`，爬取元数据。最后生成的目录结构：
```
...
├── SVGAL-009
│   ├── metadata.json
│   ├── SVGAL-009-fanart.jpg
│   ├── SVGAL-009.mp4
│   ├── SVGAL-009.nfo
│   └── SVGAL-009-poster.jpg
├── thumb
│   ├── JULIA.jpg
│   ├── ちゃんよた.jpg
│   ├── 七森莉莉.jpg
│   ├── 七泽米亚.jpg
...
```

2. 下载单个资源：
```bash
python3 main.py <车牌号>
```

3. 强制下载（忽略重复检查——下载过程中ctrl-c就会出现这种情况）：
```bash
python3 main.py <车牌号> -f
```

### 使用Docker下載

0. 前置作業如基本使用

1. Build Docker (初次使用才需要)
```bash
(sudo) docker build -t nassav .
```

2. 下載
```bash
(sudo) docker run --rm -v "<本機存片位置>:<cfg/configs.json存片位置>" nassav <車號>
```

### 批量下载

1. 将车牌号添加到 `db/download_queue.txt` 中
2. 设置定时任务：
```bash
20 * * * * cd /path/to/NASSAV && bash cron_task.sh
```

### HTTP API 服务

1. 编译并启动 HTTP 服务器：
```bash
cd backend
go build -o main
./main
```

2. 发送下载请求：
```bash
curl -X POST http://127.0.0.1:49530/process -d "车牌号"
```

### 前后端服务

刮削时下载了大量fanart，故提供一个网页预览。

后端提供了两个API：
1. 获取车牌号列表：/api/videos
2. 获取车牌号详细信息：/api/videos/FPRE-017

前端部署方式：
1. 先调整`frontend/src/api/videos.js`中后端的配置：
```js
import axios from 'axios'
const API_BASE = 'http://192.168.31.61:31471' // 改成自己的ip
```
2. 重新生成静态文件到dist目录下：
```bash
cd frontend
npm run build
```
3. 使用nginx部署静态网页：`127.0.0.1:5177`

## 配置说明

### 下载器配置

在 `configs.json` 中可以配置多个下载源及其优先级：

```json
"Downloader": [
    {
        "downloaderName": "MissAV",
        "domain": "missav.ai",
        "weight": 300
    },
    {
        "downloaderName": "Jable",
        "domain": "jable.tv",
        "weight": 500
    },
    {
        "downloaderName": "HohoJ",
        "domain": "hohoj.tv",
        "weight": 0
    }
]
```

### 数据源说明

1. **MissAV**
   - 优点：资源全面，反爬限制较少
   - 缺点：清晰度一般（720p-1080p）

2. **Jable**
   - 优点：中文字幕资源多，清晰度高（1080p）
   - 缺点：反爬限制较严格

3. **HohoJ**
   - 优点：清晰度高（1080p），基本无反爬限制
   - 缺点：中文字幕资源较少

4. **Memo**
   - 优点：资源较新，更新及时
   - 缺点：部分资源需要会员


## 开发指南

### 添加新的下载器

1. 在 `src/downloader/` 目录下创建新的下载器类
2. 继承 `Downloader` 基类，实现必要的方法：
   - `getDownloaderName()`
   - `getHTML()`
   - `parseHTML()`
3. 在 `DownloaderMgr` 中注册新下载器
4. 在配置文件中添加相应的配置项

示例代码：
```python
class NewDownloader(Downloader):
    def getDownloaderName(self) -> str:
        return "NewDownloader"

    def getHTML(self, avid: str) -> Optional[str]:
        # 实现获取HTML的逻辑
        pass

    def parseHTML(self, html: str) -> Optional[AVDownloadInfo]:
        # 实现解析HTML的逻辑
        pass
```
### 有需求请自行fork修改，如果想要贡献代码发起PR即可

![](pic/IMG_5150.JPG)

## 注意事项

- 使用本项目需要稳定的代理服务
- 请遵守相关法律法规，合理使用本工具
- 建议定期备份数据库文件
- 下载过程中请确保网络稳定
- 下载频率不要过高。否则会被cloudflare安排进贤者时间

## Reference

前人栽树，后人乘凉

1. m3u8-Downloader-Go（m3u8命令行下载器）: https://github.com/Greyh4t/m3u8-Downloader-Go
2. mrjet（如何获取到missav的m3u8）: https://github.com/cailurus/mrjet


## 许可证

本项目采用 MIT 许可证。详见 [LICENSE](LICENSE) 文件。 
