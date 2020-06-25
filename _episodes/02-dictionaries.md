---
title: "Dictionaries"
teaching: 0
exercises: 0
questions:
- "How can we best fit Python's data structures to our objectives?"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

## Dictionaries
In Python, a dictionary is an unordered collection of key-value pairs. Every key in a dictionary must be unique.
A dictionary is useful when you care about the
relationship between a key and value, and do not care about the ordering of the key-value pairs. One good example of this
is the atom names and coordinates from a pdb file. While we could use two arrays, one for the atom names, and one for the
coordinates, this system is fragile! What would happen if we deleted a coordinate from the coordinates array, but forgot
to delete the matching atom name? The rest of the data would be off by one, so we would get completely incorrect coordinates!
A dictionary is a better choice for this type of data.

To create an empty dictionary, use empty curly braces like so.
~~~
my_dict = {}
~~~
{: .language-python}

To create a populated dictionary, use curly braces with some key-value pairs inside. The pattern is `key: value`. Lets
create a dictionary to hold the atoms that make up water.
~~~
water = {'H': 2, 'O': 1}
~~~
{: .language-python}

Since values are stored by key, we access the value by looking up the key in the dictionary using square brackets.
~~~
water = {'H': 2, 'O': 1}
print(water['H'])
~~~
{: .language-python}
~~~
2
~~~
{: .output}

> ## Check Your Understanding
>
> Create a new dictionary and assign the key `hello` to the value `world`.
>
> > ## Solution
> >
> > There are a few ways to solve this problem. One is to create an empty dictionary,
> > then set the value of the key `hello`
> > ~~~
> > my_dict = {}
> > my_dict['hello'] = 'world'
> > print(my_dict)
> > ~~~
> > {: .language-python}
> >
> > ~~~
> > {'hello': 'world'}
> > ~~~
> > {: .output}
> >
> > Another, more common way, is to initialize the dictionary with some default key value pairs.
> >
> > ~~~
> > my_dict = {'hello': 'world'}
> > print(my_dict)
> > ~~~
> > {: .language-python}
> >
> > ~~~
> > {'hello': 'world'}
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

If we try to look up a key that doesn't exist, we get a `KeyError`.
~~~
water = {'H': 2, 'O': 1}
print(water['N'])
~~~
{: .language-python}
~~~
KeyError: 'N'
~~~
{: .error}

To avoid this error, it is a good idea to check for a key's existence in a dictionary before attempting to access it.
This is similar to how you wouldn't try to access the 50th element of a list without knowing that the list is at least 50
elements long! To check if a key exists, we use the `in` operator.
~~~
water = {'H': 2, 'O': 1}
if 'N' in water:
    print(water['N'])
else:
    print('Yep, no Nitrogen in water!')
~~~
{: .language-python}
~~~
Yep, no Nitrogen in water!
~~~
{: .output}

What if we want to use every key in a dictionary? Dictionaries are iterable, so we can use them in a `for` loop.
~~~
water = {'H': 2, 'O': 1}
for atom in water:
    print(atom)
~~~
{: .language-python}
~~~
H
O
~~~
{: .output}

Notice that this only gives us the _keys_ in the dictionary. If we want the values as well, we use the `items()` function.
Using `items()` in a for loop allows you to iterate over every key-value pair in a dictionary.
~~~
water = {'H': 2, 'O': 1}
for atom, count in water.items():
    print(f"{atom}: {count}")
~~~
{: .language-python}
~~~
H: 2
O: 1
~~~
{: .output}

> ## Dictionary Functions & Caveats
> Python has a few other functions to access the keys or values of a dictionary. These include `.keys()` to get the keys and
> `values()` to get all of the values. These return a _view object_, which can be thought of as a window into the dictionary
> being accessed. It is important to note that these do not create a copy of the dictionary, so any changes made to the dictionary
> will be reflected in the view object. This is is illustrated with the following snippet.
>
> ~~~
> water = {'H': 2, 'O': 1}
> water_items = water.items()
> print(water.items())
> water['N'] = 0
> print(water.items())
> ~~~
> {: .python}
>
> ~~~
> dict_items([('H', 2), ('O', 1)])
> dict_items([('H', 2), ('O', 1), ('N', 0)])
> ~~~
> {: .output}
>
> Notice that the new key-value pair that we inserted was reflected in the `water_items` variable automatically! An
> important caveat with this is that, since dictionaries are unordered, we are not guaranteed that any inserted elements
> will be after the current element. So, we cannot change the size of the dictionary while iterating over it, or else we
> could skip elements. This bug only manifests _sometimes_, which makes it especially nasty to track down. The takeaway is,
> no insertion or deletion on the dictionary while iterating over its contents. Actions such as modifying the values for keys
> do not change the size of the dictionary, so they are completely safe.
{: .callout}

