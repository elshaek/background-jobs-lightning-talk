# Background Jobs

## What
- refers to (asynchronous) work that is processed outside of the usual request/response workflow 
- Usual: web applications receive a request from the outside world, do some processing (such as querying a database) and immediately return a response within a few milliseconds

![diagram-no-bg-service](https://s3.amazonaws.com/heroku-devcenter-files/article-images/310-imported-1443570179-Screen-20shot-202012-04-12-20at-204.00.09-20PM.png)

- Background jobs: 
    + work that require a longer time to complete than the normal request
    + these requests cannot be processed immediately and return a response (∴ asynchronous)
    + In order to not interrupt the normal synchronous workflow of an application, asynchronous tasks are .

![diagram-bg-service](https://s3.amazonaws.com/heroku-devcenter-files/article-images/310-imported-1443570182-Screen-20shot-202012-04-12-20at-203.59.12-20PM.png)


#### Examples of tasks that could be processed as background jobs:
- fetching data from remote APIs
- resizing images
- uploading data to Amazon Simple Storage Service (S3)
- sending newsletters


## Why use background jobs?

![diagram-no-bg-service-delay](https://s3.amazonaws.com/heroku-devcenter-files/article-images/310-imported-1443570180-Screen-20shot-202012-04-12-20at-202.49.38-20PM.png)

- ensures that web requests can always return immediately
- reduces compounding performance issues that occur when requests become backlogged
- they transfer both time and computationally intensive tasks from the web layer to a background process outside the user request/response lifecycle
- ∴ helps in building scalable web apps


## Notes
- ~ Rails version 4.2 has built-in support for executing background jobs using Active Job
- Rule of thumb: avoid web requests which run longer than 500ms. If app has requests that take one, two, or more seconds to complete, consider using a background job instead


## Active Job
- framework for declaring jobs and making them run on a variety of queueing backends (e.g. regularly scheduled clean-ups, to billing charges, mailings)

#### Create the job

`rails generate job guests_cleanup` will create: app/jobs/guests\_cleanup\_job.rb

```ruby
class GuestsCleanupJob < ActiveJob::Base
  queue_as :default
 
  def perform(*args)
    # Do something later
  end
end
```

#### Enqueue the job
```ruby
# Enqueue a job to be performed as soon the queueing system is free
MyJob.perform_later record
```

```ruby
# Enqueue a job to be performed tomorrow at noon
MyJob.set(wait_until: Date.tomorrow.noon).perform_later(record)
```

```ruby
# Enqueue a job to be performed 1 week from now
MyJob.set(wait: 1.week).perform_later(record)
```

#### Job execution
Rails only provides an in-process queuing system, which only keeps the jobs in RAM (if the process crashes or the machine is reset, then all outstanding jobs are lost with the default async backend). ∴ we need to set up a queuing backend with a 3rd-party queuing library for enqueuing and executing jobs in production.
Active Job has built-in adapters for multiple queuing backends (Sidekiq, Resque, Delayed Job and others): [API Documentation for ActiveJob::QueueAdapters](http://edgeapi.rubyonrails.org/classes/ActiveJob/QueueAdapters.html)

Setting the backend:

```ruby
# config/application.rb
module YourApp
  class Application < Rails::Application
    # Be sure to have the adapter's gem in your Gemfile
    # and follow the adapter's specific installation
    # and deployment instructions.
    config.active_job.queue_adapter = :sidekiq
  end
end
```


## Gems
- [Resque](https://github.com/resque/resque): gem 'resque'
- [Sidekiq](https://github.com/mperham/sidekiq): gem 'sidekiq' (Sidekiq relies on Redis)
- [Delayed::Job (or DJ)](https://github.com/collectiveidea/delayed_job): gem 'delayed_job_active_record'


## References:
- [Rails Guides](http://edgeguides.rubyonrails.org/active_job_basics.html)
- [Heroku Dev Center](https://devcenter.heroku.com/articles/background-jobs-queueing)
- [Which Ruby background job framework is right for you?](http://blog.scoutapp.com/articles/2016/02/16/which-ruby-background-job-framework-is-right-for-you)