# 轻览广告拦截 · 规则订阅源

轻览浏览器（qinglan）内置广告拦截模块的**规则订阅仓库**。
端上在冷启动时检查 manifest（最短间隔 15 分钟），验签通过且版本变化时下载规则包并热替换。

> 为什么是 15 分钟而不是每天一次：manifest 只有约 1.2KB，而 87KB 的 quicklist
> 只在版本变化时才下载 —— 所以"多久检查一次"的真实成本只有那 1.2KB。
> 早先按 24 小时节流，导致发布一条修复后设备最长要等一天才生效。

## 订阅地址（端上写死这两条，永久不变）

| 通道 | manifest 地址 |
|---|---|
| dev（debug 构建） | `https://github.com/fFfhhyy/qinglan-adblock-rules/releases/download/dev/manifest.dev.json` |
| stable（正式构建） | `https://github.com/fFfhhyy/qinglan-adblock-rules/releases/download/stable/manifest.stable.json` |

> 当前端上实际走的是 `raw.githubusercontent.com/.../main/dist/manifest.<channel>.json`。
> 上面的 Releases 路径是另一条可选分发路线，两条都保留。

Rule pack 与 manifest 同 Release、同名 tag，地址由 manifest 内的 `packUrl` 指向。

## 目录

本仓库的 git 历史只保存文档；**规则产物以 Release 资产 / `dist/` 发布**（文件名按通道独立，两条通道互不覆盖）：

```
dist/
  manifest.dev.json / manifest.stable.json        已签名 manifest
  adslayer-rules.dev.pack.gz                      规则包（dev 通道）
  adslayer-rules.stable.pack.gz                   规则包（stable 通道，约 659KB）
  quicklist.dev.json / quicklist.stable.json      端上热更新用的精简表（约 87KB）
  factory-rules.pack.gz                           APK 内置出厂包（约 40KB，仅 stable）
```

端上只下载 `quicklist.<channel>.json`（约 87KB），完整的 `adslayer-rules` 包（659KB）
不作为常规路径下载。

发布方式：本地跑 `tools/publish.ps1` ——
**编译 → 规则包裁判 → 真机 DOM 回归 → 打包 → 签名 → 验签校验 → 提交推送**，
两道门任一变红就拒绝发布。不在本仓库手工提交产物。

## 规则来源

- EasyList China（社区，约 4 天更新一次）
- 乘风广告过滤规则（社区）
- 私有攻防档案（真机逆向取证：投放域、注入器路径、WebSocket 隧道、例外白名单）

编译与回归测试在本地完成，**本仓库只放签名后的发布产物**。
私钥、原始语料、逆向档案不在本仓库。

### 收录原则（防误杀）

按域拦截前必须先过四条判据，否则会把"承载内容的基础设施"当成"广告本身"：

1. 该域是否与某个常访站点**反复共现**？是 → 疑似内容 CDN，禁止按域拦
2. 该域是否**服务多站点**？是 → 可能是公共 CDN，更需谨慎
3. 该域根路径返回什么？404 / nginx 默认页 / 空 body → 更像广告端点
4. 只有当域出现在**广告位 DOM 结构**里才按域拦；否则改按结构判据（容器类名 / 链接目标域）

真实案例：`t4.qy0.ru` 同时供 manhwa18 的作品封面与 Zone 广告；`bcebos.com`（百度对象存储）
承载很多站点的正文图片 —— 按域封都会造成整页无图。

## 安全

- 所有 manifest 经 Ed25519 签名；端上内置公钥表，验签失败即拒绝
- stable 通道由离线主钥签名；dev 通道由测试钥签名，仅 debug 构建接受
- 端上校验 quicklist 的 sha256 与 size，不符即丢弃
- **任何结构异常（字段缺失以外的情况）都整包拒绝**，绝不"部分生效" ——
  否则格式损坏的包会让放行名单静默清空，导致正常站点被误杀
- manifest 含 `minClientVersion` 与紧急 `revertTo: factory` 指令位
