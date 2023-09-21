# Introduction to cryptography - terminology and first examples
This is the first of a very long blogpost series I've been wanted to do for a long time.
In this very first blogpost - I'll describe what cryptography is, what it *isn't* and give some first examples.

## Terminology
`Cryptography` is a math \ computer science branch that focuses on research of securing communication channels.
One important assumption of cryptography is that anyone could intercept arbitrary messages, and in some circumstances can send arbitrary messages as anyone else.
Usually, the parties in cryptography are going to be `Alice` and `Bob`. If `Alice` wants to send private information to `Bob` without someone (e.g. `Eve`) eavesdropping and intercepting the messages, they have to somehow pass messages that cannot be understood by a 3rd party.
There are multiple solutions to this problem:
1. `Alice` and `Bob` can agree on a secret schema to *hide messages*, for example, with [invisible ink](https://en.wikipedia.org/wiki/Invisible_ink). Anyone intercepting the messages would not be able to get the message unless they know how the message was hidden. However, there are certain disadvantages to that approach - if someone *does* learn of the hiding schema, then `Alice` and `Bob` must agree on a new method of hiding messages. A similar approach is hiding messages in bigger messages, for instance, which suffers from the same disadvantage.
2. `Alice` and `Bob` can agree on a method to pass messages that also combines a `key`. That key is used for encryption and decyption (not necessarily the same key, we will talk about that).

Those two cases lead to the following terminology:
- `Steganography` - hiding data in a larger data set. For example, hiding text encoded as pixels in a picture is a well-known [steganographic method](https://en.wikipedia.org/wiki/Steganography). We will not be discussing steganography in this series.
- `Encoding` - presenting data in a different way. For example, [Base64](https://en.wikipedia.org/wiki/Base64) is a well-known encoding method, used legitimately to encode binary data as printable text, but also used for malicious purposes (e.g. for obfuscation).
- `Encryption` - transforming data using a `key` with the goal of making the data unintelligable without the key.

Note `encoding` doesn't use keys, while `encryption` does. Usually, in encryption, we will be assuming the [Kerckhoff principle](https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle) - the encryption methodology is well-known to everyone, and the only missing piece of information is a `key`.
