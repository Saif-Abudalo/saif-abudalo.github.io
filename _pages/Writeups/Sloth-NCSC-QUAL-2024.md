---
permalink: /Writeups/Sloth-NCSC-QUAL-2024/
title: "Sloth – NCSC QUAL 2024"
---
<br>


## Challenge Description

Being lazy to come up with a nice prime generator but 16 bits is all I need to safely guard my  private key.

## <a href="#challenge-code">Challenge</a>

<p id="challenge-code"></p>

```python
from Crypto.Util.number import *
from random import randint
from sympy import nextprime,isprime
import os
from secret import *
prime_list=[32771]

while len(bin(prime_list[-1]))==18 and len(prime_list)!=200:
    prime_list.append(nextprime(prime_list[-1]))

def prod(List):
    res=1
    for  i in List:
        res*=i
    return res

def RandomPrimeGen(NumList=prime_list,elementbitlenght=16,bitlenght=128):
    k=1+(bitlenght//elementbitlenght)
    while True:
        used=[]
        gen_List=[]
        while len(gen_List)<=k:
            t=randint(0,len(NumList)-1)
            if not t in used:
                gen_List.append(NumList[t])
                used.append(t)
        p=2*prod(gen_List)+1
        if isprime(p):
            return p

def genPublicKey():
    while True:
        p=RandomPrimeGen()
        q=RandomPrimeGen()
        if p != q:
            return p,q
p,q=genPublicKey()
n=p*q
phi=(p-1)*(q-1)
while True:
    try:
        choices=input("1-Encrypt\n2-verify\n>")
        if choices=="1":
            plain=int(input("give me plaintext: "))
            e=int(input("give me your exponent: "))
            assert e>2**100 and plain>2**100
            result=pow(plain,e,n)
            if result==1:
                print("Ping")
            else:
                print("Pong")
        elif choices == "2":
            guess_phi=int(input("guess my phi : "))
            if guess_phi==phi:
                print(flag)
            else:
                print("Not today")
        else:
            print("Dont do anything sus:")
            break
    except:
        break
```


## <a href="#solve-code">Solve</a>

First, the challenge gives us a small prime, 32771, and from there it keeps generating the next primes until we have a full list of 16‑bit primes. Basically, it’s building a prime list that will later be used to construct `p` and  `q`

```python 
prime_list=[32771]

while len(bin(prime_list[-1]))==18 and len(prime_list)!=200:
    prime_list.append(nextprime(prime_list[-1]))
```

So, the `prime_list` we end up with fixed list of 200 primes:

```
prime_list = [32771, 32779, 32783, 32789, 32797, 32801, 32803, 32831, 32833, 32839, 32843, 32869, 32887, 32909, 32911, 32917, 32933, 32939, 32941, 32957, 32969, 32971, 32983, 32987, 32993, 32999, 33013, 33023, 33029, 33037, 33049, 33053, 33071, 33073, 33083, 33091, 33107, 33113, 33119, 33149, 33151, 33161, 33179, 33181, 33191, 33199, 33203, 33211, 33223, 33247, 33287, 33289, 33301, 33311, 33317, 33329, 33331, 33343, 33347, 33349, 33353, 33359, 33377, 33391, 33403, 33409, 33413, 33427, 33457, 33461, 33469, 33479, 33487, 33493, 33503, 33521, 33529, 33533, 33547, 33563, 33569, 33577, 33581, 33587, 33589, 33599, 33601, 33613, 33617, 33619, 33623, 33629, 33637, 33641, 33647, 33679, 33703, 33713, 33721, 33739, 33749, 33751, 33757, 33767, 33769, 33773, 33791, 33797, 33809, 33811, 33827, 33829, 33851, 33857, 33863, 33871, 33889, 33893, 33911, 33923, 33931, 33937, 33941, 33961, 33967, 33997, 34019, 34031, 34033, 34039, 34057, 34061, 34123, 34127, 34129, 34141, 34147, 34157, 34159, 34171, 34183, 34211, 34213, 34217, 34231, 342534757]
```

### Generating `p` and `q`

The primes p and q are generated as:

$$
p = 2 \cdot \prod_{i=0}^{9} p_i + 1, \quad
q = 2 \cdot \prod_{i=0}^{9} q_i + 1
$$

where

