# Introduction to cryptography - terminology and examples
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
        random.seed(key)

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

While you can attempt to brute-force such a generic cipher, it might take a while. For example, try to decode the following text:

```
Etkwbyh kty akw oepabw wkdez, rw'i ess kmyh wby ueuyhi. "Wyyteayh
ehhyiwyd rt oknupwyh ohrny ioetdes", "Beoxyh ehhyiwyd elwyh jetx
wenuyhrta"...
Dent Xrdi. Wbyz'hy ess esrxy.
Jpw drd zkp, rt zkph wbhyy-uryoy uizobkskaz etd 1950'i wyobtkjhert ymyh
wexy e skkx jybrtd wby yzyi kl e beoxyh? Drd zkp ymyh gktdyh gbew nedy
brn wrox, gbew lkhoyi ibeuyd brn, gbew nez bemy nksdyd brn?
R en e beoxyh, ytwyh nz gkhsd...
Nrty ri e gkhsd wbew jyarti grwb iobkks. R'my sriwytyd wk wby wyeobyh
ycusert lkh wby lrlwyytwb wrny bkg wk hydpoy e lheowrkt. R ptdyhiwetd
rw. "Tk, Nhi. Inrwb, R drdt'w ibkg nz gkhx. R drd rw rt nz byed..."
Dent xrd. Uhkjejsz okuryd rw. Wbyz'hy ess esrxy.
R nedy e driokmyhz wkdez. R lkptd e oknupwyh. Gerw e iyoktd, wbri ri
okks. Rw dkyi gbew R getw rw wk dk. Rl rw nexyi e nriwexy, rw'i jyoepiy
R iohygyd pu. Tkw jyoepiy rw dkyit'w srxy ny...
kh lyysi wbhyewytyd jz ny...
kh wbrtxi R'n e inehw eii...
kh dkyit'w srxy wyeobrta etd ibkpsdt'w jy byhy...
Dent xrd. Ess by dkyi ri usez aenyi. Wbyz'hy ess esrxy.
Etd wbyt rw beuuytyd... E dkkh kuytyd wk e gkhsd... Hpibrta wbhkpab wby
ubkty srty srxy byhkrt wbhkpab et eddrow'i myrti, et ysyowhktro upsiy ri
iytw kpw, e hylpay lhkn wby dez wk dez rtoknuywytoryi ri ikpabw... E
jkehd ri lkptd.
"Wbri ri rw... Wbri ri gbyhy R jyskta..."
R xtkg ymyhzkty byhy... Ymyt rl R'my tymyh nyw wbyn, tymyh wesxyd wk
wbyn, nez tymyh byeh lhkn wbyn eaert... R xtkg zkp ess...
Dent xrd. Wzrta pu wby ubkty srty eaert. Wbyz'hy ess esrxy...
Zkp jyw zkph eii gy'hy ess esrxy... Gy'my jyyt iukkt lyd jejz lkkd ew
iobkks gbyt gy bptayhyd lkh iwyex... Wby jrwi kl nyew wbew zkp drd syw
isru wbhkpab gyhy uhy-obygyd etd weiwysyii. Gy'my jyyt dknrtewyd jz
iedriwi, kh ratkhyd jz wby euewbywro. Wby lyg wbew bed iknywbrta wk
wyeob lkptd pi grssrta upursi, jpw wbkiy lyg ehy srxy dhkui kl gewyh rt
wby dyiyhw.
Wbri ri kph gkhsd tkg... Wby gkhsd kl wby ysyowhkt etd wby igrwob, wby
jyepwz kl wby jepd. Gy nexy piy kl e iyhmroy eshyedz ycriwrta grwbkpw
uezrta lkh gbew okpsd jy drhw obyeu rl rw geit'w hpt jz uhklrwyyhrta
aspwwkti, etd zkp oess pi ohrnrtesi. Gy ycuskhy... Etd zkp oess pi
ohrnrtesi. Gy ycriw grwbkpw ixrt okskh, grwbkpw tewrktesrwz, grwbkpw
hysrarkpi jrei... Etd zkp oess pi ohrnrtesi. Zkp jprsd ewknro jknji,
zkp geay gehi, zkp nphdyh, zkp obyew, etd sry wk pi etd whz wk nexy pi
jysrymy rw'i lkh kph kgt akkd, zyw gy'hy wby ohrnrtesi.
Zyi, R en e ohrnrtes. Nz ohrny ri wbew kl ophrkirwz. Nz ohrny ri wbew
kl qpdarta uykusy jz gbew wbyz iez etd wbrtx, tkw gbew wbyz skkx srxy.
Nz ohrny ri wbew kl kpwinehwrta zkp, iknywbrta wbew zkp grss tymyh
lkharmy ny lkh.
R en e beoxyh, etd wbri ri nz netrlyiwk. Zkp nez iwku wbri rtdrmrdpes,
jpw zkp oet'w iwku pi ess...
Elwyh ess, Gy'hy ess esrxy.
```

Obviously, punctuation symbols and spaces could be of great hint, but those could either be omitted by `Alice`, or she might use a more sophisticated method - for example, using `Base64` to make sure plaintext is (mostly) composed of letters, or even changing the cipher to work on bytes rather than letters (also effectively greatly increasing the key size).
How should an attacker approach such a problem?

## Frequency analysis
Assuming the `plaintext` is written in English, one can perform a [frequency analysis](https://en.wikipedia.org/wiki/Frequency_analysis) - there is an expected distribution of letters in the English language and we could use that! For example, it's well-known that the letter "e" is the most common one, so it's expected that the most repeated letter in the ciphertext is going to be mapped to "e". The letter "x" is not very common, so we expect one of the rarest letters in the ciphertext to be mapped to "x", and so on. We could even apply the same logic to more than one letter - for example, after "q" there is almost always a "u" in the English langugage. In other words - *the distribution of letters does not change in monoalphabetic substitution ciphers*.
Note this will not give you the key, but our goal is to decrypt the ciphertext, so this is perfectly fine.
![English letter frequencyies](2560px-English_letter_frequency_(alphabetic).svg.png)

**Exercise** - can you use that idea to decrypt the ciphertext from the previous paragraph?
If you're stuck, here's code that heuristically finds the mapping to the letter "e":

```python
def find_e(ciphertext):
    """
        Finds the most probably mapping to the English letter "e".
    """

    # Get just the letters
    letters = [ c.lower() for c in ciphertext if c.isalpha() ]

    # Build the letter distribution
    distribution = {}
    for c in letters:
        if c not in distribution:
            distribution[c] = 0
        distribution[c] += 1

    # Find the most frequent letter
    return [ c for c in distribution if distribution[c] == max(distribution.values()) ][0]
```

Indeed I can tell you that "e" was mapped to "y", just as this function indicates. Can you decrypt the entire thing?

## Summary
In this first blogpost, we've introduced some basic concepts such as the difference between `encoding`, `encryption` and `steganography`.
We've introduced the `Caesar Cipher` as an example of a larger family of ciphers called `Monoalphabetic Substitution Ciphers` and showed the basic idea of `Frequency Analysis` to break them.

In the next blogpost, we'll continue talking about `Substitution Ciphers` and then move towards more modern cryptosystems.
Stay tuned!

Jonathan Bar Or
