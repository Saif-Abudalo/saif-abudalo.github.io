

## <a href="#challenge-code">Challenge</a>

<p id="challenge-code"></p>

```python
from Crypto.Util.number import getPrime, inverse, bytes_to_long
from random import getrandbits
from secret import flag


nbits, l = 512, 300
p, q = [getPrime(nbits) for _ in range(2)]
N = p * q
e = getPrime(64)
ciphertext=pow(bytes_to_long(flag.encode()),e,N)
rp, rq = [getrandbits(64) for _ in range(2)]

helper1 = (inverse(e, p - 1) + rp * (p - 1)) >> l
helper2 = (inverse(e, q - 1) + rq * (q - 1)) >> l
out=open("output.txt","w")
out.write(f"{N = }\n")
out.write(f"{e = }\n")
out.write(f"{helper1 = }\n")
out.write(f"{helper2 = }\n")
out.write(f"{ciphertext = }\n")
```

#### Output:

```python
N = 88011942360795611244853224204734134208582854307219232067082625111297864456945365607147082297681584005350233869893457643570123454730885047494583011864053047548248835962121303360602075771517638830903464796184999942265738451242309943401604689059520132674906511572471400588787717439184743136103156904254323915901
e = 16670854822985954021
helper1 = 51228035744563254415367316729216547982149018388011869415603928591502380933406640110
helper2 = 79182358136936376315815517710468059332967653867778171178976288236501620385729199198
ciphertext = 42365737111589144866181018205234058566225823457764996523122057072733466446270012975739257908355403632509291004024980906851761155727480185173558931566470213951478880482791442917146639230752203121983275919318630596015865038266436985606561409738239208900358031704076568502711787895554861799920977677370235151077

```



In this challenge, we are given `helper1` and `helper2`. These represent the **Most Significant Bits (MSBs)** of the blinded private keys $\tilde{d}_p$ and $\tilde{d}_q$, which are obscured by random multiples of $(p-1)$ and $(q-1)$:

$$\tilde{d}_p = d_p + r_p(p-1)$$

$$\tilde{d}_q = d_q + r_q(q-1)$$

Since `helper1` and `helper2` are the MSBs (shifted by $l=300$):

$$h_1 = \tilde{d}_p \gg l$$
$$h_2 = \tilde{d}_q \gg l$$

We can rewrite $\tilde{d}_p$ and $\tilde{d}_q$ as:
$$\tilde{d}_p = h_1 \cdot 2^l + x_p$$

$$\tilde{d}_q = h_2 \cdot 2^l + x_q$$

where $x_p$ and $x_q$ represent the unknown **Least Significant Bits (LSBs)**.

----

### Computing $(K, L)$ 

From RSA, we know:

$$e \cdot d_p = 1 + k(p-1)$$

Multiplying the blinded value $\tilde{d}_p$ by the public exponent $e$:

$$e \cdot \tilde{d}_p = e(d_p + r_p(p-1))$$

$$e \cdot \tilde{d}_p = \underbrace{e \cdot d_p}_{1+k(p-1)} + e \cdot r_p(p-1)$$

$$e \cdot \tilde{d}_p = 1 + k(p-1) + e \cdot r_p(p-1)$$

Factoring out $(p-1)$:

$$e \cdot \tilde{d}_p = 1 + (k + e \cdot r_p)(p-1)$$

Let us define a new coefficient $K = k + e \cdot r_p$. For large numbers, we can approximate:

$$e \cdot \tilde{d}_p = 1 + K(p-1) \approx K \cdot p$$

Similarly for $q$, with coefficient $L$:

$$e \cdot \tilde{d}_q \approx L \cdot q$$

Multiply the approximations:

$$(e \cdot \tilde{d}_p) \cdot (e \cdot \tilde{d}_q) \approx (K \cdot p) \cdot (L \cdot q)$$

$$e^2 \cdot \tilde{d}_p \cdot \tilde{d}_q \approx (K \cdot L) \cdot (p \cdot q)$$

$$e^2 \cdot \tilde{d}_p \cdot \tilde{d}_q \approx (K \cdot L) \cdot N$$

We are given the MSBs $h_1$ and $h_2$, so $\tilde{d}_p \approx h_1 2^l$ and $\tilde{d}_q \approx h_2 2^l$. We can compute the approximate product of coefficients, denoted as $A$:

