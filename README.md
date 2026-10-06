# LocalCDN（FakeOSS）

本地开发用 Nginx，模拟 OSS / CDN。

生产环境资源走真实 CDN / OSS，**客户端与官网只配一个 BaseURL**。  
本地开发用本仓库，生产直接换 URL，零代码改动。

> 组织总览： [LongLongGames](https://github.com/LongLongGames) · 端口规划见 [profile README §5](https://github.com/LongLongGames/.github/blob/main/profile/README.md)

## 端口

| 服务 | Host | Container |
|------|------|-----------|
| local-cdn | **12280** | 80 |

## 快速开始

```bash
docker compose up -d
curl http://localhost:12280/health   # → ok
```

```
BaseUrl = "http://localhost:12280/"          # 本地
BaseUrl = "https://cdn.yourdomain.com/"      # 生产
```

## data 目录结构（游戏 → 平台 → 渠道）

```
data/
├── ab/{game}/{platform}/                    # AssetBundle 热更
├── app/{game}/{platform}/{channel}/         # CI 构建产物
├── download/{game}/{platform}/{channel}/    # 官网下载中心
├── download/launcher/{windows,osx}/         # GameLauncher
├── config/{game}/                           # 版本清单、热更配置
├── localization/                            # 多语言（见下）
│   ├── common/
│   └── {game}/
└── common/                                  # 跨游戏静态资源（见下）
    ├── images/
    ├── legal/
    └── misc/
```

`game` = match3 | act | flying-chess  
`platform` = android | windows | ios | osx  
`channel` = official | steam | bilibili | taptap | epic …

### 各目录用途

| 目录 | 用途 |
|------|------|
| `ab/` | 各游戏 AssetBundle 热更包 |
| `app/` | CI 产物（含 Steam 构建，内部取包） |
| `download/` | 官网对外稳定下载包 |
| `config/` | 版本清单等（如 `config/match3/version.json`） |
| **`localization/`** | **多语言文案/字体/本地化表**（游戏运行时用；对应 Localization 组件产物；公共放 `localization/common/`） |
| **`common/`** | **跨游戏静态资源**：官网/启动器品牌图、隐私政策等（**不是** AB，不按游戏热更） |

### 路径示例

| 类型 | 路径 |
|------|------|
| match3 Android AB | `/ab/match3/android/xxx.ab` |
| match3 官网官包 | `/download/match3/windows/official/Game_Setup.exe` |
| GameLauncher | `/download/launcher/windows/GameLauncher_Setup.exe` |
| match3 版本清单 | `/config/match3/version.json` |
| 公共多语言 | `/localization/common/zh-CN.json` |
| 隐私政策 | `/common/legal/privacy.html` |

详细约定见 [`data/README.md`](data/README.md)。

## 配置说明

- CORS + autoindex + 缓存头 + `/health`
- 挂载：`./data` → `/usr/share/nginx/html`（只读）

```bash
HOST_CDN_PORT=12280
docker compose down
```
