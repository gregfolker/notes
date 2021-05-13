# The GNU Privacy Guard
## Table of Contents
1. [Overview](#overview)
2. [Generating a Key Pair](#generating-a-key-pair)
3. [Managing Keys](#managing-keys)
4. [Signing Files](#signing-files)
5. [Encrypting Files](#encrypting-files)
6. [Sources](#sources)

## Overview
The GNU Privacy Guard (or, `gpg`) is a popular free software implementation of the [Pretty Good Privacy](https://en.wikipedia.org/wiki/Pretty_Good_Privacy)
standard. It offers the functionality required to generate asymmetric keys, manage key pairs, sign and
encrpyt files

This document does not focus on the algorithmic approach to encryption but rather common usages of the
open-source `gpg` tool

## Generating a Key Pair
Add the following lines to the file `~/.gnupg/gpg.conf` on your system

```
personal-digest-preferences SHA256
cert-digest-algo SHA256
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
```

This will configure `gpg` to use the stronger hashing algorithms for generating signatures

From here, generating a key pair can be done using `gpg --gen-key`

```
$ gpg --gen-key
```

The default key type is RSA with a default bit length of 2048 (256 bytes). Prompts will follow this
command for parameters related to the creation of the new key pair

## Managing Keys
After creating a key pair, it can be viewed with the `--list-secret-keys` and `--list-public-keys`
commands

```
$ gpg --list-secret-keys                                                                                                  git:master*
/home/greg/.gnupg/pubring.kbx
-----------------------------
sec   rsa2048 2021-05-11 [SC] [expires: 2023-05-11]
      A6FB80180D3B4D598E12574ABD04A8B526FBB695
uid           [ultimate] Gregory Folker <gregfolker.dev@gmail.com>
ssb   rsa2048 2021-05-11 [E] [expires: 2023-05-11]
```

By default, these keys are located in the `~/.gnupg` directory. It is **highly** recommended to keep
this directory private and to have a secure backup of it, preferably on a removable media drive

Keys managed by `gpg` can be easily be referred to using the email of the owner. For example, if we
want our public key in ASCII format to share with someone we can use the following command:

```
$ gpg --armor --export gregfolker.dev@gmail.com                                                                           git:master*
-----BEGIN PGP PUBLIC KEY BLOCK-----

mQENBGCaxHsBCACb8FGmxzXpsoSfvQsQi3mLaLlYYuKh5+R1QUGk3jZQBKn/Rmpc
5NJ46t6LlXJfd+JrhkP9W1K3KAVTI5UXYACtUK63UQGPIHMcmPEF5u/Qk0bHdVeE
tCgnGhmBmEGanuVg20UjebrRFBNrpMKwdWGgEpuMVguVSG2t9FpAzSqu2xGweEfW
HwafvAKXTRF0rSFuwVfx97UgSpGsPU7BgAnJJHe9tKPMWjOzKWF+1R0XQX6pZE3A
8p5p/pJwaPfpzroa4CFC3GCJFhRGbMh3llmfL58P0bJCtWagGAv0MdysQ1gm/6Gb
db5OQTERFNnP1ow7MyqeE0KqIcgscdreLKhNABEBAAG0KUdyZWdvcnkgRm9sa2Vy
IDxncmVnZm9sa2VyLmRldkBnbWFpbC5jb20+iQFUBBMBCAA+FiEEpvuAGA07TVmO
EldKvQSotSb7tpUFAmCaxHsCGwMFCQPCZwAFCwkIBwMFFQoJCAsFFgIDAQACHgEC
F4AACgkQvQSotSb7tpVurAf/XbM9ZXXQucvJ5eROg2SHOILtm7CDCtKt5ZxS+UYb
fg+ZtoR+7fp0xpy+kMQM/DqNzQt9I66yR3MPNj4vxsQ2gIwCDZAcgG7lSweIN0Gn
f+8FcAkuOrAWAwRRDVZl1bIdkam1ICx3uiJGxOPiaunZ8BCwB8mYr9Myq8KgnlkX
LGOlRCFBvD7pLxWzz745QF/wu50IvYbMUe7Nr5nDmJHoLzk+Eqsxmnn15mkz2tz5
QAha5ntkEOlVszE9Hgiu1Buzhty6lzRKDzSqVLSejDONEWgy/ohrLNbmboYVHpHg
Dp6a3cb4yqYe0a8BYhM/1ZqAq6RCCoBVYRD+I24TjnNT0LkBDQRgmsR7AQgAwKWG
L8tFunvXawRf1KFRg0hNqVhPU9ZgzvmzLiodjvyNkEkeOKXP6I6JR7eOxEXeX0Sn
IlqBtNAlOeizEU576wmDjUcgRNgyXGhom9aay41z5n3CRDNXpsXtIYiOAheePiLA
r2GHdLOq2dKLyVlBUMYLAbAalhJYOHwrVQ9rkhqjErOn/equMvs+DgSJ7W6GzqCT
TzbAGqlj2m1tF6WGzXmR2WHGGfOADqcq4ReFZjnyQQZ39Wg7aN15ftaQy1KovmQ2
OK40bo8TwhPUy+HVfU6HvObtSIcDfztueR8Devz/uinOTGxfP/xygkI0W45tI2WO
qsd/F8cQLIGqxI+T5wARAQABiQE8BBgBCAAmFiEEpvuAGA07TVmOEldKvQSotSb7
tpUFAmCaxHsCGwwFCQPCZwAACgkQvQSotSb7tpWIDAf/dgm1AUJ2chCVCxD0M5Yv
O2c3DOOxp4lD6Ty6oz4RVsQw22TsxFQvbaadXf9IQzSGv6fqykuR/4HSwk2RFCq+
Eb+pJYkrNcjqx+Z2gV4EbiJCt8Kg17AwscfgKJz/WrbxbSTRejCr0f0/hY3hoJ29
hDsdWdc0AaCuSzcSD7AYCw9aX/7uV5JD8PlWFgJvHNPBUG7zYRapq0fcgZKqavPy
M8OVGkQ40DzRtj7sMdsIyQPCoXocsqhni64xcixsyuBfEmHJDidXll4SF24hwJAj
1Es2uby42uXgic2xVGRoM8l/STJ2oIvbJLPHI+FB5JN2DhWYfw91F86ZCDP9HfqC
1A==
=oJ/u
-----END PGP PUBLIC KEY BLOCK-----
```

Of course, this can be piped to a file instead of `stdout` by doing something like the following:

```
$ gpg --armor --export gregfolker.dev@gmail.com > greg-folker.pub.asc
```

## Signing Files
Signing a file is done using the *private* key. Readers of the file can then verify the signature
using the public key that they (should) have access to. Signing a file using `gpg` can be accomplished
with the `--clearsign` option:

```
$ gpg --clearsign message.txt
```

This will prompt you for the passphrase you created along with the private key in the earlier section.
A new file will then be created which contains the original contents of `message.txt` and the generated
signature at the end of it

```
$ cat message.txt.asc
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

This is a test file to sign using gpg
-----BEGIN PGP SIGNATURE-----

iQEzBAEBCAAdFiEEpvuAGA07TVmOEldKvQSotSb7tpUFAmCayZYACgkQvQSotSb7
tpX4SQf+OLHrbhG9NpI14mHcVmCAZbDtQi1fONtNPEmbXRkOHYVk7geTonyZrHj3
PQVZpxwU05p8GQY5np2L1+tBuLs9bA9iDK3qZHKP11lg2YDkH68QqB+bz6Ke3ViU
x4YNUx6M8TBIK1k3HoptcybmZ+/ZqSS6tF0SORi+VkzpL6I+Xczsp03lfgvXQdzM
bmWDEQ23H59JBcJL1ALKymziuFddEbH3KpGdUdni7fQHG/9IzQ+rwpGfRcW79DgA
KQaspUlJWvWNgj/hsM41t1Rso6i7NO12mZyso9MkESaZKrFQjSrfnwHK9ES7rwT4
AW0m2ef94hB8fVEs/zk47NJGpdEcvg==
=Nc+9
-----END PGP SIGNATURE-----

```

Remember, signing a file does not encrypt it. Digital signaures are simply used so that readers of the
file can verify the contents of the file have not been tampered with

Anyone with access to the public key can now verify the contents of this file with `gpg --verify`

```
$ gpg --verify message.txt.asc
gpg: Signature made Tue 11 May 2021 01:14:46 PM CDT
gpg:                using RSA key A6FB80180D3B4D598E12574ABD04A8B526FBB695
gpg: Good signature from "Gregory Folker <gregfolker.dev@gmail.com>" [ultimate]
gpg: WARNING: not a detached signature; file 'message.txt' was NOT verified!
```

The `WARNING` at the end is clarifying that the *original* file `message.txt` was not verified; in
this case, only the new file with the signature was verified along with it's contents. The default
behavior of `gpg` is to not detach signatures with the `--detach-sign` option. If you want the signature
stored in it's own file, use a command similar to the following:

```
$ gpg --armor --detach-sign message.txt
```

After successfully verifying your passphrase for access to the private key to sign with, the signature
will be written out to `message.txt.sig`. This signature file can be used again by anyone with your
public key to verify the contents of the *original* file

```
$ gpg --verify message.txt.sig message.txt
gpg: Signature made Thu 13 May 2021 10:10:34 AM CDT
gpg:                using RSA key A6FB80180D3B4D598E12574ABD04A8B526FBB695
gpg: Good signature from "Gregory Folker <gregfolker.dev@gmail.com>" [ultimate]iiiz
```

## Encrypting Files
Encrypting files is done using the *public* key where the decryption is done using the *private* key.
The public key used should be the recipient's public key since they will need to use their own private
key to decrypt and read the contents of the file we send them

If I was going to encrypt the contents of a file for myself using the same key-pair we have been
using throughout this document, it would look like the following:

```
$ gpg --armor --recipient gregfolker.dev@gmail.com --encrypt secret-message.txt
```

Which generates the file `secret-message.txt.asc` as the following:

```
$ cat secret-message.txt.asc
-----BEGIN PGP MESSAGE-----

hQEMA6+06krMK6MLAQgAi3MbkrCV9u1umay2XhLVhfJRnjeJp2z3fiEqrDr22u+I
nSdxS/0PfmDKGiUQzW2DidRSsOR43gX4PZtOkUmdRT6hIu0GBfaLVlUqy8Ia/5kx
oYMao7PeJ/BNSuDiPcylTifTsrMmaVgvfITXob07RngX70ApzLSJcwEhPd/GNjvD
J0KnDBZt+3HkplxioWqj6CaqsK5KKlN1dBv77uN23ifzi2qOAPrrOV/svWIh85HS
BXsekObrh08KcweOR4V/CE4J0cUcyDM9cydGrXsvMul1nprzNbv98Qin5ikhN5lP
VUwEc1EiRyWLl7ir4Qd2PqabyK0KtNPxbnxAp25QVNLA/AGcBwfR9dBhVzLigFdK
fLgrEDBRZ9TWnw5dlN6znvDdaKxRSbqCKd6FaB5c2Q+nLuFs0WsrpPL6oXVEJUJL
fbeWwyS6OWrHSmdgZbmPtiGTe3azWQzIrrzQy/EWQ5gCbjn+xI6GY3D3DIbwNw0y
eze0s4aPPtdgFLHsLZjv1LpjeW3Jw+bGzSAQPVPNKntMW+D5mQa2sfOZ04/W+EmN
9avEbUbM7zdbKMT6eFH7lAtUQ6NFqRch/GgmDS7Ulw9Rk0m4xU2TM2foZ80dC/4U
pUShAzXa7tcLX9/g6ZwIhaTxdoat2Z3QDmCpKVF4tDLBKOmi2NrxmjcyaI6ZaYFi
UxtqETjRqVvFPPZGYoWxQAEsWnOZsuj4zxnqfZXYIuCWQsVPefY8Sd8OMGTbnlKC
w9EflyebCTzHjmagnr9STKHGHikq1chDOi/tuDj+SvxseBnfHypt+y7IQ0WWJWA2
TGOkHJB7DuQYorSSPCp0OlsBAXTZGa3E9oE1ww4EkIXf4qWuOBseyce3RyRmdrjU
qagquEQjeDFOC0l2CSHx/VkEq3mOlUuA0HbCrv0d3yY3+eju6dsLeeZrSr99YQ==
=qegn
-----END PGP MESSAGE-----
```

From here, decrypting the file can be done with the `--decrypt` option, assuming you have access to
the necessary private key

```
$ gpg --decrypt secret-message.txt.asc
gpg: encrypted with 2048-bit RSA key, ID AFB4EA4ACC2BA30B, created 2021-05-11
      "Gregory Folker <gregfolker.dev@gmail.com>"
This is a secret test file to encrypt using gpg
gpg: Signature made Tue 11 May 2021 01:29:24 PM CDT
gpg:                using RSA key A6FB80180D3B4D598E12574ABD04A8B526FBB695
gpg: Good signature from "Gregory Folker <gregfolker.dev@gmail.com>" [ultimate]
```

In this example, the encrypted file was also signed and the signature is verifed as well by default
when using `gpg --decrypt`

## Sources
- [Arabesque - GNU/Linux Crypto](https://blog.sanctum.geek.nz/series/gnu-linux-crypto/)
