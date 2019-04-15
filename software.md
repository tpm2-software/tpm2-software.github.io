---
layout: page
title: Software
permalink: /software/
---

# Software with TPM 2.0 support
- [TPM2-Tools](https://github.com/tpm2-software/tpm2-tools)
- [PKCS11](https://github.com/tpm2-software/tpm2-pkcs11)
- [OpenSSL](https://github.com/tpm2-software/tpm2-tss-engine)
- [OpenConnect](http://git.infradead.org/users/dwmw2/openconnect.git/) [Docs](http://www.infradead.org/openconnect/tpm.html)
- [cryptsetup/LUKS](https://gitlab.com/cryptsetup/cryptsetup/merge_requests/51)
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
- WebCrypto (Firefox, WebKit, Chromium)
- WebAuthn ([Firefox](https://github.com/mozilla/gecko-dev/tree/master/dom/webauthn), WebKit, Chromium)
- OpenSSH HostKey ((non-)PKCS11), ClientKey ((non-)PKCS11)
- Wireshark TPM Cmd/Rsp/Buffer with TCTI-PCAP module or /dev/tpmrm0 sniffing (partial, TPM-Headers only yet)

