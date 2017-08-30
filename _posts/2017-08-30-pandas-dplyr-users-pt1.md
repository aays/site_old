# An introduction to `pandas` for bewildered `dplyr` users - Part 1

Personally, it's a little hard to fathom what data frame operations were like before `dplyr` and the tidyverse came around. An elegant and intuitive R toolkit for working with data frames, or table-formatted data objects, `dplyr` offers a frightening amount of utility with incredibly straightforward syntax. A lot of this elegance is owed to its seamless integration with the R pipe operator, or `%>%`, which facilitates the chaining together of operations for quick and efficient data frame manipulation.

Of course, the Python world has since noticed the awesome potential of data frames, and Python's lack of a native data frame object class eventually led to the development of `pandas`. Much like the tidyverse, `pandas` largely centers around working with `DataFrame` objects, and exhibits some clear parallels with its R equivalents. At the same time, its syntax does include many quirks that may be a little difficult for a native R user to wrap their head around -- perhaps partially because of the object-oriented nature of Python.

To be up front for a second here: I'm writing this post mostly for myself. I am the titular bewildered `dplyr` user, and yet a mix of fate and circumstance has resulted in me having to use `pandas` far more than I'd have previously thought myself comfortable with. Which is not to imply that `pandas` is somehow inferior and to be avoided -- I just like `dplyr`, and indeed the tidyverse at large, which has yet to leave literally any of my data analysis needs wanting. But why put all your eggs in one basket, you know? (Yes, even despite the barrage of clumsy data science blog posts about the supposedly brutal and frenzied war between R and Python or whatever, but I digress)

This post here is my attempt to make some sense out of `pandas` operations in light of my familiarity and comfort with the tidyverse, and `dplyr` in particular. I'm hoping that in explaining `pandas` within the context of some core `dplyr` verbs, it's perhaps even the tiniest bit easier for R users to wrap their heads around this nifty Python library. But remember: a little knowledge can go a long way!

## Getting Started

Like any good data frame-related tutorial would do, I figured the classic `iris` dataset would be a good environment for us to explore `pandas` in. Loading it into R is simple enough, given that it's a built-in dataset. Let's also get `dplyr` up and running.

```r
library(dplyr)
iris
```

Getting `iris` is moderately trickier in Python. If you've got the `scikit-learn` library installed, it can be grabbed from there. Let's also import `pandas` -- by convention, it is often abbreviated as `pd`.

```python
import pandas as pd
from sklearn.datasets import load_iris

iris = load_iris()
iris = pd.DataFrame(iris.data, columns = iris.feature_names)
```

If you don't have `scikit-learn` but do have `rpy2` installed (as is the case with our lab server) [the following code](https://stackoverflow.com/questions/28417293/sample-datasets-in-pandas) will do the trick instead. Don't bother trying to understand it if you don't already -- what's going on here is not super relevant for what we're actually talking about today anyways.

```python
import pandas as pd
import rpy2.robjects as ro
import rpy2.robjects.conversion as conversion
from rpy2.robjects import pandas2ri
pandas2ri.activate()

R = ro.r
iris = conversion.ri2py(R['iris'])
```

Now, two things to note before we get started:
1. Much like in `dplyr`, it appears a given operation on a `pandas` dataframe will _always_ return a new data frame object. That being said, multiple `pandas` functions do offer a boolean `inplace` argument if you'd like for an operation to modify an object as such. Frankly, I don't personally know enough about programming principles to know whether doing so is necessarily a better or worse approach - just go with whatever makes the code easiest to implement and read after the fact.
2. As far as I know, `pandas` _has no pipe operator._ That's right. I know that's a huge part of the intuitiveness of `dplyr` gone off the bat. There _is_ a semi-equivalent in the form of the `.pipe()` method but we'll get into why that doesn't function in exactly the same way. The `pandas` cheat sheet does recommend 'method chaining` as an option, which I will do a bit of here, but which I personally find to be not particularly conducive to easy reading.<sup>[[1](#footnote1)]</sup>

Onwards - let's start our discussion with some quick functions for describing our data frames.

### Looking at our data

`head`, the timeless convenience function for peeking at the top of our data frame, remains largely unchanged across both languages. The only difference is that it's a _function_ in R and a _class method_ in Python. 

