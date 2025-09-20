---
permalink: /Writeups/NCSC-Chall3-Finals-2023/
title: "NCSC Chall#3 Finals 2023"
mathjax: true
---
<br>

<script type="text/javascript" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

<a href="{{ '/assets/images/files/public_ciphers.txt' | relative_url }}" download>
  Challenge File
</a>


This challenge was really fun! We were given:  

- 100 RSA moduli:

  $$
  N_1, N_2, \dots, N_{100}
  $$

- 100 corresponding ciphertexts:
  
  $$
  ct_1, ct_2, \dots, ct_{100}
  $$

- The public exponent:
  
  $$
  e = 65537
  $$

The flag was encrypted 100 times with different moduli.

## <a href="#solve-code">Solve</a>

At first, I thought about using the **Chinese Remainder Theorem (CRT)** to solve this challenge.

 **CRT Theorem:**  
Suppose you have the following system of congruences:  

$$
\begin{cases}
x \equiv a_1 \pmod{N_1} \\
x \equiv a_2 \pmod{N_2} \\
\vdots \\
x \equiv a_k \pmod{N_k}
\end{cases}
$$  

**Condition for CRT:**  
All moduli must be coprime:  

$$
\gcd(N_i, N_j) = 1 \quad \text{for all } i \neq j
$$


After trying to apply CRT, I realized that this condition did not apply to our moduli. Since the moduli were not coprime, CRT wasn’t going to work here.  
But the good thing is, because this condition didn’t hold, we could take advantage of it: by computing the **GCD** between the moduli, we could find a shared factor. 

$$
p = \gcd(N_1, N_2, \dots, N_k)
$$

Once we had a factor \(p\), we could calculate the corresponding \(q\) for that particular \(N\) and use it to decrypt the ciphertext that was encrypted with that modulus.

$$
q = \frac{N_i}{p}
$$


<p id="solve-code"></p>

```python
 from Crypto.Util.number import long_to_bytes, inverse


def euclidean_gcd(a, b):
    while b:
        a, b = b, a % b
    return a


with open("public_ciphers.txt") as f:
    e = int(f.readline().split("=")[1].strip())
    ns = eval(f.readline().split("=")[1].strip())
    cts = eval(f.readline().split("=")[1].strip())


for i in range(len(ns)):
    for j in range(i+1, len(ns)):
        p = euclidean_gcd(ns[i], ns[j])
        if p != 1:
            print(f"[+] Common factor found between N[{i}] and N[{j}]: p = {p}")
            for idx in (i, j):
                n = ns[idx]
                q = n // p
                phi = (p-1)*(q-1)
                d = inverse(e, phi)
                m = pow(cts[idx], d, n)
                print(f"[+] Decrypted message {idx}: {long_to_bytes(m)}")

#FLAG{sharing_caring?that's_not_caring_for_me}
```


