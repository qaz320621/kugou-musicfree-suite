<p align="center">
  <img src="https://img.shields.io/badge/酷狗音乐-自由套件-2F6FED?style=for-the-badge&logo=music" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/平台-Windows%20%7C%20Android-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/语言-TypeScript%20%7C%20Python%20%7C%20Java-orange?style=flat-square" />
  <img src="https://img.shields.io/badge/许可证-MIT-green?style=flat-square" />
  <img src="https://img.shields.io/badge/状态-可用-brightgreen?style=flat-square" />
</p>

<h1 align="center">🎵 酷狗音乐自由套件</h1>

<p align="center">
  <b>基于抓包逆向的酷狗音乐全平台方案</b>：<code>MusicFree 插件</code> ＋ <code>扫码登录 Token 服务</code>（PC / Android）
</p>

<p align="center">
  关键词：`抓包分析` · `TLS 解密` · `签名逆向` · `扫码登录` · `MusicFree 插件`
</p>

---

## 📌 依赖关系

> 本仓库为**父项目**：仅作总览与组织，**不包含任何子项目工程代码**。三个子项目相互独立，可单独 clone / 单独使用。

```
                    ┌─────────────────────────────────────────────┐
                    │        kugou-musicfree-suite（父项目）        │
                    │  总览文档 · 不包含工程 · 无代码               │
                    └───────────────────┬─────────────────────────┘
                                        │  组织与说明
        ┌───────────────────────────────┼───────────────────────────────┐
        ▼                               ▼                               ▼
┌───────────────────┐   ┌────────────────────────┐   ┌────────────────────────┐
│ kugou-musicfree-  │──▶│  kugou-token-server-pc  │   │ kugou-token-server-     │
│      plugin       │   │   （PC 版 Token 服务）   │   │      android            │
│  MusicFree 插件    │   └────────────────────────┘   │  （Android 版 Token 服  │
└───────────────────┘            ▲    ▲               │         务）            │
         │                       │    │               └────────────────────────┘
         │  运行时依赖（二选一）    │    │                          ▲
         │  http://127.0.0.1:8765/token                         │
         └───────────────────────┴────┴──────────────────────────┘
```

| 项目 | 角色 | 依赖方向 |
|---|---|---|
| [kugou-musicfree-plugin](https://github.com/qaz320621/kugou-musicfree-plugin) | MusicFree 插件（搜索/播放/歌词） | **运行时依赖** Token 服务（无 token 无法播放） |
| [kugou-token-server-pc](https://github.com/qaz320621/kugou-token-server-pc) | PC 版扫码登录 Token 服务（Python） | 无上游业务依赖；**被插件调用** |
| [kugou-token-server-android](https://github.com/qaz320621/kugou-token-server-android) | Android 版扫码登录 Token 服务（Java） | 无上游业务依赖；**被插件调用** |

---

## ✨ 特性

- ✅ **MusicFree 插件**：搜索 / 播放 / 歌词，`dist/plugin.js` 即插即用
- ✅ **扫码登录（中间人）**：展示酷狗官方二维码 → 手机扫码 → **token 自动捕获保存**，无需手动复制
- ✅ **双平台 Token 服务**：PC（Python）与 Android（NanoHTTPD + ZXing）功能等价
- ✅ **签名逆向**：酷狗 API 签名算法完整还原（`MD5(KEY + 排序参数 + KEY)`）
- ✅ **免费歌曲过滤**：自动跳过 VIP 受限歌曲（`privilege` 0/8）
- ✅ **脱敏开源**：不内置任何账号信息，token 仅存本地

---

## 🏗 架构

```
┌────────────┐    搜索(免登录)      ┌──────────────────┐
│  MusicFree │ ───────────────────▶ │ mobilecdn.kugou  │
│   插件      │                      │  /search/song    │
└─────┬──────┘                      └──────────────────┘
      │ 播放(需token)      ┌──────────────────┐
      ├──────────────────▶│ wwwapi.kugou.com │──▶ webfs.kugou.com（MP3 直链）
      │                   │ /play/songinfo   │──▶ 歌词 / 封面
      │                   └──────────────────┘
      │  拉取 token
      ▼
┌─────────────┐  扫码登录(中间人)   ┌──────────────────────┐
│ Token 服务   │ ◀──────────────── │ login-user.kugou.com │
│ (PC/Android)│    v2/qrcode       │  v2/get_userinfo_    │
└─────────────┘   轮询 status      │  qrcode              │
                                   └──────────────────────┘
```

---

## 🚀 快速开始

| 步骤 | 操作 | 子项目 |
|---|---|---|
| ① | 构建插件 `npm run build` → 导入 MusicFree | [kugou-musicfree-plugin](https://github.com/qaz320621/kugou-musicfree-plugin) |
| ② | 启动 Token 服务（PC 或 Android 二选一） | [pc](https://github.com/qaz320621/kugou-token-server-pc) / [android](https://github.com/qaz320621/kugou-token-server-android) |
| ③ | 打开 `http://127.0.0.1:8765/` → 点「扫码登录」→ 手机酷狗 App 扫码确认 | Token 服务 |
| ④ | 插件搜索播放，token 自动生效 | - |

---

## 🔬 逆向原理（摘要）

酷狗网页版**强制登录**才能获取播放地址（未登录返回 `err_code 30020`），社区插件的免登录接口已失效。本套件通过：

1. **TLS 解密**：`SSLKEYLOGFILE` + Wireshark 全链路还原网页版请求
2. **签名逆向**：从 `infSign.min.js` / `kguser.v2.min.js` 提取密钥
   `NVPh5oo715z5DIWAeQlhMDsWXXQV4hwt`，算法 `MD5(KEY + 排序参数k=v + KEY)`（已用抓包真实值验证）
3. **扫码登录中间人**：`v2/qrcode` 获取官方凭证 → 二维码内容为官方 H5 URL →
   手机 App 扫码打开官方确认页 → 后端轮询 `v2/get_userinfo_qrcode`（status 1→2→4）→
   status=4 返回 `token + userid` → 自动保存

---

## 📦 子项目

| 子项目 | 说明 | 技术栈 |
|---|---|---|
| [kugou-musicfree-plugin](https://github.com/qaz320621/kugou-musicfree-plugin) | MusicFree 插件：搜索/播放/歌词 | TypeScript + Parcel |
| [kugou-token-server-pc](https://github.com/qaz320621/kugou-token-server-pc) | PC 扫码登录 Token 服务 | Python + qrcode |
| [kugou-token-server-android](https://github.com/qaz320621/kugou-token-server-android) | Android 扫码登录 Token 服务 | Java + NanoHTTPD + ZXing |

---

## 📄 许可证

[MIT](./LICENSE)

---

## ⚠️ 免责声明

- 本项目仅供**个人学习与研究**，请勿用于商业用途
- 酷狗音乐及其内容版权归酷狗音乐所有
- 使用本项目造成的一切后果由使用者自行承担