R:
```r
head(iris)

##   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
## 1          5.1         3.5          1.4         0.2  setosa
## 2          4.9         3.0          1.4         0.2  setosa
## 3          4.7         3.2          1.3         0.2  setosa
## 4          4.6         3.1          1.5         0.2  setosa
## 5          5.0         3.6          1.4         0.2  setosa
## 6          5.4         3.9          1.7         0.4  setosa
```

Python:
```python
iris.head() # note brackets at the end

   Sepal.Length  Sepal.Width  Petal.Length  Petal.Width Species
1           5.1          3.5           1.4          0.2  setosa
2           4.9          3.0           1.4          0.2  setosa
3           4.7          3.2           1.3          0.2  setosa
4           4.6          3.1           1.5          0.2  setosa
5           5.0          3.6           1.4          0.2  setosa
```

Same goes for `dim`, except it's an _attribute_ in Python instead. All that really means is that unlike with `.head()`, we avoid placing parentheses at the end this time around.

R:
```r
dim(iris)

## [1] 150   5
```

Python:
```python
iris.shape # note lack of parentheses

## (150, 5)
```

Finally, before jumping into `dplyr` functions, we're going to rename the columns of our `pandas` data frame to eliminate the periods. This is because the period in Python is a means of denoting methods/attributes of an object: for instance, where `my.data` is a perfectly reasonable object name in R, Python will interpret that as 'return the `data` attribute of the object `my`'!. We can rename our columns by passing a dictionary object to the `.rename()` attribute: 

```python
iris = iris.rename(columns = {'Sepal.Length': 'Sepal_Length', 'Petal.Length': 'Petal_Length', 'Sepal.Width': 'Sepal_Width', 'Petal.Width': 'Petal_Width'})
```

Now for some actual `dplyr` functions!

### `filter` - subsetting rows

In `dplyr`, `filter` allows us to subset rows based on a condition. The syntax is quite straightforward:

```r
filter(iris, Sepal.Length > 7)
# or: iris %>% filter(Sepal.Length > 7)

##    Sepal.Length Sepal.Width Petal.Length Petal.Width   Species
## 1           7.1         3.0          5.9         2.1 virginica
## 2           7.6         3.0          6.6         2.1 virginica
## 3           7.3         2.9          6.3         1.8 virginica
## 4           7.2         3.6          6.1         2.5 virginica
## 5           7.7         3.8          6.7         2.2 virginica
## 6           7.7         2.6          6.9         2.3 virginica
## 7           7.7         2.8          6.7         2.0 virginica
## 8           7.2         3.2          6.0         1.8 virginica
## 9           7.2         3.0          5.8         1.6 virginica
## 10          7.4         2.8          6.1         1.9 virginica
## 11          7.9         3.8          6.4         2.0 virginica
## 12          7.7         3.0          6.1         2.3 virginica
```

`pandas` has two ways of doing this same operation.

With the `.query()` method:
```python
iris.query('Sepal_Length > 7') # note how the query is a string

##      Sepal_Length  Sepal_Width  Petal_Length  Petal_Width    Species
## 103           7.1          3.0           5.9          2.1  virginica
## 106           7.6          3.0           6.6          2.1  virginica
## 108           7.3          2.9           6.3          1.8  virginica
## 110           7.2          3.6           6.1          2.5  virginica
## 118           7.7          3.8           6.7          2.2  virginica
## 119           7.7          2.6           6.9          2.3  virginica
## 123           7.7          2.8           6.7          2.0  virginica
## 126           7.2          3.2           6.0          1.8  virginica
## 130           7.2          3.0           5.8          1.6  virginica
## 131           7.4          2.8           6.1          1.9  virginica
## 132           7.9          3.8           6.4          2.0  virginica
## 136           7.7          3.0           6.1          2.3  virginica
```

