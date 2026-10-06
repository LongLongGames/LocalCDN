# LocalCDN（FakeOSS）

本地开发用 Nginx，模拟 OSS / CDN。

生产环境资源走真实 CDN / OSS，**客户端与官网只配一个 BaseURL**。  
本地开发用本仓库，生产直接换 URL，零代码改动。

> 组织总览： [LongLongGames](https://github.com/LongLongGames) · 端口规划见 [profile README §5](https://github.com/LongLongGames/.github/blob/main/profile/README.md)

## 端口

| 服务 | Host | Container |
|------|------|-----------|
| local-cdn | **12280** | 80 |

符合组织组件段 `12000–12999`。

## 快速开始

```bash
docker compose up -d
curl http://localhost:12280/health   # → ok
```

客户端 / 官网本地：

```
BaseUrl = "http://localhost:12280/"
```

生产：

```
BaseUrl = "https://cdn.yourdomain.com/"
```

## data 目录结构

```
data/
├── ab/                 # AssetBundle（按平台，热更）
│   ├── android|windows|ios|osx/
├── app/                # CI 构建产物（平台 × 渠道，含 steam）
│   ├── android/{official,steam,bilibili,taptap}/
│   ├── windows/{official,steam,epic}/
│   └── osx/{official,steam}/
├── download/           # 官网下载中心（对外稳定包）
│   ├── android/{official,bilibili,taptap}/
│   ├── windows/{official,steam,epic}/
│   ├── osx/{official,steam}/
│   └── launcher/{windows,osx}/    # GameLauncher 安装包
├── config/             # 热更配置、版本清单
├── localization/
└── common/
```

### 官网下载路径示例

| 内容 | 本地 URL |
|------|----------|
| Windows 官包 | `http://localhost:12280/download/windows/official/GameName_Setup.exe` |
| Android 官包 | `http://localhost:12280/download/android/official/GameName.apk` |
| GameLauncher | `http://localhost:12280/download/launcher/windows/GameLauncher_Setup.exe` |
| Steam 引导/备用 | `http://localhost:12280/download/windows/steam/...` |

### CI 产物 vs 官网下载

| 目录 | 用途 |
|------|------|
| `app/` | CI 编译统一落点（含 Steam 构建，内部取包、对账、再上传渠道） |
| `download/` | 官网下载中心对外稳定包（发布时从 app 晋升或单独打） |

Steam EXE 最终走 Steam 管道，但 **CI 仍上传到 `app/windows/steam/`**；官网需要展示的下载入口走 `download/`。

### AssetBundle 示例

| 类型 | 路径 |
|------|------|
| Android AB | `/ab/android/match3/xxx.ab` |
| Windows AB | `/ab/windows/match3/xxx.ab` |
| 版本清单 | `/config/match3/version.json` |

新增渠道：在对应平台下新建目录即可，无需改 Nginx。

## 配置说明

- CORS + autoindex + 缓存头 + `/health`
- 挂载：`./data` → `/usr/share/nginx/html`（只读）

```bash
HOST_CDN_PORT=12280   # 可选覆盖
docker compose down
```