$$A = \left\lfloor \frac{e^2 \cdot (h_1 2^l) \cdot (h_2 2^l)}{N} \right\rfloor + 1 \approx K \cdot L$$

We now have an approximation $A \approx K \cdot L$. To find the exact values, we look at their properties modulo $e$.

From the equation $e \cdot \tilde{d}_p = 1 + K(p-1)$, if we take modulo $e$:

$$1 + K(p-1) \equiv 0 \pmod e \implies K(p-1) \equiv -1 \pmod e$$

$$1 + L(q-1) \equiv 0 \pmod e \implies L(q-1) \equiv -1 \pmod e$$

Recovering $K$ and $L$ via a Quadratic Equation Modulo $e$

$$f(x) = x^2 - (1 - A(N-1))x + A \equiv 0 \pmod e$$

The roots of this equation correspond to $K \pmod e$ and $L \pmod e$.

Since $e$ is relatively small and the actual value of $K$ is much larger than $e$, the root we found $\_k$ only gives us $K \pmod e$. 

Knowing that $A \approx K \cdot L$, we assume the true $K$ is a divisor of $A$. and satisfies $K \equiv \_k \pmod e$.

so $K$ will be:

$$K = \_k + x \cdot e$$
where
$$
x = r_p
$$


### Factoring $N$ 

With $K$ known, we return to the linear relation for $\tilde{d}_p$:

$$e \cdot \tilde{d}_p - 1 - K(p-1) = 0$$

Taking modulo $p$:

$$e \cdot \tilde{d}_p - 1 + K \equiv 0 \pmod p$$

Substituting $\tilde{d}_p = h_1 2^l + x$:

$$e \cdot (h_1 2^l + x) + K - 1 = K \cdot p$$

This gives us a polynomial $f(x)$ where the root $x$ (the LSBs) satisfies:

$$f(x) \equiv 0 \pmod p$$

Since we know the MSBs and $p$ is a factor of $N$, we can use Coppersmith's method to find the small root $x$.

Once $x$ is found, we recover $p$ by computing:

$$p = \gcd(f(x), N)$$



```python
from Crypto.Util.number import * 
from sage.all import *


N = 88011942360795611244853224204734134208582854307219232067082625111297864456945365607147082297681584005350233869893457643570123454730885047494583011864053047548248835962121303360602075771517638830903464796184999942265738451242309943401604689059520132674906511572471400588787717439184743136103156904254323915901
e = 16670854822985954021
h1 = 51228035744563254415367316729216547982149018388011869415603928591502380933406640110
h2 = 79182358136936376315815517710468059332967653867778171178976288236501620385729199198
ct= 42365737111589144866181018205234058566225823457764996523122057072733466446270012975739257908355403632509291004024980906851761155727480185173558931566470213951478880482791442917146639230752203121983275919318630596015865038266436985606561409738239208900358031704076568502711787895554861799920977677370235151077
l = 300


A = (2**600 * e**2 * h1 * h2)//N + 1 
x = PolynomialRing(Zmod(e), 'x').gen() 
f = x^2 - (1 - A*(N - 1))*x + A 
_k, _l = [int(c[0]) for c in f.roots()]


x = 0
divs = divisors(A)
for D in divs:
    if D % e == _k:
        rp = (D - _k) // e
        
        if 0 < rp < 2**70:
            x = rp
            break


k = _k + x*e

R = PolynomialRing(Zmod(k * N), name='x')
x = R.gen()
f = (e * (h1 * 2**l + x) + k - 1)
roots = f.monic().small_roots(X=2**l, beta=0.4)

if roots:
    x0= int(roots[0])
    p = gcd(int(f(x0)), N)
    print(p)

    if N % p == 0:
        q = N // p
        phi = (p - 1) * (q - 1)
        d = inverse(e, phi)
        m = pow(ct, d, N)
        print(long_to_bytes(m))
   
    else:
        print("The found p is not a factor of N.")
else:
    print("No roots found.")
    
    
#flag{bfdf8b993df8fbe4bd528eaf1c70a0c6}

```



**Reference:**  
[Approximate Divisor Multiples Factoring with Only a Third of the Secret CRT-Exponents](https://eprint.iacr.org/2022/271.pdf)
