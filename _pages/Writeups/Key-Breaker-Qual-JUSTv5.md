---
permalink: /Writeups/Key-Breaker-Qual-JUSTv5/
title: "Key Breaker Qual-JUSTv5"
---
<br>


## Challenge

In this challenge, the server sends out a **PEM‑encoded RSA key** When connecting with $nc$ the server prints:

```
b'-----BEGIN PRIVATE KEY-----\nMIIEvAIBADANBgkqhkiG9w0BAQEFAASCBK...'
Enter p> 153619693101654234530891765989459...
Enter q> 136762398106174401793914029...
```


## <a href="#solve-code">Solve</a>

Because the server provides a PEM block beginning with <div style="text-align: center; font-family: monospace; margin: 10px 0;"> -----BEGIN PRIVATE KEY----- </div>
it contains the full RSA key material  $n, e, d, p, q, dp, dq$. All we need to do is parse the PEM, extract the values of $p$ and $q$, and send them back to the server until we get our flag.

<p id="solve-code"></p>

```python
from Crypto.PublicKey import RSA
from pwn import remote

def main():

    ##### socket ####
    host = '<target-host>'
    port = <target-port>

    io = remote(host, port)

    while True:
        out = io.recvline()
        print(out)
        received_key = out.strip()[2:-1]

        endline = b'\n'
        key = received_key.replace(b'\\n', endline)

        try:
            key_obj = RSA.import_key(key)
        except ValueError as e:
            print("Error importing RSA key:", e)
            return

        p = key_obj.p
        q = key_obj.q

        print(p)
        print(q)
        print(p*q == key_obj.n)

        io.sendlineafter(b'Enter p> ', str(p).encode())
        io.sendlineafter(b'Enter q> ', str(q).encode())

if __name__ == "__main__":
    main()

#JUST{d2a4da20863a42a7a75eb466a3e3f730}
```

<script>
  window.MathJax = {
    tex: {
      inlineMath: [['$', '$'], ['\\(', '\\)']]
    }
  };
</script>

<script type="text/javascript" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>
