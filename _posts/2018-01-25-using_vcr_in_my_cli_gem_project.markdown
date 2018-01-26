---
layout: post
title:      "Using VCR in My CLI Gem Project"
date:       2018-01-25 21:00:12 -0500
permalink:  using_vcr_in_my_cli_gem_project
---


<img src="https://c2.staticflickr.com/4/3403/3287862831_af523cddac_b.jpg" alt="Drawing" style="width: 800px;"/>
<p style ="font-size:0.75em"> Image courtesy andrewcurrie @ flickr.com</p>

### Background
My CLI Gem project scrapes a [cat adoption website](http://nkla.org/) for info on cats available for adoption at the moment. Halfway through coding the gem, I realized that I had been scraping the profile pages of 178 cats repeatedly, probably tens of times. From my days as a product manager working in advertising analytics, I know that many business rely on site traffic tracking tools to gauge interest in their sites, often breaking it down at the page level. While many of these tools have filters that make sure that automated traffic doesn't appear as real site visitors in metrics, I didn't know enough about how Nokogiri and Open URI work to make me confident that NKLA wouldn't be seeing my scraping as a sudden spike in interest in cat adoption, or even certain cats!

Something else I noticed was that my tests were taking a really long time to run. Since there was so much cat info to be scraped, every time I ran my tests I had to wait a few seconds, which started to get really annoying. I got the idea to use VCR to stop making so many requests to NKLA's site and speed up my tests from an example project provided in the lesson.

[VCR](https://github.com/vcr/vcr) is a gem that records all HTTP requests that your test suite makes and "replays" them for future test runs. This also makes sure that if you write tests expecting certain site content, and that content changes, your tests won't break. After I implemented it, my tests ran super quickly (after the initial run) and I was happy not to be pinging NKLA.org so much anymore.

This is by no means meant to be a how to for the general public on VCR ([there](https://relishapp.com/vcr/vcr/v/1-10-3/docs/test-frameworks/usage-with-rspec) [are](http://www.thegreatcodeadventure.com/stubbing-with-vcr/) [plenty](https://www.natashatherobot.com/vcr-gem-rails-rspec/) [of](https://revs.runtime-revolution.com/unit-testing-with-vcr-5dd2bb5c9012) [those](https://rubyplus.com/articles/1431-How-to-use-VCR-to-speed-up-unit-tests) [already](https://thinking.philosophie.is/increasing-your-rspec-test-speeds-with-vcr-5b5aceb82857)), but I do want to let other Flatiron students know that VCR is out there (as it wasn't directly mentioned in the lesson) and give some tips about what I learned implementing it for my CLI Gem with RSpec.

### Install VCR & Webmock
VCR works with a gem called webmock that stubs out HTTP requests in your program. Basically, instead of your program requesting the actual site from the internet when you run your tests, it will look in the "cassette" file(s) vcr creates. Those files have the extension ".yml" and contain a snapshot of the website, created the first time you run your tests.

To instal VCR, you do:
```
gem 'vcr'
gem 'webmock'
```

### Update your test files to work with VCR & Webmock
Then you have to update a few files to make VCR work with your program.


**gemspec file**

Add a development dependency for Webmock.
```
spec.add_development_dependency "webmock"
```


**spec_helper file**

Add configuration for VCR (it should create the fixtures/vcr_cassettes folder & the cassettes within when you run your tests).

```
VCR.configure do |config|
  config.cassette_library_dir = "fixtures/vcr_cassettes"
  config.hook_into :webmock
end
```


**Spec files**

Update your tests so that they use VCR for web requests. You can wrap VCR around each test individually, or add the below to the test file to wrap VCR around every test in the file. For my cassette file name I just used the name of my CLI gem.

```
around(:each) do |example|
  VCR.use_cassette("your_cassette_file_name", :record => :new_episodes) do
    example.run
  end
end
```

Now, I have added the :record => :new_episodes option here because I had multiple methods making HTTP requests. By default, VCR will only allow 1 HTTP request to create a cassette, then references that cassette for future HTTP requests. This makes sense when you think about the purpose of VCR. But try as I might, I couldn't figure any other way to allow VCR to work with my program, which made multiple HTTP requests to different pages (I'm sure there's a right way to do this... I just don't know what it is!). With this option, VCR allowed me to make more than 1 HTTP request and I was able to get my cassette file to contain data from both of the pages I was scraping.

### Run tests

At this point you can run your tests. The first time, the cassettes file should be created, and after that your tests should be nice and speedy!


*Note*: One downside to VCR is that if you generate new cassettes for the project, you may have to update your specs with the new info or they'll break.

### Wrapping Up

I definitely still don't understand all of how VCR works, even after reaping its benefits for my project, but I know a lot more than I did when I began! Hopefully some of this will be useful to other students working on this project.

