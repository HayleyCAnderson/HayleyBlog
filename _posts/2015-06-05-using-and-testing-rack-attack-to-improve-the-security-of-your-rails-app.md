---
layout: post
title: Using (and Testing) Rack::Attack to Improve the Security of Your Rails App
category: Rails
---

Rack::Attack is a rack middleware intended to protect Rails
applications through customized throttling and blocking. I started using it
after attending a talk from the person who created it, and I thought it was
a brilliant option for developers who want to increase the security of their
applications with minimal effort.

You can prevent attempts at blunt-forcing passwords by throttling requests with
the email or username being attacked, or thwart troublesome scrapers or other
offenders by throttling requests from IP addresses making large volumes of requests.
Rack::Attack makes protecting your applications easy but still provides quite a
bit of freedom to choose what to throttle, block, whitelist, or blacklist.

### Using Rack::Attack

Setting up Rack::Attack is relatively easy and well explained in the
[documentation](https://github.com/kickstarter/rack-attack), but I'll include
the entire setup process here. First, of course, add `gem "rack-attack"`.
Next, set up production caching. Add the gem(s) you'd like to use
and configure in `config/environments/production.rb`. I'm using "memcachier"
and "dalli". If you're using Heroku, which I am, you'll
also need to add the
[MemCachier add-on](https://devcenter.heroku.com/articles/memcachier), which
will provide your environmental variables.

{% highlight ruby %}
config.cache_store = :dalli_store,
                      (ENV["MEMCACHIER_SERVERS"] || "").split(","),
                      {
                        username: ENV["MEMCACHIER_USERNAME"],
                        password: ENV["MEMCACHIER_PASSWORD"],
                        failover: true,
                        socket_timeout: 1.5,
                        socket_failure_delay: 0.2
                      }
{% endhighlight %}

Now that caching is set up, you need to set up Rack::Attack itself. First, add
`config.middleware.use Rack::Attack` to `config/application.rb`. Then create the
file `rack-attack.rb` under `config/initializers`. This is where you can
customize your use of Rack::Attack. Here's my configuration, based on the
[example](https://github.com/kickstarter/rack-attack/wiki/Example-Configuration)
from the Rack::Attack documentation.

{% highlight ruby %}
class Rack::Attack
  # Throttle high volumes of requests by IP address
  throttle('req/ip', limit: 20, period: 20.seconds) do |req|
    req.ip unless req.path.starts_with?('/assets')
  end

  # Throttle login attempts by IP address
  throttle('logins/ip', limit: 5, period: 20.seconds) do |req|
    if req.path == '/admins/sign_in' && req.post?
      req.ip
    elsif req.path == '/users/sign_in' && req.post?
      req.ip
    end
  end

  # Throttle login attempts by email address
  throttle("logins/email", limit: 5, period: 20.seconds) do |req|
    if req.path == '/admins/sign_in' && req.post?
      req.params['email'].presence
    elsif req.path == '/users/sign_in' && req.post?
      req.params['email'].presence
    end
  end
end
{% endhighlight %}

My version is different from the configuration mainly because I ran into a lot
of problems when I tried to test it. First, I discovered that using longer time
periods like 300 requests per 5 minutes is not feasible for testing. Second,
I'd assumed that for multiple login pages, it would simply work to make separate
throttle blocks. Testing said otherwise, which does make sense, and thus the
elsif statements. Finally, I discovered while clicking around locally that
assets do, in fact, need to be excluded, if just to avoid headaches while in
development.

### Testing Rack::Attack

So, testing this is pretty important, but how do you test it? Thanks to
[this post](http://www.mavengineering.com/blog/2014/06/20/how-to-test-rack-attack-with-rspec/),
I was able to figure out how this is possible. First and most importantly,
you need to add the [Rack::Test gem](https://github.com/brynary/rack-test):
`gem 'rack-test', require: 'rack/test'`.
Then, I placed my tests in `spec/config/initializers/rack-attack_spec.rb`.
However you want to test, the setup for Rack::Test should be the same (though
changed accordingly if you're not using RSpec):

{% highlight ruby %}
require 'rails_helper'

describe Rack::Attack do
  include Rack::Test::Methods

  def app
    Rails.application
  end

  # Your tests
end
{% endhighlight %}

When it comes to the actual tests, you have to be careful with email addresses
and IP address to keep the tests from interfering with each other, either by
causing other tests with the same credentials to fail or by causing other tests
to falsely pass by throttling by IP address instead of by email or vice versa.
So - warning, large block of code ahead! - these are the tests I wrote:

{% highlight ruby %}
describe "throttle excessive requests by IP address" do
  let(:limit) { 20 }

  context "number of requests is lower than the limit" do
    it "does not change the request status" do
      limit.times do
        get "/", {}, "REMOTE_ADDR" => "1.2.3.4"
        expect(last_response.status).to_not eq(429)
      end
    end
  end

  context "number of requests is higher than the limit" do
    it "changes the request status to 429" do
      (limit * 2).times do |i|
        get "/", {}, "REMOTE_ADDR" => "1.2.3.5"
        expect(last_response.status).to eq(429) if i > limit
      end
    end
  end
end

describe "throttle excessive POST requests to admin sign in by IP address" do
  let(:limit) { 5 }

  context "number of requests is lower than the limit" do
    it "does not change the request status" do
      limit.times do |i|
        post "/admins/sign_in", { email: "example1#{i}@gmail.com" }, "REMOTE_ADDR" => "1.2.3.6"
        expect(last_response.status).to_not eq(429)
      end
    end
  end

  context "number of admin requests is higher than the limit" do
    it "changes the request status to 429" do
      (limit * 2).times do |i|
        post "/admins/sign_in", { email: "example2#{i}@gmail.com" }, "REMOTE_ADDR" => "1.2.3.8"
        expect(last_response.status).to eq(429) if i > limit
      end
    end
  end
end

describe "throttle excessive POST requests to user sign in by IP address" do
  let(:limit) { 5 }

  context "number of requests is lower than the limit" do
    it "does not change the request status" do
      limit.times do |i|
        post "/users/sign_in", { email: "example3#{i}@gmail.com" }, "REMOTE_ADDR" => "1.2.3.7"
        expect(last_response.status).to_not eq(429)
      end
    end
  end

  context "number of user requests is higher than the limit" do
    it "changes the request status to 429" do
      (limit * 2).times do |i|
        post "/users/sign_in", { email: "example4#{i}@gmail.com" }, "REMOTE_ADDR" => "1.2.3.9"
        expect(last_response.status).to eq(429) if i > limit
      end
    end
  end
end

describe "throttle excessive POST requests to admin sign in by email address" do
  let(:limit) { 5 }

  context "number of requests is lower than the limit" do
    it "does not change the request status" do
      limit.times do |i|
        post "/admins/sign_in", { email: "example5@gmail.com" }, "REMOTE_ADDR" => "#{i}.2.4.9"
        expect(last_response.status).to_not eq(429)
      end
    end
  end

  context "number of requests is higher than the limit" do
    it "changes the request status to 429" do
      (limit * 2).times do |i|
        post "/admins/sign_in", { email: "example6@gmail.com" }, "REMOTE_ADDR" => "#{i}.2.5.9"
        expect(last_response.status).to eq(429) if i > limit
      end
    end
  end
end

describe "throttle excessive POST requests to user sign in by email address" do
  let(:limit) { 5 }

  context "number of requests is lower than the limit" do
    it "does not change the request status" do
      limit.times do |i|
        post "/users/sign_in", { email: "example7@gmail.com" }, "REMOTE_ADDR" => "#{i}.2.6.9"
        expect(last_response.status).to_not eq(429)
      end
    end
  end

  context "number of requests is higher than the limit" do
    it "changes the request status to 429" do
      (limit * 2).times do |i|
        post "/users/sign_in", { email: "example8@gmail.com" }, "REMOTE_ADDR" => "#{i}.2.7.9"
        expect(last_response.status).to eq(429) if i > limit
      end
    end
  end
end
{% endhighlight %}

It was pretty exciting when all these tests passed, and now anyone trying to
brute-force my application or cause other problems is going to face a bit more
of a challenge. So, if you're interested in having a little more security for
your Rails applications, I encourage you to definitely check out Rack::Attack,
and to also try testing it using Rack::Test!
