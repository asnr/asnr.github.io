---
layout:     post
title:      Powershell equivalents of Unix commands
date:       2015-06-09 12:31:19
summary:    
categories: [powershell, unix]
---

Windows happens. So do text files. Sometimes, they happen together.

(Put `Cygwin` down.)


### Encodings

Powershell, like the rest of Windows, thinks in UTF-16. One charming consequence of this is that passing an ASCII data file through a Powershell pipeline will double its size on disk. This is often not what you or your sysadmin wants, so instead of redirecting output to a file using `>`,

```posh
> cat data.csv > data_copy.csv
```

use the `out-file` cmdlet (`>` is just synctatic sugar for `out-file` anyway):

```posh
> cat data.csv | out-file data_copy.csv -encoding utf8
```

You can also use `-encoding ascii`.


### `$ $CMD --help`

```posh
> help $CMD
```

The command `help` is an alias for `get-help`. The "REMARKS" section at the bottom of the `help` output has good further pointers, like what command to use to see examples and more detailed information.


### `$ cheat`

(If you're not already using `cheat`, then let me introduce you to [rainbows and sunshine](https://github.com/chrisallenlane/cheat).)

```posh
> get-help $CMD -examples
```

Weirdly, `help $CMD -examples` is presented slightly differently; try both and see which output you prefer.


### `$ head`

```posh
> get-content $FILE -TotalCount 10
> cat $FILE -t 10
```


### `$ tail`

```posh
> get-content $FILE | select-object -last 10
> cat $FILE | select -l 10
```


### Vanilla `$ grep`

```posh
> select-string $PATTERN $FILE
```

`select-string` accepts perl regular expressions:

```posh
> (echo "<foo> <bar>" | select-string "<.*?>").matches | select -exp value
<foo>
```

You can also use the `-match` command to iterate regexes quickly:

```posh
> "foo bar" -match "b|c"
True
```


### Recursive `$ grep`

```posh
> get-childitem -include *.txt -recurse | select-string $PATTERN
> ls -i *.txt -r | select-string $PATTERN
```


### `$ sed`


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


### MOAR

 *  (A whole book)[https://www.penflip.com/powershellorg/a-unix-persons-guide-to-powershell]
 *  (grep translations)[https://communary.wordpress.com/2014/11/10/grep-the-powershell-way/]
