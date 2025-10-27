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
the value of $SECRET$ 