Or by using conditions within square brackets (quite like base R):
```python
iris[iris.Sepal_Length > 7]

##      Sepal_Length  Sepal_Width  Petal_Length  Petal_Width    Species
## 103           7.1          3.0           5.9          2.1  virginica
## 106           7.6          3.0           6.6          2.1  virginica
## 108           7.3          2.9           6.3          1.8  virginica
## 110           7.2          3.6           6.1          2.5  virginica
## 118           7.7          3.8           6.7          2.2  virginica
## 119           7.7          2.6           6.9          2.3  virginica
## 123           7.7          2.8           6.7          2.0  virginica
## 126           7.2          3.2           6.0          1.8  virginica
## 130           7.2          3.0           5.8          1.6  virginica
## 131           7.4          2.8           6.1          1.9  virginica
## 132           7.9          3.8           6.4          2.0  virginica
## 136           7.7          3.0           6.1          2.3  virginica
```

The base R equivalent to that, for reference:

```r
iris[iris$Sepal.Length > 7, ]

##     Sepal.Length Sepal.Width Petal.Length Petal.Width   Species
## 103          7.1         3.0          5.9         2.1 virginica
## 106          7.6         3.0          6.6         2.1 virginica
## 108          7.3         2.9          6.3         1.8 virginica
## 110          7.2         3.6          6.1         2.5 virginica
## 118          7.7         3.8          6.7         2.2 virginica
## 119          7.7         2.6          6.9         2.3 virginica
## 123          7.7         2.8          6.7         2.0 virginica
## 126          7.2         3.2          6.0         1.8 virginica
## 130          7.2         3.0          5.8         1.6 virginica
## 131          7.4         2.8          6.1         1.9 virginica
## 132          7.9         3.8          6.4         2.0 virginica
## 136          7.7         3.0          6.1         2.3 virginica
```


### A quick aside - what's in an index?

You may have noticed how in the above example, the indices of the R data frame have been reset following our usage of `filter`, but those of the `pandas` one haven't (same goes for the base R operation, but that's another conversation entirely). This can be an unexpected source of frustration more often than you might think, especially if you're performing operations involving repeated subsetting and indexing of data frames. To reset the index each time, simply take on the `.reset_index()` method at the end of a `pandas` operation. Careful, however - doing so will create a new column, simply called `index`, which will store all of the 'old' indices. That's probably something to watch out for if you're selecting columns by position and not name!

```python
iris.query('Sepal_Length > 7').reset_index()

##     index  Sepal_Length  Sepal_Width  Petal_Length  Petal_Width    Species
## 0     103           7.1          3.0           5.9          2.1  virginica
## 1     106           7.6          3.0           6.6          2.1  virginica
## 2     108           7.3          2.9           6.3          1.8  virginica
## 3     110           7.2          3.6           6.1          2.5  virginica
## 4     118           7.7          3.8           6.7          2.2  virginica
## 5     119           7.7          2.6           6.9          2.3  virginica
## 6     123           7.7          2.8           6.7          2.0  virginica
## 7     126           7.2          3.2           6.0          1.8  virginica
## 8     130           7.2          3.0           5.8          1.6  virginica
## 9     131           7.4          2.8           6.1          1.9  virginica
## 10    132           7.9          3.8           6.4          2.0  virginica
## 11    136           7.7          3.0           6.1          2.3  virginica
```

### `select` - subsetting columns

`select` allows us to pull out columns by name. Further columns can simply be fed to `select` as further arguments. The `-` operator also lets us remove a column if we'd like, while the `:` operator allows us to select multiple consecutive columns.

```r
iris %>% select(Sepal.Length, Species) %>% head()

##   Sepal.Length Species
## 1          5.1  setosa
## 2          4.9  setosa
## 3          4.7  setosa
## 4          4.6  setosa
## 5          5.0  setosa
## 6          5.4  setosa

iris %>% select(-Species) %>% head()

##   Sepal.Length Sepal.Width Petal.Length Petal.Width
## 1          5.1         3.5          1.4         0.2
## 2          4.9         3.0          1.4         0.2
## 3          4.7         3.2          1.3         0.2
## 4          4.6         3.1          1.5         0.2
## 5          5.0         3.6          1.4         0.2
## 6          5.4         3.9          1.7         0.4

iris %>% select(Sepal.Length:Petal.Length) %>% head()

##   Sepal.Length Sepal.Width Petal.Length
## 1          5.1         3.5          1.4
## 2          4.9         3.0          1.4
## 3          4.7         3.2          1.3
## 4          4.6         3.1          1.5
## 5          5.0         3.6          1.4
## 6          5.4         3.9          1.7
```

