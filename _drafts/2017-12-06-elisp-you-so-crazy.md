---
layout:     post
title:      Elisp you so crazy
date:       2017-12-06
summary:    💰💰💰
categories: [emacs, elisp]
---

- Namespace, or lack of. Have to prepend everything (variables, functions) with `my-module-name-`
- Managing syntax tables:

  ```
  (modify-syntax-entry ?\/ ". 23" a-syntax-table)
  ```

  The period, the space after the period, and the `12b` are all flags. Why couldn't they just be positional parameters with meaningful symbol names?

  ```
  (modify-syntax-entry ?\/
                       'punctuation
                       nil
                       (comment-starter 2)
                       (comment-ender 1)
                       a-syntax-table)
  ```

  or, you know, why can't we have keyword arguments? Racket does it.
