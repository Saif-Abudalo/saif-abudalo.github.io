---
permalink: /Writeups/CTFJO-FINAL-Cisco-Router-Configuration/
title: "CTFJO FINAL Cisco Router Configuration"
---
<br>




## Challenge Description

We were provided with a Cisco router configuration file and tasked with locating the enable password in plaintext. The configuration contained various encrypted and hashed passwords.

## Challenge File
<a href="{{ '/assets/images/files/CTF-Cisco.pdf' | relative_url }}" target="_blank">
  Challenge File
</a>
in this challenge I was stuck to crack **enable password** because it is hashed in MD5, which is not easily reversible, so cracking this hash was not feasible within the given constraints, especially since the flag was not present in any well-known wordlist like **rockyou**.

There are three passwords in the configuration file:
```
username user1 password 7 08204E4D290C1612005A
line con 0 password 7 08204E4D291A0A1901040001
enable secret 5 $1$mERr$8vWGHtwDX7z6lp2VZQJj81
```

When you configure your router and set the command `enable secret 5` it is used to hash the password using MD5 as shown:

```
enable secret 5 $1$mERr$8vWGHtwDX7z6lp2VZQJj81
```
- `'$1$` →  MD5
- `mERr$8vWGHtwDX7z6lp2VZQJj81` → hash + salt


when you write the command `service password-encryption` while configuring your router it uses (Type 7 Passwords) weak encryption; actually it uses **Vigenere cipher**, which was cracked in 1995. so from here we can start to solve our challenge.

```
username user1 password 7 08204E4D290C1612005A
line con 0 password 7 08204E4D291A0A1901040001
```


There are many online tools  available that can immediately crack Cisco type 7 encrypted passwords. I used this tool when solving the challenge  [Cisco Type 7 Password Cracker](https://cisco-type-7-password-cracker.goffinet.org/?utm_source=chatgpt.com)

#### Decrypted Results:

- `username user1 password 7 08204E4D290C1612005A` → `abc@user1`
- `line con 0 password 7 08204E4D291A0A1901040001` → `abc@console`

The core idea in this challenge is that when the admin configured the password there was a pattern he used which is `abc@xxxxxx`. when he wanted to set a password for line console he used `abc@console` so of course when he tried to set a password for enable mode he used `abc@enable`.


Flag: 
```
abc@enable
```


## Reference

- [Cisco Password Types: Best Practices](https://media.defense.gov/2022/Feb/17/2002940795/-1/-1/1/CSI_CISCO_PASSWORD_TYPES_BEST_PRACTICES_20220217.PDF)



