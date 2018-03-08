---
layout:     post
title:      Customising spacemacs for colemak
date:       2017-09-09
summary:
categories: [literature, novels]
---

- `SPC h d k`: describe-key, you enter a key combination and spacemacs tells you what that combination is bound to, as well as which keymap the binding exists in! This is super duper handy.
- If a key binding doesn't work, check that it isn't being shadowed by another key binding. This is particularly common for bindings in `evil-motion-state-map`, which is shadowed by `evil-normal-state-map`
