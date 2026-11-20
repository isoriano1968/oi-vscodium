#
# This file and its contents are supplied under the terms of the
# Common Development and Distribution License ("CDDL"), version 1.0.
# You may only use this file in accordance with the terms of version
# 1.0 of the CDDL.
#
# A full copy of the text of the CDDL should have accompanied this
# source.  A copy of the CDDL is also available via the Internet at
# http://www.illumos.org/license/CDDL.
#

#
# Copyright 2026 isoriano1968
#

BUILD_BITS =		64
BUILD_STYLE =		justmake

include ../../../make-rules/shared-macros.mk

# The version, in one place. COMPONENT_VERSION is the VSCodium release tag;
# VSCODE_VERSION and VSCODE_COMMIT are the VS Code tag it builds
# (upstream/stable.json in the VSCodium tree). To move up or down: change these
# three and the two hashes, then rebase patches/ (the README has the steps;
# older patch sets are in this repository's history).
COMPONENT_VERSION =	1.143.17232
COMPONENT_REVISION =	1
VSCODE_VERSION =	1.143.1
VSCODE_COMMIT =		e646868227c3e3a9e3f24277c8c7220be42c0be7

# The commit in product.json. A VSCodium client asks for a server of its own
# commit, and VSCodium's releases stamp sha1("<release>\n") there, not the VS
# Code commit (version.sh: echo "${RELEASE_VERSION}" | sha1sum).
VSCODIUM_COMMIT :=	$(shell echo $(COMPONENT_VERSION) | /usr/bin/sha1sum | cut -d' ' -f1)

COMPONENT_NAME =	vscodium-server
COMPONENT_SUMMARY =	VSCodium server: the editor in a web browser, and remote host for VSCodium clients
COMPONENT_DESCRIPTION =	The server part of VSCodium (VS Code without Microsoft branding and telemetry, with the Open VSX extension gallery), built as the reh-web target. It serves the full editor to a web browser (vscodium-web, the VS Code (OI) menu entry), and VSCodium clients connect to it over ssh (open-remote-ssh). Runs on Node.js.
COMPONENT_PROJECT_URL =	https://vscodium.com/
COMPONENT_FMRI =	editor/vscodium-server
COMPONENT_CLASSIFICATION = Development/Integrated Development Environments
COMPONENT_LICENSE =	MIT
COMPONENT_LICENSE_FILE = LICENSE

# Source 0: VSCodium's build recipe (product.json, its patch series,
# prepare_vscode.sh). patches/*.patch apply to it.
COMPONENT_SRC =		vscodium-$(COMPONENT_VERSION)
COMPONENT_ARCHIVE =	$(COMPONENT_SRC).tar.gz
COMPONENT_ARCHIVE_URL =	https://github.com/VSCodium/vscodium/archive/refs/tags/$(COMPONENT_VERSION).tar.gz
COMPONENT_ARCHIVE_HASH = sha256:5cc688b22f09bc08a20dfbcec862f3e4625ac8ce4a59f0b23c80743322e17912

# Source 1: VS Code itself. patches/*.patch_1 (the illumos changes) apply to
# it before VSCodium's series does.
COMPONENT_SRC_1 =	vscode-$(VSCODE_VERSION)
COMPONENT_ARCHIVE_1 =	$(COMPONENT_SRC_1).tar.gz
COMPONENT_ARCHIVE_URL_1 = https://github.com/microsoft/vscode/archive/refs/tags/$(VSCODE_VERSION).tar.gz
COMPONENT_ARCHIVE_HASH_1 = sha256:d0aa0fe95602998ecdab87a1f51139455b43486a3a8b50f7f62f6b459ffd31a8

NODE_VERSION =		24
NODE_DIR =		$(USRDIR)/node/$(NODE_VERSION)

# The build runs in a copy of the VSCodium tree, with VS Code copied to its
# vscode/ subdirectory, the layout VSCodium's scripts expect.
CLONEY_MODE =		copy

TEST_TARGET =		$(NO_TESTS)

include $(WS_MAKE_RULES)/common.mk

VSCODE_TREE =		$(BUILD_DIR_64)/vscode

