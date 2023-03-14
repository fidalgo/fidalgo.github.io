---
layout: post
title: First Padrino framework contribution
subtitle: How a find for bug ended in one contribution
tags: ruby padrino
---

For those who don't know what Padrino is, it's a web framework written in Ruby,  known to be
lighter compared with Ruby on Rails, but complete enough to create full-blown web applications.

Padrino allows setting default values in the controllers to be passed as URL parameters
in case we don't provide it, either via HTTP request parameters or via parameters in the
`absolute_url` method, to compute the final URL of a request.
In a quest to find the source of an odd behaviour, I ended finding a bug in the
framework.


So, in order to isolate the problem I've created a [sample application](https://github.com/fidalgo/padrino-absolute-url)
and the most important code is this controller:

{% highlight ruby linenos %}
module Padre
  App.controller provider: 'google' do
    get :index, map: '/:provider/index' do
    end

    post :a_to_a_to_agent_answered, map: '/:provider/a2a/:call_sid/agent/:agent_id/answered' do
    end
  end
end
{% endhighlight %}

In this case I set the default provider to google, so the parameters in the method should
include `{provider: 'google'}`.

That was not the case, so I need to keep digging!

The first thing I've start looking was at the tests, and I found a very interesting test:
{% highlight ruby linenos %}
it 'should use default values' do
  mock_app do
    controller :lang => :it do
      get(:index, :map => "/:lang") { "lang is #{params[:lang]}" }
    end
    # This is only for be sure that default values
    # work only for the given controller
    get(:foo, :map => "/foo") {}
  end
  assert_equal "/it",  @app.url(:index)
  assert_equal "/foo", @app.url(:foo)
  get "/en"
  assert_equal "lang is en", body
end
{% endhighlight %}
To check if the padrino code was performing as expected I've added the line:
`assert_equal "/pt", @app.url(:index, lang: 'pt')`

and got the confirmation of the bug:
```
Expected: "/pt"
  Actual: "/it"
```

So using a TDD approach start digging in the code and found the problematic code:

{% highlight ruby linenos %}
##
# Expands the path by using parameters.
#
def expand(params)
  params = params.merge(@default_values) if @default_values.is_a?(Hash)
  params, query = params.each_with_object([{}, {}]) do |(key, val), parts|
    parts[handler.names.include?(key.to_s) ? 0 : 1][key] = val
  end
  expanded_path = handler.expand(:append, params)
  expanded_path += ?? + Padrino::Utils.build_uri_query(query) unless query.empty?
  expanded_path
end
{% endhighlight %}

In the method's first line, the code is using the given parameters and merging them
with the defaults ones, but what we want the other way around.
We want the default parameters to be overridden by the ones we provide. So it seems a quick fix!

So I went to read the [contribution guidelines](https://github.com/padrino/padrino-framework#bug-reporting) and opened a [bug](https://github.com/padrino/padrino-framework/issues/2113).
Later on, I've opened a [Pull Request](https://github.com/padrino/padrino-framework/pull/2114).

It was good to see that if you have a good Git work flow and follow some principles on creating issues and commit messages, it will ease the acceptance.
So in this case, it was accepted without any question!
