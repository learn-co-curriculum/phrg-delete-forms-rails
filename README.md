# DELETE Forms and Requests

So far, we've worked with three pieces of the CRUD puzzle:

- **C**REATING records, using HTTP `POST` requests.
- **R**EADING records, using HTTP `GET` requests.
- **U**PDATING records, using HTTP `PATCH` requests.

One piece remains:

- **D**ELETING records, using HTTP `DELETE` requests.

But, all is not well in Browsertown. In many cases, sending a request with the `PATCH` or `DELETE` method will not work. In this lesson, we'll focus on `DELETE`, but many of the same issues arise with `PATCH` requests, as well.

Why? What can we do as a workaround?


# Objectives

After this lesson, you'll be able to...

- Draw a `delete` route mapping to a `#destroy()` action
- Explain the problem with submitting `delete` requests
- Use `form_tag` to build a delete form for an object
- Build a `#destroy()` action that finds the instance, destroys it, and redirects to the `index` action
- Use `link_to` and `button_to :method => :delete` to destroy an object without a form


# Ignorance is bliss

Before we dive into the problem with `DELETE` (and `PATCH`) requests, let's proceed as if we were none the wiser, setting up our route and form as usual:

```ruby
# config/routes.rb

delete 'people/:id', to: 'people#destroy'
```

```erb
# app/views/people/show.html.erb

<h2><%= @person.name %></h2>
<%= @person.email %>
<%= form_tag people_path(@person.id), method: "delete" %>
  <%= submit_tag "Delete #{@person.name}" %>
<% end %>
```

But, wait a minute... there's something weird about the output we get:

```html
<h2>Caligula</h2>
caligula@rome-circa-40-AD.com
<form accept-charset="UTF-8" action="/people/1" method="post">
  <input name="_method" type="hidden" value="delete" />
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input name="authenticity_token" type="hidden" value="f755bb0ed134b76c432144748a6d4b7a7ddf2b71" />
  <input name="commit" type="submit" value="Delete Caligula" />
</form>
```

Enhance!
```html
<form accept-charset="UTF-8" action="/people/1" method="**post**">
  <input name="_method" type="hidden" value="**delete**" />
```

What's going on? Why the extra input?


# Programming is hard

Web developers love to be on the cutting edge. We hoover up new tools and techniques and aren't afraid of breaking a few million eggs to figure out that we actually have no idea how to make an omelette. Projects like Rails themselves are a product of this incredible devotion to progress and automation.

**Browser** and **server** developers are the yin to the web developers' yang. These are the people who own and maintain the tools themselves: Internet Explorer, Firefox, Chrome, Apache, and so on.

When a web developer makes a mistake, it might affect the users of their site.

When a browser or server developer makes a mistake, it might affect [***over a billion people***](http://venturebeat.com/2015/05/28/google-chrome-now-has-over-1-billion-users/)!

Because of this, browser/server developers have to go slow. *Really* slow. And they have to resist the urge to release "duct tape" solutions because duct tape doesn't scale to a billion users!

This means that, sometimes, incomplete solutions can remain in place for years, or even decades, while the maintainers go back and forth trying to find a better approach that won't open an eldritch portal to Bosch's [Garden of Earthly Delights](https://www.khanacademy.org/humanities/renaissance-reformation/northern/hieronymus-bosch/a/bosch-the-garden-of-earthly-delights).


# What's all this have to do with `DELETE` requests?

As of HTML5, forms officially do not support `DELETE` and `PATCH` for their methods.

There's no short and sweet answer to explain this. If you want to dive deep and understand as much as possible about the decisions that went into it, you can read more on [this illuminating StackExchange post](http://programmers.stackexchange.com/questions/114156/why-are-there-are-no-put-and-delete-methods-on-html-forms), but, for the purposes of succinct explanation, you can always stick with the tried and true "for historical reasons."

What you're seeing in the above `#form_tag()` behavior is a **workaround** implemented for us by Rails itself. With this in mind, we get the best of both worlds:

- We get to be **good HTTP-abiding citizens** who use the correct request methods for their corresponding goals (`GET` for read, `PATCH` for update, and so on).
- We get to **maintain our sanity** and not worry about W3C drama while writing views.


# That's great. Can we actually delete something now?

Thus enlightened, we can (finally) proceed with our original goal:

```ruby
# app/controllers/people_controller.rb

  def destroy
    Person.find(params[:id]).destroy
    redirect_to people_url
  end
```

Nothing too special happening here except for a bit of method-chaining to immediately destroy the found instance.


# Fancy JavaScript Helper

As shown, you have to go to a user's `show` page to delete them. What if we want an admin control panel where users can be deleted from a list?

```erb
<!-- app/views/people/index.html.erb //-->

<% @people.each do |person| %>
<div class="person">
  <span><%= person.name %></span>
  <%= link_to "Delete", person, method: :delete, data: { confirm: "Really?" } %>
</div>
<% end %>
```

[`link_to`](http://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to) is a method of `UrlHelper` that has a number of convenient features.

The HTML generated by that call to `link_to` looks like this:

```html
<a data-confirm="Really?" rel="nofollow" data-method="delete" href="/people/1">Delete</a>
```

The `data-confirm` attribute and the `data-method` attribute rely on some JavaScript built into Rails.

`data-method` will "submit" a `DELETE` request as if a form had been submitted. It will use `GET` (the default method used by all browsers for HTML links) if the user has JavaScript disabled.

`data-confirm` pops up a confirmation window before the link is followed, allowing the user to make sure they're ready to delete someone forever (what a decision!).

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/delete-forms-rails' title='DELETE Forms and Requests'>DELETE Forms and Requests</a> on Learn.co and start learning to code for free.</p>
