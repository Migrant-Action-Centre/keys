# Published certificates

Public certificate material for the Migrant Action Centre.

Served at <https://keys.migrantaction.ca>.

The web key directory (WKD), for automatic machine lookup, is a **separate repo**: `Migrant-Action-Centre/pgp` → `openpgpkey.migrantaction.ca`. WKD is deployed using the advanced method.

**WKD publishes a subset of this register.** It cannot carry multiple keys for the same address and does not carry S/MIME, and it serves exactly one certificate per address, so it holds the v4 certificates only. Anything it serves should match a certificate below. If it does not, one of the two is out of date and neither should be used until that is settled.

## Migrant Action Centre: trust anchor

Certification only. No email address or encryption subkey: you cannot encrypt to it. It exists to sign other keys so a signature on someone's certificate traces back to the organization. However, because it has no email address, it cannot be published on the WKD.

|             | v4                                         | v6                                                                 |
| ----------- | ------------------------------------------ | ------------------------------------------------------------------ |
| Fingerprint | `4383A9D6A304C721EED4742B4A08EFC6E8832FBB` | `3DFDDC0FDA8ACF1B84C9D4C53784CA135B7FEC099ED50CD16F9A5B82EBABB186` |
| Algorithm   | EdDSA (ed25519)                            | ML-DSA-87+Ed448                                                    |
| Created     | 2026-06-09                                 | 2026-07-22                                                         |
| Expires     | 2031-06-08                                 | 2029-07-21                                                         |
| File        | `pgp/mac-org-v4.asc`                       | `pgp/mac-org-v6.asc`                                               |

## Adi (adi@migrantaction.ca)

| | v4 | v6 |
|---|---|---|
| Fingerprint | `B16FE02FECC50042523E1CE315986AC8EBD7AFDA` | `ED3698D8741DE9C416E27B14E9AB5EAD0AE3223EC97523567C42A782D381C57C` |
| Algorithm | EdDSA (ed25519) / ECDH (cv25519) | ML-DSA-65+Ed25519 / ML-KEM-768+X25519 |
| Created | 2026-06-09 | 2026-07-25 |
| Expires | 2031-06-08 primary, **subkeys 2028-06-08** | 2028-07-25 |
| File | `pgp/adi-v4.asc` | `pgp/adi-v6.asc` |
| In WKD | yes, `hu/sxwx66sba9uicaxbcfjkmz8b9h5k8aky` | no, deliberately |

Both are certified by the matching-version MAC anchor: v4 by `4383A9D6`, v6 by `3DFDDC0F`.

### S/MIME

|                 |                                                                                                   |
| --------------- | ------------------------------------------------------------------------------------------------- |
| SHA-256         | `71:8C:91:E3:30:43:04:B6:78:9E:2C:61:51:D2:C6:25:E9:F7:B3:79:6D:E9:A6:44:05:86:7F:3F:03:6E:03:BB` |
| Serial          | `503E9A60B1A6A359455A241231AB16A4`                                                                |
| Issuer          | SSL.com Client Certificate Intermediate CA RSA R2                                                 |
| Validation      | Sponsor-validated (policy OID `2.23.140.1.5.3.2`)                                                 |
| Verified entity | Migrant Action Centre Inc., organizationIdentifier `NTRCA-97082`                                  |
| Key             | RSA 3072-bit                                                                                      |
| Valid           | 2026-03-28 to **2027-03-20**                                                                      |
| File            | `smime/adi.crt` (PEM: leaf + SSL.com intermediate; root omitted, already in trust stores)         |

Sponsor-validated means the CA verified both the individual and the affiliation to MAC as a registered entity. Read from the policy OID, not taken on trust.

## Renewal watchlist

| Date | What |
|---|---|
| 2027-03-20 | Adi S/MIME expires |
| 2028-06-08 | Adi v4 **subkeys** expire. Practical end of life, not 2031 |
| 2028-07-25 | Adi v6 expires |
| 2029-07-21 | MAC v6 anchor expires |
| 2031-06-08 | MAC v4 anchor expires |

## Reporting a problem

If a fingerprint here does not match one you were given elsewhere, or a certificate will not verify, stop using it until the discrepancy is resolved. This applies to anything the WKD serves that does not match a certificate above.

Report it two ways where possible: open an issue on `Migrant-Action-Centre/keys`, which keeps it on the record, and reach a person at contact@migrantaction.ca.

To report without attaching a name to it, use <https://secure.migrantaction.ca>, or the same service as an onion:

    http://6j463ov2a5fg5hqnqg2725lzv46oj46e4tqaztoj76246smeuofzpjid.onion
