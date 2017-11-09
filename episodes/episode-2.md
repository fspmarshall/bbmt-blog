# The Birds, the Bees, and the Merkle Trees ep[2]: The Identity Problem

*This is episode 2 of 'The Birds, the Bees, and the Merkle Trees',
a developer-focused educational series on blockchain technology.
The purpose of this series is to give a bottom-up view of how blockchains
work, by experimenting with and building core blockchain technologies.
This series is for educational purposes only and should not be construed
as advice or recommendations concerning best-practices for security,
cryptography, etc... This series is developed openly on github,
at `forrest-marshall/bbmt-blog`.*


## Introduction

In [episode zero](./episode-0.md) I gave a brief overview of the properties of
blockchains that make them interesting and useful, as well as my motivations for
writing this series.  In [episode one](./episode-1.md) I introduced cryptographic
hashing functions, and we wrote a toy protocol showing how hashing might be used
to prove knowledge of a secret without revealing it to eavesdroppers.  I suggest
reading both of these posts first if you haven't already.

The focus of this episode will be identity related cryptography.  More specifically,
digital signatures and public/private key schemes.  While one could theoretically
build a blockchain without such things, these technologies are what allow us to enforce
one of the most useful properties that most blockchains employ: permissioned roles.
Systems which integrate permissioned roles specify some set of actions which may
or may not be permissible based on *who* you are, as well as the current state of 
your identity within the system.  In the case of a cryptocurrency, for example, I can 
only spend the balance stored in my own account, and only up to the amount that I
actually posses.

This episode will focus on solving the problem of identity, so it is worth
deciding what exactly we mean by identity for the purposes of this discussion.
Given that beginning our discussions with a poorly contextualized Wikipedia quote is
practically a tradition at this point, here you go:

> ...the necessary and sufficient conditions under which a person at one 
> time and a person at another time can be said to be the same person, 
> persisting through time.  --Wikipedia 

The above quote is taken from an article about the philosophy of identity, but it isn't a 
half bad place to start for the purposes of cryptography.  If I want to check if you are 
who you claim to be, I need some set of past information (the identity that you are claiming), 
as well as some set of current information (your claim to that identity).  Given that scoundrels
and eavesdroppers abound in this world, it would be ideal if I could perform this verification 
in a manner that is both difficult to subvert, and does not require a the existence of a 
shared secret (such as a password). 

One of the simplest and oldest forms of validating a claim of identity (without a shared secret)
is with a handwritten signature.  Assuming a signature is difficult to forge, it allows identity
to be confirmed only with public knowledge: the expected appearance of the signature.  Signatures
may be added to messages, transactions (bank checks, package receipts, etc..), and contracts.
Given that a handwritten signature does not show what it signs, wax seals, watermarks, and
other techniques were often used to ensure that the item being singed was tamper-evident.
Operating under the dubious assumption that signatures and wax seals are protections that
cannot be circumvented, the assumption can be made that a properly sealed and signed message
represents the true intention of the identity in question.

In this episode we will examine digital signature algorithms, their history, basic usage,
and the specific implementation used by Ethereum and Ethereum-like blockchains.


## Interlude

If you are following along with code examples, setup for this episode is the same as in
[episode one](./episode-1.md); generate a binary project with `cargo new`, add the
`eth-crypto` crate to the resultant `Cargo.toml` file, and import its `prelude`.

The helpful folks on the Rust subreddit pointed out that it is potentially problematic to 
be casually tossing around examples based on a nonstandard crypto crate.  I am using the 
`eth-crypto` crate to reduce front-loaded complexity for readers that are unfamiliar with Rust
and/or cryptography.  The `eth-crypto` crate is a wrapper around a set of cryptography crates
which are commonly used in the Ethereum ecosystem.  The goal of the wrapper is to simplify
the APIs of the underlying crates, as well as to conform to the standard formatting and 
idioms of Ethereum.  I'd like to take a moment to give a very stern warning; cryptography is
very easy to get wrong, and none of the examples we cover here necessarily represent
best-practices in terms of design-patterns or algorithm choice.

In the previous episode we used the `keccak-256` hashing
algorithm to hash a password as part of one of our examples; `keccak` is *not* a password
hashing algorithm, and should not be used this way in the wild.  Very specific considerations
are put into designing the algorithms that are used for password hashing.  Always do the legwork of 
determining the best-practices and most appropriate algorithms for your *specific* need.  
If you are looking for Rust cryptography crates, the [Rust Crypto](https://github.com/RustCrypto)
project is a great place to start your search.

## A Brief History of Signing

TODO


