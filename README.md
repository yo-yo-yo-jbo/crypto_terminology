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

## First example - Caesar cipher
It's impossible to write a crypgotraphy introduction without mentioning `Caesar's cipher`. Without diving into the historical context, we will mention it was one of the first documented ciphers used.
The idea is simple: we use a key (between 1 and 25), and "rotate" all English letters according to the key. Historically, Caesar cipher used `key=3`, but we can use any other number between 1 and 26.
Decryption means :

```python
def caesar_encrypt(plaintext, key):
    """
        Encrypts with Caesar cipher.
    """

    # Validate key
    assert key >= 1 and key <= 25, Exception(f'Invalid key: {key}')

    # Work letter by letter
    result = ''
    for c in plaintext:
        if not c.isalpha():
            result += c
            continue
        base = ord('a') if c.islower() else ord('A')
        result += chr(base + ((ord(c) - base - key) % 26))

    # Return result
    return result

def caesar_decrypt(ciphertext, key):
    """
        Decrypts with Caesar cipher.
    """

    # Decrypt by encrypting the opposite key
    return caesar_encrypt(ciphertext, 26-key)
```

For example, "THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG" turns into "QEB NRFZH YOLTK CLU GRJMP LSBO QEB IXWV ALD" when `key=3`.
Also note how decryption uses the same key as the encryption key, as well as simply calling the encryption method (due to the cyclic nature of the cipher).

Is the Caesar cipher good? It's certainly *fast*, but when we talk about security it's not great.
First of all, the size of the key is quite small. Usually we assess the strength by the number of bits of the key. Since the key is a number between 1 and 25, the number of bits requires for the key is `5` (since 2 to the power of 5 is over the amount of keys possible), which means an attacker needs to brute-force 5 bits (practically less than that) to brute-force the cipher - a task achievable even by hand.
In its historical context, the Caesar Cipher was fine, but in today's standards it's quite weak as it can be brute-forced in less than a second.

## Subsitution ciphers
The Caesar cipher is a member of a larger family of ciphers, called [Substitution Ciphers](https://en.wikipedia.org/wiki/Substitution_cipher) - the cipher "simply" substitutes letters.
Specifically, Caesar cipher substitutes a single letter to a different single letters, so we say it's a `Monoalphabetic Substitution Cipher`.
Generally we can think of *all* monoalphabetic substitution ciphers as ciphers that take letters and apply some function to each letter seperately.
Obviously, there are many possibilities - even for English letters we have `26! = 403291461126605635584000000` possibilities! This number has roughly 89 bits, which will take a while to brute-force with a computer, and not possible to perform by-hand.
Here's an example of such a cipher - it takes an integer key and creates a monoalphabetic cipher by seeding the random with it and creating a random shuffle:

```python
import random
import string

class MonoalphabeticCipher(object):
    """
        Generic monoalphabetic substitution cipher.
    """
    def __init__(self, key):
        """
            Creates an instance.
        """

        # Use the key to seed the PRNG
        random.seed(self.key)

        # Create the encryption table
        self.enc_table = list(string.ascii_lowercase)
        random.shuffle(self.enc_table)
        self.enc_table = ''.join(self.enc_table)

        # Create the decryption table
        self.dec_table = ''.join([ string.ascii_lowercase[self.enc_table.find(i)] for i in string.ascii_lowercase ])

    def _cipher(self, s, enc):
        """
            Performs the cipher operation.
        """

        # Builds the result and concludes the right table to use
        result = ''
        table = self.enc_table if enc else self.dec_table

        # Work character by character 
        for c in s:
            if not c.isalpha():
                result += c
                continue
            result += table[ord(c.lower()) - ord('a')].upper() if c.isupper() else table[ord(c) - ord('a')]

        # Return the result
        return result

    def encrypt(self, plaintext):
        """
            Encrypt a plaintext.
        """

        # Simply encrypt
        return self._cipher(plaintext, True)

    def decrypt(self, ciphertext):
        """
            Decrypts a ciphertext.
        """

        # Simply decrypt
        return self._cipher(ciphertext, False)
```


I would like to discuss another disadvtantage - one can cleverly choose the most probable result!

## Frequency analysis
Assuming the `plaintext` is written in English, one can perform a [frequency analysis](https://en.wikipedia.org/wiki/Frequency_analysis) - there is an expected distribution of letters in the English language and we could use that! For example



