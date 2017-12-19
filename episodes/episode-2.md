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

First off, I would like to apologize to those of you following along in real-time;
this episode is coming out far later than I'd planned.  It would appear that 
working at a young tech startup does not necessarily lend itself to spare
time for side-projects.  Please feel free to pause for a moment while you process 
the this novel and shocking observation.

The focus of this episode will be identity related cryptography.  More specifically,
digital signatures and public/private key schemes.  While one could theoretically
build a blockchain without such things, these technologies are what allow us to enforce
one of the most useful properties that blockchains typically employ: permissioned roles.
Systems which integrate permissioned roles specify some set of actions which may
or may not be allowed based on *who* you are, as well as the current state of 
your identity within the system.  Cryptocurrencies are an excellent example of the
importance of identity.  In order for a cryptocurrency blockchain to be useful,
it must ensure two key properties: only I can spend the balance that I hold, and
I cannot spend more than I actually hold.  Cryptographically proveable identity
is key to making these properties enforceable.

Since this episode will be exploring the problem of identity, it is worth
deciding what exactly we mean by "identity" (in this context anyhow).
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
shared secret that we previously agreed upon (e.g.; a password). 

One of the simplest and oldest forms of validating a claim of identity (without a shared secret)
is with a handwritten signature.  Assuming a signature is difficult to forge, it allows identity
to be confirmed with only public knowledge: the expected appearance of the signature.  Signatures
may be added to messages, transactions (bank checks, package receipts, etc..), and contracts.
Given that a handwritten signature does not encode what it actually signs, wax seals, watermarks, and
other techniques have been used to ensure that the item being singed is tamper-evident.
Operating under the dubious assumption that signatures and wax seals are protections that
cannot be circumvented, the assumption can be made that a properly sealed and signed message
represents the true intention of the identity in question.

Much like the sealed and signed letters of the past, modern digital signature algorithms attempt
to verify the source and content of some piece of data without any pre-existing secret knowledge.
The output of a good digital signature algorithm should be a compact piece of information which can
have its source and associated data easily verified, but which is difficult to counterfeit.  The astute 
reader may notice that this is remarkably similar to the goal of a cryptographic hashing function (covered
in episode 1).  The difference here is that a hash need only show association with a piece of data,
whereas a digital signature needs to show association with a piece of data and an entity.

The rest of this episode will be an exploration of digital signature algorithms, their history, 
basic usage, and the specific implementation used by Ethereum and Ethereum-like blockchains.


## Interlude

If you are following along with code examples, setup for this episode is the same as in
[episode one](./episode-1.md); generate a binary project with `cargo new`, add the
`eth-crypto` crate to the resultant `Cargo.toml` file, and import the contents of
the `prelude` submodule.

The helpful folks on the Rust subreddit pointed out that it is potentially problematic to 
be casually tossing around examples based on a nonstandard cryptography crate.  The `eth-crypto` 
crate is a wrapper around a set of cryptography crates which are commonly used in the Ethereum 
ecosystem.  The goal of the wrapper is to simplify the APIs of the underlying crates, as well 
as to conform to the standard formatting and idioms of Ethereum.  I'd like to take a moment 
to give a very stern warning; cryptography is very easy to get wrong, and none of the examples 
we cover here necessarily represent best-practices in terms of design-patterns or algorithm choice.

In the previous episode we used the `keccak-256` hashing
algorithm to hash a password as part of one of our examples; `keccak` is *not* a password
hashing algorithm, and should not be used this way in the wild.  Very specific considerations
are put into designing the algorithms that are used for password hashing.  Always do the legwork of 
determining the best-practices and most appropriate algorithms for your *specific* need.  

If you are looking for Rust cryptography crates, the [Rust Crypto](https://github.com/RustCrypto)
project is a great place to start your search.  They have Rust implementations of many different
cryptographic algorithms, including a number of password-hashing algorithms.


## A Brief History of Signing

Cryptographic signature schemes have their root in a more general set of cryptographic algorithms
known as asymmetric cryptography.  Until the mid twentieth century, the passing of secret messages 
was typically performed via encryption with a shared secret (symmetric cryptography).  This practice
basically boiled down to using some secret piece of information to scramble a message such that
unscrambling it was infeasible without knowing the information.  This had serious drawbacks; no
two parties could ever exchange secret information without first securely exchanging a secret
of some kind.  To put this in perspective, if we were still limited to symmetric cryptography today,
you would likely need to meet in secret with the owner of a website before you would be able
to generate a secure connection to their server.  No fun whatsoever.

Asymmetric cryptography works on a fundamentally different scheme than symmetric cryptography.
Rather than sharing a single secret between corresponding parties, each individual generates
a pair of values.  One value, known as the public key is published freely without concern
about who sees it.  The other value, known as the private, or secret, key, is kept concealed
and never shared with anyone.  If an individual would like to send you a secret message,
they only need to know your public key.  The message may be encrypted using the public
key such that only the corresponding private key can decrypt it.

Aymmetric cryptography can feel like magic when you first hear about it, so lets dig a little 
deeper.  The math behind the various asymmetric cryptography algorithms is non-trivial, and
well beyond the scope of this series, but we can approximate it to some degree.  Lets say
you have two functions, `f` and `g`.  These functions are inverses of each other, meaning that
if `f(x) = y`, then `g(y) = x`.  If you generate `y` by executing `f(x)` and share it with me,
I can execute `g(y)` to derive the original `x` that you used.  If `f(x)` was defined as `x + 3`,
then `g(y)` might be defined as `y - 3`.  For an operation like `x + 3`, calculating the inverse
function is easy, so this doesn't really achieve much.  Imagine, however, if we could find some
function `f` which wasn't easy to reverse.  In this case, if I know `f` and `g`, I can freely distribute `f`,
without fearing that someone might derive `g`.  Then, if someone wanted to share a secret message
with me, they can pass the message through `f` before broadcasting it.  If only I know `g`, only I can
decode the message.  This is, in wildly oversimplified terms, how asymmetric cryptography works.

Real asymmetric cryptography algorithms are based on certain special mathematical problems which
are believed to have no efficient solution (i.e.; no shortcuts to their solutions).  If I have some
function `f` for which there exists no shortcut to invert it, then the most a potential attacker
can do is try random (or sufficiently close to random to not matter) permutations until they
succeed in guessing `f`.  Therefore, all i need to do is decide on a variant of `f` with sufficient
degrees of freedom (possible variations) to ensure that finding its inverse is impractical with modern
technology.  This is why most cryptographic algorithms have a bit-length associated with them
(`keccak-256`, `RSA-2048`, `SHA-512`, `secp256-k1`, etc...).  As computers get faster, the size of the 
values used in our cryptographic algorithms can be increased to ensure that they are still impractical
to circumvent.

The importance of the degrees of freedom in a cryptographic algorithm is not necessarily intuitive.
To put things into perspective, our sun will have expanded into a red giant and boiled the oceans off 
of the earth well before my laptop could check all the possible permutations of a 256 bit number.
Despite this impressive statistic, it is worth noting that bigger isn't always better in the crypto world.  
A 256 bit ECC key is more difficult to crack than a 2048 bit RSA key, because their security is based on
different underyling problems.  Additionally, the ability to increase degrees of freedom does not necessarily
guarantee future security.  Most asymmetric cryptography algorithms in use today will be much easier 
to crack once quantum computing is economically viable.  Hashing algorithms, on the other hand, don't
appear to be at significant risk from quantum computers.  Once again, the math behind this is beyond
the scope of this series, but what problems quantum computers do and don't offer improved solutions
for is fairly nuanced.



