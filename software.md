---
layout: page
title: Software
permalink: /software/
---

# Software with TPM 2.0 support
- [tpm2-tss](https://github.com/tpm2-software/tpm2-tss) [![latest packaged version(s)](https://repology.org/badge/latest-versions/tpm2-tss.svg)](https://repology.org/project/tpm2-tss/versions)  [![Packaging status](https://repology.org/badge/tiny-repos/tpm2-tss.svg)](https://repology.org/project/tpm2-tss/versions)
- [TPM2-Tools](https://github.com/tpm2-software/tpm2-tools) [![latest packaged version(s)](https://repology.org/badge/latest-versions/tpm2-tools.svg)](https://repology.org/project/tpm2-tools/versions) [![Packaging status](https://repology.org/badge/tiny-repos/tpm2-tools.svg)](https://repology.org/project/tpm2-tools/versions)
- [PKCS11](https://github.com/tpm2-software/tpm2-pkcs11) (wip)
- [OpenSSL](https://github.com/tpm2-software/tpm2-tss-engine) [![latest packaged version(s)](https://repology.org/badge/latest-versions/tpm2-tss-engine.svg)](https://repology.org/project/tpm2-tss-engine/versions) [![Packaging status](https://repology.org/badge/tiny-repos/tpm2-tss-engine.svg)](https://repology.org/project/tpm2-tss-engine/versions)
- [OpenConnect](http://git.infradead.org/users/dwmw2/openconnect.git/) [Docs](http://www.infradead.org/openconnect/tpm.html)
- [cryptsetup/LUKS](https://gitlab.com/cryptsetup/cryptsetup/merge_requests/51) (wip)
- [StrongSwan](https://wiki.strongswan.org/projects/strongswan/wiki/TPMPlugin)
- [Clevis](https://github.com/latchset/clevis) ([Howto](https://blog.dowhile0.org/2017/10/18/automatic-luks-volumes-unlocking-using-a-tpm2-chip/))
- [TPM2-TOTP](https://github.com/tpm2-software/tpm2-totp)
- [LVFS / fwupd](https://fwupd.org/): [Post1](https://blogs.gnome.org/hughsie/2018/12/14/firmware-attestation/), [Post2](https://blogs.gnome.org/hughsie/2019/04/10/using-a-client-certificate-to-set-the-attestation-checksum/)

# Projects requiring TPM 2.0 support
- OpenVPN
- WireGuard
- Tinc
- NetworkManager/wpa_supplicant 802.1X
- gnome-keyring
- KDE wallet
- GNU-TLS
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
- [GNOME Keyring](https://wiki.gnome.org/Projects/GnomeKeyring/SecurityFAQ#I_have_a_TPM_.28Trusted_Platform_Module.29_chip_on_my_machine._Can_I_use_it_to_protect_my_passwords.3F)

Please feel free to also add notes to this list wrt means of integration, e.g. if a project could be enabled using tpm2-pkcs11 and p11-kit because it already provides a pkcs11 interfaces for authentication.
