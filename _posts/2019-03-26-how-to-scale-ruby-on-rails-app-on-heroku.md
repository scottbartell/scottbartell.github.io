---
layout: post
title:  "Scaling a Ruby on Rails app on Heroku"
date:   2019-03-26
description: "How to Scale a Ruby on Rails app with a Postgres datastore that is hosted on Heroku"
---

I have had to spend a considerable amount of time scaling Rails applications for things like major product launches to live TV events like [SharkTank][plated-shark-tank] and [Beyond The Tank][plated-beyond-the-tank] and national TV marketing campaigns.

From all that experience, the most effective way I have found to scale a web application is to do the following:
1. Implement sufficient application performance monitoring (I use [NewRelic][heroku-new-relic])
1. Gather data (ideally, load test until you scale out)
1. Identify what the bottleneck was
1. Fix the bottleneck
1. 🔁 Repeat (until you hit a goal or run out of time)

The reason why this is so effective is because it ensures that you spend your time fixing the actual constraint of your system. 

A great way to think about this come from the [Theroy of Constraints][theory-of-constraints]:
> Any improvements made anywhere besides the bottleneck are an illusion. -Gene Kim

So, in my experience, when scaling a Rails app the bottleneck typically falls into one of two large areas:
1. The Database (Heroku Postgres in our case)
2. The Rails Application 

Let's dive into each of those.

## Heroku Postgres

### Goals

- 📉 Decrease time spent per database query
- 📉 Decrease number of database queries
- 📈 Increase database connection utilization
- 📈 Increase number of application requests per database connection

### Solutions

- [Update your Heroku Postgres database][heroku-pg-update-plan] plan
- [Upgrade Postgres to latest][heroku-pg-upgrade-version] version
- Implement caching in front of the Database where appropriate
- Optimize / remove any long running and/or time consuming queries
- Reduce number of queries by rewriting application to make fewer queries
- Optimize / configure DB connection pooling
    - Optimize [`ActiveRecord`'s connection pool][ar-connection-pool]
    - Implement/Optimize [`PgBouncer`][heroku-connection-pooling]
    - Ensure your application does not run out of DB connections
- Configure a [`statement_timeout`][pg-statement-timeout]
- Ensure your database is properly indexed (for both reads and writes)
- Configure a [read only follower database][heroku-follower-db] and use where appropriate
- Ensure DB backups are configured to run off-peak hours


## Heroku Rails Web Application

### Goals
- 📉 Decrease application throughput
- 📉 Decrease application response time (95th percentile)

### Solutions

- Upgrade Ruby to latest version
- Reduce throughput/requests to your application
  - Serve static content (including assets/HTML/JSON) from a CDN e.g) [Cloudflare][cloudflare]
  - Cache content that can be stale (including HTML/JSON) in a CDN e.g) [Cloudflare][cloudflare]
  - Break off any high throughput/independent parts of your app into independent applications
  - Optimize clients to make fewer requests to application
  - Properly and aggressively set [Cache-Control headers][cache-control-headers]
- Cache expensive calculations and/or queries
  - On the Rails server in memory, in Redis, on an Edge server, on the client, or any combination
- Optimize your web server (`Puma`)
  - Test multiple `Puma` configurations including multi-threaded, multi-process, and a combination
- Upgrade to [Performance web dynos][heroku-web-dynos] (dedicated)
- Off load any long running processes into background jobs (`Sidekiq`)
- Have an aggressive request timeout solution ([`RackTimeout`][rack-timeout])
- Ensure any external API request clients have aggressive timeouts configured
- Configure database to have a [`statement_timeout`][pg-statement-timeout]
- Optimize database performance/queries/indexes ([see above](#heroku-postgres))

[heroku-new-relic]: https://elements.heroku.com/addons/newrelic
[theory-of-constraints]: https://en.wikipedia.org/wiki/Theory_of_constraints
[plated-shark-tank]: https://blog.heroku.com/customers_shark_tank#plated-aired-4-4-at-9pm-et-on-abc
[plated-beyond-the-tank]: https://www.plated.com/morsel/shark-tank-episode-recap-plated-founders-nick-taranto-josh-hix/
[pg-statement-timeout]: https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-STATEMENT-TIMEOUT
[ar-connection-pool]: https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/ConnectionPool.html
[heroku-connection-pooling]: https://devcenter.heroku.com/articles/postgres-connection-pooling
[heroku-follower-db]: https://devcenter.heroku.com/articles/heroku-postgres-follower-databases
[heroku-web-dynos]: https://devcenter.heroku.com/articles/dyno-types
[rack-timeout]: https://github.com/heroku/rack-timeout
[cache-control-headers]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control
[cloudflare]: https://www.cloudflare.com/
[heroku-pg-update-plan]: https://devcenter.heroku.com/articles/updating-heroku-postgres-databases
[heroku-pg-upgrade-version]: https://devcenter.heroku.com/articles/upgrading-heroku-postgres-databases
