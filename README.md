# draft-drake-email-hardware-attestation

**Hardware Attestation for Email Sender Verification**

This is the companion repository for the IETF Internet-Draft
[draft-drake-email-hardware-attestation-03](https://datatracker.ietf.org/doc/draft-drake-email-hardware-attestation/).
The files of the earlier revisions (-00, -01, -02) stay in this repository
for reference.

## Abstract

This document defines two email bindings for proving properties of an
automated sender using the durable identity and anchor architecture of
the Agent Identity Registry System (AIRS).

1. **Mode 1** (`Hardware-Attestation` header): a detached CMS signature
   (RFC 5652) made by a message-signing proof key.  It can provide
   manufacturer-rooted hardware provenance, Registrar-backed AIRS identity
   binding, or both.

2. **Mode 2** (`Hardware-Trust-Proof` header): a per-message SD-JWT
   (RFC 9901) that allows a Registrar to assert selected properties without
   requiring disclosure of the sender's canonical AIRS identifier.

The AIRS identity model, the canonical `aid` identifier, trust tiers, anchor
semantics, enrollment rules and the Registrar trust boundary are defined by
`draft-drake-agent-identity-problem-statement` and
`draft-drake-agent-identity-registry`; authoritative resolution of the current
Registrar/OAuth issuer is defined by `draft-drake-agent-identity-resolution`.
This document defines how email messages bind to and verify them.

Together, these mechanisms provide Sybil-resistant email authentication
at the tiers whose anchors are scarce physical units (sovereign and
portable): each identity there requires its own manufacturer-attested
anchor.

## Repository Contents

```
draft-drake-email-hardware-attestation-03.xml   # I-D source (xml2rfc v3)
draft-drake-email-hardware-attestation-03.txt   # rendered text
draft-drake-email-hardware-attestation-03.html  # rendered HTML
examples/
  example_1_sovereign_tpm_combined.eml            # Sovereign: Intel PTT firmware TPM (Windows 11)
  example_2_portable_piv_combined.eml             # Portable: YubiKey PIV
  example_3_enclave_secure_enclave_combined.eml   # Enclave: Apple M4 Secure Enclave
  example_4_virtual_vtpm_combined.eml             # Virtual: VMware vTPM (Windows 11)
  example_5_declared_software_combined.eml        # Declared: software P-256 key
  SHA256SUMS                                      # exact bytes (CRLF) of the five emails
```

The `.eml` files are real emails, each carrying both modes (Combined mode),
generated on 27 October 2026 (AEST) through the live system with the
**published** reference implementation only: `oneid` 4.0.0 (PyPI), the helper
`oneid-enroll` v3.0.0 (GitHub release; the Windows executable is
Authenticode-signed, and the macOS binaries used here are Developer ID signed
and notarized) and `hw-attest-verify` 3.0.0 (PyPI).  Every run used a private
credentials folder, and the Windows runs were made by an ordinary,
non-elevated user.  The messages were sent through MailPal.com
(From and To are the identity's own mailbox), stamped by the receiving
verifier (the MailPal milter) and copied raw from the receiving message
store (the server received them between 18:21 and 18:26 UTC on 26 October).
They are the same emails reproduced in the draft's appendix, and they replace
the 2026-09-25 examples of revision -02 (still in this repository's git
history).

| File | Identity | Hardware |
|---|---|---|
| example_1 | id-vwgkj-mhtjp-mfscq-bdchb | Intel PTT firmware TPM in an LG gram 17, Windows 11 Pro |
| example_2 | id-csnxq-phcmb-pvcmm-xcdmb | YubiKey (firmware 5.4.3, PIV slot 9a, ECDSA P-256), in a Windows 11 Pro guest |
| example_3 | id-ddmzh-dpcfv-fqrxj-xmxsj | Apple M4 Secure Enclave (Mac mini, macOS 26.4.1) |
| example_4 | id-tpfkx-qdzts-wmtbm-rnjpc | VMware virtual TPM 2.0 in a Windows 11 Pro guest |
| example_5 | id-hgsst-vgcfx-pwdvw-jkbjq | software P-256 key (declared tier) on an LG gram 16, Windows 11 Pro |

Examples 1, 3 and 5 are brand-new enrollments (example 1 ran the TPM
authenticated-creation ceremony of `draft-drake-agent-identity-registry`
through the Windows TBS service with no elevation).  A YubiKey and a vTPM
endorsement key can be registered to only one identity, so examples 2 and 4
reuse identities that already existed on that hardware; for those two, only
the send used the published packages.  The `Received` fields are as recorded
by the MailPal server, including the sender's public address.

## Building the Draft

The XML source uses [xml2rfc](https://xml2rfc.tools.ietf.org/) v3 format:

```bash
pip install xml2rfc
xml2rfc draft-drake-email-hardware-attestation-03.xml --html
xml2rfc draft-drake-email-hardware-attestation-03.xml --text
```

## Verifying the Example Emails

Install the reference verification tool.  Version 3.0.0 is the first one that
implements revision -03; version 2.0.2 and earlier implement revision -02 and
reject these examples:

```bash
pip install 'hw-attest-verify>=3.0.0'
```

Verify any example email with the draft's appendix command (use
`--no-time-check` since the attestation timestamps are in the past; no
trust store is needed for these examples: Mode 1 is verified through the
Registrar binding and Mode 2 as an identified presentation):

```bash
python3 -m hw_attest_verify --auth-results --no-time-check --hostname mailpal.com \
  < examples/example_1_sovereign_tpm_combined.eml
```

The `.eml` files are stored byte-exact (CRLF; `examples/.gitattributes` says
`*.eml -text`), because the signatures and `SHA256SUMS` cover those bytes.  The
verifier also accepts LF-only copies, such as the text in the draft.

## Trust Tiers

The registry draft defines five assurance tiers: sovereign, portable, enclave,
virtual and declared.  At the sovereign and portable tiers each independently
accepted identity requires its own manufacturer-attested physical anchor, and
each accepted anchor backs at most one identity.  The enclave and virtual tiers
can provide hardware-backed key protection without making that
physical-scarcity claim, and the declared tier permits software-only
participation.  The details (anchors, enrollment, upgrades between tiers) are
in `draft-drake-agent-identity-registry`.

## Related Resources

| Resource | URL |
|----------|-----|
| **1id.com** (reference Registrar implementation) | https://1id.com |
| **Attestation verifier** (`pip install hw-attest-verify`) | https://github.com/1id-com/hw-attest-verify |
| **Python SDK** (`pip install oneid`) | https://github.com/1id-com/oneid-sdk |
| **Node.js SDK** (`npm install 1id`) | https://github.com/1id-com/oneid-node |
| **Go binary** (TPM/PIV/Enclave operations) | https://github.com/1id-com/oneid-enroll |
| **TPM Manufacturer CA Trust Store** | https://github.com/1id-com/tpm-manufacturer-cas |
| **IETF Datatracker** | https://datatracker.ietf.org/doc/draft-drake-email-hardware-attestation/ |

## Implementation Status

Per [RFC 7942](https://www.rfc-editor.org/rfc/rfc7942.html), a working
implementation exists at [1id.com](https://1id.com).  The implementation
includes:

- **Server-side verification** of hardware identity certificate chains
  with anti-Sybil enforcement (one identity per hardware anchor).
- **Client-side attestation** via Python SDK, Node.js SDK, and a
  cross-platform Go binary (`oneid-enroll`) supporting Windows TBS,
  Linux /dev/tpmrm0, macOS Secure Enclave (via Swift helper), and
  PIV smart cards.
- **SD-JWT trust proof issuance** via the 1id.com enrollment API,
  with selective disclosure per RFC 9901.
- **Direct hardware attestation** (Mode 1) with CMS SignedData per
  RFC 5652, bound to the AIRS identity by a Registrar binding.
  `hw-attest-verify` 3.0.0 implements the Registrar-bound trust path;
  the manufacturer-rooted path is reserved in revision -03, because that
  revision names no evidence profile for it.
- **Sender-constrained tokens**: every access token carries `cnf.jwk`
  (the enrolled key), and the 1id.com and MailPal.com APIs accept a token
  only with an RFC 9421 HTTP Message Signature by that key.
- **Production email delivery** through Stalwart SMTP with DKIM,
  verified end-to-end across all five trust tiers using the
  `hw-attest-verify` reference tool.

## License

The Internet-Draft XML source is subject to the IETF Trust Legal
Provisions (TLP).  See https://trustee.ietf.org/license-info for
details.

Example emails are provided under the
[Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

## Author

Christopher Drake <cnd@1id.com>
https://1id.com
