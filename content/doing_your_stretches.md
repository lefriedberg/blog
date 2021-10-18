+++
title = "Doing your stretches"
date = 2021-10-10
draft = false
[extra]
show_date = true
+++

Last weekend I sped up a test suite from ~140 seconds to ~15 seconds by fixing a devise configuration mistake. I wrote this post to describe the issue, walk through how I profiled the test suite and reflect on why performance issues like this can go unnoticed.

# Backstory

Earlier this spring I started a new project. Instead of using the traditional rails application layout, I chose a strategy that I've become familiar with at my last two jobs. I like it for a few reasons, not least of which is that it skirts the typical problem of a 1000 line `user.rb` file. You can read more about it [here](https://betterment.engineering/whats-the-best-authorization-framework-none-at-all-9bd315c3729b).

TLDR: We have a monorepo with four rails projects:
```
/service_provider_application
/homeowner_application
/internal_agent_application
/shared_rails_engine
```
The rails engine is a dependency of all apps and owns all modeling and business logic. Each app is designed to be used by a single persona. The personas in our domain are typical of a B2B2C project, we have business users (service providers), consumers (homeowners) and internal agents (agent).  Because we don't want to maintain bespoke authentication, we use [Devise](https://github.com/heartcombo/devise). Each user (service provider, homeowner and agent) is a devise user who can authenticate into their own app.

There are some differences in how these three users authenticate so we have a configuration - `config/initializers/devise.rb` - in each application. Devise generates this file when you install it. The shared rails engine mostly doesn't care about how users authenticate, but, as our users are central to our business logic so we still have a configuration file that looked like:
```rb
require "devise"

Devise.setup do |config|
  require "devise/orm/active_record"
end
```

# Inciting incident

When we started in April our test suites were ripping fast - if only because they were small. By October, our test suite in the shared rails engine had grown to ~700 tests and ran for just over 2 minutes.

I wanted it to be faster but was busy delivering features to get our business off the ground. One morning, I had a split screen and was running our homeowner's request tests in one pane and some unit tests in the other. I noticed that the unit tests were bizarrely slower than the request tests. This piqued my interest because it indicated that there might be some low hanging fruit for improving performance.

# Profiling our test suite

To profile our tests, I installed the [test-prof](https://github.com/test-prof/test-prof) and [stackprof](https://github.com/tmm1/stackprof) gems locally
```rb
gem "test-prof"
gem "stackprof"
```

And then ran `TEST_STACK_PROF=1 TEST_STACK_PROF_FORMAT=json rspec spec/models/homeowner_spec.rb`


Then I used [speedscope](https://www.speedscope.app/) to help narrow the problem down.
<img width="1434" alt="Speedscope screenshot" src="/speedscope_screenshot.png">

This made me realize that we were spending 6 out of 7 seconds of test execution time encrypting passwords for homeowners. Surprising!

I realized that we set the "stretches" to 1 in the apps that own each devise user, but not in the shared rails engine.

To confirm this theory, I ran the following in the shared rails engine console and in the homeowner console.
```rb
puts Benchmark.measure { FactoryBot.create(:homeowner) }
```

Sure enough, it took ~ 300 miliseconds in the shared engine and ~3 miliseconds in the homeowner application! 

Ultimately, this one line change cut our test time (in circle ci) from ~140 seconds to ~15 seconds - almost a 10x difference, a whole order of magnitude! 

```rb
  config.stretches = Rails.env.test? ? 1 : 12
```

# Reflection

First off, I'd like to thank the authors of test-prof, who allowed my to identify a performance issue in less than 20 minutes.

Why did this issue persist for the last few months? 
1. I setup devise before we already had tests so there was no "regression"
1. The tests were "fast enough". 2 minutes is not the end of the world. If it were 10 minutes, I'd prioritize it higher.
1. The speed of generating a factory is a black box.
