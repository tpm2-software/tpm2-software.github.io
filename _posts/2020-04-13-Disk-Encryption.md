# Introduction
In an increasingly data heavy world we live in today, data security has become
critical for connected-devices, corporations, and individuals alike. It is
important to keep data protected at rest especially at source and also in
transit. Hence digital encryption schemes have become paramount for protecting
the confidentiality and integrity of the data. Of the multitude of encryption
mechanisms for protecting the confidentiality AES XTS (cipher text stealing)
mode is most commonly used. Encryption schemes function by virtue of protecting
the encryption secret which is also termed encryption key.

# Trusted Platform Module (TPM)
The robustness rules for protecting the  encryption secret or keys involves both
physical access protection to the key data as well as access restrictions
to sensitive operations with the key like decryption and digital signing
enforced through authentication mechanisms. These requirements were termed as
root of trust for storage (RTS) and root of trust for reporting (RTR) by the
Trusted Computing Group (TCG). TCG is a consortium of industry and academic
expertise coming together to define specifications for a security module that
achieves these objectives. This security module is called the Trusted Platform
Module or TPM. The TPM came about in two major standard specifications, the 1.2
and the now 2.0. Algorithm flexibility and enhanced authorization schemes are of
the major advancements in 2.0. Secure persistent storage, platform configuration
register or PCR for recording system software state(more on this later), a per
TPM unique certified key for digital identity, and hierarchies for partitioning
roles based access to TPM objects are of the notable TPM properties. TPM is a
limited resourced device.

# Trusted Software Stack (TSS) and TPM2 Software (tpm2-software):
TPMs have a secure persistent storage of about 14KB and a smaller fast memory or
RAM to support a maximum 3 sessions that quickly exhaust depending on the
application requirement. In addition to the TPM specifications, TCG also created
a specification for a software stack to optimally function with the limited
resourced TPM device. This is called the Trusted Software Stack or TSS. The spec
defines not only the mechanisms and interfaces to manage the TPM resources, it
also details how the software partitions the stack into well defined interfaces
for device communication, command response buffer transport, access brokering,
resource management, session management and feature profiles for most common use
cases.<br>

Feature API or FAPI:The abstraction should profile common use cases and
provides context management.<br>

Enhanced System API or ESAPI or ESYS: Provides cryptographic functions for
sessions and session management.<br>

System API or SAPI: A one to one mapping of all the available TPM2 commands.
While sufficient it requires expert TPM knowledge for managing sessions,
contexts and cryptographic functions. Marshalling and unmarshalling of command
and response buffer is also handled here.<br>

Trusted Access Broker and Resource Manager or TAB/RM: Abstracts resource
limitation. In a multi application setup vying from TPM access, it ensures TPM
commands are singularly processed. There is a user space dbus based
implementation as well an in kernel implementation.<br>

TPM Command Transmission Interface or TCTI: Decouples the API generating the
TPM command and responses and instead focuses on the transport mechanism.<br>

Trusted Device Driver or TDD: This is the primary device interface handling
the command-response buffering, control registers and interface management. This
is part of the mainline kernel tree in the character device driver category.<br>

## TPM2 Software:
The community supported tpm2-software project on github is an implementation of
the trusted software stack specification by TCG. It is getting widespread
adoption in the linux community and mainstream distributions. Following is a
growing list of the tpm2-software projects:<br>

