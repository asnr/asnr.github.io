---
layout:     post
title:      Honesty is the best policy
date:       2016-03-06
summary:    
categories: [man-pages]
---

From the man page for `strlcpy` and `strlcat`:

```
Since it is known how many characters were copied the first time, things can be sped up a bit by using a copy instead of an append

    char *dir, *file, pname[MAXPATHLEN];
    size_t n;

    ...

    n = strlcpy(pname, dir, sizeof(pname));
    if (n >= sizeof(pname))
           goto toolong;
    if (strlcpy(pname + n, file, sizeof(pname) - n) >= sizeof(pname) - n)
           goto toolong;

However, one may question the validity of such optimizations, as they defeat the whole purpose of strlcpy() and strlcat().  As a matter of fact, the first version of this manual page got it wrong.
```
