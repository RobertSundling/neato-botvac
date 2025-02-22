# Adding a neato.cloud precertificate to a Neato Botvac D3, D3 Pro, D4, D5, and D7 firmware package

This guide will help you replace an expired certificate in a Neato Botvac D3, D3 Pro, D4, D5, and D7 firmware package with an unexpired neato.cloud precertificate.

This method allows installing firmware until at least March 19, 2026, when the most recent neato.cloud precertificate expires.

**If you wish to simply download a pre-made firmware package with a precertificate already installed, see the [main README](../README.md) for this repository.**

## Table of contents

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Table of contents](#table-of-contents)
- [Motivation](#motivation)
- [Details](#details)
  - [Prerequisites](#prerequisites)
- [How to replace the certificate (automatic method)](#how-to-replace-the-certificate-automatic-method)
- [How to replace the certificate (manual method)](#how-to-replace-the-certificate-manual-method)

<!-- /code_chunk_output -->

## Motivation

When installing firmware on a Neato Botvac D3, D3 Pro, D4, D5, or D7, the robot verifies the signature of the firmware image to ensure it is authentic and has not been tampered with. It uses a certificate file, included in the firmware package, to verify the signature. If the certificate has expired, the robot will refuse to install the firmware, even if the firmware image itself is valid.

Neato Robotics has ceased operations. The last known certificate included with the firmware images expired on February 19, 2025.

However, on February 18, 2025, the parent company of Neato Robotics, Vorwerk & Co. KG, acquired a new certificate that expires on March 19, 2026. They have not released any new firmware with this certificate, so we do not have the certificate itself. However, we do have the *precertificate* from the the public [Certificate Transparency](https://certificate.transparency.dev/) logs. Thankfully, this precertificate can be used to verify firmware signatures, because the robot does not care about the extra "CT Precertificate Poison" section that is present in precertificates.

## Details

A Neato `.tgz` firmware images contain three files. For example, in `Neato_4.5.3_189.tgz`, you will find:

1. `Neato_4.5.3_189.bin`: The firmware image itself.
2. `Signing.crt`: The certificate used to sign the firmware image.
3. `Neato_4.5.3_189.signed`: The signature of the firmware image.


The `Signing.crt` is a certificate that Neato Robotics obtained from a certificate authority. The corresponding private key was used to sign the firmware image, generating the `.signed` file. The certificate and signature are in standard [OpenSSL](https://www.openssl.org/) format.

When firmware is installed, the robot verifies the signature using the public key from the certificate file before allowing the update to proceed, checking that the certificate has not yet expired.

We can replace the expired certificate with a new one, as long as the new certificate uses the same private key as the old one, and the same signature will continue to be valid.

The precertificate expiring in 2026 does use this same private key. And, although precertificates have an extra section called "CT Precertificate Poison" that is not present in certificates, the robots ignore that section. Therefore, a precertificate works just as well as a certificate for the robot to verify the signature of the firmware image.

### Prerequisites

To create the new firmware using the `Makefile` provided here, you will need to have `make`, `curl`, `tar`, and `openssl` installed on your system.

**Note:** If `openssl` is not available, you can skip signature verification and build the firmware images anyway by adding the `SKIP_VERIFY=1` option to the `make` command. The verification step is not strictly necessary but is just there to make sure everything is working as expected.

## How to replace the certificate (automatic method)

This directory contains a Linux `Makefile` that automates the process of replacing the expired certificate with a neato.cloud precertificate. This will download all known official firmware images from the Internet Archive, extract them, replace the certificates, and repack the images.

To use it, simply download this repository to a Linux system and run

```bash
git clone https://github.com/RobertSundling/neato-botvac.git
cd neato-botvac/precertificate-firmware
make
```

This will create a folder called `firmware` with the modified firmware images. If you do not have `openssl` installed, you can skip signature verification by instead running:

```bash
git clone https://github.com/RobertSundling/neato-botvac.git
cd neato-botvac/precertificate-firmware
make SKIP_VERIFY=1
```

This will build the modified firmware images without verifying the signatures.

## How to replace the certificate (manual method)

If you prefer to do this manually, rather than using the provided Makefile, you can follow these steps:

1. Download the neato.cloud precertificate from this repository. All known precertificates are located in the [precertificate-firmware/precertificates ](./precertificates/) directory, organized into directories by expiration date. You can also obtain the precertificate yourself from a Certificate Transparency list site such as [crt.sh](https://crt.sh/?q=neato.cloud).

2. Download the firmware image you want to install from the Internet Archive. Links to the firmware images are found in the [main README](../README.md).

3. Extract the `.tgz` file using:

    ```bash
    tar -xzf Neato_4.5.3_189.tgz
    ```
    You may receive a warning about unknown extended header keywords due to MacOS-specific metadata in the archive. This is normal and can be ignored, or can be suppressed using the `--warning=no-unknown-keyword` option with tar.

4. Replace the `Signing.crt` file with the neato.cloud precertificate you downloaded in step 1.

5. View the details of the certificate using:

    ```bash
    openssl x509 -in Signing.crt -text -noout | grep 'Not After'
    ```

    You should see the expiration date of the certificate. If it is before March 19, 2026, you have not replaced the certificate correctly. If you do not have `openssl` installed, you will need to skip this step.

6. If you wish to verify the signature of the firmware image, and make sure that the certificate matches, you can do so using:

    ```bash
    openssl x509 -pubkey -in Signing.crt -noout -out pubkey.pem
    openssl dgst -verify pubkey.pem -signature *.signed *.bin
    rm pubkey.pem
    ```

    If the signature is valid, you will see `Verified OK` after the second command. If you do not have `openssl` installed, you will need to skip this step.

7. Repack the `.tgz` file using:

    ```bash
    tar -czf Neato_4.5.3_189.tgz Signing.crt Neato_4.5.3_189.bin Neato_4.5.3_189.signed
    ```
