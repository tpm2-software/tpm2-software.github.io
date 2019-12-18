---
layout: post
title:  "Remote Attestation"
date:   2019-12-18 13:24:05 +0100
categories: tpm2-tss getting-started
tags: attestation
---

# Remote Attestation Best Known Methods

## Introduction
There are many cases where it is crucial to verify the correctness of software
and hardware configuration of a computer system. For instance, to improve the
security of a corporate network, companies need to ensure that the systems that
are allowed to connect to the corporate network are legitimate, and that the
software that runs on the system is in a valid, known state. There are many
different methods to ensure the above. This document describes a method that
uses a Trusted Platform Module (TPM). TPM device can be used to validate a system
integrity by implementing an attestation protocol. Trusted Computing Group (TCG)
published a Trusted Attestation Protocol Information Model (TAP), which
can be found at [1]. The TCG specification describes roles and message flows.
The specific data format and binding to a protocol require additional “binding”
specifications. The two main roles in this scenario are Verifier and Attester.
A Verifier sends a challenge query to a system that selects the scope of
attestation. The Attester collects attestation evidence (within the expected
scope) that is to be verified (Attester). The Attester typically describes its
boot state, current configuration and identity. Machine identities can establish
supply chain provenance if the machine identity credentials are provisioned
during manufacturing. One of the identities is known as the Endorsement Key
(EK) certificate. The EK key may be used to provision attestation identities
known as an Attestation Key certificate. Alternatively, the manufacturer may
pre-provision an Attestation Key known as an Initial Attestation Key (IAK)
which avoids having to provision it at onboarding or deployment.
The Attestation Key is used to sign attestation evidence that is conveyed to the
Verifier. After receiving the response, the Verifier can cryptographically check
if the evidence satisfies the Verifier’s trust appraisal policy and if the
device originated from an approved manufacturer. The manufacturer embedded EK
and/or IAK and associated certificate path is what establishes device provenance
to a vendor.

## Key Provisioning
A platform with a TPM root-of-trust may be pre-provisioned with attestation and
identities or they may be provisioned as part of device onboarding and deployment.
Pre-provisioned keys and credentials are typically referred to as "Initial"
while credentials provisioned at onboarding typically are referred to as "local".
However, a TPM pre-provisioned with only an EK may provision “Initial” keys at
different stages in the supply chain including as part of device onboarding.
Hence, TPM supports provisioning that happens over different stages.
There are keys that are instantiated during manufacturing time by the TPM Vendor,
other keys need to be instantiated during system assembly by the OEM, and other
keys are instantiated by the system owner or user.  TCG Provisioning Guidance [2]
specification provides recommendations for key provisioning.
Usually, TPM vendors provision the TPM device with a Primary Endorsement Key (PEK),
and generate a certificate for that key (EK cert) in x.509 format at the time of
manufacture. The EK Certificate contains the public part of the PEK, as well as
other useful fields, such as TPM manufacturer name, part model, part version,
key id, etc. This information can be used to uniquely identify the TPM and if
the device OEM securely attaches a TPM to the device it can be used as a device
identifier. The EK certificate may be stored in the TPM Non Volatile Memory (NVM),
where it can be made available to client software or it could be made available
online via the OEM’s website.
The EK certificate is issued (cryptographically signed) by the TPM vendor’s CA
and verifiers are assumed to trust the vendor’s root certificate. It is important
to note that the private part of the EK key is expected to never be exposed
outside of the TPM hardware. This ensures an attack TPM cannot easily masquerade
as coming from a trusted vendor. To improve interoperability between vendors and
adopters TCG also provides recommendations and guidelines for EK Credential
values, which can be found in the TCG EK Credential Profile [3].
The Vendor may also instantiate an IDevID Key, that is used to uniquely identify
the TPM device. At a later point the user may create more keys in the Endorsement
hierarchy, that will be protected (wrapped) by the Primary EK. Tpm2_createek
command can be used to generate a new EK. Note that Tpm2_createek will create
the same key when invoked with the same template and the same Endorsement
Primary Seed. Next, when the TPM is integrated into a system, the OEM generates
a Primary Attestation Key (PAK) that is also stored in the TPM’s Non Volatile
Memory and Attestation Key certificate, which references the Primary EK stored
in the TPM. The OEM may also generate Initial Attestation Key (IAK) and IAK
Certificate, which along with IDevID Key are used to platform identification,
and signs it with the OEM’s preferred CA. Provisioning of other keys, such as
Primary Storage Key (PSK), that are not used in attestation process are out of
scope of this document. These keys can be generated on demand by the owner/user,
don’t need to be signed by third party (Vendor, OEM), and no certificates are
needed for them. Diagram 1 depictures the steps how a new TPM device is
provisioned when it is installed in a system.

![Diagram 1:  Attestation key provisioning](/images/diag1.png)

