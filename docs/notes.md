# notes

The birds, the bees, and the merkle trees

## episode ideas

0. blockchains from scratch
  - who this is for: targeted at developers, but I'll try to explain everything in plain english.
  - what I hope to achieve with this series: educate developers who learn through doing.
  - why "from scratch" matters: understanding a system by tinkering with it.
  - why rust: low-level, friendlier than C, explicit, fun.
  - setup: install rust, etc...
  - whats next: meet the crypto

1. hashing things out
  - recap
  - introduce cryptography and core cryptographic concerns of blockchains.
  - introduce hashing functions: data fingerprinting, one-way functions, etc...
  - demonstrate the asymmetry of input diff vs output diff.
  - build a toy hash-based crypto algorithm.
  - TODO
  - whats next: the identity problem


2. the identity problem
  - recap
  - discuss importance of proving identity; blockchain or otherwise.
  - introduce asymmetric crypto: signing, identitiy, public keys...
  - show basic ECC signing.
  - make toy transaction system.
  - TODO
  - whats next: build a toy blockchain


3. build a toy blockchain
  - TODO



## supporting crates

Use `eth-crypto` crate's simplified apis to allow readers
to experiment with hashing/ecc/etc... without getting caught
up in irrelevant details.


