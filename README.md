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

Neither copy is automatically correct. This repo is the broader register: it carries v6, S/MIME and the organizational anchor, none of which WKD can hold, and it can carry several certificates for one address. That is a claim about coverage, not about which side to believe when the two disagree.

In practice this repo is the likelier one to be stale. Publishing to WKD is `sq network wkd publish` and a push. Updating here means exporting the certificate, editing KEYS.md, editing index.html, updating the Keys entry in Webflow, and a signed commit. The heavier path is the one that gets skipped, so a mismatch more often means this repo is behind than that WKD is serving something wrong. Establish which certificate is actually current before changing either side, and do not reconcile by copying this repo over WKD out of habit.

Run `tools/check-wkd-sync` to catch drift. For every `@migrantaction.ca` email UID in `pgp/`, it fetches what the live WKD actually serves and requires the fingerprint to match at least one local certificate for that address. The "at least one" is because v4/v6 pairs share an address and WKD carries only the v4. It fetches the live URL rather than the sibling checkout, so it also catches a certificate that was committed but never deployed. Exits non-zero on mismatch, so it drops into CI unchanged. A failure usually means this repo is behind rather than that WKD is wrong; read the fingerprint it reports and confirm against the key material before deciding which side to change.

### What the landing page tells visitors

`index.html` deliberately does not tell readers to believe this page when it disagrees with WKD. A visitor cannot distinguish ordinary drift from any other cause, and a page instructing people to trust it is worth nothing precisely when the page is the thing that is wrong. So it says to stop using both copies until the mismatch is resolved, then to report it two ways: an issue on this repo, which keeps it on the record whether or not we act on it, and a person at contact@migrantaction.ca. `secure.migrantaction.ca` and its onion service are offered for reporting without attaching a name.

The page telling readers to stop using both copies is deliberate, and it matches the fact that neither side is automatically correct. Do not replace it with a rule preferring one over the other, and do not put the maintenance detail above on the public page.

Certificate identity and contact are separate. The `<h2>` heading, the `sq network wkd search` argument and the `gpg --fingerprint` argument all name adi@migrantaction.ca because that is the UID on the certificate and what WKD is keyed on. Reporting goes to contact@migrantaction.ca. Do not unify them; changing the lookup arguments breaks WKD resolution and mislabels the certificate.

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

### Security headers

`_headers` sets a Content-Security-Policy on every response. Each header name appears in exactly one block, because Cloudflare Pages applies every matching rule and concatenates the results, so a name repeated across two overlapping blocks is sent twice.

`Cross-Origin-Resource-Policy` is `cross-origin` so that it agrees with the `Access-Control-Allow-Origin` above. `same-origin` would block browser-based verifiers after they had already passed CORS, which defeats the point of setting CORS on the certificate directories at all.

`style-src` is `'self'` rather than `'unsafe-inline'` because the stylesheet lives in `styles.css`. Moving it back inline would force the weaker policy.

### Email obfuscation

Cloudflare's Scrape Shield rewrites email addresses in HTML responses and injects a decoder from same-origin `/cdn-cgi/scripts/` to restore them in the browser. The policy therefore sets `script-src 'self'`. Under `default-src 'none'` alone the decoder is blocked, the placeholder is never replaced, and every reader sees `[email protected]` permanently, including inside the `sq` and `gpg` commands they are meant to copy and run. `script-src 'self'` still blocks the cross-origin Cloudflare Insights beacon, which is what keeps the page's claim about third party requests true; relax it and that sentence in the footer becomes false.

Obfuscation only rewrites HTML, so `KEYS.md` is served with its addresses intact. The page links it as the fallback for anyone browsing without JavaScript, which is why the reporting addresses and the onion appear there as well. Keep them in sync.
