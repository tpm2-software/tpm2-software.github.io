---
layout: page
title: Software
permalink: /software/
---

<script>
  ((window.gitter = {}).chat = {}).options = {
    room: 'tpm2-software/community'
  };
</script>
<script src="https://sidecar.gitter.im/dist/sidecar.v1.js" async defer></script>

# TPM 2.0 software
- [tpm2-tss](https://github.com/tpm2-software/tpm2-tss) [![Documentation Status](https://readthedocs.org/projects/tpm2-tss/badge/?version=latest)](https://tpm2-tss.readthedocs.io/en/latest/?badge=latest) [![latest packaged version(s)](https://repology.org/badge/latest-versions/tpm2-tss.svg)](https://repology.org/project/tpm2-tss/versions)  [![Packaging status](https://repology.org/badge/tiny-repos/tpm2-tss.svg)](https://repology.org/project/tpm2-tss/versions)
- [TPM2-Tools](https://github.com/tpm2-software/tpm2-tools) [![latest packaged version(s)](https://repology.org/badge/latest-versions/tpm2-tools.svg)](https://repology.org/project/tpm2-tools/versions) [![Packaging status](https://repology.org/badge/tiny-repos/tpm2-tools.svg)](https://repology.org/project/tpm2-tools/versions)
- [PKCS11](https://github.com/tpm2-software/tpm2-pkcs11) [![latest packaged version(s)](https://repology.org/badge/latest-versions/tpm2-pkcs11.svg)](https://repology.org/project/tpm2-pkcs11/versions) [![Packaging status](https://repology.org/badge/tiny-repos/tpm2-pkcs11.svg)](https://repology.org/project/tpm2-pkcs11/versions)
- [OpenSSL 1.x](https://github.com/tpm2-software/tpm2-tss-engine) [![latest packaged version(s)](https://repology.org/badge/latest-versions/tpm2-tss-engine.svg)](https://repology.org/project/tpm2-tss-engine/versions) [![Packaging status](https://repology.org/badge/tiny-repos/tpm2-tss-engine.svg)](https://repology.org/project/tpm2-tss-engine/versions)
- [OpenSSL 3.x](https://github.com/tpm2-software/tpm2-openssl) [![latest packaged version(s)](https://repology.org/badge/latest-versions/tpm2-openssl.svg)](https://repology.org/project/tpm2-openssl/versions) [![Packaging status](https://repology.org/badge/tiny-repos/tpm2-openssl.svg)](https://repology.org/project/tpm2-openssl/versions)
- [TPM2-TOTP](https://github.com/tpm2-software/tpm2-totp) [![latest packaged version(s)](https://repology.org/badge/latest-versions/tpm2-totp.svg)](https://repology.org/project/tpm2-totp/versions) [![Packaging status](https://repology.org/badge/tiny-repos/tpm2-totp.svg)](https://repology.org/project/tpm2-totp/versions)
- [ESAPI Rust Wrapper](https://crates.io/crates/tss-esapi) [Docs](https://docs.rs/tss-esapi/1.0.1/tss_esapi/)
- [tpm2-pytss](https://github.com/tpm2-software/tpm2-pytss) [![PyPI version](https://img.shields.io/pypi/v/tpm2-pytss.svg)](https://pypi.org/project/tpm2-pytss)
- [TPM-JS](https://google.github.io/tpm-js/)

# Software with direct TPM 2.0 support
- [OpenConnect](http://git.infradead.org/users/dwmw2/openconnect.git/) [Docs](http://www.infradead.org/openconnect/tpm.html)
- [(systemd-)cryptsetup/LUKS](https://0pointer.net/blog/unlocking-luks2-volumes-with-tpm2-fido2-pkcs11-security-hardware-on-systemd-248.html)
- [StrongSwan](https://wiki.strongswan.org/projects/strongswan/wiki/TPMPlugin)
- [Clevis](https://github.com/latchset/clevis) ([Howto](https://blog.dowhile0.org/2017/10/18/automatic-luks-volumes-unlocking-using-a-tpm2-chip/))
- [LVFS / fwupd](https://fwupd.org/): [Post1](https://blogs.gnome.org/hughsie/2018/12/14/firmware-attestation/), [Post2](https://blogs.gnome.org/hughsie/2019/04/10/using-a-client-certificate-to-set-the-attestation-checksum/)
- [libsecret/gnome-keyring](https://gitlab.gnome.org/Teams/Engagement/gsoc-2021/-/issues/13)

# Software with indirect TPM 2.0 support
- NGINX via [OpenSSL tpm2-tss-egnine](https://github.com/tpm2-software/tpm2-tss-engine) [Demo](https://youtu.be/NFQ22SBlejk?t=604)
- SSH via [tpm2-PKCS11](https://github.com/tpm2-software/tpm2-pkcs11) [Demo](https://youtu.be/NFQ22SBlejk?t=944)
- GIT via SSH and [tpm2-PKCS11](https://github.com/tpm2-software/tpm2-pkcs11) [Demo](https://youtu.be/NFQ22SBlejk?t=944)
- TODO (add links to demos): Firefox, Chromium, Thunderbird, Evolution, JDK-Keystore, wpa_supplicant, GNU-TLS (all via tpm2-pkcs11)

# Ideas for adding TPM 2.0 support
- OpenVPN
- WireGuard
- Tinc
- NetworkManager/wpa_supplicant 802.1X
- KDE wallet
- GNU-TLS
- certbot (to create Certs with PKCS11 support)
- WebCrypto (Firefox, WebKit, Chromium, epiphany)
- WebAuthn ([Firefox](https://github.com/mozilla/gecko-dev/tree/master/dom/webauthn), WebKit, Chromium)
- OpenSSH HostKey ((non-)PKCS11), ClientKey ((non-)PKCS11)
- Wireshark TPM Cmd/Rsp/Buffer with TCTI-PCAP module or /dev/tpmrm0 sniffing (partial, TPM-Headers only yet)
- mbed-crypto / mbed-tls
- OpenJDK keystore
- Firefox/Thunderbird/Chromium/epiphany password managers: Epiphany via gnome-keyring ?
- systemd-journald signing
- systemd-networkd 802.1x
- empathy/telepathy jabber (via PKCS11?)
- GnuPG (also leads to git tag and release signing)
- Telegram desktop

Please feel free to also add notes to this list wrt means of integration, e.g. if a project could be enabled using tpm2-pkcs11 and p11-kit because it already provides a pkcs11 interfaces for authentication.
