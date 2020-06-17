---
layout: page
title: Versions
permalink: /versions/
---

<script>
  ((window.gitter = {}).chat = {}).options = {
    room: 'tpm2-software/community'
  };
</script>
<script src="https://sidecar.gitter.im/dist/sidecar.v1.js" async defer></script>

# Packaging

| **tpm2-tss** | **tpm2-tools** | **tpm2-abrmd** | **tpm2-tss-engine** | **tpm2-totp** | **tpm2-pkcs11** |
| :---: | :---: | :---: | :---: | :---: | :---: |
| [![Packaging status](https://repology.org/badge/vertical-allrepos/tpm2-tss.svg)](https://repology.org/project/tpm2-tss/versions) [![Packaging status](https://repology.org/badge/vertical-allrepos/tpm2-0-tss.svg)](https://repology.org/project/tpm2-0-tss/versions) | [![Packaging status](https://repology.org/badge/vertical-allrepos/tpm2-tools.svg)](https://repology.org/project/tpm2-tools/versions) | [![Packaging status](https://repology.org/badge/vertical-allrepos/tpm2-abrmd.svg)](https://repology.org/project/tpm2-abrmd/versions) | [![Packaging status](https://repology.org/badge/vertical-allrepos/tpm2-tss-engine.svg)](https://repology.org/project/tpm2-tss-engine/versions) | [![Packaging status](https://repology.org/badge/vertical-allrepos/tpm2-totp.svg)](https://repology.org/project/tpm2-totp/versions) | [![Packaging status](https://repology.org/badge/vertical-allrepos/tpm2-pkcs11.svg)](https://repology.org/project/tpm2-pkcs11/versions) |

# tpm2-tools

tpm2-tools has an intricate dependency on the tpm2-tss project and the tpm2-abrmd project. so below is a list tracking the dependencies of each release/tag of tpm2-tools.

**NOTE: tpm2-abrmd is an optional but suggested component**

| tpm2-tools version | tpm2-tss version | tpm2-abrmd version|
|--------------------|------------------|-------------------|
|[master](https://github.com/01org/tpm2-tools)|[master](https://github.com/01org/tpm2-tss)|[master](https://github.com/01org/tpm2-abrmd)|
|[4.2](https://github.com/tpm2-software/tpm2-tools/releases/tag/4.2)|[>=2.4.0](https://github.com/tpm2-software/tpm2-tss/releases/tag/2.4.0)|[>=2.3.1](https://github.com/tpm2-software/tpm2-abrmd/releases/tag/2.3.1)
|[4.1.1](https://github.com/tpm2-software/tpm2-tools/releases/tag/4.1.1)|[>=2.3.1](https://github.com/tpm2-software/tpm2-tss/releases/tag/2.3.1)|[>=2.3.0](https://github.com/tpm2-software/tpm2-abrmd/releases/tag/2.3.0)
|[4.1](https://github.com/tpm2-software/tpm2-tools/releases/tag/4.1)|[>=2.3.1](https://github.com/tpm2-software/tpm2-tss/releases/tag/2.3.1)|[>=2.3.0](https://github.com/tpm2-software/tpm2-abrmd/releases/tag/2.3.0)
|[4.0](https://github.com/tpm2-software/tpm2-tools/releases/tag/4.0)|[>=2.3.1](https://github.com/tpm2-software/tpm2-tss/releases/tag/2.3.1)|[>=2.2.0](https://github.com/tpm2-software/tpm2-abrmd/releases/tag/2.2.0)
|[3.2.0](https://github.com/tpm2-software/tpm2-tools/releases/tag/3.2.0-rc0)|[2.0.0](https://github.com/tpm2-software/tpm2-tss/releases/tag/2.0.0)|[2.0.0](https://github.com/tpm2-software/tpm2-abrmd/releases/tag/2.0.0)
|[3.1.0](https://github.com/tpm2-software/tpm2-tools/releases/tag/3.1.0)|[2.0.0](https://github.com/tpm2-software/tpm2-tss/releases/tag/2.0.0)|[2.0.0](https://github.com/tpm2-software/tpm2-abrmd/releases/tag/2.0.0)
|[3.0.2](https://github.com/intel/tpm2-tools/releases/tag/3.0.2)|[1.3.0](https://github.com/intel/tpm2-tss/releases/tag/1.3.0)|[1.2.0](https://github.com/intel/tpm2-abrmd/releases/tag/1.2.0)|
|[3.0.1](https://github.com/intel/tpm2-tools/releases/tag/3.0.1)|[1.3.0](https://github.com/intel/tpm2-tss/releases/tag/1.3.0)|[1.2.0](https://github.com/intel/tpm2-abrmd/releases/tag/1.2.0)|
|[3.0](https://github.com/intel/tpm2-tools/releases/tag/3.0)|[1.3.0](https://github.com/intel/tpm2-tss/releases/tag/1.3.0)|[1.2.0](https://github.com/intel/tpm2-abrmd/releases/tag/1.2.0)|
|[2.1.0](https://github.com/01org/tpm2-tools/releases/tag/2.1.0)|[1.2.0](https://github.com/01org/tpm2-tss/releases/tag/1.2.0)|[1.1.1](https://github.com/01org/tpm2-abrmd/releases/tag/1.1.1)|
|[df751ae](https://github.com/01org/tpm2.0-tools/tree/df751ae5bea0bb057c9ee4cb0c1176c48ff68492)(master)|[1.1.0](https://github.com/01org/TPM2.0-TSS/releases/tag/1.1.0)|[1.0.0](https://github.com/01org/tpm2-abrmd/releases/tag/1.0.0)|
|[v2.0.0](https://github.com/01org/tpm2.0-tools/releases/tag/2.0.0)|[1.0](https://github.com/01org/TPM2.0-TSS/releases/tag/1.0)|old resourcemgr|
|[v1.1.0](https://github.com/01org/tpm2.0-tools/releases/tag/v1.1.0)|[1.0](https://github.com/01org/TPM2.0-TSS/releases/tag/1.0)|old resourcemgr|
|[v1.1-beta_1](https://github.com/01org/tpm2.0-tools/releases/tag/v1.1-beta_1)|[1.0-beta_1](https://github.com/01org/TPM2.0-TSS/releases/tag/1.0-beta_1)|old resourcemgr|
|[v1.1-beta_0](https://github.com/01org/tpm2.0-tools/releases/tag/v1.1-beta_0)|[v1.0-beta_0](https://github.com/01org/TPM2.0-TSS/releases/tag/v1.0-beta_0)|old resourcemgr|
|[14a7ff5](https://github.com/01org/tpm2.0-tools/tree/14a7ff527bc0411c215bd9d575f2866e1f2e71cf)|[210b770](https://github.com/01org/TPM2.0-TSS/tree/210b770c1dff47b11be623e1d1e7ffb02298fca5)|old resourcemgr|
|[4b4cbea](https://github.com/01org/tpm2.0-tools/tree/4b4cbeafe30430f42826592dee2abafec818385f)|[d4f23cc](https://github.com/01org/TPM2.0-TSS/tree/d4f23cc25c4c0fb66dd36897d2fad8e1e37c6443)|old resourcemgr|
|[e8150e4](https://github.com/01org/tpm2.0-tools/tree/e8150e48dd47f761dff10583631b2a0a30ee4d90)|[60ec042](https://github.com/01org/TPM2.0-TSS/tree/60ec04237b5344666435e129bd85f7496a6a9985)|old resourcemgr|
|[84d5f26](https://github.com/01org/tpm2.0-tools/tree/84d5f262f281556c57f7ec2fba06eda3acadd26c)|[371fdbc](https://github.com/01org/TPM2.0-TSS/tree/371fdbc638c55b9ac8a0eaec9375dbca0412861c)|old resourcemgr|
|[v1.0.1](https://github.com/01org/tpm2.0-tools/releases/tag/v1.0.1)|[1.0-alpha_0](https://github.com/01org/TPM2.0-TSS/releases/tag/1.0-alpha_0)|old resourcemgr|
