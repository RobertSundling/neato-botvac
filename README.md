# Neato Botvac D3, D3 Pro, D4, D5, and D7 Firmware

**Quick tip:** If you came here just to get firmware with a non-expired certificate, and otherwise know what you're doing, download the first link in the [Firmware download links](#firmware-download-links) section. It's good through February 19, 2025.

## Table of Contents

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

- [Table of Contents](#table-of-contents)
- [Introduction](#introduction)
- [Firmware download links](#firmware-download-links)
- [Motivation](#motivation)
- [Update procedure](#update-procedure)
  - [WARNING](#warning)
  - [Before starting](#before-starting)
  - [Installing the firmware](#installing-the-firmware)
  - [After the update](#after-the-update)
- [Updating faster](#updating-faster)
- [Firmware certificates and signatures](#firmware-certificates-and-signatures)
  - [Bypassing certificate expiration](#bypassing-certificate-expiration)
    - [Easy method](#easy-method)
    - [Advanced methods](#advanced-methods)
    - [Future potential methods](#future-potential-methods)
      - [Firmware signature format](#firmware-signature-format)
      - [Verifying firmware signatures](#verifying-firmware-signatures)
      - [Generating your own signatures](#generating-your-own-signatures)
- [USB OTG Cable](#usb-otg-cable)
- [Factory reset](#factory-reset)
- [Firmware Version Notes](#firmware-version-notes)
  - [4.5.3_189](#453_189)
  - [4.6.0_72](#460_72)
  - [4.2.0_102](#420_102)
- [Document revisions](#document-revisions)

<!-- /code_chunk_output -->

## Introduction

This document contains links to firmware images on the official Neato Robotics server for the Neato Botvac D3, D3 Pro, D4, D5, and D7 robot vacuums as well as information on how to install them, including how to bypass expired certificates.

## Firmware download links

| Version   | Firmware Date | Certificate Validity         | Download   |
|-----------|---------------|------------------------------|------------|
| 4.5.3_189 | 2019-10-29    | **2024-01-19 to 2025-02-19** | [From Neato Robotics](https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.5.3_189.tgz) ([Internet Archive Mirror](https://web.archive.org/web/20240506174708/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.5.3_189.tgz)) |
| 4.6.0_72  | 2020-01-27    | 2019-03-20 to 2021-03-19     | [From Neato Robotics](https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.6.0_72.tgz) ([Internet Archive Mirror](https://web.archive.org/web/20240506174741/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.6.0_72.tgz)) |
| 4.2.0_102 | 2018-07-12    | 2018-01-17 to 2019-05-11     | [From Neato Robotics](https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.2.0_102.tgz) ([Internet Archive Mirror](https://web.archive.org/web/20240506174833/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.2.0_102.tgz)) |

Most users should choose `4.5.3_189`. See the [Firmware Version Notes](#firmware-version-notes) section below for more information on these firmware versions.

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

1. Obtain a USB flash drive and ensure it is formatted as FAT-32. Some have said it needs to be at least 16 GB (the firmware images themselves are under 16 MB).
2. Create a folder on the flash drive named `RobotData`. Capitalization is important.
3. Copy the firmware .tgz file directly into the `RobotData` folder on the flash drive. Do not extract the contents of the archive; the robot will do this itself.

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

## Firmware certificates and signatures

The firmware images are signed by Neato Robotics, and the certificates are valid for a certain period of time. **If the certificate has expired, the robot will *not* accept the firmware image.**

This has, historically, been the largest stumbling block to updating firmware outside of the Neato app.

### Bypassing certificate expiration

#### Easy method

For now, you can simply move the `Signing.crt` file from a non-expired firmware image to an expired one, and the robot will accept the firmware image. This is because the certificates all use the same private key, so the existing signatures remain valid.

The `4.5.3_189` firmware image contains a certificate which is valid until 2025-02-19. Up to February 19, 2025, you can place the `Signing.crt` file from that `.tgz` into the `.tgz` file of another firmware image, and the robot will accept it. You can use [7-Zip](https://www.7-zip.org/) to open the `.tgz` files and replace the `Signing.crt` file.

After February 19, 2025, if Vorwerk does not obtain and provide a new certificate, this method will no longer work.

#### Advanced methods

Once all certificates have expired, the above method will not work. However, you can still install the firmware by tricking the robot into believing the date is before the certificate expiration date.

There are currently at least two methods for doing this:

1. You can factory-reset the robot so it loses its Wi-Fi settings, then remove its battery so it resets its clock, and reboot it. [Here is a tutorial from /u/cof53a on reddit](https://www.reddit.com/r/NeatoRobotics/comments/13oryys/refreshed_d3_thru_d7_453_firmware_for_manual_usb/kv6bujz/). This prevents it from obtaining the current date, and it will accept any signed firmware image.
2. The robot obtains its date from the pool.ntp.org NTP servers. You can set your router to intercept these NTP requests from the robot (such as by modifying DNS replies) to redirect them to your own NTP server. You must then configure your NTP server to return a date before the certificate expiration date.  This is not trivial.

#### Future potential methods

I have not tested whether the Neato Botvac checks the validity of the signing certificate by any method other than ensuring the date is valid. For example, if the robot doesn't have hard-coded root certificates, it may not check the certificate chain. It *may*, for example, only care that the certificate is issued to `www.neato.cloud` and that it's not expired.

If that is the case, it is possible the robots may accept *any* valid certificate, such as a self-signed certificate, which could permit the use of custom firmware in the future. This is worth exploring.

##### Firmware signature format

The signatures for the `.bin` firmware images are stored in separate `.signed` files in the `.tgz` archives. These signatures are in the standard [OpenSSL](https://www.openssl.org/) signature format.

##### Verifying firmware signatures

The signatures are easy to verify. First, extract the files from the `.tgz` file. Next, convert the `Signing.crt` certificate into a public key file using OpenSSL:

```
openssl x509 -pubkey -in Signing.crt -noout -out pubkey.pem
```

Next, verify the firmware image like this:

```
openssl dgst -sha256 -verify pubkey.pem -signature Neato_4.6.0_72.signed Neato_4.6.0_72.bin
```

Of course, substitute the firmware image names as appropriate. If the signature is valid, you will see:

```
Verified OK
```

##### Generating your own signatures

If the robot does not verify the certificate chain, you could use the parallel commands to the above sign any firmware file using a self-generated private key (for example, one that doesn't expire for a hundred years). If you try this, please let me know the results.

## USB OTG Cable

You will need a USB OTG ("On-The-Go") cable to connect a USB flash drive to your Neato Botvac. Your Botvac may have come with one, but there is no need to use the official cable (which sells for around $30 US).

If you do not already have one, what you are looking for is an [OTG Micro USB 2.0 Male to USB Female cable](https://www.google.com/search?q=OTG+Micro+USB+2.0+Male+to+USB+Female+cable). You can find them on Amazon or eBay for a few dollars. (They are mostly used to connect USB devices to older smartphones and tablets.)

## Factory reset

The Botvac is supposed to keep a secondary, backup firmware image from when it was originally shipped. If something goes wrong and your robot is not in a usable state, you may wish to attempt to have the bot revert to this backup image by performing a factory reset (also sometimes called a "hard reset"). This involves a process of holding down the front bumper in a certain way while pressing and releasing the power button, at certain intervals.

The specific steps to do this are [outlined in this reddit post by /u/woutske](https://www.reddit.com/r/botvac/comments/c7hznj/comment/esjzc1i/). Note that not all of the steps may be necessary.

Of course, it is not guaranteed that this process will work or that you will always be able to do this.

## Firmware Version Notes

### 4.5.3_189

This is the latest official firmware pushed to the robots over the air before Neato Robotics shut down, and enables all known features and provides bug fixes over earlier firmware versions. 

**This is likely the safest firmware version to install.**

As far as the current `.tgz` file as of this writing, it appears that a Vorwerk employee repackaged it on February 9, 2024 with a new, non-expired certificate. We can be thankful.

Although the firmware `.bin` file in this archive has a newer date than in earlier archives, the `.bin` itself is identical to earlier copies. (The SHA256 hash of the `Neato_4.5.3_189.bin` file, which is the actual firmware image inside the .tgz, is
`3d36076fbf3c196ef452b81d54857c75c17ac6eca24ef614aff27a8decc56ef8`.)

This `.tgz` file also contains a (normally hidden) `._Signing.crt` metadata file, likely because the archive was created manually on a Macintosh computer. The good news is that this additional file does not interfere with the update procedure and does not need to be removed.

### 4.6.0_72

Some users have reported that the `4.6.0_72` firmware was installed by Neato on their robots after RMA service. It has never been pushed to robots over the air, but it is available on the Neato server. The changes in this firmware version compared to `4.5.3_189` are not publicly documented.

Its certificate is currently expired, but this can be bypassed using the methods described above.

### 4.2.0_102

This is the [earliest documented firmware update for the Neato Botvac D7 Connected](https://shopeu.neatorobotics.com/pages/software-update-d7), and was shipped with at least some robots. If your robot shipped with this firmware, this is the version that your Botvac will revert to if you perform a factory reset. At this time, there is no known reason to install it manually.

Its certificate is currently expired, but this can be bypassed using the methods described above.

## Document revisions

This document was last updated on May 6, 2024. The download links worked as of that date. If you find that the links are broken, please open a GitHub issue or a pull request to update them. If you have any other information or corrections to add, please do the same.