This is where `pandas` diverges quite a bit, based on which of these operations we'd like to perform.

Selecting one or more columns is done with double square brackets:
```python
iris[['Sepal_Length', 'Species']]

##    Sepal_Length Species
## 1           5.1  setosa
## 2           4.9  setosa
## 3           4.7  setosa
## 4           4.6  setosa
## 5           5.0  setosa
## etc...
```

But dropping columns necessitates providing a list of column names to the `.drop()` method:
```python
iris.drop(['Species'], axis = 1)

##    Sepal_Length  Sepal_Width  Petal_Length  Petal_Width
## 1           5.1          3.5           1.4          0.2
## 2           4.9          3.0           1.4          0.2
## 3           4.7          3.2           1.3          0.2
## 4           4.6          3.1           1.5          0.2
## 5           5.0          3.6           1.4          0.2
```
Where `axis = 1` denotes that we want to drop by column.

Finally, 'slicing' columns is done with `.loc()`:
```python
iris.loc[:, 'Sepal_Length':'Petal_Length']

##    Sepal_Length  Sepal_Width  Petal_Length
## 1           5.1          3.5           1.4
## 2           4.9          3.0           1.4
## 3           4.7          3.2           1.3
## 4           4.6          3.1           1.5
## 5           5.0          3.6           1.4
## etc...
```
The `:` before the comma there simply indicates that we want all rows to be returned. It's possible, however, to put a `range` object there if we'd like to subset by row as well.

```python
iris.loc[range(3,6), 'Sepal_Length':'Petal_Length']

##    Sepal_Length  Sepal_Width  Petal_Length
## 3           4.7          3.2           1.3
## 4           4.6          3.1           1.5
## 5           5.0          3.6           1.4
```

### `arrange` - sort data (+ how to chain methods, aka _Ceci n'a pas une pipe_)

As `dplyr` verbs go, `arrange` is quite self explanatory.

```r
iris %>%
    select(Sepal.Length, Species) %>%
    filter(Sepal.Length > 7) %>%
    arrange(Sepal.Length)
    
##    Sepal.Length   Species
## 1           7.1 virginica
## 2           7.2 virginica
## 3           7.2 virginica
## 4           7.2 virginica
## 5           7.3 virginica
## 6           7.4 virginica
## 7           7.6 virginica
## 8           7.7 virginica
## 9           7.7 virginica
## 10          7.7 virginica
## 11          7.7 virginica
## 12          7.9 virginica    
```

The `pandas` equivalent to this code would necessitate some method chaining. This can be thought of as a series of methods tacked onto one another, but broken up into separate lines.

```python
iris[['Sepal_Length', 'Species']] \
.query('Sepal_Length > 7') \
.sort_values(['Sepal_Length'])

# aka iris[['Sepal_Length', 'Species']].query('Sepal_Length > 7').sort_values(['Sepal_Length'])
# awful, I know

##      Sepal_Length    Species
## 103           7.1  virginica
## 110           7.2  virginica
## 126           7.2  virginica
## 130           7.2  virginica
## 108           7.3  virginica
## 131           7.4  virginica
## 106           7.6  virginica
## 118           7.7  virginica
## 119           7.7  virginica
## 123           7.7  virginica
## 136           7.7  virginica
## 132           7.9  virginica
```

Notice how we use `\` to continue code on new lines. As far as I know, this is the closest way to approximate the pipe-driven syntax `dplyr` offers.

---

That's it for the first part here. In the second installment, I'll be going over how to use the `.pipe()` method (and why it's really not the best way to go a lot of the time...), `group_by`, `mutate`, `summarise`, and `sample_n`/`sample_frac`. A potential third installment might go beyond that and cover things like `.iterrows()` and other `pandas`-specific tools, but that depends on how far my own understanding of `pandas` manages to grow!

<a name="footnote1"><sup>1</sup></a> One [can apparently code a 'hack' pipe in Python](https://stackoverflow.com/questions/28252585/functional-pipes-in-python-like-from-dplyr) using the `infix` library, but I haven't done so myself and can't really comment on whether that's worth your time.
