.SILENT:  # Makes all commands silent by default

# Variables
BUILD_DIR = build
VERIFY_DIR = $(BUILD_DIR)/verify
FIRMWARE_URL = https://web.archive.org/web/20240506174708/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.5.3_189.tgz
FIRMWARE_FILE = Neato_4.5.3_189.tgz
ORIGINAL_FIRMWARE_FILE = $(BUILD_DIR)/Original_$(FIRMWARE_FILE)
FIRMWARE_BIN = $(BUILD_DIR)/Neato_4.5.3_189.bin
SIGNED_FILE = $(BUILD_DIR)/Neato_4.5.3_189.signed
CERT_FILE = $(BUILD_DIR)/Signing.crt
KEY_FILE = $(BUILD_DIR)/Signing.key
CSR_FILE = $(BUILD_DIR)/Signing.csr
PUBKEY_FILE = $(BUILD_DIR)/pubkey.pem
CONFIG_FILE = $(BUILD_DIR)/neato.cnf
EXPECTED_SHA256 = 3d36076fbf3c196ef452b81d54857c75c17ac6eca24ef614aff27a8decc56ef8

.PHONY: all clean verify_sha256 verify_sign verify_package

all: $(FIRMWARE_FILE) verify_package

$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

# Download the firmware
$(ORIGINAL_FIRMWARE_FILE): | $(BUILD_DIR)
	echo "📥 Downloading firmware..."
	curl -s -o $@ $(FIRMWARE_URL)
	echo "✅ Download complete"

# Extract the firmware (initial extraction)
$(FIRMWARE_BIN): $(ORIGINAL_FIRMWARE_FILE)
	echo "📦 Extracting firmware..."
	cd $(BUILD_DIR) && tar --warning=no-unknown-keyword -xzf $(notdir $<) $(notdir $@)
	touch $@
	echo "✅ Extraction complete"

# Generate OpenSSL config
$(CONFIG_FILE): | $(BUILD_DIR)
	echo "📝 Generating OpenSSL config..."
	echo "[req]" > $@
	echo "default_bits = 2048" >> $@
	echo "prompt = no" >> $@
	echo "default_md = sha256" >> $@
	echo "distinguished_name = dn" >> $@
	echo "x509_extensions = v3_ext" >> $@
	echo "[dn]" >> $@
	echo "CN = www.neato.cloud" >> $@
	echo "[v3_ext]" >> $@
	echo "basicConstraints = critical, CA:FALSE" >> $@
	echo "keyUsage = critical, digitalSignature, keyEncipherment" >> $@
	echo "extendedKeyUsage = serverAuth, clientAuth" >> $@
	echo "subjectAltName = @alt_names" >> $@
	echo "[alt_names]" >> $@
	echo "DNS.1 = www.neato.cloud" >> $@
	echo "DNS.2 = neato.cloud" >> $@
	echo "✅ Config generated"

# Generate key and certificate
$(KEY_FILE) $(CSR_FILE): $(CONFIG_FILE)
	echo "🔑 Generating keys..."
	cd $(BUILD_DIR) && openssl req -new -newkey rsa:2048 -nodes \
		-keyout $(notdir $(KEY_FILE)) \
		-out $(notdir $(CSR_FILE)) \
		-config $(notdir $(CONFIG_FILE)) 2>/dev/null
	echo "✅ Keys generated"

$(CERT_FILE): $(CSR_FILE) $(KEY_FILE) $(CONFIG_FILE)
	echo "📜 Generating certificate..."
	cd $(BUILD_DIR) && openssl x509 -req -days 36500 \
		-in $(notdir $(CSR_FILE)) \
		-signkey $(notdir $(KEY_FILE)) \
		-out $(notdir $(CERT_FILE)) \
		-extfile $(notdir $(CONFIG_FILE)) \
		-extensions v3_ext 2>/dev/null
	echo "✅ Certificate generated"

# Extract public key
$(PUBKEY_FILE): $(CERT_FILE)
	echo "🔐 Extracting public key..."
	cd $(BUILD_DIR) && openssl x509 -pubkey \
		-in $(notdir $(CERT_FILE)) \
		-noout -out $(notdir $(PUBKEY_FILE))
	echo "✅ Public key extracted"

# Sign the firmware
$(SIGNED_FILE): $(FIRMWARE_BIN) $(KEY_FILE)
	echo "✍️  Signing firmware..."
	cd $(BUILD_DIR) && openssl dgst -sha256 \
		-sign $(notdir $(KEY_FILE)) \
		-out $(notdir $(SIGNED_FILE)) \
		$(notdir $(FIRMWARE_BIN))
	echo "✅ Firmware signed"

# Verify SHA256 before signing
verify_sha256: $(FIRMWARE_BIN)
	echo "🔍 Verifying SHA256 checksum..."
	cd $(BUILD_DIR) && sha256sum $(notdir $(FIRMWARE_BIN)) | \
		awk '{print $$1}' | \
		grep -q "^$(EXPECTED_SHA256)$$" && \
		echo "✅ SHA256 checksum verified!" || \
		{ echo "❌ SHA256 checksum FAILED!"; exit 1; }

# Verify signature after signing
verify_sign: $(SIGNED_FILE) $(PUBKEY_FILE)
	echo "🔍 Verifying signature..."
	cd $(BUILD_DIR) && openssl dgst -sha256 \
		-verify $(notdir $(PUBKEY_FILE)) \
		-signature $(notdir $(SIGNED_FILE)) \
		$(notdir $(FIRMWARE_BIN)) >/dev/null && \
		echo "✅ Signature verified!" || \
		{ echo "❌ Signature verification FAILED!"; exit 1; }

# Package the final firmware
$(FIRMWARE_FILE): $(FIRMWARE_BIN) $(CERT_FILE) $(SIGNED_FILE) verify_sha256 verify_sign
	echo "📦 Creating modified firmware package..."
	cd $(BUILD_DIR) && tar czf ../$@ \
		$(notdir $(FIRMWARE_BIN)) \
		$(notdir $(CERT_FILE)) \
		$(notdir $(SIGNED_FILE))
	echo "✅ Firmware repackaged as $@"

# Final verification of the package
verify_package: $(FIRMWARE_FILE)
	echo "🔍 Performing final verification..."
	rm -rf $(VERIFY_DIR)
	mkdir -p $(VERIFY_DIR)
	cd $(VERIFY_DIR) && tar xzf ../../$(FIRMWARE_FILE)
	cd $(VERIFY_DIR) && openssl x509 -pubkey -in $(notdir $(CERT_FILE)) -noout -out verify_pubkey.pem
	cd $(VERIFY_DIR) && openssl dgst -sha256 \
		-verify verify_pubkey.pem \
		-signature $(notdir $(SIGNED_FILE)) \
		$(notdir $(FIRMWARE_BIN)) >/dev/null && \
		echo "✅ Final package verification successful!" || \
		{ echo "❌ Final package verification FAILED!"; exit 1; }
	echo "🎉 All done!"

clean:
	echo "🧹 Cleaning up..."
	rm -rf $(BUILD_DIR)
	rm -f $(FIRMWARE_FILE)
	echo "✨ All clean!"