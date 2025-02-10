# Creating a self-signed firmware package for the Neato Botvac D3, D3 Pro, D4, D5, and D7

## Table of contents

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

- [Table of contents](#table-of-contents)
- [Motivation](#motivation)
- [Background](#background)
- [Option 1: Use the pregenerated files](#option-1-use-the-pregenerated-files)
  - [Step 1. Download the original firmware images](#step-1-download-the-original-firmware-images)
  - [Step 2. Inject a certificate and signature file](#step-2-inject-a-certificate-and-signature-file)
  - [Step 3. Repackage the firmware](#step-3-repackage-the-firmware)
- [Option 2: Automated Process Using Make](#option-2-automated-process-using-make)
  - [Prerequisites](#prerequisites)
  - [Creating the self-signed firmware package](#creating-the-self-signed-firmware-package)
- [Option 3: Manual Process](#option-3-manual-process)
  - [Step 1: Retrieve the firmware file](#step-1-retrieve-the-firmware-file)
  - [Step 2: Generate a Self-Signed Certificate](#step-2-generate-a-self-signed-certificate)
    - [Step 2a: Create an OpenSSL Configuration File](#step-2a-create-an-openssl-configuration-file)
    - [Step 2b: Generate a Private Key and CSR](#step-2b-generate-a-private-key-and-csr)
    - [Step 2c: Generate a Self-Signed Certificate (Valid for 100 Years)](#step-2c-generate-a-self-signed-certificate-valid-for-100-years)
  - [Step 3: Sign the Firmware File](#step-3-sign-the-firmware-file)
  - [Step 4: Verify the Signature](#step-4-verify-the-signature)
  - [Step 5: Repackage the firmware](#step-5-repackage-the-firmware)

<!-- /code_chunk_output -->

## Motivation

When installing firmware on a Neato Botvac D3, D3 Pro, D4, D5, or D7, the robot verifies the signature of the firmware image to ensure it is authentic and has not been tampered with. It uses a certificate file, included in the firmware package, to verify the signature. If the certificate has expired, the robot will refuse to install the firmware, even if the firmware image itself is valid.

Since Neato Robotics has ceased operations, the last known certificate used to sign the firmware images expires on February 19, 2025.

This repository provides instructions for creating a self-signed firmware package for the Neato Botvac D3, D3 Pro, D4, D5, and D7. This package includes a new certificate and signature that will (hopefully) allow you to install the firmware on your robot even after the original certificate has expired.

**As of February 9, 2025, we do not know if this is the case. If you try this method, please open a GitHub issue or discussion on this repository to report your findings.**

## Background

A Neato `.tgz` firmware images contain three files. For example, in `Neato_4.5.3_189.tgz`, you will find:

1. `Neato_4.5.3_189.bin`: The firmware image itself.
2. `Signing.crt`: The certificate used to sign the firmware image.
3. `Neato_4.5.3_189.signed`: The signature of the firmware image.

The firmware itself is an encrypted binary file that runs on a custom Texas Instruments AM335x chip in the robot, according to extensive work done by Jiska Classen for [her PhD thesis](https://tuprints.ulb.tu-darmstadt.de/11422/). We have no way to modify the firmware itself. However, we can attempt to modify the certificate and signature files.

The `Signing.crt` is a certificate that Neato Robotics obtained from a certificate authority. The corresponding private key was used to sign the firmware image, generating the `.signed` file. The certificate and signature are in standard [OpenSSL](https://www.openssl.org/) format.

When firmware is installed, the robot verifies the signature using the public key from the certificate file before allowing the update to proceed.

Here is where we can potentially bypass the certificate expiration date. If the `.crt` file is used *only* for this step in the process, and is not also used to decrypt the firmware image itself, *and* if the robot does not verify the certificate chain of the `.crt` file, then we can simply generate our own certificate and private key, sign the firmware image with it, and the robot will accept the firmware image because it had a valid signature.

This section covers generating a self-signed certificate and using it to sign the firmware file `Neato_4.5.3_189.bin`.

There are three ways to create a self-signed firmware package:

1. Use the [pregenerated certificate and signatures](#option-1-use-the-pregenerated-files) provided in this repository.
2. Use the [automated Makefile process](#option-2-automated-process-using-make).
3. Follow the [manual steps](#option-3-manual-process) below.

The manual steps are (essentially) identical to the automated process, but you will need to run the commands yourself.

Once you have generated the firmware image, you can install it using the methods described in the main [README.md](../README.md#update-procedure) file of this repository.

## Option 1: Use the pregenerated files

### Step 1. Download the original firmware images

Download the firmware file you wish to use from the Internet Archive. The links are found in the main [README.md](../README.md#firmware-download-links) file of this repository.

### Step 2. Inject a certificate and signature file

Download the corresponding pre-generated `.signed` and `.crt` files provided  in the [self-signed-firmware/pregenerated](./pregenerated/) directory of this repository.

### Step 3. Repackage the firmware

Use a utility such as [7-Zip](https://www.7-zip.org/) to add the `.signed` and `.crt` files to the `.tgz` file, replacing the original files.

## Option 2: Automated Process Using Make

In the [self-signed-firmware/automated](./automated/) directory of this repository, you will find a Makefile that automates the entire process of generating a self-signed firmware package for firmware version 4.5.3_189. This will work on Linux, Windows (with [MSYS2](https://www.msys2.org/) installed), and OS X.

### Prerequisites

You will need to have `make`, `curl`, `tar`, `awk`, `grep`, `sha256sum`, and `openssl` installed on your system. For a Windows system running MSYS2, you can install these packages with:

```
pacman -Syu
pacman -S --needed make curl tar gawk grep coreutils openssl
```

For Ubuntu:

```
sudo apt update
sudo apt install -y make curl tar gawk grep coreutils openssl
```

### Creating the self-signed firmware package

Simply:

1. Open a terminal in the `make-self-signed-firmware` directory
2. Run:
   ```sh
   make
   ```

The `Makefile` will:
- Download the firmware from the Internet Archive
- Verify its SHA256 checksum
- Generate a self-signed certificate valid for 100 years
- Sign the firmware with the new certificate
- Verify the signature
- Package everything into a new `.tgz` file
- Perform final verification of the package

The process provides status updates as it runs and will create a new `Neato_4.5.3_189.tgz` file in the directory when complete.

If you want to start fresh, you can run:
```sh
make clean
```

## Option 3: Manual Process

If you prefer to do things manually, you can follow these steps:

### Step 1: Retrieve the firmware file

Download the firmware file from the Internet Archive and extract the `.bin` file:

```sh
curl https://web.archive.org/web/20240506174708/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.5.3_189.tgz | tar xz Neato_4.5.3_189.bin
```

You can verify that the `.bin` file is correct by comparing its SHA256 hash to the known hash:

```sh
sha256sum Neato_4.5.3_189.bin
```

The hash should be `3d36076fbf3c196ef452b81d54857c75c17ac6eca24ef614aff27a8decc56ef8`.

### Step 2: Generate a Self-Signed Certificate

#### Step 2a: Create an OpenSSL Configuration File

Create a file named `neato.cnf` with the following contents:

```ini
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
x509_extensions = v3_ext

[dn]
CN = www.neato.cloud

[v3_ext]
basicConstraints = critical, CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = www.neato.cloud
DNS.2 = neato.cloud
```

#### Step 2b: Generate a Private Key and CSR

Run the following command to create a new private key and a Certificate Signing Request (CSR):

```sh
openssl req -new -newkey rsa:2048 -nodes -keyout Signing.key -out Signing.csr -config neato.cnf
```

- `-newkey rsa:2048`: Creates a 2048-bit RSA key.
- `-nodes`: No password protection on the key.
- `-keyout Signing.key`: Saves the private key.
- `-out Signing.csr`: Saves the CSR.
- `-config neato.cnf`: Uses the configuration file to set the correct subject and extensions.

#### Step 2c: Generate a Self-Signed Certificate (Valid for 100 Years)

```sh
openssl x509 -req -days 36500 -in Signing.csr -signkey Signing.key -out Signing.crt -extfile neato.cnf -extensions v3_ext
```

- `-req`: Indicates a CSR-based certificate generation.
- `-days 36500`: Sets expiration to 100 years.
- `-signkey Signing.key`: Uses the private key to self-sign.
- `-out Signing.crt`: Saves the final certificate.
- `-extfile neato.cnf -extensions v3_ext`: Ensures the required X.509 extensions are included.

### Step 3: Sign the Firmware File

Next, we will sign `Neato_4.5.3_189.bin` using the private key.

```sh
openssl dgst -sha256 -sign Signing.key -out Neato_4.5.3_189.signed Neato_4.5.3_189.bin
```

- `-sha256`: Uses SHA-256 hashing.
- `-sign Signing.key`: Signs using the private key.
- `-out Neato_4.5.3_189.signed`: Saves the signature.

### Step 4: Verify the Signature

You can verify the signature using the public key in the certificate:

```sh
openssl x509 -pubkey -in Signing.crt -noout -out pubkey.pem
openssl dgst -sha256 -verify pubkey.pem -signature Neato_4.5.3_189.signed Neato_4.5.3_189.bin
```

If the signature is valid, you will see:

```
Verified OK
```

If it is not valid, something went wrong with the steps above.

### Step 5: Repackage the firmware

Finally, create a new `.tgz` file containing the original `.bin` file and the new `Signing.crt` and `Neato_4.5.3_189.signed` files:

```sh
tar czf Neato_4.5.3_189_modified.tgz Neato_4.5.3_189.bin Signing.crt Neato_4.5.3_189.signed
```
