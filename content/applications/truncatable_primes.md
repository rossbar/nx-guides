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

What's *particularly* interesting about the `357686312646216567629137` is that it is the
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

We know `ltps` contains *all* possible left-truncatable primes because we've
exhaustively checked the primality of every possible value.
Nevertheless, it's always a good idea to double-check our work.
As usual when presented with a sequence of numbers, the [OEIS](https://oeis.org)
is an excellent resource for learning more.
The left-truncatable primes are described in entry [A024785](https://oeis.org/A024785).
The first thing we notice is that there are 4,260 values in this sequence.

```{code-cell}
len(ltps) == 4260
```

How does our sequence match up to the expected result:

```{code-cell}
import pandas as pd

# Use pandas to read directly from OEIS
df = pd.read_csv(
    "https://oeis.org/A024785/b024785.txt", names=("idx", "prime"), delimiter=" "
)

# Extract the primes from the dataframe and explicitly convert to Python ints
# to avoid any issues with representing large integers
expected = [int(p) for p in df["prime"].tolist()]

# The OEIS sequence is in ascending order, our sequence is not (necessarily)
sorted(ltps) == expected
```

## Representing Truncatable Primes

So far so good, but... what does any of this have to do with networks?
So far all we've done is implement a procedure to provide us a list of
left-truncatable primes.
We can think about individual digits as nodes - then, as we increase the place
value (e.g. go from the "one's place" to the "ten's place") and search for
candidates, we are searching for potential edges to downstream nodes that also
represent prime numbers.

Let's modify our original procedure to produce this directed tree instead of
a simple list.

```{code-cell}
import networkx as nx

ltp_tree = nx.DiGraph()
# Add a "root" node from which our tree will grow
root = ""
ltp_tree.add_node(root, value="")

# Modify the candidate queue to store the "parent" number and the digit to be
# prepended to it resulting in the new number
candidates = deque((root, d) for d in digits)
while candidates:
    parent, digit = candidates.popleft()
    if sympy.isprime(child := int(digit + str(parent))):
        ltp_tree.add_node(child, value=digit)
        ltp_tree.add_edge(parent, child)
        candidates.extend([(child, d) for d in digits])
```

We expect this graph to have 4,261 nodes - one node for each left-truncatable
prime, plus the "root" node that we added ourselves:

```{code-cell}
print(ltp_tree)
```

We see there are also 4,260 edges --- this too matches our expectation, as there
should be exactly one edge connecting each "child" prime to the "parent" from
which it derives.
In graph theory terms that means our `ltp_tree` is an [arborescence][nx-arborescence],
as every node in the tree has an in-degree of one:

[nx-arborescence]: https://networkx.org/documentation/stable/reference/algorithms/generated/networkx.algorithms.tree.recognition.is_arborescence.html

```{code-cell}
nx.is_arborescence(ltp_tree)
```

What other properties do we expect our `ltp_tree` to have?
Given how we constructed the tree, we'd expect it to have layers corresponding
to the number of digits in the prime.
We can use BFS to capture the hierarchical structure of our `ltp_tree`.

```{code-cell}
layers = list(nx.bfs_layers(ltp_tree, sources=root))

# View the first few layers
for n, lyr in enumerate(layers[:4]):
    print(f"All {n}-digit LTPs: {lyr}")
```

The the total number of digits in the largest LTP should be equal to the number
of layers in our `ltp_tree` (minus 1 to account for our "root" layer)

```{code-cell}
largest_ltp = str(max(ltps))
len(largest_ltp)
```

```{code-cell}
len(layers) - 1
```


We should be able to "reconstruct" any prime by traversing the 
