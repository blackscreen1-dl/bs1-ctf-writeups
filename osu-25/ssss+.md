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

we have $14$ queries to find a degree $14$ polynomial. with either lagrange interpolation or gaussian elimination, we can find a degree $m$ polynomial with $m+1$ values. so we are one value short.  

the thing to note here is that we don't need to get the entire polynomial. since the coefficients follow some pattern, we will eventually be able to find 
the value of $SECRET$ anyways.

let $P(x) = a_0 + a_1 x + a_2 x^2 + \ldots + a_{14} x^{14}$. the key to note here is that $P(x)$ and $P(-x)$ have similar terms.

if we try summing the 2, we get a new polynomial $Q(x) = P(x) + P(-x) = $(a_0 + a_0) + (a_1 x - a_1 x) + (a_2 x^2 + a_2 x^2) + \ldots)$.

then $Q(x) = 2(a_0 + a_2 x^2 + a_4 x^4 + \ldots + a_{14} x^{14})$. Notice that $Q(\sqrt{x})$ is a polynomial of degree 7!

very sadly, Finding one value of $Q(x)$ requires 2 queries of $P$ of $x$ and $-x$. thus we have $14/2 = 7$ queries to determine a polynomial 
of degree $7$, which is not possible.

What about taking the different of the 2?

we get $Q(x) = P(x) - P(-x) = $(a_0 - a_0) + (a_1 x + a_1 x) + (a_2 x^2 - a_2 x^2) + \ldots)$.

then $Q(x) = 2(a_1 x + a_3 x^3 + \ldots + a_{13} x^{13})$$. Now let $R(x) = \frac{Q(\sqrt{x})}{\sqrt{x}}$.

now $R(x) = 2(a_1 + a_3 x + \ldots + a_13 x^6)$. We have $7$ queries to determine a polynomial of degree $6$!

in the oracle, start by querying all integers from $1$ to $7$, then query all integers from $p-7$ to $p-1$.

set $c_i = \frac{P(i) + P(-i)}{2i} (mod p)$. Then $R(i^2) = c_i$.

i did this in sage by generating the list of numbers i would throw in:

```
p = 2**255 - 19
k = 15
tos = []
for i in range(7):
    tos.append(i+1)
for i in range(7):
    tos.append(p-i-1)
print("\n".join([str(_) for _ in tos]))
```

then throwing the result into a variable called `res`:

```
pts = []
for i in range(7):
    pts.append([(i+1)^2, ((res[i]-res[i+7])*pow(2*(i+1), -1, p))%p])
```

now, we need to fit the polynomial $R$. we do this using lagrange interpolation, which is available on sage.

```
F = GF(p)
R = F['x']
P = R.lagrange_polynomial(pts)
c = P.coefficients(sparse=False)

# all the coefficients of P are still mod p, so we convert them to normal integers
for i in range(len(c)): c[i] = int(c[i])
```

now, we have all the odd-indexed values of the original polynomial in $c$! note that we are now out of the world of $p$, so all future things will be in terms of $pp$.

but we still need to figure out what $pp$ is to make the modulo any useful.

we can probably try using some facts about the lfsr function. note that one element of $c$ is used to generate the next element. it follows the following relation:

$c_{n+1} = a^2c_n + m$, where $m = ab + b$

hmm, we need a way to figure out what the modulo is, which would be easy if we knew both sides of the equation. what happens if we consider the next term as well?

$c_{n+2} = a^2c_{n+1} + m$

seems like we can cancel out the $m$ here, if we subtract the 2 equations.

$c_{n+2} - c{n+1} = a^2(c_{n+1} - c_n)$

now only $a$ is left... can we cancel it out again? use the next term:

$a^2(c_{n+2} - c_{n+1} = c_{n+3} - c{n+2}$

here, we intentionally flip the sides so that we can multiply the 2 equations together. since $a$ cannot be divisible by $pp$,

$(c_{n+2} - c{n+1})^2 = (c_{n+1} - c_n)(c_{n+3} - c{n+2})$

now we know both sides of the equation, so we get $pp$ divides the difference of the equation!

$pp | (c_{n+2} - c{n+1})^2 - (c_{n+1} - c_n)(c_{n+3} - c{n+2})$

note that we have 6 values of $c$, so we can recover 4 numbers that $pp$ can divide. if we take the gcd, we are quite likely to end up with $pp$!

```
for i in range(1, len(c)): adj.append(int(c[i]) - int(c[i-1]))
divs = []
for i in range(1, len(adj)-1): divs.append(adj[i]^2 - adj[i-1]*adj[i+1])
print(divs)
pp = divs[0]
for i in range(1, len(divs)): pp = gcd(pp, divs[i])
```

ok, so now we have $pp$, how do we get $a$, $b$ and finally $SECRET$?

well, we can start with finding $a$, since we already have from earlier:

$a^2(c_{n+2} - c_{n+1} = c_{n+3} - c{n+2}$

we can find the value of $a^2$ modulo $pp$. so we need to find the square root of $a^2$ modulo $pp$!

i don't know any systematic way to do this, but for certain types of $pp$, there are quite fast solutions. i just happened to face a case where $pp$ was $3$ mod $4$, which is an easy case. i may have rerolled for another $pp$ if i didn't get this.

let $p = 4n + 3$.

$r^2 = a \implies a^{2n+1} = r^{2(2n+1)} = 1$ all modulo $pp$, which is a famous result.

since $a^{2n+1} = 1$, then $a^{2n+2} = a$ and thus $a^{n+1} = r$, all modulo $pp$.

note that to divide by $k$, we need to use `pow(k, -1, pp)` for the mod inverse.

```
k = (pp-1)//2
tafa2 = ((c[2]-c[1])*pow(c[1]-c[0], -1, pp))%pp
a = pow(tafa2, (k+1)//2, pp)
```

now we can get $b$:

$c_{n+1} = a^2c_n + ab + b \implies b = \frac{c_{n+1} - a^2c_n}{a+1}$.

finally, $au + b = c_0 \implies u = \frac{c_0 - b}{a}$.

```
b = ((a*a*c[0] - c[1])*pow(-a-1, -1, pp))%pp
u = ((c[0]-b)*pow(a, -1, pp))%pp
print(a, b, u)
```

testing locally one more time using the values of $a$, $b$ and $u$ (since technically $-a$ is still a possible solution$, we verify that the numbers we get are correct. by throwing $u$ at the oracle, we get the flag. yay!
