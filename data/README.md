# LocalCDN data 目录约定

生产 OSS / CDN 保持相同路径，客户端与官网只换 BaseURL。

**分层原则：游戏 → 平台 → 渠道**

```
data/
├── ab/                                  # AssetBundle（热更）
│   ├── match3/{android,windows,ios,osx}/
│   ├── act/{android,windows,ios,osx}/
│   └── flying-chess/{android,windows,ios,osx}/
├── app/                                 # CI 构建产物落点
│   ├── match3/
│   │   ├── android/{official,steam,bilibili,taptap}/
│   │   ├── windows/{official,steam,epic}/
│   │   └── osx/{official,steam}/
│   ├── act/          # 同上结构
│   └── flying-chess/ # 同上结构
├── download/                            # 官网下载中心（对外稳定包）
│   ├── match3/
│   │   ├── android/{official,bilibili,taptap}/
│   │   ├── windows/{official,steam,epic}/
│   │   └── osx/{official,steam}/
│   ├── act/          # 同上
│   ├── flying-chess/ # 同上
│   └── launcher/{windows,osx}/          # GameLauncher（跨游戏）
├── config/                              # 热更配置、版本清单（按游戏分子目录）
│   ├── match3/version.json
│   ├── act/version.json
│   └── ...
├── localization/                        # 多语言文案 / 字体 / 本地化资源（按游戏或公共）
│   ├── common/                          # 全游戏共用（如通用 UI 文案）
│   ├── match3/
│   ├── act/
│   └── flying-chess/
└── common/                              # 跨游戏、跨平台的非 AB 公共静态资源
    ├── images/                          # 官网/启动器共用图、品牌素材
    ├── legal/                           # 隐私政策、用户协议等
    └── misc/
```

## 各目录用途（必读）

| 目录 | 存什么 | 谁用 | 说明 |
|------|--------|------|------|
| `ab/{game}/{platform}/` | AssetBundle 热更包 | 游戏客户端 | 按游戏隔离，再按平台分 |
| `app/{game}/{platform}/{channel}/` | CI 编译产物（安装包/exe） | 内部、渠道上传 | Steam 等也先落到这里 |
| `download/{game}/...` | 官网对外稳定下载包 | 官网、用户 | 发布时从 app 晋升或单独打 |
| `download/launcher/` | GameLauncher 安装包 | 官网 | 跨游戏，单独放 |
| `config/{game}/` | 版本清单、热更配置 JSON | 客户端启动检查 | 例：`config/match3/version.json` |
| **`localization/`** | **多语言文案、字体、本地化表** | **客户端 / 导表流程** | 对应组织 [Localization](https://github.com/setsuodu/Localization) 组件产物；可按游戏分子目录，公共文案放 `localization/common/` |
| **`common/`** | **跨游戏静态资源（非 AB）** | **官网、启动器、多游戏共用** | 品牌图、法务文档、共享图标等；**不是** AssetBundle，也**不**按单一游戏隔离 |

### localization  vs  common

- **localization**：给**游戏运行时**用的多语言资源（文案 key、字体、按语言拆的表），通常由导表/Localization 管线产出，客户端按语言加载。
- **common**：给**官网 / 启动器 / 多游戏共用**的静态文件（Logo、隐私政策 HTML/PDF、通用宣传图），不进游戏热更包，也不按 match3/act 拆（除非你主动分子目录）。

两者都不是「某个游戏的 AB」，所以不放在 `ab/{game}/` 下。

## 路径示例

| 类型 | 路径 |
|------|------|
| match3 Android AB | `/ab/match3/android/xxx.ab` |
| act Windows AB | `/ab/act/windows/xxx.ab` |
| match3 Steam CI 包 | `/app/match3/windows/steam/Game_v1.0.0.exe` |
| match3 官网官包 | `/download/match3/windows/official/Game_Setup.exe` |
| GameLauncher | `/download/launcher/windows/GameLauncher_Setup.exe` |
| match3 版本清单 | `/config/match3/version.json` |
| 公共多语言 | `/localization/common/zh-CN.json` |
| match3 多语言 | `/localization/match3/en.json` |
| 隐私政策 | `/common/legal/privacy.html` |
| 品牌 Logo | `/common/images/logo.png` |

## 渠道说明

| 渠道目录 | 含义 |
|----------|------|
| `official` | 自有 / 官网 |
| `steam` | Steam（CI → `app/`；官网入口 → `download/`） |
| `bilibili` / `taptap` / `epic` … | 按实际上架渠道扩展 |

新游戏：在 `ab/`、`app/`、`download/`、`config/`、`localization/` 下各建目录即可。  
新渠道：在对应游戏 + 平台下新建目录即可，无需改 Nginx。
