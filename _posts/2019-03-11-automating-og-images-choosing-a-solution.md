---
layout: post
title:  "Automating Open Graph Images: Choosing A Solution"
date:   2019-03-11
---

**Note:** This post assumes some basic knowledge of Open Graph images, if you need some context take a look at my previous post, [Implementing iMessage Link Previews][imessage-link-preview-post].
{: .note}

-----

## In This Post
{: .no_toc}

- 
  {:toc}

-----

## Background
[Push of Love][push-of-love] is a web/iOS/Android app that sends daily push notifications with motivational messages. 
Push of Love's website has individual pages for each piece of content that it sends. 

We're going to focus on enhancing the experience of sharing one of those pages.

## Our Goal
*Goal: fully automate Open Graph image generation based on the content of the page.*

When someone shares a link from Push of Love we want a rich experience that both demonstrates what Push of Love is and sufficiently represents the content of page that is being shared.

In other words... Let's make this happen without having to do any extra work:

![Push of Love iMessage Preview Example][push-of-love-imessage-preview-final]{:height="367" width="333"}

## Existing System Outline

### High Level Architecture

Our current system is made up of a React Native app for both [iOS][push-of-love-ios] and [Android][push-of-love-android] and a Nuxt.js/Vue.js web app that serves [pushoflove.com][push-of-love].

Each of the clients make requests to a Ruby on Rails API with a PostgreSQL data store both of which are hosted on Heroku.

![Push of Love High Level Architecture][push-of-love-high-level-architecture]{:height="403" width="394"}

We also use Cloudflare as a reverse proxy for both the Backend API and the Web Client and have it configured to cache all of our content on the edge.

### Process Outline

In the backend Rails API we have a `Notification` model that represents each message:

```rb
class Notification < ApplicationRecord
  validates :title, presence: true
  validates :message, presence: true

  scope :unsent, -> { where(sent_at: nil) }
end
```

#### Content Creation

My co-founder and I write the content for the Notifications a few days before they are sent (🤔 well, in an ideal world it's a few days in advance... in the real world we're definitely not that responsible 😬).

#### Notification Sending

