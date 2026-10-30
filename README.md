<div align="center">
<h1>OpenWrt — 云编译</h1>

## 特别提示

- **本人不对任何人因使用本固件所遭受的任何理论或实际的损失承担责任！**

- **本固件禁止用于任何商业用途，请务必严格遵守国家互联网使用相关法律规定！**

## 项目说明
- 默认管理地址：**`192.168.2.1`**，默认用户：**`root`**，默认密码：**`none`**，默认设备名：**`Zhe28`**
- LuCI 页脚署名为 `Built by Zhe28 <构建时间>`，并链接到本仓库 [Releases](https://github.com/Zhe28/openwrt-ci-roc/releases)
- [云编译来源](https://github.com/haiibo/OpenWrt) [视频教程](https://www.youtube.com/watch?v=6j4ofS0GT38) [问题合集](https://github.com/LiBwrt/openwrt-6.x/issues)

## 仓库说明
- 本人 fork 的仓库：[ImmortalWrt](https://github.com/laipeng668/immortalwrt) [LibWrt](https://github.com/laipeng668/openwrt-6.x)，内容大体一致。
- `ImmortalWrt` 和 `LibWrt` 分别通过 rebase 和 merge 进行更新，相互印证。
- `LibWrt` 因为 DTS 更为丰富，所以支持更多的机型。

## 编译机型
- **京东云雅典娜 `jdcloud_re-cs-02`**（SOC `ipq6010`，无线芯片 QCN9074 `pci17cb,1104`，512MB 内存 / 128GB eMMC），共三条流水线：`IPQ60XX-LibWrt`、`JDCloud-ImmortalWrt`、`JDCloud-NoWiFi-ImmortalWrt`（NoWiFi 版把 ath11k 全部关闭）。
- **x86-64**（`configs/x86-64.config`，输出 ISO / QCOW2 / VDI / VMDK / VHDX）。
- 雅典娜的 LuCI 由 **nginx** 承载（`nginx-full` + `nginx-mod-ubus` + `nginx-mod-luci` + `uwsgi` + `uwsgi-luci-support`），`uhttpd` 作为兜底保留；x86-64 仍用普通 `uhttpd`。
- 已精简掉、不再编译的机型：`ipq60xx` 子目标里除雅典娜以外的全部机型（含京东云亚瑟 `jdcloud_re-ss-01`、`jdcloud_re-cs-07`、红米 `redmi_ax5` 等），以及 IPQ807X 全系列编译。`configs/IPQ807X.config` 与 `IPQ807X-LibWrt.yml` 保留，但已从 `Trigger-All-Workflows` 里注释掉；需要时把该 workflow 的 `ipq807x_libwrt` 开关打开即可单独跑。
- 想加回机型时在对应 config 里写 `CONFIG_TARGET_DEVICE_qualcommax_ipq60xx_DEVICE_<厂商>_<型号>=y`，名称必须与上游 `target/linux/qualcommax/image/ipq60xx.mk` 的 `TARGET_DEVICES` 完全一致。
- 官方 ImmortalWrt 源里**没有雅典娜**这个机型，所以设备定义依赖 `laipeng668` 的 fork。

## 定时与手动触发
- `Trigger-All-Workflows`：定时 `0 18 * * 5`（周五 18:00 UTC，北京时间周六 02:00）串起 `IPQ60XX-LibWrt`、`JDCloud-ImmortalWrt`、`JDCloud-NoWiFi-ImmortalWrt`、`x86-64-ImmortalWrt` 与 `Build-Packages`；手动运行时每条流水线都有独立的布尔开关。
- `IPQ60XX-LibWrt` 另有独立定时 `0 0 * * 6`（北京时间周六 08:00）。
- 各机型 wrapper 同时支持 `workflow_dispatch`，只想编一台机器时可以直接运行它自己的 yml；`Build-OpenWrt.yml` 是 `workflow_call` 复用工，不能单独运行。
- `Build-On-Like`：在标题为 `👍 点赞编译触发器` 的 Issue 下由仓库 OWNER 发表评论，即可触发一次 `IPQ60XX-LibWrt` 构建。
- 各 wrapper 自身的 `schedule` 已注释掉，避免与 `Trigger-All-Workflows` 重复构建。

## 内核与官方软件源
- kmod 类包的依赖被写成 `kernel (= <版本>~<vermagic>-r<release>)`（见上游 `include/kernel.mk:216`），而 `vermagic` 是内核 `.config` 中所有 `=y/m` 行排序后的 MD5（`include/kernel-defaults.mk:129`）。**版本号、内核配置哈希、release 三样必须同时和软件源一致**，否则 `apk` 安装任何 `kmod-*` 都会失败。
- 路由器里的 kmods 源地址同样由这三样拼出（`include/feeds.mk:41` / `:55`）：`<仓库>/targets/<target>/<subtarget>/kmods/<版本>-<release>-<vermagic>/packages.adb`。
- 因此 `JDCloud.config` 与 `JDCloud.NoWiFi.config` 关闭 `CONFIG_TESTING_KERNEL`，内核回到 `6.12.112`，与官方 `25.12-SNAPSHOT` 的 `kmods/6.12.112-1-<哈希>` 对齐；`General.config` 里也不允许出现会写入内核 `.config` 的 `CONFIG_KERNEL_*` 覆盖项（已移除 ZRAM 后端那三行）。
- 已知差异：`IPQ60XX-LibWrt` 因 LibWrt 自身改动内核配置，vermagic 与官方快照不同，**无法直接安装官方源的 kmod**；需要 kmod 时请走 `JDCloud-ImmortalWrt` / `JDCloud-NoWiFi-ImmortalWrt`。

## 定制固件
- 首先要登录 Github 账号，然后 fork 此项目到你自己的 Github 仓库。
- 修改 `configs` 目录对应的文件添加或删除插件，或者上传自己的 `xx.config` 配置文件。
- 不需要的软件包请把 `y` 改成 `n` ，仅在前面添加 `#` 是无效的。
- 插件对应名称及功能请参考恩山网友帖子：[OpenWrt软件包全量解释](https://www.right.com.cn/FORUM/forum.php?mod=viewthread&tid=8384897)。
- 如需修改默认 IP、添加或删除插件包以及一些其他设置请在 `scripts/Roc-script.sh` 文件内修改。
- 固件构建只会拉取设备配置和 `configs/General.config` 中实际启用的第三方软件包，并始终使用对应分支的最新提交。
- `configs/General.config` 对**全部固件构建**生效，目前内置的数据库与运行时服务：`redis-server` / `redis-cli`、`pgsql-server` / `pgsql-cli`（PostgreSQL 17.5）、`mariadb-server` / `mariadb-client`（官方源里没有 MySQL 本体，`mariadb-server` 通过 `PROVIDES:=mysql-server` 顶下这个名字）、`node` + `node-npm`（产物名带 ABI，即 `node127`）、`python3` + `python3-pip`。这些包都是用户态，不受内核 vermagic 影响。
- 雅典娜只有 512MB 内存：数据库与 Node/Python 服务默认开机自启，同时全开容易 OOM，请按实际用途只启用需要的几个。
- 每次固件构建都会记录第三方仓库的实际分支和 commit，并在 Release 中附带 `<固件前缀>.third-party-sources.txt` 供核对；该记录文件不会写入固件。
- 添加或修改 `xx.yml` 文件，最后点击 `Actions` 运行要编译的 `workflow` 即可开始编译。
- 内置 node / python3 / mariadb 之后单次固件编译约 2-4 小时，`Build-OpenWrt` 的 `timeout-minutes` 为 360；编译完成后在仓库主页 [Releases](https://github.com/Zhe28/openwrt-ci-roc/releases) 对应 Tag 标签内下载固件。

## 单独编译软件包
- 点击 `Actions` 运行 `Build-Packages`，`sdk_version` 可选择 `ALL` 同时编译全部版本，或选择 `main` 主线 snapshots、`23.05`、`24.10`、`25.12` 系列的最新稳定版 SDK。
- 默认同时编译 `x86-64` 和 `aarch64` 两个架构：`x86/64` 使用 `configs/x86-64.config + configs/Packages.config`；`aarch64` 在 `main`、`25.12` 使用 `configs/JDCloud.config + configs/Packages.config`，在 `23.05`、`24.10` 使用 SDK 脚本内置的 rax3000m 配置 + `configs/Packages.config`。
- `package` 默认是 `ALL`，下拉里除了 `nginx` 和 `luci-app-aria2`、`luci-app-frpc`、`luci-app-frps`、`luci-app-gecoosac`、`luci-app-lucky`、`luci-app-openlist2`、`luci-theme-argon`、`luci-theme-aurora` 这些 LuCI 入口，还新增了 `redis`、`pgsql`、`mariadb`、`node`、`python3`；选择 LuCI 软件包时会同时编译并发布对应基础包或主题配置插件，其中 `luci-app-aria2` 会一并处理 `aria2` 和 `ariang`。旧的 `aria2`、`ariang`、`frp`、`gecoosac`、`lucky`、`openlist2` 输入仅作为兼容别名保留，`postgres`/`postgresql`、`mysql`、`nodejs`/`npm`、`python` 同样是别名。
- `node` 和 `python3` 是重编译包（`node` 单个就要 40~90 分钟），`Build-Packages` 的 `timeout-minutes` 已相应提到 360。只想编一个时请把 `package` 选成对应项而不是 `ALL`。
- 脚本会按每个矩阵的版本和架构从 `https://downloads.openwrt.org/` 自动查找对应 SDK，先验证官方签名和 SHA-256，再在本次构建中使用同一组精确 URL 与哈希，避免 snapshots 更新造成前后不一致。
- SDK 签名信任锚来自 OpenWrt 官方 [`openwrt/keyring`](https://github.com/openwrt/keyring) 密钥仓库的构建公钥，仓库内固定主指纹为 `8A8BC12F46B836C0F9CDB36F1D53D1877742E911`；未知签名会直接终止构建，官方换钥时需人工核对后更新公钥。
- `Build-Packages` 不使用 GitHub Actions 持久缓存；SDK 每次构建都会重新下载并验证签名和 SHA-256，软件包源码也会重新拉取。
- 实际编译的软件包会参考 `configs/Packages.config` 里的软件包选项，例如 `aria2`、`ariang`、`frpc`、`frps`、`nginx-full`、`lucky`、`luci-app-gecoosac`、`luci-app-argon-config`、`luci-app-aurora-config`、`luci-app-lucky`、`luci-app-openlist2`、`luci-theme-argon`、`luci-theme-aurora`，以及 `redis-server` / `redis-cli`、`pgsql-server` / `pgsql-cli` / `libpq`、`mariadb-server` / `mariadb-client`、`node`（产物 `node127`）/ `node-npm`、`python3` / `python3-pip`。
- `redis`、`pgsql`、`mariadb`、`python3` 直接编译自 SDK 自带的官方 packages feed，本仓库不 fork、不覆盖版本。
- `node` 是个例外：**官方 OpenWrt packages feed 里没有 node 运行时**（`lang/node` 下只有一个 `node-javascript-obfuscator`），node 只存在于 `immortalwrt/packages`，所以脚本会 sparse clone `immortalwrt/packages` 的 `openwrt-25.12` 分支里的 `lang/node/node`，做法与 `nginx` 一致。
- 软件包产物里只包含所选分组本身，不包含依赖包；`libc`、`libopenssl`、`resolveip`、`sudo` 之类的依赖仍需从软件源补齐。
- 编译的软件包来源及跟踪分支如下；每次构建都会拉取对应分支的最新提交，并把实际 commit 写入 `BUILDINFO.json`，不会固定第三方源码版本：
  - `https://github.com/laipeng668/packages` 的 `aria2` 分支：`net/aria2`
  - `https://github.com/laipeng668/packages` 的 `ariang` 分支：`net/ariang`
  - `https://github.com/laipeng668/packages` 的 `frp-binary` 分支：`net/frp`
  - `https://github.com/laipeng668/packages` 的 `nginx` 分支：`net/nginx`
  - `https://github.com/laipeng668/luci` 的 `frp` 分支：`applications/luci-app-frpc`、`applications/luci-app-frps`
  - `https://github.com/laipeng668/luci-app-gecoosac` 的 `main` 分支：`gecoosac`、`luci-app-gecoosac`
  - `https://github.com/gdy666/luci-app-lucky` 的 `main` 分支：`lucky`、`luci-app-lucky`
  - `https://github.com/laipeng668/luci-app-openlist2` 的 `main` 分支：`openlist2`、`luci-app-openlist2`
  - `https://github.com/immortalwrt/packages` 的 `openwrt-25.12` 分支：`lang/node/node`（官方 feed 没有 node 运行时，只有 ImmortalWrt 的 feed 有）
  - `https://github.com/jerrykuku/luci-theme-argon` 的 `master` 分支：`luci-theme-argon`
  - `https://github.com/jerrykuku/luci-app-argon-config` 的 `master` 分支：`luci-app-argon-config`
  - `https://github.com/eamonxg/luci-theme-aurora` 的 `master` 分支：`luci-theme-aurora`
  - `https://github.com/eamonxg/luci-app-aurora-config` 的 `master` 分支：`luci-app-aurora-config`
- 编译产物会按 `<SDK>-<软件包>-<架构>.zip` 分组打包，上传到本次 workflow 的 `Artifacts`，并发布到 `Packages` 这个 [Releases](https://github.com/Zhe28/openwrt-ci-roc/releases/tag/Packages)；普通分组内的 `.apk/.ipk` 文件也使用对应架构后缀。`luci-theme-argon` 和 `luci-theme-aurora` 是通用主题包，使用 `all` 架构后缀，`arch=ALL` 时各自只保留一份。下载后先解压，`apk` 使用 `apk add --allow-untrusted *.apk` 安装，`ipk` 使用 `opkg install *.ipk` 安装。
- 编译矩阵中部分任务失败时，工作流仍会把其他成功任务的产物发布到 `Packages` Release；如果没有任何成功产物，则跳过 Release 上传。用户主动取消工作流时不会继续发布。
- 单个矩阵内的软件包会独立记录编译结果；部分软件包失败时，已成功的软件包仍会打包上传，同时该矩阵任务保持失败状态。每个 ZIP 内包含 `BUILDINFO.json` 和 `SHA256SUMS`，用于核对 SDK、源码版本、编译结果和文件哈希。

## 页面预览
![Homepage](Homepage.png)
</div>
