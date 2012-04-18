[TOC]

# Core

Generators and coroutines are a some of the most powerful and
often misunderstood structures in modern Python. Even though both
have been around since Python 2.5, there is very little
good information on their advanced usage.

# Comprehensions

Lets do a quick course in comprehensions, since these will be
constantly in the examples for iterators, generators and
coroutines.

** List Comprehensions **

List comprehensions are powerful. Rather than discuss them in
detail, I'll just give a few examples and assume you're familiar
with them.


** Guarded comprehensions **

List comprehensions can restrict their output by adding guards on
to their end. These can contain arbitrarily complex logic.

[[[cog
print([a for a in xrange(8) if a > 3])
print([a for a in xrange(8) if a > 3 and a != 4])
]]]
[[[end]]]

** Compound Comprehensions **

List comprehensions can compounded to traverse multiple
iterators. This generally considered bad practice as it can
produce very complex code.

[[[cog
combinations = list([
    x + y
    for x in xrange(5)
    for y in xrange(5)
])
print(combinations)
]]]
[[[end]]]

The one exception is if you're working with matrices or other
multi-index data structures.

[[[cog
matrix = [
    [1 , 2  , 3  , 4]  ,
    [5 , 6  , 7  , 8]  ,
    [9 , 10 , 11 , 12] ,
]
print( [[row[i] for row in matrix] for i in range(4)] )
]]]
[[[end]]]

** Side effectful comprehensions **

Comprehensions can also execute arbitrary side effects anywhere
in their construction. Again, also considered bad practice in
most sane scenarios.

[[[cog
lst = []
[lst.append(val) for val in [1,2,3]]
print(lst)
]]]
[[[end]]]

** Dictionary Comprehensions **


In Python 2.7+ you can write comprehensions over dictionaries.

[[[cog
dct = { a : b for a, b in [('foo', 'bar'), ('fizz', 'pop')] }
print( dct )
]]]
[[[end]]]

** Set Comprehensions **

In Python 2.7+ you can also write comprehensions over sete.

[[[cog
print({a for a in [1,1,2,3]})
]]]
[[[end]]]

# Iterators

An iterator is something that can, well be iterated over. Many
operations can be phrased in the language of iteration and many
Python builtins consume iterables as their primary arguments. Its
a very usefull pattern and is used constantly throughout all of
Python.

Many Python builtins are iterable. For example:

[[[cog

for element in [3.141, 2.718, 2.685]:
    print(element)

for letter in "Python":
    print(letter)

for key in {'foo': 1, 'bar': 2}:
    print(key)

]]]
[[[end]]]

Iterators are an interface based around two methods
``__iter__`` and ``next``.  A ``next`` method either returns a
value or raises a Python built-in ``StopIteration`` exception.
Once an iterator has been consumed it is consumed permanently. It
should return ``StopIteration`` for every call thereafter.

[[[cog

it = iter([1,2,3])

try:
    print( it.next() )
    print( it.next() )
    print( it.next() )
    print( it.next() )
except StopIteration:
    print('done')

]]]
[[[end]]]

## ``iter``

The builtin function ``iter`` is the prefix form of
``obj.__iter__()``. It returns the interable interface of the object
if it exists.

This function is implicitly invoked in quite a few language
constructs. For example placing the iteraable as the operand in a
for loop implicitly invokes ``iter``. In this example
``internal`` is the equivelant code that Python is doing behind
the scenes for both ``forloop`` and ``comprehension``.

[[[cog

f = lambda x: x
obj = [1,2,3]

def forloop():
    results = []
    for x in obj:
        results += [f(x)]
    return results

def comprehension():
    return [f(x) for x in obj]

def internal():
    results = []

    it = obj.__iter__()
    while True:
        try:
            results += [f( it.next() )]
        except StopIteration:
            break
    return results

print( forloop() == comprehension() == internal() )
]]]
[[[end]]]

## Custom Iterables

Of course, one is not limited to the builtin iterables, you can
easily define your own.

It is common to refer to a object that returns itself ( i.e. ``self`` )
as an **iterator**, it must also define a ``next``. While a 
object that returns anything else as being **iterable**, it need
not necessarily define a ``next`` method.

