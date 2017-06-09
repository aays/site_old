---
layout: post
title: Working with lists of data frames in R with lapply
date: 2017-06-07
tags: R
---

For those times when having only one data frame just doesn't cut the cheese.

## Motivation

My own work in genomics often involves working with datasets on the scale of
entire chromosomes, and rarely am I working with just one at a time. This results in
me having to work on several highly similar datasets at once that more or less exhibit 
the exact same structure.

The intuitive solution may be to write a script to be run in the command
line, taking in one dataset at a time and performing analyses as directed. This also makes it easy to exploit bash tools (such as `parallel`) and is probably the better long-term solution for its sheer scalability.

So then why might we want to work with lists of data frames in a single script/environment?
Well, a number of reasons:

1. Exploratory work in RStudio/a Jupyter Notebook, for instance, when testing elements of what may become a command line-ready script - but not wanting to do so with just one dataset;
2. Writing a single-input script that involves splitting a dataset at some point (bit of a weird reason, frankly, but do you);
3. Performing comparative analyses between multiple datasets in a quick and inherently 'parallel'<sup>[[1](#footnote1)]</sup> way.

## `lapply` - Introduction

The entire process of working with a list of data frames ultimately hinges on one elegant function - `lapply`. `lapply` will take in a vector<sup>[[2](#footnote2)]</sup> or list, perform a specified operation on it, and return the modified version as a list. Here's a reasonably straightforward example with some code:

```
> ones <- list(1,2,3,4)
> exponentials <- lapply(ones, exp) # e^x for each element in our list
> exponentials
[[1]]
[1] 2.718282
[[2]]
[1] 7.389056
[[3]]
[1] 20.08554
[[4]]
[1] 54.59815

> class(exponentials)
[1] "list"
```

It's also possible to define 'quick and dirty' functions within the body of `lapply` itself:

```
> ones <- c(1,2,3,4)
> twos <- lapply(ones, function(x) x*2)
> twos
[[1]]
[1] 2
[[2]]
[1] 4
[[3]]
[1] 6
[[4]]
[1] 8
```

Finally, custom defined functions also work just fine:

```
> twoandone <- function(x){
	  out <- x*2 + 1
	  return(out)
  }

> ones <- c(1,2,3,4)
> newlist <- lapply(ones, twoandone)
> newlist
[[1]]
[1] 3
[[2]]
[1] 5
[[3]]
[1] 7
[[4]]
[1] 9

```

## `lapply` and Data Frames

This is where things get interesting. Think about it - virtually any function you can run on a single data frame, you can now run on multiple at once with hardly an infinitesimal increase in effort.

But before we even get started on the possibilities that allows, we can actually use `lapply` itself to create our list of data frames in the first place. Let's make some sample data:

```
> data1 <- data.frame(first = c(1:5), second = c(2:6))
> data2 <- data.frame(first = c(3:7), second = c(4:8))
```

Now, assuming our dfs are named in a consistent way that allows for [globbing](https://en.wikipedia.org/wiki/Glob_(programming)), we can instantiate `ls()` with `get` to bring them together into a list:

```
> ls()
[1] "data1" "data2"
> dflist <- lapply(ls(pattern = 'data*'), get)
> dflist
[[1]]
  first second
1     1      2
2     2      3
3     3      4
4     4      5
5     5      6
[[2]]
  first second
1     3      4
2     4      5
3     5      6
4     6      7
5     7      8
```

And now any work you'd like to do on one df can be done at both at once, solely by sticking your function of choice into `lapply` instead of running it in the command line as is. For instance:

```
> lapply(dflist, dim)
[[1]]
[1] 5 2
[[2]]
[1] 5 2

> lapply(dflist, function(df) mean(df$first))
[[1]]
[1] 3
[[2]]
[1] 5
```

This synergizes beautifully with tidyverse operations too, of course:

```
> library(tidyverse)
> lapply(dflist, function(df) mutate(df, third = second * first))
[[1]]  
  first second third
1     1      2     2
2     2      3     6
3     3      4    12
4     4      5    20
5     5      6    30
[[2]]
  first second third
1     3      4    12
2     4      5    20
3     5      6    30
4     6      7    42
5     7      8    56
```

Of course, remember that `lapply` is not an inplace function! The original list is not modified unless the `lapply` function is assigned back to it.

```
> lapply(dflist, function(df) mutate(df, third = second * first))
[[1]]  
  first second third
1     1      2     2
2     2      3     6
3     3      4    12
4     4      5    20
5     5      6    30
[[2]]
  first second third
1     3      4    12
2     4      5    20
3     5      6    30
4     6      7    42
5     7      8    56

> dflist # checking original list
[[1]]
  first second
1     1      2
2     2      3
3     3      4
4     4      5
5     5      6
[[2]]
  first second
1     3      4
2     4      5
3     5      6
4     6      7
5     7      8

> dflist <- lapply(dflist, function(df) mutate(df, third = second * first))
> dflist # original list is now modified
[[1]]
  first second third
1     1      2     2
2     2      3     6
3     3      4    12
4     4      5    20
5     5      6    30
[[2]]
  first second third
1     3      4    12
2     4      5    20
3     5      6    30
4     6      7    42
5     7      8    56

```

One last thing remains: sometimes (and especially with larger lists of data frames) it can be hard to keep track of them all, and it doesn't help that `lapply` has a tendency to remove a data frame's name when it's been pulled into a list. This can be quickly corrected using `names`, however:

	> names(dflist) <- c('data1', 'data2')

Furthermore, if you used `ls` to glob data frames into a script, feeding the exact same pattern you used for the globbing into `names` will also do the trick:

```
> dflist <- lapply(ls(pattern = 'data*'), get)
> names(dflist) <- ls(pattern = 'data*')
> dflist
$data1
  first second
1     1      2
2     2      3
3     3      4
4     4      5
5     5      6

$data2
  first second
1     3      4
2     4      5
3     5      6
4     6      7
5     7      8
```

Enjoy!

<a name="footnote1"><sup>1</sup></a> `lapply` is not internally parallelized in memory far as I know, even if it does constitute what's called an [embarrassingly parallel](https://en.wikipedia.org/wiki/Embarrassingly_parallel) function in terms of how it reads through the elements of a list. There are parallelized options for `lapply`-like work, such as those found in the `parallel` library, but those are outside the scope of this post (and probably a Google search away).

<a name="footnote2"><sup>2</sup></a> While `lapply` can take in either a list or a vector as input, it will _always_ return a list, while keeping the object types of said list's elements intact. 

