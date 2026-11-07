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

include $(INCLUDE_DIR)/package.mk

define Package/bun
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Node.js
  TITLE:=Fast all-in-one JavaScript runtime & toolkit
  URL:=https://bun.sh/
  DEPENDS:=@HAS_FPU @(aarch64||x86_64)
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

define Build/Compile
endef

# npm tarball 解包后带 package/ 前缀：$(PKG_BUILD_DIR)/package/bin/bun
define Package/bun/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(CP) $(PKG_BUILD_DIR)/package/bin/bun $(1)/usr/bin/
endef

$(eval $(call BuildPackage,bun))
