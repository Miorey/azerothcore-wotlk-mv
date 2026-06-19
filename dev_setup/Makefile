SHELL := /bin/bash
ROOT_DIR := $(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))/../
ETC_DIR  := $(ROOT_DIR)env/dist/etc

-include $(ROOT_DIR)dev_setup/.env
export

RED=\033[1;31m
CYA=\033[1;36m
GRN=\033[0;32m
END=\033[0m

.PHONY: help setup conf modules-conf check-env

help:
	@echo -e ""
	@echo -e " $(CYA)dev_setup targets$(END)"
	@echo -e ""
	@echo -e "  $(CYA)setup$(END)         — full setup: copy and patch all conf files (run on fresh clone)"
	@echo -e "  $(CYA)conf$(END)          — copy and patch worldserver.conf + authserver.conf from CICD"
	@echo -e "  $(CYA)modules-conf$(END)  — copy module conf files (transmog.conf etc.) from CICD"
	@echo -e "  $(CYA)check-env$(END)     — verify .env exists and required variables are set"
	@echo -e ""
	@echo -e " First time? Copy $(CYA).env.dist$(END) to $(CYA).env$(END) and fill in your local values."
	@echo -e ""

check-env:
	@if [ ! -f "$(ROOT_DIR)dev_setup/.env" ]; then \
		echo -e "$(RED)ERROR:$(END) dev_setup/.env not found."; \
		echo -e "       Copy dev_setup/.env.dist to dev_setup/.env and fill in your values."; \
		exit 1; \
	fi
	@for var in CICD_PATH DB_HOST DB_PORT DB_USER DB_PASSWORD DB_AUTH DB_WORLD DB_CHARACTERS DATA_DIR; do \
		if [ -z "$${!var}" ]; then \
			echo -e "$(RED)ERROR:$(END) $$var is not set in dev_setup/.env"; \
			exit 1; \
		fi; \
	done
	@echo -e "$(GRN)✓$(END) .env looks good"

conf: check-env
	@echo -e "$(CYA)conf$(END): patching worldserver.conf from CICD..."
	@mkdir -p $(ETC_DIR)
	@cp $(ROOT_DIR)$(CICD_PATH)/wow/world/etc/worldserver.conf $(ETC_DIR)/worldserver.conf
	@sed -i 's|^LoginDatabaseInfo.*|LoginDatabaseInfo     = "$(DB_HOST);$(DB_PORT);$(DB_USER);$(DB_PASSWORD);$(DB_AUTH)"|' \
		$(ETC_DIR)/worldserver.conf
	@sed -i 's|^WorldDatabaseInfo.*|WorldDatabaseInfo     = "$(DB_HOST);$(DB_PORT);$(DB_USER);$(DB_PASSWORD);$(DB_WORLD)"|' \
		$(ETC_DIR)/worldserver.conf
	@sed -i 's|^CharacterDatabaseInfo.*|CharacterDatabaseInfo = "$(DB_HOST);$(DB_PORT);$(DB_USER);$(DB_PASSWORD);$(DB_CHARACTERS)"|' \
		$(ETC_DIR)/worldserver.conf
	@sed -i 's|^DataDir = .*|DataDir = "$(DATA_DIR)"|' $(ETC_DIR)/worldserver.conf
	@sed -i 's|^LogsDir = .*|LogsDir = "$(LOGS_DIR)"|' $(ETC_DIR)/worldserver.conf
	@echo -e "$(GRN)✓$(END) worldserver.conf written to $(ETC_DIR)/worldserver.conf"

	@echo -e "$(CYA)conf$(END): patching authserver.conf from CICD..."
	@cp $(ROOT_DIR)$(CICD_PATH)/wow/auth/etc/authserver.conf $(ETC_DIR)/authserver.conf
	@sed -i 's|^LoginDatabaseInfo.*|LoginDatabaseInfo = "$(DB_HOST);$(DB_PORT);$(DB_USER);$(DB_PASSWORD);$(DB_AUTH)"|' \
		$(ETC_DIR)/authserver.conf
	@sed -i 's|^LogsDir = .*|LogsDir = "$(LOGS_DIR)"|' $(ETC_DIR)/authserver.conf
	@echo -e "$(GRN)✓$(END) authserver.conf written to $(ETC_DIR)/authserver.conf"

modules-conf: check-env
	@echo -e "$(CYA)modules-conf$(END): copying module conf files from CICD..."
	@mkdir -p $(ETC_DIR)/modules
	@cp $(ROOT_DIR)$(CICD_PATH)/wow/world/etc/modules/*.conf \
	   $(ROOT_DIR)$(CICD_PATH)/wow/world/etc/modules/*.conf.dist \
	   $(ETC_DIR)/modules/ 2>/dev/null || true
	@echo -e "$(GRN)✓$(END) module confs copied to $(ETC_DIR)/modules/"

setup: conf modules-conf
	@echo -e ""
	@echo -e "$(GRN)Dev setup complete.$(END)"
	@echo -e "Reload CMake in CLion if needed, then build and start the worldserver."