[[[cog

class IteratorExample(object):

    def __init__(self, n):
        self.source = [n**2 for n in range(n)]

    def __iter__(self):
        self.i = 0
        return self

    def next(self):
        if self.i >= len(self.source):
            raise StopIteration
        else:
            value = self.source[self.i]
            self.i += 1
            return value

class IterableExample(object):

    def __init__(self, n):
        self.source = [n % 2 for n in range(n)]

    def __iter__(self):
        # Uses a listiterator
        return iter(self.source)

a = IterableExample(5)
b = IteratorExample(5)

print(iter(a))
print(iter(b))

print([x for x in a])
print([x for x in b])

# You get many operations for free!
print(3 in a)
print(filter(lambda x: x>3, b))

]]]
[[[end]]]

## Sortable Iterables

Building sortable objects is a task that shows up enough that it
is worth discussing. For a thorough discussion of orderings see 
[Order Theory for Computer
Scientists](http://matt.might.net/articles/partial-orders/). What
we will construct is a **total-ordering**.

[[[cog
from functools import total_ordering

@total_ordering
class Age:

    def __init__(self, age):
        self.year, self.interval = age.split()

    def __eq__(self, other):
        return(
            self.year     == other.year and
            self.interval == other.interval
        )

    def __lt__(self, other):
        if self.interval == other.interval:
            return self.year < other.year
        else:
            if self.interval == 'BC' and other.interval == 'AD':
                return True
            else:
                return False

    def __repr__(self):
        return ' '.join([self.year, self.interval])

stoneage      = Age('2000 BC')
bronzeage     = Age('600 BC')
enlightenment = Age('1800 AD')

ages = [stoneage , enlightenment, bronzeage ]

print( sorted(ages) )
]]]
[[[end]]]

## Callable Iterators

The little known other syntax for ``iter`` takes two arguments, a
callable and a sentinel. The callable is repeatedly called until 
sentinel it equals ( ``==`` ) the sentinel. This a
``callable-iterator``.

[[[cog

count = 0

def counter():
    global count
    count += 1
    return count

it = iter(counter, 5)
print(type(it))
print(list(it))

]]]
[[[end]]]

In practice this is rarely used, but is sometimes usefull for
reading files and doing other IO style iterations.

[[[cog

# Read from file until we reach EOF
fd = open('/proc/sys/kernel/version')
lines = iter(fd.readline, '')
print(list(lines))

# Read input from the user until we get a done message
iter(raw_input, 'done')

]]]
[[[end]]]

## The Many Flavors of Iterators

[[[cog
def typeof(obj):
    print(type(obj))

typeof(iter(xrange(5)))
typeof(iter(set([1,2,3])))
typeof(iter([1,2,3]))
typeof(iter({'foo': 'bar'}))
typeof(iter((1,2,3)))

# The very important empty iterator
iter([])

]]]
[[[end]]]

## Enumerate

A very usefull function is the builtin enumerate, which can
consume an interable and assign a index to each element.

[[[cog
lst = ['a', 'b', 'c']

print( [a for a in enumerate(lst)] )

for index, value in enumerate(lst):
    print(index, value)
]]]
[[[end]]]


# Generators

There are two points defining characteristics of generators,
their **schedule** and their **values**. Both are controlled by
the keyword ``yield``.  Ostensibly a generator is a function that
when started will produce a scheduled sequence of values which
can be lazily referenced.

The ``yield`` is often confusing to newcomers because it does two
actions, it suspends execution of the generator and
simultaneously returns a value ( possibly ``None`` ). When the
generator is invoked again it resumes execution. There is also no
restriction on the number of yield statements in a generator, in
this sense they are analogous to a the ``return``, but this
comparison is somewhat misleading because they are fundamentally
different actions. A return is a statement, while a yield is an 
expression.

A generator is a special case of the more general type of
structures known as asymmetric coroutines.  The distinction
between a coroutine and a generator is that a coroutine can
accept further arguments after it has been invoked, whereas a
generator cannot. We'll cover coroutines later. For now, here's 
a simple example of a generator:

[[[cog
def generator():
    yield 'a'
    yield 'b'
    yield 'c'

for x in generator():
    print(x)

]]]
[[[end]]]

To illustrate the vast difference between a yield and a return,
lets embed yield expressions in a variety of weird contexts, just
to illustrate a point.

[[[cog
def weird():
    { (yield 'foo'), (yield 'bar') }

    [(yield x) for x in [1,2,3]]

    # more on this later
    ret1  = (yield 'fizz')
    yield ret1

print(list(a for a in weird()))
]]]
[[[end]]]

## Generator Schedules (1)

Lets take a deeper look inside a timed schedule. In this example
we see that the generator "returns" its results on a 0.1 second
interval. The iterator, in turn, returns its results in one large
lump at 1 second ( ``10 x 0.1s`` ).

[[[cog
import time
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
    return iter(result)

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

## Forcing a Generator

**Forcing** a generator causes it to be consumed in its entirety at a
given time. This is is potentially a blocking operation and will
force all the side-effects of the generator to bubble up.

One of the most common ways of forcing a generator is to cast it
into a list or list comprehension.

[[[cog
def generator():
    yield 1
    yield 2

forced = [a for a in generator()]
forced = list(generator())
notforced = (a for a in generator())

print(type(forced))
print(type(notforced))
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


Here we start to see the beginning of the power of generators in
the we can pass around side-effectful, possibly blocking code to
functions which can operate on their effects but without forcing
the generators. Operating on data in a generally **lazy** manner.

[[[cog

from time import sleep, time

def counter():
    for i in xrange(5):
        sleep(0.1)
        yield i

def doubler(x):
    for j in x:
        yield j
        yield j

def printer(x):
    tic = time()
    for l in x:
        print('%.2f %s' % (time() - tic, l))

pipeline = printer(doubler(counter()))

]]]
[[[end]]]

## Generator Expressions

A common mistake is to compare generator expressions with list
comprehensions. In the case of a generator expression the
operator ``==`` will simply default to comparing the memory
pointers of the two generators in question, i.e 
``a == b = id(a) == id(b)``.

[[[cog
x = (a for a in [1,2,3]) # generator expression
y = (b for b in [1,2,3]) # generator expression
z = [c for c in [1,2,3]] # list comprehension

print(type(x))
print(type(z))

print('Common Gotcha', x == y)
print('Common Gotcha', x == z)
print(list(x) == list(y))

]]]
[[[end]]]

For functions that take a single argument, a generator expression
can be passed in without extra parenthesis. For multiple argument
functions they are still require

[[[cog
# Not required
print( sum(x*x for x in range(10)) )

# Required
print( map(lambda x: x+1, (a for a in xrange(5))) )

]]]
[[[end]]]

## Concurrent List Comprehensions

## Itertools

Inside the standard library there is a very powerful library
called ``itertools`` which contains a variety of utilities
dealing with iterators and generators. I can't hope to possibly
cover all the use cases, refer to Doug Hellemmans excellent
description of the library instead.

But to just give you a taste of its power, lets infinite list can
be constructed and sliced usings itertools methods.

[[[cog
from itertools import count, islice

integers = count()
slice = islice(integers, 10)
print( list(slice) )

]]]
[[[end]]]

## Generator Schedules (2)

The important concept in studying generators is that they execute
through blocks of code on a schedule. We see hints of it
becoming powerful control flow system. More of this later when we
look at coroutines.

[[[cog

def generator():
    print('[Inside] Hello from inside')
    yield
    print('[Inside] hello again from inside')
    yield

a = generator()
print('[Outside] Hello from outside')
a.next()
print('[Outside] Hello again outside')
a.next()
]]]
[[[end]]]

Or in this example, a "parallel" executor.

[[[cog

from itertools import izip

def generator(name):
    print('[%s] task 1' % name)
    yield
    print('[%s] task 2' % name)
    yield

# Run tasks in "parallel"
t1 = generator('foo')
t2 = generator('bar')

for t in izip(t1,t2):
    print('[Scheduler] Switching')

]]]
[[[end]]]

## Context Managers

One curious use case for generators is to provide a terse syntax
for creating context managers. In this case the
``@contextmanager`` decorator exectes the side effects before
the yield block as the ``__enter__`` method in the context
manager. The same idea with the code after the yield, it
corresponds to the cleanup invoked in the ``__exit__`` method.

[[[cog
from contextlib import contextmanager

@contextmanager
def trace():
    print('started')
    yield
    print('exited')

with trace():
    for i in xrange(5):
        print(i)

]]]
[[[end]]]

## Generator Internals

[[[cog
from sys import _getframe

def gen():
    yield
    print('genframe', id(_getframe(0)), 'parentframe', id(_getframe(0).f_back))
    yield
    print('genframe', id(_getframe(0)), 'parentframe', id(_getframe(0).f_back))
    yield
    print('genframe', id(_getframe(0)), 'parentframe', id(_getframe(0).f_back))

def call():
    a = gen()
    a.next()
    print('callframe', id(_getframe(0)), 'parentframe', id(_getframe(0).f_back))
    a.next()
    print('callframe', id(_getframe(0)), 'parentframe', id(_getframe(0).f_back))
    a.next()
    print('callframe', id(_getframe(0)), 'parentframe', id(_getframe(0).f_back))

call()
]]]
[[[end]]]

[[[cog
from dis import dis

def gen():
    yield

gr = gen()

def call():
    gr.next()
    gr.next()

dis(call)
]]]
[[[end]]]

# Coroutines

The best way to think about coroutines is as a new type of
control-flow structure, similar to the way you think about 
loops or if-else statements. This is probably the biggest
cognitive leap in understanding them.

To help with this, let us consider the special cases of
coroutines: subroutines and generators.

Coroutines have ``(N, M)``:

* N entry points
* Take M inputs

Subroutines have ``( N=1, M=1 )``:

* 1 entry point
* Take only one input

Generators have ``( N, M=0 )``:

* N entry points
* Take no input

## send

[[[cog

def recv1():
    value = (yield)
    yield value

def recv2():
    yield (yield) + 1

r1 = recv1()
print( r1.next() )
print( r1.send('Hi') )

r2 = recv2()
print( r2.next() )
print( r2.send(1) )

]]]
[[[end]]]

## next

The observant reader will notice that the distinction between
send and next are actually the same operation. Indeed ``next`` is
in fact functionally equivelant to ``send(None)``. 

[[[cog
def recv():
    yield 1
    yield 2

r = recv()
print( r.next() )
print( r.next() )

r = recv()
print( r.send(None) )
print( r.send(None) )

r = recv()
print( r.next() )
print( r.send('ignored as well') )
]]]
[[[end]]]


[[[cog
def gen():
    yield (yield)

r = gen()

try:
    r.send('i will fail!')
except TypeError as exc:
    print(exc)

r = gen()
r.send(None)
print('ok')

r = gen()
r.next()
print('ok')

]]]
[[[end]]]


It is worth noting that you cannot send a non-null value into a
just-started generator, you must either send ``None`` or
equivelantly invoke ``next``. Not doing so will throw a
``TypeError``

## throw

[[[cog
def gen():
    try:
        yield
    except Exception as exc:
        print("Caught exception inside")

r = gen()
r.next()

try:
    r.throw(Exception)
except Exception as exc:
    print("Caught exception outside")

]]]
[[[end]]]

## close

Again, the observant reader will notice that ``close`` is a
speical case of ``throw``. It simply corresponds to
``throw(GeneratorExit)``.

[[[cog

def gen():
    for i in xrange(5):
        yield i

r = gen()
r.next()
r.close()

# -- equivelantly --

gr = gen()

for a in gr:
    if a == 3:
        gr.send(GeneratorExit)
    else:
        print(a)

]]]
[[[end]]]

## Examples

[[[cog
def subroutine(x, y):
    result = x + y
    return result

def cosubroutine():
    x,y = (yield)
    result = x + y
    yield result

args = (1,2)

# Subroutine Style
# ----------------
result = subroutine(*args)
print(result)

# Coroutine Style
# ---------------
sub = cosubroutine()
sub.next()              # start the coroutine
result = sub.send(args) # send the values into it
print(result)

]]]
[[[end]]]

Generators are also trivial to implement in coroutine format. In
fact because of the Python implementation of coroutines as
enhanced generators they are exactly the same syntax.

The one pattern that you see frequently in the above code is that
we have an extraneous ``next()`` statement to kickstart the
coroutine. This is superflous and can be eliminated using a
clever decorator.


[[[cog
def coroutine(f):
    def start(*args,**kwargs):
        co = f(*args,**kwargs)
        co.next()
        return co
    return start

@coroutine
def cosubroutine():
    x,y = (yield)
    result = x + y
    yield result

sub = cosubroutine()
print( sub.send((1,2)) )

]]]
[[[end]]]

## Trampoline Scheduler

## Greenlets

## Pep 380

[Pep 3800](http://www.python.org/dev/peps/pep-0380/)
