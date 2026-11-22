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

# The AMIX extension (vscode-amix) as a built-in extension of VSCodium:
# it goes into VSCodium's own extensions directory
# (usr/lib/vscodium/resources/app/extensions/vscode-amix), where VSCodium
# finds its bundled extensions, so it works for every user without
# "codium --install-extension".  Pure JavaScript, nothing is compiled.

BUILD_BITS =		NO_ARCH
BUILD_STYLE =		justmake

include ../../../make-rules/shared-macros.mk

COMPONENT_NAME =	vscode-amix
COMPONENT_VERSION =	0.6.9
COMPONENT_SUMMARY =	AMIX development tools for VSCodium
COMPONENT_DESCRIPTION =	VSCodium extension for AMIX (Amiga Unix, m68k) development: builds with the AMIX gcc 2.7 and gcc 14 cross toolchains, m68k assembler syntax, snippets, console and CDE sample projects, and running and debugging on an AMIX machine.
COMPONENT_PROJECT_URL =	https://github.com/isoriano1968/amix-userland
COMPONENT_FMRI =	editor/vscodium/extension/vscode-amix
COMPONENT_CLASSIFICATION = Development/Integrated Development Environments
COMPONENT_LICENSE =	MIT
COMPONENT_LICENSE_FILE = extension/vscode.license

COMPONENT_SRC =		$(COMPONENT_NAME)-$(COMPONENT_VERSION)
# The release asset is the .vsix (a zip archive).  It is kept under a .zip
# name, which userland-unpack knows; the content is the pristine download.
COMPONENT_ARCHIVE =	$(COMPONENT_SRC).zip
COMPONENT_ARCHIVE_URL =	https://github.com/isoriano1968/amix-userland/releases/download/vscode-amix-$(COMPONENT_VERSION)/vscode-amix-$(COMPONENT_VERSION).vsix
COMPONENT_ARCHIVE_HASH = sha256:6b1a6e154524231817b3b4414d26ab90d8fc2d2d08a7d48b48b1ad4c56bb0963
# The vsix has no top directory.
UNPACK_ARGS =		--relocate-to $(COMPONENT_SRC)

TEST_TARGET =		$(NO_TESTS)

include $(WS_MAKE_RULES)/common.mk

EXT_DIR =		$(PROTO_DIR)$(USRLIBDIR)/vscodium/resources/app/extensions/vscode-amix

# Nothing to build; install the vsix' extension/ folder as it is.
COMPONENT_BUILD_ACTION =	true
COMPONENT_INSTALL_ACTION = \
	$(RM) -r $(EXT_DIR) && $(MKDIR) $(EXT_DIR) && \
	$(CP) -rp $(SOURCE_DIR)/extension/. $(EXT_DIR)/
