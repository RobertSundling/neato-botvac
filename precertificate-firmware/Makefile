# Update Neato Botvac D3, D3 Pro, D4, D5, and D7 firmware with current certificates

.SILENT:  # Makes all commands silent by default

SKIP_VERIFY ?= 0

BASE_420=Neato_4.2.0_102
BASE_453=Neato_4.5.3_189
BASE_460=Neato_4.6.0_72

URL_420=https://web.archive.org/web/20240506174833/https://neatorobotics-ota.s3.amazonaws.com/production/$(BASE_420).tgz
URL_453=https://web.archive.org/web/20240506174708/https://neatorobotics-ota.s3.amazonaws.com/production/$(BASE_453).tgz
URL_460=https://web.archive.org/web/20240506174741/https://neatorobotics-ota.s3.amazonaws.com/production/$(BASE_460).tgz

VERSIONS=420 453 460

# Use this certificate
CURRENT_CERTIFICATE=precertificates/2026-03-19/Signing.crt
CERTIFICATE_NAME=Signing.crt

TMP_DIR=tmp
DOWNLOAD_DIR=$(TMP_DIR)/download
WORK_DIR=$(TMP_DIR)/work
OUTPUT_DIR=firmware

.PHONY: all clean

# Default target builds all versions
all: $(foreach ver,$(VERSIONS),$(OUTPUT_DIR)/$(BASE_$(ver)).tgz)
	echo ""
	echo "🎉 All firmware files built successfully!"

# Clean up previous builds
clean:
	rm -rf $(TMP_DIR) $(OUTPUT_DIR)

# Download the firmware files
define download_rule
$(DOWNLOAD_DIR)/$(BASE_$(1)).tgz:
	echo ""
	echo "📥 Downloading firmware ($(1))..."
	mkdir -p $(DOWNLOAD_DIR)
	curl -s -o $$@ "$(URL_$(1))"
	echo "✅ Downloaded $$(notdir $$@)"
endef

$(foreach ver,$(VERSIONS),$(eval $(call download_rule,$(ver))))

# Extract the firmware files and remove any extraneous files
$(WORK_DIR)/%: $(DOWNLOAD_DIR)/%.tgz
	echo "📦 Extracting firmware..."
	mkdir -p $@
	tar --warning=no-unknown-keyword -xzf $< -C $@
	rm -f $@/"._$(CERTIFICATE_NAME)"
	echo "✅ Extracted files from $$(basename $<)"

# Move the certificate file into the work dir and verify the signature.
$(WORK_DIR)/%/$(CERTIFICATE_NAME): $(WORK_DIR)/% $(CURRENT_CERTIFICATE)
	echo "📜 Copying certificate to work dir..."
	cp $(CURRENT_CERTIFICATE) $@
	echo "✅ Replaced certificate with $(CURRENT_CERTIFICATE)"
	if [ "$(SKIP_VERIFY)" = "0" ]; then \
		echo "🔍 Verifying signature..." && \
		cd $(@D) && openssl x509 -pubkey -in $(CERTIFICATE_NAME) -noout -out pubkey.pem && \
		openssl dgst -sha256 \
			-verify pubkey.pem \
			-signature *.signed \
			*.bin >/dev/null && \
		echo "✅ Signature verified with new certificate" || \
		{ echo "❌ Signature verification FAILED!"; exit 1; } && \
		rm -f pubkey.pem; \
	else \
		echo "⚠️ Skipping signature verification (SKIP_VERIFY=1)"; \
	fi

$(OUTPUT_DIR)/%.tgz: $(WORK_DIR)/% $(WORK_DIR)/%/$(CERTIFICATE_NAME)
	echo "📦 Building $$(basename $@)..."
	mkdir -p $(OUTPUT_DIR)
	cd $(WORK_DIR)/$* && tar -czf "$(CURDIR)/$@" *
	rm -rf $(TMP_DIR)
	echo "✅ $(OUTPUT_DIR)/$* built successfully!"