$$
p_i, q_i \in \text{prime list}
$$

Then:

$$
\phi(n) = (p-1) \cdot (q-1) = (2 \cdot \prod_{i=0}^{9} p_i) \cdot (2 \cdot \prod_{i=0}^{9} q_i) = 4 \cdot \prod_{i=0}^{9} p_i \cdot \prod_{i=0}^{9} q_i
$$

Euler’s theorem:

$$
x^{\phi(n)} \equiv 1 \pmod{n}
$$

where 
$$
x
$$
is coprime with 
$$
n
$$.

In the challenge, we defined:

$$
y = 4 \cdot \prod_{i=0}^{199} p_i
$$

and for each prime 
$$
i
$$
in the list, we tested:

$$
x^{y/i} \mod n
$$

- If
$$
i \mid \phi(n)
$$
, then 

$$
x^{\phi(n)/i} \not\equiv 1 \pmod{n} \quad \Rightarrow \text{Pong}
$$

- If
$$
i \nmid \phi(n)
$$
, then 

$$
x^{y/i} \equiv 1 \pmod{n} \quad \Rightarrow \text{Ping}
$$

By iterating through all primes in the list and multiplying those that returned Pong, we reconstructed:

$$
\phi(n) = 4 \cdot \prod_{i=0}^{9} p_i \cdot \prod_{i=0}^{9} q_i
$$



Finally, we send our reconstructed φ(n) to the server, and it returns the flag.
 

<p id="solve-code"></p>

```python
from pwn import*
from Crypto.Util.number import *
from tqdm import *

io = remote("<target-host>", <target-port>)
x = getPrime(300)

def prod(List):
    res=1
    for  i in List:
        res*=i
    return res
    
def o(i):
    io.read()
    io.sendline(b'1')
    io.read()
    io.sendline(str(x).encode())
    io.read()
    io.sendline(str(y//i).encode())
    return b'Pong' in io.readline()

prime_list = [32771, 32779, 32783, 32789, 32797, 32801, 32803, 32831, 32833, 32839, 32843, 32869, 32887, 32909, 32911, 32917, 32933, 32939, 32941, 32957, 32969, 32971, 32983, 32987, 32993, 32999, 33013, 33023, 33029, 33037, 33049, 33053, 33071, 33073, 33083, 33091, 33107, 33113, 33119, 33149, 33151, 33161, 33179, 33181, 33191, 33199, 33203, 33211, 33223, 33247, 33287, 33289, 33301, 33311, 33317, 33329, 33331, 33343, 33347, 33349, 33353, 33359, 33377, 33391, 33403, 33409, 33413, 33427, 33457, 33461, 33469, 33479, 33487, 33493, 33503, 33521, 33529, 33533, 33547, 33563, 33569, 33577, 33581, 33587, 33589, 33599, 33601, 33613, 33617, 33619, 33623, 33629, 33637, 33641, 33647, 33679, 33703, 33713, 33721, 33739, 33749, 33751, 33757, 33767, 33769, 33773, 33791, 33797, 33809, 33811, 33827, 33829, 33851, 33857, 33863, 33871, 33889, 33893, 33911, 33923, 33931, 33937, 33941, 33961, 33967, 33997, 34019, 34031, 34033, 34039, 34057, 34061, 34123, 34127, 34129, 34141, 34147, 34157, 34159, 34171, 34183, 34211, 34213, 34217, 34231, 34253, 34259, 34261, 34267, 34273, 34283, 34297, 34301, 34303, 34313, 34319, 34327, 34337, 34351, 34361, 34367, 34369, 34381, 34403, 34421, 34429, 34439, 34457, 34469, 34471, 34483, 34487, 34499, 34501, 34511, 34513, 34519, 34537, 34543, 34549, 34583, 34589, 34591, 34603, 34607, 34613, 34631, 34649, 34651, 34667, 34673, 34679, 34687, 34693, 34703, 34721, 34729, 34739, 34747, 34757]

y = 4*prod(prime_list)
phi = 4
for i in tqdm(prime_list):
    r = o(i)
    if r:
        phi *= i

print(io.read())
io.sendline(b'2')
print(io.read())
io.sendline(str(phi).encode())
print(io.readline())

#flag{8b3fd475ce65757c0c4d0453f3e4f996}
```
<script type="text/javascript" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

