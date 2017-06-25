---
layout:     post
title:      Data fixes in database migrations
date:       2016-03-06
summary:    
categories: [migrations]
---

By data fix I mean a change to records in a table, as opposed to a change to a table schema. And yesterday, data fixes in a migration broke my tests. Not broke as in "these tests failed when they should have passed", but broke as in "I couldn't run any tests at all until I found the magical rake task that sidestepped the migrations". (`rake db:schema:load` in case you were wondering.)


## Awkward bedfellows

The thing with migrations is that they need to run on *all* instances of your app database. This includes local development databases and test databases (which will be empty if you are not using fixtures).

Data fixes, on the other hand, assume that the database tables contain broken data to begin with. A data fix might do something like this, for example:

```rb
thailand_tax = Taxes.where(country: 'Thailand').first
thailand_tax.sales = 0.1
thailand_tax.save!
```

This is fine and dandy so long there is a tax record for Thailand, but if there isn't, you're going to get a `undefined method 'sales' for nil` error, killing your TDD flow.

Things could get even weirder if there are multiple instances of your app running, which might be the case if customers can install the app on their own network. In this case, writing a data fix requested by one customer as a data migration will lead to all of your other customers' data changing as well. Happy happy fun times.



## Data migrations in rake tasks

The standard way of implementing data fixes is as a rake task, which are only run when you run them. While this is exactly the flexibility we want, it can also be an operational risk. If on top of production you have separate QA and user acceptance testing environments that sync against production data, you'll to have run the task three times on three different boxes over ssh. That's three manual interventions to remember and three opportunities to make a mistake.

This is exactly the situation my team found itself in
