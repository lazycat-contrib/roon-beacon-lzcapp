# Roon Beacon · LazyCat

Roon Beacon 的懒猫 LPK v2 包装仓库。发现同一局域网的 Roon Core，在 Web 界面浏览媒体库、搜索和控制播放，并通过实验性 AirPlay 接收器在浏览器收听。

- 上游源码与 Docker：<https://github.com/ca-x/roon-beacon>
- 包装仓库：<https://github.com/lazycat-contrib/roon-beacon-lzcapp>
- 包名：`community.lazycat.app.roon-beacon`
- 平台：LazyCat OS ≥ 1.5.0，Linux amd64。上游镜像另支持 arm64，本仓库的 LPK 目前以 amd64 为目标。
- 发布渠道：**仅喵喵商店**，不上传官方应用商店。采用镜像加速模式，发布前核验目标平台的镜像 digest 与 GHCR 源镜像一致。

## 使用

1. 安装后通过懒猫应用入口打开；该入口需要懒猫身份认证，并在网关侧注入应用令牌，无需手动登录。
2. 在 Roon「设置 → 扩展」启用 **Roon Beacon**，回到网页选择 Core 与播放区域。
3. 如需浏览器收听，在 Roon「设置 → 音频」启用 **Roon Beacon Web**，选择该输出并播放，再点击网页的「加入收听」。浏览器要求用户手势才能播放音频。
4. 界面支持中英切换、浅色 / 深色 / 跟随系统主题，设置保存在当前浏览器。

音频链路是 **Roon AirPlay → Shairport Sync → FFmpeg MP3 → 浏览器**，不是原生 RAAT / Roon Bridge。有损、有缓冲延迟，不保证多个浏览器同步。Roon Core 及其音乐来源仍由用户提供，本应用不是独立音乐库服务器。

## 网络与数据

`services.roon-beacon.network_mode: host` 是必需配置：SOOD 与 AirPlay/mDNS 要进入真实局域网。Web 监听 **39339/TCP**；懒猫入口使用 `http://host.lzcapp:39339/`，而不是 bridge 模式下的服务名。容器及懒猫入口的健康检查均使用新端口，覆盖上游镜像内置的 9339 检查。

应用保持后台运行且不启用多实例。不要在同一主机部署第二个使用相同端口 / AirPlay 接收端的实例。AirPlay 与发现协议所需的端口仍使用上游默认值（包括 5353/UDP 的 mDNS、5000/TCP 的 AirPlay 和 9003/UDP 的 SOOD）；39339 仅为 Web 端口。放行这些流量应限定可信局域网，不应转发至公网。跨 VPN 的 SOOD 转发不会自动转发 AirPlay。

授权、配置与状态持久化在 `/lzcapp/var/data`，映射容器 `/data`。入口令牌由 `stable_secret` 派生为 64 位十六进制，每次安装独立且重启稳定，不写入仓库、浏览器脚本或公开日志。只有经过懒猫认证的网关请求会注入令牌；局域网直接访问 `http://主机IP:39339` 仍需令牌，且不会自动登录。

首次安装可能需要等待局域网发现；检查 Roon 的扩展授权和音频输出配置。若服务无法启动，先检查 39339、5000 是否被其他服务占用，以及 host 网络的组播是否可达。共享 Core 的控制权限与音频面向有权打开此懒猫应用的用户，并非逐用户隔离播放器。

## 构建与发布

```sh
mkdir -p dist
lzc-cli project build -o dist/roon-beacon.lpk
```

GitHub Actions 每 6 小时检查 `ghcr.io/ca-x/roon-beacon` 的稳定 SemVer 标签，也可手动执行。手动运行默认 dry-run；关闭后才发布。流程核验 GHCR 与加速镜像的 Linux amd64 digest、构建并验证 LPK，再上传版本化 GitHub Release 资产，最后使用该资产的 URL 与本地计算的 SHA256 提交喵喵商店。正式 LPK 不提交 Git。

调用方显式传递组织 / 仓库 Secrets：`APPSTORE_URL`、`APPSTORE_TOKEN`，以及可选的 `PRIVATE_STORE_GROUP_CODES`。组织 Secrets 必须授权本仓库；同名仓库 Secret 会覆盖组织 Secret。镜像模式不需要 LazyCat 镜像复制凭据，禁止在配置文件中填写凭据。

本地直接打包不会执行镜像自动更新或镜像一致性核验，正式发布以 GitHub Actions 的结果为准。已有版本资产应复用并校验，不能重建覆盖后冒充同一个已发布版本。

## 品牌与许可

`icon.png` 由用户提供的 Roon Beacon 原始 Logo 缩放得到，未重新设计。应用及本仓库采用 GPL-3.0-only；React 组件的第三方许可证已包含在上游镜像与二进制中。本项目为社区工具，与 Roon 官方无隶属关系。
