# bun 运行时改用 Bun 官方发布在 npm 上的预编译 musl 二进制包
# (@oven/bun-linux-*-musl：内容与 GitHub release 的 bun 二进制相同，包内仅 bin/bun)。
# tar.gz 走 OpenWrt 原生 Download 解包+校验，不依赖 unzip 宿主工具
# （OpenWrt tools/zip 只安装 zip，不安装 unzip）。
# 版本钉死 1.4.4，与 node wrapper 同款策略；升版本时同步换 PKG_HASH。
# 与 node 共存：feed 的 node-* 模块包仍走 node(ABI127)，bun 是路由器上直接用的
# 运行时+包管理器(bun install/bun run)。回退方法：Roc-script/SDK-script 停用本目录。
include $(TOPDIR)/rules.mk

PKG_NAME:=bun
PKG_VERSION:=1.4.4
PKG_RELEASE:=1

PKG_LICENSE:=MIT

# aarch64 保持原名 / x86_64 -> x64（与 @oven 包名一致）
BUN_CPU:=$(subst x86_64,x64,$(ARCH))

PKG_SOURCE:=bun-linux-$(BUN_CPU)-musl-$(PKG_VERSION).tgz
PKG_SOURCE_URL:=https://registry.npmjs.org/@oven/bun-linux-$(BUN_CPU)-musl/-/
PKG_HASH:=20b64dd9e5364e80fc965bed24a052bd21d3ffcd169af720988063c17d275edd
ifeq ($(BUN_CPU),x64)
  PKG_HASH:=284a0bfe9bd624d54c0f20e31521b26deb5c04a2c927dc2804fecd49339a948a
endif

PKG_BUILD_DIR:=$(BUILD_DIR)/bun-$(PKG_VERSION)

# OpenWrt 默认解包（include/unpack.mk:6）是 tar -C $(PKG_BUILD_DIR)/.. -xf -，
# 不剥顶层目录，要求 tar 内顶层目录名与 PKG_BUILD_DIR 一致。npm tarball 的顶层
# 目录固定是 package/，与 bun-<版本> 不匹配，默认解包会把内容落在 build_dir 下
# 的兄弟目录 package/ 里。改用官方 UNPACK_CMD 钩子边解包边剥一层，直接落进
# PKG_BUILD_DIR（先例：openwrt/packages 的 debootstrap、ariang 同款写法）。
UNPACK_CMD=$(STAGING_DIR_HOST)/bin/libdeflate-gzip -dc $(DL_DIR)/$(PKG_SOURCE) | $(TAR) -C $(1) -x --strip-components=1 -f -

include $(INCLUDE_DIR)/package.mk

define Package/bun
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Node.js
  TITLE:=Fast all-in-one JavaScript runtime & toolkit
  URL:=https://bun.sh/
  # libstdc++.so.6 是 bun 的真实动态依赖（run17 打包期依赖检测确认）。
  # 包符号名是 libstdcpp（main tree package/libs/toolchain，NAME:=libstdc++），
  # 不存在的写法 libstdcxx 会让 kconfig 报 "does not exist" 且不安装（run18 死因）。
  DEPENDS:=@HAS_FPU @(aarch64||x86_64) +libstdcpp
endef

define Package/bun/description
  Bun is a fast all-in-one JavaScript runtime: a single binary that runs
  JavaScript/TypeScript, installs npm packages, runs scripts and tests.
  This package ships the official musl prebuilt binary from Bun's npm
  distribution (@oven/bun-linux-*-musl); no cross-compilation involved.
endef

# 预编译二进制：解包即完成，无 configure/compile
define Build/Configure
endef

# run17 打包期依赖检测报缺 libc.musl-aarch64.so.1：bun 官方 musl 二进制在
# Alpine 构建，DT_NEEDED 用 Alpine 的 musl SONAME（libc.musl-<arch>.so.1），
# OpenWrt 的 musl 提供的是 libc.so，OpenWrt 打包按 DT_NEEDED 查提供者查不到。
# 用 patchelf 改写 DT_NEEDED 为 libc.so（tools/patchelf 是 main tree 自带
# 宿主工具；node 的 unofficial-builds 二进制本身就链 libc.so 所以没这问题）。
define Build/Compile
	$(STAGING_DIR_HOST)/bin/patchelf --replace-needed libc.musl-$(ARCH).so.1 libc.so $(PKG_BUILD_DIR)/bin/bun
endef

# UNPACK_CMD 已剥掉 npm 的 package/ 顶层目录，二进制就在 PKG_BUILD_DIR/bin 下
define Package/bun/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(CP) $(PKG_BUILD_DIR)/bin/bun $(1)/usr/bin/
endef

$(eval $(call BuildPackage,bun))
