# node 运行时改用 Node.js 社区 unofficial-builds 的 musl 预编译二进制，
# 不再在 CI 里交叉编译 V8（单组约 3 小时 -> 几分钟）。
# 包骨架（node / node-npm 符号、ABI_VERSION=127 产物名 node127、host 官方 x64 二进制）
# 与 immortalwrt/packages lang/node/node 保持一致，configs 与产物过滤器零改动。
# 回退方法：Roc-script/SDK-script 中停用本目录即恢复 feeds 源码编译。
include $(TOPDIR)/rules.mk

PKG_NAME:=node
PKG_VERSION:=22.23.4
PKG_RELEASE:=1

PKG_MAINTAINER:=Hirokazu MORIKAWA <morikw2@gmail.com>
PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE
PKG_CPE_ID:=cpe:/a:nodejs:node.js

# aarch64 -> arm64 / x86_64 -> x64（与 feeds 版一致的命名映射）
NODEJS_CPU:=$(subst aarch64,arm64,$(subst x86_64,x64,$(ARCH)))

PKG_SOURCE:=$(PKG_NAME)-v$(PKG_VERSION)-linux-$(NODEJS_CPU)-musl.tar.gz
PKG_SOURCE_URL:=https://unofficial-builds.nodejs.org/download/release/v$(PKG_VERSION)/
PKG_HASH:=78acbffed1d4095e8b4db55eb8b43ae928b2d41d14ab2ade9ee46a1ca6e677df
ifeq ($(NODEJS_CPU),x64)
  PKG_HASH:=117eae38a39e45d534f1a18c2217709dc0b09835f61689af490137de231eb42c
endif

PKG_BUILD_DIR:=$(BUILD_DIR)/node-v$(PKG_VERSION)-linux-$(NODEJS_CPU)-musl
HOST_BUILD_DIR:=$(BUILD_DIR_HOST)/node-v$(PKG_VERSION)-linux-x64

include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/package.mk

define Package/node
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Node.js
  TITLE:=Node.js is a platform built on Chrome's JavaScript runtime
  URL:=https://nodejs.org/
  DEPENDS:=@HAS_FPU @(aarch64||x86_64)
  ABI_VERSION:=127
endef

define Package/node/description
  Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine.
  This build ships the official community (unofficial-builds) musl binary:
  V8/ICU/OpenSSL/zlib/cares/nghttp2 are statically linked; only libc+libgcc_s
  are required at runtime.
endef

define Package/node-npm
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Node.js
  TITLE:=NPM stands for Node Package Manager
  URL:=https://www.npmjs.com/
  DEPENDS:=+node
endef

define Package/node-npm/description
	NPM is the package manager for NodeJS
endef

# 预编译二进制：解包即完成，无 configure/compile
define Build/Configure
endef

define Build/Compile
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/include
	$(CP) $(PKG_BUILD_DIR)/include/node $(1)/usr/include/
endef

define Package/node/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(CP) $(PKG_BUILD_DIR)/bin/node $(1)/usr/bin/
endef

define Package/node-npm/install
	$(INSTALL_DIR) $(1)/usr/lib/node_modules
	$(CP) $(PKG_BUILD_DIR)/lib/node_modules/npm $(1)/usr/lib/node_modules/
	$(INSTALL_DIR) $(1)/usr/bin
	$(CP) $(PKG_BUILD_DIR)/bin/npm $(1)/usr/bin/
	$(CP) $(PKG_BUILD_DIR)/bin/npx $(1)/usr/bin/
endef

define Package/node-npm/postrm
#!/bin/sh
rm -rf /usr/lib/node_modules/npm || true
endef

# host 侧沿用 nodejs.org 官方 x64 二进制（CI 主机即 x86_64），供 node-* 模块构建用
NODEJS_BIN:=node-v$(PKG_VERSION)-linux-x64.tar.gz
NODEJS_BIN_SUM:=b8492284c76a54baf58d61668dd0e41ee0fc98adabf9503d30f05d08f55b4406

define Download/nodebin
	URL:=https://nodejs.org/dist/v$(PKG_VERSION)/
	FILE:=$(NODEJS_BIN)
	HASH:=$(NODEJS_BIN_SUM)
endef

define Host/Prepare
	$(eval $(call Download,nodebin))
endef

define Host/Configure
	# nothing to do
endef

define Host/Compile
	# nothing to do
endef

define Host/Install
	rm -f $(1)/bin/npm
	rm -f $(1)/bin/npx
	rm -rf $(1)/lib/node_modules/npm
	rm -f $(1)/bin/corepack
	rm -rf $(1)/lib/node_modules/corepack
	$(TAR) xvf $(DL_DIR)/$(NODEJS_BIN) -C $(1) --strip-components=1
	rm -f $(1)/{CHANGELOG.md,LICENSE,README.md}
endef

$(eval $(call HostBuild))
$(eval $(call BuildPackage,node))
$(eval $(call BuildPackage,node-npm))
