---
title: "Function Caching"
teaching: 0
exercises: 0
questions:
- "How can I make my script more efficient?"
objectives:
- "Describe what makes a function expensive"
- "Identify pure functions"
- "Demonstrate the use of caching, and explain its advantages"
keypoints:
- "A pure function has no observable effect other than the value it returns, and it returns the same value every time it
is called with the same parameters"
- "Caching a function avoids recomputation when that function is called again with the same parameters"
- "Any pure function can be safely cached, but it is only beneficial to cache expensive functions"
---

## Function Caching
Although we think of computers as lightning quick, I'm sure that we have all been stuck waiting for a computer more times
than we can count. In some fields of computational chemistry, this is actually quite common. If we are running complex simulations
or very heavy computations, the runtime of our scripts starts to add up. There are many ways that we can make our code
faster, and one of those ways is to implement is _function caching_. In function caching, we save the answers to all
of all of our past calculations, so that we can look them up instead of recomputing them. This can save us huge amounts
of time, but there are some important caveats. For a function to be a good candidate for caching, it must be both **expensive** and **pure**. First, we will learn how to identify which functions to cache. Then, we will dive into some code and learn how to implement caching in Python!


## Pure Functions

Our goal in this section is to understand what makes a function _pure_. To get there, we must first learn about _state_.
The state of a program is the contents of everything in memory at a given time. A function can be said to _mutate_ state
if it changes anything besides its local variables. To illustrate this concept, let's look at the function `count_atoms`
below.

~~~
def count_atoms(molecule, counts):
    num_atoms = 0
    for atom in molecule:
        num_atoms += 1
        counts[atom] += 1
    return num_atoms

if __name__ == "__main__":
    counts = {'H': 0, 'O': 0}
    print(count_atoms(['H', 'H', 'O'], counts))
    print(counts)
~~~
{: .language-python}

~~~
3
{'H': 2, 'O': 1}
~~~
{: .output}


`count_atoms` mutates state, because it modifies the variable `counts`, which is not one of its local variables. A quick
check to see if a function mutates state is to see if it modifies anything that is not defined internal to it. Looking
at `count_atoms`, we see the following line, which modifies the `counts` dictionary. Since `counts` is a parameter, this
qualifies as mutating state.

~~~
counts[atom] += 1
~~~
{: .language-python}

One of the effects that a function can have is mutating state. A function can have many different effects, so we divide
them broadly into two categories: _main_ effects and _side_ effects. The main effect is easy: for any function, its main
effect is _always_ its return value. Any observable effects outside of this are called _side effects_. Side effects
include mutating state, writing to files, printing to the console, and more. In short, if a function does anything that
is not a main effect and can be detected from outside the function, either by you, or by your code, then that is a
side effect.

Most functions can be rewritten to get rid of side effects. Lets modify `count_atoms` so that it does not have any side
effects.
~~~
def count_atoms_no_sides(molecule):
    atom_counts = {}
    num_atoms = 0
    for atom in molecule:
        num_atoms += 1
        if atom in atom_counts:
            atom_counts[atom] += 1
        else:
            atom_counts[atom] = 1
    return num_atoms, atom_counts

if __name__ == "__main__":
    total_atoms, type_counts = count_atoms_no_sides(['H', 'H', 'O'])
    print(total_atoms)
    print(type_counts)
~~~
{: .language-python}

~~~
3
{'H': 2, 'O': 1}
~~~
{: .output}

The new function, `count_atoms_no_sides`, does almost the same thing as the old version, but it returns the `atom_counts`
dictionary instead of modifying it. Since it has no effect besides the return statement, `count_atoms_no_sides` has no
side effects!

Having no side effects is part of being a pure function. The other component is that the function must always produce
the same return value when it is given the same parameters. Intuitively, we would expect any function to return the same
value every time it is called with the same parameters, but that is not necessarily the case. We can see this with the
below function.

~~~
global_var = 5

def f(x):
    return global_var + x

print(f"f(5) = {f(5)}")
global_var = 7
print(f"f(5) = {f(5)}")
~~~
{: .language-python}

~~~
f(5) = 10
f(5) = 12
~~~
{: .output}

`f(5)` produces two different results, because `f` depends on the _state_ of the program, and that state was mutated
between calls to `f`. Notice that, by changing `global_var`, we can make `f(5)` equal to any number! Since we need to
know the state of the program in order to know what `f(5)` will evaluate to, we as programmers cannot easily trust `f`.
Writing functions that depend on the state of the program is generally considered bad practice, as it can quickly become
difficult to keep track of what is going on. This makes it much easier to write buggy code.

