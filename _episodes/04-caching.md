---
title: "Function Caching"
teaching: 0
exercises: 0
questions:
- "How can we avoid expensive recomputations?"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

> ## Check Your Understanding
>
> Which of these functions would be the best candidates for caching?
>
> ~~~
> # pure & expensive, ideal candidate for caching
> def calculation1(atom1, atom2):
>   # TODO figure out what to put in here, make sure it stays pure & expensive
>
> # pure, but really, really fast, no need to cache
> def bond_check(atom_distance, minimum_length=0, maximum_length=1.5):
>   return minimum_length < atom_distance <= maximum_length
>
> def calculation2(atom1, atom2):
>   # TODO do a calculation here that modifies one of the input paramaters so it is impure
>   # make it expensive so that we can tell if students understand both requireements for caching
>
> ~~~
> {: .language-python}
>
> > ## Solution
> >
> > *   `calculation1` is the best candidate for caching, because it is pure _and_ expensive.
> > *   `bond_check` could also be cached,
> > but it is such a fast function that it would take longer to look up the cache than it would to recompute the value.
> > *   `calculation2`, on the other hand, is expensive, so it is tempting to cache it. However, since `calculation2` is not
> > a pure function, we cannot cache it. Recall that impure functions have side-effects that other parts
> > of your program may depend on. Since a cached function will only execute when it is called with new parameters, some
> > runs of the program may never actually execute the function. This means that the side-effects that our program depends on
> > may never exist!
> >
> > {: .output}
> {: .solution}
{: .challenge}
