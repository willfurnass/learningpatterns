+++
title = "Book review: Ed Mastery by Michael W Lucas"
date = 2018-04-08
tags = ["ed", "sed", "grep", "awk", "vi", "vim", "unix"]
featured_image = ""
description = ""
aliases = [
    "/posts-output/2018-04-08-ed-mastery/",
]
+++

Why learn Latin in 2018 if you're not a member of the catholic church?  Or Greek if you're not greek? 
Well, both are great at putting things in context and highlighting commonality:
Latin does this for the romance languages and Greek does this for much scientific terminology.

And, **Ed**, a 46-47 year old positively archaic text editor, does this for parts of the Unix userland 
that are still very much in common use (specifically sed, awk, grep, vi), 
as I learned from reading Michael W Lucas' new book *[Ed Mastery](https://www.michaelwlucas.com/tools/ed)*.

![Ed Mastery book - front cover](/img/ed-mastery.jpg)

It was published on April 1st. 
It focuses exclusively on a tool that no-one bar the neckbeardiest of [neckbeards](https://encyclopediadramatica.rs/Neckbeard) has used in recent decades.
Its interface is beyond parsimonious: it doesn't even print your file to the screen by default (as it was designed in the era of [teletypes](https://en.wikipedia.org/wiki/Teleprinter)).
And yet...and yet learning its obscure interface was oddly enlightening as its most powerful features, 
the abilities to 

* apply operations to lines by addresses or address ranges, 
* identify lines by regular expressions (regexes) and
* perform substitutions using regexes,

were incorporated into many of the Unixy tools we used today.
In some cases the syntax is common to Ed and 'modern' tools; 
in other cases its just the ideas that are shared.

For example, in Ed you can run the command

```
g/nice_ferret/m$
```

to move (`m`) all lines in the buffer (`g`) containing `nice_ferret` to the end of the buffer (`$`). 
The same works in vim if, in command-mode, you enter it after a `:`.

You can also print out all lines that contain the regular expression `re` using `g/re/p` - now why does that look familiar?

In short, sure, you could the book's something of a joke and 
a large chunk of the material is on the basics of regexes, which may already be familiar to you, 
but it's still worth a read. 
Plus it might give some [Vim Golf](https://vimgolf.com/) pros a new challenge to get their teeth into!
