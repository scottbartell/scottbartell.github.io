---
layout: post
title:  "Automating Rails Database Migrations on Heroku"
date:   2020-01-22
description: "How to configure Heroku to automatically run database migrations for a Rails app during deployment"
---

One problem that teams tend to run into with Rails apps on Heroku is forgetting to run database migrations before the code that depends on them is released.

Luckily, there is a way to automate the running of database migrations on Heroku and to guarantee that they are always run before the dependent code is released: the [Heroku Release Phase][heroku-release-phase].

## Heroku Release Phase
The Heroku Release Phase allows you run certain tasks [after your app is built but before the app is released][heroku-release-phase-when-run].

![Heroku Release Phase Diagram][heroku-release-phase-diagram]

## Specifying release phase tasks

To specify the tasks to run during release phase, define a `release` process type in your app’s [Procfile][heroku-procfile].

In our case we want to run database migrations directly in our release task so we do the following:
```
web: bundle exec puma -C config/puma.rb
release: bundle exec rake db:migrate
```

That's all you need to do. The next time you push code Heroku will see that you have a release task configured and automatically run `bundle exec rake db:migrate` after the app is built.

## Notable Implications

In my experience this flow works really well. It helps guarantee that we never forget to run migrations, makes it so that we don't have to worry about code being released before a migration is run, and couples the running of the database migrations with the releasing of code (giving us an audit trail for when migrations were run).

That being said, this does introduce some complexity and some new cases that we will need to consider.

#### 1. If the Release Phase fails the app will not be deployed

If the [Release Phase ever fails][heroku-release-phase-fail] (in our case, if the migration fails) the app will not be released. 
This means that, in order to avoid the database ending up in a partially migrated state, we need to ensure all database migrations are wrapped in transactions.
For Rails apps this typically isn't a problem because by default `ActiveRecord` wraps all migrations in a transaction.
However, if we ever want to take advantage of something like [Postgres's `CONCURRENTLY` option for `CREATE INDEX`][postgres-concurrent-indexes], or run some sort of large data backfill outside of a transaction, you might need to disable transactions.

#### 2. The Release Phase runs on Review apps
If you are using [Heroku Review Apps][heroku-review-apps] you should consider what will happen when the release task runs in that environment.

For example, if you have configured Review Apps to share the same database as your staging environment, when a Review App is deployed the release task will run which will run database migrations in the Review App code against the staging database.

One potential solution to this is to have the release task look for an environment variable that is configured differently for Review Apps:

Create a new file in `bin/release-tasks`:
```sh
#!/usr/bin/env bash

if [ "$DISABLE_RAILS_DB_MIGRATIONS" = "true" ]; then
  echo "SKIPPING DATABASE MIGRATIONS"
else
  bundle exec rake db:migrate
fi
```

Update `Procfile` to call the new task:
```
...
release: bin/release-tasks
```

#### 3. Database Migrations will always run before the new code is released
All database migrations will run before the new code is shipped. 
This means that certain schema changes (such as column renames) may result in exceptions and/or downtime.

In order to avoid this you must ensure all of your schema changes/database migrations support both the new code as well as the code that is running before the new code is released.

[heroku-procfile]: https://devcenter.heroku.com/articles/procfile
[heroku-release-phase]: https://devcenter.heroku.com/articles/release-phase
[heroku-release-phase-when-run]: https://devcenter.heroku.com/articles/release-phase#when-does-the-release-command-run
[heroku-release-phase-fail]: https://devcenter.heroku.com/articles/release-phase#release-command-failure
[postgres-concurrent-indexes]: https://thoughtbot.com/blog/how-to-create-postgres-indexes-concurrently-in
[heroku-review-apps]: https://devcenter.heroku.com/articles/github-integration-review-apps

[heroku-release-phase-diagram]: {{ site.url }}/assets/heroku-release-phase-diagram.png
