# LocalCDN（FakeOSS）

本地开发用 Nginx，模拟 OSS / CDN。

生产环境资源走真实 CDN / OSS（阿里云 OSS、Cloudflare、自建对象存储等），**客户端只配一个 BaseURL**。  
本地开发用本仓库，生产直接换 URL，零代码改动。

> 组织总览： [LongLongGames](https://github.com/LongLongGames) · 端口规划见 [profile README §5](https://github.com/LongLongGames/.github/blob/main/profile/README.md)

## 端口

| 服务 | Host | Container |
|------|------|-----------|
| local-cdn | **12280** | 80 |

符合组织组件段 `12000–12999`（bugreport 120xx / mail 121xx，本仓占 122xx）。

## 快速开始

```bash
# 1. 把资源放进 data/（保持与生产 OSS 相同的目录结构）
mkdir -p data/match3/ab
# 例如：data/match3/ab/xxx.ab  data/common/config.json ...

# 2. 启动
docker compose up -d

# 3. 验证
curl http://localhost:12280/health          # → ok
open http://localhost:12280/                # 可浏览目录
```

客户端本地配置示例：

```
ResourceBaseUrl = "http://localhost:12280/"
```

生产环境只改配置：

```
ResourceBaseUrl = "https://cdn.yourdomain.com/"
# 或真实 OSS 域名
```

## 目录约定建议

```
data/
├── match3/
│   └── ab/          # AssetBundle
├── act/
│   └── ab/
├── common/
│   ├── config/
│   └── localization/
└── ...
```

路径尽量与生产 OSS 保持一致，这样客户端代码完全不用改。

## 配置说明

- `nginx/default.conf`：已开启
  - `autoindex`（本地方便看目录）
  - CORS（`Access-Control-Allow-Origin *`）
  - 缓存头（模拟 CDN，7 天 immutable）
  - `/health` 健康检查
- 挂载只读：`./data` → `/usr/share/nginx/html`

如需改端口，在 `.env` 或环境变量覆盖：

```bash
HOST_CDN_PORT=12280
```

## 关闭

```bash
docker compose down
```

## 为什么单独开仓

与 BugReport / Mail 一样，作为**可复用组件镜像**（这里直接用官方 nginx:alpine + 自定义 conf）。  
全公司本地开发共用一个实例即可，不占用任何游戏实例端口段。