## Attestation Procedure
Before a system can be attested the owner needs to obtain the public key of the
TPM Vendor generated Endorsement Key and OEM generated Attestation Key along with
the appropriate certificates. Certificates can be obtained by reading the TPM
NonVolatile memory at a predefined location, or by obtaining them from Vendor/OEM websites.
The Verifier normally stores these keys and certificates in a local database,
usually integrated with the corporate PKI infrastructure.
Another object typically stored in the Verifier’s local database is a policy that
identifies an expected initial state of the device as defined by the software installed
on the Attester system. The various vendor(s) of boot software contribute digests
of the software to Verifiers to be used as a reference for comparing digests to
attestation evidence.
Boot software needs to be measured during platform setup stage, and include
measurements (digests) for the bootloader, OS, drivers, and appropriate user
space applications. The measurements are stored in a local database and are used later,
during attestation, to make determination if a system is in a good state.
The user may also create a policy that will make the Attestation Key bound,
or sealed to the initial state of the system.
For a data center deployments, where many nodes run on similar hardware configuration
and run the same software, this can be measured for a single platform and then used
as base measurement for different systems with similar hardware and software configuration.
Another piece of functionality that needs to be enabled on the to be attested
systems is measured boot. If it is enabled the system software will measure the
images of every software module, on different levels during the boot process,
record the measured values in the Platform Configuration Registers (PCRs) within
the TPM, and created a system Event Log with a separate entry for every software
image measured. First ‘Secure Boot’ option needs to be enabled in the BIOS configuration,
and it also needs to be supported by the bootloader and the Operation System.
Most of the recent UEFI based platforms, bootloaders and Operating Systems do
support measured/secure boot.
The attestation operation is initialized by the verifier.
It can be started on a time based interval or in response to a network resource
access request. The verifier requests the attester’s AK and EK certificates,
validate them against the corporate Root CA, and if they are valid it creates a
challenge including key id, PCR selection, and random nonce. The attestor loads
the Endorsement Key (EK) and Attestation Key (AK), if  they are not already loaded,
and issues a TPM2_Quote command to the TPM, referencing the Attestation Key.
TPM will only be able to use the key if the given PCRs will be in a valid state,
defined at the time when the PCR policy is created. After that TPM creates and
signs an instance of TPMS_ATTEST structure as defined by [5] and sends it back
to the verifier. The Verifier uses the information included in the TPMS_ATTEST
structure to perform appraisal by validating the public part of the AK and comparing
the software measurement logs (Event Log) against the good-know-state of the system
software configuration stored in the local database. Diagram 2 shows the
attestation challenge flow.

![Diagram 2:  Attestation challenge](/images/diag2.png)


The verify can compare every entry in the event log stored in the database with
the corresponding entry in the TPMS_ATTEST structure to verify the state of
bootloader, Operating System, user applications, and make arbitrary decisions
based on the results of the verification process. The Diagram 3 shows the
verification step performed by the verifier.

![Diagram 3: Verification of the quote](/images/diag3.png)


## Field upgrade and disaster recovery
When planning the implementation of the attestation in the corporate infrastructure
it is also important to plan for platform firmware upgrades, operating system,
or user space application updates. It is also important to plan recovery procedures
in case of major failures, 0-days vulnerability disclosures, etc. Just as any other
piece of corporate infrastructure TPM can become a target for an attack.
Recently published TPM fail [6] paper showed that even some hardware TPMs can
be vulnerable to timing attacks. In case of any of the mentioned situations it might
be required to re-instantiate the affected TPM keys, and even the TPM modules itself,
measure the system hardware and software configuration state and make appropriate
adjustments in the local database, in which the measurements, keys, and key
certificates are stored. TPM key upgrade and recovery should be part of an
enterprise key management strategy. TPM device can generate and store keys that
are non-migratable, and EK is an example of a non-migratable key.
In this case it is recommended to follow the TPM vendor’s recommendations and
procedures for managing the keys. When making a decision on what TPM vendor to
choose it is important to check beforehand what level of support is provided,
and what are the recommended recovery procedures [7].


## References
1. [TCG, Trusted Attestation Protocol (TAP)](https://trustedcomputinggroup.org/wp-content/uploads/TNC_TAP_Information_Model_v1.00_r0.29A_publicreview.pdf)
2. [TCG, TPM v2.0 Provisioning Guidance](https://trustedcomputinggroup.org/wp-content/uploads/TCG-TPM-v2.0-Provisioning-Guidance-Published-v1r1.pdf)
3. [TCG, EK Credential Profile](https://trustedcomputinggroup.org/wp-content/uploads/TCG_IWG_Credential_Profile_EK_V2.1_R13.pdf)
4. [TCG, TPM2.0 Library Specification, part 1- Architecture](https://trustedcomputinggroup.org/wp-content/uploads/TPM-Rev-2.0-Part-1-Architecture-01.38.pdf)
5. [TCG, TPM2.0 Library Specification, part 2- Structures](https://trustedcomputinggroup.org/wp-content/uploads/TPM-Rev-2.0-Part-2-Structures-01.38.pdf)
6. [Timing and Lattice Attacks on the TPM](https://tpm.fail/tpmfail.pdf)
7. [Infineon, TPM Key backup and recovery](https://www.infineon.com/dgdl/Infineon-TPM_Key_Backup_and_Recovery-AP-v01_00-EN.pdf?fileId=db3a304412b407950112b41656d7203a)
