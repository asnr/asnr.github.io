---
layout:     post
title:      Unix on Powershell
date:       2015-05-24 12:31:19
summary:    1st
categories: one
---


`$ head`
--------

```posh
get-content $FILE -TotalCount $n
```


`$ tail`
--------

```posh
get-content $FILE | select-object -last $n
```


`$ grep`
--------

```posh
> get-content $FILE | where {$_ -match "$PATTERN"}
```

Supports at least Extended Regular Expressions. (Note you can quickly test your regular expressions using `"foo bar" -match "b|c"`.)


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

