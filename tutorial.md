[TOC]

# Core

## xrange

You probably already use generators. Everytime you use an xrange,
you're using a generator.


[[[cog
for i in xrange(1,5):
    print(i)
]]]
[[[end]]]

## ``__iter__`` and ``__next__`` Magic Methods

[[[cog
import time
from timeit import timeit
from itertools import izip

def show_schedule(stream):
    tic = time.time()
    stic = time.time()

    print('%5s  %5s  %2s ' % ('time', 'dt', 'i'))
    for x in stream():
        toc = time.time()
        print('%5.2f  %5.2f  %2s ' % (toc-stic, toc-tic, x))
        tic = toc

def iterator():
    result = []
    for i in range(10):
        time.sleep(0.1)
        result.append(i)
    return result

def generator():
    for i in range(10):
        time.sleep(0.1)
        yield i

print('-- Generator --')
show_schedule( generator )

print('-- Iterator -- ')
show_schedule( iterator )

]]]
[[[end]]]

[[[cog
from timeit import timeit

def blocker():
    yield
    time.sleep(1)
    yield

# The blocking in ``blocker`` bubbles up to this context when
# consumer the generator.
forced_consumer = lambda: [a for a in blocker()]
async_consumer = lambda: (a for a in blocker())

print(timeit(forced_consumer, number=1))
print(timeit(async_consumer, number=1))

]]]
[[[end]]]

This is called forcing the generator.

## Forcing a Generator
[[[cog
def generator():
    yield 1
    yield 2

forced = [a for a in generator()]
notforced = (a for a in generator())

print(type(forced).__name__)
print(type(notforced).__name__)
]]]
[[[end]]]

Here we start to see the beginning of the power of generators in
the we can pass around side-effectful, possibly blocking code to
functions which can operate on their effects but without forcing
the generators. Operating on data in a generally **lazy** manner.

[[[cog

from time import sleep, time

def counter():
    for i in xrange(5):
        sleep(0.2)
        yield i

def doubler(x):
    for j in x:
        yield j
        yield j

def squarer(x):
    for k in x:
        yield k**2 

def printer(x):
    tic = time()
    for l in x:
        print('%.2f %s' % (time() - tic, l))

pipeline = printer(doubler(counter()))

]]]
[[[end]]]

In fact if Unix pipes are more your thing we can build a 'unix'
like syntatic sugar.

[[[cog
class pipe():
    def __init__(self, gen):
        self.gen = gen

    def __or__(self, other):
        return other(self())

    def __call__(self, *args, **kwargs):
        return self.gen(*args, **kwargs)

def counter():
    for i in xrange(25):
        yield i

def printer(x):
    for i in x:
        yield i

producer = pipe(counter)
consumer = pipe(printer)

print([a for a in producer | consumer])

]]]
[[[end]]]
# The Many Flavors of Iterators

[[[cog
def typeof(obj):
    print(type(obj).__name__)

typeof(iter(xrange(5)))
typeof(iter(set([1,2,3])))
typeof(iter([1,2,3]))
typeof(iter({'foo': 'bar'}))

# The very important empty iterator
iter([])

]]]
[[[end]]]

# Enumerate

[[[cog
lst = ['a', 'b', 'c']

print( [a for a in enumerate(lst)] )

for index, value in enumerate(lst):
    print(index, value)
]]]
[[[end]]]

# Iterators vs Generators

[[[cog
x = (a for a in [1,2,3])
y = [b for b in [1,2,3]]

print('Commong Gotcha', x == y)
]]]
[[[end]]]

# Comprehensions

## List Comprehensions

## Dictionary Comprehensions

[[[cog
dct = { a : b for a, b in [('foo', 'bar'), ('fizz', 'pop')] }
print( dct )
]]]
[[[end]]]

## Set Comprehensions

In Python 2.7+ there are also set comprehensionsset
comprehensions.

[[[cog
print({a for a in [1,1,2,3]})
]]]
[[[end]]]

# Generators

## Yield

[[[cog
def generator():
    yield 'a'
    yield 'b'
    yield 'c'

for x in generator():
    print(x)

]]]
[[[end]]]

# Itertools

## ``iter``
## 
