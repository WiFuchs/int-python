---
title: "Dictionaries & Comprehensions"
teaching: 0
exercises: 0
questions:
- "How can we leverage Python's builtin tools to write faster, more readable code?"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

> ## Check Your Understanding
>
> Write a function `atom_counts(filename)` that takes in a .xyz file and prints out the number of each type of atom in
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
> > def atom_counts(filename):
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
