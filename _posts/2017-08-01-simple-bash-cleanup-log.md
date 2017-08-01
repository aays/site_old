---
layout: post
title: Setting up a simple cleanup log in bash
date: 2017-08-01
tags: bash
---

A simple function to automatically append info about deleted files to a cleanup log!

### Motivation

(I originally wanted to just tweet this little trick out, but 140 characters is rarely sufficient for this sort of thing.)

Anyhow - sometimes, when dealing with large amounts of intermediate files in a project, it can be useful to log what's being deleted when and where. Here's a quick bash trick for setting up a function that logs a) which files where deleted, b) when they were deleted, and c) how much space was freed up as a consequence.

I'm sure there are more elegant ways to do this, but this has worked well for me thus far!

### Method

First, let's create a log file for our project. This is where our function will log everything we delete. 

```sh
touch path/to/projectdir/cleanup.log
```

Next, we'll open up our `.bash_profile` in order to create our function.

```sh
vi ~/.bash_profile
```

Here's the template for my function, which I call `rml()` ('remove and log'). Paste the following anywhere within the body of your `.bash_profile`.

```sh
rml () {
printf '\n' >> [log file]
date >> [log file]
echo $1 >> [log file]
du -hcs $1 >> [log file]
rm $1
}
```

Annotated:

```sh
rml () {
printf '\n' >> [log file] # skip a line for readability
date >> [log file] # log the date of deletion
echo $1 >> [log file] # the name of the file
du -hcs $1 >> [log file] # the size of the file
rm $1 # and finally, delete it
}
```

Feel free to customize this however you'd like! When you're done, exit out of `vi` and make sure to `source` your `.bash_profile`.

```sh
source ~/.bash_profile
```

You only have to do this once, and then you're ready to go.

From there on out, just use `rml` in place of `rm` to log a file's deletion. For instance, the following command:

```sh
rml projectdir/bigintermediatefile.txt
```

Will add something like this to `cleanup.log`:

```
Tue Aug  1 12:09:01 EDT 2017
bigintermediatefile.txt
6.0G    bigintermediatefile.txt
6.0G    total
```

Enjoy!