We have a script that is scheduled to run multiple times a day (right now it's scheduled to run twice) that finds an unsent `Notification`, sends a push notification with the notification's content, and finally updates the notification's `sent_at` attribute with the current time.

## Determining a Solution

### Defining Selection Criteria

Before we dive into implementation, let's first consider what we want out of our solution and define the criteria that we will use to evaluate each of our options.

A good solution must:
- Satisfy our goal with a fully automated solution
- **Performance:** Have no negative performance impact on our existing application
- **Cost:** Keep server costs as low as possible (Push of Love has zero revenue right now 😅)
- **Resiliency:** Have graceful degradation; everything should work as it does now if this new process fails
- **Self-actualization:** Give me something new to learn 😄

### Generating Possible Solutions

**1. Rails Synchronous on Create**: Create image when a Notification is created

*Background Info: Currently we manually create Notifications from a One-Off Dyno in a Rails console. This is not ideal and we will eventually create an admin web interface for this process*
  - *In One-Off Dyno*
    - ✅ Simple solution
    - 🚫 If the image creation process breaks we won't be able to create new Notifications
    - ✅ Independent process means it should have minimal impact on application performance
    - ⚠️️ Some additional cost depending on added run time[^heroku-dyno-pricing]
  - *In Web Request*
    - ✅ Simple solution
    - 🚫 If the image creation process breaks we won't be able to create new Notifications
    - 🚫 May result in overall increased response time due to [how Heroku Routing works][heroku-routing-request-distribution]
    - 🚫 Needs to take less than 30-seconds due to Heroku's hard [30-second HTTP timeout][heroku-http-timeouts]

**2. Rails Synchronous on Request**: Create image when a Notification is requested and its image does not exist
  - ✅ No additional costs
  - 🚫 If the image creation process breaks we my not be able to display Notifications
  - 🚫 May result in overall increased response time due to [how Heroku Routing works][heroku-routing-request-distribution]
  - 🚫 Needs to take less than 30-seconds due to Heroku's hard [30-second HTTP timeout][heroku-http-timeouts]

**3. Nuxt.js/Vue.js Synchronous**: Create image when a Notification is requested and its image does not exist
  - *Same Pros/Cons as "Rails Synchronous on Request" option*
  - ✅ Moves image creation to where we use it
  - 🚫 Dynamic creation of image content/routes might be tricky in Nuxt.js/Vue.js

**4. Rails Background Job**: Trigger a job when a Notification is created that will create the image
  - ✅ Ruby/Rails has some solid background job frameworks: *Sidekiq, Resque, and DelayedJob*
  - ✅ Background jobs will not tie up web resources
  - ✅ If the image creation process breaks it will have minimal impact on the application
  - 🚫 Requires a separate worker machine which will cost ~$7/mo[^heroku-dyno-pricing]
  - 🚫 *Sidekiq* and *Resque* require Redis. This adds complexity to the app and ~$15/mo in cost[^heroku-redis-pricing]
  - 🚫 *DelayedJob* uses your app's main data store which, in my opinion, is usually a bad idea

**5. Rails Cron Job**: Periodically create images for all Notifications without an image
  - ✅ Simple solution
  - ✅ Independent process means it should have minimal impact on application performance
  - ⚠️ Some additional cost depending on added run time[^heroku-dyno-pricing]
  - 🚫 Leads to large batches introducing scale concerns and complexity around failures/retries
  - 🚫 Increases chance of image not being created before Notification is sent

**6. AWS Lambda**: Trigger an AWS Lambda function when Notification is created
  - ⚠️ Introduces new dependencies between services
  - ✅ Independent process means it should have no impact on application performance
  - ✅ Automatic scalability
  - ✅ No additional costs (thanks to AWS Lambda's free tier[^aws-lambda-free-tier])

### Evaluating and Selecting a Solution

To help make an objective decision when selecting the best solution, I like to take each of the options we came up with and score them on the criteria we generated previously. 

We scored each of the options between 1 and 3 with 3 being really good and 1 being really bad.

Here are the results:

|                              | Performance | Cost  | Resiliency | Self-Actualization | Total  |
| :--------------------------- | :---------: | :---: | :--------: | :----------------: | :---:  |
| Rails Synchronous on Create  | 1           | 2     | 2          | 1                  | 6      |
| Rails Synchronous on Request | 1           | 2     | 1          | 1                  | 5      |
| Nuxt.js/Vue.js Synchronous   | 1           | 2     | 1          | 2                  | 6      |
| Rails Background Job         | 2           | 1     | 1          | 1                  | 5      |
| Rails Cron Job               | 1           | 2     | 2          | 1                  | 6      |
| **AWS Lambda**               | **3**       | **3** | **3**      | **3**              | **12** |

Boom! AWS Lambda FTW!

Stay tuned for the next post about how we implemented this!

---

[push-of-love-imessage-preview-final]: {{ site.url }}/assets/push-of-love-imessage-preview-final.png
[push-of-love-high-level-architecture]: {{ site.url }}/assets/push-of-love-high-level-architecture.png

[imessage-link-preview-post]: 2019-03-05-implementing-imessage-link-previews.md
[push-of-love]: https://pushoflove.com
[push-of-love-ios]: https://itunes.apple.com/us/app/push-of-love/id1418962368?mt=8
[push-of-love-android]: https://play.google.com/store/apps/details?id=com.pushoflove.app
[heroku-routing-request-distribution]: https://devcenter.heroku.com/articles/http-routing#request-distribution
[heroku-http-timeouts]: https://devcenter.heroku.com/articles/request-timeout
[heroku-pricing]: https://www.heroku.com/pricing
[heroku-redis]: https://elements.heroku.com/addons/heroku-redis
[aws-lambda-pricing]: https://aws.amazon.com/lambda/pricing/

[^heroku-dyno-pricing]: [Heroku Hobby Dynos][heroku-pricing] currently cost $7/dyno/month prorated to the second
[^heroku-redis-pricing]: [Heroku Redis][heroku-redis]'s lowest tier with Persistence is $15/month
[^aws-lambda-free-tier]: Currently, [AWS Lambda's free tier][aws-lambda-pricing] includes 1M free requests per month and 400,000 GB-seconds of compute time per month. 