Lets put this all into practice by modifying a function from the (FIXME which lesson was this function from?) lesson to
use a dictionary instead of paired arrays. Consider the original function.

~~~
def open_pdb(f_loc):
    # This function reads in a pdb file and returns the atom names and coordinates.
    with open(f_loc) as f:
        data = f.readlines()
    c = []
    sym = []
    for line in data:
        if 'ATOM' in line[0:6] or 'HETATM' in line[0:6]:
            sym.append(line[76:79].strip())
            c2 = []
            for x in line[30:55].split():
                c2.append(float(x))
            c.append(c2)
    coords = np.array(c)
    return sym, coords
~~~
{: .language-python}

Rather than relying on the ordering of the paired arrays, we can create a mapping from symbol to coordinate. This approach
only works if the symbols in the file are unique, as the dictionary keys must be unique. If the symbols are not unique,
we could write over a previous symbols coordinates on accident! Sometimes behavior like this is desired, sometimes it is
a bug. Regardless, it is important to carefully consider your goals and the nature of your data when you are deciding on
a data structure to store it. A well crafted data structure makes writing the rest of your script much easier and more
enjoyable.

~~~
def open_pdb_dict(f_loc):
    with open(f_loc) as f:
        data = f.readlines()
    sym_map = {}
    for line in data:
        if 'ATOM' in line[0:6] or 'HETATM' in line[0:6]:
            c2 = []
            for x in line[30:55].split():
                c2.append(float(x))
            sym_map[line[76:79].strip()] = c2
    return sym_map
~~~
{: .language-python}

> ## Assign vs Access
>
> Notice that we do not ever test if the key exists before we access it. This is safe because we are only _assigning_ to the
> dictionary, never _accessing_ its value. This does have the caveat that, if the key already exists in the dictionary,
> we overwrite the old value and could never know. If you do not wish to leave this possibility open, you could check
> if the key exists in the dictionary, and throw an error if it does.
> ~~~
> def open_pdb_dict(f_loc):
>     with open(f_loc) as f:
>         data = f.readlines()
>     sym_map = {}
>     for line in data:
>         if 'ATOM' in line[0:6] or 'HETATM' in line[0:6]:
>             sym = line[76:79].strip()
>             if sym in sym_map:
>                 raise Exception(f"key '{sym}' already exists in sym_map")
>             c2 = []
>             for x in line[30:55].split():
>                 c2.append(float(x))
>             sym_map[line[76:79].strip()] = c2
>     return sym_map
> ~~~
> {: .language-python}
{: .callout}

> ## Exercise
>
> Write a function `count_atoms(filename)` that takes in a .xyz file and prints out the number of each type of atom in
> the molecule. Then, run it on the provided `benzene.xyz` file.
>
> > ## Solution
> >
> > Benzene.xyz has 6 carbons and 6 hydrogens, so you should get something similar to the below
> > ~~~
> > C: 6
> > H: 6
> > ~~~
> > {: .output}
> > If your output looks more like
> >
> > ~~~
> > {'C': 6, 'H': 6}
> > ~~~
> > {: .output}
> >
> > then you are printing the dictionary directly. This is perfectly acceptable for testing purposes, but it is
> > generally preferable to format your output into a more easily digestible form. Recall that you can iterate through a
> > dictionary using `your_dict.items()`
> >
> > ~~~
> > def print_dictionary(dict):
> >     # dict.items() is a generator that yields tuples of (key, value) for every entry in the dictionary
> >     # don't worry too much about 'generators' now, we will cover them in the next section
> >     for key, value in dict.items():
> >         print(f"{key} -> {value}")
> > ~~~
> > {: .language-python}
> >
> > A common error when trying to build a dictionary is a `KeyError`:
> >
> > ~~~
> > KeyError: 'C'
> > ~~~
> >{: .error}
> >
> > If you are getting this error, it is because you are trying to look up a key that does not exist in your dictionary.
> > Recall that we can use Python's `in` operator to test for membership in an iterable. So, to safely create the dictionary,
> > we should do something like this:
> >
> > ~~~
> > def counts_atoms(filename):
> >     xyz_file = numpy.genfromtxt(fname=filename, skip_header=2, dtype='unicode')
> >     # create an empty dictionary, called counts, to keep track of the counts
> >     counts = {}
> >
> >     # iterate over every 'line' from the file in the numpy array xyz_file
> >     for l in xyz_file:
> >         # if the atom is in our dictionary, increment the number, because we have found another
> >         if l[0] in counts:
> >             counts[l[0]] += 1
> >         # if the atom is not in our dictionary, it must be the first of it's kind that we have seen.
> >         # so, we add it to the dictionary and set its count to 1
> >         else:
> >             counts[l[0]] = 1
> >
> >     # iterate through our counts and print out the count of each atom
> >     for atom, count in counts.items():
> >         print(f'{atom}: {count}')
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}
