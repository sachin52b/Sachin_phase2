# 1.All Signs Align 


The encryption turned each bit of the flag into “residue = 0” and “non-residue = 1” without hiding that property. So decrypting was just testing each number to see if it was a residue or not.

## Solution:

I started by reading the encryption code line by line . The code looked long but the core idea was actually very small. First I noticed that the program picks a huge prime p and then defines two functions: get_x() and get_y(). What struck me was that get_x keeps picking a random number until pow(x, (p-1)//2, p) becomes 1. I came to know ftest for quadratic residue mod p. So get_x always returns a number that is a residue. Then get_y simply returns -get_x() mod p, and the interesting part is that for primes where p ≡ 3 mod 4, the negative of a residue becomes a non-residue. So the code had basically created one number that always tests as residue and another that always tests as non-residue.


Then I looked at how the flag was being turned into bits. The line:

```
for b in bin(bytes_to_long(flag))[2:]:
```

showed me that the flag is converted to a big integer and then to a binary string. For each bit, if the bit is 0 the program outputs a * get_x() mod p and if the bit is 1 the program outputs a * get_y() mod p. Here a is some random multiplier but that didn’t matter because multiplying by a nonzero a does not change whether a number is a residue or not. So the encryption was basically:


bit 0 → value that is a quadratic residue


bit 1 → value that is a non-residue


So I realized that the entire ciphertext list(given in out.txt) was literally a list of “this number is a residue” or “this number is not a residue”. Which means decrypting it was just checking pow(i, (p-1)//2, p) for every value. If pow(i, (p-1)//2, p) equals 1 then that element represents a 0 bit. Otherwise it represents a 1 bit. That gave me the original bitstring for the flag.


So I wrote my decrypt script. It looped through the ciphertext list, checked each number with the Legendre test, and appended ‘1’ or ‘0’ accordingly. After that I joined the bits into one binary string. Then I turned that binary string into an integer:

```
n = int(s, 2)
```

and finally used long_to_bytes to get back the flag. This step worked smoothly because the encryption was basically leaking bits directly.


Here is the exact script I used:

```
from Crypto.Util.number import *
p = 9129026491768303016811207218323770273047638648509577266210613478726929333106121387323539916009107476349319902011390210650434835260358014251332047605739279
l = [...]  # ciphertext list from out.txt

l2 = []
for i in l:
    if pow(i, (p-1)//2, p) == 1:
        l2.append('1')
    else:
        l2.append('0')

s = "".join(l2)
n = int(s,2)
y = long_to_bytes(n)
print(y)
```


The moment I ran it, the flag was obtaied. The encryption looked long but the actual security was zero because the Legendre test(pow(a, (p - 1)//2, p)) is fast. The code unintentionally leaked the bit value directly through “residue = one bit value and non-residue = the other bit value”. I didn’t even need the value of a.


## Flag:

```
nite{r3s1du35_f4ll1ng_1nt0_pl4c3}
```

## Concepts learnt:
1.What a Legendre test and symbol Was?


2.The concept of quadratic residue.


3.The challenge also helped me get more comfortable with going from bytes → integer → binary → back again.


## Incorrect Tangents

At first I was trying to see if the multiplier a mattered and thought maybe I had to divide it out or brute force it. Then I realized that multiplying by a nonzero number doesn’t change residue status so a was irrelevant. I also kept checking if maybe get_x would sometimes return a non-residue by mistake but the repeated pow check makes it 100 percent residue. After that everything lined up and decryption became straightforward.

## Resources:

https://youtu.be/aBn7BaRxu2g?si=vqta98Kmo9xphW9k

***

# 2.Residue Refinery

A polynomial-style mod-257 field is used to encrypt flag bytes two at a time.
Knowing one plaintext block lets us recover the key and undo the custom multiplication to decrypt the flag.


## Solution:

When I first looked at the encryption code I saw that the challenge author had already printed the first two bytes of the flag and also the ciphertext hex. These weren’t things I extracted myself, they were included in the question as comments, so I used them as the main starting point. The comment said:

```
flag[:2].hex() = '316d'
ct.hex() = 9813d3838178abd17836f3e2e752a99d5cd3fba291205f90c1d0a78b6eca
```

So before walking through the code I tried to understand what this '316d' meant. Converting hex to ASCII is the easiest part. 0x31 is 49 which is the character '1', and 0x6d is 109 which is 'm'. That told me that the first two plaintext bytes used in the encryption were simply


(49, 109)


or in bytes form b'\x31\x6d'. This was useful because knowing even one plaintext block lets you build equations for the key.


Then I moved on to the ciphertext. The printed hex was:


98 13 d3 83 81 78 ab d1 ...


but I knew I couldn’t directly work with this yet because the encryption code uses a custom .to_bytes() which reverses the byte order. I checked the definition:

```
def to_bytes(self):
    return bytes(self.n.tolist())[::-1]
```


This told me the internal ciphertext inside the Num object is [C0, C1] but what is stored in ct is [C1, C0]. So to recover the real order for each 2-byte block I just needed to flip each pair. The first ciphertext block printed was:


98 13


After reversing it I got:


13 98


This is (C0, C1) for the first block in decimal:


C0 = 0x13 = 19
C1 = 0x98 = 152


So now I had both ends of the encryption for the first block


plaintext block: (49, 109)


ciphertext block: (19, 152) after undoing the reversal


With these values I started reading the encryption function. The important part is the multiplication inside the Num class:

```
(prod[0] + 3*prod[2]) % p, prod[1] % p
```


and the way prod is filled it basically implements this formula:


Given


plaintext = (P0, P1)


key       = (k0, k1)


C0 = (k0*P0 + 3*k1*P1) mod 257


C1 = (k0*P1 + k1*P0) mod 257


So this is the entire encryption function. Once I had this fully understood I could build direct equations for the key using the known first block. I substituted


P0 = 49


P1 = 109


C0 = 19


C1 = 152


into the formulas. That gave me the two equations:


C1 = k0*P1 + k1*P0  (mod 257)


152 = 109*k0 + 49*k1  (mod 257)


C0 = k0*P0 + 3*k1*P1  (mod 257)


19  = 49*k0 + 327*k1  (mod 257)


Now I had two equations and two unknowns. This is the part where I spent the most time because I wanted to make sure I wasn’t messing up the byte reversal or the modulo. I treated them as simultaneous equations modulo 257. I isolated k0 from one equation and substituted into the other. Solving them step by step got me:


k0 = 60


k1 = 6


This matched the example solution in the challenge’s own code and gave me confidence that I had the key correct. Once the key was known the next step was decryption which is basically multiplying by the inverse of the key element in this small field.


The challenge’s solution used helper functions for mul and inv so I reused the same structure. The inverse of the key is obtained by computing the modular inverse of the determinant (k0*k0 - 3*k1*k1) and then applying the standard formula for inverses in this form of quadratic extension field. The inversion formula becomes:


inv(k0, k1) = (k0*d , -k1*d)   where d = inverse(k0^2 - 3*k1^2 mod 257)


After computing this I got invks which is the inverse of the key pair (60, 6).


Now I moved to the ciphertext. The full ciphertext hex string was converted into bytes and processed in 2 byte chunks. For every chunk I first reversed the bytes to undo .to_bytes(), so (ct[i], ct[i+1]) became (C0, C1). Then I multiplied this block with the inverse key using the same multiplication rule as encryption. The result of this multiplication is (r0, r1) which correspond to the original plaintext bytes but the encryption writes bytes reversed so during decryption I extended them as [r1, r0].


This loop gradually rebuilt the entire plaintext. When everything was processed I printed the output. The code that worked in the end looked like this:

```
def mul(a, b):
    a0, a1 = a; b0, b1 = b
    return ((a0*b0 + 3*a1*b1) % p, (a0*b1 + a1*b0) % p)

def inv(a):
    a0, a1 = a
    d = pow((a0*a0 - 3*a1*a1) % p, -1, p)
    return ((a0*d) % p, (-a1*d) % p)

ks = (60, 6)
invks = inv(ks)

ct = bytes.fromhex("9813d3838178abd17836f3e2e752a99d5cd3fba291205f90c1d0a78b6eca")
plain = bytearray()

for i in range(0, len(ct), 2):
    block = (ct[i+1], ct[i])
    r0, r1 = mul(invks, block)
    plain.extend([r1, r0])

print(b"nite{" + plain + b"}")
```

## Flag:

```
nite{m10p7rm_d0lu?31_4__Mh7_30mud3l}
```

## Concepts learnt:
This challenge actually made me understand finite-field arithmetic in a very hands on way. Even though the polynomial thing looked complicated, the encryption itself was just a structured way of mixing two bytes using small modular arithmetic.
I also learnt to be careful with byte order because a simple reversal like the .to_bytes() step can completely change how the ciphertext needs to be interpreted.

## Incorrect Tangents

At first I kept thinking I needed big algebra or matrix inversion, but when I stepped back and rewrote the multiplication rule plainly the system became much simpler. I also wasted some time trying to guess the key without using the given plaintext block. Once I used the first block properly the key solved itself.

## Resources:
https://youtu.be/Q_V_itu_kbs?si=QW5RSdYGLVmgfo9R

***

# 3.Quixorte

Clutter, clutter everywhere and not a byte to use.
nc mars.picoctf.net 31890

## Solution:
When I first looked at this challenge the encryption code looked small but there were two layers hiding inside it. The file bytes were first being rotated based on their index and only then they were passed through a sliding XOR step that used an 8 byte random key.


The moment I saw that the original file was a PNG I felt relieved because PNG files always start with the same fixed header. That meant I could use those predictable first bytes to recover the key.


I started by reading the encrypted file and printing the first 8 bytes just to check what I was dealing with.

```
with open("quote.png.enc", "rb") as f:
    data = f.read()

first8 = data[:8]
print(first8)
print(list(first8))
```


Seeing the list printed out as numbers made it easier for me to think about what needed to be undone.


These were my encrypted first bytes:


[101, 81, 10, 83, 97, 220, 190, 117]


Then I manually defined the known PNG header bytes:


89 50 4E 47 0D 0A 1A 0A


The encryption rotated them to the right by (i % 8). So I wrote the same rotation function and applied it:

```
def rotate_right(b, i):
    i = i % 8
    return ((b >> i) | (b << (8 - i))) & 0xFF

png = [0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A]
rotated = [rotate_right(b, i) for i, b in enumerate(png)]
print(rotated)
```


This gave me:


[137, 40, 147, 232, 208, 80, 104, 20]


So now I had the rotated PNG header and the first 8 encrypted bytes. The next idea that came to my mind was the XOR pattern used in the sliding process. Each encrypted byte depends on a running XOR of key bytes. The relation looked like this in my head:


C[0] = R[0] XOR key0

C[1] = R[1] XOR key0 XOR key1

C[2] = R[2] XOR key0 XOR key1 XOR key2


and so on.
Once I realised this structure the key recovery became easy.
I wrote the triangular XOR solve:

```
C = [101, 81, 10, 83, 97, 220, 190, 117]
R = [137, 40, 147, 232, 208, 80, 104, 20]

key = []
running_xor = 0

for i in range(8):
    k = C[i] ^ R[i] ^ running_xor
    key.append(k)
    running_xor ^= k

print(key)
```

This gave me the correct key:


[236, 149, 224, 34, 10, 61, 90, 183]


Once I had the key the rest honestly felt straightforward. I loaded the encrypted file again but this time as a bytearray so I could modify it in-place.

```
with open("quote.png.enc", "rb") as f2:
    data3 = bytearray(f2.read())
```


Then I undid the sliding XOR by simply running the same loops the encryption ran because XOR cancels itself if you apply it twice with the same key.
```
def xoragain(data3, key):
    for i in range(len(data3) - len(key) + 1):
        for j in range(len(key)):
            data3[i+j] ^= key[j]
    return data3

data3 = xoragain(data3, key)
```


Now the bytes were XOR cleaned but they were still rotated. Since the encryption rotated to the right I now had to rotate to the left. So I wrote the left rotation function:
```
def rotate_left(b, i):
    i = i % 8
    return ((b << i) | (b >> (8 - i))) & 0xFF
```

And applied it across the whole file:
```
for i in range(len(data3)):
    data3[i] = rotate_left(data3[i], i)
```

After that I saved the result as recovered.png:
```
with open("recovered.png", "wb") as f2:
    f2.write(data3)
```

Opening recovered.png confirmed that everything had been reversed properly.

### Final Full Decrypt Code
```
with open("quote.png.enc", "rb") as f:
    data = f.read()
first8 = data[:8]
print(first8)
data2 = b'eQ\nSa\xdc\xbeu'
print(list(data2))
    
def rotate_right(b, i):
    i = i % 8
    return ((b >> i) | (b << (8 - i))) & 0xFF
png = [0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A]
rotated = [rotate_right(b, i) for i, b in enumerate(png)]
print(rotated)

C = [101, 81, 10, 83, 97, 220, 190, 117]
R = [137, 40, 147, 232, 208, 80, 104, 20]

key = []
running_xor = 0

   
for i in range(8):
    k = C[i] ^ R[i] ^ running_xor
    key.append(k)
    running_xor ^= k
print( key)
f.close()

with open("quote.png.enc", "rb") as f2:
    data3 = bytearray(f2.read())

def xoragain(data3,key):
    for i in range(len(data3) - len(key) + 1):
        for j in range(len(key)):
            data3[i+j] ^= key[j]

    return data3
data3 = xoragain(data3, key)
def rotate_left(b, i):
    i = i % 8
    return ((b << i) | (b >> (8 - i))) & 0xFF
for i in range(len(data3)):
    data3[i] = rotate_left(data3[i], i)
with open("recovered.png", "wb") as f2:
    f2.write(data3)
```

## Flag:

```
nite{t0_b3-XOR_not_t0_b3333}
```

## Concepts learnt:
The first thing that clicked for me was how predictable file headers can save you when the encryption reuses structure. I also got a clearer idea of how sliding XOR works and why it creates that triangular pattern. Another thing was the rotation idea. Bit rotation always looks trickier than it is but once I saw that I just had to rotate the other way it felt natural. And bytearray helped me understand exactly how Python handles bytes versus characters because in XOR operations only the numeric value matters.


## Incorrect Tangents

At first I tried undoing the entire sliding XOR by hand thinking I would somehow reverse each position separately but then I realised that wasn’t needed and the triangular relationship was already enough for the first 8 bytes. I also briefly tried rotating the encrypted bytes first before undoing the XOR but I caught myself because encryption did rotation first so decryption had to undo XOR first. Another tangent was thinking the bytes printed like characters but I eventually reminded myself they are just numbers from 0 to 255 inside bytearray.

## Resources:
https://www.w3.org/TR/PNG-Structure.html

***

# 4.Willy’s Chocolate Experience 

You’re given two massive numbers generated by a recurrence, and the goal is to peel it back to the hidden exponent powering the whole system. Solving that exponent recovers the final ticket that becomes the flag.

## Solution:

When I first saw the problem, it looked a bit intimidating but as I read the code line by line I started seeing the pattern. Basically, imagination_lab(m) just computes
```
13**m + 37**m mod p
```


for some huge prime p. And the “ticket” is just the flag converted to a big integer using bytes_to_long. The code then builds a list of candies by running imagination_lab(i) for all i from 0 to ticket-1 and appends the real candy at the end. The question gives you the last two values of this candy bag and asks to recover the ticket.


So I thought okay, we have

```
f_prev = imagination_lab(ticket-1)
f_cur = imagination_lab(ticket)
```

and I know the formula:
```
f(m) = 13**m + 37**m mod p
```


 I noticed that if you try to express f_cur in terms of f_prev, you can do something like solving for constants A and B such that
```
f(m) = A * 13**m + B * 37**m
```

Then it becomes a simple modular equation. Using modular inverse, I can solve for B as
```
B = (f_cur - 13*f_prev) * mod_inverse(24, p) % p
```


and then A = f_prev - B. Once I had A, I realized I basically needed to solve

```
13**x = A mod p
```

to find x = ticket-1. That’s a discrete logarithm problem. Normally that’s hard, but I knew I could use sympy.discrete_log which handles it efficiently. So I just ran:
```
x = discrete_log(p, A, 13)
ticket = x + 1
```


Finally, I converted the ticket back to bytes using long_to_bytes to get the flag. It all fell into place and printed the flag in some seconds.

### FINAL PYTHON SCRIPT:
```
from sympy import discrete_log

def long_to_bytes(n):
    return n.to_bytes((n.bit_length() + 7)//8, 'big')

p = 396430433566694153228963024068183195900644000015629930982017434859080008533624204265038366113052353086248115602503012179807206251960510130759852727353283868788493357310003786807
f_prev = 124499652441066069321544812234595327614165778598236394255418354986873240978090206863399216810942232360879573073405796848165530765886142184827326462551698684564407582751560255175
f_cur  = 208271276785711416565270003674719254652567820785459096303084135643866107254120926647956533028404502637100461134874329585833364948354858925270600245218260166855547105655294503224


inv24 = pow(24, -1, p)
B = ((f_cur - 13*f_prev) * inv24) % p
A = (f_prev - B) % p


x = discrete_log(p, A, 13)
ticket = x + 1

flag = long_to_bytes(ticket)
print("Flag:", flag)
```

## Flag:

```
nite{g0ld3n_t1ck3t_t0_gl4sg0w}
```

## Concepts learnt:
1.Using modular inverse and modular arithmetic to solve for unknown constants


2.How to use sympy.discrete_log to recover exponents in modular arithmetic


3.Going from bytes → integer → solving math → back to bytes to get the original message.


## Incorrect Tangents

1.Brute-forcing small powers: Before approaching the discrete log properly, looping over small values of x to see if 13**x % p == A was considered. Even modest values of x were enormous, making brute force impossible.


2.Hesitation using SymPy: SymPy has a discrete_log function, but there was hesitation because of uncertainty about whether it could handle such large numbers.

## Resources:

https://stackoverflow.com/questions/1832617/calculate-discrete-logarithm


***

