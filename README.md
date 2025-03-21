# Neato Botvac D3, D3 Pro, D4, D5, and D7 firmware

**Quick tip:** If you came here just to get firmware with a non-expired certificate, and otherwise know what you're doing, jump to the [Firmware download links](#firmware-download-links) section. There are working firmware images there.

## Table of contents

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

- [Table of contents](#table-of-contents)
- [Introduction](#introduction)
- [Firmware download links](#firmware-download-links)
  - [Firmware downloads with non-expired certificates](#firmware-downloads-with-non-expired-certificates)
  - [Original firmware downloads (All certificates expired)](#original-firmware-downloads-all-certificates-expired)
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
  - [Replacing the certificate](#replacing-the-certificate)
    - [Replacing the certificate with a precertificate](#replacing-the-certificate-with-a-precertificate)
    - [Signing the firmware with a self-signed certificate](#signing-the-firmware-with-a-self-signed-certificate)
  - [Faking the date](#faking-the-date)
    - [Blocking date acquisition (easiest)](#blocking-date-acquisition-easiest)
    - [Faking the date via NTP (harder)](#faking-the-date-via-ntp-harder)
- [Firmware version notes](#firmware-version-notes)
  - [4.5.3_189](#453_189)
  - [4.6.0_72](#460_72)
  - [4.2.0_102](#420_102)
- [Feedback](#feedback)

<!-- /code_chunk_output -->

## Introduction

This document contains links to firmware images from the Internet Archive mirror of the official Neato Robotics server for the Neato Botvac D3, D3 Pro, D4, D5, and D7 robot vacuums as well as information on how to install them, including how to bypass expired certificates.

## Firmware download links

### Firmware downloads with non-expired certificates

These are the official firmware packages but with the expired certificate replaced by non-expired certificates. The firmware images themselves are unchanged.

**If you try any of these firmware images, whether or not your robot accepts the firmware through [the update process below](#installing-the-firmware), please create a [GitHub discussion](https://github.com/RobertSundling/neato-botvac/discussions) on this repository to report your findings. I am still collecting information to refine and streamline the installation process.**

| Version   | Firmware Date | Certificate Validity         | Download   |
|-----------|---------------|------------------------------|------------|
| **4.5.3_189** | **2019-10-29**    | **2025-02-20 to 2125-01-27** | **[Neato_4.5.3_189.tgz (self-signed certificate)](https://github.com/RobertSundling/neato-botvac/releases/download/v0.0.1-self-signed/Neato_4.5.3_189.tgz)** |
| **4.5.3_189** | **2019-10-29**    | **2025-02-18 to 2026-03-19** | **[Neato_4.5.3_189.tgz (precertificate)](https://github.com/RobertSundling/neato-botvac/releases/download/v0.0.1-precert/Neato_4.5.3_189.tgz)** |
| 4.6.0_72  | 2020-01-27    | 2025-02-18 to 2026-03-19 | [Neato_4.6.0_72.tgz (precertificate)](https://github.com/RobertSundling/neato-botvac/releases/download/v0.0.1-precert/Neato_4.6.0_72.tgz) |
| 4.2.0_102 | 2018-07-12    | 2025-02-18 to 2026-03-19 | [Neato_4.2.0_102.tgz (precertificate)](https://github.com/RobertSundling/neato-botvac/releases/download/v0.0.1-precert/Neato_4.2.0_102.tgz) |

Most users should choose `4.5.3_189`. See the [Firmware Version Notes](#firmware-version-notes) section below for more information on these firmware versions.

**It is up to you whether to use the precertificate or self-signed certificate version.** There is no actual difference in the firmware itself. Both were originally provided in case one or the other didn't work, but the precertificate firmware [was confirmed to work](https://github.com/RobertSundling/neato-botvac/discussions/6), and [so was the self-signed firmware](https://github.com/RobertSundling/neato-botvac/discussions/9). Eventually the precertificate firmware will no longer be able to installed, so I'd personally just go with the self-signed firmware.

### Original firmware downloads (All certificates expired)

These are the original firmware images and signatures, now all with expired certificates. Additionally, as of February 8, 2025, the firmware images are no longer available directly from the official Neato Robotics server, which now returns an "Access Denied" error. Copies remain accessible through the Internet Archive.

| Version   | Firmware Date | Certificate Validity         | Download   |
|-----------|---------------|------------------------------|------------|
| 4.5.3_189 | 2019-10-29    | 2024-01-19 to 2025-02-19 | [Neato_4.5.3_189.tgz (via Internet Archive)](https://web.archive.org/web/20240506174708/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.5.3_189.tgz) |
| 4.6.0_72  | 2020-01-27    | 2019-03-20 to 2021-03-19     | [Neato_4.6.0_72.tgz (via Internet Archive)](https://web.archive.org/web/20240506174741/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.6.0_72.tgz) |
| 4.2.0_102 | 2018-07-12    | 2018-01-17 to 2019-05-11     | [Neato_4.2.0_102.tgz (via Internet Archive)](https://web.archive.org/web/20240506174833/https://neatorobotics-ota.s3.amazonaws.com/production/Neato_4.2.0_102.tgz) |

## Motivation

Neato Robotics, a subsidiary of Vorwerk & Co. KG, ceased operations in 2023. This document is an effort to keep their robots running.

When troubleshooting or repairing a Neato Botvac D3, D3 Pro, D4, D5, or D7, it may be necessary to install new firmware on the robot. The firmware is generally no longer obtainable through the Neato app, even though the app may suggest that an update is available. Resetting your robot may revert it to outdated firmware.

Unfortunately, the official firmware images can no longer be installed directly due to expired certificates. This document provides information on how to bypass this limitation. There are many links to repackaged versions of the firmware images with different expiration dates available online, but generally those are expired now as well.

I created this document to share what I learned while exploring the options to update my Botvac D7 Connected to the `4.5.3_189` firmware after a factory reset reverted it to `4.2.0_102`. I found that the app could not update the firmware and all of the firmware images shared on forums had expired certificates. I hope this document will help others in the same situation.

## Update procedure

### WARNING

**Please note that updating firmware involves inherent risks, including the possibility of making your robot inoperable or "bricking" it. By following these procedures, you acknowledge and accept these risks. If you are uncertain about these steps or their consequences, it is advised not to proceed. The information provided here is for educational purposes only, and the author assumes no liability for any damage or loss resulting from its use.**

### Before starting

You may wish to read the [Updating faster](#updating-faster) section below to speed up the install process.

If you need to install old firmware with an expired certificate, see the [Bypassing certificate expiration](#bypassing-certificate-expiration) section below first.

### Installing the firmware

To install firmware on your Neato Botvac, you generally do *not* need to perform a [factory reset](#factory-reset) or press any complicated button sequences on your robot. You really only have to turn off the robot, plug in a flash drive, turn the robot back on, and the robot handles the rest.

First, prepare a USB flash drive:

1. Obtain a USB flash drive and ensure it is formatted as FAT-32 (not exFAT).

   > **Note:** You may need to try a couple drives. Some users have said the drive needs to be at least 16 GB (even though the firmware images themselves are under 16 MB). Others have reported problems with USB 3.x drives, with the upgrade freezing during the "copying logs" stage, and suggest USB 2.0 drives. I've had perfect success with USB 3.x drives, myself, but if you have issues, you may wish to try another drive.
  
1. Create a folder on the flash drive named `RobotData`. Capitalization is important.
1. Copy the firmware `.tgz` file directly into the `RobotData` folder on the flash drive. Do not extract the contents of the archive; the robot will do this itself.

At this point, I suggest having the robot fully charged and on its charging base. Then:

1. Power down the robot by pressing and holding the power button until the robot turns off.

    > **Note:** This guide didn't used to include this power-down step. However, [multiple](https://github.com/RobertSundling/neato-botvac/discussions/6) [users](https://github.com/RobertSundling/neato-botvac/discussions/9) have had to power-cycle the robot to get the new firmware to take effect. So I have added this step to the guide.

1. Remove the robot's dustbin to expose its micro USB port.
1. Plug the flash drive into your robot using a [USB OTG cable](#usb-otg-cable).
1. Power the robot back up by pressing and holding the power button until the robot turns on.
1. The robot will automatically detect the firmware file and begin the update process, with the lights on the left flashing rhythmically.
1. The robot will play a sound and reboot once the update is complete.

In my experience, if I have cleared the log files first (as explained in the [Updating faster](#updating-faster) section), the update process only takes a minute or two. If the firmware update did not take, you may wish to try keeping the USB flash drive connected and power-cycling again.

**You would be well-advised to *not* interrupt a firmware update while in progress.** You'll notice the lights flashing a lot more than normal. Wait until that stops before doing anything.

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

Although the neato.cloud certificate [was renewed according to crt.sh](https://crt.sh/?q=neato.cloud), we do not have a copy of the full certificate, only the precertificate. However, it has been confirmed that the robot accepts a precertificate just like a regular certificate.

## Bypassing certificate expiration

### Replacing the certificate

In the past, you could have moved the `Signing.crt` file from a non-expired firmware image to an expired one, and the robot would accept the firmware image. This is because the certificates all use the same private key, so the existing signatures remain valid.

The `4.5.3_189` firmware image contains a certificate which was valid until 2025-02-19. Up to February 19, 2025, you could have placed the `Signing.crt` file from that `.tgz` into the `.tgz` file of another firmware image, using a program like [7-Zip](https://www.7-zip.org/) to open the `.tgz` files and replace the `Signing.crt` file, and the robot would have accepted it.

Now we need to replace the `Signing.crt` file with something else. We have two options.

#### Replacing the certificate with a precertificate

**This is currently the recommended method to use as of Feburary 22, 2025, as it is known to work.**

It is possible to replace the expired certificate with a non-expired neato.cloud precertificate. For example, there is a neato.cloud precertificate that expires in March 2026.

Full details on this method are provided in the [Precertificate Firmware](./precertificate-firmware/README.md) directory of this repository.  You can also download a ready-to-use firmware image created in this way directly from the [link above](#firmware-download-links).

#### Signing the firmware with a self-signed certificate

It is also possible to sign the firmware with a self-signed certificate that you generate yourself, with an expiration date hundreds of years in the future. This works because the robot does not verify the certificate chain, and does not use the certificate for anything other than the initial signature verification.

Full detaileds on this method are provided in the [Self-Signed Firmware](./self-signed-firmware/README.md) directory of this repository. You can also download a ready-to-use firmware image created in this way directly from the [link above](#firmware-download-links).

### Faking the date

If you *must* install firmware with an expired certificate, you need to trick the robot into believing the date is before the certificate expiration date.

#### Blocking date acquisition (easiest)

The idea is to prevent the robot from obtaining the current date, so it will accept any signed firmware image regardless of the expiration date.

You may be able to first remove the battery from your robot (so that it loses the current time). Once your robot is without power, you need to prevent the robot from connecting to your Wi-Fi network when it turns back on. Some methods to do this include:

* Temporarily turning off your Wi-Fi router, or 
* Changing your Wi-Fi SSID or password, or
* Blocking its MAC address in your router settings, or 
* Bringing it to someone else's house (for example), or
* Deleting your Wi-Fi network from the robot

*Some of those are easier than others, and many may cause other issues, so be careful. If you are unsure about these, taking it to someone else's house is the easiest option. Be sure to bring your charging base.*

Once you have done one of those, then reinstall the battery. The robot should turn back on but be unable to obtain the current date and time.

**Important note:** I am continuously working to make this guide as easy-to-use as possible. If you try any of these methods, whether or not they work for you, please create a [GitHub discussion](https://github.com/RobertSundling/neato-botvac/discussions) on this repository to report your findings.

There is also a guaranteed known-working way to do this. You can instead do a full factory reset on your robot, disconnect the battery, then reinstall it. [Here is a tutorial from u/cof53a on reddit](https://www.reddit.com/r/NeatoRobotics/comments/13oryys/refreshed_d3_thru_d7_453_firmware_for_manual_usb/kv6bujz/). However, a factory reset should generally be avoided if possible, so only try this if nothing else works.

#### Faking the date via NTP (harder)

The robot obtains its date from the pool.ntp.org NTP servers. You can set your router to redirect the robot to your own NTP server, using something like Pi-hole or a custom DNS server. You could then configure your own NTP server to return a date before the certificate expiration date.

Explaining how to do this is beyond the scope of this document and is left as an exercise for advanced readers.

## Firmware version notes

### 4.5.3_189

This is the latest official firmware pushed to the robots over the air before Neato Robotics shut down, and enables all known features and provides bug fixes over earlier firmware versions. 

**This is likely the safest firmware version to install.**

As far as the current `.tgz` file as of this writing, it appears that a Vorwerk employee repackaged it on February 9, 2024 with a certificate valid through February 19, 2025.

Although the firmware `.bin` file in this archive has a newer date than in earlier archives, the `.bin` itself is identical to earlier copies.

The SHA256 hash of the `Neato_4.5.3_189.bin` file, which is the actual firmware image inside the `.tgz`, is
`3d36076fbf3c196ef452b81d54857c75c17ac6eca24ef614aff27a8decc56ef8`.

This `.tgz` file also contains a (normally hidden) `._Signing.crt` metadata file, likely because the archive was created manually on a Macintosh computer. This unnecessary, additional file does not interfere with the update procedure and does not need to be removed.

*The certificate contained in this firmware package is currently expired, but this can be bypassed using the methods described above.*

### 4.6.0_72

Some users have reported that the `4.6.0_72` firmware was installed by Neato on their robots after RMA service. It has never been pushed to robots over the air, but was available on the Neato server. The changes in this firmware version compared to `4.5.3_189` are not publicly documented.

The SHA256 hash of the `Neato_4.6.0_72.bin` file, which is the actual firmware image inside the `.tgz`, is
`38973d99f40df5ae7a51eed8db361bfa80c1fe21a274b66df9d6b461d97d8a72`.

*The certificate contained in this firmware package is currently expired, but this can be bypassed using the methods described above.*

### 4.2.0_102

This is the [earliest documented firmware update for the Neato Botvac D7 Connected](https://shopeu.neatorobotics.com/pages/software-update-d7), and was shipped with at least some robots. If your robot shipped with this firmware, this is the version that your Botvac will revert to if you perform a factory reset. At this time, there is no known reason to install it manually.

The SHA256 hash of the `Neato_4.2.0_102.bin` file, which is the actual firmware image inside the `.tgz`, is
`4c67919bf53771f730bc1c3756532079e3ed7e51bb349a03559417d4645a9fb7`.

*The certificate contained in this firmware package is currently expired, but this can be bypassed using the methods described above.*

## Feedback

If you try one of these firmware images, whether or not your robot accepts the firmware, or if you notice any problems with or have any suggestions for this document, please create a [GitHub discussion](https://github.com/RobertSundling/neato-botvac/discussions) on this repository to report your findings.
