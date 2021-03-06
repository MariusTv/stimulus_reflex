---
description: How to use StimulusReflex in your app
---

# Quick Start

## Before you begin...

**A great user experience can be created with Rails alone.** Tools such as [UJS remote elements](https://guides.rubyonrails.org/working_with_javascript_in_rails.html#remote-elements), [Stimulus](https://stimulusjs.org/), and [Turbolinks](https://github.com/turbolinks/turbolinks) are incredibly powerful when combined. Could you build your application using these tools without introducing StimulusReflex?

{% hint style="warning" %}
See the [Stimulus TodoMVC](https://github.com/hopsoft/stimulus_todomvc) example application if you are unsure how to do this. **Important note**: this link _does not make use of StimulusReflex_. It is presented for technical comparison and soulful reflection.

We are only alive for a short while and learning any new technology is a sacrifice of time spent with those you love, creating art or walking in the woods.

Every framework you learn is a lost opportunity to build something that could really matter to the world. **Please choose responsibly.**
{% endhint %}

It might strike you as odd that we would start by questioning whether you need this library at all. Our motivations are an extension of the question we hope more people will ask.

Instead of "_Which Single Page App framework should I use?_" we believe that StimulusReflex can empower people to wonder "**Do we still need React, given what we now know is possible?**"

## Hello, Reflex

Bringing your first Reflex to life couldn't be simpler:

1. Declare the appropriate data attributes in HTML.
2. Create a server side reflex object with Ruby.

### Call Reflex methods on the server without any Javascript

This example will automatically update the page with the latest count whenever the anchor is clicked.

{% code title="app/views/pages/index.html.erb" %}
```text
<a href="#"
  data-reflex="click->CounterReflex#increment"
  data-step="1" 
  data-count="<%= @count.to_i %>"
>Increment <%= @count.to_i %></a>
```
{% endcode %}

We use data attributes to declaratively tell StimulusReflex to pay special attention to this anchor link. `data-reflex` is the command you'll use on almost every action. The format follows the Stimulus convention of `[browser-event]->[ServerSideClass]#[action]`. The other two attributes, `data-step` and `data-count` are used to pass data to the server. You can think of them as arguments.

{% code title="app/reflexes/counter\_reflex.rb" %}
```ruby
class CounterReflex < StimulusReflex::Reflex
  def increment
    @count = element.dataset[:count].to_i + element.dataset[:step].to_i
  end
end
```
{% endcode %}

StimulusReflex maps your requests to Reflex classes that live in your `app/reflexes` folder. In this example, the increment method is executed and the count is incremented by 1. The `@count` instance variable is passed to the template when it is re-rendered.

Yes, it really is that simple.

{% hint style="success" %}
**Concerns like managing state and rendering views are handled server side.** This technique works regardless of how complex the UI becomes. For example, we could render multiple instances of `@count` in unrelated sections of the page and they will all update.
{% endhint %}

### Manually call a Reflex from a Stimulus controller

Real world applications will benefit from additional structure and more granular control. Building on the solid foundation that Stimulus provides, we can use Controllers to build complex functionality and respond to events.

Let's build on our increment counter example by adding a Stimulus Controller and manually calling a Reflex action.

1. Declare the appropriate data attributes in HTML.
2. Create a client side StimulusReflex controller with JavaScript.
3. Create a server side Reflex object with Ruby.
4. Create a server side Example controller with Ruby.

{% code title="app/views/pages/index.html.erb" %}
```text
<a href="#"
  data-controller="counter"
  data-action="click->counter#increment"
>Increment <%= @count %></a>
```
{% endcode %}

Here, we rely on the standard Stimulus `data-controller` and `data-action` attributes. There's no StimulusReflex-specific markup required.

{% code title="app/javascript/controllers/counter\_controller.js" %}
```javascript
import { Controller } from 'stimulus';
import StimulusReflex from 'stimulus_reflex';

export default class extends Controller {
  connect() {
    StimulusReflex.register(this)
  }

  increment(event) {
    event.preventDefault()
    this.stimulate('CounterReflex#increment', 1)
  }
}
```
{% endcode %}

The Controller connects during the page load process and we tell StimulusReflex that this Controller is going to be calling server-side Reflex actions. The `register` method has an optional 2nd argument that accepts options, but we'll cover that later.

When the user clicks the anchor, Stimulus calls the `increment` method. All StimulusReflex Controllers have access to the `stimulate` method. The first parameter is the `[ServerSideClass]#[action]` syntax, which tells the server which Reflex class and method to call. The second parameter is an optional argument which is passed to the Reflex method. If you need to pass multiple arguments, consider using a JavaScript object `{}` to do so.

{% hint style="warning" %}
If you're responding to an event like click on an element that would have a default action \(such as an `a` or a `button` element\) it's very important that you call preventDefault\(\) on that event, or else you will experience undesirable side effects such as page navigation.
{% endhint %}

{% code title="app/reflexes/counter\_reflex.rb" %}
```ruby
class CounterReflex < StimulusReflex::Reflex
  def increment(step = 1)
    session[:count] = session[:count].to_i + step
  end
end
```
{% endcode %}

Here, you can see how we accept an optional `step` argument to our `increment` Reflex action. We're also now switching to using the Rails session object to persist our values across multiple page load operations.

{% code title="app/controllers/pages\_controller.rb" %}
```ruby
class PagesController < ApplicationController
  def index
    @count = session[:count].to_i
  end
end
```
{% endcode %}

Finally, we set the value of the @count instance variable in the controller action. When the page is first loaded, there will be no session\[:count\] value and @count will be nil. nil converts to an integer as 0, our starting value.

{% hint style="success" %}
In a typical Rails app, we would set the value of `@count` after fetching it from a persistent data store such as Postgres or Redis. To keep this example simple, we use Rails' `session` to store our counter value.
{% endhint %}

## StimulusReflex Generator

The StimulusReflex generator is like scaffolding for StimulusReflex:

```bash
bundle exec rails generate stimulus_reflex user
```

This will create but not overwrite the following files:

1. `app/javascript/controllers/application_controller.js`
2. `app/javascript/controllers/user_controller.js`
3. `app/reflexes/application_reflex.rb`
4. `app/reflexes/user_reflex.rb`

