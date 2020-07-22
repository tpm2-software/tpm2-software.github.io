# Basics
The Feature API (FAPI) contains the concept of cryptographic profiles.
These define a combination of cryptographic algorithms and properties that shall be used.
On a typical installation they are located under `/etc/tpm2-tss/fapi-profiles`.
One of these profiles is selected using the fapi config file; key `profile_name` in `/etc/tpm2-tss/fapi-config.json`.

When calling `Fapi_Profision()` the keystore of Fapi is initialized, the `SRKs` are created and the `EK` is checked based on the currently selected profile. From that point on, the parameters of the profile are used for all relevant operations.

# Multiple Profiles
The FAPI includes a concept for cryptographic transition. The purpose is that if the crypto policy changes one can transition to the new policy whilst retaining access to old key material. As such, one can change the `profile_name` in the `fapi-config.json` and call `Fapi_Provision()` in order to populate the new keystore.

Note that you have to make sure that the persistent handles between the old and the new profile in `/etc/tpm2-tss/fapi-profiles` do not collide. On a standard installation they both are set to `0x81000001`.

Once the new profile is provisioned, the old profile can be access by prefixing all keypaths accordings, e.g. `/P_OLDPROFILE/HS/SRK/...`

# Asymmetric Encryption
At the time of this writing only the RSA crypto profiles support asymmetric encryption and decryption. The reason is that even though the Elliptic Curve Integrated Encryption Scheme (ECIES) seems to be well established, there exists no one standard for the exact semantics and data encoding formats. See also OpenSSL for a reference: https://github.com/openssl/openssl/issues/9314#issuecomment-508724551

Thus in order to utilize the functions `Fapi_Encrypt()` and `Fapi_Decrypt()` the RSA profile must be selected. Note that multiple profiles can be provisioned at once, as mentioned above.

# Updates
This page may be updated in the future with more information.

# Author
Andreas Fuchs @Fraunhofer SIT
