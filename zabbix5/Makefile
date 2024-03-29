#
# Copyright (C) 2006-2024 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=zabbix
PKG_VERSION:=5.0.42
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://cdn.zabbix.com/zabbix/sources/stable/5.0/
PKG_HASH:=06e83e5b2eb3ffe93f60ae4de94164bd00a3a65bc4d0c5849eb627a54184a64d

PKG_MAINTAINER:=KWRT <kwrt@openwrt.org>
PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=COPYING
PKG_CPE_ID:=cpe:/a:zabbix:zabbix

PKG_INSTALL:=1
PKG_BUILD_PARALLEL:=1

PKG_CONFIG_DEPENDS:= \
  CONFIG_ZABBIX_GNUTLS \
  CONFIG_ZABBIX_OPENSSL

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/nls.mk

define Package/zabbix-agentd/config
comment "SSL support"

choice
        prompt "Selected SSL library"
        default ZABBIX_NOSSL

        config ZABBIX_OPENSSL
                bool "OpenSSL"

        config ZABBIX_GNUTLS
                bool "GnuTLS"

        config ZABBIX_NOSSL
                bool "No SSL support"

endchoice
endef

define Package/zabbix/Default
  SECTION:=admin
  CATEGORY:=Administration
  SUBMENU:=Zabbix5
  TITLE:=Zabbix5
  URL:=https://www.zabbix.com/
  USERID:=zabbix=53:zabbix=53
  DEPENDS += $(ICONV_DEPENDS) +libpcre +zlib +ZABBIX_GNUTLS:libgnutls +ZABBIX_OPENSSL:libopenssl
endef

define Package/zabbix-agentd
  $(call Package/zabbix/Default)
  TITLE+= agentd
endef

define Package/zabbix-extra-mac80211
  $(call Package/zabbix/Default)
  TITLE+= discovery/userparameters for mac80211
  DEPENDS = +zabbix-agentd @PACKAGE_MAC80211_DEBUGFS @KERNEL_DEBUG_FS
endef

define Package/zabbix-extra-network
  $(call Package/zabbix/Default)
  TITLE+= discovery/userparameters for network
  DEPENDS = +zabbix-agentd +libubus-lua +lua
endef

define Package/zabbix-extra-wifi
  $(call Package/zabbix/Default)
  TITLE+= discovery/userparameters for wifi
  DEPENDS = +zabbix-agentd +libiwinfo-lua +libubus-lua +lua
endef

define Package/zabbix-sender
  $(call Package/zabbix/Default)
  TITLE+= sender
endef

define Package/zabbix-get
  $(call Package/zabbix/Default)
  TITLE+= get
endef

define Package/zabbix-extra-mac80211/description
An extra package for zabbix-agentd that adds a discovery rule for mac80211 wifi phy and many userparameters.
It contains an suid helper to allow zabbix-agentd to still run as zabbix user and not as root.
See https://openwrt.org/docs/guide-user/services/network_monitoring/zabbix for ready to use zabbix templates.
endef

define Package/zabbix-extra-network/description
An extra package for zabbix-agentd that adds a discovery rule for openwrt network interfaces.
The idea here is to discover only interfaces listed in /etc/config/network (discover br-lan and not eth0.1 and wlan0)
See https://openwrt.org/docs/guide-user/services/network_monitoring/zabbix for ready to use zabbix templates.
endef

define Package/zabbix-extra-wifi/description
An extra package for zabbix-agentd that adds a discovery rule for wifi interfaces and many userparameters.
As it uses libiwinfo, it works with all wifi devices supported by openwrt.
See https://openwrt.org/docs/guide-user/services/network_monitoring/zabbix for ready to use zabbix templates.
endef

CONFIGURE_ARGS+= \
	--enable-agent \
	--disable-agent2 \
	--disable-server \
	--disable-proxy \
	--disable-java \
	$(call autoconf_bool,CONFIG_IPV6,ipv6) \
	--with-libevent=$(STAGING_DIR)/usr/include/libevent \
	--with-libpcre=$(STAGING_DIR)/usr/include \
	--with-zlib=$(STAGING_DIR)/usr/include \
	$(if $(CONFIG_ZABBIX_GNUTLS),--with-gnutls="$(STAGING_DIR)/usr") \
	$(if $(CONFIG_ZABBIX_OPENSSL),--with-openssl="$(STAGING_DIR)/usr")

CONFIGURE_VARS += \
	ac_cv_header_sys_sysinfo_h=no

MAKE_FLAGS += ARCH="linux"

