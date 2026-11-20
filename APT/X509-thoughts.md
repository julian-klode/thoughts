# Thoughts on X.509 signing and witness-cosigned skip-chained release file architecture

As many know we are working on X.509 signed APT repositories and
I would like to share some thoughts regarding what the future of signing
entails past the X.509 foundations.

## Signing restrictions
X.509 CAs and keys can include restrictions on what Origin and
Suite/Codename combinations they can sign. Think of this like
the Subject and SAN in the web but instead of domains it's
release file fields.

This allows building a nice structure like:

    CA - Origin: Ubuntu
        CA - Origin: Ubuntu/Codename: tenacious
            Key - Origin: Ubuntu/Suite: tenacious
            Key - Origin: Ubuntu/Suite: tenacious-proposed

## Rollback protection

We are designing an auxiliary scheme where given two chains
and signatures S(A, B, t) from key A to B at time t, a new
chain must not include a signature(S, A, B', t') for some
key B' at a time t' < t.

In X.509 terms, this is mapped to notBefore.

This is being modeled in Lean 4 with corresponding proofs
regarding soundness and completeness of key rotations.

## Skip-chained Release files

This was also discussed previously, Release files gain a new field

    Updates:
         SHA256:<hex digest> Timestamp:<Date of Release file, in seconds since epoch>
         SHA256:<hex digest> Timestamp:<Date of Release file, in seconds since epoch>

The previous file must be included and additional skip levels can
be introduced to improve look up performance. This is easy to maintain
if a bit less efficient than a Release files

## Witnesses with k out of n quorum

A witness observes a signature of the release file and signs it
with its own key.

Clients will configure a Cosigned-By keyring determining valid
witnesses an the number of keys needed for quorum, say:

    Cosigned-By: 3 of /usr/share/keyrings/debian-cosigner-keyring.pem

This, in combination with the requirements imposed on a witness, means that a
repository cannot present split views to different clients.

A witness is inductive:

0. Nothing observed
1. A given Origin/Suite/Codename triplet is observed the first time
2. (and so on) A given Origin/Suite/Codename is observed an 2nd time

It effectively observes a tail of the log, not necessarily the entire log, 
albeit it could replay the history from the snapshot service.

n-th cosignatures require the witness to verify that for each
ancestor inside Updates it is:

- An ancestor prior to the witness' birth
- The hash was witnessed for the given Date by the witness