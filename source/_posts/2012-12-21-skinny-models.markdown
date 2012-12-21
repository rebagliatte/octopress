---
layout: post
title: "Skinny Models"
date: 2012-12-21 10:38
comments: true
sharing: true
published: true
categories: ruby
author: Sebastian Borrazas

description: We are presenting our Rails Rumble entry, tripbudget.us. The app allows people to create trips, pick destinations and estimate their expenses in a clean an organized way, centralizing all the alternatives that came up as well as the discussions about them.
---

Many times I found myself with fat models (ugh) having lots of methods for different formats of the data they have so they can be easily accessed on the views/controllers.
Here's a few examples of it:

``` ruby
class Project

  # Example 1
  attr_accessor :tasks_count, :completed_tasks_count

  def pretty_completed_percentage # Not pretty :(
    "#{completed_tasks_count * 100 / tasks_count}% completed"
  end

  # Example 2
  attr_accessor :comments

  def last_comment
    doc = Nokogiri::HTML(comments.last.content) # Crying already
    # ...
    doc
  end

  # Example 3
  def to_json
    # WHY ?!?!?
  end
end
```

<!-- more -->

All of these things could have been made out of the actual project attributes which are publicly visible.
The problem here is that we are delegating the presentation and formatting of the data in the models.
To many this might seem OK, but to me, models should not be responsible for it, and should remain skinny.

``` ruby
class Project
  attr_accessor :tasks_count, :completed_tasks_count, :started_at
end
```

A model is basically in charge of dealing with the data storage and application logic. You shouldn't treat your models as an API willing to do whatever you need.
In an MVC environment, adding all these logic into the controllers might not be a very good idea either, since you might end up stuffing too much code into them.
The best solution (which has worked for me) is to add new 'services' (or w/e you want to call it) in between and leave models alone. After all, MVC might not be the best for large web applications..
