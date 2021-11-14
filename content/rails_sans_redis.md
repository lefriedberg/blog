+++
title = "Rails sans Redis"
date = 2021-11-14
draft = false
[extra]
show_date = true
+++

A quick story from earlier this summer.  My product counterpart wanted to be able to "login as" our users. I proposed we use (devise masquerade)[https://github.com/oivoodoo/devise_masquerade]. It's a simple little library that adds just the functionality I was looking for. Unfortunately, after reading the code, I realized it relies on `Rails.cache`. We don't have a cache store configured. Rails defaults to a file store in production. However, we use heroku, which has an (ephemeral file system)[https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem] so we can't rely on it for our cache. Often times, rails applications are already using redis for their queue or websockets and so it's convenient for caching use cases like this one.

What are our options? 
1. Use file store - volitile
2. Add redis or mem_cache

This feature will be used a few times per day. Maybe 100 when we're busy. We're not using redis for anything else, so it feels bizarre to add it just for this use case. I wish I could 

Thus I wrote this blog post.

You might not need redis, at least not right now.

Redis is great. It's a great technology and I have no complaints about it. I have nothing against redis, and this article doesn't contain any technical complaint against redis.

My basic premise is that, many rails applications use redis because has become the community default. 
Also, if you're willing to 

Less moving parts makes your life easier: 
1. Setting up local development environments is faster and less bug prone
2. You only have 



Typical redis use cases: 
1. Websocket sessions for ActionCable
1. Job queue for Sidekiq
1. Cache for `Rails.cache`
1. Session store
1. Database / key value store

Your RDBMs can do all of these things (within some bounds) 

1. Websocket sessions via postgres
1. DelayedJob or GoodJob or Queue
1. Cache in your database
1. ActiveRecord session store
1. Use a key value store like `Github::DS` 


Will it scale? 
For a while.


Each of these use cases has their own scaling dynamics.

If you're using websockets heavily, don't use postgres as an adapter. However, if you only need it for 20 users in your internal admin tool, go ahead, it will save you operational complexity.