define Package/zabbix/install/sbin
	$(INSTALL_DIR) \
		$(1)/usr/sbin

	$(INSTALL_BIN) \
		$(PKG_INSTALL_DIR)/usr/sbin/zabbix_$(2) \
		$(1)/usr/sbin/
endef

define Package/zabbix/install/bin
	$(INSTALL_DIR) \
		$(1)/usr/bin

	$(INSTALL_BIN) \
		$(PKG_INSTALL_DIR)/usr/bin/zabbix_$(2) \
		$(1)/usr/bin/
endef

define Package/zabbix/install/etc
	$(INSTALL_DIR) \
		$(1)/etc

	$(INSTALL_CONF) \
		$(PKG_INSTALL_DIR)/etc/zabbix_$(2).conf \
		$(1)/etc/
endef

define Package/zabbix/install/init.d
	$(INSTALL_DIR) \
		$(1)/etc/init.d

	$(INSTALL_BIN) \
		./files/zabbix_$(2).init \
		$(1)/etc/init.d/zabbix_$(2)
endef

define Package/zabbix/install/zabbix.conf.d
	$(INSTALL_DIR) \
		$(1)/etc/zabbix_agentd.conf.d

	$(INSTALL_BIN) \
		./files/$(2) \
		$(1)/etc/zabbix_agentd.conf.d/$(2)
endef

define Package/zabbix-agentd/conffiles
/etc/zabbix_agentd.conf
endef

ifdef CONFIG_PACKAGE_zabbix-extra-mac80211
define Build/Prepare/zabbix-extra-mac80211
	mkdir -p $(PKG_BUILD_DIR)/zabbix-extra-mac80211
	$(CP) ./files/zabbix_helper_mac80211.c $(PKG_BUILD_DIR)/zabbix-extra-mac80211/
endef

define Build/Compile/zabbix-extra-mac80211
	$(TARGET_CC) $(TARGET_CFLAGS) $(PKG_BUILD_DIR)/zabbix-extra-mac80211/zabbix_helper_mac80211.c -o $(PKG_BUILD_DIR)/zabbix-extra-mac80211/zabbix_helper_mac80211
endef
endif

define Build/Prepare
	$(call Build/Prepare/Default)
	$(call Build/Prepare/zabbix-extra-mac80211)
endef

define Build/Compile
	$(call Build/Compile/Default)
	$(call Build/Compile/zabbix-extra-mac80211)
endef

define Package/zabbix-agentd/install
	$(INSTALL_DIR) $(1)/etc/zabbix_agentd.conf.d
	$(call Package/zabbix/install/sbin,$(1),agentd)
	$(call Package/zabbix/install/etc,$(1),agentd)
	$(call Package/zabbix/install/init.d,$(1),agentd)
endef

define Package/zabbix-extra-mac80211/install
	$(call Package/zabbix/install/zabbix.conf.d,$(1),mac80211)
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/zabbix-extra-mac80211/zabbix_helper_mac80211 $(1)/usr/bin/
	chmod 4755 $(1)/usr/bin/zabbix_helper_mac80211
endef

define Package/zabbix-extra-network/install
	$(call Package/zabbix/install/zabbix.conf.d,$(1),network)
	$(INSTALL_DIR) $(1)/usr/share/acl.d
	$(INSTALL_DATA) ./files/zabbix-network-ubus-acl.json $(1)/usr/share/acl.d/zabbix-network.json
endef

define Package/zabbix-extra-network/postinst
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
	killall -s HUP ubusd
fi
endef

define Package/zabbix-extra-wifi/install
	$(call Package/zabbix/install/zabbix.conf.d,$(1),wifi)
	$(INSTALL_DIR) $(1)/usr/share/acl.d
	$(INSTALL_DATA) ./files/zabbix-wifi-ubus-acl.json $(1)/usr/share/acl.d/zabbix-wifi.json
endef

define Package/zabbix-extra-wifi/postinst
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
	killall -s HUP ubusd
fi
endef

define Package/zabbix-sender/install
	$(call Package/zabbix/install/bin,$(1),sender)
endef

define Package/zabbix-get/install
	$(call Package/zabbix/install/bin,$(1),get)
endef

$(eval $(call BuildPackage,zabbix-agentd))
$(eval $(call BuildPackage,zabbix-extra-mac80211))
$(eval $(call BuildPackage,zabbix-extra-network))
$(eval $(call BuildPackage,zabbix-extra-wifi))
$(eval $(call BuildPackage,zabbix-sender))
$(eval $(call BuildPackage,zabbix-get))