* [tpm2-tss](https://github.com/tpm2-software/tpm2-tss): OSS implementation of
the TCG TPM2 Software Stack (TSS2).
* [tpm2-abrmd](https://github.com/tpm2-software/tpm2-abrmd): TPM2 Access Broker
& Resource Management Daemon implementing the TCG specification.
* [tpm2-tools](https://github.com/tpm2-software/tpm2-tools): The source
repository for the Trusted Platform Module (TPM2.0) tools.
* [tpm2-pkcs11](https://github.com/tpm2-software/tpm2-pkcs11): A PKCS#11
interface for TPM2 hardware.
* [tpm2-tss-engine](https://github.com/tpm2-software/tpm2-tss-engine): OpenSSL
Engine for TPM2 devices.
* [tpm2-tcti-uefi](https://github.com/tpm2-software/tpm2-tcti-uefi): TCTI
module for use with TSS2 libraries in UEFI environment.
* [tpm2-pytss](https://github.com/tpm2-software/tpm2-pytss): Python bindings for
TSS.
* [tpm2-swig](https://github.com/tpm2-software/tpm2-swig): SWIG interface for
high level language bindings.
* [tpm2-totp](https://github.com/tpm2-software/tpm2-totp): Attest the
trustworthiness of a device against a human using time-based one-time passwords.
* [tpm2-software-container](https://github.com/tpm2-software/tpm2-software-container):
Metadata and scripts used to generate the container images used for continuous
integration (CI) by the various tpm2-software projects.
* [tpm2-software.github.io](https://github.com/tpm2-software/tpm2-software.github.io):
This is the source for the community website of the tpm2-software namespace on
GitHub. It uses github pages (with Jekyl).

All the projects use *autotools* since it follows the normal build conventions
and  to make it easier to distribute code portably by creating automatic
dependency generation. For all the projects above: *bootstrap*, *configure*
*make* and *make install* are generally sufficient. The bootstrap and configure
scripts should inform the user of missing libraries and packages. The user can
then install them using the package manager specific to the distribution in use.

# A simple use case - Creating an RSA Key pair and securely persisting in TPM

Once you have all the necessary tpm2 software installed, it is fairly simple to
create an RSA key pair and store it in the TPM persitent memory.<br>

Create the RSA key<br>
1. `tpm2_createprimary --hierarchy=o --key-algorithm=rsa --key-context=prim.ctx`<br>

Save it to the TPM persistent memory<br>
2. `tpm2_evictcontrol --hierarchy=o --object-context=prim.ctx 0x81010001`<br>

NOTE:
* The sensitive portion of the key is never released outside of the TPM2 device.
This makes it a more secure way of generating and protecting the key compared to
the conventional way of using system software like openssl.
* Once the key is generated and persisted in the TPM2 device, it can now be used
throughout the boot sequence as soon as TPM becomes available which is much
earlier than the operating system loads.

# Use case - Disk encryption using Linux Unified Key Setup (LUKS) with TPM2 as the encryption key protector:

## The need for Disk Encryption:

Disk encryption protects storage devices from an attacker with intention to dump
sensitive information by mounting the storage device on alternative operating
environments under attacker control. Of the objectives for disk encryption are:

* Make it difficult for attacker to dump sensitive plain text on disk.
* Choose a strong pass-phrase & a vetted disk encryption software.
* Protect the disk encryption pass-phrase on the platform: Just-in-time OR
secure-key-store.
* Protect pass-phrase access with restricted access to specific platform and
specific system software state
* Flexibility in unsealing pass-phrase during intended system software state
updates/ changes.

To elicit the importance of protecting disk encryption keys on a TPM, let's
first setup a disk image without encryption and see if we can extract user
generated content.<br>

Create a disk image and write some content:<br>
3. `dd if=/dev/zero of=plain.disk bs=1M count=10`<br>
4. `mkfs.ext4 plain.disk`<br>
5. `mkdir -p mountpoint`<br>
6. `sudo mount plain.disk mountpoint`<br>
7. `sudo touch mountpoint/plain.txt`<br>
8. `sudo chmod 777 mountpoint/plain.txt`<br>
9. `sudo echo "This is my plain text" > mountpoint/plain.txt`<br>
10. `sudo umount mountpoint`<br>

At this point we have a disk with a file plain.txt which has the content "This
is my plain text". Let's suppose this is our private and sensitive data. We can
see that it is not protected by dumping the disk and looking at the content to
find the plain.txt. To do this simply do the following:<br>
11. `strings plain.disk`<br>

NOTE:
* Having a week authentication mechanism for login control is not mitigated by
disk encryption. Mounted encrypted volumes without user intervention are
available in clear to the logged in user.
* Data assets especially storage must remain confidential either with disk/ file
 encryption. Choosing and protecting a strong pass-phrase for encryption is
 paramount.

## Simple disk encryption scheme using Linux Unified Key Setup (LUKS) - Plaintext pass-phrase on disk:

Most standard Linux implementations use the LUKS specification. LUKS ensures
storage data confidentiality at rest. LUKS stores setup info in partition header
to aid easy migration. LUKS uses the kernel device mapper subsystem. 8 separate
authentication slots can be specified for single volume with support for
revocation. LUKS volumes can be automatically mounted at startup with standard
linux provisions like services, daemons, etc. Authentication/ Passphrase can be
provided as a password just in time (default) and or specified as key file
(command line argument). The cryptsetup user space utility aids creating and
managing LUKS volumes.<br>

Let's setup a new LUKS volume with a simple passphrase as key protector:<br>
12. `dd if=/dev/zero of=enc.disk bs=1M count=20`<br>
13. `dd if=/dev/urandom of=disk.key bs=1 count=32`<br>
14. `loopdevice=$(losetup -f) && sudo losetup $loopdevice enc.disk`<br>
15. `sudo cryptsetup luksFormat --key-file=disk.key $loopdevice`<br>

At this point you have setup the luks volume and it should pop a warning about
overriding the data. Next let's open the LUKS volume by authenticating with the
disk.key and complete the setting up the disk with a filesystem.<br>
16. `sudo cryptsetup luksOpen --key-file=disk.key $loopdevice enc_volume`<br>
17. `sudo mkfs.ext4 -j /dev/mapper/enc_volume`<br>
18. `sudo mount /dev/mapper/enc_volume mountpoint`<br>

Now lets create a plain text file again and add user content to it:<br>
19. `sudo touch mountpoint/plain.txt`<br>
20. `sudo chmod 777 mountpoint/plain.txt`<br>
21. `sudo echo "This is my plain text" > mountpoint/plain.txt`<br>
22. `sudo umount mountpoint`<br>
23. `sudo cryptsetup remove enc_volume`<br>
24. `sudo losetup -d $loopdevice`<br>

You will now see that you cannot dump the information from the disk image simply
: <br>
25. `strings enc.disk | grep -i plain`<br>

NOTE:
* Do not store the pass-phrase in clear on the disk. In this case the disk.key
passphrase file needs to be secured.
* Either securely provide it  just-in-time or seal it on a security token that
anchors to the platform, like a TPM.

## LUKS disk encryption scheme with pass-phrase stored in TPM2 as the protector.

Auto mount would necessitate providing pass-phrase/ key to cryptsetup at runtime
without user intervention, hence the secret has to be provided to LUKS in clear.
The goal then to prevent an attacker mount the storage on another system can be
trivially defeated by giving access to the encryption secret in clear on the
file-system.<br>

Solution:<br>
a. Seal the secret into a TPM device.<br>
b. Unseal the secret in memory and pass it to cryptsetup.<br>

Please note that there is an upstreaming effort in the cryptsetup
[feature-branch](https://gitlab.com/cryptsetup/cryptsetup/-/tree/wip-tpm) to
enable tpm2 as a key protector built natively in to libcryptsetup as well as
cryptsetup. It does so by adding a new keyslot-handler internally and extending
the libcryptsetup API. In this tutorial however, we will be using tpm2-tools
with cryptsetup that does not have built-in tpm2 support.<br>

Let's start with creating and persisting a sealing object and sealing a random
byte sequence as the disk key.<br>
26. `tpm2_createprimary -Q -C o -c prim.ctx`<br>
27. `dd if=/dev/urandom bs=1 count=32 status=none | tpm2_create -Q -g sha256 -u seal.pub -r seal.priv -i- -C prim.ctx`<br>
28. `tpm2_load -Q -C prim.ctx -u seal.pub -r seal.priv -n seal.name -c seal.ctx`<br>
29. `tpm2_evictcontrol -C o -c seal.ctx 0x81010001`<br>

Now lets change the authentication from previously created disk.key to the new
sealed secret and after that shred the disk.key since it's no longer useful:<br>
30. `tpm2_unseal -Q -c 0x81010001 | sudo cryptsetup luksChangeKey enc.disk --key-file disk.key`<br>
31. `shred disk.key; rm -f disk.key`<br>

Now let's mount the volume with the new authentication sealed up in the tpm:<br>
32. `sudo losetup $loopdevice enc.disk`<br>
33. `tpm2_unseal -Q -c 0x81010001 |sudo cryptsetup luksOpen --key-file=- $loopdevice enc_volume`<br>
34. `sudo mount /dev/mapper/enc_volume mountpoint`<br>

You can now see that disk access is granted with the new secret:<br>
35. `ls mountpoint`
<br>

Finally unmount the disk:<br>
36. `sudo umount mountpoint`<br>
37. `sudo cryptsetup remove enc_volume`<br>
38. `sudo losetup -d $loopdevice`<br>

Attacker now additionally needs the TPM on platform along with the disk since
decryption key isn’t on the disk, its safely stored on the TPM anchored to the
specific platform.

NOTE:
* Access control to security token has similar challenges as protecting
pass-phrases especially if it needs to be provided just-in-time. 
* An authentication mechanism  that anchors to trusted system state and the
platform can be achieved with a TPM and it’s PCR sealing provision. 

## PCR policy authentication - Access control of sealed pass-phrase on TPM2 with PCR sealing

We can add a passphrase to the sealing object. But that necessitates storing it
in clear on the file-system. TPM cannot tell apart who presents the
authentication if any is set for a TPM object; and access is simply granted. At
a higher robustness level, the concern may also be that the attacker can now on
the same system, boot to an alternative OS whose login credentials are fully
controlled by the attacker. In this scenario, the sealing object without any
authentication defeats the security objective of keeping pass-phrase a secret to
the system.<br>

Solution:<br>
a. Implement Measured boot in pre-boot software and measure critical components
including OS-bootloader, kernel, initrd, etc. to PCRs so any changes would
reflect in PCR0.<br>
b. Use PCR as proxy authentication for the TPM sealing object.<br>
c. After unsealing the pass-phrase; extend the sealing PCRs so that the
pass-phrase cannot be unsealed gain.<br>

Let's begin with creating a pcr policy with current value in PCR0 sha256 bank<br>
39. `tpm2_startauthsession -S session.ctx`<br>
40. `tpm2_policypcr -Q -S session.ctx -l sha256:0 -L pcr0.sha256.policy`<br>
41. `tpm2_flushcontext session.ctx`<br>

Now replace the seal object in TPM NV memory protecting the disk encryption
secret with a new one that adds the pcr policy we just created as an
authentication mechanism to access the sealed secret.<br>
42. `tpm2_unseal -c 0x81010001 | tpm2_create -Q -g sha256 -u pcr_seal_key.pub -r pcr_seal_key.priv -i- -C prim.ctx -L pcr0.sha256.policy `<br>
43. `tpm2_evictcontrol -C o -c 0x81010001`<br>
44. `tpm2_load -Q -C prim.ctx -u pcr_seal_key.pub  -r pcr_seal_key.priv -n pcr_seal_key.name -c pcr_seal_key.ctx`<br>
45. `tpm2_evictcontrol -c pcr_seal_key.ctx 0x81010001 -C o`<br>

Now let's try to mount the encrypted disk again but this time the secret is
unsealed off a TPM object whose unsealing operation can only be accessed by
satisfying the PCR policy; in other words authenticating by virtue of intended
system software state being unchanged as reflected by the PCR value.<br>
46. `sudo losetup $loopdevice enc.disk`<br>
47. `tpm2_startauthsession --policy-session -S session.ctx `<br>
48. `tpm2_policypcr -Q -S session.ctx -l sha256:0`<br>

At this point ideally you would want unseal the secret in memory and pipe it
directly to the cryptsetup app like this --> "tpm2_unseal -p session:session.ctx
-c 0x81010001 | sudo cryptsetup luksOpen --key-file=- $loopdevice enc_volume".
However for the purpose of demonstrating flexible PCR in a later section we will
make a copy of the unsealed secret at this point to seal it with a new object
with flexible pcr policy. This breakdown to two steps<br>
49. `tpm2_unseal -p session:session.ctx -c 0x81010001 > disk_secret.bkup`<br>
50. `cat disk_secret.bkup | sudo cryptsetup luksOpen --key-file=- $loopdevice enc_volume`<br>
51. `tpm2_flushcontext session.ctx `<br>
52. `sudo mount /dev/mapper/enc_volume mountpoint/`<br>
53. `ls mountpoint/`<br>

To prevent further attempts of unsealing the disk encryption secret, at this
point the RTM (trusted system software) is expected to extend a random value to
the PCR to prevent further unsealing. This is the same consequence of unintended
or malicious software state reflecting in the PCR. Thus causing the intended
consequence of failed policy check and thus a failed unsealing attempt.<br>
<br>

Let's look at the PCR state prior to extending it and then again after
extending: <br>
54. `tpm2_pcrlist -l sha256:0`<br>
55. `tpm2_pcrextend 0:sha256=0000000000000000000000000000000000000000000000000000000000000000`<br>
56. `tpm2_pcrlist -l sha256:0`<br>

Now let's try to unseal the sealed disk encryption secret with the dirty
PCR:<br>
57. `tpm2_startauthsession --policy-session -S session.ctx `<br>
58. `tpm2_policypcr -Q -S session.ctx -l sha256:0`<br>
The following operation should result in policy check failure preventing the
unseal operation:<br>
59. `tpm2_unseal -p session:session.ctx -c 0x81010001`<br>
60. `tpm2_flushcontext session.ctx`<br>

NOTE:
* The goal is to prevent unsealing of the secret during unintended software
change causing resulting PCR changes.
* However this scheme would break with intended software change and resulting
PCR changes.

## Addressing PCR brittleness from intentional software updates with authorized policies

System software updates is an indispensable part of sane system design. However,
it presents a problem when it comes to sealing secrets to PCR reflecting a
specific system software state. In that, you now are in state where you have a
new set of valid PCRs value that cannot be known ahead of time and yet the 
authentication of the TPM sealing object must be tied to the PCR somehow.<br>

Solution:<br>
Authorized PCR policy as an authentication mechanism for the TPM sealing object.
Instead of using a rigid PCR policy tied to raw PCR values we now seal it to a
PCR signature. The PCR sets are signed by the system designer and verified by
the TPM. This is achieved in following steps:

__a. Get the new set of PCR and sign the pcr policy with signer private key.__<br>
61. `tpm2_startauthsession -S session.ctx`<br>
62. `tpm2_policypcr -Q -S session.ctx -l sha256:0 -L set2.pcr.policy`<br>
63. `tpm2_flushcontext session.ctx`<br>
64. `openssl genrsa -out signing_key_private.pem 2048`<br>
65. `openssl rsa -in signing_key_private.pem -out signing_key_public.pem -pubout`<br>

We now need the name which is a digest of the TCG public key format of the
public key to include in the policy. We can use the loadexternal tool for this:<br>
66. `tpm2_loadexternal -G rsa -C o -u signing_key_public.pem -c signing_key.ctx -n signing_key.name`<br>

Let's now create the signer policy:<br>
67. `tpm2_startauthsession -S session.ctx`<br>
68. `tpm2_policyauthorize -S session.ctx -L authorized.policy -n signing_key.name -i set2.pcr.policy`<br>
69. `tpm2_flushcontext session.ctx`<br>

Let's create a new sealing object with the authorized policy which will also
require the sealing secret for which we will use the disk_secret.bkup we created
at #49 earlier to avoid rebooting the platform to match the PCR we originally
had prior to extending.<br>
70. `cat disk_secret.bkup | tpm2_create -g sha256 -u auth_pcr_seal_key.pub -r auth_pcr_seal_key.priv -i- -C prim.ctx -L authorized.policy`<br>

Let's replace the old persistent sealing object with the one we created
above with policy_authorize policy associated with signer public key:<br>
71. `tpm2_evictcontrol -C o -c 0x81010001`<br>
72. `tpm2_load -Q -C prim.ctx -u auth_pcr_seal_key.pub  -r auth_pcr_seal_key.priv -n auth_pcr_seal_key.name -c auth_pcr_seal_key.ctx`<br>
73. `tpm2_evictcontrol -c auth_pcr_seal_key.ctx 0x81010001 -C o`<br>
Let's now sign the pcr_policy with the signer private key:<br>
74. `openssl dgst -sha256 -sign signing_key_private.pem -out set2.pcr.signature set2.pcr.policy`<br>

__b. Load the signer public key to the tpm and verify the signature on the pcr 
and get the tpm verification tkt:__<br>
75. `tpm2_loadexternal -G rsa -C o -u signing_key_public.pem -c signing_key.ctx -n signing_key.name`<br>
76. `tpm2_verifysignature -c signing_key.ctx -g sha256 -m set2.pcr.policy  -s set2.pcr.signature -t verification.tkt -f rsassa`<br>

__c. Satisfy the authorized policy and then run policyauthorize:__<br>
77. `tpm2_startauthsession --policy-session -S session.ctx`<br>
78. `tpm2_policypcr -l sha256:0 -S session.ctx`<br>
79. `tpm2_policyauthorize -S session.ctx -i set2.pcr.policy -n signing_key.name -t verification.tkt`<br>

__d. Pipe unseal output to the cryptsetup application:__<br>
80. `sudo losetup $loopdevice enc.disk`<br>
81. `tpm2_unseal -p session:session.ctx -c 0x81010001 | sudo cryptsetup luksOpen --key-file=- $loopdevice enc_volume`<br>
82. `tpm2_flushcontext session.ctx `<br>
83. `sudo mount /dev/mapper/enc_volume mountpoint/`<br>
84. `ls mountpoint/`<br>

# Summarizing the various roles in setting up the disk encryption

Disk encryption is a system designer's liability. Loss of the encryption key is
an unintended consequence of denial of service since in this scenario an
attacker cannot be told apart. It may generally be a good idea to save a rescue
key; LUKS affords this flexibility by providing up to 8 slots of authentication
token, all of which can decrypt the disk. Look in to cryptsetup luksAdd option.

## Platform Provisioning<br>
__a. Initialize disk with LUKS :__<br>
   `commands: #12 through #21`<br>
__b. Create pcr policy.__<br>
   `commands: #61, #62, #63`<br>
__c. Create signing authority for signing the policies and provision public key in system.__<br>
   `commands: #64, #65`<br>
__d. Create authorized policy for creating the sealing object.__<br>
   `commands: $67, #68, #69`<br>
__e. Create secure passphrase and seal to the sealing object.__<br>
   `commands: #70`<br>

## System/ Software integrator<br>
__a. Sign valid pcr policies and provide the policy and the signature.__<br>
   `commands: #74`<br>

## Platform runtime<br>
__a. Load signer public key__<br>
   `commands: #75`<br>
__b. Verify signature on the PCR policy and generate verification ticket.__<br>
   `commands: #76`<br>
__c. Satisfy PCR policy and authorized policy.__<br>
   `commands: #77, #78, #79`<br>
__d. Unseal the secret and pipe to cryptsetup to open the LUKS encrypted volume.__<br>
   `commands: #80, #81, #82, #83`<br>

# Author
Imran Desai
