---
layout:     post
title:      Define custom inspect methods in delegators
date:       2017-06-19
summary:
categories: [ruby]
---

Otherwise debugging is hard!

I had an expect().with(model) test that passed, despite Delegator.new(model) being passed in, because the delegator was delegating #==, and printouts were hard to understand because #inspect was also being delegated! So in delegators, define custom inspect methods, something along the lines of `MyDelegator(#<Delegatee:0x0abcd>, object_id=0x12345)`
