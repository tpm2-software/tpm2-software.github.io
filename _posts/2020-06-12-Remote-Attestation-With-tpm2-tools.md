# Table of Contents

- [Introduction](#introduction)
  * [Software required](#software-required)
  * [Tools and utilities used from the tpm2-tools project](#tools-and-utilities-used-from-the-tpm2-tools-project)
- [Attestation-Goals](#attestation-goals)
  * [Privacy considerations](#privacy-considerations)
    + [Why anonymity or privacy matters?](#why-anonymity-or-privacy-matters)
    + [Solution](#solution)
- [TPM attestation](#tpm-attestation)
  * [What is a PCR and how are PCR values generated](#what-is-a-pcr-and-how-are-pcr-values-generated)
    + [Initial state of the PCR](#initial-state-of-the-pcr)
    + [Extending values into PCR indices](#extending-values-into-pcr-indices)
    + [Golden or reference PCR](#golden-or-reference-pcr)
  * [System software state](#system-software-state)
  * [Roots of trust for reporting (RTR)](#roots-of-trust-for-reporting-(rtr))
- [Roles identified in the bare bone remote attestation model](#roles-identified-in-the-bare-bone-remote-attestation-model)
  * [Device service registration](#device-service-registration)
  * [Service request Part 1 (Platform Anonymous Identity Validation)](#service-request-part-1-platform-anonymous-identity-validation)
  * [Service request Part 2 (Platform Software State Validation)](#service-request-part-2-(platform-software-state-validation))
  * [Service delivery](#service-delivery)
- [Simple attestation with tpm2-tools](#simple-attestation-with-tpm2-tools)
- [Scripts for implementation of the simple attestation framework](#scripts-for-implementation-of-the-simple-attestation-framework)
  * [Device-Node](#device-node)
  * [Service-Provider](#service-provider)
  * [Privacy-CA](#privacy-ca)
- [FAQ](#FAQ)

# Introduction

This article shows how to use the utilities/ tools from the
[tpm2-tools](https://github.com/tpm2-software/tpm2-tools) project to set up a
bare bone remote attestation of the system software state as reflected by the
TPM2 platform-configuration-registers (PCR). The intent is to provide a general
guidance on the topic and does not discuss or reference any attestation
framework in particular.

***CAUTION: All code samples for the sample attestation framework are strictly for
demonstration purpose. It has not been evaluated for production use.***

![Attestation-demo](/images/tpm2-attestation-demo/tpm2-attestation-demo.gif)

## Software required
* [tpm2-tss v2.3.0](https://github.com/tpm2-software/tpm2-tss)
* [tpm2-abrmd v2.2.0](https://github.com/tpm2-software/tpm2-abrmd)
* [tpm2-tools v4.0](https://github.com/tpm2-software/tpm2-tools)
* [ibmswtpm](https://sourceforge.net/projects/ibmswtpm2/)
* [openssl](https://linux.die.net/man/1/openssl)

## Tools and utilities used from the tpm2-tools project
* [tpm2_createprimary](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_createprimary.1.md)
* [tpm2_create](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_create.1.md)
* [tpm2_createek](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_createek.1.md)
* [tpm2_createak](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_createak.1.md)
* [tpm2_getekcertificate](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_getekcertificate.1.md)
* [tpm2_makecredential](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_makecredential.1.md)
* [tpm2_activatecredential](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_activatecredential.1.md)
* [tpm2_pcrextend](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_pcrextend.1.md)
* [tpm2_pcrread](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_pcrread.1.md)
* [tpm2_quote](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_quote.1.md)
* [tpm2_checkquote](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_checkquote.1.md)
* [tpm2_getrandom](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_getrandom.1.md)
* [tpm2_readpublic](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_readpublic.1.md)

# Attestation-Goals

“Attestation is the evidence or proof of something. It is a declaration that
something exists or is the case. It is the action of being a witness to or
formally certifying something.”<br>

The [literary-definition](https://www.lexico.com/en/definition/attestation)
very much applies in the context of this tutorial which discusses attesting of
the PCR contents in a TPM.

An attestation has two critical parts for it to hold true, namely:

1. Attestor identity:
    To believe something is true, one needs to vouch the authenticity of what is
    being attested. This inherently means the identity of the one who attests
    is known and or trusted.

2. Attestation data integrity:
    To believe something is true, both the process to generate the information
    and the process to protect the information from tampering need to be
    inherently trusted.

## Privacy considerations

While private or sensitive portion of keys used to sign attestation blobs must
remain confidential or secret, it is possible to identify a unique signer with
the public key used to verify the signature over the attestation blobs. This
defeats the anonymity or privacy of the platform user.

### Why anonymity or privacy matters?

Let's use a hypothetical example of a smart-lock device that has an embedded
application. It has integrated multiple microservices from different providers
that provide various services, namely:
1. Two-factor-authentication(2FA).
2. Facial-recognition.
3. Device-Logs.

If the device uses the same key across all the microservices for public key
authentication and or signing attestation quotes. The service provider or a
third party analytics could potentially stage a privacy violation knowingly or
unknowingly simply by tracking usage of same public key. And so it can be
determined that the device owner uses facial recognition for 2FA and operates
the device consistently certain times of the day which updates the device logs.

### Solution

There are at least four possible ways to resolve this.
1. Every user uses the same private/public key pair to sign attestation blobs.
   Cryptographically, this method is as strong as the process and controls
   involved in duplicating the private key into multiple platforms.
   Additionally, the strength is also dependent on the robustness of the methods
   and mechanisms to store the private key once imported in the platform.
   Revocation of the key implies that all the platforms having the key must be
   revoked and re-provisioned with new key. Refer [tpm2_import](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_import.1.md)
   that demonstrates a way to import external keys into the TPM.
2. Every user has a unique private key, however, all the private keys keys map
   to a single public key. This is possible with [Intel® EPID](https://eprint.iacr.org/2009/095.pdf)
   technology. EPID also affords a mechanism to revoke a single key or all the
   platforms in the group. A similar direct-anonymous-attestation (DAA) based on
   elliptic curve cryptography scheme is also available in the TPM and is called
   ECDAA; it is not discussed in this tutorial.
3. Every user generates and uses a new key (ephemeral key) every time an
   attestation blob has to be signed. There are some unique challenges with this.
   If every attestation blob is signed with a brand-new key, how to infer the
   anonymous identity at the minimum to determine the genuineness of the platform
   and hence the attestation. It then follows that we need an anonymous identity
   that is cryptographically bound to a unique trusted identity and that the
   unique identity is never revealed to any entity other than the Privacy-CA.
4. A variant of #3. One AIK per party that's verifying attestations. Allows
   abstracting the client ID, but the verifier knows it's the same client on
   subsequent verifications. This is useful in applications like banking etc.

Note: Platform anonymity can also be defeated if the platform's host-name and or
ip-address remains the same on every connection. This is not a topic of
discussion here, yet there is an assumption made here that sufficient measures
are taken by the platform user to resolve this using a
[VPN](https://www.pcmag.com/how-to/how-to-hide-your-ip-address) or other
reasonable mechanisms.

# TPM attestation

Depending on the robustness and privacy rules of the system, platform
anonymity may not be mandatory and so privacy considerations don't apply. If
that is the case or if methods #1 or #2 discussed above suffice your attestation
and privacy needs, subsequent sections detailing information on TCG endorsement
keys (EK) and attestation identity keys (AIK) are irrelevant.

Note: A more detailed discussion on TPM attestation terminologies can be found
[here](https://tpm2-software.github.io/tpm2-tss/getting-started/2019/12/18/Remote-Attestation.html).
This document expands on that discussion to demonstrate how one can set up a
minimal or bare bones attestation framework using tpm2-tools.

With these definitions in mind, let's dive straight into TPM attestation topic.
Of the various object types in the TPM, the following discussion is restricted
to how the TPM PCR data can be attested securely and anonymously using method #3.

## What is a PCR and how are PCR values generated

PCR or platform configuration registers are special TPM2 objects that can only
be modified or written to by hash extension mechanism. In that, the incoming
new value is concatenated with the existing value in the PCR and hashed. The
new hash value now replaces the old value. This means that though it is a single
location, the final value reflects a history of all the hash extensions. PCRs
are arranged in banks, one each for a specific hash algorithm SHA1, SHA256, etc.
Every bank has up to 32 PCRs. The Trusted Computing Group publishes a guideline
of what part of system software should be extended to specific PCR indices in
the [PC Client specification](https://trustedcomputinggroup.org/wp-content/uploads/TCG_PCClientSpecPlat_TPM_2p0_1p04_pub.pdf).
There is also a debug PCR at index 16 that can be reset to 0 by issuing the
tpm2_pcrreset command. Other PCRs reset on TPM resets.

The fact that the only way to modify a PCR is to extend a hash and that the
robustness mechanisms that prevent physical tampering of the values, make the
TPM a root of trust for storage (RTS).

### Initial state of the PCR

The initial values in a PCR index is determined by the platform-specific
[specification](https://www.trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2-0-v43-150126.pdf).
Values at a given PCR index can be read using the **tpm2_pcrread** tool and
specifying the PCR-selection string. Relevant to our example, let's read PCR
indices 0,1,2 in SHA1 and SHA256 banks.

```bash
tpm2_pcrread sha1:0,1,2+sha256:0,1,2
```

If running it on a simulator, the PCR are in the initial state, in that the PCR
are not extended just yet. Hence, the default values (00/ FF) should be displayed
as shown below.

SHA1<br>
  0 : 0000000000000000000000000000000000000000<br>
  1 : 0000000000000000000000000000000000000000<br>
  2 : 0000000000000000000000000000000000000000<br>

SHA256<br>
  0 : 0000000000000000000000000000000000000000000000000000000000000000<br>
  1 : 0000000000000000000000000000000000000000000000000000000000000000<br>
  2 : 0000000000000000000000000000000000000000000000000000000000000000<br>

### Extending values into PCR indices

A PCR index can be modified only by extending it. In that, it cannot be set to
an absolute value like a register. One of the three tools can modify the PCR
values, namely:

1. **tpm2_pcrextend**: A pre-calculated digest of data is presented to the TPM.
2. **tpm2_pcrevent**: The data is directly presented to the TPM and the TPM
   calculates the data-digest prior to extending.
3. **tpm2_pcrreset**: The PCR index value is reset to zero. Not all PCR indices
   are resettable.

In the case of **tpm2_pcrextend** and **tpm2_pcrevent**, the TPM ultimately
concatenates the incoming data-digest with the current value at the PCR index,
hashes the concatenation-data and replaces the PCR index value in place.

It should be noted that the entity extending the PCR is inherently trusted. This
trusted entity is called a Root-of-trust-for-measurement (RTM) in TCG
terminology. [Intel (R) Bootguard](https://en.wikipedia.org/wiki/Intel_vPro#Intel_Boot_Guard)
is an example. The tamper resistant code extends the initial measurements of the
critical portions of the pre-boot software.

Relevant to our example scripts in this tutorial below is an example of how to
use **tpm2_pcrextend** to modify PCR indices.

```bash
#
# Let's suppose the critical portion of software to be extended is a plain text
# "CRITICAL-DATA". We need to first calculate the hash using a crypto lib/tool
# like openssl and then pass it to the tpm2_pcrextend command.
#
SHA256_DATA=`echo "CRITICAL-DATA" | openssl dgst -sha256 -binary | xxd -p -c 32`
SHA1_DATA=`echo "CRITICAL-DATA" | openssl dgst -sha1 -binary | xxd -p -c 20`

tpm2_pcrextend 0:sha1=$SHA1_DATA,sha256=$SHA256_DATA
tpm2_pcrextend 1:sha1=$SHA1_DATA,sha256=$SHA256_DATA
tpm2_pcrextend 2:sha1=$SHA1_DATA,sha256=$SHA256_DATA

# Let's read the PCR values now

tpm2_pcrread sha1:0,1,2+sha256:0,1,2

#
# The new PCR values should be as follows:
#sha1:
#  0 : 0xA3EBF00F6520B2C85DBBF3D32B6A8B3A30ABB748
#  1 : 0xA3EBF00F6520B2C85DBBF3D32B6A8B3A30ABB748
#  2 : 0xA3EBF00F6520B2C85DBBF3D32B6A8B3A30ABB748
#sha256:
#  0 : 0xAF42D77065F4791B6738DA5944E6B4074E3190F0993B5EE5D42DC4FBED424ABA
#  1 : 0xAF42D77065F4791B6738DA5944E6B4074E3190F0993B5EE5D42DC4FBED424ABA
#  2 : 0xAF42D77065F4791B6738DA5944E6B4074E3190F0993B5EE5D42DC4FBED424ABA
#

# Let's see how we got one of these values as a demonstration of PCR extension.
INITIAL_SHA1_DATA="0000000000000000000000000000000000000000"
CONCATENATED=`echo -ne $INITIAL_SHA1_DATA; echo $SHA1_DATA`
echo $CONCATENATED | xxd -r -p | openssl dgst -sha1
# This should output 0xA3EBF00F6520B2C85DBBF3D32B6A8B3A30ABB748
```

### Golden or reference PCR

The term golden/ reference is not a formal terminology. It is simply a digest of
the content of the PCR indices in a PCR selection. The selection is a choice of
PCR indices across all the PCR banks. Values in a PCR index itself is a digest
of the binary blob of software/ data that represents a known good state.
The digest algorithm for individual PCR indices is the same as that of the
specific bank. Whereas, the digest of the golden/ reference state has to be
specified.

The golden/reference PCR state can be calculated in one of three ways:

1. Using the **tpm2_quote** tool. The signing scheme used to sign the quote
determines the digest algorithm for the quote. This should be done once on a
reference platform.

```bash
tpm2_quote \
--key-context rsa_ak.ctx \
--pcr-list sha1:0,1,2+sha256:0,1,2 \
--message pcr_quote.plain \
--signature pcr_quote.signature \
--qualification SERVICE_PROVIDER_NONCE \
--hash-algorithm sha256 \
--pcr pcr.bin

GOLDEN_PCR=`cat pcr.bin | openssl dgst -sha256 -binary | xxd -p -c 32`
```

2. If only the PCR data is to be read from the reference platform, it can
always be done using **tpm2_pcrread** tool as well. Also, the output of the
**tpm2_pcrread** tool can be passed to the **tpm2_checkquote** tool directly.
However, when doing so, specifying the PCR-selection information to the
**tpm2_checkquote** tool is a must. Calculating the golden/ reference PCR data
using this method is shown below.

```bash
tpm2_pcrread sha1:0,1,2+sha256:0,1,2 -o pcr.bin

GOLDEN_PCR=`cat pcr.bin | openssl dgst -sha256 -binary | xxd -p -c 32`
```


3. Entirely without the TPM as shown below.

```bash
CONCATENATE_ALL_DIGESTS=`\
echo $SHA1_DATA
echo $SHA1_DATA
echo $SHA1_DATA
echo $SHA256_DATA
echo $SHA256_DATA
echo $SHA256_DATA
`
GOLDEN_PCR=`\
echo $CONCATENATE_ALL_DIGESTS |
xxd -r -p | openssl -dgst -sha256 -binary | xxd -p -c 32
`
```

Using any of the 3 methods above the GOLDEN_PCR value is
"e756e3af77a4f15a3f2ed489a7411a93d91d619506b6d1ed1121faaeaf45d8de".

## System software state

In a nutshell, this is a single digest aka measurement of interesting/ critical
pieces of system software. To do this, a trusted portion of the software does
the following:
1. Calculates the hash of the software module that will be loaded for execution.
2. Sends over the digest to the TPM and requests it be extended in the PCR.

Note: The trusted portion of the system software extending measurements is
termed root-of-trust-for-measurement (RTM). This is inherently trusted.

Throughout the platform boot process, a log of all executable code and relevant
configuration data is created and extended into PCRs. Each time a PCR is
extended, a log entry is made in the TCG event log. This allows a challenger to
see how the final PCR digests were built. The event log can be examined using
the [tpm2_eventlog](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_eventlog.1.md)
tool. The event log is typically stored at this location
/sys/kernel/security/tpm0/binary_bios_measurements.

## Roots of trust for reporting (RTR)

The private or sensitive portion of the TPM key object is protected by the TPM.
Couple this with authentication and enhanced authorization models the TPM
design affords, the signatures generated from such keys are deemed trustworthy.
As discussed earlier that identity is yet another facet to the trustworthiness of
the signature and hence the data. We also determined that in order to preserve
the privacy of the end user there is need for both anonymous identity and
a unique identity that are cryptographically bound. Together these requirements
form a criterion for defining a TPM object and these properties make the TPM a
root of trust for reporting (RTR).

Of the four hierarchies the TPM is partitioned into, the endorsement hierarchy
is to be used by the privacy administrator. The primary key created under the
endorsement hierarchy provides the unique identity and is called the endorsement
key (EK). Following properties make EKs special amongst other primary keys that
can be created under any of the hierarchies:
1. TCG EK cannot be used as a signing key.
2. TCG EK authentication is a policy that references endorsement hierarchy
authorization.
3. TCG EK are certified by the TPM manufacturer.
More details on the key can be found from
[TCG EK credential specification](https://trustedcomputinggroup.org/wp-content/uploads/TCG_IWG_Credential_Profile_EK_V2.1_R13.pdf).
As already mentioned, the EK is a primary key and so can be created using the
[tpm2_createprimary](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_createprimary.1.md)
tool by supplying the right attributes and authorization policy. In order to
simplify attestation, a tool
[tpm2_createek](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_createek.1.md)
is also available that has all the defaults specified by TCG EK credential spec.

The anonymous identity is the attestation identity key (AIK) created with the EK
as its parent. There is no specific key template that is mandated by TCG that
determines the AIK key attributes or authorization model. Since the key is
typically used in privacy sensitive operations like quoting/ signing/ certifying
, the key is a signing key created under endorsement hierarchy with privacy
administrator controls. And so it's authorization model typically references the
authorization of the endorsement hierarchy through a policy. The association of
the AIK to an EK is done using a cryptographical method called credential
activation. Unlike EK which is a constant/ unique primary key that can be
re-created, the AIK keys are ephemeral. In that, every time an AIK is created it
results in a brand-new key and thus makes the key anonymous. The AIK as
mentioned is a child of the EK primary key and can be created with the tool
[tpm2_create](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_create.1.md).
Alternatively, the key can also be created using the [tpm2_createak](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_createak.1.md)
in order to simply attestation. The tool implements the TPM2_CC_Create command
with the most commonly used AIK properties.

In summary,
1. The TPM manufacturer EK certificate vouches for unique identity;
while the credential activation process vouches the association of the AIK to
the EK and hence the TPM.
2. tpm2_createek and tpm2_createak tools can be used to create EK and AIK. Any
further customizations to the keys outside the chosen defaults can be done by
creating the objects with tpm2_createprimary and tpm2_create respectively.

Note:
System software state can also be deemed as a system's trusted identity without
requiring a signing key. Such an identity is only useful for self attestation of
the system. This is also known as local attestation. Local attestation has use
cases like [sealing-encryption-secrets](https://tpm2-software.github.io/2020/04/13/Disk-Encryption.html)
or using PCR state as proxy authentication model to make authorization for TPM
objects valid as long as system identity aka system software state does not
change.

# Roles identified in the bare bone remote attestation model

The primary goals for the minimal bare bone remote attestation are:
1. To demonstrate verification of an attestation quote.
2. The quote should contain the digest of the PCR values representing system
   software state.
3. Signer of the quote should be anonymous to the verifier; yet the verifier
   should be able to make a determination that the quote was signed by a valid
   signer.

To achieve the goals it is sufficient to have 3 players, namely:
1. Device-Node: The edge platform with a TPM whose system software state is of
   interest. It generates the attestation structures with digest of PCR data
   included in the quotes. The platform signs the quotes with an attestation
   identity key for anonymity. AIK is cryptographically bound to the unique
   identity key on the same platform. The unique identity key is the
   endorsement-key (EK).
2. Privacy-CA: The only trusted entity that can prove the association of an AIK
   to a valid EK without disclosing the EK to the “Service-Provider”. This is
   also the only entity in addition to the “Device-Node” that knows the EK for a
   given AIK from the “Device-Node“.
3. Service-Provider: The entity that the “Device-Node“ communicates with to
   avail services. The “Service-Provider“ need to ensure the following:<br>
   a. Entity requesting services has registered its unique identity with
      the “Privacy-CA“.<br>
   b. Anonymous identity belongs to the pool of registered unique identities
      that the “Privacy-CA“ stores.<br>
   c. System software state of the “Device-Node“ is an acceptable one.

**In practice, however the various roles as shown here can be further broken
down. As an example, all the verifications for anonymous-identity and
system-software state can be handed off to an additional “Verifier“ role.**

Let's now look at the various stages of our example attestation framework.
1. Service-registration.
2. Service-request Part 1 (Anonymous identity validation).
3. Service-request Part 2 (Platform software state validation).
4. Service-delivery.

## Device service registration

This is the stage where the “Device-Node“ requests services from the
“Service-Provider“. The “Service-Provider“ has two requirements at this stage.
1. The “Device-Node“ sends its unique key to the “Privacy-CA“
2. The “Privacy-CA“ verifies the genuineness of the EK from the EK certificate
   validation and also the presence of the EK on the platform through the
   credential-activation process.

In order to verify that #1 & #2 above happened, the “Service-Provider“ creates
an ephemeral secret called REGISTRATION-TOKEN that it shares only with the
Privacy-CA who in turn reveals it to the “Device-Node“ if and only if all
verifications on EK-certificate and Credential-Activation pass which are only
possible if “Device-Node“ communicates the platform EK and AIK with the
Privacy-CA. The “Device-Node“ then presents the REGISTRATION-TOKEN to the
“Service-Provider“ to complete registration.

Note: The “Service-Provider“ has no further information recorded about the
platform at the end of the registration. It only knows that a registration
request was made by some platform and that it's EK is registered in the pool
of valid EKs with the Privacy-CA.

The sequence diagram below shows the interactions during the registration.
![Registration](/images/tpm2-attestation-demo/registration.png)

## Service request Part 1 (Platform Anonymous Identity Validation)

This is the stage where a registered platform makes a service request to the
“Service-Provider“ and sends an ephemeral AIK to securely procure the services.
The “Service-Provider“ needs to know the following:
1. The AIK is a valid one and that the Privacy-CA can prove that the AIK is
   bound to an EK from the Privacy-CA's trusted pool of EKs.
2. The AIK is from a TPM currently accessible to the platform.

In order to achieve this, The Privacy-CA needs to request the EK from the
platform and also ensure the AIK is present on the platform through the
credential activation process.

In order to verify these interactions between the “Privacy-CA“ and the
“Device-Node“ occurred, the “Service-Provider“ creates an ephemeral secret called
SERVICE-TOKEN that it shares with the “Privacy-CA“ who in turn reveals it to the
“Device-Node“ after a successful credential activation process. The “Device-Node“
now proves the validity of the service request by presenting the SERVICE-TOKEN
back to the “Service-Provider“.

The sequence diagram below shows the interactions during the service-request.
![Identity-validation](/images/tpm2-attestation-demo/identity-validation.png)

## Service request Part 2 (Platform Software State Validation)

At this stage, by validating the “SERVICE-TOKEN” presented by the device, the
“Service-Provider“ has ascertained the AIK comes from a registered platform and
that it is a trusted signing key. Now the “Service-Provider“ needs to assured of
the system-software state of the “Device-Node“. To achieve this the
“Service-Provider“ requests an attestation quote from the “Device-Node“ that has
to be signed with the AIK. In order to prevent replay attacks, the
“Service-Provider“ generates a NONCE that needs to be added to the attestation
quote before signing.

Below is a YAML representation of PCR included attestation data returned by
tpm2_quote.

```yaml
TPM2B_ATTEST_DATA:
    size
    attestationData:
        magic
        type
        qualifiedSigner
        extraData
        clockInfo:
            clock
            resetCount
            restartCount
            safe
        firmwareVersion
        quote:
            pcrSelect
                count
                pcrSelections[TPM2_NUM_PCR_BANKS]:
                    hash
                    sizeofSelect
                    pcrSelect[TPM2_PCR_SELECT_MAX]
            pcrDigest
```

Shown above in yaml representation is all the information included in an
attestation quote. Brief description of the important fields follow:

* magic: the indication that this structure was created by a TPM always
TPM2_GENERATED_VALUE.

* type: The type of the attestation structure. It is TPM2_ST_ATTEST_QUOTE for
the type that has the PCR information and being discussed at length here in.

* qualifiedSigner: Qualified Name of the signing key. The term qualified name
is the digest of all the Names of all the ancestor keys back to the Primary Seed
at the root of the hierarchy.

* extraData: External information supplied by caller. The NONCE generated by the
“Service-Provider“ is added here in this field.

* Clock: The time in milliseconds during which the TPM has been powered. This
value is reset to zero when the Storage Primary Seed is changed TPM2_Clear.

* resetCount: The number of occurrences of TPM Reset since the last TPM2_Clear.

* restartCount: The number of times that TPM2_Shutdown or _TPM_Hash_Start have
occurred since the last TPM Reset or TPM2_Clear.

* Safe: Indicates that no value of Clock greater than the current value of Clock
has been previously reported by the TPM. Set to YES on TPM2_Clear.

* firmwareVersion: TPM vendor-specific value identifying the version number of
the firmware.

* pcrSelect: The information on algID, PCR selected, and the digest.

* count: The number of selection structures. A value of zero is allowed. This
indicates the count of selected PCR banks (SHA1, SHA256, etc.)
  
* pcrSelections: This is a list of PCR selection structures.
   1. hash: The hash algorithm associated with the selection bank.
   
   2. sizeofSelect: The size in octets of the pcrSelect array. This
   indicates number of bytes required to represent all the
   PCRs in a bank. Every PCR is represented as a bit. E.g. For 24 PCRs per bank
   the sizeofselect should be 3 bytes.

   3. pcrSelect: The bit map of selected PCR (the least significant byte first)

* pcrDigest: The digest of selected PCR using hash algorithm of the signing key.

Below is an example dump from an attestation quote for PCR0 from SHA1 bank and
PCR0, PCR1 from SHA256 bank.

```yaml
magic: ff544347
type: 8018
qualifiedSigner: 000bfebea9500be6aff07565dc09537ae5c887e1cc550a1a4653a618f86486ac28fe
extraData: 12345678
clockInfo:
  clock: 2260078
  resetCount: 0
  restartCount: 0
  safe: 1
firmwareVersion: 3636160019061720
attested:
  quote:
    pcrSelect:
      count: 2
      pcrSelections:
        0:
          hash: 11 (sha256)
          sizeofSelect: 3
          pcrSelect: 010000
        1:
          hash: 4 (sha1)
          sizeofSelect: 3
          pcrSelect: 030000
    pcrDigest: 834a709ba2534ebe3ee1397fd4f7bd288b2acc1d20a08d6c862dcd99b6f04400
```

The sequence diagram below shown the interactions during this stage.
![Software-state-validation](/images/tpm2-attestation-demo/software-state-validation.png)

## Service delivery

At this stage, by validating the “SERVICE-TOKEN“ and the attestation-quote it
received from the “Device-Node“, the “Service-Provider“ has ascertained the
“Device-Node“ anonymous identity and system-software state. With the system in
a known state, the “Service-Provider“ can now wrap the “SERVICE-SECRET“ with the
AIK Public Key.

![Service-delivery](/images/tpm2-attestation-demo/service-delivery.png)

# Simple attestation with tpm2-tools

The following demonstration shows the tpm2-tools involved in the sequence
diagrams discussed above. Following the demonstration of the tpm2-tools in this
section, there is a subsequent section showing the scripts for the different
roles and their interactions in the simple attestation framework. It involves
scripting the tools and the logic to move identities, quotes, tokens, nonce,
etc. between the “Service-Provider“, “Device-Node“ and the “Privacy-CA“.

With that preface, let's dive straight into the tpm2-tools.

1. “Device-Node“ creating the endorsement-key and the attestation-identity-key.

```bash

tpm2_createek \
--ek-context rsa_ek.ctx \
--key-algorithm rsa \
--public rsa_ek.pub

tpm2_createak \
--ek-context rsa_ek.ctx \
--ak-context rsa_ak.ctx \
--key-algorithm rsa \
--hash-algorithm sha256 \
--signing-algorithm rsassa \
--public rsa_ak.pub \
--private rsa_ak.priv \
--ak-name rsa_ak.name
```

2. “Device-Node“ retrieving the endorsement-key-certificate to send to the
“Privacy-CA“. There are two possible locations where the endorsement key
certificates are provided by the TPM manufacturer. While most TPM manufacturers
store them in the [TCG specified NV indices]((https://trustedcomputinggroup.org/wp-content/uploads/TCG_IWG_Credential_Profile_EK_V2.1_R13.pdf))
, some make it available for download through a web hosting. Let's look at both
these methods.

```bash
# Location 1 - TPM2 NV Index 0x1c00002 is the TCG specified location for RSA-EK-certificate.
RSA_EK_CERT_NV_INDEX=0x01C00002

NV_SIZE=`tpm2_nvreadpublic $RSA_EK_CERT_NV_INDEX | grep size |  awk '{print $2}'`

tpm2_nvread \
--hierarchy owner \
--size $NV_SIZE \
--output rsa_ek_cert.bin \
$RSA_EK_CERT_NV_INDEX

# Location 2 - Web hosting. This applies specifically to Intel(R) PTT RSA-EK-certificate.
tpm2_getekcertificate \
--ek-public rsa_ek.pub \
--offline \
--allow-unverified \
--ek-certificate rsa_ek_cert.bin

## convert to a standard DER format
sed 's/-/+/g;s/_/\//g;s/%3D/=/g;s/^{.*certificate":"//g;s/"}$//g;' \
rsa_ek_cert.bin | base64 --decode > rsa_ek_cert.bin
```

3. “Privacy-CA“ and the “Device-Node“ performing a credential activation
challenge in order to verify the AIK is bound to the EK from the EK-certificate
originally shared by the “Device-Node“. This is done in two different instances
in the proposed simple-attestation-framework — Once when the “Service-Provider“
requests the “Device-Node“ to send over the identities as part of the service
registration process. And the second time when the “Device-Node“ sends its AIK
to the “Service-Provider“ and the “Service-Provider“ in turn sends it over to
the “Privacy-CA“ in order to verify the anonymous identity.

```bash
# Privacy-CA creating the wrapped credential and encryption key
file_size=`stat --printf="%s" rsa_ak.name`
loaded_key_name=`cat rsa_ak.name | xxd -p -c $file_size`

echo "this is my secret" > file_input.data
tpm2_makecredential \
--tcti none \
--encryption-key rsa_ek.pub \
--secret file_input.data \
--name $loaded_key_name \
--credential-blob cred.out

# Device-Node activating the credential
tpm2_startauthsession \
--policy-session \
--session session.ctx

TPM2_RH_ENDORSEMENT=0x4000000B
tpm2_policysecret -S session.ctx -c $TPM2_RH_ENDORSEMENT

tpm2_activatecredential \
--credentialedkey-context rsa_ak.ctx \
--credentialkey-context rsa_ek.ctx \
--credential-blob cred.out \
--certinfo-data actcred.out \
--credentialkey-auth "session:session.ctx"

tpm2_flushcontext session.ctx
```

4. “Device-Node“ generating the PCR attestation quote on request from the
“Service-Provider“. The “Service-Provider“ specifies the PCR-banks, PCR-indices,
and the ephemeral NONCE data. The NONCE is to ensure there is no possibility of
a replay attack on the quote verification and validation process. Validity of
the signing key for attestation quote is ascertained to be a valid one by the
“Privacy-CA“.

```bash
echo "12345678" > SERVICE_PROVIDER_NONCE

tpm2_quote \
--key-context rsa_ak.ctx \
--pcr-list sha1:0,1,2+sha256:0,1,2 \
--message pcr_quote.plain \
--signature pcr_quote.signature \
--qualification SERVICE_PROVIDER_NONCE \
--hash-algorithm sha256 \
--pcr pcr.bin
```

5. “Service-Provider“ verifying the attestation quote generated and signed by
the “Device-Node“. To make the determination of the software-state of the
“Device-Node“, after the signature and nonce verification process, the
“Service-Provider“ validates the digest of the PCR values in the quote against
a known-good-valid —the [golden/ reference value](#golden-or-reference-pcr)
ascertained previously.

```bash
tpm2_checkquote \
--public rsa_ak.pub \
--message pcr_quote.plain \
--signature pcr_quote.signature \
--qualification SERVICE_PROVIDER_NONCE \
--pcr pcr.bin
```

# Scripts for implementation of the simple attestation framework

## Device-Node
```bash
#!/bin/bash

# Fixed location
service_provider_location="$PWD/../SP"

# PCA location
privacy_ca_location=""

# Location for node 1, node 2, etc.
device_location="$PWD"

# State
event_file_found=0
device_registration_request=0
device_service_request=0

wait_loop() {
    counter=1
    until [ $counter -gt $1 ]
    do
       test -f $2
       if [ $? == 0 ];then
          event_file_found=1
          break
       else
          echo -ne "Waiting $1 seconds: $counter"'\r'
       fi
       ((counter++))
       sleep 1
    done
}

LOG_ERROR() {
    errorstring=$1
    echo -e "\033[31mFAIL: \e[97m${errorstring}\e[0m"
}

LOG_INFO() {
    messagestring=$1
    echo -e "\033[93mPASS: \e[97m${messagestring}\e[0m"
}

await_and_compelete_credential_challenge() {
    
    # Wait for credential challenge
    cred_status_string="Encrypted credential receipt from Privacy-CA."
    max_wait=60
    wait_loop $max_wait cred.out
    if [ $event_file_found == 0 ];then
        LOG_ERROR "$cred_status_string"
        return 1
    fi
    event_file_found=0
    LOG_INFO "$cred_status_string"

    tpm2_startauthsession --policy-session --session session.ctx -Q

    TPM2_RH_ENDORSEMENT=0x4000000B
    tpm2_policysecret -S session.ctx -c $TPM2_RH_ENDORSEMENT -Q

    tpm2_activatecredential --credentialedkey-context rsa_ak.ctx \
    --credentialkey-context rsa_ek.ctx --credential-blob cred.out \
    --certinfo-data actcred.out --credentialkey-auth "session:session.ctx" -Q
    
    rm -f cred.out

    tpm2_flushcontext session.ctx -Q

    rm -f session.ctx
}

device_registration() {

    # Send device location to service-provider
    echo "device_location: $device_location" > d_s_registration.txt
    cp d_s_registration.txt $service_provider_location/.
    rm -f d_s_registration.txt

    # Wait for PCA location information from service provider
    max_wait=60
    wait_loop $max_wait s_d_registration.txt
    registration_status_string="Privacy-CA information receipt from Service-Provider."
    if [ $event_file_found == 0 ];then
        LOG_ERROR "$registration_status_string"
        return 1
    fi
    event_file_found=0
    LOG_INFO "$registration_status_string"
    privacy_ca_location=`grep privacy_ca_location s_d_registration.txt | \
    awk '{print $2}'`
    rm -f s_d_registration.txt

    registration_status_string="Acknowledgement reciept from Privacy-CA."
    wait_loop $max_wait p_d_pca_ready.txt
    if [ $event_file_found == 0 ];then
        LOG_ERROR "$registration_status_string"
        return 1
    fi
    event_file_found=0
    LOG_INFO "$registration_status_string"
    rm -f p_d_pca_ready.txt

    #Ready EKcertificate, EK and AIK and set ready status so PCA can pull
    tpm2_createek --ek-context rsa_ek.ctx --key-algorithm rsa \
    --public rsa_ek.pub -Q

    tpm2_startauthsession -S session.ctx --policy-session -Q
    tpm2_policysecret -S session.ctx -c e -Q
    tpm2_create -C rsa_ek.ctx -c rsa_ak.ctx -u rsa_ak.pub -r rsa_ak.priv \
    -P session:session.ctx -Q
    tpm2_readpublic -c rsa_ak.ctx -f pem -o rsa_ak.pub -n rsa_ak.name -Q
    tpm2_flushcontext session.ctx -Q

    touch fake_ek_certificate.txt

    touch d_p_device_ready.txt
    cp d_p_device_ready.txt $privacy_ca_location/.
    rm -f d_p_device_ready.txt

    registration_status_string="Credential activation challenge."
    await_and_compelete_credential_challenge
    if [ $? == 0 ];then
        LOG_INFO "$registration_status_string"
        cp actcred.out $privacy_ca_location/.
        rm -f actcred.out
        return 0
    else
        LOG_ERROR "$registration_status_string"
        return 1
    fi
}

request_device_registration () {

    device_registration
    if [ $? == 1 ];then
        return 1
    fi

    device_registration_status_string="Registration token receipt from Privacy-CA."
    max_wait=60
    wait_loop $max_wait p_d_registration_token.txt
    if [ $event_file_found == 0 ];then
        LOG_ERROR "$device_registration_status_string"
        return 1
    fi
    LOG_INFO "$device_registration_status_string"
    event_file_found=0
    cp p_d_registration_token.txt \
    $service_provider_location/d_s_registration_token.txt
    rm -f p_d_registration_token.txt

    return 0
}

#
# Request service with the Service-Provider
# Read the Privacy-CA location from Service-Provider
# Deliver EK, AIK, EKcertificate to the Privacy-CA
# Complete credential challenge with the Privacy-CA
# Retrieve the SERVICE-TOKEN from the Privacy-CA
# Present the SEVICE-TOKEN to the Service-Provider
#
process_device_anonymous_identity_challenge() {

   # Start device service
   test -f $device_service_aik
   if [ $? == 1 ];then
      LOG_ERROR "Aborting service request - AIK could not be found."
      return 1
   else
      echo "device_location: $device_location" > d_s_service.txt
      cp d_s_service.txt $service_provider_location/.
      rm -f d_s_service.txt
      cp $device_service_aik $service_provider_location/d_s_service_aik.pub
   fi

   identity_challenge_status_string="Privacy-CA information receipt from Service-Provider."
   max_wait=60
   wait_loop $max_wait s_d_service.txt
   if [ $event_file_found == 1 ];then
    event_file_found=0
    privacy_ca_location=`grep privacy_ca_location s_d_service.txt | \
    awk '{print $2}'`
    rm -f s_d_service.txt
    LOG_INFO "$identity_challenge_status_string"
   else
    LOG_ERROR "$identity_challenge_status_string"
    return 1
   fi

    identity_challenge_status_string="Acknowledgement receipt from Privacy-CA."
    wait_loop $max_wait p_d_pca_ready.txt
    if [ $event_file_found == 0 ];then
        LOG_ERROR "$identity_challenge_status_string"
        return 1
    fi

    LOG_INFO "$identity_challenge_status_string"
    event_file_found=0
    rm -f p_d_pca_ready.txt

    touch d_p_device_ready.txt
    cp d_p_device_ready.txt $privacy_ca_location/.
    rm -f d_p_device_ready.txt

    identity_challenge_status_string="Credential activation challenge."
    await_and_compelete_credential_challenge
    if [ $? == 0 ];then
        LOG_INFO "$identity_challenge_status_string"
        cp actcred.out $privacy_ca_location/.
        rm -f actcred.out
    else
        LOG_ERROR "$identity_challenge_status_string"
        rm -f actcred.out
        return 1
    fi

    identity_challenge_status_string="Service-Token receipt from Privacy-CA."
    wait_loop $max_wait p_d_service_token.txt
    if [ $event_file_found == 0 ];then
        LOG_ERROR "$identity_challenge_status_string"
        return 1
    fi
    LOG_INFO "$identity_challenge_status_string"
    event_file_found=0
    cp p_d_service_token.txt \
    $service_provider_location/d_s_service_token.txt
    rm -f p_d_service_token.txt

   return 0
}

process_device_software_state_validation_request() {

    software_state_string="PCR selection list receipt from Service-Provider"
    max_wait=60
    wait_loop $max_wait s_d_pcrlist.txt
    if [ $event_file_found == 0 ];then
        LOG_ERROR "$software_state_string"
        return 1
    fi
    LOG_INFO "$software_state_string"
    event_file_found=0
    pcr_selection=`grep pcr-selection s_d_pcrlist.txt | \
    awk '{print $2}'`
    service_provider_nonce=`grep nonce s_d_pcrlist.txt | \
    awk '{print $2}'`
    rm -f s_d_pcrlist.txt

    tpm2_quote --key-context rsa_ak.ctx --message attestation_quote.dat \
    --signature attestation_quote.signature \
    --qualification "$service_provider_nonce" \
    --pcr-list "$pcr_selection" \
    --pcr pcr.bin -Q

    cp attestation_quote.dat attestation_quote.signature \
    $service_provider_location/.

    return 0
}

process_encrypted_service_data_content() {

    service_data_status_string="Encrypted service-data-content receipt from Service-Provider"
    max_wait=6
    wait_loop $max_wait s_d_service_content.encrypted
    if [ $event_file_found == 0 ];then
        LOG_ERROR "$service_data_status_string"
        return 1
    fi
    LOG_INFO "$service_data_status_string"
    event_file_found=0

    service_data_status_string="Decryption of service-data-content receipt from Service-Provider"
    tpm2 rsadecrypt -c rsa_ak.ctx -o s_d_service_content.decrypted \
    s_d_service_content.encrypted -Q
    if [ $? == 1 ];then
        LOG_ERROR "$service_data_status_string"
        rm -f s_d_service_content.encrypted
        return 1
    fi
    LOG_INFO "$service_data_status_string"

    SERVICE_CONTENT=`cat s_d_service_content.decrypted`
    LOG_INFO "Service-content: \e[5m$SERVICE_CONTENT"
    rm -f s_d_service_content.*

    return 0
}

request_device_service() {

    request_service_status_string="Device anonymous identity challenge."
    process_device_anonymous_identity_challenge
    if [ $? == 1 ];then
        LOG_ERROR "$request_service_status_string"
        return 1
    fi
    LOG_INFO "$request_service_status_string"

    request_service_status_string="Device software state validation"
    process_device_software_state_validation_request
    if [ $? == 1 ];then
        LOG_ERROR "$request_service_status_string"
        return 1
    fi
    LOG_INFO "$request_service_status_string"

    request_service_status_string="Service data content processing"
    process_encrypted_service_data_content
    if [ $? == 1 ];then
        LOG_ERROR "$request_service_status_string"
        return 1
    fi

    return 0
}

tput sc
read -r -p "Demonstration purpose only, not for production. Continue? [y/N] " response
tput rc
tput el
if [[ "$response" =~ ^([yY][eE][sS]|[yY])$ ]]
then
    echo "===================== DEVICE-NODE ====================="
else
    exit
fi


while getopts ":hrt:" opt; do
  case ${opt} in
    h )
      echo "Pass 'r' for registration or 't' for service request"
      ;;
    r )
      device_registration_request=1
      ;;
    t )
      device_service_request=1
      device_service_aik=$OPTARG
      ;;
  esac
done
shift $(( OPTIND - 1 ))

if [ $device_registration_request == 1 ];then
   if [ $device_service_request == 1 ];then
      echo "Specify either 'registration' or 'service' request not both"
      exit 1
   fi
fi

status_string="Device registration request."
if [ $device_registration_request == 1 ];then
   request_device_registration
   if [ $? == 1 ];then
      LOG_ERROR "$status_string"
      exit 1
   fi
   LOG_INFO "$status_string"
fi

status_string="Device service request."
if [ $device_service_request == 1 ];then
   request_device_service
   if [ $? == 1 ];then
      LOG_ERROR "$status_string"
      exit 1
   fi
fi

if [ $device_registration_request == 0 ];then
   if [ $device_service_request == 0 ];then
      echo "Usage: device-node.sh [-h] [-r] [-t AIK.pub]"
      exit 1
   fi
fi

# No errors
exit 0
```

## Service-Provider
```bash
#!/bin/bash

# Fixed location
pca_location="$PWD/../PCA"

# Device Location not fixed
device_location=""

# State
event_file_found=0
device_registration_request=0
device_service_request=0

# Attestation Data
GOLDEN_PCR_SELECTION="sha1:0,1,2+sha256:0,1,2"
GOLDEN_PCR="59bf9091f4cbbd2a8796bfe086a501c57226c42739dcf8ad323e7493ad51e38f"

# Service Data
SERVICE_CONTENT="Hello world!"

wait_loop() {
    counter=1
    until [ $counter -gt $1 ]
    do
       test -f $2
       if [ $? == 0 ];then
          event_file_found=1
          break
       else
          echo -ne "Waiting $1 seconds: $counter"'\r'
       fi
       ((counter++))
       sleep 1
    done
}

LOG_ERROR() {
    errorstring=$1
    echo -e "\033[31mFAIL: \e[97m${errorstring}\e[0m"
}

LOG_INFO() {
    messagestring=$1
    echo -e "\033[93mPASS: \e[97m${messagestring}\e[0m"
}

device_registration() {

    REGISTRATION_TOKEN=`dd if=/dev/urandom bs=1 count=32 status=none | \
    xxd -p -c32`

    device_location=`grep device_location d_s_registration.txt | \
    awk '{print $2}'`
    rm -f d_s_registration.txt

    data_to_privacy_ca="
    device_location: $device_location
    registration_token: $REGISTRATION_TOKEN
    "

    echo "$data_to_privacy_ca" > s_p_registration.txt
    cp s_p_registration.txt $pca_location/.
    rm -f s_p_registration.txt

    # Send privacy-CA information to device
    echo "privacy_ca_location: $pca_location" > s_d_registration.txt
    cp s_d_registration.txt $device_location/.
    rm -f s_d_registration.txt

    # Wait for device_registration_token from device
    registration_status_string="Registration-Token reciept from device."
    wait_loop $max_wait d_s_registration_token.txt
    if [ $event_file_found == 0 ];then
      LOG_ERROR "$registration_status_string"
      return 1
    fi
    LOG_INFO "$registration_status_string"
    event_file_found=0
    test_registration_token=`grep registration_token \
    d_s_registration_token.txt | awk '{print $2}'`
    rm -f d_s_registration_token.txt

    registration_status_string="Registration-Token validation"
    if [ $test_registration_token == $REGISTRATION_TOKEN ];then
      LOG_INFO "$registration_status_string"
      return 0
    else
      LOG_ERROR "$registration_status_string"
      return 1
    fi
}

device_node_identity_challenge() {
    SERVICE_TOKEN=`dd if=/dev/urandom bs=1 count=32 status=none | \
    xxd -p -c32`

    device_location=`grep device_location d_s_service.txt | \
    awk '{print $2}'`
    rm -f d_s_service.txt

    data_to_privacy_ca="
    device_location: $device_location
    service_token: $SERVICE_TOKEN
    "

    echo "$data_to_privacy_ca" > s_p_service.txt
    cp s_p_service.txt $pca_location/.
    rm -f s_p_service.txt

    # Send privacy-CA information to device
    echo "privacy_ca_location: $pca_location" > s_d_service.txt
    cp s_d_service.txt $device_location
    rm -f s_d_service.txt

   identity_challenge_status_string="Aborting service request - AIK not found."
   test -f d_s_service_aik.pub
   if [ $? == 1 ];then
      LOG_ERROR "$identity_challenge_status_string"
      return 1
   else
      cp d_s_service_aik.pub $pca_location/s_p_service_aik.pub
   fi

   identity_challenge_status_string="Service-Token receipt from device."
   wait_loop $max_wait d_s_service_token.txt
   if [ $event_file_found == 0 ];then
     LOG_ERROR "$identity_challenge_status_string"
     return 1
   fi
   LOG_INFO "$identity_challenge_status_string"
   event_file_found=0
   test_service_token=`grep service-token \
   d_s_service_token.txt | awk '{print $2}'`
   rm -f d_s_service_token.txt

   identity_challenge_status_string="Service-Token validation."
   if [ $test_service_token == $SERVICE_TOKEN ];then
     LOG_INFO "$identity_challenge_status_string"
     return 0
   fi
   LOG_ERROR "$identity_challenge_status_string"

   return 1
}

system_software_state_validation() {

   rm -f attestation_quote.dat attestation_quote.signature
   echo "pcr-selection: $GOLDEN_PCR_SELECTION" > s_d_pcrlist.txt
   NONCE=`dd if=/dev/urandom bs=1 count=32 status=none | xxd -p -c32`
   echo "nonce: $NONCE" >> s_d_pcrlist.txt
   cp s_d_pcrlist.txt $device_location/.
   rm -f s_d_pcrlist.txt

   software_status_string="Attestation data receipt from device"
   max_wait=60
   wait_loop $max_wait attestation_quote.dat
   if [ $event_file_found == 0 ];then
      LOG_ERROR "$software_status_string"
      return 1
   fi
   LOG_INFO "$software_status_string"
   event_file_found=0

   software_status_string="Attestation signature receipt from device"
   max_wait=60
   wait_loop $max_wait attestation_quote.signature
   if [ $event_file_found == 0 ];then
      LOG_ERROR "$software_status_string"
      return 1
   fi
   LOG_INFO "$software_status_string"
   event_file_found=0

   software_status_string="Attestation quote signature validation"
   tpm2_checkquote --public d_s_service_aik.pub  --qualification "$NONCE" \
   --message attestation_quote.dat --signature attestation_quote.signature \
   --pcr pcr.bin -Q
   retval=$?
   rm -f attestation_quote.signature
   if [ $retval == 1 ];then
      LOG_ERROR "$software_status_string"
      return 1
   fi
   LOG_INFO "$software_status_string"

   software_status_string="Verification of PCR from quote against golden reference"
   testpcr=`tpm2_print -t TPMS_ATTEST attestation_quote.dat | \
   grep pcrDigest | awk '{print $2}'`
   rm -f attestation_quote.dat
   if [ "$testpcr" == "$GOLDEN_PCR" ];then
      LOG_INFO "$software_status_string"
   else
      LOG_ERROR "$software_status_string"
      echo -e "      \e[97mDevice-PCR: $testpcr\e[0m"
      echo -e "      \e[97mGolden-PCR: $GOLDEN_PCR\e[0m"
      return 1
   fi

   return 0
}

request_device_service() {
   # Start device service registration with device identity challenge
   request_device_service_status_string="Anonymous identity validation by Privacy-CA."
   device_node_identity_challenge
   if [ $? == 1 ];then
      LOG_ERROR "$request_device_service_status_string"
      rm -f d_s_service_aik.pub
      return 1
   fi
   LOG_INFO "$request_device_service_status_string"

   # Check the device software state by getting a device quote
   request_device_service_status_string="Device system software validation."
   system_software_state_validation
   if [ $? == 1 ];then
      LOG_ERROR "$request_device_service_status_string"
      rm -f d_s_service_aik.pub
      return 1
   fi
   LOG_INFO "$request_device_service_status_string"

   # Encrypt service data content and deliver
   echo "$SERVICE_CONTENT" > service-content.plain
    openssl rsautl -encrypt -inkey d_s_service_aik.pub -pubin \
    -in service-content.plain -out s_d_service_content.encrypted

    cp s_d_service_content.encrypted $device_location/.
    rm -f d_s_service_aik.pub
    rm -f s_d_service_content.encrypted
    rm -f service-content.plain
    LOG_INFO "Sending service-content: \e[5m$SERVICE_CONTENT"

   return 0
}

tput sc
read -r -p "Demonstration purpose only, not for production. Continue? [y/N] " response
tput rc
tput el
if [[ "$response" =~ ^([yY][eE][sS]|[yY])$ ]]
then
    echo "===================== SERVICE-PROVIDER ====================="
else
    exit
fi

counter=1
max_wait=60
until [ $counter -gt $max_wait ]
do
   ! test -f d_s_registration.txt
   device_registration_request=$?
   ! test -f d_s_service.txt
   device_service_request=$?

   status_string="Device registration request."
   if [ $device_registration_request == 1 ];then
      device_registration
      if [ $? == 1 ];then
         LOG_ERROR "$status_string"
         exit 1
      fi
      LOG_INFO "$status_string"
      break
   elif [ $device_service_request == 1 ];then
      status_string="Device service request."
      request_device_service
      if [ $? == 1 ];then
         LOG_ERROR "$status_string"
         exit 1
      fi
      LOG_INFO "$status_string"
      break
   else
      echo -ne "Waiting $1 seconds: $counter"'\r'
   fi
   ((counter++))
   sleep 1
done

if [ $device_registration_request == 0 ];then
   if [ $device_service_request == 0 ];then
      LOG_ERROR "Exiting as there are no device requests to process"
      exit 1
   fi
fi

# No errors
exit 0
```


## Privacy-CA

```bash
#!/bin/bash

# Fixed location
service_provider_location="$PWD/../SP"

# Location for node 1, node 2, etc.
device_location=""
registration_token=""

# State
event_file_found=0

wait_loop() {
    counter=1
    until [ $counter -gt $1 ]
    do
       test -f $2
       if [ $? == 0 ];then
          event_file_found=1
          break
       else
          echo -ne "Waiting $1 seconds: $counter"'\r'
       fi
       ((counter++))
       sleep 1
    done
}

LOG_ERROR() {
    errorstring=$1
    echo -e "\033[31mFAIL: \e[97m${errorstring}\e[0m"
}

LOG_INFO() {
    messagestring=$1
    echo -e "\033[93mPASS: \e[97m${messagestring}\e[0m"
}

process_device_registration_request_from_service_provider() {

    device_location=`grep device_location s_p_registration.txt | \
    awk '{print $2}'`
    registration_token=`grep registration_token s_p_registration.txt | \
    awk '{print $2}'`
    rm -f s_p_registration.txt

    return 0
}

credential_challenge() {

    file_size=`stat --printf="%s" rsa_ak.name`
    loaded_key_name=`cat rsa_ak.name | xxd -p -c $file_size`

    echo "this is my secret" > file_input.data
    tpm2_makecredential --tcti none --encryption-key rsa_ek.pub \
    --secret file_input.data --name $loaded_key_name \
    --credential-blob cred.out
    
    cp cred.out $device_location/.

    credential_status_string="Activated credential receipt from device."
    max_wait=60
    wait_loop $max_wait actcred.out
    if [ $event_file_found == 0 ];then
        LOG_ERROR "$credential_status_string"
        return 1
    fi
    LOG_INFO "$credential_status_string"
    event_file_found=0

    diff file_input.data actcred.out
    test=$?
    rm -f rsa_ak.* file_input.data actcred.out cred.out
    credential_status_string="Credential activation challenge."
    if [ $test == 0 ];then
        LOG_INFO "$credential_status_string"
        return 0
    else
        LOG_ERROR "$credential_status_string"
        return 1
    fi
}

process_device_registration_processing_with_device() {

    touch p_d_pca_ready.txt
    cp p_d_pca_ready.txt $device_location/.
    rm -f p_d_pca_ready.txt

    process_registration_status_string="Device-ready acknowledgement receipt from device."
    max_wait=60
    wait_loop $max_wait d_p_device_ready.txt
    if [ $event_file_found == 0 ];then
        LOG_ERROR "$process_registration_status_string"
        return 1
    fi
    LOG_INFO "$process_registration_status_string"
    event_file_found=0
    rm -f d_p_device_ready.txt

    cp $device_location/rsa_ek.pub .
    cp $device_location/rsa_ak.pub .
    cp $device_location/rsa_ak.name .
    LOG_INFO "Received EKcertificate EK and AIK from device"

    credential_challenge
    if [ $? == 1 ];then
        return 1
    fi

    return 0
}

request_device_registration() {

    mkdir -p Registered_EK_Pool

    registration_request_status_string="Device info and registration-token receipt from service-provider."
    process_device_registration_request_from_service_provider
    if [ $? == 1 ];then
        LOG_ERROR "$registration_request_status_string"
        return 1
    fi
    LOG_INFO "$registration_request_status_string"

    registration_request_status_string="Registration-token dispatch to device."
    process_device_registration_processing_with_device
    if [ $? == 1 ];then
        LOG_ERROR "$registration_request_status_string"
        return 1
    else
        LOG_INFO "$registration_request_status_string"
        echo "registration_token: $registration_token" > \
        p_d_registration_token.txt
        cp p_d_registration_token.txt $device_location/.
        rm -f p_d_registration_token.txt
    fi

    mv rsa_ek.pub Registered_EK_Pool/$registration_token
    fdupes --recurse --omitfirst --noprompt --delete --quiet \
    Registered_EK_Pool | grep -q rsa_ek.pub

    return 0
}

request_device_service() {

    device_location=`grep device_location s_p_service.txt | \
    awk '{print $2}'`
    service_token=`grep service_token s_p_service.txt | \
    awk '{print $2}'`
    rm -f s_p_service.txt

    cp s_p_service_aik.pub $device_location/rsa_ak.pub
    rm -f s_p_service_aik.pub
    process_device_registration_processing_with_device
    if [ $? == 1 ];then
        LOG_ERROR "AIK received from service provider is not on the device"
        return 1
    fi

    cp rsa_ek.pub Registered_EK_Pool
    fdupes --recurse --omitfirst --noprompt --delete --quiet \
    Registered_EK_Pool | grep -q rsa_ek.pub
    retval=$?
    rm -f rsa_ek.pub Registered_EK_Pool/rsa_ek.pub
    if [ $retval == 1 ];then
        LOG_ERROR "EK from device does not belong to the registered EK pool"
        return 1
    fi

    echo "service-token: $service_token" > p_d_service_token.txt
    cp p_d_service_token.txt $device_location
    rm -f p_d_service_token.txt

    return 0
}

tput sc
read -r -p "Demonstration purpose only, not for production. Continue? [y/N] " response
tput rc
tput el
if [[ "$response" =~ ^([yY][eE][sS]|[yY])$ ]]
then
    echo "===================== PRIVACY-CA ====================="
else
    exit
fi


device_registration_request=0
device_service_request=0
counter=1
max_wait=60
until [ $counter -gt $max_wait ]
do
   ! test -f s_p_registration.txt
   device_registration_request=$?
   ! test -f s_p_service.txt
   device_service_request=$?

   if [ $device_registration_request == 1 ];then
      status_string="Device registration request."
      request_device_registration
      if [ $? == 1 ];then
        LOG_ERROR "$status_string"
        exit 1
      fi
      LOG_INFO "$status_string"
      break
   elif [ $device_service_request == 1 ];then
      status_string="Device service request received."
      request_device_service
      if [ $? == 1 ];then
        LOG_ERROR "$status_string"
        exit 1
      fi
      LOG_INFO "$status_string"
      break
   else
      echo -ne "Waiting $1 seconds: $counter"'\r'
   fi
   ((counter++))
   sleep 1
done

if [ $device_registration_request == 0 ];then
   if [ $device_service_request == 0 ];then
      LOG_ERROR "Exiting as there are no service provider requests to process."
      exit 1
   fi
fi

# No errors
exit 0
```

# FAQ

1. ***If the EK or AIK had already been generated but the public key file isn't
   available, is there a way to generate the public key?***

   It is possible to recreate public key from an already created EK or AIK key.
   There are two possible forms for a TPM object to reside on the TPM. Both of
   these eventually translate as a TPM handle number that can be invoked by
   specific TPM commands.
   
   Persistent-handles: As the term implies, these reside in the TPM and can be
   invoked using TPM persistent-handle at location range 81xxxxxxx. Since these
   reside on the TPM NV, they survive TPM resets and restarts.
   
   Transient handles, on the contrary, do not reside on the TPM NV. However, they
   are still protected objects that need additional steps of loading and validating
   integrity prior to their use. Such an object has 3 important parts before it is
   translated into a usable handle.
   a. The wrapped sensitive portion of the key object (termed as private in the
   tpm2-tools).
   b. The non-sensitive portion which includes key attributes, authorization policy
   digest, etc (termed as public in the tpm2-tools).
   c. A context blob that is generated when such an object is successfully
   created-->loaded-->offloaded from the TPM. Once ready to use the
   object, the context blob is referenced in the tpm2 commands that then get
   assigned a transient handle for use just like the persistent handles but at
   location range 80xxxxxx.
   
   In either of the cases above, a tool called [tpm2_readpublic]([tpm2_readpublic](https://github.com/tpm2-software/tpm2-tools/blob/master/man/tpm2_readpublic.1.md)
   can be used to view and or dump a public portion in tss or pem format by passing
   the persistent handle or the context file as an input. In fact, this tool has
   been used in the device-node.sh scripts in this tutorial as well to generate a
   pem formatted file.

2. ***Why is tpm2_createak tool not used to create the AIK in the demo scripts?***

   In our demo example we intend to have an AIK with following properties:
   a. It will have to be validated for anonymous identity relationship with EK.
   b. It has to be a signing key for it to be used to sign an attestation quote.
   c. It has to be usable as RSA encrypt/ decrypt key.

   The combination of the above properties is not the default attributes chosen
   in the tpm2_createak tool. Specifically, the key generated with tpm2_createak
   cannot be used as a decryption key.
   
   Note that the authorization for using the endorsement key which is the parent
   of the attestation identity key needs to be satisfied to be able to create
   the AIK and is satisfied through a policy session using a policy
   "policysecret" to reference the authorization of the endorsement hierarchy.

# Author
Imran Desai
