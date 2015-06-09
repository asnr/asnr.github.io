---
layout:     post
title:      Powershell equivalents of Unix commands
date:       2015-06-09 12:31:19
summary:    
categories: [powershell, unix]
---

Windows happens. So do plain text files. Sometimes, they happen together.

The greybeard in you may reach instinctively for `Cygwin`,  


### Encodings

Powershell, like the rest of Windows, thinks in UTF-16. One charming consequence of this is that passing an ASCII data file through a Powershell pipeline can double its size on disk. This is often not what you or your sysadmin wants, so instead of redirecting output to a file using `>`,

```posh
> type data.csv > data_copy.csv
```

use the `out-file` cmdlet (`>` is just synctatic sugar for `out-file` anyway):

```posh
> type data.csv | out-file data_copy.csv -encoding utf8
```

You can also use `-encoding ascii`.


### `$ $CMD --help`
```posh
> help $CMD
```
The command `help` is an alias for `get-help`. The "REMARKS" section at the bottom of the `help` output has good further pointers, like what command to use to see examples and more detailed information.


### `$ cheat`

If you're not already using `cheat`, then stop reading and [be enlightened](https://github.com/chrisallenlane/cheat).

```posh
> get-help $CMD -examples
```
Weirdly, `help $CMD -examples` is presented slightly differently; try both and see which you prefer.


### `$ head`

```posh
> get-content $FILE -TotalCount 10
```


### `$ tail`

```posh
> get-content $FILE | select-object -last 10
```


### Vanilla `$ grep`

```posh
> get-content $FILE | where {$_ -match "$PATTERN"}
```

Here we make use of the regular expression matching command `-match`, which supports at least Extended Regular Expressions:

```posh
> "foo bar" -match "b|c"
True
```


### Recursive `$ grep`

**What about returning lines instead of paths?**

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


A less than pleasant example
----------------------------

To replace all of the occurences of `||` in `in.txt` with `| |` and send the result to `out.txt` encoded in UTF-8:

```posh
get-content in.txt | % {$_ replace "\|\|","| |"} | % {$_ -replace "\|\|","| |"} | out-file out.txt -encoding utf8
```

