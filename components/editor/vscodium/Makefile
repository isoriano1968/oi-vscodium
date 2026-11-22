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

# The desktop VSCodium on our Electron, as FreeBSD's editors/vscode does it
# (see the README): the same sources and
# patches as editor/vscodium-server, the Linux target, our Electron fed in
# through Electron's download cache, native modules built against Electron's
# headers.

BUILD_BITS =		64
BUILD_STYLE =		justmake

include ../../../make-rules/shared-macros.mk

# The version, in one place, as editor/vscodium-server (keep the two equal:
# a VSCodium client asks for a server of its own commit).
COMPONENT_VERSION =	1.143.17232
# rev 3: bundles electron-43 rev 5 (decommit fix)
COMPONENT_REVISION =	3
VSCODE_VERSION =	1.143.1
VSCODE_COMMIT =		e646868227c3e3a9e3f24277c8c7220be42c0be7
VSCODIUM_COMMIT :=	$(shell echo $(COMPONENT_VERSION) | /usr/bin/sha1sum | cut -d' ' -f1)

# Our Electron (developer/electron-43). VS Code 1.143 targets 43.7.7; 43.8.10
# has the same Node ABI (NODE_MODULE_VERSION 148).
ELECTRON_VERSION =	43.8.10
ELECTRON_DIR =		$(USRLIBDIR)/electron-43

COMPONENT_NAME =	vscodium
COMPONENT_SUMMARY =	VSCodium: the VS Code editor without Microsoft branding and telemetry
COMPONENT_DESCRIPTION =	VSCodium, the desktop VS Code editor (Code - OSS) without Microsoft branding and telemetry, with the Open VSX extension gallery, on Electron 43.
COMPONENT_PROJECT_URL =	https://vscodium.com/
COMPONENT_FMRI =	editor/vscodium
COMPONENT_CLASSIFICATION = Development/Integrated Development Environments
COMPONENT_LICENSE =	MIT
COMPONENT_LICENSE_FILE = LICENSE

COMPONENT_SRC =		vscodium-$(COMPONENT_VERSION)
COMPONENT_ARCHIVE =	$(COMPONENT_SRC).tar.gz
COMPONENT_ARCHIVE_URL =	https://github.com/VSCodium/vscodium/archive/refs/tags/$(COMPONENT_VERSION).tar.gz
COMPONENT_ARCHIVE_HASH = sha256:5cc688b22f09bc08a20dfbcec862f3e4625ac8ce4a59f0b23c80743322e17912

COMPONENT_SRC_1 =	vscode-$(VSCODE_VERSION)
COMPONENT_ARCHIVE_1 =	$(COMPONENT_SRC_1).tar.gz
COMPONENT_ARCHIVE_URL_1 = https://github.com/microsoft/vscode/archive/refs/tags/$(VSCODE_VERSION).tar.gz
COMPONENT_ARCHIVE_HASH_1 = sha256:d0aa0fe95602998ecdab87a1f51139455b43486a3a8b50f7f62f6b459ffd31a8

# The server's patches (VSCodium recipe and the illumos changes), linked into
# patches/ from ../vscodium-server/patches, plus ours (patches/
# 17-desktop-sunos.patch_1: the desktop startup paths and the Electron
# checksums). Links, not ADDITIONAL_PATCHES: prep-patch.mk names the stamps
# after $(PATCH_DIR)/% and makes rules only for suffixes found in patches/,
# so patches from another directory were silently skipped.
SERVER_DIR_SRC =	$(COMPONENT_DIR)/../vscodium-server

NODE_VERSION =		24
NODE_DIR =		$(USRDIR)/node/$(NODE_VERSION)

CLONEY_MODE =		copy

TEST_TARGET =		$(NO_TESTS)

include $(WS_MAKE_RULES)/common.mk

VSCODE_TREE =		$(BUILD_DIR_64)/vscode

