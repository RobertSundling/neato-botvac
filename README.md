# Neato Botvac D3, D3 Pro, D4, D5, and D7 firmware

**Quick tip:** If you came here just to get firmware with a non-expired certificate, and otherwise know what you're doing, download the first link in the [Firmware download links](#firmware-download-links) section. It's good through **February 19, 2025** (which is unfortunately fast approaching, so don't wait!).

## Table of contents

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

- [Table of contents](#table-of-contents)
- [Introduction](#introduction)
- [Firmware download links](#firmware-download-links)
- [Motivation](#motivation)
- [Update procedure](#update-procedure)
  - [WARNING](#warning)
  - [Before starting](#before-starting)
  - [Installing the firmware](#installing-the-firmware)
  - [After the update](#after-the-update)
- [Updating faster](#updating-faster)
- [USB OTG Cable](#usb-otg-cable)
- [Factory reset](#factory-reset)
- [Firmware certificates and signatures](#firmware-certificates-and-signatures)
- [Bypassing certificate expiration](#bypassing-certificate-expiration)
  - [Easy bypass method (through February 19, 2025)](#easy-bypass-method-through-february-19-2025)
  - [Working advanced methods (after February 19, 2025)](#working-advanced-methods-after-february-19-2025)
    - [Factory reset and blocking date acquisition (easiest)](#factory-reset-and-blocking-date-acquisition-easiest)
    - [Faking the date (harder)](#faking-the-date-harder)
  - [Potential advanced method: Self-signed certificate](#potential-advanced-method-self-signed-certificate)
    - [Automated Process Using Make](#automated-process-using-make)
      - [Prerequisities](#prerequisities)
      - [Creating the self-signed firmware package](#creating-the-self-signed-firmware-package)
    - [Manual Process](#manual-process)
      - [Step 1: Retrieve the firmware file](#step-1-retrieve-the-firmware-file)
      - [Step 2: Generate a Self-Signed Certificate](#step-2-generate-a-self-signed-certificate)
        - [Step 2a: Create an OpenSSL Configuration File](#step-2a-create-an-openssl-configuration-file)
        - [Step 2b: Generate a Private Key and CSR](#step-2b-generate-a-private-key-and-csr)
        - [Step 2c: Generate a Self-Signed Certificate (Valid for 100 Years)](#step-2c-generate-a-self-signed-certificate-valid-for-100-years)
      - [Step 3: Sign the Firmware File](#step-3-sign-the-firmware-file)
      - [Step 4: Verify the Signature](#step-4-verify-the-signature)
      - [Step 5: Repackage the firmware](#step-5-repackage-the-firmware)
- [Firmware version notes](#firmware-version-notes)
  - [4.5.3_189](#453_189)
  - [4.6.0_72](#460_72)
  - [4.2.0_102](#420_102)
- [Document revisions](#document-revisions)

<!-- /code_chunk_output -->

## Introduction

This document contains links to firmware images from the Internet Archive mirror of the official Neato Robotics server for the Neato Botvac D3, D3 Pro, D4, D5, and D7 robot vacuums as well as information on how to install them, including how to bypass expired certificates.

## Firmware download links

| Version   | Firmware Date | Certificate Validity         | Download   |
|-----------|---------------|------------------------------|------------|
| 4.5.3_189 | 2019-10-29    | **2024-01-19 to 2025-02-19** | [Neato_4.5.3_189.tgz (via Internet Archive)](https://web.archive.org/web/20240506174708/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.5.3_189.tgz) |
| 4.6.0_72  | 2020-01-27    | 2019-03-20 to 2021-03-19     | [Neato_4.6.0_72.tgz (via Internet Archive)](https://web.archive.org/web/20240506174741/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.6.0_72.tgz) |
| 4.2.0_102 | 2018-07-12    | 2018-01-17 to 2019-05-11     | [Neato_4.2.0_102.tgz (via Internet Archive)](https://web.archive.org/web/20240506174833/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.2.0_102.tgz) |

Most users should choose `4.5.3_189`. See the [Firmware Version Notes](#firmware-version-notes) section below for more information on these firmware versions.

*Note: As of February 8, 2025, the firmware images are no longer available directly from the official Neato Robotics server, which now returns an "Access Denied" error.* Copies remain accessible through the Internet Archive.

## Motivation

Neato Robotics, a subsidiary of Vorwerk & Co. KG, ceased operations in 2023. This document is an effort to keep their robots running.

When troubleshooting or repairing a Neato Botvac D3, D3 Pro, D4, D5, or D7, it may be necessary to install new firmware on the robot. The firmware is generally no longer obtainable through the Neato app, even though the app may suggest that an update is available. Resetting your robot may revert it to outdated firmware.

Unfortunately, some of the official firmware images can no longer be installed as-is due to expired certificates. This document provides information on how to bypass this limitation. There are many links to repackaged versions of the firmware images with different expiration dates available online, but generally those are expired now as well.

I created this document to share what I learned while exploring the options to update my Botvac D7 Connected to the `4.5.3_189` firmware after a factory reset reverted it to `4.2.0_102`. I found that the app could not update the firmware and all of the firmware images shared on forums had expired certificates. I hope this document will help others in the same situation.

## Update procedure

### WARNING

**Please note that updating firmware involves inherent risks, including the possibility of making your robot inoperable or "bricking" it. By following these procedures, you acknowledge and accept these risks. If you are uncertain about these steps or their consequences, it is advised not to proceed. The information provided here is for educational purposes only, and the author assumes no liability for any damage or loss resulting from its use.**

### Before starting

You may wish to read the [Updating faster](#updating-faster) section below to speed up the install process.

If you need to install firmware with an expired certificate, see the [Bypassing certificate expiration](#bypassing-certificate-expiration) section below.

### Installing the firmware

To install firmware on your Neato Botvac, you generally do *not* need to press any buttons or perform a [factory reset](#factory-reset) on your robot. You really only have to plug in a flash drive and the robot handles the rest.

First, prepare a USB flash drive:

1. Obtain a USB flash drive and ensure it is formatted as FAT-32 (not exFAT).

   > **Note:** You may need to try a couple drives. Some users have said the drive needs to be at least 16 GB (even though the firmware images themselves are under 16 MB). Others have reported problems with USB 3.x drives, with the upgrade freezing during the "copying logs" stage, and suggest USB 2.0 drives. I've had perfect success with USB 3.x drives, myself, but if you have issues, you may wish to try another drive.
  
2. Create a folder on the flash drive named `RobotData`. Capitalization is important.
3. Copy the firmware `.tgz` file directly into the `RobotData` folder on the flash drive. Do not extract the contents of the archive; the robot will do this itself.

At this point, I suggest having the robot fully charged and on its charging base. Ensure the robot is turned on. Then:

1. Remove the robot's dustbin to expose its micro USB port.
2. Plug the flash drive into your robot using a [USB OTG cable](#usb-otg-cable). 
3. The robot will automatically detect the firmware file and begin the update process, with the lights on the left flashing rhythmically.
4. The robot will play a sound and reboot once the update is complete.

In my experience, if I have cleared the log files first (as explained in the [Updating faster](#updating-faster) section), the update process only takes a minute or two.

**You would be well-advised to *not* interrupt a firmware update while in progress.** 

### After the update

The next time you open the Neato app, it will inform you that the firmware update was successful. You can check the firmware version in the Neato app or with the free [Neato Toolio](https://github.com/jdredd87/NeatoToolio) utility.

When the robot processes an update, whether it is successful or not, it will generally delete the firmware file from the flash drive. This is likely to ensure that when the robot reboots it doesn't try to update itself again. If you are planning to update a second robot, you will need to copy the firmware file back onto the flash drive and start again.

If the firmware file was not deleted, it was likely in the wrong folder or had the wrong name and was not recognized by the robot.

## Updating faster

Before installing new firmware, your robot will create a folder called `RobotLogs` on the USB drive and fill it with things like log files and crash dumps. This can take a long time.

You can use the free [Neato Toolio](https://github.com/jdredd87/NeatoToolio) utility to clear these files from the robot. If done in advance, this can speed up the firmware update process.

Connecting your robot using Neato Toolio is outside the scope of this document, however it is fairly straightforward with an appropriate USB cable connected to your computer. Once you have it connected and talking, in Toolio use the "ClearFiles" option under "Tools" and click the second button to clear all of the data. (You may also safely click each of the two buttons in turn if you want to delete the files in stages.)

There is no known reason to keep these files, as they are likely for Neato support to diagnose issues. Deleting them does not result in the loss of any history, maps, or settings on your robot.

## USB OTG Cable

You will need a USB OTG ("On-The-Go") cable to connect a USB flash drive to your Neato Botvac. Your Botvac may have come with one, but there is no need to use the official cable (which sells for around $30 US).

If you do not already have one, what you are looking for is an [OTG Micro USB 2.0 Male to USB Female cable](https://www.google.com/search?q=OTG+Micro+USB+2.0+Male+to+USB+Female+cable). You can find them on Amazon or eBay for a few dollars. (They are mostly used to connect USB devices to older smartphones and tablets.)

## Factory reset

The Botvac is supposed to keep a secondary, backup firmware image from when it was originally shipped. If something goes wrong and your robot is not in a usable state, you may wish to attempt to have the bot revert to this backup image by performing a factory reset (also sometimes called a "hard reset"). This involves a process of holding down the front bumper in a certain way while pressing and releasing the power button, at certain intervals.

The specific steps to do this are [outlined in this reddit comment by u/woutske](https://www.reddit.com/r/botvac/comments/c7hznj/comment/esjzc1i/). Note that not all of the steps may be necessary.

Of course, it is not guaranteed that this process will work or that you will always be able to do this.

## Firmware certificates and signatures

The firmware images are signed by Neato Robotics, and the certificates are valid for a certain period of time. **If the certificate has expired, the robot will *not* accept the firmware image.**

This has, historically, been the largest stumbling block to updating firmware outside of the Neato app.

Further, as of February 8, 2025, the neato.cloud certificate [still has not been renewed according to crt.sh](https://crt.sh/?q=neato.cloud). Previously, the certificate was regularly renewed weeks in advance. Also, the official Neato Robotics download site now returns "Access Denied" messages when trying to obtain the firmware image files.

This does not bode well and, unfortunately, almost certainly means that the true end of life for this firmware is indeed **February 19, 2025**. One of the methods in the [Working advanced methods](#working-advanced-methods-after-february-19-2025) section will likely need to be used to bypass the firmware expiration date going forwards.

## Bypassing certificate expiration

### Easy bypass method (through February 19, 2025)

For now, you can simply move the `Signing.crt` file from a non-expired firmware image to an expired one, and the robot will accept the firmware image. This is because the certificates all use the same private key, so the existing signatures remain valid.

The `4.5.3_189` firmware image contains a certificate which is valid until 2025-02-19. Up to February 19, 2025, you can place the `Signing.crt` file from that `.tgz` into the `.tgz` file of another firmware image, and the robot will accept it. You can use [7-Zip](https://www.7-zip.org/) to open the `.tgz` files and replace the `Signing.crt` file.

**After February 19, 2025, if Vorwerk does not obtain and provide a new certificate, this method will no longer work.**

### Working advanced methods (after February 19, 2025)

Once all certificates have expired, the above easy bypass method will not work. However, you can still install the firmware by tricking the robot into believing the date is before the certificate expiration date.

#### Factory reset and blocking date acquisition (easiest)

**This is currently the recommended method for use after February 19, 2025.**

You can perform a factory reset on the robot so it loses its Wi-Fi settings, then remove its battery so it resets its clock, and reboot it. [Here is a tutorial from u/cof53a on reddit](https://www.reddit.com/r/NeatoRobotics/comments/13oryys/refreshed_d3_thru_d7_453_firmware_for_manual_usb/kv6bujz/). This prevents the robot from obtaining the current date, so it will accept any signed firmware image.

#### Faking the date (harder)

The robot obtains its date from the pool.ntp.org NTP servers. You can set your router to intercept these NTP requests from the robot (such as by modifying DNS replies) to redirect them to your own NTP server. You must then configure your NTP server to return a date before the certificate expiration date.  This is not trivial.

### Potential advanced method: Self-signed certificate

A Neato `.tgz` firmware images contain three files. For example, in `Neato_4.5.3_189.tgz`, you will find:

1. `Neato_4.5.3_189.bin`: The firmware image itself.
2. `Signing.crt`: The certificate used to sign the firmware image.
3. `Neato_4.5.3_189.signed`: The signature of the firmware image.

The firmware itself is an encrypted binary file that runs on a custom Texas Instruments AM335x chip in the robot, according to extensive work done by Jiska Classen for [her PhD thesis](https://tuprints.ulb.tu-darmstadt.de/11422/). We have no way to modify the firmware itself. However, we can attempt to modify the certificate and signature files.

The `Signing.crt` is a certificate that Neato Robotics obtained from a certificate authority. The corresponding private key was used to sign the firmware image, generating the `.signed` file. The certificate and signature are in standard [OpenSSL](https://www.openssl.org/) format.

When firmware is installed, the robot verifies the signature using the public key from the certificate file before allowing the update to proceed.

Here is where we can potentially bypass the certificate expiration date. If the `.crt` file is used *only* for this step in the process, and is not also used to decrypt the firmware image itself, *and* if the robot does not verify the certificate chain of the `.crt` file, then we can simply generate our own certificate and private key, sign the firmware image with it, and the robot will accept the firmware image because it had a valid signature.

**As of February 9, 2025, we do not know if this is the case. If you try this method, please open a GitHub issue or discussion on this repository to report your findings.**

This section covers generating a self-signed certificate and using it to sign the firmware file `Neato_4.5.3_189.bin`.

There are two ways to create a self-signed firmware package:

1. Use the [automated Makefile process](#automated-process-using-make) (recommended).
2. Follow the [manual steps](#manual-process) below.

The manual steps are provided for educational purposes to give you a better understanding of the process used by the Makefile.

#### Automated Process Using Make

In the `make-self-signed-firmware` directory of this repository, you'll find a Makefile that automates the entire process of generating a self-signed firmware package for firmware version 4.5.3_189. This will work on Linux, Windows (with [MSYS2](https://www.msys2.org/) installed), and OS X.

##### Prerequisities

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

##### Creating the self-signed firmware package

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

#### Manual Process

If you prefer to do things manually, you can follow these steps:

##### Step 1: Retrieve the firmware file

Download the firmware file from the Internet Archive and extract the `.bin` file:

```sh
curl https://web.archive.org/web/20240506174708/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.5.3_189.tgz | tar xz Neato_4.5.3_189.bin
```

You can verify that the `.bin` file is correct by comparing its SHA256 hash to the known hash:

```sh
sha256sum Neato_4.5.3_189.bin
```

The hash should be `3d36076fbf3c196ef452b81d54857c75c17ac6eca24ef614aff27a8decc56ef8`.

At this point, you can either:

1. Follow the steps below to generate your own `.signed` and `.crt` files, or
2. Use the pre-generated `.signed` and `.crt` files provided  in the [self-signed-files](./self-signed-files/) subdirectory of this repository. You can download those and then skip directly to the [Verify the signature](#step-4-verify-the-signature) step below. If you do not have OpenSSL, may wish to simply trust that things are working and skip to [Repackage the firmware](#step-5-repackage-the-firmware).

##### Step 2: Generate a Self-Signed Certificate

###### Step 2a: Create an OpenSSL Configuration File

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

###### Step 2b: Generate a Private Key and CSR

Run the following command to create a new private key and a Certificate Signing Request (CSR):

```sh
openssl req -new -newkey rsa:2048 -nodes -keyout Signing.key -out Signing.csr -config neato.cnf
```

- `-newkey rsa:2048`: Creates a 2048-bit RSA key.
- `-nodes`: No password protection on the key.
- `-keyout Signing.key`: Saves the private key.
- `-out Signing.csr`: Saves the CSR.
- `-config neato.cnf`: Uses the configuration file to set the correct subject and extensions.

###### Step 2c: Generate a Self-Signed Certificate (Valid for 100 Years)

```sh
openssl x509 -req -days 36500 -in Signing.csr -signkey Signing.key -out Signing.crt -extfile neato.cnf -extensions v3_ext
```

- `-req`: Indicates a CSR-based certificate generation.
- `-days 36500`: Sets expiration to 100 years.
- `-signkey Signing.key`: Uses the private key to self-sign.
- `-out Signing.crt`: Saves the final certificate.
- `-extfile neato.cnf -extensions v3_ext`: Ensures the required X.509 extensions are included.

##### Step 3: Sign the Firmware File

Next, we will sign `Neato_4.5.3_189.bin` using the private key.

```sh
openssl dgst -sha256 -sign Signing.key -out Neato_4.5.3_189.signed Neato_4.5.3_189.bin
```

- `-sha256`: Uses SHA-256 hashing.
- `-sign Signing.key`: Signs using the private key.
- `-out Neato_4.5.3_189.signed`: Saves the signature.

##### Step 4: Verify the Signature

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

##### Step 5: Repackage the firmware

Finally, create a new `.tgz` file containing the original `.bin` file and the new `Signing.crt` and `Neato_4.5.3_189.signed` files:

```sh
tar czf Neato_4.5.3_189_modified.tgz Neato_4.5.3_189.bin Signing.crt Neato_4.5.3_189.signed
```

## Firmware version notes

### 4.5.3_189

This is the latest official firmware pushed to the robots over the air before Neato Robotics shut down, and enables all known features and provides bug fixes over earlier firmware versions. 

**This is likely the safest firmware version to install.**

As far as the current `.tgz` file as of this writing, it appears that a Vorwerk employee repackaged it on February 9, 2024 with a new, non-expired certificate. We can be thankful.

Although the firmware `.bin` file in this archive has a newer date than in earlier archives, the `.bin` itself is identical to earlier copies. (The SHA256 hash of the `Neato_4.5.3_189.bin` file, which is the actual firmware image inside the `.tgz`, is
`3d36076fbf3c196ef452b81d54857c75c17ac6eca24ef614aff27a8decc56ef8`.)

This `.tgz` file also contains a (normally hidden) `._Signing.crt` metadata file, likely because the archive was created manually on a Macintosh computer. This unnecessary, additional file does not interfere with the update procedure and does not need to be removed.

*Its certificate is valid through February 19, 2025.*

### 4.6.0_72

Some users have reported that the `4.6.0_72` firmware was installed by Neato on their robots after RMA service. It has never been pushed to robots over the air, but was available on the Neato server. The changes in this firmware version compared to `4.5.3_189` are not publicly documented.

*Its certificate is currently expired, but this can be bypassed using the methods described above.*

### 4.2.0_102

This is the [earliest documented firmware update for the Neato Botvac D7 Connected](https://shopeu.neatorobotics.com/pages/software-update-d7), and was shipped with at least some robots. If your robot shipped with this firmware, this is the version that your Botvac will revert to if you perform a factory reset. At this time, there is no known reason to install it manually.

*Its certificate is currently expired, but this can be bypassed using the methods described above.*

## Document revisions

This document was last updated on February 9, 2025. The download links worked as of that date. If you find that the links are broken, please submit a GitHub issue or pull request to update them. If you have any other information or corrections to add, please do the same.
