---
title: "Comprehensions & Generator Expressions"
teaching: 0
exercises: 0
questions:
- "How can we efficiently iterate over massive data sets?"
- "Is there a more readable way to create lists and dictionaries?"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

> ## Exercise
>
> Convert the function `open_pdb_dict` from the last episode to use comprehensions.
>
> ~~~
> def open_pdb_dict(f_loc):
>     with open(f_loc) as f:
>         data = f.readlines()
>     sym_map = {}
>     for line in data:
>         if 'ATOM' in line[0:6] or 'HETATM' in line[0:6]:
>             c2 = []
>             for x in line[30:55].split():
>                 c2.append(float(x))
>             sym_map[line[76:79].strip()] = c2
>     return sym_map
> ~~~
> {: .language-python}
>
> > ## Solution
> >
> > Keeping in mind that a main point of comprehensions is to make code more readable, there are a few ways that we can
> > change `open_pdb_dict`. Which one you use depends on what is most readable to you at the time that you are writing it.
> >
> > The first is a partial substitution, where we use a comprehension for the list `c2`, but not for the dict `sym_map`.
> >
> > ~~~
> > def open_pdb_comp(f_loc):
> >    with open(f_loc) as f:
> >       data = f.readlines()
> >    sym_map = {}
> >    for line in data:
> >       if 'ATOM' in line[0:6] or 'HETATM' in line[0:6]:
> >          sym_map[line[76:79].strip()] = [float(x) for x in line[30:55].split()]
> >    return sym_map
> > ~~~
> > {: .language-python}
> >
> > Even if this form takes you an extra second to process now, as you gain more experience, you will find it much more
> > readable than the original form with the nested for loops. This is because a list comprehension always does exactly
> > one thing: creates a list. A loop, on the other hand, is incredibly versatile, so it takes extra effort to figure out
> > exactly what it is doing.
> >
> > We can go even further and replace the dictionary construction with a comprehension as well. Recall that comprehensions
> > can have conditional statements in them, so we can reduce the whole function to three lines. If the dictionary construction were
> > any more complicated, I would not substitute it for a comprehension, because the comprehension would be more difficult
> > to read than the original code. With something this straightforward, however, the one-line comprehension solution is
> > readable and concise.
> >
> > ~~~
> > def open_pdb_comp(f_loc):
> >    with open(f_loc) as f:
> >        data = f.readlines()
> >    return {l[76:79].strip(): [float(x) for x in l[30:55].split()] for l in data if 'ATOM' in l[0:6] or 'HETATM' in l[0:6]}
> > ~~~
> > {: .language-python}
> >
> >
> {: .solution}
{: .challenge}