> ## Why Use Impure Functions?
> Some operations require impure functions, like reading or writing to files.
> If you are interested in avoiding impure functions, there is a whole programming paradigm, called
> [functional programming](https://www.smashingmagazine.com/2014/07/dont-be-scared-of-functional-programming/),
> that heavily favors pure functions. Although Python is not a functional language, many of the ideas from functional
> programming can still be used in Python to write more understandable and testable code.
{: .callout}

> ## Check Your Understanding
>
> Which of the following would be considered "side effects" of a function?
>
> 1. Returning a value
> 1. Printing to the console
> 1. Changing a parameter that is passed into the function
> 1. Creating a new local array
> 1. 2 & 3
> 1. 1 & 4
>
> > ## Solution
> > The correct answer is (5). Recall that the _main effect_ of a function is to return a value. Any other observable effects besides
> > this are considered _side effects_. Creating a new array, or any variable, local to the function is not observable from outside, so it is
> > not a side effect. Modifying a parameter or printing to the console, on the other hand, are both observable, so they are side
> > effects. There are many different types of side effect, but they can all be detected by checking for effects outside of the function.
> {: .solution}
{: .challenge}

## Function Expense - The timeit module

A computer has two primary resources that our code can use up: _time_ and _memory_. Usually, there is a tradeoff between
them: we can write really fast code that uses a lot of memory, really slow code that uses only a small amount of
memory, or find some middle ground. The tuning of these resources is application specific, and you generally don't have
to worry about it until you notice a problem. The _expense_ of any given function is the sum of all of the resources
that it uses. One way to estimate function expense is with the `timeit` module. `timeit` is used to time different code
snippets.

### How to use timeit

To time code using the `timeit` module, you need to import the module, then use `Timer` to time a code snippet. The argument for timer is the code you would like to time as a string.

~~~
import timeit

add = timeit.Timer("x= 5 + 5")
~~~
{: .language-python}

You can then time how long this code takes using

~~~
add_time = add.timeit()
~~~
{: .language-python}

With no argument, `timeit` will execute the code snippet 1,000,000 times and give you the amount of time it took. You can override this to execute fewer or more times:

~~~
# Execute 1000 times
count = 10000
add_time = add.timeit(count)
~~~

If your code snippet requires set-up (like importing a library), you specify that as a second argument to `Timer`. For the following example, our code snippet requires `numpy`, so we add `import numpy` as a second argument:

~~~
np_create = timeit.Timer("numpy.arange(20)", "import numpy")
np_create_time = np_create.timeit(count)
~~~
{: .language-python}

If you want to use `timeit` on a function you have defined, you can use a formatted string when creating your timer. For example, to time your `count_atoms_no_sides` function:

~~~
import timeit

ethane = ['C','H','H','H','C','H','H']

count_atoms_timer = timeit.Timer(f"{count_atoms(ethane)}")

print(count_atoms_timer.timeit(1000)/1000)
~~~
{: .language-python}

### Using timeit to time function expense

Let's try this with a few more examples:

~~~
import timeit
count = 10000

add = timeit.Timer("x = 5+5")
add_time = add.timeit(count)
print(f"Addition:             {add_time/count} sec")

np_create = timeit.Timer("numpy.arange(20)", "import numpy")
np_create_time = np_create.timeit(count)
print(f"Numpy Array Creation: {np_create_time/count} sec")

list_lookup = timeit.Timer("our_list[5] = 43", "our_list = [x for x in range(0, 10)]")
list_lookup_time = list_lookup.timeit(count)

print(f"List Access:          {list_lookup_time/count} sec")

print(f"Numpy array creation took {np_create_time/add_time} times longer than adding two numbers.")
~~~
{: .language-python}

~~~
Addition:             1.6892600000062428e-08 sec
Numpy Array Creation: 9.034520000000157e-07 sec
List Access:          3.797419999997942e-08 sec
Numpy array creation took 53.482116429482545 times longer than adding two numbers.
~~~
{: .output}

The operations varied greatly in the time that they took to complete. Creating an array takes about 50 times longer than
adding two numbers (your timing may vary)! Over time, your intuition about the relative expense of different operations will develop. For now, you can assume that operations that appear more complicated are more expensive. This includes operations like reading
and writing to files or creating large arrays. Keep in mind that even an expensive operation like creating an array only
takes a fraction of a second to complete. Since modern computers are so fast, it is better to write code that is first
and foremost easy to read, and only worry about making it less expensive when you have a good reason to do so.


> ## Check Your Understanding
> Which of these code chunks would we expect to take longer to execute?
>
> ~~~
> import numpy
> arr = numpy.arange(20)
> ~~~
> {: .language-python}
>
> ~~~
> for i in range(0, 1000):
>   x = 5 + 5
> ~~~
> {: .language-python}
>
> > ## Solution
> > The array creation is faster. Although the individual statement `x = 5 + 5` is faster than the array creation, that
> > statement occurs inside a loop. In this case, `x = 5 + 5` is run 1000 times, while `arr = numpy.arange(20)` is only
> > run once. It is important to be cognizant of how
> > many times an operation is likely to occur in order to accurately estimate the expense of a function.
> {: .solution}
{: .challenge}

Now that we understand what makes a function expensive, we can test our intuition on some real functions.

> ## Check Your Understanding
>
> Rank the following functions, which all compute the sum of numbers 1 through n inclusive, in order of least expensive
> to most expensive.
>
> ~~~
> def sum_1(n):
>   sum = 0
>   for i in range(1, n+1):
>       sum += i
>   return sum
> ~~~
> {: .language-python}
>
> ~~~
> def sum_2(n):
>   return (n * (n+1)) / 2
> ~~~
> {: .language-python}
>
> ~~~
> def sum_3(n):
>    sum = 0
>    for i in range(1, n+1):
>        el = numpy.array([i])
>        sum += el[0]
>    return sum
> ~~~
> {: .language-python}
>
> ~~~
> def sum_4(n):
>    sum = 0
>    arr = numpy.arange(n+1)
>    for el in arr:
>        sum += el
>    return sum
> ~~~
> {: .language-python}
>
> > ## Solution
> > *   `sum_2` is the least expensive because it is only performing a few arithmetic operations.
> > *   `sum_1` is more expensive because it performs `n` caculations. However, the individual calculations are very
> > simple, so it is not _too_ bad. 
> > *   `sum_4` is next worst, because it creates a numpy array with `n` elements. This has a significant impact on the time
> > cost of `sum_4`, but it is even more detrimental to the space cost of `sum_4`, as that massive array must be stored somewhere.
> > *   Finally, `sum_3` is the most expensive. Creating an array is an expensive operation, and, since `sum_3` creates a new
> > array for every item in the sum, it creates `n` arrays!
> >
> > For reference, here are the execution times for `n=10000` on my computer, with 1,000 repetitions. If you are ever interested
> > in the execution time of competing code snippets, you can use [the timeit module](https://docs.python.org/3/library/timeit.html).
> >
> > ~~~
> > sum_1:	0.463774397 sec
> > sum_2:	0.00012605599999993 sec
> > sum_3:	8.620602577 sec
> > sum_4:	1.7466740949999995 sec
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

## Caching

_Caching_ is when we store the output for a given set of inputs to a function. Then, when we call the function with the
 same inputs again, we can simply return the saved value, rather than re-run the function. If done properly, caching can
 greatly improve the running time of your scripts.

> ## Two Types of Caching
> There are two types of caching - memory, and disk. In memory caching, the cache is erased when the program terminates.
> In disk caching, the cache is saved to disk, so it persists between runs. Memory caching is generally faster, but, as it
> is not persisted and it cannot work with numpy arrays, we will cover disk caching today. If you are interested in memory
> caching, I encourage you to check out [Python's functools](https://docs.python.org/3/library/functools.html#functools.lru_cache)
{: .callout}

Since we are using disk caching, we have to retrieve the cache from files on the disk. Recall that opening a file has
some significant overhead. If the cost of retrieving the cache is less than the cost of running the function, we are
speeding up our script! However, if the function is less expensive than the cache lookup, we are actually slowing down our
script. Also, caching takes up memory on your computer. For these reasons, it is only beneficial to cache expensive
functions.

To illustrate some limitations of caching, let's modify our `count_atoms` function to use the cache.

First, we import and configure joblib.

~~~
from joblib import Memory

location = './cache'
memory = Memory(location)
~~~
{: .language-python}

We import joblib, then tell it where on our computer to locate the cache. I have set it to `./cache` in the line
`location = './cache'`. This will create a new directory called `cache` in the same directory as my script and populate
it with the cache. You can change this to store your cache files anywhere on your computer.
We then save this configuration by calling `Memory(location)`.

We can tell joblib to cache a function by annotating the function with `@memory.cache` like so:
~~~
@memory.cache
def my_function(args):
    pass
~~~
{: .language-python}

> ## Exercise
> Import and configure joblib to cache our `count_atoms` function. Then, run it twice. What differences do you expect
> between the subsequent runs?
>
> > ## Solution
> > ~~~
> > from joblib import Memory
> >
> > location = './cache'
> > memory = Memory(location)
> >
> > @memory.cache
> > def count_atoms(molecule, counts):
> >     num_atoms = 0
> >     for atom in molecule:
> >         num_atoms += 1
> >         counts[atom] += 1
> >     return num_atoms
> >
> > if __name__ == "__main__":
> >    counts = {'H': 0, 'O': 0}
> >    print(f"Number of atoms: {count_atoms(['H', 'H', 'O'], counts)}")
> >    print(f"Atom Breakdown: {counts}")
> > ~~~
> > {: .language-python}
> >
> > On the first run, below, we can see that `count_atoms` is called.
> > ~~~
> > ________________________________________________________________________________
> > [Memory] Calling __main__.count_atoms...
> > count_atoms(['H', 'H', 'O'], {'H': 0, 'O': 0})
> > ______________________________________________________count_atoms - 0.0s, 0.0min
> > Number of atoms: 3
> > Atom Breakdown: {'H': 2, 'O': 1}
> > ~~~
> > {: .output}
> >
> > On the second run, it is simply retrieved from the cache!
> >
> > ~~~
> > Number of atoms: 3
> > Atom Breakdown: {'H': 0, 'O': 0}
> > ~~~
> > {:. output}
> >
> > Notice, however, that the breakdown of atom type in the second run is incorrect! This is because
> > setting the breakdown dictionary is a _side effect_ of `count_atoms`, and, since `count_atoms` was
> > never actually run, none if its side effects took place! Since a cached function will not always be run, we can
> > only safely cache pure functions.
> {: .solution}
{: .challenge}

> ## Check Your Understanding
>
> What makes a function a good candidate for caching? An ideal function to cache is...
>
> 1. Expensive
> 2. Impure
> 3. Pure
> 4. Called Infrequently
> 6. 1 & 3
> 7. 2 & 4
>
> > ## Solution
> > The correct answer is (6). A function that is both pure and expensive is a good candidate for caching.
> > An impure function can not be cached on account of its side effects, while caching an inexpensive function could negatively impact
> > your program's performance. Other factors that make a function a good choice for caching include if it is called
> > frequently, and if it is likely to be called many times with identical inputs.
> {: .solution}
{: .challenge}

Now, lets apply what we learned about function pureness, expense, and the benefits of caching to analyze some real
functions from past episodes.

> ## Check Your Understanding
>
> Which of these three functions would be good candidates for caching?
>
> ~~~
> # calculate the molecular mass of a molecule given by 'symbols', a list of atoms
>def calculate_molecular_mass(symbols):
>    atomic_weights = {'H': 1.00784,
>                      'C': 12.0107,
>                      'N': 14.0067,
>                      'O': 15.999,
>                      'P': 30.973762,
>                      'F': 18.998403,
>                      'Cl': 35.453,
>                      'Br': 79.904}
>    mass = 0
>    for atom in symbols:
>        mass += atomic_weights[atom]
>    return mass
> ~~~
> {: .language-python}
>
> ~~~
> def bond_check(atom_distance, minimum_length=0, maximum_length=1.5):
>   return minimum_length < atom_distance <= maximum_length
> ~~~
> {: .language-python}
>
> ~~~
> def draw_molecule(coordinates, symbols, draw_bonds=None, save_location=None, dpi=300):
>
>    # Draw a picture of a molecule using matplotlib.
>
>    # Create figure
>    fig = plt.figure()
>    ax = fig.add_subplot(111, projection='3d')
>
>    # Get colors - based on atom name
>    colors = []
>    for atom in symbols:
>        colors.append(atom_colors[atom])
>
>    size = np.array(plt.rcParams['lines.markersize'] ** 2)*200/(len(coordinates))
>
>    ax.scatter(coordinates[:,0], coordinates[:,1], coordinates[:,2], marker="o",
>               edgecolors='k', facecolors=colors, alpha=1, s=size)
>
>    # Draw bonds
>    if draw_bonds:
>        for atoms, bond_length in draw_bonds.items():
>            atom1 = atoms[0]
>            atom2 = atoms[1]
>
>            ax.plot(coordinates[[atom1,atom2], 0], coordinates[[atom1,atom2], 1],
>                    coordinates[[atom1,atom2], 2], color='k')
>
>    # Save figure
>    if save_location:
>        plt.savefig(save_location, dpi=dpi, graph_min=0, graph_max=2)
>
>    return ax
>
> ~~~
> {: .language-python}
>
> > ## Solution
> >
> > *   `calculate_molecular_mass` is the best candidate for caching, because it is pure _and_ expensive.
> > *   `bond_check` could also be cached,
> > but it is such a fast function that it would take longer to look up the cache than it would to recompute the value.
> > *   `draw_molecule`, on the other hand, is expensive, so it is tempting to cache it. However, since `draw_molecule` is not
> > a pure function, we cannot cache it. In this case, `draw_molecule` is impure because it writes an image file to the disk. What
> > would happen if the image file from a previous run were deleted? If we cached this function, it would not be re-run, so a new image file would be
> > not be generated. Since this is not the expected behavior, it is a bug. In general, recall that impure functions have side-effects that other parts
> > of your program may depend on. Since a cached function will only execute when it is called with new parameters, some
> > runs of the program may never actually execute the function. This means that the side-effects that our program depends on
> > may never exist! To avoid introducing bugs when caching, make sure that you only cache _pure_ functions.
> >
> > {: .output}
> {: .solution}
{: .challenge}
