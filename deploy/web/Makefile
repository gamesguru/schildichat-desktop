# SchildiChat Web Deploy Makefile
# Run on VPS to download latest build from GitHub and install to /var/www/schil
#
# Prerequisites: gh CLI authenticated (gh auth login)

REPO ?= gamesguru/schildichat-desktop
WEB_ROOT ?= /var/www/schil
ARTIFACT_NAME ?= schildichat-web
TMP_DIR ?= /tmp/schildichat-deploy
BACKUP_DIR ?= $(WEB_ROOT)/.tmp/old-$(shell date +%Y-%m-%d)

# Config files to preserve across deploys
CONFIGS := config.json config.schil.nutra.tk.json

.PHONY: install
install: download backup deploy cleanup
	@echo "Done. Previous version backed up to $(BACKUP_DIR)"

.PHONY: download
download:
	@echo "Downloading latest build artifact from $(REPO)..."
	rm -rf $(TMP_DIR)
	mkdir -p $(TMP_DIR)
	gh run download --repo $(REPO) --name $(ARTIFACT_NAME) --dir $(TMP_DIR)
	@# Extract tarball if present inside the artifact zip
	cd $(TMP_DIR) && for f in *.tar.gz; do [ -f "$$f" ] && tar xf "$$f" && rm "$$f"; done || true
	@# Flatten if extracted into a subdirectory (e.g. schildichat-web-x.y.z/)
	@cd $(TMP_DIR) && if [ $$(ls -d */ 2>/dev/null | wc -l) -eq 1 ]; then \
		subdir=$$(ls -d */); \
		mv "$$subdir"* . 2>/dev/null; \
		rmdir "$$subdir" 2>/dev/null; \
	fi || true
	@echo "Downloaded to $(TMP_DIR)"
	@ls $(TMP_DIR)

.PHONY: backup
backup:
	@echo "Backing up $(WEB_ROOT) to $(BACKUP_DIR)..."
	mkdir -p $(BACKUP_DIR)
	cd $(WEB_ROOT) && find . -maxdepth 1 ! -name '.' ! -name '.tmp' ! -name '.git' \
		-exec cp -a {} $(BACKUP_DIR)/ \;
	@echo "Backup complete"

.PHONY: deploy
deploy:
	@echo "Preserving config files..."
	@for cfg in $(CONFIGS); do \
		[ -f $(WEB_ROOT)/$$cfg ] && cp $(WEB_ROOT)/$$cfg $(TMP_DIR)/_preserve_$$cfg 2>/dev/null; \
	done || true
	@echo "Installing to $(WEB_ROOT)..."
	cd $(WEB_ROOT) && find . -maxdepth 1 ! -name '.' ! -name '.tmp' ! -name '.git' -exec rm -rf {} +
	cp -a $(TMP_DIR)/* $(WEB_ROOT)/ 2>/dev/null || true
	@for cfg in $(CONFIGS); do \
		[ -f $(WEB_ROOT)/_preserve_$$cfg ] && mv $(WEB_ROOT)/_preserve_$$cfg $(WEB_ROOT)/$$cfg; \
	done || true
	@echo "Installed"

.PHONY: cleanup
cleanup:
	@echo "Cleaning up..."
	rm -rf $(TMP_DIR)

.PHONY: rollback
rollback:
	@if [ ! -d "$(BACKUP_DIR)" ]; then echo "No backup found at $(BACKUP_DIR)"; exit 1; fi
	@echo "Rolling back from $(BACKUP_DIR)..."
	cd $(WEB_ROOT) && find . -maxdepth 1 ! -name '.' ! -name '.tmp' ! -name '.git' -exec rm -rf {} +
	cp -a $(BACKUP_DIR)/* $(WEB_ROOT)/
	@echo "Rolled back"
