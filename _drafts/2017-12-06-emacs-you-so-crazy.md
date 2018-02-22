---
layout:     post
title:      Emacs you so crazy
date:       2017-12-06
summary:    💰💰💰
categories: [emacs, elisp]
---

## Elisp

- Namespace, or lack of. Have to prepend everything (variables, functions) with `my-module-name-`

- Importing code from another file is a pain! No simple way to import another file in the same directory.

- Elisp packages have metadata in comments of *every file*. As in, every file in an emacs pacakage is supposed to start with something like this:

  ```
  ;; My rad package --- It does this useful thing
  ;;; Commentary:

  ;; It's a package.

  ;;; Code:
  ```

  and end with:

  ```
  ;;; my-rad-package.el ends here
  ```

  Because if we didn't have a helpful comment to tell us where the file ends, what's to stop us scrolling down for ever?

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
