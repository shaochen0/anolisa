# Makefile for copilot-shell

.PHONY: help deps deps-ci build bundle prepare-package test clean \
        install uninstall \
        lint format preflight start debug rpm

.DEFAULT_GOAL := help

# ── Variables ──
DESTDIR  ?=
PREFIX   ?= /usr/local
LIBDIR    = $(PREFIX)/lib/copilot-shell
DATADIR   = $(PREFIX)/share/copilot-shell
DOCDIR    = $(PREFIX)/share/doc/copilot-shell
BINDIR    = $(PREFIX)/bin

# Platform detection for vendor binaries (maps to vendor/ripgrep/{ARCH}-{PLATFORM}/)
UNAME_S  := $(shell uname -s)
UNAME_M  := $(shell uname -m)
VENDOR_PLATFORM := $(if $(filter Linux,$(UNAME_S)),linux,$(if $(filter Darwin,$(UNAME_S)),darwin,))
VENDOR_ARCH     := $(if $(filter x86_64,$(UNAME_M)),x64,$(if $(filter aarch64 arm64,$(UNAME_M)),arm64,))
VENDOR_RG_DIR   := vendor/ripgrep/$(VENDOR_ARCH)-$(VENDOR_PLATFORM)

help:
	@echo "Usage: make <target>"
	@echo ""
	@echo "Standard targets:"
	@echo "  deps             Install dependencies (npm ci --ignore-scripts)"
	@echo "  build            Build dist/ for distribution (bundle + prepare)"
	@echo "  test             Run the test suite"
	@echo "  clean            Remove generated files"
	@echo "  install          Install to PREFIX (supports DESTDIR for RPM)"
	@echo "  uninstall        Remove installed files"
	@echo ""
	@echo "Build sub-targets:"
	@echo "  bundle           Run esbuild bundling only"
	@echo "  prepare-package  Prepare dist/package.json and assets"
	@echo ""
	@echo "Development:"
	@echo "  lint             Lint the code"
	@echo "  format           Format the code"
	@echo "  preflight        Run full pre-merge checks"
	@echo "  start            Start the CLI"
	@echo "  debug            Start the CLI in debug mode"
	@echo "  rpm              Build RPM package"

# ══════════════════════════════════════════════════════
# Build & Test
# ══════════════════════════════════════════════════════

deps:
	npm ci --ignore-scripts

deps-ci: deps  ## Backward-compatible alias for 'deps'

build: bundle prepare-package

bundle:
	npm run bundle

prepare-package:
	npm run prepare:package

test:
	npm run test

clean:
	npm run clean

# Build artifacts (map to the step that produces them)
dist/cli.js: bundle
dist/package.json: prepare-package

# ══════════════════════════════════════════════════════
# Install / Uninstall
# ══════════════════════════════════════════════════════
# RPM build:   make install DESTDIR=%{buildroot} PREFIX=/usr
# Direct:      make deps && make build && sudo make install

install: dist/cli.js dist/package.json
	@echo "Installing copilot-shell to $(DESTDIR)$(LIBDIR) ..."
	# -- runtime code --
	install -d "$(DESTDIR)$(LIBDIR)"
	install -m 644 dist/cli.js "$(DESTDIR)$(LIBDIR)/cli.js"
	install -m 644 dist/package.json "$(DESTDIR)$(LIBDIR)/package.json"
	@if [ -d dist/node_modules ]; then \
		cp -pr dist/node_modules "$(DESTDIR)$(LIBDIR)/"; \
	fi
	# -- arch-independent data --
	install -d "$(DESTDIR)$(DATADIR)/locales"
	cp -pr dist/locales/* "$(DESTDIR)$(DATADIR)/locales/"
	ln -snf "$(DATADIR)/locales" "$(DESTDIR)$(LIBDIR)/locales"
	@if [ -d hooks ]; then \
		install -d "$(DESTDIR)$(DATADIR)/hooks"; \
		cp -pr hooks/* "$(DESTDIR)$(DATADIR)/hooks/"; \
	fi
	# -- vendor binaries (current platform only) --
	@if [ -n "$(VENDOR_ARCH)" ] && [ -n "$(VENDOR_PLATFORM)" ] && [ -d "dist/$(VENDOR_RG_DIR)" ]; then \
		install -d "$(DESTDIR)$(LIBDIR)/$(VENDOR_RG_DIR)"; \
		install -m 755 dist/$(VENDOR_RG_DIR)/rg "$(DESTDIR)$(LIBDIR)/$(VENDOR_RG_DIR)/rg"; \
	fi
	@if [ -f "dist/vendor/ripgrep/COPYING" ]; then \
		install -d "$(DESTDIR)$(LIBDIR)/vendor/ripgrep"; \
		install -m 644 dist/vendor/ripgrep/COPYING "$(DESTDIR)$(LIBDIR)/vendor/ripgrep/COPYING"; \
	fi
	# -- documentation --
	install -d "$(DESTDIR)$(DOCDIR)"
	install -m 644 dist/README.md "$(DESTDIR)$(DOCDIR)/README.md"
	install -m 644 dist/LICENSE "$(DESTDIR)$(DOCDIR)/LICENSE"
	# -- executables --
	install -d "$(DESTDIR)$(BINDIR)"
	sed 's|@@LIBDIR@@|$(LIBDIR)|g' scripts/cosh-wrapper.sh > "$(DESTDIR)$(BINDIR)/cosh"
	chmod 755 "$(DESTDIR)$(BINDIR)/cosh"
	ln -sf cosh "$(DESTDIR)$(BINDIR)/co"
	ln -sf cosh "$(DESTDIR)$(BINDIR)/copilot"
	@echo "Installed: $(DESTDIR)$(BINDIR)/{cosh,co,copilot}"

uninstall:
	rm -f "$(DESTDIR)$(BINDIR)/cosh" "$(DESTDIR)$(BINDIR)/co" "$(DESTDIR)$(BINDIR)/copilot"
	rm -rf "$(DESTDIR)$(LIBDIR)"
	rm -rf "$(DESTDIR)$(DATADIR)"
	rm -rf "$(DESTDIR)$(DOCDIR)"
	@echo "Uninstalled copilot-shell from $(DESTDIR)$(PREFIX)"

# ══════════════════════════════════════════════════════
# Development helpers (forwarding to npm scripts)
# ══════════════════════════════════════════════════════

lint:
	npm run lint

format:
	npm run format

preflight:
	npm run preflight

start:
	npm run start

debug:
	npm run debug

rpm:
	npm run package:rpm
