---
layout:     post
title:      Emacs you so crazy
date:       2017-12-06
summary:    💰💰💰
categories: [emacs, elisp]
---

As Tristan Hume [says](http://thume.ca/2017/03/04/my-text-editor-journey-vim-spacemacs-atom-and-sublime-text#brokenness), "everything in Emacs, and especially Spacemacs, is a hack".

Emacs refuses to do decide on *anything*, which means that conventions are all over the place and packages tread on each other and double up on work. And the code, oh my god, the code. The fact that the built in help docs link to the source code for *every* function and global variable in Emacs is amazing, but after diving under the covers a handful of times you start to hold your nose before the next adventure.

- Syntax highlighting. This is as fundamental as it gets for a text editor. There should be one way to do it, and it should robust. In Emacs, you have three. Syntax tables provide a first, basic layer which is often used to take care of things like comments (but also things like navigation, because, why not?). You can manipulate a syntax table using delightfully transparent calls such as

  ```
  (modify-syntax-entry ?\/ ". 23" a-syntax-table)
  ```

  That magic string, `". 23"`? That sets a group of flags. Yum.

  If that doesn't cut it, and it won't, you can use font lock mode. Font lock mode lets you pick out chunks of syntax using lists of regular expressions. The constructs can get rather complicated. They're also a little slow, so by default when you modify text, the syntax highlighting will be reavaluated for only that line. In theory you should still be able to get font lock mode to highlight just about anything, but in practice this is far from the case. I've seen multiline syntax highlighting and nested string interpolation that is completely broken. For example, both ruby mode and terraform mode choke on code that looks like

  ``` ruby
  "Some text #{my_method("nested string #{nested_interpolation}")}"
  ```

  Ruby mode will simply give up and colour the rest of the file as a string.

  The only mode that I've seen get nested interpolation right is `js2-mode`, a javascript mode that was originally written by Steve Yegge. The reason it gets it right is that it skips font lock mode and directly parses the javascript.

- Syntax errors in `js2-mode` that overlap with `flycheck` syntax errors.

- I noticed that while editing elisp code if I switched to another window and back to Spacemacs, my cursor would jump to another point in the buffer, seemingly at random. After digging around for a couple of hours, I discovered the following chain of execution:

  1. On switching windows, a `focus-out-hook` that I defined fires, saving all unsaved buffers.
  2. The `after-save-hook` hooks fire. In particular, `auto-compile-byte-compile`, which was added to the hook by the `auto-compile` package, is executed.
  3. To check whether it should try to compile the current buffer, `auto-compile-byte-compile` first checks that there aren't any unbalanced parantheses by calling `check-parens`, which is defined in core Emacs.
  4. If `check-parens` finds an unbalanced parenthesis pair, it will helpfully move the pointer to the offending parenthesis. It will also print a message to the minibuffer stating the problem, but this was being obscured by another message from a failing call to `re-search-forward` (that was a lovely goose chase that took an hour of my time).

  Why is `check-parens` modifying the pointer position at all? If there are unbalanced parens, I don't want my pointer neurotically jumping back and forth, and a minibuffer message is a terrible way to communicate the problem to me. There is a standard solution to this problem, and that's syntax errors provided by the likes of [flycheck](http://www.flycheck.org/en/latest/). But of course, because core Emacs doesn't have any opinions about communicating static code errors to users, we get this bizarro behaviour that doesn't fit with anything else.

## Elisp

- Namespaces, or lack of. Have to prepend everything (variables, functions) with `my-module-name-`

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
