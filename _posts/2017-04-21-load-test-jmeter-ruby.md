---
layout: post
title: Load test using Ruby and JMeter
subtitle: How can we use Ruby to create JMeter tests
tags: ruby jmeter load-testing
---

In this post I will describe the steps to start creating load tests to your
API or application, in Ruby, that will use [Apache JMeter](http://jmeter.apache.org/)
to run the tests and collect the metrics.

# Why Apache JMeter

Because it's open source and can be scriptable to run in headless mode, avoiding
all the overhead GUI's create, specially in an application written in Java.
Also, because it's written in Java we can take advantage of the thread system, and
squeeze our machines to do more requests.
On Google, is one of the top results for `HTTP load test software`,
on Stackoverflow, when asking for a [tool to perform stress tests on web application](http://stackoverflow.com/q/7492/1006863)
it also came as the [first suggestion](http://stackoverflow.com/a/92501/1006863).

# Setup the base application to be tested

Instead of creating another useless blog application to perform the tests against it,
I will use a code base I'm already familiar. My choice is [Spree Commerce](https://github.com/spree/spree).

### Get Spree up and running

I'm assuming that you have a working ruby system on your machine, if not install ruby, rubygems and bundler.

1. Install Ruby on Rails: `gem install rails`
2. Create a new Rails app: `rails new spree`
3. Add Spree to the `Gemfile`:
    ```ruby
    gem 'spree', '~> 3.2.0'
    gem 'spree_auth_devise', '~> 3.2.0.beta'
    gem 'spree_gateway', '~> 3.2.0.beta'
    ```
4. Install the dependencies: `bundle install`
5. Run the migrations and seed data: `rails g spree:install`
6. Press <kbd>ENTER</kbd> when asked for Email and Password, so they will be `spree@example.com`
and `spree123`
7. Start the application: `rails s`
8. Go to your [User Profile](http://localhost:3000/admin/users/1/edit)
9. Get the API Key ![API Key]({{ site.baseurl }}/assets/images/posts/jmeter-spree-api-key.png)
10. Do a trial request with cURL `curl --header "X-Spree-Token:a41383d518f8a8d8bbfad7a749c88c1b928f20c99a4c8a15" http://localhost:3000/api/v1/products.json`


### Install Apache JMeter

The fun part of Ruby is we're using it for a DSL to create the Apache JMeter tests.
The hard work is done by the [ruby-jmeter gem](https://github.com/flood-io/ruby-jmeter) from
[flood.io](https://flood.io/).

1. Install Java (Yeah I know... life is not perfect!)
2. Install Apache JMeter and make it available on your `$PATH`.
  In my case I've saved the file on `~/tmp` and used `~/bin` because it's already in my `$PATH`
  1. Download the `tgz` file from http://jmeter.apache.org/download_jmeter.cgi
  2. Extract it:  `tar -xzf apache-jmeter-3.1.tgz -C tmp`
  3. Link the binary: `ln -s ~/tmp/apache-jmeter-3.2/bin/jmeter ~/bin/jmeter`
  4. Execute 'jmeter -v' and check the output to confirm it's working
3. Install 'ruby-jmeter' gem with `gem install ruby-jmeter`

# Create and running tests

The first test I'll write will be to mimic the request I've used with `curl` previously.
So in a `load-test.rb` file I will add:

{% highlight ruby linenos %}
auth_token = 'a41383d518f8a8d8bbfad7a749c88c1b928f20c99a4c8a15'
test do
  threads count: 10 do
    header({name: 'X-Spree-Token', value: auth_token})
    visit name: 'Products listing', url: 'http://localhost:3000/api/v1/products.json'
  end
end.run
{% endhighlight %}

Now you have a `jmeter.jtl` file you can use to see the results on Apache JMeter. To see the results
and explore ways to get the data visualization, you can add listeners to your test plan and
then load the `jtl` file.
Another option, is to instruct Apache JMeter to create a log with a summary of the tests. To do this
we need to create a `jmeter.properties` file and fill with the following content:

{% highlight properties linenos %}
# Define the following property to automatically start a summariser with that name
# (applies to non-GUI mode only)
summariser.name=summary
#
# interval between summaries (in seconds) default 3 minutes
summariser.interval=180
#
# Write messages to log file
summariser.log=true
#
# Write messages to System.out
summariser.out=true
{% endhighlight %}

After creating the file add `properties: 'jmeter.properties'` to the run arguments in the test.
You can see more details in the [summarizer documentation](https://jmeter.apache.org/usermanual/component_reference.html#Generate_Summary_Results)

After a second run you should be able to see the results in the log file:
{% highlight properties %}
INFO o.a.j.r.Summariser: summary +      4 in 00:00:02 =    1.8/s Avg: 10667 Min: 10308 Max: 11156 Err:     0 (0.00%) Active: 0 Started: 10 Finished: 10
INFO o.a.j.r.Summariser: summary =     59 in 00:01:09 =    0.9/s Avg: 10664 Min:  2053 Max: 12874 Err:     0 (0.00%)
{% endhighlight %}

# Conclusions
With this Ruby DSL, it's very easy to create a maintainable version of our load tests and still have the flexibility to control some outputs.
Also, we can still use JMeter to draw some graphics if needed, like if we had specified all the tests in the JMeter itself.
So we get the best of both worlds, first the ability to easily define our tests and second the power to use the same old tools to do the dirty job.


### Disclaimer
I'm not affiliated with flood.io by any means, I mention them because I appreciate their work on the gem.
Also, you don't need any plan to run the tests on your machines, because the gem is open source.
But if you need a cloud option for load tests, please take a look at their page.
