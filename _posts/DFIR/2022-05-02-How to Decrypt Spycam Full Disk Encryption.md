---
title : "Blog #34: How to Decrypt Spycam Full Disk Encryption"
category :
  - IoT & Embedded
tag : 
  - Android
  - Spycam
  - Full Disk Encryption
  - Mobile

sidebar_main : true
author_profile : true
use_math : true
toc: true
toc_sticky: true
toc_label: "Table of Contents"
header:
  overlay_image : /assets/images/post.jpg
  overlay_filter: 0.5
published : true
---
스파이캠 FDE 복호화

In the [previous post](https://kyl3song.github.io/iot%20forensics/Dumping-Data-from-Spycam-with-ADB/), we were dealing with a smartphone-looking spycam's extraction and checked how it makes the difference between Logical and Physical extraction. Also we've seen how to perform physical extraction using ADB to get the full dump as well as the userdata partition respectively.

One would ask, if it's similar to a smartphone, it would be weird to have around because spycam is supposed to be small, tiny thing that's easy to hide in some bag or a place to take a peek.
You are true, but this thing goes way beyond common sense. It has a camera on top(literally on top of its head), not in the front, so the device can record video even it's placed on the flat table. The front & back side cameras are fake that they don't operate like normal smartphone does, but it makes others think  you just have your own smartphone even though you take it out on the table recording videos.

Enough of the spec, let's get back to what we're going to do today where we left off. We will be decrypting the user partition from the full dump that's fully encrypted.


## What is FDE(Full-Disk Encryption) 

FDE was introduced in Android 4.4(Kitkat) to provide users with the option to encrypt the entire user data partition at the flash block level. As for Samsung Galaxy series, for example, devices launching with Android 7.0 or higher, the partition is encrypted by default. From Android 4.4 version up to 9 only support full-disk encryption. Devices that launched with Android 10 or higher must use file-based encryption instead.

FDE uses AES-CBC algorithm with 128-bit key to encrypt its userdata partition. The key is called DEK(Disk Encryption Key) is placed in **Crypto Footer** after it is encrypted by KEK(Key Encryption Key).


## Terms
These are the terminologies that will be mentioned much in this article.

- DEK (Disk Encryption Key): Also called Master Key
- KEK (Key Encryption Key): User PIN/Password based key to encrypt DEK
- KDF (Key Derivation Function): e.g. PBKDF2/Scrypt/etc
- AES: Advanced Encryption Standard
- CBC: Cipher Block Chaining
- ESSIV: One of IV generators. "encrypted sector\|salt initial vector", the sector number is encrypted with the bulk cipher using a salt as key. The salt is derived from the bulk cipher's key via hashing.



## How Android Full Disk Encryption/Decryption Works
Android full disk encryption is based on **dm-crypt**, a kernel feature that works at the block device layer to implement disk encryption for userdata which supports different ciphers. The encryption algorithm for Android FDE is **AES-128 with CBC and ESSIV:SHA256**. The master key is encrypted with 128-bit. (or higher 256-bit optionally)

Key points to know.
- Userdata partition is AES-CBC encrypted with one master key(DEK, Disk Encryption Key).
- Based on the user PIN/Password, the master key is encryped with KEK(Key Encryption Key)
- Crypto parameters for encryption/decryption are stored in 'Crypto Footer' located in /metadata(Google), /data(Samsung), /encrypt(LG).

The following describes how decryption works in Android 4.4 through 5.0. (use Scrypt for key derivation function).
> Android 3.0 - 4.3 uses PBKDF2 for key derivation function.
> Android 4.4 adds improvements one of which replaced the PBKDF2 with scrypt.
> Android 5.0 enhances security by adding process of RSA signing with Hardware Bound Key(HBK) to encrypt the DEK which makes harder to brute force.

- User enters lock screen **PIN/Password/Pattern**.
- Android gets **Encrypted DEK, Salt, DEK Size, scrypt paramers(N, r, p)** from the crypto footer.
- All bold paramers above runs through the Scrypt(KDF), decrypts the master key(DEK) which leads dm-crypt to decrypt the partition in order to mount afterwards.


### Crypto Footer
Crypto metadata for encryption/decryption is saved in a place called crypto footer where its structure is defined at [cryptfs.h](https://android.googlesource.com/platform/system/vold/+/android-cts-8.0_r16/cryptfs.h) (crypto footer version 1.3).


``` cpp
#define MAX_CRYPTO_TYPE_NAME_LEN 64
#define MAX_KEY_LEN 48
#define SALT_LEN 16

struct crypt_mnt_ftr {
  __le32 magic;         /* See above */
  __le16 major_version;
  __le16 minor_version;
  __le32 ftr_size;      /* in bytes, not including key following */
  __le32 flags;         /* See above */
  __le32 keysize;       /* in bytes */
  __le32 crypt_type;    /* how master_key is encrypted. Must be a
                         * CRYPT_TYPE_XXX value */
  __le64 fs_size;       /* Size of the encrypted fs, in 512 byte sectors */
  __le32 failed_decrypt_count; /* count of # of failed attempts to decrypt and
                                  mount, set to 0 on successful mount */
  unsigned char crypto_type_name[MAX_CRYPTO_TYPE_NAME_LEN]; /* The type of encryption
                                                               needed to decrypt this
                                                               partition, null terminated */
  __le32 spare2;        /* ignored */
  unsigned char master_key[MAX_KEY_LEN]; /* The encrypted key for decrypting the filesystem */
  unsigned char salt[SALT_LEN];   /* The salt used for this encryption */
  __le64 persist_data_offset[2];  /* Absolute offset to both copies of crypt_persist_data
                                   * on device with that info, either the footer of the
                                   * real_blkdevice or the metadata partition. */
  __le32 persist_data_size;       /* The number of bytes allocated to each copy of the
                                   * persistent data table*/
  __le8  kdf_type; /* The key derivation function used. */
  /* scrypt parameters. See www.tarsnap.com/scrypt/scrypt.pdf */
  __le8  N_factor; /* (1 << N) */
  __le8  r_factor; /* (1 << r) */
  __le8  p_factor; /* (1 << p) */
  __le64 encrypted_upto; /* If we are in state CRYPT_ENCRYPTION_IN_PROGRESS and
                            we have to stop (e.g. power low) this is the last
                            encrypted 512 byte sector.*/
  __le8  hash_first_block[SHA256_DIGEST_LENGTH]; /* When CRYPT_ENCRYPTION_IN_PROGRESS
                                                    set, hash of first block, used
                                                    to validate before continuing*/
```

Let's check out the full physical dump of spycam below. It's easy to find crypto footer sit in the metadata partition based on the algorithm highlighted in red.


<p align="center">
  <img src="https://i.imgur.com/oE7UCfi.png" alt="image"/>
</p>

Now the detaild information about the crypto footer explained is as follows:

<p align="center">
  <img src="https://i.imgur.com/MjOiQof.png" alt="image"/>
</p>

- Magic: 0xC4B1B5D0
- Major Version: 1
- Minor Version: 3
- Footer Size: 2,320 bytes
- Flags: 0x00000000
- DEK Size: 16 bytes(0x10)
- **Crypto Type: 1 (DEFAULT : Master_key is encrypted with 'default_password')**
- Encrypted Size(in sector): 54,623,232 Sector (27,967,094,784 bytes)
- Crypto Type Name: aes-cbc-essiv:sha256
- **DEK: 0x6792d070725b1fa5dbda570493892c18**
- **Salt: 0x62eaad405cc7a1461d70de898d885617**
- KDF Type: 2 (KDF_SCRYPT)
- **N factor: 15**
- **r factor: 3**
- **p factor: 1**


## How to Decrypt
Based on the crypto footer, the spycam uses **scrypt** as key derivation function to encrypt DEK which tells us that we have a chance to decrypt the userdata partition by the offline brute force attack.
Forturnately, the spycam is not screenlocked with any password which gives us a clue that it is encrypted with 'default_password' string.

 - 0 : PASSWORD - Master_key is encrypted with Password
 - **1 : DEFAULT - Master_key is encrypted with 'default_password'**
 - 2 : PATTERN - Master_key is encrypted with PATTERN
 - 3 : PIN - Master_key is encrypted with PIN


### Block Diagram of Decryption
The diagram below indicates how we can decrypt FDE using crypto parameters. Some parameters can be omitted. One thing to notice is that ESSIV is one of IV generators. Its length must be multiple of AES message block size

<p align="center">
  <img src="https://i.imgur.com/NnrMo7S.png" alt="image"/>
</p>


Following python code to decrypt the FDE is referenced as well as customized from a bunch of codes, I have coverted to Python3 and add some logic for my purpose. Credits to [Thomascannon](https://github.com/thomascannon/android-fde-decryption), [Santoku](https://github.com/santoku/Santoku-Linux/blob/master/tools/android/android_bruteforce_stdcrypto/bruteforce_stdcrypto.py), [Gonigoni](https://blog.naver.com/PostView.naver?blogId=sjhmc9695&logNo=222443055766&parentCategoryNo=&categoryNo=52&viewDate=&isShowPopularPosts=true&from=search). Script can be viewed at my [Github](https://github.com/kyl3song/Android_FDE_BruteforceDecrypt).


## Result
By running the code, we successfully managed to decrypt the userdata patition. yay.

<p align="center">
  <img src="https://i.imgur.com/h64vSf0.png" alt="image"/>
</p>

<p align="center">
  <img src="https://i.imgur.com/4qwZox4.png" alt="image"/>
</p>


## Reference
- <https://source.android.com/security/encryption/full-disk>
- <https://resources.infosecinstitute.com/topic/understanding-disk-encryption-android-ios/>
- <https://gitlab.com/cryptsetup/cryptsetup/-/wikis/DMCrypt#iv-generators>
- <https://docs.samsungknox.com/admin/knox-platform-for-enterprise/kbas/kba-360039577713.htm>
- <https://velog.io/@palza4dev/%ED%8C%A8%EC%8A%A4%EC%9B%8C%EB%93%9C-%EC%95%94%ED%98%B8%ED%99%94-PBKDF2-scrypt-bcrypt-argon2>
- <https://scienceon.kisti.re.kr/commons/util/originalView.do?cn=JAKO201904533923641&oCn=JAKO201904533923641&dbt=JAKO&journal=NJOU00291864>
- <https://source.android.com/security/encryption/full-disk>
- <https://android.googlesource.com/platform/system/vold/+/refs/heads/kitkat-dev/cryptfs.h>
- <https://nelenkov.blogspot.com/2014/10/revisiting-android-disk-encryption.html>
- <https://android.googlesource.com/platform/system/vold/+/android-cts-8.0_r16/cryptfs.h>


## Copyright (CC BY-NC 2.0)
<img src="/assets/images/creativecommon_by-nc.png" width="30%" height="30%">

- <http://ccl.cckorea.org/about/>
- <https://creativecommons.org/>
- <https://creativecommons.org/faq/#what-are-creative-commons-licenses>