# Electron's download cache: @electron/get looks for
# <cache>/<sha256 of the release URL>/electron-v<ver>-linux-x64.zip (FreeBSD
# Mk/Uses/electron.mk, electron-generate-electron-zip). @electron/get 5 takes
# <cache> from env-paths only ($XDG_CACHE_HOME/electron), no longer from
# electron_config_cache: without XDG_CACHE_HOME it fetched the official Linux
# zip into ~/.cache/electron and packaged that (2026-11-19).
XDG_CACHE =		$(BUILD_DIR)/xdg-cache
ELECTRON_CACHE =	$(XDG_CACHE)/electron
ELECTRON_URL_HASH :=	$(shell printf %s https://github.com/electron/electron/releases/download/v$(ELECTRON_VERSION) | /usr/bin/sha256sum | cut -d' ' -f1)
ELECTRON_ZIP_DIR =	$(ELECTRON_CACHE)/$(ELECTRON_URL_HASH)

VSCODE_BUILD_ENV =	PATH=$(NODE_DIR)/bin:$(GNUBIN):$(USRBINDIR)
VSCODE_BUILD_ENV +=	CC=$(CC) CXX=$(CXX) MAKE=$(GMAKE) PYTHON=$(PYTHON)
VSCODE_BUILD_ENV +=	npm_config_cache=$(BUILD_DIR)/npm-cache
VSCODE_BUILD_ENV +=	npm_config_maxsockets=4 npm_config_fetch_retries=6
VSCODE_BUILD_ENV +=	npm_config_fetch_retry_maxtimeout=120000
VSCODE_BUILD_ENV +=	XDG_CACHE_HOME=$(XDG_CACHE)
VSCODE_BUILD_ENV +=	ELECTRON_SKIP_BINARY_DOWNLOAD=1
VSCODE_BUILD_ENV +=	PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1
VSCODE_BUILD_ENV +=	VSCODE_SKIP_PRELAUNCH=1
VSCODE_BUILD_ENV +=	VSCODE_SKIP_NODE_VERSION_CHECK=1
VSCODE_BUILD_ENV +=	BUILD_SOURCEVERSION=$(VSCODIUM_COMMIT)
VSCODE_BUILD_ENV +=	GIT_CEILING_DIRECTORIES=$(BUILD_DIR)
VSCODE_BUILD_ENV +=	VSCODE_QUALITY=stable OS_NAME=linux VSCODE_ARCH=x64
VSCODE_BUILD_ENV +=	CI_BUILD=no RELEASE_VERSION=$(COMPONENT_VERSION)
VSCODE_BUILD_ENV +=	MS_COMMIT=$(VSCODE_COMMIT) MS_TAG=$(VSCODE_VERSION)
VSCODE_BUILD_ENV +=	NODE_OPTIONS=--max-old-space-size=8192
# prepare_vscode.sh writes it into package.json's node command lines (upstream's
# build.sh sets it); empty there, it overrode NODE_OPTIONS with 4 GB.
VSCODE_BUILD_ENV +=	MAX_OLD_SPACE_SIZE=8192
# node's malloc is libumem, which grows only by brk; V8's randomly placed heap
# chunks can land just above the break, and the bundling step then died in
# malloc (bad_alloc) with memory free. mmap has no such limit.
VSCODE_BUILD_ENV +=	UMEM_OPTIONS=backend=mmap

NPM_ENV =		$(ENV) $(VSCODE_BUILD_ENV)
NPM_NO_SCRIPTS =	$(NPM_ENV) npm_config_ignore_scripts=true

NPM_PATCHES =		$(sort $(wildcard $(COMPONENT_DIR)/npm-patches/*.patch) \
			    $(wildcard $(SERVER_DIR_SRC)/npm-patches/*.patch))

# 1. As the server: prepare_vscode.sh, the other npm roots, npm-patches.
# 2. Our Electron version in .npmrc (build/lib/electron.ts reads it there),
#    its zip in the download cache (without mksnapshot and
#    v8_context_snapshot_generator, 280 MB the install drops anyway:
#    gulp-electron holds the zip's files in memory, and with them
#    package-linux-x64 ran out of memory, 2026-11-19), its electron.d.ts (with its sha256 in
#    build/checksums/electron.txt: the root postinstall, run again by npm
#    rebuild, else fetches 43.8.10's from GitHub against 43.7.7's sum).
# 3. npm rebuild in the root, as upstream's npm ci: the root's native modules
#    by its .npmrc (runtime electron, target $(ELECTRON_VERSION); node-gyp
#    fetches those headers from electronjs.org, the same ABI as ours: npm 11
#    no longer passes npm_config_nodedir), then its postinstall rebuilds
#    every other npm root (build/, remote/, extensions) by that root's own
#    .npmrc. No npm_config_runtime/target/nodedir in the environment: they
#    win over build/.npmrc and remote/.npmrc, and build/'s tree-sitter then
#    compiled against Electron's headers and failed (2026-11-19).
COMPONENT_PRE_BUILD_ACTION += \
	$(RM) -r $(VSCODE_TREE) && \
	$(CP) -rp $(SOURCE_DIR_1) $(VSCODE_TREE) && \
	( cd $(@D) && $(NPM_NO_SCRIPTS) $(USRBINDIR)/bash ./prepare_vscode.sh ) && \
	( cd $(VSCODE_TREE) && $(NPM_NO_SCRIPTS) $(NODE_DIR)/bin/node build/npm/postinstall.ts ) && \
	for p in $(NPM_PATCHES) ; do \
		n=$$(basename $$p .patch) ; v=$${n\#\#*+} ; m=$$(echo $${n%+*} | tr + /) ; hit= ; \
		for d in $$(cd $(VSCODE_TREE) && $(FIND) . -path "*/node_modules/$$m" -prune -type d) ; do \
			[ "$$($(NODE_DIR)/bin/node -p "require('$(VSCODE_TREE)/$$d/package.json').version")" = "$$v" ] || continue ; \
			echo "npm-patches: $$n in $$d" ; hit=1 ; \
			$(GPATCH) -d $(VSCODE_TREE)/$$d -p1 -s < $$p || exit 1 ; \
		done ; \
		[ -n "$$hit" ] || { echo "npm-patches: $$n matches no module (renamed or new version?)" ; exit 1 ; } ; \
	done && \
	$(GSED) -i -E 's|^(target=").*(")$$|\1$(ELECTRON_VERSION)\2|' $(VSCODE_TREE)/.npmrc && \
	$(RM) -r $(ELECTRON_CACHE) && $(MKDIR) $(ELECTRON_ZIP_DIR) && \
	( cd $(ELECTRON_DIR) && /usr/bin/zip -q -r $(ELECTRON_ZIP_DIR)/electron-v$(ELECTRON_VERSION)-linux-x64.zip . -x 'node_headers/*' 'gen/*' mksnapshot v8_context_snapshot_generator ) && \
	( cd $(ELECTRON_ZIP_DIR) && /usr/bin/sha256sum *.zip | $(GSED) -e 's/ / */' > SHASUMS256.txt ) && \
	$(MKDIR) $(VSCODE_TREE)/.build/typings && \
	$(CP) $(ELECTRON_DIR)/gen/electron/tsc/typings/electron.d.ts $(VSCODE_TREE)/.build/typings/ && \
	s=$$(/usr/bin/sha256sum $(VSCODE_TREE)/.build/typings/electron.d.ts | cut -d' ' -f1) && \
	$(GSED) -i -E "s/^[0-9a-f]{64} (\*electron\.d\.ts)$$/$$s \1/" $(VSCODE_TREE)/build/checksums/electron.txt && \
	/usr/gnu/bin/grep -q "^$$s \*electron\.d\.ts$$" $(VSCODE_TREE)/build/checksums/electron.txt && \
	( cd $(VSCODE_TREE) && $(NPM_ENV) npm rebuild --foreground-scripts ) ;

COMPONENT_BUILD_ACTION = \
	cd $(VSCODE_TREE) && $(NPM_ENV) npm run gulp vscode-linux-x64-min

APP_BUILD =		$(BUILD_DIR_64)/VSCode-linux-x64
APP_DIR =		$(USRLIBDIR)/$(COMPONENT_NAME)

# Left out, as FreeBSD: Electron's build-only tools and headers. rg from the
# system, as the server. Also left out: prebuilt binaries for other systems
# that the Linux target brings along (ripgrep's linux-x64 rg, the sandbox
# runtime's Linux seccomp helper, the Linux os-proxy-resolver, js-debug's
# win32 .node files) and node-gyp's obj.target copies.
APP_RES =		$(PROTO_DIR)$(APP_DIR)/resources/app
COMPONENT_INSTALL_ACTION = \
	$(RM) -r $(PROTO_DIR)$(APP_DIR) && \
	$(MKDIR) $(PROTO_DIR)$(USRLIBDIR) && \
	$(CP) -rp $(APP_BUILD) $(PROTO_DIR)$(APP_DIR) && \
	$(RM) -r $(PROTO_DIR)$(APP_DIR)/mksnapshot $(PROTO_DIR)$(APP_DIR)/v8_context_snapshot_generator \
	    $(PROTO_DIR)$(APP_DIR)/gen $(PROTO_DIR)$(APP_DIR)/node_headers && \
	$(FIND) $(APP_RES) -path '*/build/Release/obj.target' -prune -exec $(RM) -r {} + && \
	$(FIND) $(APP_RES) \( -path '*/@vscode/ripgrep-universal/bin/*-*' -o \
	    -path '*/@vscode/sandbox-runtime/vendor/seccomp' -o \
	    -path '*/@vscodium/os-proxy-resolver-linux-*' \) -prune -exec $(RM) -r {} + && \
	$(FIND) $(APP_RES)/extensions -name '*.win32-*.node' -exec $(RM) {} + && \
	for b in $$($(FIND) $(APP_RES) -type d -path '*/@vscode/ripgrep-universal/bin') ; do \
		$(MKDIR) $$b/sunos-x64 && \
		$(SYMLINK) $$(/usr/bin/realpath -m --relative-to=$$b/sunos-x64 $(PROTO_DIR)$(USRBINDIR)/rg) $$b/sunos-x64/rg || exit 1 ; \
	done && \
	$(MKDIR) $(PROTOUSRBINDIR) && \
	$(RM) $(PROTOUSRBINDIR)/codium && \
	$(SYMLINK) ../lib/$(COMPONENT_NAME)/bin/codium $(PROTOUSRBINDIR)/codium && \
	$(MKDIR) $(PROTOUSRSHAREDIR)/applications $(PROTOUSRSHAREDIR)/pixmaps && \
	$(INSTALL) -m 0444 $(VSCODE_TREE)/resources/linux/code.png $(PROTOUSRSHAREDIR)/pixmaps/vscodium.png && \
	for f in code:codium code-url-handler:codium-url-handler ; do \
		$(GSED) -e 's/@@NAME_LONG@@/VSCodium/g' -e 's/@@NAME_SHORT@@/VSCodium/g' \
		    -e 's/@@NAME@@/codium/g' -e 's|@@EXEC@@|/usr/bin/codium|g' \
		    -e 's/@@ICON@@/vscodium/g' -e 's/@@DESKTOP_NAME@@/VSCodium/g' \
		    -e 's/@@URLPROTOCOL@@/vscodium/g' \
		    $(VSCODE_TREE)/resources/linux/$${f%%:*}.desktop > $(PROTOUSRSHAREDIR)/applications/$${f\#\#*:}.desktop || exit 1 ; \
	done ;

# The desktop patch turns validateChecksum off and sets
# unsafelyDisableChecksums; npm-patches/@vscode+gulp-electron+1.43.1.patch
# passes the latter on to @electron/get, which otherwise checks our cached zip
# against the release's sums and replaces it with the official Linux build
# (the 2026-11-18 reading missed that). vscode-linux-x64-min names the platform
# linux itself; .moduleignore.sunos comes from the server's patch 11.
# The rg link is computed: on the desktop it lives in node_modules.asar.unpacked.

REQUIRED_PACKAGES += developer/electron-43
REQUIRED_PACKAGES += developer/versioning/git
REQUIRED_PACKAGES += compress/zip
REQUIRED_PACKAGES += file/gnu-coreutils
REQUIRED_PACKAGES += text/jq
REQUIRED_PACKAGES += runtime/nodejs-24
REQUIRED_PACKAGES += text/ripgrep
# Auto-generated dependencies
REQUIRED_PACKAGES += $(GCC_RUNTIME_PKG)
REQUIRED_PACKAGES += $(GXX_RUNTIME_PKG)
REQUIRED_PACKAGES += SUNWcs
REQUIRED_PACKAGES += gnome/accessibility/at-spi2-core
REQUIRED_PACKAGES += library/desktop/cairo
REQUIRED_PACKAGES += library/desktop/gtk3
REQUIRED_PACKAGES += library/desktop/pango
REQUIRED_PACKAGES += library/expat
REQUIRED_PACKAGES += library/glib2
REQUIRED_PACKAGES += library/nspr
REQUIRED_PACKAGES += service/security/kerberos-5
REQUIRED_PACKAGES += shell/bash
REQUIRED_PACKAGES += shell/ksh93
REQUIRED_PACKAGES += system/library
REQUIRED_PACKAGES += system/library/libdbus
REQUIRED_PACKAGES += system/library/math
REQUIRED_PACKAGES += system/library/mozilla-nss
REQUIRED_PACKAGES += system/library/security/gss
REQUIRED_PACKAGES += text/ripgrep
REQUIRED_PACKAGES += x11/library/libdrm
REQUIRED_PACKAGES += x11/library/libx11
REQUIRED_PACKAGES += x11/library/libxcb
REQUIRED_PACKAGES += x11/library/libxcomposite
REQUIRED_PACKAGES += x11/library/libxdamage
REQUIRED_PACKAGES += x11/library/libxext
REQUIRED_PACKAGES += x11/library/libxfixes
REQUIRED_PACKAGES += x11/library/libxkbfile
REQUIRED_PACKAGES += x11/library/libxrandr
REQUIRED_PACKAGES += x11/library/mesa
