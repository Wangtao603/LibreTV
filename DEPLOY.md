# LibreTV 部署说明（GitHub 纯静态）

本项目已改造为**纯静态**运行，可直接托管在 GitHub 上，无需任何后端。

## 一、在线地址

当前已上线（由 raw.githack.com 直接渲染 GitHub 仓库内容，`Content-Type: text/html`）：

**https://raw.githack.com/Wangtao603/LibreTV/cline/zr7yw6vw/**

> 该地址跟随分支 `cline/zr7yw6vw` 的最新提交，无需构建。

## 二、启用永久 GitHub Pages 地址（推荐，一步）

由于自动化账号缺少 `pages` 权限，需你手动开启一次（无需额外账号）：

1. 打开 https://github.com/Wangtao603/LibreTV/settings/pages
2. **Source** 选择 `Deploy from a branch`
3. **Branch** 选择 `cline/zr7yw6vw`，目录选 `/ (root)`，点击 **Save**
4. 等待约 1 分钟，访问：**https://wangtao603.github.io/LibreTV/**

仓库根目录已放置 `.nojekyll`，避免 Jekyll 处理导致资源缺失。

如果你更希望用 GitHub Actions 部署，把 `deploy/pages.yml` 复制到 `.github/workflows/pages.yml` 后推送即可
（需要具备 `workflows` 权限的账号推送）。

## 三、可用的视频源（重点）

GitHub Pages / githack 都是**纯静态**托管，浏览器只能直连**返回 CORS 头**的接口。
经批量实测 200+ 个公开苹果CMS源后，以下源满足「HTTPS + `Access-Control-Allow-Origin: *` + 关键词搜索可用」：

| 源 | 接口 |
| --- | --- |
| 暴风资源 | `https://bfzyapi.com` |
| 量子资源 | `https://cj.lziapi.com` |

这两个源已写入 `js/config.js` 的 `API_SITES`，并在设置中默认勾选。

> 其余绝大多数源（如意、天涯、极速、卧龙、虎牙、最大、百度云……）**不返回 CORS 头**，
> 浏览器直连会被拦截，必须经由服务端代理才能使用。

## 四、解锁全部源（可选）

若要使用全部 20+ 个源，只需自建一个代理，然后把地址填回 `js/config.js`：

```js
const PROXY_URL = 'https://你的代理域名/?url=';
```

要求：代理接收 **URL 编码后的目标地址**，原样返回响应体。例如一个最小的 Cloudflare Worker：

```js
export default {
  async fetch(request) {
    const target = new URL(request.url).searchParams.get('url');
    if (!target) return new Response('missing url', { status: 400 });
    const res = await fetch(target, { headers: { 'User-Agent': 'Mozilla/5.0' } });
    return new Response(res.body, {
      status: res.status,
      headers: {
        'Content-Type': res.headers.get('content-type') || 'application/json',
        'Access-Control-Allow-Origin': '*',
      },
    });
  },
};
```

设置 `PROXY_URL` 后，代码会自动从「直连」切换为「走代理」，`API_SITES` 里加回其它源即可全部可用。

## 五、已知限制

- **搜索 / 详情**：已实测可用（`暴风资源` 16 条、`量子资源` 18 条结果，详情可正确返回播放地址）。
- **播放**：播放器直连各源 CDN 的 m3u8。这些 CDN 普遍有**地域限制/防盗链**，
  本机（境外数据中心 IP）测试时全部返回 404；在能直连这些 CDN 的网络环境下可正常播放。
  若无法播放，请换源，或按第四节配置代理后经代理播放。
- **豆瓣热门**：依赖代理，已默认关闭（纯静态模式下不可用）。
