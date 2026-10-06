# LocalCDN data 目录约定

生产 OSS / CDN 保持相同路径，客户端与官网只换 BaseURL。

```
data/
├── ab/                              # AssetBundle（按平台，热更用）
│   ├── android/
│   ├── windows/
│   ├── ios/
│   └── osx/
├── app/                             # CI 构建产物落点（平台 × 渠道）
│   ├── android/
│   │   ├── official/
│   │   ├── steam/
│   │   ├── bilibili/
│   │   └── taptap/
│   ├── windows/
│   │   ├── official/
│   │   ├── steam/                   # Steam 构建产物（CI 上传，内部取包）
│   │   └── epic/
│   └── osx/
│       ├── official/
│       └── steam/
├── download/                        # 官网下载中心（对外稳定包）
│   ├── android/
│   │   ├── official/                # 官网 Android 官包
│   │   ├── bilibili/
│   │   └── taptap/
│   ├── windows/
│   │   ├── official/                # 官网 Windows 官包
│   │   ├── steam/                   # 官网「Steam 版」说明/引导（或备用包）
│   │   └── epic/
│   ├── osx/
│   │   ├── official/
│   │   └── steam/
│   └── launcher/                    # GameLauncher 安装包（官网下载）
│       ├── windows/
│       └── osx/
├── config/                          # 热更配置、版本清单
├── localization/                    # 多语言
└── common/                          # 公共资源
```

## 两类「包」的分工

| 目录 | 用途 | 谁写 | 谁读 |
|------|------|------|------|
| `app/` | CI 编译产物统一落点（含 Steam 构建） | CI | 内部取包、对账、灰度、再上传渠道 |
| `download/` | 官网下载中心对外稳定包 | 发布流程（从 app 晋升或单独打） | 官网页面、用户直链下载 |

官网页面链接示例（本地）：

```
http://localhost:12280/download/windows/official/GameName_Setup.exe
http://localhost:12280/download/android/official/GameName.apk
http://localhost:12280/download/launcher/windows/GameLauncher_Setup.exe
```

生产只换域名：

```
https://cdn.yourdomain.com/download/windows/official/GameName_Setup.exe
```

## AssetBundle 路径示例

| 类型 | 路径 |
|------|------|
| Android AB | `/ab/android/match3/xxx.ab` |
| Windows AB | `/ab/windows/match3/xxx.ab` |
| 版本清单 | `/config/match3/version.json` |

## 渠道说明

| 渠道目录 | 含义 |
|----------|------|
| `official` | 自有 / 官网 |
| `steam` | Steam（CI 产物进 `app/`；官网可放引导页或备用包进 `download/`） |
| `bilibili` / `taptap` / `epic` … | 按实际上架渠道扩展 |

新增渠道：在对应平台下新建目录即可，无需改 Nginx。
