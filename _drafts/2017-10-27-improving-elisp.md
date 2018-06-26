---
layout:     post
title:      Improving elisp
date:       2017-10-27
summary:    💰💰💰
categories: [emacs, elisp]
---

0. A symbol holds distinct value and function cells (the technical term for this is that elisp is lisp-2, as opposed to Scheme and Racket, which are lisp-1). This is extremely uncommon in modern languages and complicates the language's adoption by younger programmers. For a discussion of the tradeoffs, see http://www.nhplace.com/kent/Papers/Technical-Issues.html).
0. It should be easy to `require` files in the same directory. Use case: getting up and running with multi-file elisp projects. So many elisp packages are thousands of lines long, which is crazy.
