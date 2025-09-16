---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.3
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Representing Truncatable Primes

What's special about the number `357686312646216567629137`?
If you're a fan of the [Numberphile YouTube channel](https://www.youtube.com/@numberphile),
you may recognize this particular problem framing.
Indeed there is an excellent video on this topic:

<center>
  <iframe 
    width="560" 
    height="315"
    src="https://www.youtube.com/embed/azL5ehbw_24?si=WCQ1OVQuXlY8Ppds"
    title="YouTube video player"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    referrerpolicy="strict-origin-when-cross-origin"
    allowfullscreen>
  </iframe>
</center>


```{admonition} Spoiler Alert!
:class: warning

If you want a comprehensive answer to the motivating question, watch the video
before you continue reading!
```

---

As you've gathered from the video, `357686312646216567629137` is a
*left-truncatable* prime number.
In other words, if you iteratively remove one digit starting from the left-hand
side of the number, and the remaining number after each removal is also prime.

Let's take a moment to double-check that this is true.

```{code-cell}
# Represent the integer as a string so we can use slicing to truncate the value
ltp_candidate = "357686312646216567629137"
```

First, let's check that the number itself is prime:

```{code-cell}
import sympy

sympy.isprime(int(ltp_candidate))
```

Then we can check that every value we get from lopping off the left-most digit
is also a prime number:

```{code-cell}
all(sympy.isprime(int(ltp_candidate[i:])) for i in range(1, len(ltp_candidate)))
```

## Finding Left-truncatable Primes

What's *particularly* about the `357686312646216567629137` is that it is the
largest truncatable prime --- the number of left-truncatable primes is finite.
In fact, there are relatively few of them! Could we develop a procedure to find
*all* the left-truncatable primes?

The video describes just such a procedure: for a given prime, simply add digits
on the left-hand side to create new numbers, then check if they too are prime.
Given the efficient prime test provided by `sympy.isprime`, it is relatively
straightforward to perform an exhaustive search starting from single digit
numbers.

```{code-cell}
from collections import deque  # An efficient FIFO queue

digits = "123456789"
ltps = []

candidates = deque(digits)
while candidates:
    pc = candidates.popleft()
    if sympy.isprime(val := int(pc)):
        ltps.append(val)
        candidates.extend([d + pc for d in digits])
```
