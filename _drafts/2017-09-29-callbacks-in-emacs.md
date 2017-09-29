---
layout:     post
title:      Callbacks in Emacs lisp
date:       2017-09-29
summary:    💰💰💰
categories: [emacs, elisp]
---

NEED TO FIND DIFFERENT API THAT USES CALLBACKS BECAUSE THE TIMER CREATED BY `run-at-time` DOESN'T CLEAN UP ITSELF AND CLEANING IT UP MANUALLY IS A PAIN

I wanted to create a simple stopwatch in Emacs, and to do this I had to learn how to create and refer to callback functions in elisp. Inserting text into the current buffer after 5 seconds have elapsed is easy enough:

```elisp
(defun insert-dummy-text ()
  (insert " ... 5 second have elapsed ... "))

(run-at-time 5 nil 'insert-dummy-text)
```

If we want to keep track of state across invocations of the callback, things quickly get complicated. Dynamic binding by default, so mimicking closures doesn't work:

```elisp
(defun insert-dummy-text-at-most-twice ()
  (let ((number-times-called 0))
    (lambda ()
      (setq number-times-called (1+ number-times-called))
      (when (greater-than????? number-times-called 2)
        (print "thing")))
)
```

Use lexical binding instead:

```elisp
;; -*- lexical-binding: t; -*-

(defun insert-dummy-text-at-most-twice ()
  (let ((number-times-called 0))
    (lambda ()
      (setq number-times-called (1+ number-times-called))
      (when (greater-than????? number-times-called 2)
        (print "thing")))
)

```

