# pgvector: PostgreSQL 向量扩展，官方包源无现成包，独立自建小包。
# 编译只依赖 feeds postgresql 已 staged 的服务端头文件（Build/InstallDev 提供
# usr/include/postgresql/{server,internal}），手动交叉编译为单个 vector.so，
# 不走 PGXS，几分钟完成。与 firmware 内置的 pgsql-server 同 feed 线，版本天然对齐。
include $(TOPDIR)/rules.mk

PKG_NAME:=pgvector
PKG_VERSION:=0.8.7
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/pgvector/pgvector.git
PKG_SOURCE_VERSION:=f37c13f68b57d2c3472b2214fbcff699d6d34876
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.xz
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

# feed 包名是 pgsql-server（PKG_NAME 为 postgresql），引用不存在的 "pgsql" 会解析不到构建 stamp
PKG_BUILD_DEPENDS:=pgsql-server

include $(INCLUDE_DIR)/package.mk

define Package/pgvector
  SECTION:=database
  CATEGORY:=Database
  TITLE:=pgvector vector data type for PostgreSQL
  URL:=https://github.com/pgvector/pgvector
  DEPENDS:=+pgsql-server
endef

define Package/pgvector/description
  The pgvector vector type and operators let you store embeddings and run
  similarity searches (L2/inner product/cosine, L1, binary, sparse vectors)
  with IVFFlat and HNSW indexes directly inside PostgreSQL.
  Requires CREATE EXTENSION vector in the database.
endef

define Build/Configure
endef

define Build/Compile
	$(TARGET_CC) $(TARGET_CFLAGS) $(TARGET_CPPFLAGS) \
		-I$(STAGING_DIR)/usr/include \
		-I$(STAGING_DIR)/usr/include/postgresql/server \
		-I$(STAGING_DIR)/usr/include/postgresql/internal \
		-fPIC -shared \
		$(PKG_BUILD_DIR)/src/*.c \
		-o $(PKG_BUILD_DIR)/vector.so
	sed 's/@extversion@/$(PKG_VERSION)/g' $(PKG_BUILD_DIR)/sql/vector.sql \
		> $(PKG_BUILD_DIR)/sql/vector--$(PKG_VERSION).sql
endef

define Package/pgvector/install
	$(INSTALL_DIR) $(1)/usr/lib/postgresql
	$(CP) $(PKG_BUILD_DIR)/vector.so $(1)/usr/lib/postgresql/
	$(INSTALL_DIR) $(1)/usr/share/postgresql/extension
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/vector.control $(1)/usr/share/postgresql/extension/
	$(CP) $(PKG_BUILD_DIR)/sql/vector--*.sql $(1)/usr/share/postgresql/extension/
endef

$(eval $(call BuildPackage,pgvector))
