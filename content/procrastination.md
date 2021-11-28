+++
title = "Procrastinate: Lazy loading for the lazy developer"
date = 2021-11-28
draft = false
[extra]
show_date = true
+++

When I was playing with [turbo](https://turbo.hotwired.dev/) the other day, I worked out a simple trick for adding lazy loading to a rails app without the overhead of new routes and controllers. It's a bit of a hack, but I thought it was interesting so I wanted to share. You can find the standard approach [here](https://boringrails.com/tips/turboframe-lazy-load-skeleton). That strategy works nicely, but migrating an existing page to use it is much more complex than what I'll show you.

Note: My sample code uses the [view_component](https://viewcomponent.org/) library, but it's not essential.

Say you have some expensive calculation - e.g. the sum of all transactions, or a very expensive table:

```erb
<%= render "expensive_calculation" %>
```

The `procrastinate` helper lets you rewrite your code like:
```erb
<%= procrastinate do |component| %>
  <% component.loader do %>
    <em>Loading</em>
  <% end %>

  <% component.body do %>
    <%= render "expensive_calculation" %>
  <% end %>
<% end %>
```

When the page first loads, you will see the contents of the `loader` block. Then another request with a `procrastinate=false` query parameter. This tells us to render the `body` block. Turbo then replaces the loader with the body.

How does this work? It's just two files. A helper like this: 

```ruby
# app/helpers/procrastination_helper.rb
module ProcrastinationHelper
  def procrastinate
    wait = ActiveModel::Type::Boolean.new.cast(params["procrastinate"] || true)

    options = {}
    options[:src] = add_query_param(request.path, "procrastinate", "false") if wait

    turbo_frame_tag "procrastinate", **options do
      render(ProcrastinationComponent.new(procrastinate: wait)) do |component|
        yield component
      end
    end
  end

  private

  def add_query_param(path, key, value)
    URI(path).tap do |uri|
      new_query = URI.decode_www_form String(uri.query)
      new_query << [key, value]
      uri.query = URI.encode_www_form new_query
    end.to_s
  end
end
```

And a view component that conditionally renders the loader or the body:

```ruby
# app/components/procrastination_component.rb
class ProcrastinationComponent < ApplicationComponent
  renders_one :loader
  renders_one :body

  def initialize(procrastinate:)
    @procrastinate = procrastinate
  end

  def call
    @procrastinate ? loader : body
  end
end
```
And that's it! Magic 🪄

# This approach has a couple of downsides.
1. Parts of the page are unnecessary rendered twice. The user shouldn't notice, but it's unnecessary work for the server.
2. It only works if you need to lazy load a single part of the page. For granular lazy loading, you should use the standard approach.
3. The server still has to perform the expensive calculation every time it's requested. Depending on how expensive it is, caching may be a better approach. (On the flip side, this approach gives a speed boost without the complexity of caching!)

# When would I use this helper? 
When the benefit of simplicity outweighs the potential downsides. Maybe an admin tool? A site that has low traffic and a lot of aggregated content?

I'm likely not the first person to come up with this idea, but I can't find a reference to it anywhere. Let me know if you use it! :D
