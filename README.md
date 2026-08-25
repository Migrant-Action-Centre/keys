# Migrant Action Centre: public keys

Public certificates for `migrantaction.ca`, served at `keys.migrantaction.ca` via Cloudflare Pages. See **KEYS.md** for the register of what is published, with fingerprints and expiry dates.

Public certificates only. No secret key material. `.gitignore` blocks the formats that carry it: `.p12` and `.pfx` bundle a private key alongside the certificate.

## Relationship to the `pgp` repo

`Migrant-Action-Centre/pgp` hosts Web Key Directory at `openpgpkey.migrantaction.ca`.  Deliberately separated:

|             | this repo                        | `pgp` repo                                  |
| ----------- | -------------------------------- | ------------------------------------------- |
| Audience    | people verifying manually        | mail clients and other tools, automatically |
| Carries     | v4, v6, S/MIME, future key types | v4 OpenPGP only                             |
| Per address | as many certs as needed          | exactly one                                 |

This repo is the source of truth; WKD publishes a subset of it. When a certificate changes here and it is also in WKD, the other repo must be updated too; nothing enforces that automatically. 

Run `tools/check-wkd-sync` to catch drift. For every `@migrantaction.ca` email UID in `pgp/`, it fetches what the live WKD actually serves and requires the fingerprint to match at least one local certificate for that address. The "at least one" is because v4/v6 pairs share an address and WKD carries only the v4. It fetches the live URL rather than the sibling checkout, so it also catches a certificate that was committed but never deployed. Exits non-zero on mismatch, so it drops into CI unchanged.

## Why both v4 and v6

v6 (RFC 9580), and specifically keys used here (RFC 9980) uses post-quantum algorithms and is preferred where the sender's client supports it. v4 (RFC 4880) is the fallback.

GnuPG follows LibrePGP (`draft-koch-librepgp`), not RFC 9580, and rejects a v6 certificate outright rather than degrading:

    gpg: packet(6) with unknown version 6
    gpg: read_block: read error: Invalid packet

That is a standards split, not a feature still landing. Do not remove the v4 certificates expecting support to have arrived.

## Adding a certificate

    sq cert export --cert <fingerprint> > pgp/<name>.asc

Always the full fingerprint, never a key ID. Key IDs are short enough to collide.

Never paste armored text into a web form or CMS field. The blank line after the `-----BEGIN-----` header gets collapsed on paste; GnuPG then rejects the whole block while Sequoia silently accepts it, so it looks fine right up until it reaches the person who needs it. Commit the file and  link to its URL.

Name files after the identity, not the key parameters.

Then update KEYS.md, and the Keys entry in Webflow, in the same commit.

### S/MIME

Download the chain from the CA (any PEM option: "single bundle" or "single certificates"). It arrives as leaf + intermediate + root. Drop the root:

    awk '/BEGIN CERTIFICATE/{n++} n<3' <downloaded> > smime/[name].crt

Then confirm two certs and a valid chain:

    grep -c 'BEGIN CERTIFICATE' smime/[name].crt
    openssl verify -untrusted smime/[name].crt smime/[name].crt

The same file appears twice because it contains both the certificate being verified and the intermediate needed to verify it.

The root is omitted deliberately. It must already be in the verifier's trust store to mean anything, and publishing a CA root invites people to install one from a website. Never commit the `.p12`; it contains the private key.

## Commits are signed

Signed with the appropriate @migrantaction.ca email's PGP key supported by GitHub. As of 2026-08-23, GitHub only supports RFC 4880 keys (ie OpenPGP v4). Verify history with:

    git log --show-signature

## Serving

`_headers` sets `application/pgp-keys` for `/pgp/*` and `application/pkix-cert` for `/smime/*`. The S/MIME file is PEM containing the leaf plus SSL.com's intermediate `pkix-cert` is strictly the DER type. Kept because it reliably triggers a download rather than rendering as text, and macOS keys off the `.crt` extension regardless. Both get `Access-Control-Allow-Origin: *` so browser-based verifiers can fetch them, and a short `max-age` so a revocation propagates in minutes rather than being cached as current.