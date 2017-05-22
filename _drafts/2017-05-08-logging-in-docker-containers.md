---
layout: post
title:  Logging in Docker containers
date:   2017-05-08
categories: [TIL]
---

Logs can get buffered before they're shipped. This happened to me today and it could have been Logspout or Docker that was holding onto them. If you want to see logs move through your system, wrap your `curl` requests in a while.

Public Ansible documentation is tracks the github master branch! It can be out of sync with the current stable version :O. There is no easy way of seeing docs for old versions. See the [github issue](https://github.com/ansible/ansible/issues/10967) complaining about this.

