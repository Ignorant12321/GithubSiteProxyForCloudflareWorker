# GitHubSiteProxyForCloudflareWorker

这是一个基于 Cloudflare Workers 的 GitHub 代理项目。
仓库里有两份主要实现：

- `src/snippet.js`：基础版
- `src/snippet-improved.js`：改进版，在基础版之上增加首页、白名单转换页和接口化转换能力

这份 README 以 `src/snippet.js` 为基线，说明 `src/snippet-improved.js` 的改进点和对应使用方式。

## 项目特点

- 支持 GitHub 常见资源的域名映射和代理访问
- 使用白名单限制可代理的原始域名
- 自动把原始域名转换为 `*-gh.` 前缀的代理域名
- 处理 GitHub 页面中的文本重写和重定向地址改写
- 处理部分嵌套 URL 路径，减少仓库页面跳转异常
- 改进版提供首页入口和 `/api/convert` 转换接口

## 文件说明

- `src/snippet.js`：基础代理实现
- `src/snippet-improved.js`：改进版实现，包含 `HOME_PREFIX`、`/go` 和 `/api/convert`
- `home.html`：改进版首页的静态页面参考
- `wrangler.toml`：Cloudflare Workers 配置

## 域名白名单

基础版和改进版都使用同一组白名单：

- `github.com`
- `avatars.githubusercontent.com`
- `github.githubassets.com`
- `collector.github.com`
- `api.github.com`
- `raw.githubusercontent.com`
- `gist.githubusercontent.com`
- `github.io`
- `assets-cdn.github.com`
- `cdn.jsdelivr.net`
- `securitylab.github.com`
- `www.githubstatus.com`
- `npmjs.com`
- `git-lfs.github.com`
- `githubusercontent.com`
- `github.global.ssl.fastly.net`
- `api.npms.io`
- `github.community`
- `desktop.github.com`
- `central.github.com`

如果输入的链接不在白名单中，转换会失败。

## 基础版行为

`src/snippet.js` 会：

1. 识别 `*-gh.` 形式的代理域名
2. 根据前缀解析回原始 GitHub 域名
3. 强制使用 HTTPS
4. 将请求转发到源站
5. 重写响应中的链接、重定向和部分文本内容
6. 修复部分 `latest-commit` / `tree-commit-info` 之类的嵌套 URL 路径

## 改进版行为

`src/snippet-improved.js` 在基础版上额外增加了：

- `home-gh.<你的域名>` 首页入口
- `/go?url=...` 直接跳转
- `/api/convert?url=...` 返回 JSON 格式的代理链接
- 首页里内置的白名单展示
- 本地输入后直接生成代理链接、复制链接、打开链接的交互

改进版的首页转换逻辑会优先使用本地白名单判断，在线运行时则调用 `/api/convert`。

## 使用方法

### 基础版

访问形式通常是：

```text
https://github-com-gh.<你的域名>/owner/repo
https://raw-githubusercontent-com-gh.<你的域名>/owner/repo/main/file.js
```

### 改进版首页

访问：

```text
https://home-gh.<你的域名>/
```

在输入框里填入原始链接，例如：

```text
https://github.com/owner/repo
https://raw.githubusercontent.com/owner/repo/main/file.js
https://gist.githubusercontent.com/user/id/raw/file
```

然后可以：

- 点击 `转换` 生成代理链接
- 点击 `复制` 复制结果
- 点击 `打开` 直接打开代理页

### 改进版接口

改进版提供：

```text
/api/convert?url=https://github.com/owner/repo
```

返回示例：

```json
{
  "ok": true,
  "proxy_url": "https://github-com-gh.<你的域名>/owner/repo"
}
```

## 部署说明

如果你部署的是基础版，把 Snippet入口指向 `src/snippet.js`。

如果你部署的是改进版，把 Snippet入口指向 `src/snippet-improved.js`。

示例：

```toml
name = "gh"
main = "src/snippet-improved.js"
compatibility_date = "2024-04-06"
```

建议配置的路由：

- `home-gh.<你的域名>/*`
- `*-gh.<你的域名>/*`

## 嵌套路径处理

项目对以下路径做了特殊截断处理：

```text
/owner/repo/latest-commit/main/https://...
/owner/repo/tree-commit-info/main/https://...
```

这样可以减少 GitHub 仓库页面中嵌套链接导致的异常跳转。

## 注意事项

- 项目不保证所有 GitHub 页面都能完全兼容
- 登录、注册等敏感流程不建议作为重点支持目标
- 使用前请确认符合当地法律法规和 GitHub 服务条款

## 许可证

见 [LICENSE](./LICENSE)。