# npm ci fetches the npm packages at build time, pinned by sha512 in the
# package-lock.json files (as make-rules/cargo-vendor.mk fetches crates).
# The native modules are built from source with gcc against the Node.js
# headers.
VSCODE_BUILD_ENV =	PATH=$(NODE_DIR)/bin:$(GNUBIN):$(USRBINDIR)
VSCODE_BUILD_ENV +=	CC=$(CC) CXX=$(CXX) MAKE=$(GMAKE) PYTHON=$(PYTHON)
VSCODE_BUILD_ENV +=	npm_config_cache=$(BUILD_DIR)/npm-cache
# postinstall runs up to 8 npm installs at once: fewer sockets each, and more
# retries, or registry fetches time out (ETIMEDOUT) on the build machine's network.
VSCODE_BUILD_ENV +=	npm_config_maxsockets=4 npm_config_fetch_retries=6
VSCODE_BUILD_ENV +=	npm_config_fetch_retry_maxtimeout=120000
VSCODE_BUILD_ENV +=	ELECTRON_SKIP_BINARY_DOWNLOAD=1
VSCODE_BUILD_ENV +=	PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1
VSCODE_BUILD_ENV +=	VSCODE_SKIP_PRELAUNCH=1
VSCODE_BUILD_ENV +=	VSCODE_SKIP_NODE_VERSION_CHECK=1
VSCODE_BUILD_ENV +=	BUILD_SOURCEVERSION=$(VSCODIUM_COMMIT)
# VSCodium applies its patch series with git apply. The build tree lies inside
# the oi-userland git checkout, where git apply would take the paths as
# relative to that repository and silently skip them all: stop git's
# repository search at the build directory.
VSCODE_BUILD_ENV +=	GIT_CEILING_DIRECTORIES=$(BUILD_DIR)
# What VSCodium's build.sh sets for a local (non-CI) Linux build; OS_NAME
# linux selects its patches/linux series, which the server shares.
VSCODE_BUILD_ENV +=	VSCODE_QUALITY=stable OS_NAME=linux VSCODE_ARCH=x64
VSCODE_BUILD_ENV +=	CI_BUILD=no RELEASE_VERSION=$(COMPONENT_VERSION)
VSCODE_BUILD_ENV +=	MS_COMMIT=$(VSCODE_COMMIT) MS_TAG=$(VSCODE_VERSION)

NPM_ENV =		$(ENV) $(VSCODE_BUILD_ENV)
NPM_NO_SCRIPTS =	$(NPM_ENV) npm_config_ignore_scripts=true

