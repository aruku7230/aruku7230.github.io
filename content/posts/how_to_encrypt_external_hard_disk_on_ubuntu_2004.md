---
title: "How to Encrypt External Hard Disk on Ubuntu 20.04"
date: 2023-05-07
lastmod: 2024-05-23
draft: false
tags:
  - linux
  - encryption
---

## LUKS Versions

There are two versions of LUKS:
- Use LUKS version 2 whenever possible.
- Use LUKS version 1 if compatibility is a problem.

default LUKS format version
- LUKS1 with [cryptsetup](https://archlinux.org/packages/?name=cryptsetup) < 2.1.0
- LUKS2 with [cryptsetup](https://archlinux.org/packages/?name=cryptsetup) ≥ 2.1.0

## Before Encrypt Hard Disk

1. Check which devices are connected: `sudo fdisk -l`. Device is referred as something as `/dev/sdX`, where `X` may be `a`, `b` and so on. **It is best to disconnect other devices to prevent operation on wrong disk**.
2. Back up any data that you want to keep. No need to do if the disk is a new one without important data.
3. Optional Wipe all file systems and data from the hard drive
    - `sudo shred -v -n 3 -z /dev/sdX`.   This command will overwrite the entire hard disk with random data three times, and then a fourth time with zeros. Replace "sdX" with the device name of the hard disk you want to wipe. It can take very long time for a large disk (about 10-12 hour to overwrite a 4 TB disk one time). Consider changing `-n 3` to `-n 1` if it takes too long.
    - `sudo dd if=/dev/urandom of=/dev/sdX bs=1M status=progress`.  It might be (not always) faster than `shred`. It took me 12 hours to fill a 4 TB hard disk.
4. Optional instead of wiping all file system data, quickly wipe only file system header: `swip -a /dev/sdX`.

## Encrypt Entire External Hard Disk

1. Create LUKS container on device: Run `sudo cryptsetup --verbose --verify-passphrase luksFormat --type luks2 /dev/sdX`. Type `YES` to confirm overwrite data irrevocably. And Enter a password for the encryption. It creates a version 2 LUKS container.
2. Open the LUKS container: `sudo cryptsetup open /dev/sdX my_encrypted_disk`.
   - A device `/dev/mapper/my_encrypted_disk` is created to represent the unlocked LUKS container.
   - `open` command without `--type` option is default to `open --type luks`.
   - `luksOpen` is old syntax. For backward compatibility, `luksOpen` is an alias of `open --type luks`.
3. Create a new filesystem on unlocked device: `sudo mkfs.ext4 /dev/mapper/my_encrypted_disk`.
4.  If mount point does not exist, create it firstly: `sudo mkdir "/media/$USER/my_disk"`. Then mount mapped device: `sudo mount /dev/mapper/my_encrypted_disk "/media/$USER/my_disk"`.
5. Optional set label for container: `sudo cryptsetup config /dev/sdX --label Disk-XXX-Encrypted`. This label is displayed when disk is not encrypted.
6. Set volume label (at most 16 characters): `sudo e2lable /dev/mapper/my_encrypted_disk Disk-XXX`. This label is displayed after disk has been encrypted. If a volume label is not set, the device will be shown as "4.0 TB Volume" at sidebar of Files on Ubuntu.
7. Adjust permissions of mounted file system: `sudo chown -R $USER:$USER /media/$USER/my_disk`. It sets mounted file system owner and group to current user.
8. Make a LUKS header backup (see below) and plan for a container backup.

Check LUKS Container information:
- Check LUKS version and other information: `sudo cryptsetup luksDump /dev/sdX`.
- Test if device is a LUKS container: `sudo cryptsetup -v isLuks <device>`.
- Test if device is a LUKS version 2 container: `sudo cryptsetup -v isLuks --type luks2 <device>`.
- Show device type: `blkid -p <device>`.

Backup and restore LUKS header.
- If the header of a LUKS encrypted partition gets destroyed, you will not be able to decrypt your data. Backup : `sudo cryptsetup luksHeaderBackup --header-backup-file <file.dmp> /dev/sdX`
- Restore: `sudo cryptsetup luksHeaderRestore --header-backup-file <file.dmp> /dev/sdX`

Umount and lock LUKS container:
- Umount: `sudo umount /media/my_user_name/my_disk`
- Lock LUKS container: `sudo cryptsetup close my_encrypted_disk`.
  - `closeLuks` is old syntax. For backward compatibility, `luksClose` is an alias of `close` command.

Note: Tested in Ubuntu 20.04 with cryptsetup 2.2.2 and bash 5.0.

## FAQ

### Can LUKS encrypt entire hard disk without partitioning it?

From this [answer](https://unix.stackexchange.com/a/558489/246292), and Yes.
Basically, cryptsetup doesn't care what the LUKS device is, partition, disk, or loop device, so you can use whichever is appropriate.

```sh
sudo cryptsetup -v -y luksFormat /dev/sda
```

will create a LUKS container using all of `/dev/sda`.

### What is key, volume key, key-slot, passphrase?

Volume key is the key used to encrypt device. It is generated when LUKS container created.
Passphrases is used to derive the volume key. Changing or adding new password do not change volume key.
To change volume key, you have to reencrypt LUKS container, or backup the filesystem , re-create LUKS container and restore filesystem.

And from [LUKS2 specification](https://gitlab.com/cryptsetup/LUKS2-docs/blob/main/luks2_doc_wip.pdf)
- key-slot: Encrypted area on disk that contains a key.
- Volume key: The key used for data encryption on disk. Sometimes called as Media Encryption Key (MEK).

## References

- [cryptsetup Frequently Asked Questions](https://gitlab.com/cryptsetup/cryptsetup/-/wikis/FrequentlyAskedQuestions)
- [archlinux- dm-crypt/Device encryption](https://wiki.archlinux.org/title/Dm-crypt/Device_encryption)
- [Ubuntu - Full_Disk_Encryption_Howto](https://help.ubuntu.com/community/Full_Disk_Encryption_Howto_2019)
- [How to Encrypt an External Hard Drive on Linux](https://www.wikihow.com/Encrypt-an-External-Hard-Drive-on-Linux)
