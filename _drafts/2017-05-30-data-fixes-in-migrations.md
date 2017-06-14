---
layout:     post
title:      Data fixes in database migrations
date:       2016-03-06
summary:    
categories: [migrations]
---

By data fix I mean a change to records in a table, as opposed to a change to a table schema. And yesterday, data fixes in a migration broke my tests. Not broke as in "these tests failed when they should have passed", but broke as in "I couldn't run any tests at all until I found the magical rake task that sidestepped the migrations".


## Awkward bedfellows

The thing with migrations is that they need to run on *all* instansces of your app database. This includes local development databases and test databases (which will be empty if you are not using fixtures). Ruby on Rails, for example, will not let you run any tests until you have run all of the migrations against the test database.

Data fixes, on the other hand, assume that the database tables contain broken data to begin with. A data fix might do something like this, for example:

```rb
thailand_tax = Taxes.where(country: 'Thailand').first
thailand_tax.sales = 0.1
thailand_tax.save!
```

This is fine and dandy so long there is a tax record for Thailand, but if there isn't you're going to get a `undefined method 'sales' for nil` error. Which will 
https://relishapp.com/rspec/rspec-rails/docs/upgrade#pending-migration-checks

There is one inconvenient thing about using rake tasks for data fixes: they need to be applied manually for every environment. If you have staging, user acceptance testing and production enviroments for example, you're going to have to run the task three times. That's three opportunities to make a mistake and three lots of deploys to wait for.