# npm-patches/<scope>+<name>+<version>.patch (the patch-package naming): the
# illumos changes to npm modules, applied to every copy of that module and
# version in the tree before its native code is compiled.
NPM_PATCHES =		$(sort $(wildcard $(COMPONENT_DIR)/npm-patches/*.patch))

# 1. VSCodium's prepare_vscode.sh: its product.json, its patch series, and
#    npm ci of the root, without install scripts.
# 2. The other npm roots (build/, remote/, extensions/), without install
#    scripts.
# 3. npm-patches.
# 4. npm rebuild: the install scripts, native modules compiled with gcc.
COMPONENT_PRE_BUILD_ACTION += \
	$(RM) -r $(VSCODE_TREE) && \
	$(CP) -rp $(SOURCE_DIR_1) $(VSCODE_TREE) && \
	( cd $(@D) && $(NPM_NO_SCRIPTS) $(USRBINDIR)/bash ./prepare_vscode.sh ) && \
	( cd $(VSCODE_TREE) && $(NPM_NO_SCRIPTS) $(NODE_DIR)/bin/node build/npm/postinstall.ts ) && \
	for p in $(NPM_PATCHES) ; do \
		n=$$(basename $$p .patch) ; v=$${n\#\#*+} ; m=$$(echo $${n%+*} | tr + /) ; \
		for d in $$(cd $(VSCODE_TREE) && $(FIND) . -path "*/node_modules/$$m" -prune -type d) ; do \
			[ "$$($(NODE_DIR)/bin/node -p "require('$(VSCODE_TREE)/$$d/package.json').version")" = "$$v" ] || continue ; \
			echo "npm-patches: $$n in $$d" ; \
			$(GPATCH) -d $(VSCODE_TREE)/$$d -p1 -s < $$p || exit 1 ; \
		done ; \
	done && \
	for d in . build remote ; do \
		( cd $(VSCODE_TREE)/$$d && $(NPM_ENV) npm rebuild --foreground-scripts ) || exit 1 ; \
	done ;

# reh-web: the remote host plus the web client, so one server does both.
COMPONENT_BUILD_ACTION = \
	cd $(VSCODE_TREE) && $(NPM_ENV) npm run gulp vscode-reh-web-sunos-x64-min

# The build writes the server next to the vscode tree.
REH_BUILD =		$(BUILD_DIR_64)/vscode-reh-web-sunos-x64
SERVER_DIR =		$(USRLIBDIR)/$(COMPONENT_NAME)

# vscodium-web (files/) starts the user's server and opens the browser; the
# .desktop entry "VS Code (OI)" runs it, with VSCodium's icon.
# The bin script runs $ROOT/node: link it to the system's Node.js, and give
# @vscode/ripgrep-universal the system's rg (it ships no illumos binary).
# Left out: the Linux seccomp helpers of sandbox-runtime, js-debug's Windows
# addons, and node-gyp's object directories.
COMPONENT_INSTALL_ACTION = \
	$(RM) -r $(PROTO_DIR)$(SERVER_DIR) && \
	$(MKDIR) $(PROTO_DIR)$(USRLIBDIR) && \
	$(CP) -rp $(REH_BUILD) $(PROTO_DIR)$(SERVER_DIR) && \
	$(RM) -r $(PROTO_DIR)$(SERVER_DIR)/node_modules/@vscode/sandbox-runtime/vendor/seccomp $(PROTO_DIR)$(SERVER_DIR)/node_modules/@vscode/sandbox-runtime/vendor/seccomp-src && \
	$(RM) $(PROTO_DIR)$(SERVER_DIR)/extensions/ms-vscode.js-debug/src/win32-*.node && \
	$(FIND) $(PROTO_DIR)$(SERVER_DIR)/node_modules -path '*/build/Release/obj.target' -prune -exec $(RM) -r {} + && \
	$(SYMLINK) ../../node/$(NODE_VERSION)/bin/node $(PROTO_DIR)$(SERVER_DIR)/node && \
	$(MKDIR) $(PROTO_DIR)$(SERVER_DIR)/node_modules/@vscode/ripgrep-universal/bin/sunos-x64 && \
	$(SYMLINK) ../../../../../../../bin/rg $(PROTO_DIR)$(SERVER_DIR)/node_modules/@vscode/ripgrep-universal/bin/sunos-x64/rg && \
	$(MKDIR) $(PROTOUSRBINDIR) && \
	$(RM) $(PROTOUSRBINDIR)/codium-server && \
	$(SYMLINK) ../lib/$(COMPONENT_NAME)/bin/codium-server $(PROTOUSRBINDIR)/codium-server && \
	$(INSTALL) -m 0555 $(COMPONENT_DIR)/files/vscodium-web $(PROTOUSRBINDIR)/vscodium-web && \
	$(MKDIR) $(PROTOUSRSHAREDIR)/applications $(PROTOUSRSHAREDIR)/pixmaps $(PROTOUSRSHAREDIR)/icons/hicolor/512x512/apps && \
	$(INSTALL) -m 0444 $(COMPONENT_DIR)/files/vscodium-web.desktop $(PROTOUSRSHAREDIR)/applications/vscodium-web.desktop && \
	$(INSTALL) -m 0444 $(SOURCE_DIR)/src/stable/resources/linux/code.png $(PROTOUSRSHAREDIR)/pixmaps/vscodium-web.png && \
	$(INSTALL) -m 0444 $(SOURCE_DIR)/src/stable/resources/server/code-512.png $(PROTOUSRSHAREDIR)/icons/hicolor/512x512/apps/vscodium-web.png ;

# Build tools that VSCodium's scripts run (git apply, sha1sum, jq).
REQUIRED_PACKAGES += developer/versioning/git
REQUIRED_PACKAGES += file/gnu-coreutils
REQUIRED_PACKAGES += text/jq

# Auto-generated dependencies
REQUIRED_PACKAGES += $(GCC_RUNTIME_PKG)
REQUIRED_PACKAGES += $(GXX_RUNTIME_PKG)
REQUIRED_PACKAGES += SUNWcs
REQUIRED_PACKAGES += runtime/nodejs-24
REQUIRED_PACKAGES += service/security/kerberos-5
REQUIRED_PACKAGES += shell/bash
REQUIRED_PACKAGES += shell/ksh93
REQUIRED_PACKAGES += system/library
REQUIRED_PACKAGES += system/library/math
REQUIRED_PACKAGES += system/library/security/gss
REQUIRED_PACKAGES += text/ripgrep
