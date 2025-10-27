# ssss+

## problem details

problem name: ssss+  
problem author: wwm  
category: crypto  
description: can you do it again, but with hidden?  
attachments: server.py

```
#!/usr/local/bin/python3
from Crypto.Util.number import *
import random

p = 2**255 - 19
k = 15
SECRET = random.randrange(0, p)

def lcg(x, a, b, p):
    return (a * x + b) % p

pp = getPrime(256)
a = random.randrange(0, pp)
b = random.randrange(0, pp)
poly = [SECRET]
while len(poly) != k: poly.append(lcg(poly[-1], a, b, pp))

def evaluate_poly(f, x):
    return sum(c * pow(x, i, p) for i, c in enumerate(f)) % p

print("welcome to ssss", flush=True)
for _ in range(k - 1):
    x = int(input())
    assert 0 < x < p, "no cheating!"
    print(evaluate_poly(poly, x), flush=True)

if int(input("secret? ")) == SECRET:
    FLAG = open("flag.txt").read()
    print(FLAG, flush=True)
```

## funny stuff

so i tried ssss first (ofc), and i managed to solve it - only to realise that my teammate solved it 1 hour earlier (whoops). 
turns out i overcomplicated ssss. but at least that solution was useful for ssss+!  

i have no idea how intended this solution is, since i approach crypto from a pure math, 0 background crypto knowledge standpoint, 
so my solution could be a bit scuffed. this is also why i got stuck on ssss for so long, because i didn't know how overpowered 
sage was. guess i need to get better at crypto.

## problem analysis

looking at the code, the program will first generate the first $k = 15$ terms of an lfsr (linear-feedback shift register, don't worry i also 
didn't know about this before this contest) with randomly generated $a$, $b$, prime modulo $pp$ and starting value $SECRET$, which we need to find.  

the second part of the program takes in $k - 1 = 14$ queries of $x$, and responds with the polynomial of degree 14 with coefficients of the lfsr sequence. 
interestingly, the program returns the answers modulo $p = 2^255 - 19$, which is different from the prime $pp$ used in the lfsr. after that, we need to 
determine the value of $SECRET$.  

since literally everything from the first part is unknown, i decided to start by finding stuff about the polynomial directly, ignoring the lfsr sequence.

## getting enough of the polynomial

we have $14$ queries to find a degree $14$ polynomial. with gaussian elimination, we can find a degree $m$ polynomial with $m+1$ values. so we are one value short.  

the thing to note here is that we don't need to get the entire polynomial. since the coefficients follow some pattern, we will eventually be able to find 
the value of $SECRET$ anyways.

let $$P(x) = a_0 + a_1 x + a_2 x^2 + \ldots + a_{14} x^{14}$$. the key to note here is that $P(x)$ and $P(-x)$ have similar terms.

if we try summing the 2, we get a new polynomial $$Q(x) = P(x) + P(-x) = $(a_0 + a_0) + (a_1 x - a_1 x) + (a_2 x^2 + a_2 x^2) + \ldots)$$.

Then $$Q(x) = 2(a_0 + a_2 x^2 + a_4 x^4 + \ldots + a_{14} x^{14})$$. Notice that $Q(\sqrt{x})$ is a polynomial of degree 7!

Very sadly, Finding one value of $Q(x)$ requires 2 queries of $P$ of $x$ and $-x$. Thus we have $14/2 = 7$ queries to determine a polynomial 
of degree $7$, which is not possible.

What about taking the different of the 2?

We get $$Q(x) = P(x) - P(-x) = $(a_0 - a_0) + (a_1 x + a_1 x) + (a_2 x^2 - a_2 x^2) + \ldots)$$.

Then $$Q(x) = 2(a_1 x + a_3 x^3 + \ldots + a_{13} x^{13})$$. Now let $R(x) = \frac{Q(\sqrt{x})}{\sqrt{x}}$.

Now $$R(x) = 2(a_1 + a_3 x + \ldots + a_13 x^6)$$. We have $7$ queries to determine a polynomial of degree $6$!

In the oracle, start by querying all integers from $1$ to $7$, then query all integers from $p-7$ to $p-1$.

Set $c_i = \frac{P(i) + P(-i)}{2i} (mod p)$. Then $R(i^2) = c_i$.




