---
layout:     post
title:      Unix on Powershell
date:       2015-05-24 12:31:19
summary:    1st
categories: one
---

Remember, you probably don't want UTF-16 output, so use `| out-file "$FILE" -encoding utf8` or `| out-file "$FILE" -encoding ascii` instead of `> $FILE`, which is just syntactic sugar for `| out-file $FILE` anyway.


`$ $CMD --help`
-------
```posh
> help $CMD
```
The command `help` is an alias for `get-help`. The "REMARKS" section at the bottom of the `help` output has good further pointers, like what command to use to see examples and more detailed information.

`$ cheat`
---------
```posh
> get-help $CMD -examples
```
Weirdly `help $CMD -examples` does the `--more--` thing. I prefer using the scrollbar to pressing enter, but that's just me.


`$ head`
--------
```posh
> get-content $FILE -TotalCount $n
```


`$ tail`
--------
```posh
> get-content $FILE | select-object -last $n
```


`$ grep` within a file
----------------------
```posh
> get-content $FILE | where {$_ -match "$PATTERN"}
```

Here we make use of the regular expression matching command `-match`, which supports at least Extended Regular Expressions:

```posh
> "foo bar" -match "b|c"
True
```


Recursive `$ grep`
------------------
```posh
> dir -include *.sas -recurse | select-string -pattern "$PATTERN" | select -unique path
```


`$ sed`
-------

```posh
> get-content $FILE | % {$_ -replace "$PATTERN","$REPLACEMENT"}
```

You can capture back references with parens `()`, but to reference them it is best to use single quotes `'` around the second argument to `-replace`, otherwise the back references will be evaluated as Powershell variables:

```posh
> $1 = "NOPE"
> "bar" -replace "(bar)","foo $1"
foo NOPE
> "bar" -replace "(bar)",'foo $1'
foo bar
```


Calling Powershell from `cmd.exe`
---------------------------------

A less than pleasant example
----------------------------

To replace all of the occurences of `||` in `in.txt` with `| |` and send the result to `out.txt` encoded in UTF-8:

```posh
get-content in.txt | % {$_ replace "\|\|","| |"} | % {$_ -replace "\|\|","| |"} | out-file out.txt -encoding utf8
```

