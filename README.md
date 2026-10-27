# 青澜广告拦截 · 规则订阅源

轻览浏览器（qinglan）内置广告拦截模块的**规则订阅仓库**。
端上每日拉取一次 manifest，验签通过后下载规则包并热替换。

## 订阅地址（端上写死这两条，永久不变）

| 通道 | manifest 地址 |
|---|---|
| dev（debug 构建） | `https://github.com/fFfhhyy/qinglan-adblock-rules/releases/download/dev/manifest.dev.json` |
| stable（正式构建） | `https://github.com/fFfhhyy/qinglan-adblock-rules/releases/download/stable/manifest.stable.json` |

Rule pack 与 manifest 同 Release、同名 tag，地址由 manifest 内的 `packUrl` 指向。

## 目录

本仓库的 git 历史只保存文档；**规则产物全部以 Release 资产发布**（tag = 通道名）：

```
Release "dev" / "stable"
  ├── manifest.dev.json / manifest.stable.json   已签名 manifest
  ├── adslayer-rules.pack.gz                     规则包（659KB）
  └── factory-rules.pack.gz                      APK 内置出厂包（39.5KB，仅 stable 通道）
```

发布方式：本地跑 `tools/publish.ps1`（编译 → 回归裁判 → 打包 → 签名 → 上传），
不在本仓库手工提交产物。

## 规则来源

- EasyList China（社区，约 4 天更新一次）
- 乘风广告过滤规则（社区）
- 私有攻防档案（真机逆向取证：投放域、注入器路径、WebSocket 隧道、例外白名单）

编译与回归测试在本地完成，**本仓库只放签名后的发布产物**。
私钥、原始语料、逆向档案不在本仓库。

## 安全

- 所有 manifest 经 Ed25519 签名；端上内置公钥表，验签失败即拒绝
- stable 通道由离线主钥签名；dev 通道由测试钥签名，仅 debug 构建接受
- 端上校验 pack 的 sha256 与 size，不符即丢弃
- manifest 含 `minClientVersion` 与紧急 `revertTo: factory` 指令位
